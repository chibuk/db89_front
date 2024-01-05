<script>
    export let editDocument_id = -1;    // -1 (создание документа) | id (редактирование документа, или копирование)
    export let copy = false;            // true if copy mode
    import { onMount } from "svelte";
    import { DateInput } from 'date-picker-svelte';
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    import Docitemform from "./docitemform.svelte";

    // @ts-ignore
    const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value; // do global
    const url = {
        // base: "http://127.0.0.1:8000/api/v1/",              // dev
        // @ts-ignore
        base: "https://" + document.domain + '/api/v1/', // deploy 
        get org() {return this.base + "organizations/"},
        get num() {return this.base + "documentnumber/"},
        get documents() {return this.base + "documents/"},
        get documentitems() {return this.base + "documentitems/"}
    };

    import SimpleAutoComplete from "../lib/form/SimpleAutoComplete.svelte";
    let ac_props = {
        labelFieldName: "name",
        valueFieldName: "id",
        maxItemsToShowInList: 4,
        delay: 350,
        minCharactersToSearch: 2,
        create: false,
        lowercaseKeywords: false,
        // hideArrow: true,        // не работало пришлось убрать в исходниках hideArrow и showClear
        noInputStyles: true,
        // showClear: true,  // заменил на <input type=search>
        showLoadingIndicator: true,
        loadingText: "Загрузка...",
        moreItemsText: "не показано",
        noResultsText: "Не найдено",
        // createText: 'Не найдено, "Enter", чтобы создать.',
        placeholder: "наименование или ИНН",
        className: 'is-size-7',
        inputClassName: "input is-small",
        // name: "", // input name задаем каждому инстансу на месте
        searchFunction: searchOrganization, // по ИНН или названию
        localFiltering: false,      // чтобы при поиске по ИНН не скрывались результаты
        required: true,
        // onCreate: createItem,    // делать или нет, пока не решил
    }

    async function searchOrganization (keyword) {
        const response = await fetch(url.org + "search/" + encodeURIComponent(keyword));
        return await response.json();
    }

    function clearForm() {
        selected.sender = '';
        selected.receiver='';
        selected.payer='';
        let _qSelector = `input[name=name], input[name=seats], input[name=volume], 
                        input[name=weight], input[name=price], input[name=item_sum],
                        input[name=payer], input[name=receiver], input[name=sender]`
        let _inputs = document.querySelectorAll(_qSelector);
        _inputs.forEach(input => {
            // @ts-ignore
            input.value = '';
        });
        if (documentformdata.id) {  // документ редактируется, очищаем кроме id, number, date
            let id = documentformdata.id;
            let number = documentformdata.number;
            let date = documentformdata.date;
            documentformdata = DocumentClass(id, number, date);
        } else documentformdata = DocumentClass();
    }
    /**
     * Реализуем возможность "досохранить" документ. После его записи (Document POST), 
     * некоторые строки таблицы (DocumentItem POST)
     * могут не пройти валидацию и придется их исправлять и снова нажимать "сохранить", но
     * документ уже в БД записан, повторно записать тот же документ нельзя, поэтому делаем PUT,
     * т.е. перезапись существующего. Наличие значения переменной -> метод PUT.
     */
    // documentformdata.id = '';   // идентификатор текущего документа после сохранения
    const postData = async function (data, url, method='post'){
        let answer = {
            ok: false,   // успешно (true)
            resp: {},
        }
        try {
            const response = await fetch(url, {
                method: method,
                headers: {
                    'Content-Type': 'application/json',
                    'X-CSRFToken': csrftoken,
                },
                body: JSON.stringify(data),
            });
            if(response.ok) {
                answer.ok = true;
                answer.resp = await response.json();
            }
            else {
                answer.resp = await response.json();
            }
        }
        catch (error) {
            answer.resp.error = error.message + ' - Нет связи с сервером.';
        }
        finally {
            return answer;
        }
    }

    let selected = {    // для записи результата выбора (id)
        sender: '',
        payer: '',
        receiver: '',
    }
    /**
     * Получаем массив для объектов DocumentItem из строк табличной части и сохраняем в БД.
     * Парсит построчно табличную часть. Успешно "сохранённые строки" помечает атрибутом
     * tr data-id="id", чтобы при повторном прогоне его проверить и пропустить строку.
     * Если строка с ошибкой (нет наименования, количества или что-то в ответе сервера),
     * подсвечиваем и пропускаем её, продолжаем со следующей строки.
     * Если все строки сохранились, return true, и окно закроется,
     * если не все, то false и окно останется открытым для исправления подсвеченных ошибок.
     * Если режим copy, то в первую очередь удаляем со всех строк data-id...
     */
    const postDocumentItems = async (document_id) => {
        let _return = {
            ok: true,
            // sum: 0,  // сумму документа теперь считает сервер после каждого DocumentItem
        }
        let rows = document.querySelectorAll('tr.item');    // строки
        for(let tr of rows) {
            let inputs = tr.querySelectorAll('input');      // все инпуты на строке
            let row_data = {};                              // данные строкИ для массива
            let _method = "post";                           // POST | PUT
            let _url = url.documentitems;
            // @ts-ignore
            if (copy) tr.dataset.id = '';
            // @ts-ignore
            if (tr.dataset.id) { row_data.id = tr.dataset.id;
                _method = "put";
                _url = url.documentitems + row_data.id + "/";
            } // если есть id, значит строка редактируется TODO:...
            inputs.forEach(input => {
                row_data[input['name']] = input["value"];   // сохраняем name: value
            });
            row_data.price = inputToFloat(row_data.price);  // в формат числа
            row_data.item_sum = inputToFloat(row_data.item_sum);
            row_data.seats = parseInt(row_data.seats);
            if (row_data.name.trim() === '' && !row_data.id) { // если наименование не заполнено м строка не из БД
                if (row_data.item_sum == 0) continue ;      // и нет суммы, то игнорируем строку
                // @ts-ignore
                fillErrorDocumentFormData({name: ['Введите наименование!']}, target=tr);
                _return.ok = false;
                continue;
            } 
            else { // если есть наименование
                row_data.document = document_id;
                let response = await postData(row_data, _url, _method);
                if (!response.ok) {
                    // @ts-ignore
                    fillErrorDocumentFormData(response.resp, target=tr); // подсветим ошибки
                    _return.ok = false;
                    continue;
                }
                else { // сохранилось успешно
                    // @ts-ignore
                    tr.dataset.id = response.resp.id;   // id запмсм в БД
                }
            }
        }
        copy = false;
        return _return;    // все строки сохранились, либо нечего сохранять
    }

    function DocumentClass (
        id='',                  // для операции редактирования существующего документа
        number='',              
        date='',                // "YYYY-mm-dd", дата
        city='',                // Город
        truck='',               // авто
        spec_notes='',          // особые отметки
        destination_address='', // адрес доставки
        text='',                // условия договора
        doc_sum=0.0,            // всего к оплате
        sender=undefined,       // id отправтеля
        receiver=undefined,     // id получателя
        payer=undefined) {      // id плательщика
        return {
            id, number, date, city, truck, spec_notes, destination_address, text, doc_sum, sender, receiver, payer
        }
    }
    
    /**
     * В данном объекте поля связаны тегами через bind. поэтому можно им назначить значение оно отобразится,
     * только поля из сторонних виджетов недоступны для связывания: date, sender, payer, receiver
     * их надо обновлять отдельно (для редактирования документа или обработки ошибок)
     */
    let documentformdata = DocumentClass();
    // let documentformdata = {
    //     "id": "",       // для операции редактирования существующего документа
    //     "number": "",   // "Т03", номер документа
    //     "date": "",     // "2003-01-01", дата
    //     "city": "",     // Город
    //     "truck": "",    // Номер авто
    //     "spec_notes": "",   // особые отметки
    //     "destination_address": "", // адрес доставки
    //     "text": "",     // условия договора
    //     "doc_sum": 0,   // всего к оплате
    //     "sender": undefined,    // id отправтеля
    //     "receiver": undefined,  // id получателя
    //     "payer": undefined,     // id плательщика
    // };
    /**
     * здесь данные предыдущего документа БД для автозаполнения по клику на Label
     * или исходные данные документа, который сейчас на редактировании
     * */ 
    let lastdocument = DocumentClass();   
    async function fillFieldHandler() {
        if (!copy && editDocument_id === -1) {
            lastdocument = await fetch(url.documents+'last/').then((x) => x.json());
        }
        let field = this.parentNode.parentNode.querySelector('input');
        if (!field) field = this.parentNode.parentNode.querySelector('textarea');
        documentformdata[field.name] = lastdocument[field.name];
    }
    async function fillOrgHandler() {
        if (!copy && editDocument_id === -1) {
            lastdocument = await fetch(url.documents+'last/').then((x) => x.json());
        }
        let field = this.parentNode.parentNode.querySelector('input');
        // @ts-ignore
        document.querySelector(`input[name=${field.name}]`).value = lastdocument[field.name]['name'];
        selected[field.name] = lastdocument[field.name]['id'];
    }
    /**
     * Собираем все данные с форм и инпутов
     */
    const postDocumentFormData = async () => {
        // @ts-ignore
        let _date = document.querySelector('.date-time-field input').value.split('.').reverse().join('-'); // дата dd.mm.YYYY->YYYY-mm-dd
        documentformdata.date = _date
        documentformdata.sender = parseInt(selected.sender);
        documentformdata.receiver = parseInt(selected.receiver);
        documentformdata.payer = parseInt(selected.payer);
        // проверки заполнения формы
        let _status = {
            ok: true,
            data: {}
        }
        if (!documentformdata.number.trim()) {
            _status.ok = false;
            _status.data.number = ["Введите номер!"];
        }
        if (!/^\d\d\d\d-\d\d-\d\d$/.test(documentformdata.date)) {
            alert("Формат даты документа не верный");
            return
        }
        if (!documentformdata.sender){
            _status.ok = false;
            _status.data.sender = ["Выберите отправителя, начните вводить ИНН или наименование!"];
        }
        if (!documentformdata.receiver){
            _status.ok = false;
            _status.data.receiver = ["Выберите получателя, начните вводить ИНН или наименование!"];
        }
        if (!documentformdata.payer){
            _status.ok = false;
            _status.data.payer = ["Выберите плательщика, начните вводить ИНН или наименование!"];
        }
        if (!_status.ok) {
            fillErrorDocumentFormData(_status.data);
            return
        }
        let _method = 'post';
        let _int_pk = '';
        if (documentformdata.id) {    // если этот документ был уже сохранен
            _method = "put";
            _int_pk = documentformdata.id + '/';
        }
        // сохраняем в БД сервера
        let postDocument = await postData(documentformdata, url.documents+_int_pk, _method);    // document is saved
        if (!postDocument.ok) { // обработаем ошибки - подсветим поля и вставим текст сообщения об ошибке
            fillErrorDocumentFormData(postDocument.resp)
            return
        }
        dispatch('updatetable');    // отправим событие, обновить список документов
        documentformdata.id = postDocument.resp.id;
        let itemsdata = await postDocumentItems(postDocument.resp.id);
        if (itemsdata.ok) {
            dispatch('closemodal');
        }
    }

    /**
     * сервер вернул объекты с ошибками {field: "error_message"}
     * назначаем соответствующим полям клвсс "is-danger" и title='error_message'
     * а ещё однократный обработчик события "input", чтобы убрать выделение,
     * клгда пользователь начнет исправлять содержимое поля.
     * @param data
     */
    const fillErrorDocumentFormData = (data, target=document) => {
        for (let key in data) {         // перебираем объекты
            let element = target.querySelector(`[name=${key}]`);
            element.setAttribute('title', data[key]);
            element.classList.add('is-danger');
            element.addEventListener("input", (e)=>{
                // @ts-ignore
                e.currentTarget.classList.remove('is-danger');
                // @ts-ignore
                e.currentTarget.removeAttribute('title');
            }, {once: true});
        }
    }

    async function getDocumentNumber () {
        let number = '';
        if (documentformdata.id && !copy) number = lastdocument.number;
        else {
            const _data = await fetch(url.num).then((x) => x.json());
            number = _data["number"];
        }
        documentformdata.number = number;
    }
    /**
     * редактирование, либо создание копии документа editDocument_id
     */
    async function getDocumentEditData () {
        lastdocument = await fetch(url.documents+editDocument_id+'/').then((x) => x.json());
        lastdocument.date = lastdocument.date.split('-').reverse().join('.'); // 2022-01-02 ->  02.01.2022
        if (copy) {
            lastdocument.id = undefined;
            await getDocumentNumber();
            lastdocument.number = documentformdata.number;
            lastdocument.date = new Date().toLocaleDateString();
        }
        // @ts-ignore
        document.querySelector('.date-time-field input').value = lastdocument.date;
        for (let field of ['id','number','city','truck','spec_notes','destination_address',
            'text','doc_sum']) documentformdata[field] = lastdocument[field];
        // @ts-ignore
        document.querySelector('input[name=sender]').value = lastdocument['sender']['name'];
        // @ts-ignore
        document.querySelector('input[name=receiver]').value = lastdocument['receiver']['name'];
        // @ts-ignore
        document.querySelector('input[name=payer]').value = lastdocument['payer']['name'];
        selected.sender = lastdocument['sender']['id'];
        selected.receiver = lastdocument['receiver']['id'];
        selected.payer = lastdocument['payer']['id'];
    }

    onMount(()=>{
        // Назначим классы
        let _dateInputTag = document.querySelector('.date-time-field input');
        _dateInputTag.classList.add('input');
        _dateInputTag.classList.add('is-small');
        // New number
        if (editDocument_id !== -1) getDocumentEditData();
        else {
            getDocumentNumber();
        }
        document.querySelector('[name=number]').setAttribute('disabled', 'true');
    })

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
</script>

<div class="box">
    <h5 class="title is-5 has-text-centered">Товарно-транспортная накладная</h5>
    
    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label get" on:click={getDocumentNumber}>Номер и дата</label>
        </div>
        <div class="field-body">
            <div class="field is-grouped is-justify-content-space-between">
                <div class="control" on:dblclick={(e)=>e.currentTarget.querySelector('[name=number]').removeAttribute('disabled')}>
                    <input type="text" class="input is-small" placeholder="номер" name="number" maxlength="15"
                        bind:value={documentformdata.number} required disabled>
                </div>
                <p class="control control-dateinput">
                    <DateInput 
                        value={new Date()} 
                        format='dd.MM.yyyy'
                        locale={
                            {
                                weekdays: [ 'Вс', 'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб'],
                                months: ['Янаврь', 'Февраль', 'Март', 'Апрель', 'Май',
                                        'Июнь', 'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь'],
                                weekStartsOn: 1
                            }
                        }
                        closeOnSelection={true}
                        placeholder={new Date().toLocaleDateString()} />
                </p>
            </div>
        </div>
    </div>

    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label get" on:click={fillFieldHandler}>Город</label>
        </div>
        <div class="field-body">
            <div class="field">
                <p class="control">
                    <input type="text" class="input is-small" placeholder="город" 
                        name="city" maxlength="100" form="documentform" bind:value={documentformdata.city}>
                </p>
            </div>
        </div>
    </div>

    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label get" on:click={fillFieldHandler}>Авто</label>
        </div>
        <div class="field-body">
            <div class="field">
                <p class="control">
                    <input type="text" class="input is-small" placeholder="№ ТС"
                        name="truck" maxlength="20" form="documentform" bind:value={documentformdata.truck}>
                </p>
            </div>
        </div>
    </div>
    
    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label get" on:click={fillOrgHandler}>Отправитель</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <SimpleAutoComplete {...ac_props} 
                        placeholder={ac_props.placeholder + ' отправителя'}
                        name="sender"
                        bind:value={selected.sender} />
                </div>
            </div>
        </div>
    </div>

    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label get" on:click={fillOrgHandler}>Получатель</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <SimpleAutoComplete {...ac_props} 
                        placeholder={ac_props.placeholder + ' получателя'}
                        name="receiver"
                        bind:value={selected.receiver} />
                </div>
            </div>
        </div>
    </div>

    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label get" on:click={fillOrgHandler}>Плательщик</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <SimpleAutoComplete {...ac_props} 
                        placeholder={ac_props.placeholder + ' плательщика'}
                        name="payer"
                        bind:value={selected.payer} />
                </div>
            </div>
        </div>
    </div>
    
    <div class="field">
        <div class="control">
            <Docitemform {editDocument_id} />
        </div>
    </div>
    
    <form id="documentform" on:submit|preventDefault={postDocumentFormData} on:reset={clearForm}>
        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label get" on:click={fillFieldHandler}>Особые отметки</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <textarea class="textarea is-small" name="spec_notes" placeholder="особые отметки" rows="2"
                            bind:value={documentformdata.spec_notes}></textarea>
                    </p>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label get" on:click={fillFieldHandler}>Адрес доставки</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <input type="text" class="input is-small" placeholder="адрес доставки"
                            name="destination_address" maxlength="256" bind:value={documentformdata.destination_address}>
                    </p>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label get" on:click={fillFieldHandler}>Условия договора</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <textarea class="textarea is-small" name="text" id="" placeholder="условия договора" rows="3"
                        bind:value={documentformdata.text}></textarea>
                    </p>
                </div>
            </div>
        </div>
        
        <div class="field is-horizontal">
            <div class="field-label is-small">
                <label for="" class="label"></label>
            </div>
            <div class="field-body">
                <div class="field is-grouped">
                    <p class="control is-expanded">
                        <button class="button is-small is-link is-outlined is-fullwidth" type="submit">Сохранить</button>
                    </p>
                    <p class="control">
                        <button class="button is-small is-outlined" type="reset">Очистить</button>
                    </p>
                </div>
            </div>
        </div>
    </form>
</div>

<style>
    :global(.control-dateinput) {
        width: 15rem;
    }
    :global(.control-dateinput .date-time-field input) {
        width: 100%;
    }
    :global(label.get) {
        cursor: pointer;
    }
    :global(label.get:hover) {
        color: grey;
    }
</style>