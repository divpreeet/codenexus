<script>
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';
  
    // Initialize stores with persisted data
    const tasks = writable(JSON.parse(localStorage.getItem('tasks') || '[]'));
    const projects = writable(JSON.parse(localStorage.getItem('projects') || '[]'));
    const notes = writable(localStorage.getItem('notes') || '');
  
    // Save data to localStorage when it changes
    $: {
      localStorage.setItem('tasks', JSON.stringify($tasks));
      localStorage.setItem('projects', JSON.stringify($projects));
      localStorage.setItem('notes', $notes);
    }
  
    onMount(() => {
      // Load data from localStorage on component mount
      tasks.set(JSON.parse(localStorage.getItem('tasks') || '[]'));
      projects.set(JSON.parse(localStorage.getItem('projects') || '[]'));
      notes.set(localStorage.getItem('notes') || '');
    });
  
    import { tick } from 'svelte';
    import { fade, fly, scale, slide } from 'svelte/transition';
    import { cubicOut } from 'svelte/easing';
  
    let currentTime = new Date();
    let timerMinutes = 25;
    let timerSeconds = 0;
    let timerRunning = false;
    let timerInterval;
    let productivity = 5;
    let activeTab = 'dashboard';
    let newTask = '';
    let newProject = '';
  
    onMount(() => {
      const timer = setInterval(() => {
        currentTime = new Date();
      }, 1000);
  
      return () => clearInterval(timer);
    });
  
    function addTask() {
      if (newTask.trim()) {
        tasks.update(t => [...t, { id: Date.now(), text: newTask, completed: false }]);
        newTask = '';
      }
    }
  
    function toggleTask(id) {
      tasks.update(t => t.map(task =>
        task.id === id ? { ...task, completed: !task.completed } : task
      ));
    }
  
    function removeTask(id) {
      tasks.update(t => t.filter(task => task.id !== id));
    }
  
    function addProject() {
      if (newProject.trim()) {
        projects.update(p => [...p, { id: Date.now(), name: newProject }]);
        newProject = '';
      }
    }
  
    function removeProject(id) {
      projects.update(p => p.filter(project => project.id !== id));
    }
  
    function startTimer() {
      if (!timerRunning) {
        timerRunning = true;
        timerInterval = setInterval(() => {
          if (timerSeconds === 0) {
            if (timerMinutes === 0) {
              clearInterval(timerInterval);
              timerRunning = false;
            } else {
              timerMinutes--;
              timerSeconds = 59;
            }
          } else {
            timerSeconds--;
          }
        }, 1000);
      } else {
        clearInterval(timerInterval);
        timerRunning = false;
      }
    }
  
    function resetTimer() {
      clearInterval(timerInterval);
      timerRunning = false;
      timerMinutes = 25;
      timerSeconds = 0;
    }
  
    $: formattedTime = currentTime.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    $: formattedDate = currentTime.toLocaleDateString([], { weekday: 'long', month: 'long', day: 'numeric' });
  </script>
  
  <main class="min-h-screen bg-white text-gray-900 p-8 relative overflow-hidden">
    <div class="max-w-7xl mx-auto relative z-10">
      <header class="mb-12 flex justify-between items-center" in:fly="{{ y: -50, duration: 500, easing: cubicOut }}">
        <h1 class="text-5xl font-bold">CodeNexus</h1>
        <nav class="space-x-4">
          {#each ['dashboard', 'projects', 'notes'] as tab}
            <button
              class="px-4 py-2 rounded-full transition-all duration-300 {activeTab === tab ? 'bg-gray-900 text-white' : 'text-gray-900 hover:bg-gray-200'}"
              on:click={() => activeTab = tab}
            >
              {tab.charAt(0).toUpperCase() + tab.slice(1)}
            </button>
          {/each}
        </nav>
      </header>
  
      <div class="text-center mb-6 text-gray-600">
        <p>{formattedTime} - {formattedDate}</p>
      </div>
  
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        {#if activeTab === 'dashboard'}
          <section class="bg-gray-100 rounded-xl shadow-lg p-6" in:scale="{{ duration: 300, easing: cubicOut }}">
            <h2 class="text-2xl font-semibold mb-4">Task Tracker</h2>
            <form on:submit|preventDefault={addTask} class="mb-4 flex">
              <input
                bind:value={newTask}
                placeholder="Add a new task..."
                class="flex-grow p-3 bg-white rounded-l-full text-gray-900 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-gray-400"
              />
              <button type="submit" class="bg-gray-900 text-white px-6 py-3 rounded-r-full hover:bg-gray-800 transition-colors duration-300">Add</button>
            </form>
            <ul class="space-y-3 max-h-64 overflow-y-auto pr-2">
              {#each $tasks as task (task.id)}
                <li in:slide="{{ duration: 200 }}" out:fade="{{ duration: 150 }}" class="flex items-center bg-white rounded-full p-3 transition-all duration-300 hover:shadow-md hover:bg-gray-50">
                  <input
                    type="checkbox"
                    checked={task.completed}
                    on:change={() => toggleTask(task.id)}
                    class="mr-3 form-checkbox h-5 w-5 text-gray-900 rounded-full focus:ring-gray-400 focus:ring-offset-0"
                  />
                  <span class="flex-grow" class:line-through={task.completed}>{task.text}</span>
                  <button on:click={() => removeTask(task.id)} class="text-gray-500 hover:text-gray-900 transition-colors duration-300">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                      <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd" />
                    </svg>
                  </button>
                </li>
              {/each}
            </ul>
          </section>
  
          <section class="bg-gray-100 rounded-xl shadow-lg p-6" in:scale="{{ duration: 300, delay: 100, easing: cubicOut }}">
            <h2 class="text-2xl font-semibold mb-4">Focus Timer</h2>
            <div class="text-7xl font-bold text-center mb-6 font-mono" style="font-family: Inter;">
              {timerMinutes.toString().padStart(2, '0')}:{timerSeconds.toString().padStart(2, '0')}
            </div>
            <div class="flex justify-center space-x-4">
              <button
                on:click={startTimer}
                class="bg-gray-900 text-white px-8 py-3 rounded-full hover:bg-gray-800 transition-all duration-300 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-gray-400"
              >
                {timerRunning ? 'Pause' : 'Start'}
              </button>
              <button
                on:click={resetTimer}
                class="bg-gray-300 text-gray-900 px-8 py-3 rounded-full hover:bg-gray-400 transition-all duration-300 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-gray-400"
              >
                Reset
              </button>
            </div>
          </section>
  
          <section class="bg-gray-100 rounded-xl shadow-lg p-6" in:scale="{{ duration: 300, delay: 200, easing: cubicOut }}">
            <h2 class="text-2xl font-semibold mb-4">Productivity Pulse</h2>
            <input
              type="range"
              bind:value={productivity}
              min="1"
              max="10"
              class="w-full h-3 bg-gray-300 rounded-full appearance-none cursor-pointer"
            />
            <div class="mt-6 text-center">
              <span class="text-4xl font-bold">{productivity}</span>
              <p class="text-gray-600 mt-2">Rate your productivity today</p>
            </div>
            <div class="mt-6">
              <h3 class="text-xl font-semibold mb-2">Quick Tips</h3>
              <ul class="list-disc list-inside text-gray-600 space-y-2">
                <li>Take regular breaks</li>
                <li>Set clear goals for each session</li>
                <li>Stay hydrated</li>
                <li>Minimize distractions</li>
              </ul>
            </div>
          </section>
        {:else if activeTab === 'projects'}
          <section class="col-span-full bg-gray-100 rounded-xl shadow-lg p-6" in:scale="{{ duration: 300, easing: cubicOut }}">
            <h2 class="text-2xl font-semibold mb-4">Projects</h2>
            <form on:submit|preventDefault={addProject} class="mb-4 flex">
              <input
                bind:value={newProject}
                placeholder="Add a new project..."
                class="flex-grow p-3 bg-white rounded-l-full text-gray-900 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-gray-400"
              />
              <button type="submit" class="bg-gray-900 text-white px-6 py-3 rounded-r-full hover:bg-gray-800 transition-colors duration-300">Add</button>
            </form>
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
              {#each $projects as project (project.id)}
                <div in:fly="{{ y: 20, duration: 300, easing: cubicOut }}" out:fade="{{ duration: 150 }}" class="bg-white rounded-2xl p-4 transition-all duration-300 hover:shadow-md hover:bg-gray-50">
                  <h3 class="text-lg font-semibold mb-2">{project.name}</h3>
                  <button on:click={() => removeProject(project.id)} class="text-gray-500 hover:text-gray-900 transition-colors duration-300">
                    Remove
                  </button>
                </div>
              {/each}
            </div>
          </section>
        {:else if activeTab === 'notes'}
          <section class="col-span-full bg-gray-100 rounded-xl shadow-lg p-6" in:scale="{{ duration: 300, easing: cubicOut }}">
            <h2 class="text-2xl font-semibold mb-4">Quick Notes</h2>
            <textarea
              bind:value={$notes}
              class="w-full h-64 p-4 bg-white rounded-2xl text-gray-900 placeholder-gray-500 focus:outline-none focus:ring-2 focus:ring-gray-400 resize-none"
              placeholder="Jot down your thoughts..."
            ></textarea>
          </section>
        {/if}
      </div>
  
      <footer class="mt-12 text-center text-gray-600" in:fade="{{ duration: 500, delay: 400 }}">
        <p>Made by <a href="https://github.com/divpreeet" class="text-blue-600 hover:underline">Divpreet</a>, <a href="https://github.com/divpreeet/codenexus" class="text-blue-600 hover:underline">Source Code</a></p>
      </footer>
    </div>
  </main>
  
  <style global lang="postcss">
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
  
    :root {
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    }
  
    @tailwind base;
    @tailwind components;
    @tailwind utilities;
  
    html, body {
      @apply overflow-x-hidden;
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    }
  
    @keyframes pulse {
      0%, 100% {
        opacity: 1;
      }
      50% {
        opacity: 0.5;
      }
    }
  
    @keyframes float {
      0%, 100% {
        transform: translateY(0);
      }
      50% {
        transform: translateY(-10px);
      }
    }
  
    .animate-pulse {
      animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
    }
  
    .animate-float {
      animation: float 3s ease-in-out infinite;
    }
  
    ::-webkit-scrollbar {
      @apply w-2;
    }
  
    ::-webkit-scrollbar-track {
      @apply bg-gray-200 rounded-full;
    }
  
    ::-webkit-scrollbar-thumb {
      @apply bg-gray-400 rounded-full;
    }
  
    ::-webkit-scrollbar-thumb:hover {
      @apply bg-gray-500;
    }
  
    input[type="range"]::-webkit-slider-thumb {
      @apply appearance-none w-6 h-6 rounded-full bg-gray-900 cursor-pointer;
    }
  
    input[type="range"]::-moz-range-thumb {
      @apply w-6 h-6 rounded-full bg-gray-900 cursor-pointer border-none;
    }
  </style>
  