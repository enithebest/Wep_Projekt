<script>
	import { flip } from "svelte/animate";

	let boxA = [1, 2];
	let boxB = [3];
	let boxC = [4];
	let boxD = [];

	function handleDragStart(item, from, e) {
		e.dataTransfer.setData("item", item);
		e.dataTransfer.setData("from", from);
	}

	function handleDragOver(e) {
		e.preventDefault();
	}

	function handleDrop(to, e) {
		e.preventDefault();

		const item = Number(e.dataTransfer.getData("item"));
		const from = e.dataTransfer.getData("from");

		if (from === "A") boxA = boxA.filter(i => i !== item);
		if (from === "B") boxB = boxB.filter(i => i !== item);
		if (from === "C") boxC = boxC.filter(i => i !== item);
		if (from === "D") boxD = boxD.filter(i => i !== item);

		if (to === "A") boxA.push(item);
		if (to === "B") boxB.push(item);
		if (to === "C") boxC.push(item);
		if (to === "D") boxD.push(item);
	}
</script>

<h1 class="text-center text-xl font-semibold mb-4">Drag & Drop – Four Lists</h1>

<main class="flex justify-center gap-6 p-8 bg-gray-100 h-[400px]">
	{#each [
		{ id: "A", items: boxA },
		{ id: "B", items: boxB },
		{ id: "C", items: boxC },
		{ id: "D", items: boxD }
	] as { id, items }}
		<!-- svelte-ignore a11y_no_static_element_interactions -->
		<section
			class="h-[350px] w-[100px] bg-white border-2 border-black flex flex-col items-center justify-start p-2"
			ondragover={handleDragOver}
			ondrop={(e) => handleDrop(id, e)}>
			<h2 class="font-bold mb-2">List {id}</h2>
			{#each items as item (item)}
				<article
					class="p-3 bg-gray-400 rounded-md mb-2 cursor-move"
					draggable="true"
					ondragstart={(e) => handleDragStart(item, id, e)}
					animate:flip>
					{item}
				</article>
			{/each}
		</section>
	{/each}
</main>

