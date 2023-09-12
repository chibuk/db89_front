<script>
    /**
     * Items page app. Provide show content, add, delete & edit.
     */
    import {onMount} from "svelte";
    import Itemform from "./itemform.svelte";   // POST Item form
    
    import Delete from "../lib/delete.svelte";
    let button_disabled = true;
    
    import Tablegen from "../lib/tablegen.svelte";  // Content
    const href = "/item/";   // <a href=""> in Tablegen rows
    let tablegen_instance;
    let tablegen_data = [];
    const tablegen_id = "tablegen_" + Math.round(Math.random()*10000);

    import Notification from "../lib/notification.svelte";
    let notification_display = 'none';
    let notificationadd_class = 'is-warning';
    let notification_text = 'Test text';

    // @ts-ignore
    const url = "https://" + document.domain + app.dataset.api;

     /* sorting data by "id" */
     function srt (a, b) {
        if (a.id < b.id) return 1;
        else if (a.id > b.id) return -1;
        else return 0;
    }

    const loadTable = async () => {
        tablegen_data = await fetch(url).then((x) => x.json());    
        tablegen_instance?.$destroy();
        tablegen_instance = new Tablegen({
            target: document.getElementById(tablegen_id),
            props: {
                table_content: tablegen_data.sort(srt),
                href: href,
                checkbox_name: 'id',
            }
        })
    }

    onMount(loadTable);

    const notificationHandler = (event) => {
        notification_text = event.detail.text;
        notificationadd_class = event.detail.add_class;
        notification_display = "block";
            /* Значение задержки в зависимости от add_class */
        const delay = () => {
            let _delay;
            (notificationadd_class.includes("is-danger") || notificationadd_class.includes("is-warning")) ? _delay = 10000 : _delay = 2000;
            return _delay;
        }
        setTimeout(() => {notification_display='none';}, delay());
    }

    document.querySelector("#navbar-item-items").classList.add("is-active");
</script>
<h4 class="title is-4 pl-4 pt-5">Список наименований груза</h4>
<form>
    <div id={tablegen_id}></div>
    <Delete on:notification={notificationHandler} 
            on:updatetable={loadTable} 
            bind:disabled={button_disabled} />
</form>
<div class="box">
    <Itemform on:updatetable={loadTable} 
              on:notification={notificationHandler} />
</div>
<Notification bind:text={notification_text} 
              bind:add_class={notificationadd_class} 
              bind:display={notification_display} />
