<script>
    /**
     * TODO: Сделать подкючаемым компонентам, а не родителеи для всех.
     */
    // import Items from "../item/items.svelte";
    // import Organizations from "../organization/organizations.svelte";
    // import Documents from "../document/documents.svelte";

    /* Notifications */
    export let text = 'Test text';         // Notification content
    export let add_class = 'is-warning';   // [is-primary | is-info | is-success | is-warning | is-danger]
    export let display = "none";           // none | box
    const id = 'svelt-notyfycation';

    /* Значение прозрачности в зависимости от add_class */
    const opacity = () => {
        let _opacity;
        (add_class.includes("is-danger") || add_class.includes("is-warning")) ? _opacity = 0.5 : _opacity = 0.4;
        return _opacity;
    }

    let _opacity = opacity();

$:  if (display == "block") _opacity = opacity();
</script>

<div {id} class="notification {add_class}" style:display 
    style="opacity: {_opacity}; position: fixed;  bottom: 3rem; right: 3rem; font-size: 1rem;">
    
    {@html text}&nbsp; <button class="delete" on:click={()=>display='none'}></button>

</div>

<!-- <Items on:notification={handle}/> -->
<!-- <Organizations on:notification={handle}/> -->
<!-- <Documents on:notification={handle}/> -->
