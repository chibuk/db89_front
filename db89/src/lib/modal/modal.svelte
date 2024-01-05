<script>
    /**
     * Shows modal, returns true or false.
     */
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
    let eventname = 'modalevent';
    
    export let htmldata = "Sample modal data";
    export let active = false;     // показать или скрыть модальное окно
    let modal_return = false;

    const handleSubmit = () => {
        modal_return = true;
        active = false;
        dispatch(eventname, modal_return);
    }

    const handleReset = () => {
        modal_return = false;
        active = false;
        dispatch(eventname, modal_return);
    }
</script>
<div class="modal" class:is-active={active}>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div class="modal-background" on:click={handleReset}></div>
    <div class="modal-content">
        <form class="message is-warning" 
            on:submit|preventDefault={handleSubmit} 
            on:reset|preventDefault={handleReset}>
            <div class="message-body">
                <div class="container content">
                   {@html htmldata}
                </div>
                <div class="field is-grouped is-grouped-centered">
                    <p class="control">
                        <button type="submit" class="button is-success" title="Подтвердить">
                            <span class="icon is-small"><i class="fa-light fa-check"></i></span><span>Да</span>
                        </button>
                    </p>
                    <p class="control">
                        <button type="reset" class="button is-danger is-outlined" title="Отменить">
                            <span class="icon is-small"><i class="fa-light fa-xmark"></i></span><span>Нет</span>
                        </button>
                    </p>
                </div>
            </div>
        </form>
    </div>
</div>

<style>
    button {
        width: 10rem;
    }
</style>