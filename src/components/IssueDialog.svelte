<script>
	import { fade, scale } from 'svelte/transition';

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

	// svelte-ignore non_reactive_update
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
			dialogEl.showModal();
		} else if (dialogEl) {
			dialogEl.close();
		}
	});

	function handleClose() {
		onclose();
	}

	function submitCreate() {
		oncreate({ ...form });
		handleClose();
	}

	function submitUpdate() {
		onupdate({ ...form });
		handleClose();
	}
</script>

{#if open}
	<!-- Overlay fades in/out -->
	<div
		class="fixed inset-0 bg-black/40 backdrop-blur-sm flex items-center justify-center z-50"
		in:fade={{ duration: 200 }}
		out:fade={{ duration: 150 }}
	>
		<!-- Dialog scales smoothly -->
		<dialog
			bind:this={dialogEl}
			class="w-full max-w-sm mx-auto rounded-lg p-5 bg-white shadow-2xl dark:bg-gray-800 border border-gray-200 dark:border-gray-700"
			onclose={handleClose}
			open
			in:scale={{ start: 0.9, duration: 180 }}
			out:scale={{ end: 0.9, duration: 180 }}
			aria-labelledby="dialog-title"
		>
			<h2
				id="dialog-title"
				class="text-lg font-semibold text-gray-900 mb-3 dark:text-gray-100"
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

			<!-- Buttons -->
			<div class="flex justify-end gap-2 mt-5">
				<button
					onclick={handleClose}
					class="px-3 py-1 text-sm text-gray-600 hover:text-gray-800 dark:text-gray-300 dark:hover:text-gray-100 focus:outline-none focus:ring-1 focus:ring-gray-400 dark:focus:ring-gray-500 transition-transform hover:scale-[1.05]"
				>
					Cancel
				</button>

				{#if issue}
					<button
						onclick={submitUpdate}
						class="px-3 py-1 text-sm bg-yellow-400 text-white rounded-md hover:bg-yellow-500 focus:outline-none focus:ring-1 focus:ring-yellow-400 dark:bg-yellow-500 dark:hover:bg-yellow-600 transition-transform hover:scale-[1.05]"
					>
						Update
					</button>
				{:else}
					<button
						onclick={submitCreate}
						class="px-3 py-1 text-sm bg-green-400 text-white rounded-md hover:bg-green-500 focus:outline-none focus:ring-1 focus:ring-green-400 dark:bg-green-500 dark:hover:bg-green-600 transition-transform hover:scale-[1.05]"
					>
						Create
					</button>
				{/if}
			</div>
		</dialog>
	</div>
{/if}
