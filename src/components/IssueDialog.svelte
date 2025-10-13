<script>
import { createEventDispatcher } from 'svelte';
const dispatch = createEventDispatcher();


export let open = false;
export let issue = null; // null => create


let dialogEl;
export let form = {
title: '',
description: '',
dueDate: '',
storyPoints: 1,
priority: 'Medium'
};


// ensure dialog shows/closes when `open` changes
$: if (open && dialogEl) {
try { dialogEl.showModal(); } catch (e) { /* already open */ }
}


function close() {
dialogEl.close();
dispatch('close');
}


function submitCreate() {
dispatch('create', { ...form });
close();
}


function submitUpdate() {
dispatch('update', { ...form });
close();
}
</script>


<dialog bind:this={dialogEl} class="rounded-lg p-6" on:cancel|preventDefault={close}>
<h2 class="text-lg font-semibold mb-3">{issue ? 'Edit Issue' : 'New Issue'}</h2>


<div class="space-y-3">
<input placeholder="Title" bind:value={form.title} class="w-full border rounded px-3 py-2" />
<textarea placeholder="Description" bind:value={form.description} rows="4" class="w-full border rounded px-3 py-2"></textarea>
<div class="grid grid-cols-3 gap-2">
<input type="date" bind:value={form.dueDate} class="border rounded px-2 py-1" />
<input type="number" min="0" bind:value={form.storyPoints} class="border rounded px-2 py-1" />
<select bind:value={form.priority} class="border rounded px-2 py-1">
<option>Low</option>
<option>Medium</option>
<option>High</option>
</select>
</div>
</div>


<div class="flex justify-end gap-2 mt-4">
<button on:click={close} class="px-3 py-1 border rounded">Cancel</button>
{#if issue}
<button on:click={submitUpdate} class="px-3 py-1 bg-yellow-500 text-white rounded">Update</button>
{:else}
<button on:click={submitCreate} class="px-3 py-1 bg-green-600 text-white rounded">Create</button>
{/if}
</div>
</dialog>