<script lang="ts">
	let seats = 0;
	let rows = 0;
	let columns = 0;
	let tableData: number[][] = [];
	let columnLabels: string[] = [];
	let rowLabels: string[] = [];
	let columnType = 'letters';
	let rowType = 'numbers';
	let scale = 1;

	$: {
		if (seats && rows) {
			columns = Math.ceil(seats / rows);
			generateTable();
		}
	}

	function generateTable() {
		tableData = [];
		columnLabels = generateLabels(columns, columnType);
		rowLabels = generateLabels(rows, rowType);

		let seatNumber = 1;
		for (let r = 0; r < rows; r++) {
			let row = [];
			for (let c = 0; c < columns; c++) {
				row.push(seatNumber <= seats ? seatNumber++ : 0);
			}
			tableData.push(row);
		}
	}

	function generateLabels(count: number, type: string): string[] {
		let labels: string[] = [];
		for (let i = 0; i < count; i++) {
			labels.push(type === 'letters' ? getLetter(i) : (i + 1).toString());
		}
		return labels;
	}

	function getLetter(index: number): string {
		let letter = '';
		while (index >= 0) {
			letter = String.fromCharCode((index % 26) + 65) + letter;
			index = Math.floor(index / 26) - 1;
		}
		return letter;
	}

	function handleTouchStart(e: TouchEvent) {
		if (e.touches.length === 2) {
			e.preventDefault();
			const touch1 = e.touches[0];
			const touch2 = e.touches[1];
			const distance = Math.hypot(touch1.clientX - touch2.clientX, touch1.clientY - touch2.clientY);
			startPinchDistance = distance;
			startScale = scale;
		}
	}

	let startPinchDistance = 0;
	let startScale = 1;

	function handleTouchMove(e: TouchEvent) {
		if (e.touches.length === 2) {
			e.preventDefault();
			const touch1 = e.touches[0];
			const touch2 = e.touches[1];
			const distance = Math.hypot(touch1.clientX - touch2.clientX, touch1.clientY - touch2.clientY);

			const newScale = (distance / startPinchDistance) * startScale;
			scale = Math.min(Math.max(0.5, newScale), 3);
		}
	}

	function handleWheel(e: WheelEvent) {
		if (e.ctrlKey) {
			e.preventDefault();
			const delta = e.deltaY > 0 ? 0.9 : 1.1;
			scale = Math.min(Math.max(0.5, scale * delta), 3);
		}
	}
</script>

<div class="max-w-full bg-gray-50 p-4">
	<div class="mx-auto mb-6 max-w-md space-y-4 md:space-y-6">
		<div class="grid grid-cols-1 gap-4 md:grid-cols-2">
			<label class="block">
				<span class="text-gray-700">Seats:</span>
				<input
					type="number"
					bind:value={seats}
					min="1"
					class="mt-1 block w-full rounded-md border-gray-300 bg-white shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200"
				/>
			</label>

			<label class="block">
				<span class="text-gray-700">Rows:</span>
				<input
					type="number"
					bind:value={rows}
					min="1"
					class="mt-1 block w-full rounded-md border-gray-300 bg-white shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200"
				/>
			</label>
		</div>

		<div class="grid grid-cols-1 gap-4 md:grid-cols-3">
			<label class="block">
				<span class="text-gray-700">Columns:</span>
				<input
					type="number"
					bind:value={columns}
					min="1"
					readonly
					class="mt-1 block w-full cursor-not-allowed rounded-md border-gray-300 bg-gray-100"
				/>
			</label>

			<label class="block">
				<span class="text-gray-700">Column Type:</span>
				<select
					bind:value={columnType}
					on:change={() => generateTable()}
					class="mt-1 block w-full rounded-md border-gray-300 bg-white shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200"
				>
					<option value="letters">A, B, C...</option>
					<option value="numbers">1, 2, 3...</option>
				</select>
			</label>

			<label class="block">
				<span class="text-gray-700">Row Type:</span>
				<select
					bind:value={rowType}
					on:change={() => generateTable()}
					class="mt-1 block w-full rounded-md border-gray-300 bg-white shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200"
				>
					<option value="numbers">1, 2, 3...</option>
					<option value="letters">A, B, C...</option>
				</select>
			</label>
		</div>
	</div>

	<div
		class="relative max-h-[70vh] max-w-full overflow-auto rounded-lg border border-gray-200 bg-white shadow-lg"
		on:touchstart={handleTouchStart}
		on:touchmove={handleTouchMove}
		on:wheel={handleWheel}
	>
		<div class="seat-chart min-w-full" style="transform: scale({scale}); transform-origin: 0 0;">
			<table class="w-full border-collapse">
				<thead>
					<tr>
						<th class="corner-cell bg-blue-100 p-3"></th>
						{#each columnLabels as label}
							<th class="column-header p-3 text-sm font-semibold text-gray-700">
								{label}
							</th>
						{/each}
					</tr>
				</thead>
				<tbody>
					{#each tableData as row, rIndex}
						<tr>
							<td class="row-label p-3 text-sm font-semibold text-gray-700">
								{rowLabels[rIndex]}
							</td>
							{#each row as seat}
								<td
									class="border border-gray-200 p-3 text-center text-sm {seat
										? 'cursor-pointer hover:bg-blue-50'
										: 'bg-gray-50'}"
								>
									{seat || '-'}
								</td>
							{/each}
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	</div>

	<div class="mt-4 text-center text-sm text-gray-600">
		<p>Pinch or use Ctrl + Mouse Wheel to zoom</p>
		<p>Current zoom: {Math.round(scale * 100)}%</p>
	</div>
</div>

<style>
	/* Fixed header styles */
	.column-header {
		position: sticky;
		top: 0;
		z-index: 20;
		background-color: #f0f9ff;
		border: 1px solid #e5e7eb;
	}

	/* Fixed row label styles */
	.row-label {
		position: sticky;
		left: 0;
		z-index: 10;
		background-color: #f0f9ff;
		border: 1px solid #e5e7eb;
	}

	/* Corner cell (intersection of fixed header and row label) */
	.corner-cell {
		position: sticky;
		top: 0;
		left: 0;
		z-index: 30;
		background-color: #f0f9ff;
		border: 1px solid #e5e7eb;
	}

	/* Ensure the table container has proper scroll behavior */
	.seat-chart {
		position: relative;
	}

	/* Add box-shadow for better visual separation */
	.column-header,
	.row-label {
		box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
	}

	.corner-cell {
		box-shadow:
			0 1px 2px rgba(0, 0, 0, 0.05),
			1px 0 2px rgba(0, 0, 0, 0.05);
	}
</style>
