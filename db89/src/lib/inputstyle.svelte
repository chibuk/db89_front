<script>
    export let label = 'label';
    export let name = 'name';
    export let type = 'text';
    export let title = '';
    export let maxlength = 256;
</script>
<div class="input-out" title={title}>
    <div class="input-placeholder">{label}</div>
    <input type={type} class="input-min" name={name} maxlength={maxlength}>
</div>
<style>
    .input-out {
        margin: 1rem 0;
        display: block;
        padding: .1rem .6rem;
        border: 1px solid #999;
        border-radius: 5px;
        cursor: text;
    }
    /* :global(input.inputstyle-error) { border: 1px solid red } */
    .input-out:has(.input-min:focus) {
        box-shadow: 0 0 .2rem 0 #046ECC;
    }

    .input-placeholder {
        font: .9rem sans-serif;
        color:  #666;
        position: fixed;
    }
    .input-min {
        border: none;
        font: 1rem sans-serif;
        width: 100%;
        height: 2.2rem;
        padding: .7rem 0 0 0;
    }
    .input-min:focus {
        border: none;
        height: 2.2rem;
        outline: none;
    }
</style>