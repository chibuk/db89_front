<script>
    import { onMount } from "svelte";
    /**
     * Назначаем всем a.removeline обработчик удаления строки таблицы
     */
    const setRemoveLineHandlers = () => {
        let removeline_tags = document.querySelectorAll('a.removeline');
        removeline_tags.forEach(function(a_tag){
            a_tag.addEventListener('click', function(e){
                e.preventDefault();
                // @ts-ignore
                const tr = e.currentTarget.parentNode.parentNode;  
                tr.remove();    // removing line
            })
        })
    }
    onMount(setHandlers);
    // Добавление строки в конец таблицы
    const appendLine = (e) => {
        const line_HTML = `<tr class="item"><td class="count"><span></span></td>
            <td><input type="text" name="name" class='input is-small'></td>
            <td><input type="number" class="num input is-small" name="seats"></td>
            <td><input type="text" class="num input is-small" name="weight"></td>
            <td><input type="text" class="num input is-small" name="volume"></td>
            <td><input type="text" class="rub input is-small" name="price"></td>
            <td><input type="text" class="rub input is-small" name="summ">
                <a class="removeline" title="удалить строку" href="">
                    <span class="icon is-small has-text-grey-lighter"><i class="fa-light fa-xmark"></i></span>
                </a>
            </td>
        </tr>`;
        let table_tag = e.currentTarget.parentNode.parentNode.parentNode.parentNode;
        table_tag.childNodes[2].insertAdjacentHTML('beforeend', line_HTML);
        setHandlers();   // строке(ам) назначаем обработчик удаления
    }
    /**
     * Вычмсляем стоимость, умножая seats на price
     */
    function countSum(e) {
        let tr = e.currentTarget.parentNode.parentNode; // тег строки таблицы
        let _seats = parseFloat(tr.querySelector('[name=seats]').value); // берем сразу value, т.к. там type=number
        let _price = parseFloat(tr.querySelector('[name=price]').value.replace(/\s/g, '').replace(',', '.'));
        let _summ = (_seats * _price);
        tr.querySelector('[name=summ]').value = isNaN(_summ) // это исключает появление надписи "не число"
            ? '' 
            : new Intl.NumberFormat('ru-RU', { minimumFractionDigits: 0, maximumFractionDigits: 2 }).format(_summ);
    }
    
    /**
     * Форматирование числа 1123.45 => 1 123,45
     */
    const formatPrice = (e) => {
        // убрать пробелы и запятую заменить на точку
        let _val = parseFloat(e.currentTarget.value.replace(/\s/g, '').replace(',', '.'));
        e.currentTarget.value = isNaN(_val) 
            ? '' 
            : new Intl.NumberFormat('ru-RU', { minimumFractionDigits: 0, maximumFractionDigits: 2 }).format(_val);
    }

    /**
     * Функция назначает обработчики событий на элементы в строках табличной части
     * - сначала на все input[name=price] элементы 
     * - затем на input[name=seats]
     * - в конце, обработчики нажатий на ссылки с крестиком, чтобы удалить строку
     */
    function setHandlers() {
        let price_inputs = document.querySelectorAll('[name=price]');
        price_inputs.forEach(function(price_input){
            price_input.addEventListener('change', (e)=>formatPrice(e)); // форматирование числа
            price_input.addEventListener('input', (e)=>countSum(e));     // подсчет суммы
        })
        let seats_inputs = document.querySelectorAll('[name=seats]');
        seats_inputs.forEach(function(seats_input){
            seats_input.addEventListener('input', (e)=>countSum(e));
        })
        setRemoveLineHandlers();
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
            <td><input type="text" name="name" class="input is-small"></td>
            <td><input type="number" class="num input is-small" name="seats"></td>
            <td><input type="text" class="num input is-small" name="weight"></td>
            <td><input type="text" class="num input is-small" name="volume"></td>
            <td><input type="text" class="rub input is-small" name="price"></td>
            <td><input type="text" class="rub input is-small" name="summ"></td>
        </tr>
        <tr class="item">
            <td class="count"><span></span></td>
            <td><input type="text" name="name" class="input is-small"></td>
            <td><input type="number" class="num input is-small" name="seats"></td>
            <td><input type="text" class="num input is-small" name="weight"></td>
            <td><input type="text" class="num input is-small" name="volume"></td>
            <td><input type="text" class="rub input is-small" name="price"></td>
            <td><input type="text" class="rub input is-small" name="summ">
                <a class="removeline" title="удалить строку" href="">
                    <span class="icon is-small has-text-grey-lighter"><i class="fa-light fa-xmark"></i></span>
                </a>
            </td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td></td>
            <td><a href="" on:click|preventDefault={appendLine}>+ строка</a></td>
            <td></td><td></td><td></td><td></td><td></td>
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