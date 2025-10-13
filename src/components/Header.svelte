    <script>

import { onMount } from 'svelte';
export let onNew = () => {};
export let onExportCSV = () => {};


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
</script>


<header class="bg-white shadow-sm border-b p-4">
<div class="max-w-7xl mx-auto flex justify-between items-center">
<h1 class="text-2xl font-bold">Kanban Board</h1>
<div class="flex gap-2 items-center">
<div class="text-sm text-gray-600">{country}</div>
<button class="px-3 py-1 bg-indigo-600 text-white rounded" on:click={onNew}>+ New Isue</button>
<button class="px-3 py-1 border rounded" on:click={onExportCSV}>Export CSV</button>
</div>
</div>
</header>