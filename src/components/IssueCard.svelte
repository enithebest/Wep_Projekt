<script>
  import { createEventDispatcher } from 'svelte';
  import { parseISO, format, isAfter } from 'date-fns';

  export let issue;
  const dispatch = createEventDispatcher();

  function onDelete(e) {
    // if you keep stopPropagation here it's fine;
    // I'll leave it in case you're attaching click to the article too
    e.stopPropagation();
    dispatch('delete', issue.id);
  }

  function onShare(e) {
    e.stopPropagation();
    dispatch('share', issue.id);
  }

  function onExportICS(e) {
    e.stopPropagation();
    dispatch('exportics', issue.id);
  }

  function handleDragStart(e) {

    e.dataTransfer.setData('text/issue-id', String(issue.id));
    // include the id and (optionally) the native event
    dispatch('dragstart', { id: issue.id, event: e });
  }

  function handleClick() {
    dispatch('edit', issue);
  }

  function handleKeyDown(e) {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      dispatch('edit', issue);
    }
  }

  $: isOverdue = issue?.dueDate ? isAfter(new Date(), parseISO(issue.dueDate)) : false;
</script>

<!-- svelte-ignore a11y_no_noninteractive_element_to_interactive_role -->
<article
  class="p-4 rounded-lg cursor-pointer border-l-4 transition-all"
  draggable="true"
  ondragstart={handleDragStart}
  onclick={handleClick}
  onkeydown={handleKeyDown}
  role="button"
  tabindex="0"
  aria-label={`Issue: ${issue.title}`}
>
  <div class={isOverdue ? 'border-l-red-500 bg-red-50 p-2 rounded' : 'border-l-indigo-500 p-2 rounded'}>
    <h3 class="font-semibold">{issue.title}</h3>

    {#if issue.description}
      <p class="text-sm text-gray-600 line-clamp-2">{issue.description}</p>
    {/if}

    <p class="text-xs text-gray-500">SP: {issue.storyPoints} | Priority: {issue.priority}</p>

    {#if issue.dueDate}
      <p class="text-xs">Due: {format(parseISO(issue.dueDate), 'PP')}</p>
    {/if}

    <div class="flex gap-2 mt-2">
      <!-- use Svelte event modifiers to stop propagation (so article click doesn't fire) -->
      <button class="text-xs text-red-600 hover:underline" onclickstopPropagation={onDelete}>Delete</button>
      <button class="text-xs hover:underline" onclickstopPropagation={onShare}>Share</button>
      <button class="text-xs hover:underline" onclickstopPropagation={onExportICS}>Export ICS</button>
    </div>
  </div>
</article>
