<script>
    import { createEventDispatcher } from 'svelte';

    const props = {
        label: "Создать:",
        help: "",
        placeholder: "Введите наименование",
        maxlength: 128,
        name: "name",
        id: "id_name",
        danger: false,
        value: "",
    }

    // @ts-ignore
    const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value; // do global
    const url = "http://127.0.0.1:8000/api/v1/items/";
    const dispatch = createEventDispatcher();
    const eventname = 'newitem';
    const notificationeventname = 'notification';
    const notification_success_data = {
                add_class: "is-success is-light",
                text: '<i class="fa-light fa-check"></i>'
    };
    const notification_error_data = {
                add_class: "is-danger",
                text: 'Не удалось сохранить: <br>'
    };

    const additem = async (event) => {
        const _form = event.currentTarget;
        let data = {name: _form.elements[0].value};
        if (await send(data)) _form.elements[0].value = '';   // else ...
        else props.danger = true;
    }

    /* Стоит ли оставлять такую функцию или переделать BackEnd?
    Ошибку бэкенд возвращает в виде HTML документа. Данная функция возвращает заголовок.
    TODO: Передать ошибку из бэкенда в формате json!? */
    function getErrorData(error) {
        const parser = new DOMParser();
        let doc = parser.parseFromString(error, "text/html"); // error.reponse.data  for axios
        let header = doc.getElementById('summary').children[0].textContent;
        let message = doc.getElementById('summary').children[1].textContent;
        return header + "<br>" + message;
    }

    const send = async function (data){
        let notificationdata = {...notification_success_data};
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
                dispatch(eventname, {'response': await response.json()})
                return true;
            }
            else {
                throw(new Error(`${response.status} - ${response.statusText}<br>${getErrorData(await response.text())}`));
            }
        }
        catch (error) {
            notificationdata = {...notification_error_data};
            notificationdata.text += error.message;
            return false;
        }
        finally {
            dispatch(notificationeventname, notificationdata);
        }
    }
</script>

<form on:submit|preventDefault={additem} name="itemform">
    <div class="field is-horizontal">
        <div class="field-label is-normal">
            <label class="label" for="{props.name}">{props.label}</label>
        </div>
        <div class="field-body">
            <div class="field has-addons">
                <div class="control is-expanded">
                    <input 
                        class="input" class:is-danger={props.danger}
                        type="text" 
                        name={props.name}
                        maxlength={props.maxlength}
                        required
                        id={props.id} 
                        placeholder={props.placeholder}
                        value={props.value}
                        on:input={()=>props.danger=false}>
                </div>
                <div class="control">
                    <button class="button is-success" type="submit" title='ввод'>
                        <span class="icon">
                            <i class="fa-light fa-arrow-right"></i>
                        </span>
                    </button>
                </div>
            </div>
        </div>
    </div>
</form>
