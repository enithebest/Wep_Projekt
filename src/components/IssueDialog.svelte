<script>
	let {
		open = $bindable(false),
		issue = null,
		onclose = () => {},
		oncreate = () => {},
		onupdate = () => {}
	} = $props();

	let form = $state({
		title: '',
		description: '',
		dueDate: '',
		storyPoints: 1,
		priority: 'Medium'
	});

	let dialogEl;

	$effect(() => {
		if (issue) {
			form = {
				title: issue.title ?? '',
				description: issue.description ?? '',
				dueDate: issue.dueDate ?? '',
				storyPoints: issue.storyPoints ?? 1,
				priority: issue.priority ?? 'Medium',
				id: issue.id
			};
		} else {
			form = {
				title: '',
				description: '',
				dueDate: '',
				storyPoints: 1,
				priority: 'Medium'
			};
		}
	});

	$effect(() => {
		if (open && dialogEl) {
			console.log('IssueDialog: Opening dialog');
			dialogEl.showModal();
		} else if (dialogEl) {
			console.log('IssueDialog: Closing dialog');
			dialogEl.close();
		}
	});

	$effect(() => {
		console.log('IssueDialog: Mounted');
		return () => {
			if (dialogEl) dialogEl.close();
			console.log('IssueDialog: Unmounted');
		};
	});

	function handleClose() {
		console.log('IssueDialog: close triggered');
		onclose();
	}

	function submitCreate() {
		console.log('IssueDialog: submitCreate triggered', form);
		oncreate({ ...form });
		handleClose();
	}

	function submitUpdate() {
		console.log('IssueDialog: submitUpdate triggered', form);
		onupdate({ ...form });
		handleClose();
	}
</script>

<dialog
	bind:this={dialogEl}
	class="w-full max-w-sm mx-auto rounded-md p-4 bg-white shadow-sm backdrop:bg-black/50 dark:bg-gray-800 dark:shadow-gray-900"
	onclose={handleClose}
	aria-labelledby="dialog-title"
>
	<h2
		id="dialog-title"
		class="text-lg font-medium text-gray-900 mb-3 dark:text-gray-100"
	>
		{issue ? 'Edit Issue' : 'New Issue'}
	</h2>
	<div class="space-y-3">
		<input
			placeholder="Title"
			bind:value={form.title}
			class="w-full border border-gray-200 rounded-md px-3 py-1.5 text-sm text-gray-700 focus:outline-none focus:ring-1 focus:ring-blue-400 dark:bg-gray-700 dark:border-gray-600 dark:text-gray-200 dark:placeholder-gray-400"
			required
			aria-label="Issue title"
		/>
		<textarea
			placeholder="Description"
			bind:value={form.description}
			rows="3"
			class="w-full border border-gray-200 rounded-md px-3 py-1.5 text-sm text-gray-700 focus:outline-none focus:ring-1 focus:ring-blue-400 dark:bg-gray-700 dark:border-gray-600 dark:text-gray-200 dark:placeholder-gray-400"
			aria-label="Issue description"
		></textarea>
		<div class="grid grid-cols-3 gap-2">
			<input
				type="date"
				bind:value={form.dueDate}
				class="border border-gray-200 rounded-md px-2 py-1 text-sm text-gray-700 focus:outline-none focus:ring-1 focus:ring-blue-400 dark:bg-gray-700 dark:border-gray-600 dark:text-gray-200"
				aria-label="Due date"
			/>
			<input
				type="number"
				min="0"
				bind:value={form.storyPoints}
				class="border border-gray-200 rounded-md px-2 py-1 text-sm text-gray-700 focus:outline-none focus:ring-1 focus:ring-blue-400 dark:bg-gray-700 dark:border-gray-600 dark:text-gray-200"
				aria-label="Story points"
			/>
			<select
				bind:value={form.priority}
				class="border border-gray-200 rounded-md px-2 py-1 text-sm text-gray-700 focus:outline-none focus:ring-1 focus:ring-blue-400 dark:bg-gray-700 dark:border-gray-600 dark:text-gray-200"
				aria-label="Priority"
			>
				<option value="Low">Low</option>
				<option value="Medium">Medium</option>
				<option value="High">High</option>
			</select>
		</div>
	</div>
	<div class="flex justify-end gap-2 mt-4">
		<button
			onclick={handleClose}
			class="px-3 py-1 text-sm text-gray-600 hover:text-gray-800 dark:text-gray-300 dark:hover:text-gray-100 focus:outline-none focus:ring-1 focus:ring-gray-400 dark:focus:ring-gray-500"
		>
			Cancel
		</button>
		{#if issue}
			<button
				onclick={submitUpdate}
				class="px-3 py-1 text-sm bg-yellow-400 text-white rounded-md hover:bg-yellow-500 focus:outline-none focus:ring-1 focus:ring-yellow-400 dark:bg-yellow-500 dark:hover:bg-yellow-600"
			>
				Update
			</button>
		{:else}
			<button
				onclick={submitCreate}
				class="px-3 py-1 text-sm bg-green-400 text-white rounded-md hover:bg-green-500 focus:outline-none focus:ring-1 focus:ring-green-400 dark:bg-green-500 dark:hover:bg-green-600"
			>
				Create
			</button>
		{/if}
	</div>
</dialog>