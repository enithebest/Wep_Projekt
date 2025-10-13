<script>
	import { flip } from "svelte/animate";
	import { format, parseISO, isAfter } from "date-fns";

	const STORAGE_KEY = 'kanban-items-v1';

	const LANES = [
		{ id: 'todo', title: 'Do' },
		{ id: 'doing', title: 'Doing' },
		{ id: 'done', title: 'Done' },
		{ id: 'archive', title: 'Archive' }
	];

	let items = loadFromStorage();
	let showForm = false;

	let newIssue = {
		title: '',
		description: '',
		dueDate: '',
		storyPoints: 1,
		priority: 'Medium'
	};

	function handleDragStart(id, fromLane, e) {
		e.dataTransfer.setData("id", id);
		e.dataTransfer.setData("from", fromLane);
	}

	function handleDragOver(e) {
		e.preventDefault();
	}

	function handleDrop(toLane, e) {
		e.preventDefault();
		const id = e.dataTransfer.getData("id");
		const fromLane = e.dataTransfer.getData("from");
		if (!id || fromLane === toLane) return;

		items = items.map(it => it.id === id ? { ...it, lane: toLane } : it);
		saveToStorage();
	}

	function createIssue() {
		if (!newIssue.title) return; // require title
		const id = crypto.randomUUID ? crypto.randomUUID() : String(Date.now());
		const created = new Date().toISOString();
		const issue = { id, ...newIssue, created, lane: 'todo' };
		items = [...items, issue];
		saveToStorage();
		showForm = false;
		newIssue = {
			title: '',
			description: '',
			dueDate: '',
			storyPoints: 1,
			priority: 'Medium'
		};
	}

	function saveToStorage() {
		localStorage.setItem(STORAGE_KEY, JSON.stringify(items));
	}

	function loadFromStorage() {
		try {
			const raw = localStorage.getItem(STORAGE_KEY);
			return raw ? JSON.parse(raw) : [];
		} catch (e) {
			return [];
		}
	}

	function removeItem(id) {
		items = items.filter(i => i.id !== id);
		saveToStorage();
	}

	function formatDate(d) {
		if (!d) return '—';
		try {
			return format(parseISO(d), 'dd.MM.yyyy');
		} catch {
			return d;
		}
	}

	function isOverdue(issue) {
		if (!issue.dueDate) return false;
		try {
			const due = parseISO(issue.dueDate);
			return isAfter(new Date(), due) && issue.lane !== 'done' && issue.lane !== 'archive';
		} catch {
			return false;
		}
	}
</script>

<header class="bg-white shadow-sm border-b border-gray-200 px-6 py-4">
	<div class="flex justify-between items-center max-w-7xl mx-auto">
		<h1 class="text-2xl font-bold text-gray-900">Kanban Board</h1>
		<button
			class="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg font-medium transition-colors"
			onclick={() => showForm = !showForm}>
			+ New Issue
		</button>
	</div>
</header>

{#if showForm}
	<div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
		<div class="bg-white p-6 rounded-xl shadow-xl w-full max-w-md">
			<h2 class="text-lg font-semibold text-gray-900 mb-4">New Issue</h2>

			<input class="w-full border border-gray-300 rounded-lg px-3 py-2 mb-3" placeholder="Title" bind:value={newIssue.title} required>
			<textarea class="w-full border border-gray-300 rounded-lg px-3 py-2 mb-3 resize-none" placeholder="Description" bind:value={newIssue.description} rows="3"></textarea>

			<div class="grid grid-cols-3 gap-3 mb-4">
				<input type="date" class="border border-gray-300 rounded-lg px-3 py-2" bind:value={newIssue.dueDate}>
				<input type="number" min="0" class="border border-gray-300 rounded-lg px-3 py-2" bind:value={newIssue.storyPoints}>
				<select class="border border-gray-300 rounded-lg px-3 py-2" bind:value={newIssue.priority}>
					<option>Low</option>
					<option>Medium</option>
					<option>High</option>
				</select>
			</div>

			<div class="flex justify-end gap-3">
				<button type="button" class="px-4 py-2 text-gray-600 border border-gray-300 rounded-lg hover:bg-gray-50" onclick={() => showForm = false}>Cancel</button>
				<button type="button" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg font-medium" onclick={createIssue}>Create</button>
			</div>
		</div>
	</div>
{/if}

<main class="max-w-7xl mx-auto p-6 bg-gray-50 min-h-[70vh]">
	<div class="flex gap-6 overflow-x-auto pb-4">
		{#each LANES as lane}
			<!-- svelte-ignore a11y_no_static_element_interactions -->
			<section
				class="bg-white border border-gray-200 rounded-xl p-4 w-80 flex-shrink-0 shadow-sm hover:shadow-md transition-shadow"
				ondragover={handleDragOver}
				ondrop={(e) => handleDrop(lane.id, e)}>

				<div class="flex justify-between items-center mb-4">
					<h2 class="text-lg font-semibold text-gray-900">{lane.title}</h2>
				</div>

				{#each items.filter(i => i.lane === lane.id) as issue (issue.id)}
					<article
						class="p-4 rounded-lg cursor-move border-l-4 transition-all hover:bg-gray-100"
						class:border-red-500={isOverdue(issue)}
						class:bg-red-50={isOverdue(issue)}
						class:border-indigo-500={!isOverdue(issue)}
						class:bg-white={!isOverdue(issue)}
						draggable="true"
						ondragstart={(e) => handleDragStart(issue.id, lane.id, e)}
						animate:flip>
						<h3 class="font-semibold text-gray-900 mb-1">{issue.title}</h3>

						{#if issue.description}
							<p class="text-sm text-gray-600 mb-2">{issue.description}</p>
						{/if}

						<div class="text-xs text-gray-500 space-y-1">
							<p>Created: {formatDate(issue.created)}</p>
							{#if issue.dueDate}
								<p class={isOverdue(issue) ? 'text-red-600 font-medium' : ''}>Due: {formatDate(issue.dueDate)}</p>
							{/if}
							<p>SP: {issue.storyPoints} | Priority: {issue.priority}</p>
						</div>

						<button class="mt-2 text-xs text-red-600 hover:text-red-800" onclick={() => removeItem(issue.id)}>Delete</button>
					</article>
				{/each}
			</section>
		{/each}
	</div>
</main>
