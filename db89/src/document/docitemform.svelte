<script>
    // переменная для работы в режиме редактирования
    export let editDocument_id = -1;    // если будет другое значение, то это id документа
    let editMode = false;
    $: editMode = (editDocument_id===-1) ? false : true;    // для удобства

    import SimpleAutoComplete from "../lib/form/SimpleAutoComplete.svelte";
    let ac_props = {
        labelFieldName: "name",
        // valueFieldName: "id",
        maxItemsToShowInList: 5,
        delay: 350,
        minCharactersToSearch: 2,
        create: true,
        lowercaseKeywords: false,
        // hideArrow: true,        // не работало пришлось убрать в исходниках hideArrow и showClear
        noInputStyles: true,
        // showClear: true,  // заменил на <input type=search>
        showLoadingIndicator: true,
        loadingText: "Загрузка...",
        moreItemsText: "не показано",
        createText: 'Не найдено, "Enter", чтобы создать.',
        placeholder: "наименование груза",
        className: undefined,
        inputClassName: "input is-small",
        name: "name", // input name
        localFiltering: false,
        searchFunction: searchItem,
        onCreate: createItem,
    }
    // @ts-ignore
    const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value; // do global
    const url = {
        // base: "http://127.0.0.1:8000/api/v1/",              // dev
        // @ts-ignore
        base: "https://" + document.domain + '/api/v1/', // deploy 
        get org() {return this.base + "organizations/"},
        // get num() {return this.base + "documentnumber/"},
        get items() {return this.base + "items/"},
        get documentitems() {return this.base + "documentitems/"},
        get doclist() {return this.base + "documentitems/doclist?doc_id="}
    };
    
    async function createItem(_text, method='post') {
        const response = await fetch(url.items, {
            method: method,
            headers: {
                'Content-Type': 'application/json',
                'X-CSRFToken': csrftoken,
            },
            body: JSON.stringify({"name": _text}),
        });
        if(response.ok) {
            return await response.json();
        }
        else {
            return JSON.stringify({"id": -1, "name": `Ошибка^ ${response.status} - ${response.statusText}`});
        }
    }
    // let ac_items = [             // поля id нам не нужны совсем, на сервер отправляем name
    //     {"id":2,"name":"Коробка"},
    //     {"id":3,"name":"Контейнер"},
    //     {"id":4,"name":"Баг: при редактировании"},
    //     {"id":8,"name":"Обработать Item"},
    //     {"id":220,"name":"Ошибка"}
    // ];
    async function searchItem (keyword) {
        const response = await fetch(url.items + "search/" + encodeURIComponent(keyword));
        return await response.json();
    }
    
    import { onMount } from "svelte";
    
    /* Удаляем элемент, возвращает ''(для false), если успешно или текст http ошибки ответа */
    const deleteDocumentItem = async (id) => {
        try {
            const response = await fetch(`${url.documentitems}${id}/`, {
                method: 'delete',
                headers: {
                    'X-CSRFToken': csrftoken,
                }
            })
            if (response.ok) return '';
            else return `<br> id: ${id}, ${response.status} - ${response.statusText}`;
        }
        catch (er) {
            return `<br> id: ${id}, ${er.message}`;
        }
    }
    
    /**
     * Назначаем всем a.removeline обработчик удаления строки таблицы
     */
    const setRemoveLineHandlers = () => {
        let removeline_tags = document.querySelectorAll('a.removeline');
        removeline_tags.forEach(function(a_tag){
            // @ts-ignore
            a_tag.onclick = async function (e) {
            // a_tag.addEventListener('click', async function(e){
                e.preventDefault();
                // @ts-ignore
                const tr = e.currentTarget.parentNode.parentNode;
                if (tr.dataset.id) {
                    const resp = await deleteDocumentItem(tr.dataset.id);
                    if (resp) {alert("Ошибка!!! Не могу удалить!" + resp); return};
                }
                tr.remove();    // removing line TODO: if (dataset.id) -> remove DocumentItem(id)
                countTotalSumm();
            }
        })
    }

    onMount(()=>{
        setHandlers();
        if (editMode) fillDocumeentItems();
    });
    // Добавление строки в конец таблицы
    const appendLine = () => {
        const line_HTML = `<tr class="item"><td class="count"><span></span></td>
            <td></td>
            <td><input type="text" class="num input is-small" name="seats"></td>
            <td><input type="text" class="num input is-small" name="weight"></td>
            <td><input type="text" class="num input is-small" name="volume"></td>
            <td><input type="text" class="rub input is-small" name="price"></td>
            <td><input type="text" class="rub input is-small" name="item_sum">
                <a class="removeline" title="удалить строку" href="">
                    <span class="icon is-small has-text-grey-light"><i class="fa-light fa-xmark"></i></span>
                </a>
            </td>
        </tr>`;
        let table_tag = document.getElementById('docitemform_table'); // e.currentTarget.parentNode.parentNode.parentNode.parentNode;
        // @ts-ignore
        table_tag.childNodes[2].insertAdjacentHTML('beforeend', line_HTML);
        // create Inputlist instance
        let simpleAC_instance = new SimpleAutoComplete({
            // @ts-ignore
            target: table_tag.childNodes[2].lastChild.childNodes[2],
            props: { ...ac_props }
        })
        setHandlers();   // строке(ам) назначаем обработчики
    }
    /**
     * Преобразование строки из Input'а в число Float для вычислений.
     * Оставляем тольок цифры, заменяем запятую на точку...
     * если будет NaN, то вернет ноль.
     * @param str
     */
    function inputToFloat(str) {
        let float = parseFloat(str.replace(/\s/g, '').replace(',', '.'));
        return isNaN(float) ? 0 : float // это исключает появление надписи "не число"
    }
    /**
     * После вычислений, "денежный" Float форматируем в "денежную" строку
     * с пробелами и запятой для копеек.
     * @param float
     */
    function floatToInput(float) {
        return (float==0) ? '' : new Intl.NumberFormat('ru-RU', { minimumFractionDigits: 2, maximumFractionDigits: 2 }).format(float);
    }

    let total_summ = '';    // строка "Всего к оплате" в футере таблицы
    /**
     * Расчет суммы "Всего к оплате"
     */
    const countTotalSumm = () => {
        let _total = 0.0;
        let tbody = document.querySelector('#docitemform_table tbody');
        let inputs = tbody.querySelectorAll('[name=item_sum]');
        inputs.forEach(input => {
            // @ts-ignore
            _total += inputToFloat(input.value);
        });
        total_summ = floatToInput(_total);
    }
    /**
     * Вычмсляем стоимость, умножая seats на price
     */
    function countSum(e) {
        let tr = e.currentTarget.parentNode.parentNode; // тег строки таблицы
        let _seats = inputToFloat(tr.querySelector('[name=seats]').value); // берем сразу value, т.к. там type=number
        let _price = inputToFloat(tr.querySelector('[name=price]').value);
        let _summ = (_seats * _price);
        tr.querySelector('[name=item_sum]').value = floatToInput(_summ);
        countTotalSumm();
    }
    /**
     * Форматирование числа 1123.45 => 1 123,45
     */
    const formatPrice = (e) => {
        let _value = inputToFloat(e.currentTarget.value);
        e.currentTarget.value = floatToInput(_value);
    }
    /**
     * Функция назначает обработчики событий на элементы в строках табличной части
     * - сначала на все input[name=price] элементы 
     * - затем на input[name=seats]
     * - в конце, обработчики нажатий на ссылки с крестиком, чтобы удалить строку
     * и классы к инпуту новой строки
     */
    function setHandlers() {
        let price_inputs = document.querySelectorAll('[name=price]');
        price_inputs.forEach(function(price_input){
            price_input.addEventListener('change', (e)=>formatPrice(e)); // форматирование числа
            price_input.addEventListener('input', (e)=>{
                // @ts-ignore
                e.currentTarget.value=e.currentTarget.value.replace(/[^0-9,.]/g,'')
                countSum(e)
            });     // подсчет суммы
        })
        let seats_inputs = document.querySelectorAll('[name=seats]');
        seats_inputs.forEach(function(seats_input){
            seats_input.addEventListener('input', (e)=>{
                // @ts-ignore
                e.currentTarget.value=e.currentTarget.value.replace(/[^0-9]/g,'')
                countSum(e)
            });
        })
        let summ_inputs = document.querySelectorAll('[name=item_sum]');
        summ_inputs.forEach(element => {
            // @ts-ignore
            element.addEventListener('input', (e)=>e.currentTarget.value=e.currentTarget.value.replace(/[^0-9,.]/g,''));
            element.addEventListener('blur', (e)=>{
                formatPrice(e);
                countTotalSumm();
            });
        });
        setRemoveLineHandlers();
    }

async function fillDocumeentItems () {
    // if (!editMode) return {};
    const _data = await fetch(url.doclist + encodeURIComponent(editDocument_id)).then((x) => x.json());
    const data_len = _data.length 
    if (data_len > 2) {
        for (let i=2; i<data_len ; i++) appendLine();
    }
    const _lines = document.querySelectorAll('tr.item'); // tr
    for (let i=0; i<data_len ; i++) {
        // @ts-ignore
        _lines[i].querySelector(`[name=name]`).value = _data[i].item.name;
        // @ts-ignore
        _lines[i].querySelector(`[name=seats]`).value = parseInt(_data[i].seats);
        // @ts-ignore
        _lines[i].querySelector(`[name=weight]`).value = _data[i].weight;
        // @ts-ignore
        _lines[i].querySelector(`[name=volume]`).value = _data[i].volume;
        // @ts-ignore
        _lines[i].querySelector(`[name=price]`).value = floatToInput(inputToFloat(_data[i].price));
        // @ts-ignore
        _lines[i].querySelector(`[name=item_sum]`).value = floatToInput(inputToFloat(_data[i].item_sum));
        countTotalSumm()
        // @ts-ignore
        _lines[i].dataset.id = _data[i].id;
        // alert(JSON.stringify(line));
    }
}
</script>

<table class="table content is-small" id="docitemform_table">
    <thead>
        <tr>
            <th>№</th>
            <th>Наименование</th>
            <th>Кол-во мест</th>
            <th>Вес</th>
            <th>Объем</th>
            <th>Цена (&#x20BD;)</th>
            <th>Сумма (&#x20BD;)</th>
        </tr>
    </thead>
    <tbody>
        <tr class="item">
            <td class="count"><span></span></td>
            <td><SimpleAutoComplete {...ac_props} /></td>
            <td><input type="text" class="num input is-small" name="seats"></td>
            <td><input type="text" class="num input is-small" name="weight"></td>
            <td><input type="text" class="num input is-small" name="volume"></td>
            <td><input type="text" class="rub input is-small" name="price"></td>
            <td><input type="text" class="rub input is-small" name="item_sum">
                <a class="removeline" title="удалить строку" href="">
                    <span class="icon is-small has-text-grey-light"><i class="fa-light fa-xmark"></i></span>
                </a>
            </td>
        </tr>
        <tr class="item">
            <td class="count"><span></span></td>
            <td><SimpleAutoComplete {...ac_props} /></td>
            <td><input type="text" class="num input is-small" name="seats"></td>
            <td><input type="text" class="num input is-small" name="weight"></td>
            <td><input type="text" class="num input is-small" name="volume"></td>
            <td><input type="text" class="rub input is-small" name="price"></td>
            <td><input type="text" class="rub input is-small" name="item_sum">
                <a class="removeline" title="удалить строку" href="">
                    <span class="icon is-small has-text-grey-light"><i class="fa-light fa-xmark"></i></span>
                </a>
            </td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td></td>
            <td><a href="" on:click|preventDefault={appendLine}>+ строка</a></td>
            <td></td> <td></td> <td></td> <td align="right">Всего к оплате:</td>
            <td><strong>{total_summ}</strong></td>
        </tr>
    </tfoot>
</table>

<style>
/**Пришлось стили глобально установить, классы прописать, т.к. таблица динамически растёт,
на новые строки скомпилированный идентификатор Svelte класса не применяется */
    :global(table#docitemform_table) {
        counter-reset: lines;
        width: 100%;
    }

    :global(.input.is-small) {           /* table#docitemform_table input */
        padding-top: calc(0.2em - 1px);
        padding-bottom: calc(0.2em - 1px);
        padding-left: calc(0.4em - 1px);
        padding-right: calc(0.4em - 1px);
        height: 1.5rem;
        font-size: .85rem;
        vertical-align: middle;
    }

    :global(table#docitemform_table tr.item) {
        counter-increment: lines;
    }

    :global(table#docitemform_table .item .count span::before) {
        content: counter(lines);
    }

    :global(table#docitemform_table td) {
        vertical-align: middle;
    }

    :global(#docitemform_table .item .num) {
        max-width: 4rem;
    }
    :global(#docitemform_table tr.item .rub) {
        max-width: 6rem;
    }
    :global(#docitemform_table tr.item td) {
        white-space: nowrap;
    }
    :global(#docitemform_table tr.item td input) {
        outline-color: rgba(91, 124, 201, .5);
        
    }
    :global(#docitemform_table i.fa-xmark:hover) {
        color: brown;
    }
</style>