<script>
  import { onMount, createEventDispatcher } from 'svelte';

  export let onNew = () => {};
  export let onExportCSV = () => {};
  const dispatch = createEventDispatcher();

  let country = 'Unknown';

  onMount(async () => {
    try {
      const r = await fetch('https://ipapi.co/json/');
      if (r.ok) {
        const data = await r.json();
        country = data.country_name || data.country || 'Unknown';
      }
    } catch (e) {
      country = 'Unknown';
    }
  });

  function handleNew() {
    console.log('Header: handleNew triggered');
    dispatch('new');
    onNew(); // Directly call the prop as a fallback
  }

  function handleExportCSV() {
    console.log('Header: handleExportCSV triggered');
    dispatch('exportcsv');
    onExportCSV();
  }
</script>

<header class="bg-white shadow-sm border-b p-4">
  <div class="max-w-7xl mx-auto flex justify-between items-center">
    <h1 class="text-2xl font-bold">Kanban Board</h1>
    <div class="flex gap-2 items-center">
      <div class="text-sm text-gray-600" aria-label={`Country: ${country}`}>🌍 {country}</div>
      <button
        class="px-3 py-1 bg-indigo-600 text-white rounded hover:bg-indigo-700"
        onclick={handleNew}
      >
        + New Issue
      </button>
      <button
        class="px-3 py-1 border rounded hover:bg-gray-100"
        onclick={handleExportCSV}
      >
        Export CSV
      </button>
    </div>
  </div>
</header>