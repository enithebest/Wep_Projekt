<script>
	import IssueCard from './IssueCard.svelte';

	let {
		lane,
		issues = [],
		ondrop = () => {},
		ondragstart = () => {},
		onedit = () => {},
		ondelete = () => {},
		onshare = () => {},
		onexportics = () => {}
	} = $props();

	function handleDragOver(e) {
		e.preventDefault();
		e.dataTransfer.dropEffect = 'move';
	}

	function handleDrop(e) {
		e.preventDefault();
		ondrop(e);
	}

	let totalSP = $derived(issues.reduce((s, i) => s + Number(i.storyPoints || 0), 0));
</script>

<section
	class="w-72 bg-white dark:bg-gray-800 rounded-md shadow-sm p-3 flex-shrink-0 border border-gray-200 dark:border-gray-700"
	ondragover={handleDragOver}
	ondrop={handleDrop}
	aria-label={`Lane: ${lane.title}`}
>
	<div class="flex justify-between items-center mb-3">
		<h2 class="text-base font-medium text-gray-900 dark:text-gray-100">
			{lane.title}
		</h2>
		<span class="text-xs text-gray-500 dark:text-gray-400">
			SP: {totalSP}
		</span>
	</div>
	<div class="space-y-2">
		{#each issues as issue (issue.id)}
			<IssueCard
				{issue}
				onedit={() => onedit(issue)}
				ondelete={ondelete}
				onshare={onshare}
				onexportics={onexportics}
				ondragstart={ondragstart}
			/>
		{/each}
	</div>
</section>