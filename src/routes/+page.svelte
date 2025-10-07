<script>
	let listA = [1, 2, 3];
	let listB = [4];

	function handleDragStart(item, from, e) {
		e.dataTransfer.setData("text/plain", item);
		e.dataTransfer.setData("from", from);
	}

	function handleDragOver(e) {
		e.preventDefault();
	}

	function handleDrop(to, e) {
		e.preventDefault();
		const item = Number(e.dataTransfer.getData("text/plain"));
		const from = e.dataTransfer.getData("from");

		if (from === "A") listA = listA.filter(i => i !== item);
		if (from === "B") listB = listB.filter(i => i !== item);

		if (to === "A") listA.push(item);
		if (to === "B") listB.push(item);
	}
</script>

<h1 class="text-center text-xl font-semibold mb-4">Drag & Drop Practice</h1>

<main class="flex justify-center gap-6 p-8 bg-gray-100 h-[400px]">
	<section
		class="h-[350px] w-[100px] bg-white border-2 border-black flex flex-col items-center justify-center"
		ondragover={handleDragOver}
		ondrop={(e) => handleDrop("A", e)}>
		{#each listA as item (item)}
			<article
				class="p-3 bg-gray-400 rounded-md mb-2 cursor-move"
				draggable="true"
				ondragstart={(e) => handleDragStart(item, "A", e)}>
				{item}
			</article>
		{/each}
	</section>

	<section
		class="h-[350px] w-[100px] bg-white border-2 border-black flex flex-col items-center justify-center"
		ondragover={handleDragOver}
		ondrop={(e) => handleDrop("B", e)}>
		{#each listB as item (item)}
			<article
				class="p-3 bg-gray-400 rounded-md mb-2 cursor-move"
				draggable="true"
				ondragstart={(e) => handleDragStart(item, "B", e)}>
				{item}
			</article>
		{/each}
	</section>
</main>
