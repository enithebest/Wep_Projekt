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
	class="rounded-lg p-6 backdrop:bg-black/50"
	on:close={handleClose}
	aria-labelledby="dialog-title"
>
	<h2 id="dialog-title" class="text-lg font-semibold mb-3">
		{issue ? 'Edit Issue' : 'New Issue'}
	</h2>
	<div class="space-y-3">
		<input
			placeholder="Title"
			bind:value={form.title}
			class="w-full border rounded px-3 py-2"
			required
			aria-label="Issue title"
		/>
		<textarea
			placeholder="Description"
			bind:value={form.description}
			rows="4"
			class="w-full border rounded px-3 py-2"
			aria-label="Issue description"
		></textarea>
		<div class="grid grid-cols-3 gap-2">
			<input type="date" bind:value={form.dueDate} class="border rounded px-2 py-1" aria-label="Due date" />
			<input
				type="number"
				min="0"
				bind:value={form.storyPoints}
				class="border rounded px-2 py-1"
				aria-label="Story points"
			/>
			<select bind:value={form.priority} class="border rounded px-2 py-1" aria-label="Priority">
				<option>Low</option>
				<option>Medium</option>
				<option>High</option>
			</select>
		</div>
	</div>
	<div class="flex justify-end gap-2 mt-4">
		<button on:click={handleClose} class="px-3 py-1 border rounded hover:bg-gray-100"> Cancel </button>
		{#if issue}
			<button on:click={submitUpdate} class="px-3 py-1 bg-yellow-500 text-white rounded hover:bg-yellow-600">
				Update
			</button>
		{:else}
			<button on:click={submitCreate} class="px-3 py-1 bg-green-600 text-white rounded hover:bg-green-600">
				Create
			</button>
		{/if}
	</div>
</dialog>