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
			<button
				class="text-xs text-red-600 hover:underline"
				onclick={() => {
					// ✅ This works and still stops propagation manually
					event.stopPropagation();
					handleDelete();
				}}
			>
				Delete
			</button>

			<button
				class="text-xs hover:underline"
				onclick={() => {
					event.stopPropagation();
					handleShare();
				}}
			>
				Share
			</button>

			<button
				class="text-xs hover:underline"
				onclick={() => {
					event.stopPropagation();
					handleExportICS();
				}}
			>
				Export ICS
			</button>
		</div>
	</div>
</article>
