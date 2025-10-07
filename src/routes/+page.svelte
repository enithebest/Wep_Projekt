<script>
	import { flip } from "svelte/animate";

	let listA = [1, 2, 3];
	let listB = [4];

	function startDrag(item, source, e) {
		e.dataTransfer.setData("text/plain", item);
		e.dataTransfer.setData("source", source);
	}

	function dragOver(e) {
		e.preventDefault();
	}

	function dropTo(target, e) {
		e.preventDefault();
		const item = Number(e.dataTransfer.getData("text/plain"));
		const source = e.dataTransfer.getData("source");

		if (source === "A") listA = listA.filter(i => i !== item);
		if (source === "B") listB = listB.filter(i => i !== item);

		if (target === "A") listA.push(item);
		if (target === "B") listB.push(item);
	}
</script>

<h1 class="text-center text-xl font-semibold mb-4">Drag & Drop Practice</h1>

<main class="flex justify-center gap-6 p-8 bg-gray-100 h-[400px]">
	{#each [{ id: "A", list: listA }, { id: "B", list: listB }] as { id, list }}
		<!-- svelte-ignore a11y_no_static_element_interactions -->
		<section
			class="h-[350px] w-[100px] bg-white border-2 border-black flex flex-col items-center justify-center"
			ondragover={dragOver}
			ondrop={(e) => dropTo(id, e)}>
			<h2 class="font-bold mb-2">List {id}</h2>
			{#each list as item (item)}
				<article
					class="p-3 bg-gray-400 rounded-md mb-2 cursor-move"
					draggable="true"
					ondragstart={(e) => startDrag(item, id, e)}
					animate:flip>
					{item}
				</article>
			{/each}
		</section>
	{/each}
</main>
