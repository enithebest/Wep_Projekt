<script>
  import IssueCard from './IssueCard.svelte';
  export let lane;
  export let issues = [];
  export let onDrop = () => {};
  export let onDragStart = () => {};
  export let onEdit = () => {};
  export let onDelete = () => {};
  export let onShare = () => {};
  export let onExportICS = () => {};

  function handleDragOver(e) { e.preventDefault(); }
  function handleDrop(e) { e.preventDefault(); onDrop(lane.id, e); }
</script>

<!-- svelte-ignore a11y_no_static_element_interactions -->
<section class="bg-white border rounded-xl p-4 w-80 flex-shrink-0" on:dragover={handleDragOver} on:drop={handleDrop}>
  <div class="flex justify-between items-center mb-4">
    <h2 class="text-lg font-semibold">{lane.title}</h2>
    <span class="text-sm text-gray-500">SP: {issues.reduce((s,i) => s + Number(i.storyPoints || 0), 0)}</span>
  </div>

  {#each issues as issue (issue.id)}
    <IssueCard
      {issue}
      on:edit={() => onEdit(issue)}
      on:delete={(e) => onDelete(e.detail)}
      on:share={(e) => onShare(e.detail)}
      on:exportics={(e) => onExportICS(e.detail)}
      on:dragstart={(e) => onDragStart(e.detail || e)} />
  {/each}
</section>
