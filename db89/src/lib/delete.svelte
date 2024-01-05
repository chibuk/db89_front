<script>
    import { onDestroy } from 'svelte';
    import { delete_button_enabled } from '../item/stores';
    export let uriPath = '';

    // @ts-ignore
    const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value; // do global
    // @ts-ignore
    // const url = `http://127.0.0.1:8000/api/v1/${uriPath}/`;      // dev
    const url = "https://" + document.domain + app.dataset.api;

    import Modal from "./modal/modal.svelte";
    let modal_show = false;
    let modal_data = "<p>Удаление <span class='has-text-danger-dark'>необратимо</span>! Удалить?</p>";

    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();

    export let disabled = true;    // disabled button
    let _subscription = delete_button_enabled.subscribe((value)=>disabled=value);  
    onDestroy(_subscription);
    
    const notificationeventname = 'notification';
    const notification_error_data = {
                add_class: "is-danger",
                text: 'Не удалось удалить:'
    };

    let event_currentTarget = null; // т.к. Event сменится на другой, сохраним ссылку на тег
    const modalDeleteShow = (event) => {
        event_currentTarget = event.currentTarget;
        modal_show = true;    
    }

    async function deleteItems (event) {
        if (!event.detail) return;
        const _form = event_currentTarget.parentElement;
        let _errordelete = '';
        const checkboxes = new FormData(_form); // чекбоксы [['id','1'], ['id','2']]
        for (let id of checkboxes.values()) {
            if (id) _errordelete += await deleteItem(id);
        };
        dispatch('updatetable'); // loadTable();
        if (_errordelete) {
            let notificationdata = {...notification_error_data};
            notificationdata.text += _errordelete;
            dispatch(notificationeventname, notificationdata);
        }
        delete_button_enabled.update((n)=>false); // disable delete button
    }

    /* Удаляем элемент, возвращает ''(для false), если успешно или текст http ошибки ответа */
    const deleteItem = async (item) => {
        try {
            const response = await fetch(`${url}${item}/`, {
                method: 'delete',
                headers: {
                    'X-CSRFToken': csrftoken,
                }
            })
            if (response.ok) return '';
            else return `<br> id: ${item}, ${response.status} - ${response.statusText}`;
        }
        catch (er) {
            return `<br> id: ${item}, ${er.message}`;
        }
    }
</script>

<button type="submit" class="button is-link is-outlined mt-1 ml-2" 
    on:click|preventDefault={modalDeleteShow} 
    disabled={!disabled}
    title="Удалить отмеченные">
    <i class="fa-light fa-trash-xmark"></i>
    <div id="tgen_change_inform"></div>
</button>
<Modal htmldata={modal_data} bind:active={modal_show} on:modalevent={deleteItems} />
