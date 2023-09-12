<script>
    // import Organizationform from "./organizationform.svelte";   // POST Organization form
    import {onMount} from "svelte";

    import Notification from "../lib/notification.svelte";
    const notificationdata = {
        text: "",
        dispaly: "none",
        add_class: "",
    }

    import Delete from "../lib/delete.svelte";
    let button_disabled = true;

    import Organizationform from "./organizationform.svelte";
    import Modalcontainer from "../lib/modal/modalcontainer.svelte";
    let activate_modal = false;

    // @ts-ignore
    const url = "http://127.0.0.1:8000/api/v1/organizations/";
    // const url = "https://" + document.domain + app.dataset.api;

    import Tablegen from "../lib/tablegen.svelte";  // Content
    let tablegen_instance;
    let tablegen_data = [];
    const href = "/organization/"   // <a href=""> in Tablegen rows
    const tablegen_id = "tablegen_" + Math.round(Math.random()*10000);

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

    const notificationHandler = (event) => {
        notificationdata.text = event.detail.text;
        notificationdata.add_class = event.detail.add_class;
        notificationdata.dispaly = "block";
        setTimeout(() => {notificationdata.dispaly='none';}, 10000);
    }

    onMount(loadTable);
    document.querySelector("#navbar-item-organizations").classList.add("is-active");
</script>

<Modalcontainer bind:active={activate_modal}>
    <Organizationform on:updatetable={loadTable} on:notification={notificationHandler} />
</Modalcontainer>

<h4 class="title is-4 pl-4 pt-5">Список организаций</h4>

<form>
    <div id={tablegen_id}></div>
    <Delete on:notification={notificationHandler} 
            on:updatetable={loadTable} 
            bind:disabled={button_disabled} />
    <button class="button is-link is-outlined mt-1" on:click|preventDefault={()=>activate_modal=true}>
        <i class="fa-light fa-plus"></i>
    </button>
</form>

<Notification bind:display={notificationdata.dispaly} 
              bind:text={notificationdata.text} 
              bind:add_class={notificationdata.add_class} />
