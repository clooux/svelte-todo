<script lang="ts">
  export let completedToDos: number;
  export let clearCompleted: () => void;
</script>

<button
  on:click={clearCompleted}
  class:hidden={completedToDos === 0}
  class="clear-completed">Clear completed</button
>
