<script>
  export let text = ''
  export let row = ''
  export let col = ''
</script>

<style>
  label {
    font-weight: 800;
    text-align: center;
    font-size: 20;
    padding-bottom: 20;
  }
</style>

<label {row} {col} {text}>
  <slot />
</label>
