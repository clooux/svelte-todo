<script lang="ts">
  import { tick } from "svelte";
  import type { IToDo, FiltersType } from "src/types/todo";
  import { useStorage } from "$root/stores/useStorage";

  import AddToDo from "./AddToDo.svelte";
  import ToDo from "./ToDo.svelte";
  import ToDosLeft from "./ToDosLeft.svelte";
  import FilteredToDos from "./FilteredToDos.svelte";
  import ClearToDos from "./ClearToDos.svelte";
  import type { Writable } from "svelte/store";

  const todos: Writable<IToDo[]> = useStorage<IToDo[]>("todos", []);

  let selectedFilter: FiltersType = "all";
  let filtering = false;

  $: toDosAmount = $todos.length;
  $: incompleteToDos = $todos.filter((todo) => !todo.completed).length;
  $: filteredToDos = filterTodos($todos, selectedFilter);
  $: completedToDos = $todos?.filter((todo) => todo.completed).length;
  $: duration = filtering ? 0 : 250;

  function generateRandomId(): string {
    return Math.random().toString(16).slice(2);
  }

  function addToDo(todo: string): void {
    const newToDo: IToDo = {
      id: generateRandomId(),
      text: todo,
      completed: false,
    };
    $todos = [...$todos, newToDo];
  }

  function toggleCompleted(event: MouseEvent): void {
    const { checked } = event.target as HTMLInputElement;
    $todos = $todos.map((todo) => ({ ...todo, completed: checked }));
  }

  function completeToDo(id: string): void {
    $todos = $todos.map((todo) => {
      if (todo.id === id) {
        todo.completed = !todo.completed;
      }
      return todo;
    });
  }

  function removeToDo(id: string): void {
    $todos = $todos.filter((todo) => todo.id !== id);
  }

  function editToDo(id: string, newTodo: string): void {
    const currentTodo = $todos.findIndex((todo) => todo.id === id);
    $todos[currentTodo].text = newTodo;
  }

  async function setFilter(newFilter: FiltersType): Promise<void> {
    filtering = true;
    await tick();
    selectedFilter = newFilter;
    await tick();
    filtering = false;
  }

  function filterTodos(todos: IToDo[], filter: FiltersType): IToDo[] {
    switch (filter) {
      case "all":
        return todos;
      case "active":
        return todos.filter((todo) => !todo.completed);
      case "completed":
        return todos.filter((todo) => todo.completed);
    }
  }

  function clearCompleted(): void {
    $todos = $todos.filter((todo) => todo.completed !== true);
  }
</script>

<main>
  <h1 class="title">ToDos</h1>

  <section class="todos">
    <AddToDo {addToDo} {toggleCompleted} {toDosAmount} />

    {#if toDosAmount}
      <ul class="todo-list">
        {#each filteredToDos as todo (todo.id)}
          <ToDo {todo} {completeToDo} {removeToDo} {editToDo} {duration} />
        {/each}
      </ul>
      <div class="actions">
        <ToDosLeft {incompleteToDos} />
        <FilteredToDos {selectedFilter} {setFilter} />
        <ClearToDos {clearCompleted} {completedToDos} />
      </div>
    {/if}
  </section>
</main>

<style>
  /* Todos */

  .title {
    font-size: var(--font-80);
    font-weight: inherit;
    text-align: center;
    color: var(--color-title);
  }

  .todos {
    --width: 500px;
    --todos-bg: hsl(0 0% 98%);
    --todos-text: hsl(220 20% 14%);

    width: var(--width);
    color: var(--todos-text);
    background-color: var(--todos-bg);
    border-radius: var(--radius-base);
    border: 1px solid var(--color-gray-90);
    box-shadow: 0 0 4px var(--shadow-1);
  }

  .todo-list {
    list-style: none;
  }

  .actions {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: var(--spacing-8) var(--spacing-16);
    font-size: 0.9rem;
    border-top: 1px solid var(--color-gray-90);
  }

  .actions:before {
    content: "";
    height: 40px;
    position: absolute;
    right: 0;
    bottom: 0;
    left: 0;
    box-shadow: 0 1px 1px hsla(0, 0%, 0%, 0.2), 0 8px 0 -3px hsl(0, 0%, 96%),
      0 9px 1px -3px hsla(0, 0%, 0%, 0.2), 0 16px 0 -6px hsl(0, 0%, 96%),
      0 17px 2px -6px hsla(0, 0%, 0%, 0.2);
    z-index: -1;
  }
</style>
