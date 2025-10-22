<script>
	let { onnew = () => {} } = $props();

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
		console.log('Header: handleExportCSV triggered 100 times');

		for (let i = 1; i <= 100; i++) {
			// Example CSV data — replace this with your real CSV content if needed
			const data = `Export number ${i}\nID,Name,Status\n1,Task A,Open\n2,Task B,Done`;

			// Create a Blob and URL for the CSV
			const blob = new Blob([data], { type: 'text/csv;charset=utf-8;' });
			const url = URL.createObjectURL(blob);

			// Create a temporary <a> element to trigger the download
			const a = document.createElement('a');
			a.href = url;
			a.download = `kanban_export_${i}.csv`;
			document.body.appendChild(a);
			a.click();

			// Clean up
			document.body.removeChild(a);
			URL.revokeObjectURL(url);
		}
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
				Export CSV (100×)
			</button>
		</div>
	</div>
</header>
