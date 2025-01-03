<script>
    import { fade, slide } from 'svelte/transition';
    import { format, startOfMonth, endOfMonth, eachDayOfInterval, isToday, isSameMonth, parseISO } from 'date-fns';
    import { theme } from '$lib/stores/theme';
    
    export let todos = [];
    let currentDate = new Date();
    
    $: monthStart = startOfMonth(currentDate);
    $: monthEnd = endOfMonth(currentDate);
    $: daysInMonth = eachDayOfInterval({ start: monthStart, end: monthEnd });
    
    $: todosByDate = daysInMonth.map(date => ({
      date,
      todos: todos.filter(todo => {
        if (!todo.dueDate) return false;
        const todoDate = typeof todo.dueDate === 'string' ? parseISO(todo.dueDate) : todo.dueDate;
        return format(todoDate, 'yyyy-MM-dd') === format(date, 'yyyy-MM-dd');
      })
    }));
    
    function previousMonth() {
      currentDate = new Date(currentDate.getFullYear(), currentDate.getMonth() - 1);
    }
    
    function nextMonth() {
      currentDate = new Date(currentDate.getFullYear(), currentDate.getMonth() + 1);
    }
  </script>
  
  <div class="bg-white dark:bg-gray-800 sepia:bg-amber-50 morning:bg-blue-50 rounded-lg shadow p-4">
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-xl font-semibold dark:text-white sepia:text-amber-900 morning:text-blue-900">
        {format(currentDate, 'MMMM yyyy')}
      </h2>
      <div class="flex gap-2">
        <button 
          on:click={previousMonth} 
          class="p-2 hover:bg-gray-100 dark:hover:bg-gray-700 sepia:hover:bg-amber-100 morning:hover:bg-blue-100 rounded"
        >
          ←
        </button>
        <button 
          on:click={nextMonth} 
          class="p-2 hover:bg-gray-100 dark:hover:bg-gray-700 sepia:hover:bg-amber-100 morning:hover:bg-blue-100 rounded"
        >
          →
        </button>
      </div>
    </div>
    
    <div class="grid grid-cols-7 gap-1">
      {#each ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'] as day}
        <div class="text-center text-sm font-medium p-2 dark:text-gray-300 sepia:text-amber-800 morning:text-blue-800">
          {day}
        </div>
      {/each}
      
      {#each todosByDate as { date, todos }}
        <div 
          class="min-h-[80px] p-1 border rounded 
          {isToday(date) ? 'bg-blue-50 dark:bg-blue-900/20 sepia:bg-amber-100 morning:bg-blue-100' : ''} 
          {isSameMonth(date, currentDate) ? 
            'bg-white dark:bg-gray-800 sepia:bg-amber-50 morning:bg-blue-50' : 
            'bg-gray-50 dark:bg-gray-700 sepia:bg-amber-100/50 morning:bg-blue-100/50'}"
        >
          <div class="text-right text-sm {isToday(date) ? 'font-bold' : ''} dark:text-white sepia:text-amber-900 morning:text-blue-900">
            {format(date, 'd')}
          </div>
          {#if todos.length > 0}
            <div class="mt-1">
              {#each todos as todo}
                <div 
                  class="text-xs p-1 mb-1 rounded text-white"
                  style="background-color: red;}"
                >
                  {todo.text}
                </div>
              {/each}
            </div>
          {/if}
        </div>
      {/each}
    </div>
  </div>