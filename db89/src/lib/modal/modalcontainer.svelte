<script>
    export let active = false;
    export let modal_style = ''; // '' | justify-content: flex-start;
    export let content_style = ''; // '' | top: 1rem;
</script>

<div class="modal" class:is-active={active} style={modal_style}>
    <div class="modal-background"></div>
    <div class="modal-content" style={content_style}>
        <slot />
    </div>
    <button class="modal-close is-large" aria-label="close" on:click={()=>active=false}></button>
</div>