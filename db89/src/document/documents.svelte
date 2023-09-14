<script>
    // import Organizationform from "./organizationform.svelte";   // POST Organization form
    import {onMount} from "svelte";

    import Delete from "../lib/delete.svelte";
    let button_disabled = true;

    import Modalcontainer from "../lib/modal/modalcontainer.svelte";
    let activate_modal = false;

    // import Documentformcontainer from "./documentformcontainer.svelte";
    // let display_form = 'none';

    import Docitemform from "./docitemform.svelte";

    // @ts-ignore
    const url = "https://" + document.domain + app.dataset.api;
    // const url = "http://127.0.0.1:8000/api/v1/documents/";

    function _parse(data) {
        let pre = JSON.parse(JSON.stringify(data, ['id', 'number', 'data', 'city', 'truck', 'destination_address',]));
        for (let i=0; i<pre.length; i++) {
            pre[i].sender = data[i].sender.name;
            pre[i].receiver = data[i].receiver.name;
            pre[i].payer = data[i].payer.name;
        } 
        return pre;
    }

    import Tablegen from "../lib/tablegen.svelte";  // Content
    import Documentform from "./documentform.svelte";
    let tablegen_instance;
    const href = "/document/"   // <a href=""> in Tablegen rows
    const tablegen_id = "tablegen_" + Math.round(Math.random()*10000);

     /* sorting data by "id" */
     function srt (a, b) {
        if (a.id < b.id) return 1;
        else if (a.id > b.id) return -1;
        else return 0;
    }

    const loadTable = async () => {
        const tablegen_data = await fetch(url).then((x) => x.json());
        tablegen_instance?.$destroy();
        tablegen_instance = new Tablegen({
            target: document.getElementById(tablegen_id),
            props: {
                // @ts-ignore
                table_content: _parse(tablegen_data).sort(srt),
                href: href,
                checkbox_name: 'id',
            }
        });
        _parse(tablegen_data);
    }

    onMount(loadTable);
    document.querySelector("#navbar-item-documents").classList.add("is-active");
</script>

<Modalcontainer bind:active={activate_modal}
                modal_style='justify-content: flex-start'
                content_style='top: 3rem; max-width: 70rem; min-width: 25rem; width: 100%; max-height: calc(100% - 3rem)'>
    <!-- <butoon class="delete is-pulled-right mt-1 mr-1" on:click={()=>activate_modal=false}></butoon> -->
    <Documentform><Docitemform /></Documentform>
</Modalcontainer>
<h4 class="title is-4 pl-4 pt-5">Список документов</h4>
<form>
    <div id={tablegen_id}></div>
    <Delete on:notification on:updatetable={loadTable} bind:disabled={button_disabled} />
    <button class="button is-link is-outlined mt-1" on:click|preventDefault={()=>activate_modal=(activate_modal==true)?false:true}>
        <i class="fa-light fa-plus"></i>
    </button>
</form>
