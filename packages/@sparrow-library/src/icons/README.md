<script lang="ts">
  export let size = "20px";
  export let color = "";
  $: finalColor = color === "" ? "inherit" : color;
</script>

<i
  class="icon-ic_fluent_attach_20_regular"
  style="font-size:{size}; color:{finalColor}; display: flex; justify-content:center; align-items:center;"
></i>

File name should be refered to the class name of the icon
For example: AttachRegular.svelte
