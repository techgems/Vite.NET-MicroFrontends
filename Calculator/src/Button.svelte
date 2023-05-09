<script>
  export let onClick = () => {};
  export let spanRow = false;
  export let type = "number";
</script>

<style>
  .spanRow {
    grid-column: 1 / -1;
  }

  .number {
    background-color: #000000db;
  }

  .operator {
    background-color: #808080;
  }

  .action {
    background-color: #e66108;
  }

  h2 {
    color: #fff;
  }
</style>

<button class={type} class:spanRow on:click={onClick}>
  <h2>
    <slot>¯\_(ツ)_/¯</slot>
  </h2>
</button>
