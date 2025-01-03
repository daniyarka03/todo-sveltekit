
<script>
  import "../app.css";
  import { onMount } from 'svelte';
  import { fade, slide } from 'svelte/transition';
  import Calendar from '$lib/components/Calendar.svelte';
  import { theme } from '$lib/stores/theme';
  import MeshGradient from '$lib/components/MeshGradient.svelte';
  let todos = [];
  let newTodoText = '';
  let filter = 'all';
  let view = 'list';
  let editingTodo = null;
  const STORAGE_KEY = 'todo-app-data';
  let newTodoDueDate = '';
  let newTodoPriority = 4;
  let newTodoCategory = '';
  let categories = ['Personal', 'Work', 'Shopping', 'Health'];
  const themes = [
    { id: 'light', name: 'Light' },
    { id: 'dark', name: 'Dark' },
    { id: 'sepia', name: 'Sepia' },
    { id: 'morning', name: 'Morning Blue' }
  ];
  onMount(loadFromStorage);
  onMount(() => {
    const savedTodos = localStorage.getItem('todos');
    if (savedTodos) {
      todos = JSON.parse(savedTodos);
    }
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme) {
      theme.set(savedTheme);
    }
    
    applyTheme($theme);
  });
  theme.subscribe(value => {
    if (typeof window !== 'undefined') {
      localStorage.setItem('theme', value);
      applyTheme(value);
    }
  });
  function saveToStorage() {
    const data = {
      todos,
      theme: $theme,
      categories,
      view,
      filter
    };
    localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
  }
  
  function loadFromStorage() {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (saved) {
      const data = JSON.parse(saved);
      todos = data.todos;
      theme.set(data.theme);
      categories = data.categories;
      view = data.view;
      filter = data.filter;
    }
  }
  function applyTheme(selectedTheme) {
    document.documentElement.classList.remove('dark', 'sepia', 'morning');
    if (selectedTheme !== 'light') {
      document.documentElement.classList.add(selectedTheme);
    }
  }
  $: {
    if (typeof window !== 'undefined') {
      localStorage.setItem('todos', JSON.stringify(todos));
    }
  }
  
  function getPriorityColor(priority) {
    switch (priority) {
      case 1: return '#ef4444'; // red
      case 2: return '#f97316'; // orange
      case 3: return '#3b82f6'; // blue
      default: return '#9ca3af'; // gray
    }
  }
  
  function addTodo() {
    if (newTodoText.trim()) {
      todos = [...todos, {
        id: Date.now(),
        text: newTodoText.trim(),
        completed: false,
        createdAt: new Date(),
        dueDate: newTodoDueDate,
        priority: newTodoPriority,
        category: newTodoCategory
      }];
      resetForm();
    }
  }
  
  function resetForm() {
    newTodoText = '';
    newTodoDueDate = '';
    newTodoPriority = 4;
    newTodoCategory = '';
  }
  
  function startEdit(todo) {
    editingTodo = { ...todo };
  }
  
  function saveEdit() {
    if (editingTodo) {
      todos = todos.map(todo => 
        todo.id === editingTodo.id ? editingTodo : todo
      );
      editingTodo = null;
    }
  }
  $: {
    if (typeof window !== 'undefined') {
      saveToStorage();
    }
  }
  function toggleTodo(id) {
    todos = todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
  }
  
  function deleteTodo(id) {
    todos = todos.filter(todo => todo.id !== id);
  }
  
  function clearCompleted() {
    todos = todos.filter(todo => !todo.completed);
  }
  
  $: filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  }).sort((a, b) => a.priority - b.priority);
  
  $: remainingTodos = todos.filter(todo => !todo.completed).length;
</script>



<div class="min-h-screen transition-colors duration-200
  {$theme === 'dark' ? 'bg-gray-900 text-white' : ''}
  {$theme === 'sepia' ? 'bg-amber-100 text-amber-900' : ''}
  {$theme === 'morning' ? 'bg-blue-100 text-blue-900' : ''}
">
<MeshGradient />
<div class="max-w-4xl mx-auto p-6">
  <div class="flex justify-between items-center mb-6">
    <h1 class="text-3xl font-bold">Todo List</h1>
    <div class="flex gap-4">
        <select
          bind:value={$theme}
          class="px-2 sm:px-3 py-1 rounded border text-sm sm:text-base
            bg-white/80 backdrop-blur-sm
            dark:bg-gray-800/80 dark:border-gray-700 dark:text-white
            sepia:bg-amber-50/80 sepia:border-amber-200
            morning:bg-blue-50/80 morning:border-blue-200"
        >
          {#each themes as t}
            <option value={t.id}>{t.name}</option>
          {/each}
        </select>
        <div class="flex gap-2">
          <button
            class="px-2 sm:px-3 py-1 rounded text-sm sm:text-base {view === 'list' ? 
              'bg-blue-500 text-white' : 
              'bg-white/80 backdrop-blur-sm dark:bg-gray-800/80 sepia:bg-amber-50/80 morning:bg-blue-50/80'}"
            on:click={() => view = 'list'}
          >
            List
          </button>
          <button
            class="px-2 sm:px-3 py-1 rounded text-sm sm:text-base {view === 'calendar' ? 
              'bg-blue-500 text-white' : 
              'bg-white/80 backdrop-blur-sm dark:bg-gray-800/80 sepia:bg-amber-50/80 morning:bg-blue-50/80'}"
            on:click={() => view = 'calendar'}
          >
            Calendar
          </button>
        </div>
      </div>
    </div>
    
    <!-- Add new todo -->
    <form on:submit|preventDefault={addTodo} class="mb-6">
      <div class="flex flex-col sm:flex-row gap-3">
        <input
          type="text"
          bind:value={newTodoText}
          placeholder="What needs to be done?"
          class="flex-1 px-4 py-2 rounded border bg-white/80 backdrop-blur-sm
            dark:bg-gray-800/80 dark:border-gray-700 dark:text-white
            sepia:bg-amber-50/80 sepia:border-amber-200
            morning:bg-blue-50/80 morning:border-blue-200"
        />
        <div class="flex flex-wrap gap-2">
          <input
            type="date"
            bind:value={newTodoDueDate}
            class="w-full sm:w-auto px-4 py-2 rounded border bg-white/80 backdrop-blur-sm
              dark:bg-gray-800/80 dark:border-gray-700 dark:text-white
              sepia:bg-amber-50/80 sepia:border-amber-200
              morning:bg-blue-50/80 morning:border-blue-200"
          />
          <select
            bind:value={newTodoPriority}
            class="w-full sm:w-auto px-4 py-2 rounded border bg-white/80 backdrop-blur-sm
              dark:bg-gray-800/80 dark:border-gray-700 dark:text-white
              sepia:bg-amber-50/80 sepia:border-amber-200
              morning:bg-blue-50/80 morning:border-blue-200"
          >
            <option value={1}>P1</option>
            <option value={2}>P2</option>
            <option value={3}>P3</option>
            <option value={4}>P4</option>
          </select>
          <select
            bind:value={newTodoCategory}
            class="w-full sm:w-auto px-4 py-2 rounded border bg-white/80 backdrop-blur-sm
              dark:bg-gray-800/80 dark:border-gray-700 dark:text-white
              sepia:bg-amber-50/80 sepia:border-amber-200
              morning:bg-blue-50/80 morning:border-blue-200"
          >
            <option value="">Category</option>
            {#each categories as category}
              <option value={category}>{category}</option>
            {/each}
          </select>
          <button
            type="submit"
            class="w-full sm:w-auto px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
          >
            Add
          </button>
        </div>
      </div>
    </form>

    {#if view === 'list'}
      <!-- Filters -->
      <div class="flex flex-wrap gap-2 mb-4">
        <button
          class="px-3 py-1 rounded text-sm {filter === 'all' ? 'bg-blue-500 text-white' : 'bg-white/80 backdrop-blur-sm'}"
          on:click={() => filter = 'all'}
        >
          All
        </button>
        <button
          class="px-3 py-1 rounded text-sm {filter === 'active' ? 'bg-blue-500 text-white' : 'bg-white/80 backdrop-blur-sm'}"
          on:click={() => filter = 'active'}
        >
          Active
        </button>
        <button
          class="px-3 py-1 rounded text-sm {filter === 'completed' ? 'bg-blue-500 text-white' : 'bg-white/80 backdrop-blur-sm'}"
          on:click={() => filter = 'completed'}
        >
          Completed
        </button>
      </div>

      <!-- Todo list -->
      <ul class="space-y-2 mb-4">
        {#each filteredTodos as todo (todo.id)}
          <li 
            class="flex flex-col sm:flex-row items-start sm:items-center gap-2 p-3 bg-white/80 backdrop-blur-sm rounded
              dark:bg-gray-800/80 sepia:bg-amber-50/80 morning:bg-blue-50/80"
            transition:slide|local
            animate:flip
          >
            {#if editingTodo?.id === todo.id}
              <div class="flex-1 flex flex-col sm:flex-row gap-2 w-full">
                <input
                  type="text"
                  bind:value={editingTodo.text}
                  class="flex-1 px-2 py-1 border rounded"
                />
                <div class="flex flex-wrap gap-2">
                  <input
                    type="date"
                    bind:value={editingTodo.dueDate}
                    class="w-full sm:w-auto px-2 py-1 border rounded"
                  />
                  <select
                    bind:value={editingTodo.priority}
                    class="w-full sm:w-auto px-2 py-1 border rounded"
                  >
                    <option value={1}>P1</option>
                    <option value={2}>P2</option>
                    <option value={3}>P3</option>
                    <option value={4}>P4</option>
                  </select>
                  <select
                    bind:value={editingTodo.category}
                    class="w-full sm:w-auto px-2 py-1 border rounded"
                  >
                    <option value="">Category</option>
                    {#each categories as category}
                      <option value={category}>{category}</option>
                    {/each}
                  </select>
                  <button
                    on:click={saveEdit}
                    class="w-full sm:w-auto px-3 py-1 bg-green-500 text-white rounded hover:bg-green-600"
                  >
                    Save
                  </button>
                </div>
              </div>
            {:else}
              <input
                type="checkbox"
                checked={todo.completed}
                on:change={() => toggleTodo(todo.id)}
                class="mt-1 sm:mt-0 w-5 h-5"
              />
              <div class="flex-1">
                <div class={todo.completed ? 'line-through text-gray-500' : ''}>
                  {todo.text}
                </div>
                <div class="text-sm text-gray-500">
                  {#if todo.dueDate}
                    Due: {new Date(todo.dueDate).toLocaleDateString()} |
                  {/if}
                  {#if todo.category}
                    {todo.category} |
                  {/if}
                  Priority: P{todo.priority}
                </div>
              </div>
              <div class="flex gap-2 w-full sm:w-auto">
                <button
                  on:click={() => startEdit(todo)}
                  class="flex-1 sm:flex-none px-2 py-1 text-blue-500 hover:text-blue-600"
                >
                  Edit
                </button>
                <button
                  on:click={() => deleteTodo(todo.id)}
                  class="flex-1 sm:flex-none px-2 py-1 text-red-500 hover:text-red-600"
                >
                  Delete
                </button>
              </div>
            {/if}
          </li>
        {/each}
      </ul>

      <!-- Footer -->
      <div class="flex justify-between items-center text-sm text-gray-600 dark:text-gray-400">
        <span>{remainingTodos} item{remainingTodos === 1 ? '' : 's'} left</span>
        <button
          on:click={clearCompleted}
          class="text-blue-500 hover:text-blue-600"
        >
          Clear completed
        </button>
      </div>
    {:else}
      <Calendar {todos} />
    {/if}
  </div>
</div>