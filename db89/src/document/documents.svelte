<script>
    import {onMount} from "svelte";
    import Delete from "../lib/delete.svelte";
    let editButton_enabled = false;
    import Modalcontainer from "../lib/modal/modalcontainer.svelte";
    let activate_modal = false;

    // @ts-ignore
    const url = "https://" + document.domain + app.dataset.api;
    // const url = "http://127.0.0.1:8000/api/v1/documents/";  //dev

    function _parse(data) {
        let pre = JSON.parse(JSON.stringify(data, [     // оставим нужные поля
            'id', 'number', 'city', 'truck', 'destination_address', 'doc_sum',
        ]));
        for (let i=0; i<pre.length; i++) {
            pre[i].sender = (data[i].sender) ? data[i].sender.name : '';
            pre[i].receiver = (data[i].receiver) ? data[i].receiver.name : '';
            // pre[i].payer = (data[i].payer) ? data[i].payer.name : '';
            pre[i].date = new Date(data[i].date).toLocaleDateString("ru");
            // pre[i].created =  '<i class="fa-light fa-calendar-circle-plus" title="' +
            //                 new Date(data[i].created).toLocaleString("ru") + '"></i>';
            // pre[i].modified = "<i class='fa-light fa-calendar-pen' title='" + 
            //                 new Date(data[i].modified).toLocaleString("ru") + "'></i>";
            // pre[i].document = pre[i].number + " от " + pre[i].date;
            pre[i].number = '<span title="Изменено: ' + new Date(data[i].modified).toLocaleString("ru") + '">' +
                pre[i].number + '</span>';
        }
        return JSON.parse(JSON.stringify(pre, [     // Изменим порядок полей
            'id', 'number', "date", 'doc_sum', 'sender', 'receiver', 'city', 'truck',
            'destination_address']));
    }

    import Tablegen from "../lib/tablegen.svelte";  // Content
    import Documentform from "./documentform.svelte";
    let tablegen_instance = undefined;
    const href = "/document/";   // <a href=""> in Tablegen rows
    const tablegen_id = "tablegen_id";

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
                href: href,
                checkbox_name: 'id',
                url: url,
                parseData: _parse, // fields не нужен, т.к. всё в этой функции есть
                // fields: ['id', 'number', 'city', 'truck', 'destination_address', 'doc_sum', 'sender.name', 'receiver.name'],
            }
        });
        updatetable = false;    //updatetable tag
    }

    function closeModalHandler () {
        activate_modal = false;
    }
    let documentFormInstanсe = undefined;
    const activateDocumentForm = (_id=-1, copy=false) => {
        documentFormInstanсe?.$destroy();   // уже разрушен, но на всякий случай
        documentFormInstanсe = new Documentform ({
            target: document.getElementById('document-form-container'),
            props: { editDocument_id: _id, copy: copy, },
        })
        // _documentFormInstanсe.$on('updatetable', loadTable);        // документ сохранён, обновить таблицу
        documentFormInstanсe.$on('closemodal', closeModalHandler); // событие из компонента, что он завершается
        documentFormInstanсe.$on('updatetable', ()=>updatetable=true);
        activate_modal=true;
    }

    onMount(loadTable);
    document.querySelector("#navbar-item-documents").classList.add("is-active");
    
    let updatetable = false;
    
    $:  if (!activate_modal) {
        documentFormInstanсe?.$destroy();
        if (updatetable) loadTable();
        // editButton_enabled = false;
    }

    function getEditDocument_id () {
        if (!editButton_enabled) return -1;     // dev
        let _form = document.getElementById(tablegen_id).parentNode;
        // @ts-ignore
        const checkboxes = new FormData(_form); // чекбоксы [['id','1'], ['id','2']
        return checkboxes.get('id');          // первый из отмеченных документ идет на редактирование
    }
</script>

<Modalcontainer bind:active={activate_modal}
                modal_style='justify-content: flex-start'
                content_style='top: 3rem; max-width: 70rem; min-width: 25rem; width: 100%; max-height: calc(100% - 3rem)'>
    <div id="document-form-container"></div>
</Modalcontainer>
<h4 class="title is-4 pl-4 pt-5">Список документов</h4>
<form>
    <div id={tablegen_id}></div>
    <Delete on:notification on:updatetable={loadTable} bind:disabled={editButton_enabled} uriPath="documents" />
    <button class="button is-link is-outlined mt-1" on:click|preventDefault={()=>{
        activateDocumentForm(-1);    // создать новый документ
        }} title='Новый документ'>
        <i class="fa-light fa-plus"></i>
    </button>
    <button class="button is-link is-outlined mt-1" 
        on:click|preventDefault={()=>{
            activateDocumentForm(getEditDocument_id());  // редактировать документ
            }} 
        disabled={!editButton_enabled}
        title='Редактировать'>
        <i class="fa-light fa-pen"></i>
    </button>
    <button class="button is-link is-outlined mt-1" 
        on:click|preventDefault={()=>{
            activateDocumentForm(getEditDocument_id(), true);  // копировать документ
            }} 
        disabled={!editButton_enabled}
        title='Копировать'>
        <i class="fa-light fa-copy"></i>
    </button>
</form>
<style>
    form {
        height: calc(100% - 5rem);
    }
</style>
