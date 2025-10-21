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

<header
	class="bg-white dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700 shadow-sm px-4 py-2"
>
	<div class="max-w-7xl mx-auto flex justify-between items-center">
		<h1 class="text-lg font-medium text-gray-900 dark:text-gray-100">
			K A N B A N Board 🗂️
		</h1>
		<div class="flex gap-2 items-center">
			<div
				class="text-xs text-gray-500 dark:text-gray-400"
				aria-label={`Country: ${country}`}
			>
				🌍 {country}
			</div>
			<button
				class="px-2 py-1 text-sm bg-blue-400 text-white rounded-md hover:bg-blue-500 focus:outline-none focus:ring-1 focus:ring-blue-400 dark:bg-blue-500 dark:hover:bg-blue-600 dark:focus:ring-blue-500"
				onclick={handleNew}
			>
				+ New Issue
			</button>
			<button
				class="px-2 py-1 text-sm border border-gray-200 text-gray-600 rounded-md hover:bg-gray-100 focus:outline-none focus:ring-1 focus:ring-gray-400 dark:border-gray-600 dark:text-gray-300 dark:hover:bg-gray-700 dark:focus:ring-gray-500"
				onclick={handleExportCSV}
			>
				Export CSV
			</button>
		</div>
	</div>
</header>