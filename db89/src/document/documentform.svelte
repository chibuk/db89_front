<script>
    import { onMount } from "svelte";

    import { DateInput, DatePicker } from 'date-picker-svelte';
    let date = new Date();

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

    const url = {
        base: "http://127.0.0.1:8000/api/v1/",              // dev
        // @ts-ignore
        // base: "https://" + document.domain + '/api/v1/', // deploy 
        // get item() {return this.base + "items/" },
        get org() {return this.base + "organizations/"},
        get num() {return this.base + "documentnumber/"},
    };
    
    async function fetchListOrg () {
        const _data = await fetch(url.org).then((x) => x.json());
        list_org = JSON.parse(JSON.stringify(_data, ['name', 'inn',]));
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
        document.querySelector('[name=number]').setAttribute('disabled', 'true');
    }
</script>

<div class="box">
    <h5 class="title is-5 has-text-centered">Товарно-транспортная накладная</h5>
    
    <div class="field is-horizontal">
        <div class="field-label is-small is-hidden-mobile">
            <label for="" class="label">Номер и дата</label>
        </div>
        <div class="field-body">
            <div class="field is-grouped is-justify-content-space-between">
                <p class="control" on:dblclick={(e)=>e.currentTarget.querySelector('[name=number]').removeAttribute('disabled')}>
                    <input type="text" class="input is-small" placeholder="номер" name="number" maxlength="15"
                        bind:value={doc_number} required disabled>
                </p>
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
                        name="city" maxlength="100">
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
                        name="truck" maxlength="20">
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
    
    <form>
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
    <pre>{JSON.stringify(selected, null, 2)}</pre>
</div>


<style>
    :global(.control-dateinput) {
        width: 15rem;
    }
    :global(.control-dateinput .date-time-field input) {
        width: 100%;
    }
</style>