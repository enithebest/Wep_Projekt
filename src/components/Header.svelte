<script>
	let { onnew = () => {}, onexportcsv = () => {} } = $props();

	let country = $state('Unknown');

	$effect(() => {
		const fetchCountry = async () => {
			try {
				const r = await fetch('https://ipapi.co/json/');
				if (r.ok) {
					const data = await r.json();
					country = data.country_name || data.country || 'Unknown';
				}
			} catch (e) {
				country = 'Unknown';
			}
		};
		fetchCountry();
	});

	function handleNew() {
		console.log('Header: handleNew triggered');
		onnew();
	}

	function handleExportCSV() {
		console.log('Header: handleExportCSV triggered');
		onexportcsv();
	}
</script>

<header class="bg-white shadow-sm border-b p-4">
	<div class="max-w-7xl mx-auto flex justify-between items-center">
		<h1 class="text-2xl font-bold">Kanban Board</h1>
		<div class="flex gap-2 items-center">
			<div class="text-sm text-gray-600" aria-label={`Country: ${country}`}>🌍 {country}</div>
			<button class="px-3 py-1 bg-indigo-600 text-white rounded hover:bg-indigo-700" on:click={handleNew}>
				+ New Issue
			</button>
			<button class="px-3 py-1 border rounded hover:bg-gray-100" on:click={handleExportCSV}>
				Export CSV
			</button>
		</div>
	</div>
</header>