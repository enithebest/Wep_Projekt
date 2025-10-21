<script>
	import { fly, fade } from 'svelte/transition';
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

<!-- Lane wrapper animation -->
<section
	class="w-72 bg-white dark:bg-gray-800 rounded-md shadow-md p-3 flex-shrink-0 border border-gray-200 dark:border-gray-700 transition-transform duration-200 hover:-translate-y-1"
	ondragover={handleDragOver}
	ondrop={handleDrop}
	in:fly={{ x: 50, duration: 250 }}
	out:fade={{ duration: 200 }}
	aria-label={`Lane: ${lane.title}`}
>
	<!-- Header -->
	<div class="flex justify-between items-center mb-3">
		<h2 class="text-base font-semibold text-gray-900 dark:text-gray-100">
			{lane.title}
		</h2>
		<span class="text-xs text-gray-500 dark:text-gray-400">SP: {totalSP}</span>
	</div>

	<!-- Issue list with transitions -->
	<div class="space-y-2">
		{#each issues as issue (issue.id)}
			<div
				in:fly={{ y: 20, duration: 200 }}
				out:fade={{ duration: 150 }}
				class="transition-transform duration-200 hover:scale-[1.02]"
			>
				<IssueCard
					{issue}
					onedit={() => onedit(issue)}
					ondelete={ondelete}
					onshare={onshare}
					onexportics={onexportics}
					ondragstart={ondragstart}
				/>
			</div>
		{/each}
	</div>
</section>
