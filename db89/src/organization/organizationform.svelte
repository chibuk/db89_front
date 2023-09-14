<script>
    import { createEventDispatcher } from 'svelte';
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
            maxlength: 50
        },
        {
            name: "address", 
            placeholder: "Адрес", 
            type: "text",
            maxlength: 256
        },
        {
            name: "inn", 
            placeholder: "ИНН", 
            type: "text",
            maxlength: 10
        },
        {
            name: "kpp", 
            placeholder: "КПП", 
            type: "text",
            maxlength: 9
        },
        {
            name: "ogrn", 
            placeholder: "ОГРН", 
            type: "text",
            maxlength: 15
        },
        {
            name: "bank", 
            placeholder: "Наименование банка", 
            type: "text",
            maxlength: 150
        },
        {
            name: "bik", 
            placeholder: "БИК", 
            type: "text",
            maxlength: 9
        },
        {
            name: "pay_acount", 
            placeholder: "Расчетный счет", 
            type: "text",
            maxlength: 20
        },
        {
            name: "kor_acount", 
            placeholder: "Корр. счет", 
            type: "text",
            maxlength: 20
        },
        {
            name: "phone", 
            placeholder: "Номера телефонов", 
            type: "text",
            maxlength: 40
        },
        {
            name: "email", 
            placeholder: "Email", 
            type: "email",
            maxlength: 254
        },
    ]

// @ts-ignore
// const url = "http://127.0.0.1:8000/api/v1/organizations/";
const url = "https://" + document.domain + app.dataset.api;
// @ts-ignore
const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value; // do global

const send = async function (data) {
    try {
        const response = await fetch(url, {
            method: 'post',
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

const perform_submit = async (e) => {  // Извлекаем данные, отправляем, очищаем форму, (?) закрываем модал.
    let _form = e.currentTarget;
    const _formData = new FormData(_form);
    let json_data = JSON.stringify(Object.fromEntries(_formData.entries())); // Convert FormData->Object->JSON
    if (await send(json_data)) _form.reset();
}

const perform_reset = () => {
    _name = '';
    _inn = '';
}

let disabled = true;

let _name = '';
function checkname (e) {
    _name = e.currentTarget.value;
}

let _inn = '';
function checkinn (e) {
    _inn = e.currentTarget.value;
}

$: disabled = (_inn.length == 10 && _name.length > 3) ? false : true;

</script>
<form method="post" class="box" id='organizationform'
    on:submit|preventDefault={perform_submit} 
    on:reset={perform_reset}>
    {#each fields as field }
        <div class="field">
            <div class="control">
                {#if field.name == 'name'}
                <input class="input" type="{field.text}" name="{field.name}" placeholder="{field.placeholder}" maxlength="{field.maxlength}"
                    on:input={checkname}>
                {:else if field.name == 'inn'}
                <input class="input" type="{field.text}" name="{field.name}" placeholder="{field.placeholder}" maxlength="{field.maxlength}"
                    on:input={checkinn}>
                {:else}
                <input class="input" type="{field.text}" name="{field.name}" placeholder="{field.placeholder}" maxlength="{field.maxlength}">
                {/if}
            </div>
        </div>
    {/each}
    <div class="field is-grouped is-grouped-centered">
        <div class="control is-expanded">
            <button type="submit" class="button is-link is-fullwidth" {disabled}>Сохранить</button>
        </div>
        <div class="control is-expanded">
            <button type="reset" class="button is-fullwidth">Очистить</button>
        </div>
    </div>   
</form>
