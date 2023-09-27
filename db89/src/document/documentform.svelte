<script>
    import { onMount } from "svelte";

    import { DateInput } from 'date-picker-svelte';
    let date = new Date();

    // @ts-ignore
    const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value; // do global


    const send = async function (data, url){
        // let notificationdata = {...notification_success_data};
        try {
            const response = await fetch(url, {
                method: 'post',
                headers: {
                    'Content-Type': 'application/json',
                    'X-CSRFToken': csrftoken,
                },
                body: JSON.stringify(data),
            });
            if(response.ok) {
                // dispatch(eventname, {'response': await response.json()})
                return response.json();
            }
            else {
                throw(new Error(`${response.status} - ${response.statusText}`)) // <br>${getErrorData(await response.text())}`));
            }
        }
        catch (error) {
            // notificationdata = {...notification_error_data};
            // notificationdata.text += error.message;
            alert(error.message);
            return error;
        }
        finally {
            // dispatch(notificationeventname, notificationdata);
        }
    }



    import Inputlist from "../lib/form/inputlist.svelte";
    const placeholders = {
        sender: 'отправитель',
        payer: 'плательщик',
        receiver: 'получатель',
    }
    let list_org = [{name: 'загрузка...'}]; // список объектов для поиска
    const class_list = ['input', 'is-small',];
    let selected = {    // для записи результата выбора
        sender: {},
        payer: {},
        receiver: {},
    };
    /**
     * Получаем массив из объектов DocumentItem из строк табличной части
     */
    const getDocumentItems = () => {
        let docitemsdata = []; // объект с данными, который вернём
        let rows = document.querySelectorAll('tr.item');    // строки
        rows.forEach(tr => {
            let row_data = {};                              // данные строкИ для массива
            let inputs = tr.querySelectorAll('input');      // все инпуты на строке
            inputs.forEach(input => {
                row_data[input['name']] = input["value"];   // сохраняем name: value
            });
            row_data.name = row_data.search;                // из виджета DateInput надо "search" переименовать в "name"
            delete row_data.search;
            row_data.price = inputToFloat(row_data.price);  // в формат числа
            row_data.item_sum = inputToFloat(row_data.item_sum);
            if (row_data.item_sum != 0 && row_data.name.trim() != '') { // только если есть наименование и сумма
                docitemsdata.push(row_data);
            }
            
        });
        return docitemsdata;
    }

    let documentformdata = {};
    /**
     * Собираем все данные с форм и инпутов для сохранения в БД
     * @param e может надо убрать... Сделать отдельный хендлер?
     */
    const getDocumentFormData = async (e) => {
        // @ts-ignore
        documentformdata.number = document.querySelector('[name="number"]').value; // номер документа
        // @ts-ignore
        let _date = document.querySelector('.date-time-field input').value.split('.'); // дата dd.mm.YYYY->[dd,mm,YYYY]
        documentformdata.date = _date[2] + '-' + _date[1] + '-' + _date[0]; // YYYY-mm-dd
        let _form = e.currentTarget;
        const _formData = new FormData(_form);
        let _formObject = Object.fromEntries(_formData.entries());    // формат в объект
        // @ts-ignore
        _formObject.doc_sum = inputToFloat(document.querySelector('#documentform-sum').textContent); // сумма документа
        // let itemsdata = getDocumentItems();     // табличная часть
        documentformdata = {...documentformdata, ..._formObject,} // ...selected,}; // ...itemsdata}; // собираем всё вместе
        documentformdata.sender = selected.sender.id;
        documentformdata.receiver = selected.receiver.id;
        documentformdata.payer = selected.payer.id;
        if (!(documentformdata.number.trim() && documentformdata.date.trim())) {
            alert("надо заполнить номер и дату обязательно");
            return
        }
        let _formdata = {...documentformdata};
        let _response = await send(_formdata, url.documents);    // documetn is saved, TODO: parse errors
        let itemsdata = getDocumentItems();
        itemsdata.forEach(item => {
            // alert(JSON.stringify(_response));
            item.document = _response.id;
            let _resp = send(item, url.documentitems);
        });
        // 1 - сохранить документ (обработка валидации)
        // 2 - перебирая, сохранить каждую запись таблицы (обработка валидации)
        // успешно завершено - очистить форму и (?) закрыть модальное окно.
    }

    const url = {
        base: "http://127.0.0.1:8000/api/v1/",              // dev
        // @ts-ignore
        // base: "https://" + document.domain + '/api/v1/', // deploy 
        get org() {return this.base + "organizations/"},
        get num() {return this.base + "documentnumber/"},
        get documents() {return this.base + "documents/"},
        get documentitems() {return this.base + "documentitems/"}
    };
    
    async function fetchListOrg () {
        const _data = await fetch(url.org).then((x) => x.json());
        list_org = JSON.parse(JSON.stringify(_data, ['name', 'inn', 'id',]));
    }

    let doc_number = "";
    async function fetchDocumentNumber () {
        const _data = await fetch(url.num).then((x) => x.json());
        doc_number = _data["number"];
    }

    onMount(()=>{
        fetchListOrg();
        // Назначим классы
        for (let item of class_list) {
            let _input_element = document.querySelectorAll('[data-svelte-search] input, .date-time-field input');
            _input_element.forEach(element => {
                element.classList.add(item);
            });
        }
    })
    /**
     * Из объекта даты делает строку dd.mm.yyyy
     * @param date
     */
    function getDateString(date) {
        return date.toLocaleDateString();
    }
    export let actiate_form = false; // bind: по этому флагу запускаем момент открытия формы
$:  if (actiate_form) { 
        fetchDocumentNumber();
        document.querySelector('[name=number]').setAttribute('disabled', 'true');   // number disabled, on:doubleclick=>enabled
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
</script>

<div class="box">
    <h5 class="title is-5 has-text-centered">Товарно-транспортная накладная</h5>
    
    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label" on:click={fetchDocumentNumber}>Номер и дата</label>
        </div>
        <div class="field-body">
            <div class="field is-grouped is-justify-content-space-between">
                <div class="control" on:dblclick={(e)=>e.currentTarget.querySelector('[name=number]').removeAttribute('disabled')}>
                    <input type="text" class="input is-small" placeholder="номер" name="number" maxlength="15"
                        bind:value={doc_number} required disabled>
                </div>
                <p class="control control-dateinput">
                    <DateInput 
                        value={date} 
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
                        placeholder={getDateString(date)} />
                </p>
            </div>
        </div>
    </div>

    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label">Город</label>
        </div>
        <div class="field-body">
            <div class="field">
                <p class="control">
                    <input type="text" class="input is-small" placeholder="город" 
                        name="city" maxlength="100" form="documentform">
                </p>
            </div>
        </div>
    </div>

    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label">Номер ТС</label>
        </div>
        <div class="field-body">
            <div class="field">
                <p class="control">
                    <input type="text" class="input is-small" placeholder="№ ТС"
                        name="truck" maxlength="20" form="documentform">
                </p>
            </div>
        </div>
    </div>
    
    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label">Отправитель</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <Inputlist 
                        placeholder={placeholders.sender}
                        data={list_org}
                        on:select={({ detail }) => (selected.sender = detail.original)} 
                        on:clear={() => selected.sender = {}} />
                </div>
            </div>
        </div>
    </div>

    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label">Получатель</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <Inputlist 
                        placeholder={placeholders.receiver} 
                        data={list_org}
                        on:select={({ detail }) => (selected.receiver = detail.original)} 
                        on:clear={() => selected.receiver = {}} />
                </div>
            </div>
        </div>
    </div>

    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label">Плательщик</label>
        </div>
        <div class="field-body">
            <div class="field">
                <div class="control">
                    <Inputlist 
                        placeholder={placeholders.payer} 
                        data={list_org}
                        on:select={({ detail }) => (selected.payer = detail.original)} 
                        on:clear={() => selected.payer = {}} />
                </div>
            </div>
        </div>
    </div>
    
    <div class="field">
        <div class="control">
            <slot />
        </div>
    </div>
    
    <form id="documentform" on:submit|preventDefault={getDocumentFormData}>
        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Особые отметки</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <textarea class="textarea is-small" name="spec_notes" placeholder="особые отметки" rows="2"></textarea>
                    </p>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Адрес доставки</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <input type="text" class="input is-small" placeholder="адрес доставки"
                            name="destination_address" maxlength="256">
                    </p>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Условия договора</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <textarea class="textarea is-small" name="text" id="" placeholder="условия договора" rows="3"></textarea>
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
                        <button class="button is-small is-outlined" type="reset">Отмена</button>
                    </p>
                </div>
            </div>
        </div>
    </form>
    <pre>{JSON.stringify(documentformdata, null, 2)}</pre>
</div>


<style>
    :global(.control-dateinput) {
        width: 15rem;
    }
    :global(.control-dateinput .date-time-field input) {
        width: 100%;
    }
</style>