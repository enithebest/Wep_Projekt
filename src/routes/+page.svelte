<script>
	import { flip } from "svelte/animate";

	// Define the lanes dynamically
	const LANES = [
		{ id: 'todo', title: 'Do' },
		{ id: 'doing', title: 'Doing' },
		{ id: 'done', title: 'Done' },
		{ id: 'archive', title: 'Archive' }
	];

	// A shared list of draggable items with lane property
	let items = [
		{ id: 1, title: 'Task 1', lane: 'todo' },
		{ id: 2, title: 'Task 2', lane: 'todo' },
		{ id: 3, title: 'Task 3', lane: 'doing' },
		{ id: 4, title: 'Task 4', lane: 'done' }
	];

	function handleDragStart(id, fromLane, e) {
		e.dataTransfer.setData("id", id);
		e.dataTransfer.setData("from", fromLane);
	}

	function handleDragOver(e) {
		e.preventDefault();
	}

	function handleDrop(toLane, e) {
		e.preventDefault();
		const id = Number(e.dataTransfer.getData("id"));
		const fromLane = e.dataTransfer.getData("from");

		if (fromLane === toLane) return;

		items = items.map(it =>
			it.id === id ? { ...it, lane: toLane } : it
		);
	}
</script>

<h1 class="text-center text-xl font-semibold mb-4">Drag & Drop –- Dynamic Lanes</h1>

<main class="flex justify-center gap-6 p-8 bg-gray-100 min-h-[400px]">
	{#each LANES as lane}
		<!-- svelte-ignore a11y_no_static_element_interactions -->
		<section
			class="h-[350px] w-[150px] bg-white border-2 border-black flex flex-col items-center justify-start p-2"
			ondragover={handleDragOver}
			ondrop={(e) => handleDrop(lane.id, e)}
		>
			<h2 class="font-bold mb-2">{lane.title}</h2>

			{#each items.filter(i => i.lane === lane.id) as item (item.id)}
				<article
					class="p-3 bg-gray-400 rounded-md mb-2 cursor-move w-full text-center"
					draggable="true"
					ondragstart={(e) => handleDragStart(item.id, lane.id, e)}
					animate:flip
				>
					{item.title}
				</article>
			{/each}
		</section>
	{/each}
</main>
