<script>
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
    const url = "https://" + document.domain + app.dataset.api;
    // const url = "http://" + document.domain + ":8000" + app.dataset.api; //dev

    import Tablegen from "../lib/tablegen.svelte";  // Content
    let tablegen_instance;
    const href = "/organization/"   // <a href=""> in Tablegen rows
    const tablegen_id = "tablegen_id"

     /* sorting data by "id" */
    //  function srt (a, b) {
    //     if (a.id < b.id) return 1;
    //     else if (a.id > b.id) return -1;
    //     else return 0;
    // }

    const loadTable = async () => {  
        tablegen_instance?.$destroy();
        tablegen_instance = new Tablegen({
            target: document.getElementById(tablegen_id),
            props: {
                url: url,
                fields: ['id', 'name', 'inn', 'address', 'email', 'phone', 'bank'],
                href: href,
            }
        })
    }

    const notificationHandler = (event) => {
        notificationdata.text = event.detail.text;
        notificationdata.add_class = event.detail.add_class;
        notificationdata.dispaly = "block";
        setTimeout(() => {notificationdata.dispaly='none';}, 10000);
    }

    function getEdit_id () {
        let _form = document.getElementById('tablegen_id').parentNode;
        // @ts-ignore
        const checkboxes = new FormData(_form);  // чекбоксы [['id','1'], ['id','2']
        return checkboxes.get('id') // первый из отмеченных идет на редактирование
    }

    function closehandler() {
        activate_modal=false;
    }

    let orgFormInstance = undefined;    // чтобы потом удалять инстансы предыдущих форм
    const activateOrganizationForm = (_id='') => {
        orgFormInstance?.$destroy();
        orgFormInstance = new Organizationform({
            target: document.querySelector("#organization_form_container"),
            props: {id: _id},
        })
        orgFormInstance.$on('updatetable', loadTable);
        orgFormInstance.$on('notification', notificationHandler);
        orgFormInstance.$on('close', closehandler);
        activate_modal=true;
    }

    onMount(loadTable);
    document.querySelector("#navbar-item-organizations").classList.add("is-active");
</script>

<Modalcontainer bind:active={activate_modal}>
    <div id="organization_form_container"></div>
</Modalcontainer>

<h4 class="title is-4 pl-4 pt-5">Список организаций</h4>

<form>
    <div id={tablegen_id}></div>
    <Delete on:notification={notificationHandler} 
            on:updatetable={loadTable} 
            bind:disabled={button_disabled} uriPath="organizations" />
    
    <button class="button is-link is-outlined mt-1" on:click|preventDefault={()=>activateOrganizationForm(getEdit_id())} 
        title='Редактировать' disabled={!button_disabled}>
        <i class="fa-light fa-pen"></i>
    </button>
    <button class="button is-link is-outlined mt-1" on:click|preventDefault={()=>activateOrganizationForm()} title='Добавить'>
        <i class="fa-light fa-plus"></i>
    </button>
</form>

<Notification bind:display={notificationdata.dispaly} 
              bind:text={notificationdata.text} 
              bind:add_class={notificationdata.add_class} />

<style>
    form {
        height: calc(100% - 5rem);
    }
</style>
