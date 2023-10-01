<script lang="ts">
  export let incompleteToDos: number;
</script>

<span class="todo-count">
  {incompleteToDos}
  {incompleteToDos === 1 ? "item" : "items"} left
</span>
