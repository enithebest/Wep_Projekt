<script>
	import { parseISO, format, isAfter } from 'date-fns';

	let {
		issue,
		ondelete = () => {},
		onshare = () => {},
		onexportics = () => {},
		ondragstart = () => {},
		onedit = () => {}
	} = $props();

	function handleDelete() {
		ondelete(issue.id);
	}

	function handleShare() {
		onshare(issue.id);
	}

	function handleExportICS() {
		onexportics(issue.id);
	}

	function handleDragStart(e) {
		e.dataTransfer.setData('text/issue-id', String(issue.id));
		ondragstart({ id: issue.id, event: e });
	}

	function handleClick() {
		onedit(issue);
	}

	function handleKeyDown(e) {
		if (e.key === 'Enter' || e.key === ' ') {
			e.preventDefault();
			onedit(issue);
		}
	}

	let isOverdue = $derived(issue?.dueDate ? isAfter(new Date(), parseISO(issue.dueDate)) : false);
</script>

<!-- svelte-ignore a11y_no_noninteractive_element_to_interactive_role -->
<article
	class="w-full max-w-xs bg-white dark:bg-gray-800 rounded-lg shadow-md border-l-4 p-4 cursor-pointer hover:shadow-lg transition-shadow duration-200"
	draggable="true"
	ondragstart={handleDragStart}
	onclick={handleClick}
	onkeydown={handleKeyDown}
	role="button"
	tabindex="0"
	aria-label={`Issue: ${issue.title}`}
>
	<div
		class="{isOverdue ? 'border-l-red-500 bg-red-50 dark:bg-red-900/30' : 'border-l-indigo-500 bg-gray-50 dark:bg-gray-700'} p-3 rounded-md"
	>
		<h3 class="text-base font-semibold text-gray-900 dark:text-gray-100 line-clamp-1">
			{issue.title}
		</h3>
		{#if issue.description}
			<p class="text-sm text-gray-600 dark:text-gray-300 line-clamp-2 mt-1">
				{issue.description}
			</p>
		{/if}
		<div class="flex flex-wrap gap-2 mt-2 text-xs text-gray-500 dark:text-gray-400">
			<span>SP: {issue.storyPoints}</span>
			<span>|</span>
			<span>Priority: {issue.priority}</span>
			{#if issue.dueDate}
				<span>|</span>
				<span class={isOverdue ? 'text-red-600 dark:text-red-400' : ''}>
					Due: {format(parseISO(issue.dueDate), 'PP')}
				</span>
			{/if}
		</div>
		<div class="flex gap-3 mt-3">
			<button
				class="text-xs text-red-600 hover:text-red-700 dark:text-red-400 dark:hover:text-red-300 font-medium hover:underline focus:outline-none focus:ring-2 focus:ring-red-500"
				onclick={(event) => {
					event.stopPropagation();
					handleDelete();
				}}
			>
				Delete
			</button>
			<button
				class="text-xs text-blue-600 hover:text-blue-700 dark:text-blue-400 dark:hover:text-blue-300 font-medium hover:underline focus:outline-none focus:ring-2 focus:ring-blue-500"
				onclick={(event) => {
					event.stopPropagation();
					handleShare();
				}}
			>
				Share
			</button>
			<button
				class="text-xs text-green-600 hover:text-green-700 dark:text-green-400 dark:hover:text-green-300 font-medium hover:underline focus:outline-none focus:ring-2 focus:ring-green-500"
				onclick={(event) => {
					event.stopPropagation();
					handleExportICS();
				}}
			>
				Export ICS
			</button>
		</div>
	</div>
</article>