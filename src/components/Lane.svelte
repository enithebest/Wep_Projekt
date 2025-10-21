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
	class="bg-white border rounded-xl p-4 w-80 flex-shrink-0"
	on:dragover={handleDragOver}
	on:drop={handleDrop}
	aria-label={`Lane: ${lane.title}`}
>
	<div class="flex justify-between items-center mb-4">
		<h2 class="text-lg font-semibold">{lane.title}</h2>
		<span class="text-sm text-gray-500"> SP: {totalSP} </span>
	</div>
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
</section>