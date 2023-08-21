<script>
    /*
    Input text element with Bulma CSS styles. Horizontal flex layot.
    */
    export let label = "label";
    export let placeholder = "Ввд текста...";
    export let id = "id";
    export let name = "name";
    export let maxlength = 50;
    export let required = true;
    export let help = "help";
    export let input_class = "";            // + is-* Bulma class modifiers
    export let help_class = "";             // + is-* Bulma class modifiers
    // export let type = "text";               // password | email | tel
    export let value ="";
    export let readonly = false;
    export let danger = false;   // class:is-danger={danger}

</script>


<div class="field is-horizontal">
    <div class="field-label is-normal">
        <label class="label" for={id}>{label}</label>
    </div>
    <div class="field-body">
        <div class="field">
            <input class="input {input_class}" 
                class:is-danger={danger}
                type = 'text'
                name = {name} 
                maxlength = {maxlength} 
                required = {required} 
                id = {id} 
                placeholder = {placeholder}
                readonly = {readonly}
                value={value}>
            <p class="help {help_class}" class:is-danger={danger}>{help}</p>
        </div>
    </div>
</div>