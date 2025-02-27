<script lang="ts">
	import { ZoomIn, ZoomOut } from 'lucide-svelte';

	let quantity = 0;
	let rows = 0;
	let columns = 0;
	let tableData: string[][] = [];
	let columnLabels: string[] = [];
	let rowLabels: string[] = [];
	let columnType = 'numbers';
	let rowType = 'letters';
	let scale = 1;
	let startPinchDistance = 0;
	let startScale = 1;

	function handleQuantityChange() {
		if (quantity) {
			if (rows) {
				columns = Math.ceil(quantity / rows);
				generateTable();
			} else if (columns) {
				rows = Math.ceil(quantity / columns);
				generateTable();
			}
		}
	}

	function handleRowsChange() {
		if (quantity && rows) {
			columns = Math.ceil(quantity / rows);
			generateTable();
		}
	}

	function handleColumnsChange() {
		if (quantity && columns) {
			rows = Math.ceil(quantity / columns);
			generateTable();
		}
	}

	function getLetter(index: number): string {
		let letter = '';
		while (index >= 0) {
			letter = String.fromCharCode((index % 26) + 65) + letter;
			index = Math.floor(index / 26) - 1;
		}
		return letter;
	}

	function generateLabels(count: number, type: string): string[] {
		return Array.from({ length: count }, (_, i) =>
			type === 'letters' ? getLetter(i) : (i + 1).toString()
		);
	}

	function generateTable() {
		if (!rows || !columns) return;

		tableData = [];
		columnLabels = generateLabels(columns, columnType);
		rowLabels = generateLabels(rows, rowType);

		const tableDataObject: { [key: string]: { paid: boolean } } = {};

		for (let r = 0; r < rows; r++) {
			const row: string[] = [];
			for (let c = 0; c < columns; c++) {
				const seatNumber = c + 1;
				let currentSeat = '';

				if (seatNumber <= quantity) {
					currentSeat =
						columnType === 'letters'
							? `${columnLabels[c]}${rowLabels[r]}`
							: `${rowLabels[r]}${columnLabels[c]}`;
				}

				row.push(currentSeat);

				if (currentSeat) {
					tableDataObject[currentSeat] = {
						paid: false
					};
				}
			}
			tableData.push(row);
		}

		console.log(JSON.stringify(tableDataObject, null, 2));
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

	function handleTouchMove(e: TouchEvent) {
		if (e.touches.length === 2) {
			e.preventDefault();
			const touch1 = e.touches[0];
			const touch2 = e.touches[1];
			const distance = Math.hypot(touch1.clientX - touch2.clientX, touch1.clientY - touch2.clientY);

			const newScale = (distance / startPinchDistance) * startScale;
			// Only allow zooming in if current scale is less than 1
			if (scale < 1 || newScale < scale) {
				scale = Math.min(Math.max(0.5, newScale), 1);
			}
		}
	}

	function handleWheel(e: WheelEvent) {
		if (e.ctrlKey) {
			e.preventDefault();
			const delta = e.deltaY > 0 ? 0.9 : 1.1;
			// Only allow zooming in if current scale is less than 1
			if (scale < 1 || delta < 1) {
				scale = Math.min(Math.max(0.5, scale * delta), 1);
			}
		}
	}

	function handleZoom(direction: 'in' | 'out') {
		const delta = direction === 'in' ? 1.1 : 0.9;
		const newScale = scale * delta;

		// Prevent zooming in beyond 100%
		if (newScale > 1) {
			scale = 1;
		} else {
			scale = Math.min(Math.max(0.5, newScale), 1);
		}
	}

	$: if (quantity || rows || columns || columnType || rowType) {
		generateTable();
	}
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-50 p-6">
	<div class="mx-auto max-w-7xl">
		<h1 class="mb-8 text-center text-3xl font-bold text-gray-800">Seating Visualizer</h1>

		<div class="mb-8 rounded-xl bg-white p-6 shadow-lg">
			<div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
				<div class="space-y-2">
					<label for="quantity" class="block text-sm font-medium text-gray-700">Quantity</label>
					<input
						type="number"
						id="quantity"
						bind:value={quantity}
						min="1"
						on:change={handleQuantityChange}
						class="w-full rounded-lg border border-gray-300 px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-200 focus:outline-none"
						placeholder="Enter total seats"
					/>
				</div>

				<div class="space-y-2">
					<label for="rows" class="block text-sm font-medium text-gray-700">Rows</label>
					<input
						type="number"
						id="rows"
						bind:value={rows}
						min="1"
						on:change={handleRowsChange}
						class="w-full rounded-lg border border-gray-300 px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-200 focus:outline-none"
						placeholder="Enter number of rows"
					/>
				</div>

				<div class="space-y-2">
					<label for="columns" class="block text-sm font-medium text-gray-700">Columns</label>
					<input
						id="columns"
						type="number"
						bind:value={columns}
						min="1"
						on:change={handleColumnsChange}
						class="w-full rounded-lg border border-gray-300 px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-200 focus:outline-none"
						placeholder="Enter number of columns"
					/>
				</div>

				<div class="space-y-2">
					<label for="columnType" class="block text-sm font-medium text-gray-700">Column Type</label
					>
					<select
						id="columnType"
						bind:value={columnType}
						class="w-full rounded-lg border border-gray-300 px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-200 focus:outline-none"
					>
						<option value="numbers">1, 2, 3...</option>
						<option value="letters">A, B, C...</option>
					</select>
				</div>

				<div class="space-y-2">
					<label for="rowType" class="block text-sm font-medium text-gray-700">Row Type</label>
					<select
						id="rowType"
						bind:value={rowType}
						class="w-full rounded-lg border border-gray-300 px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-200 focus:outline-none"
					>
						<option value="letters">A, B, C...</option>
						<option value="numbers">1, 2, 3...</option>
					</select>
				</div>
			</div>
		</div>

		<div class="relative rounded-xl bg-white p-4 shadow-lg">
			<div class="mb-4 flex items-center justify-between">
				<h2 class="text-lg font-semibold text-gray-800">Seating Layout</h2>
				<div class="flex items-center gap-2">
					<button
						on:click={() => handleZoom('out')}
						class="rounded-lg p-2 text-gray-600 hover:bg-gray-100"
						title="Zoom Out"
					>
						<ZoomOut size={20} />
					</button>
					<span class="min-w-[4rem] text-center text-sm text-gray-600">
						{Math.round(scale * 100)}%
					</span>
					<button
						on:click={() => handleZoom('in')}
						class="rounded-lg p-2 text-gray-600 hover:bg-gray-100"
						title="Zoom In"
					>
						<ZoomIn size={20} />
					</button>
				</div>
			</div>
			<div class="table-wrapper">
				<div
					class="max-h-[60vh] overflow-auto rounded-lg border border-gray-200"
					on:touchstart={handleTouchStart}
					on:touchmove={handleTouchMove}
					on:wheel={handleWheel}
				>
					<div class="min-w-full origin-top-left" style="transform: scale({scale})">
						<table class="w-full border-collapse">
							<thead>
								<tr>
									<th class="sticky top-0 left-0 z-30 bg-blue-50 p-3"></th>
									{#each columnLabels as label}
										<th
											class="sticky top-0 z-20 bg-blue-50 p-3 text-sm font-semibold text-gray-700"
										>
											{label}
										</th>
									{/each}
								</tr>
							</thead>
							<tbody>
								{#each tableData as row, rowIndex}
									<tr>
										<td
											class="sticky left-0 z-10 bg-blue-50 p-3 text-sm font-semibold text-gray-700"
										>
											{rowLabels[rowIndex]}
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
			</div>

			<p class="mt-4 text-center text-sm text-gray-500">
				Use Ctrl + Mouse Wheel or pinch gesture to zoom
			</p>
		</div>
	</div>
</div>

<style>
	/* Ensure proper z-index stacking for sticky elements */
	:global(thead) {
		position: sticky;
		top: 0;
		z-index: 20;
	}

	:global(td:first-child) {
		position: sticky;
		left: 0;
		z-index: 10;
	}

	:global(th:first-child) {
		position: sticky;
		left: 0;
		z-index: 30;
	}
</style>
