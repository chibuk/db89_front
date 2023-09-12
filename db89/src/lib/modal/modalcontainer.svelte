<script>
    export let active = false;
    export let width = '640px';
</script>

<div class="modal" class:is-active={active}>
    <div class="modal-background"></div>
    <div class="modal-content" style:width>
        <slot />
    </div>
    <button class="modal-close is-large" aria-label="close" on:click={()=>active=false}></button>
</div>