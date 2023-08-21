<script>
    // import Organizationform from "./organizationform.svelte";   // POST Organization form
    import {onMount} from "svelte";

    import Delete from "../lib/delete.svelte";
    let button_disabled = true;

    const url = "http://127.0.0.1:8000/api/v1/documents/";

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
    let tablegen_instance;
    let tablegen_data = [];
    const href = "/document/"   // <a href=""> in Tablegen rows
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
                // @ts-ignore
                table_content: _parse(tablegen_data).sort(srt),
                href: href,
                checkbox_name: 'id',
            }
        });
        _parse(tablegen_data);
    }

    onMount(loadTable);
</script>
<h4 class="title is-4 pl-4 pt-5">Список документов.</h4>
<form>
    <div id={tablegen_id}></div>
    <Delete on:notification on:deleteitem={loadTable} bind:disabled={button_disabled} />
</form>
