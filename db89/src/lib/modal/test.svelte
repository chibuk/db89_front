<script>
// @ts-nocheck

    import AutoComplete from "simple-svelte-autocomplete";
    const colors = ["White", "Red", "Yellow", "Green", "Blue", "Black", "Mät bläck", "<i>Jét Black</i>"];
    let selectedColor;
  </script>
  
  <AutoComplete items={colors} bind:selectedItem={selectedColor} /> Selected: {selectedColor}
  