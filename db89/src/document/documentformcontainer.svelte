<script>
    export let display = "none";
</script>

<div class="columns">
    <div class="column" style:display>    
        <div class="box" style="border-right: 1px solid #abc">
            <button class="delete is-pulled-right" aria-label="close" on:click={()=>display='none'}></button>
            <slot name="form" />
        </div>
    </div>
    <div class="column">
        <slot name="table" />
    </div>
</div>
