<script>
    import { createEventDispatcher } from 'svelte';
    import Inputstyle from '../lib/inputstyle.svelte';
    import { onMount } from 'svelte';
    export let id = undefined;  // "0" (или эквивалент false) - создание новой записи | id - редактирование
    const dispatch = createEventDispatcher();
    const notificationeventname = 'notification';
    const notification_error_data = {
                add_class: "is-danger",
                text: 'Не удалось сохранить: <br>'
    };

    const fields = [
        {
            name: "name", 
            placeholder: "Название организации", 
            type: "text",
            maxlength: 50,
            onlynumbers: false,
        },
        {
            name: "address", 
            placeholder: "Адрес", 
            type: "text",
            maxlength: 256,
            onlynumbers: false,
        },
        {
            name: "inn", 
            placeholder: "ИНН", 
            type: "text",
            maxlength: 10,
            onlynumbers: true,
        },
        {
            name: "kpp", 
            placeholder: "КПП", 
            type: "text",
            maxlength: 9,
            onlynumbers: true,
        },
        {
            name: "ogrn", 
            placeholder: "ОГРН", 
            type: "text",
            maxlength: 15,
            onlynumbers: true,
        },
        {
            name: "bank", 
            placeholder: "Наименование банка", 
            type: "text",
            maxlength: 150,
            onlynumbers: false,
        },
        {
            name: "bik", 
            placeholder: "БИК", 
            type: "text",
            maxlength: 9,
            onlynumbers: true,
        },
        {
            name: "pay_account", 
            placeholder: "Расчетный счет", 
            type: "text",
            maxlength: 20,
            onlynumbers: true,
        },
        {
            name: "kor_account", 
            placeholder: "Корр. счет", 
            type: "text",
            maxlength: 20,
            onlynumbers: true,
        },
        {
            name: "phone", 
            placeholder: "Номера телефонов", 
            type: "text",
            maxlength: 40,
            onlynumbers: false,
        },
        {
            name: "email", 
            placeholder: "Email", 
            type: "email",
            maxlength: 254,
            onlynumbers: false,
        },
    ]

// @ts-ignore
// const url = "http://127.0.0.1:8000/api/v1/organizations/";
const url = "https://" + document.domain + app.dataset.api;
// @ts-ignore
const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value; // do global

const send = async function (data, method='post') {
    let _url = url;
    if (id) _url += id + "/";
    try {
        const response = await fetch(_url, {
            method: method,
            headers: {
                'Content-Type': 'application/json',
                'X-CSRFToken': csrftoken,
            },
            body: data,
        });
        if(response.ok) {
            dispatch('updatetable'); // loadTable();
            return true;
        }
        else {
            fillErrors(await response.json());
            throw(new Error(`${response.status} - ${response.statusText}`));
        }
    }
    catch (error) {
        let notificationdata = {...notification_error_data};
        notificationdata.text = error.message;
        dispatch(notificationeventname, notificationdata);
        return false;
    }
}

function fillErrors (data) {
    const _inputstyle = document.querySelector('#organizationform');
    for (let key in data) {
        _inputstyle[key].parentNode.style.border = '1px solid red';
        _inputstyle[key].parentNode.setAttribute('title', data[key]);
        _inputstyle[key].addEventListener('input', (e)=>{
            e.currentTarget.parentNode.removeAttribute('title');
            e.currentTarget.parentNode.style.border = '1px solid #999';
        }, {once: true})
    }
}

const perform_submit = async (e) => {  // Извлекаем данные, отправляем, очищаем форму, (?) закрываем модал.
    let _form = e.currentTarget;
    const _formData = new FormData(_form);
    let json_data = JSON.stringify(Object.fromEntries(_formData.entries())); // Convert FormData->Object->JSON
    let _method = "post";
    if (id) _method = "patch";
    else json_data['id'] = '';
    if (await send(json_data, _method)) _form.reset();
}

const perform_reset = () => {
    _name = '';
    _inn = '';
    id = '';
    dispatch("close");
}

let disabled = true;    // для кнопки, не активна или активна

/**
 * Редактирование записи. Закгружаем из БД имеющиеся данные для редактирования
 * и заполняем или форму.
 */
async function fill_edit_data () {
    const _form = document.querySelector('#organizationform')
    const _content = await fetch(`${url}${id}/`).then((x) => x.json());
    for (let key in _content) {
        _form[key].value = _content[key];
    }
    _name = _content['name'];
    _inn = _content['inn'];
}

// @ts-ignore
let _name = '';
function checkname (e) {
    _name = e.currentTarget.value;
}

// @ts-ignore
let _inn = '';
function checkinn (e) {
    onlyNumbers(e);
    _inn = e.currentTarget.value;
}

const setHandlers = () => {
    const _form = document.querySelector('#organizationform');
    _form['name'].addEventListener('input', (e)=>checkname(e));
    _form['inn'].addEventListener('input', (e)=>checkinn(e));
    for (let key of fields) {
        if (key.onlynumbers) _form[key.name].addEventListener('input', (e)=>onlyNumbers(e));
    }
}

// Оставляет только цифры. Для инпута.
function onlyNumbers (e) {
    e.currentTarget.value = e.currentTarget.value.replace(/\D/g, '');
}

$: disabled = (_inn.length === 10 && _name.length > 3) ? false : true;

onMount(()=>{
    setHandlers();
    if (!id) return;
    // Загрузим данные,
    // Заполним форму данными
    fill_edit_data();
})

</script>
<form method="post" class="box" id='organizationform'
    on:submit|preventDefault={perform_submit} 
    on:reset={perform_reset}>
    {#each fields as field }
        <!-- <div class="field">
            <div class="control"> -->
                <!-- {#if field.name === 'name'}
                <input class="input" type="{field.type}" 
                    name="{field.name}" placeholder="{field.placeholder}" maxlength="{field.maxlength}"
                    on:input={checkname}>
                {:else if field.name === 'inn'}
                <input class="input" type="{field.type}"   
                    name="{field.name}" placeholder="{field.placeholder}" maxlength="{field.maxlength}"
                    on:input={checkinn}>
                {:else if field.onlynumbers}
                <input class="input" type="{field.type}" 
                    name="{field.name}" placeholder="{field.placeholder}" maxlength="{field.maxlength}"
                    on:input={onlyNumbers}>
                {:else} -->
        <Inputstyle name={field.name} label={field.placeholder} maxlength={field.maxlength} type={field.type} />
                <!-- <input class="input" type="{field.type}" 
                    name="{field.name}" placeholder="{field.placeholder}" maxlength="{field.maxlength}"> -->
                <!-- {/if} -->
            <!-- </div>
        </div> -->
    {/each}
    {#if id} 
        <input type="hidden" name='id'>
    {/if}
    <div class="field is-grouped is-grouped-centered">
        <div class="control is-expanded">
            <button type="submit" class="button is-link is-fullwidth" {disabled}>Сохранить</button>
        </div>
        <div class="control is-expanded">
            <button type="reset" class="button is-fullwidth">Отмена</button>
        </div>
    </div>   
</form>

<style>
    input::placeholder {
        color: #555;
    }
</style>
