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

  function handleDragOver(e) {
    e.preventDefault();
    e.dataTransfer.dropEffect = 'move';
  }

  function handleDrop(e) {
    e.preventDefault();
    onDrop(e);
  }
</script>

<section
  class="bg-white border rounded-xl p-4 w-80 flex-shrink-0"
  ondragover={handleDragOver}
  ondrop={handleDrop}
  aria-label={`Lane: ${lane.title}`}
>
  <div class="flex justify-between items-center mb-4">
    <h2 class="text-lg font-semibold">{lane.title}</h2>
    <span class="text-sm text-gray-500">
      SP: {issues.reduce((s, i) => s + Number(i.storyPoints || 0), 0)}
    </span>
  </div>

  {#each issues as issue (issue.id)}
    <IssueCard
      {issue}
      onedit={() => onEdit(issue)}
      ondelete={(e) => onDelete(e.detail)}
      onshare={(e) => onShare(e.detail)}
      onexportics={(e) => onExportICS(e.detail)}
      ondragstart={(e) => onDragStart(e.detail || e)}
    />
  {/each}
</section>
