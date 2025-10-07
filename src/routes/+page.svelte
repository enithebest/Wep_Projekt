<script>
	let listA = [1, 2, 3];
	let listB = [4];
</script>

<h1 class="text-center text-xl font-semibold mb-4">Drag & Drop Practice</h1>

<main class="flex justify-center gap-6 p-8 bg-gray-100 h-[400px]">
	<section class="h-[350px] w-[100px] bg-white border-2 border-black flex flex-col items-center justify-center">
		{#each listA as item}
			<article class="p-3 bg-gray-400 rounded-md mb-2">{item}</article>
		{/each}
	</section>

	<section class="h-[350px] w-[100px] bg-white border-2 border-black flex flex-col items-center justify-center">
		{#each listB as item}
			<article class="p-3 bg-gray-400 rounded-md mb-2">{item}</article>
		{/each}
	</section>
</main>
