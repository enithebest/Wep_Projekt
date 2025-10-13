<script>
import { createEventDispatcher } from 'svelte';
import { parseISO, format, isAfter } from 'date-fns';
const dispatch = createEventDispatcher();
export let issue;


function onDelete(e) { e.stopPropagation(); dispatch('delete', issue.id); }
function onShare(e) { e.stopPropagation(); dispatch('share', issue.id); }
function onExportICS(e) { e.stopPropagation(); dispatch('exportics', issue.id); }


$: isOverdue = issue.dueDate ? isAfter(new Date(), parseISO(issue.dueDate)) : false;
</script>


<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_noninteractive_element_interactions -->
<article class="p-4 rounded-lg cursor-pointer border-l-4 transition-all" draggable="true" on:click={() => dispatch('edit', issue)}>
<div class={isOverdue ? 'border-l-red-500 bg-red-50' : 'border-l-indigo-500'}>
<h3 class="font-semibold">{issue.title}</h3>
{#if issue.description}
<p class="text-sm text-gray-600 line-clamp-2">{issue.description}</p>
{/if}
<p class="text-xs text-gray-500">SP: {issue.storyPoints} | Priority: {issue.priority}</p>
{#if issue.dueDate}
<p class="text-xs">Due: {format(parseISO(issue.dueDate), 'PP')}</p>
{/if}
<div class="flex gap-2 mt-2">
<button class="text-xs text-red-600" on:click|stopPropagation={onDelete}>Delete</button>
<button class="text-xs" on:click|stopPropagation={onShare}>Share</button>
<button class="text-xs" on:click|stopPropagation={onExportICS}>Export ICS</button>
</div>
</div>
</article>