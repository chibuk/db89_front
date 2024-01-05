<script>
    import { createEventDispatcher } from 'svelte';

    let input_danger = false;
    let input_value = '';

    // @ts-ignore
    const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value; // do global
    // @ts-ignore
    const url = "https://" + document.domain + app.dataset.api;
    // const url = "http://127.0.0.1:8000/api/v1/items/";      //dev
    const dispatch = createEventDispatcher();
    const eventname = 'updatetable';
    const notificationeventname = 'notification';
    const notification_success_data = {
                add_class: "is-success is-light",
                text: '<i class="fa-light fa-check"></i>'
    };
    const notification_error_data = {
                add_class: "is-danger",
                text: 'Не удалось сохранить: <br>'
    };

    const additem = async () => {
        if (await send({name: input_value})) input_value = '';   // else ...
        else input_danger = true;
    }

    /* Стоит ли оставлять такую функцию или переделать BackEnd?
    Ошибку бэкенд возвращает в виде HTML документа. Данная функция возвращает заголовок.
    TODO: Передать ошибку из бэкенда в формате json!? */
    function getErrorData(error) {
        const parser = new DOMParser();
        let doc = parser.parseFromString(error, "text/html");
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
<div class="mt-1 ml-2" id='svelte-itemform-contaner'>
    <div class="field is-horizontal">
        <div class="field-label is-normal">
            <label class="label" for="nmae_item">Создать:</label>
        </div>
        <div class="field-body">
            <div class="field has-addons">
                <div class="control">
                    <input class="input" class:is-danger={input_danger} type="text" name='name_item' maxlength=256 
                        placeholder='Наименование' bind:value={input_value} on:input={()=>input_danger=false} />
                </div>
                <div class="control">
                    <button class="button is-link" type="submit" title='ввод' on:click={additem}>
                        <span class="icon">
                            <i class="fa-light fa-arrow-right"></i>
                        </span>
                    </button>
                </div>
            </div>
        </div>
    </div>
</div>
<style>
    div#svelte-itemform-contaner {
        display: inline-block;
    }
</style>
