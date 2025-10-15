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
    dispatch('new');
    onNew();
  }
  function handleExportCSV() {
    dispatch('exportcsv');
    onExportCSV();
  }
</script>

<header class="bg-white shadow-sm border-b p-4">
  <div class="max-w-7xl mx-auto flex justify-between items-center">
    <h1 class="text-2xl font-bold">Kanban Board</h1>
    <div class="flex gap-2 items-center">
      <div class="text-sm text-gray-600">🌍 {country}</div>
      <button class="px-3 py-1 bg-indigo-600 text-white rounded" onclick={handleNew}>+ New Issue</button>
      <button class="px-3 py-1 border rounded" onclick={handleExportCSV}>Export CSV</button>
    </div>
  </div>
</header>
