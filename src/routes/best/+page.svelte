<script lang="ts">
	import panzoom from 'panzoom';
	import { onMount } from 'svelte';

	type SeatConfig = {
		rows: number;
		seatsPerRow: number;
		rowStartChar: string;
		seatStartNum: number;
		rowOrder: 'up' | 'down';
		seatOrder: 'left' | 'right';
		rowLabel: 'Left Side' | 'Right Side' | 'No Label' | 'Show All';
	};

	type Section = {
		seatConfig: SeatConfig;
		seats: boolean[][];
	};

	type LabelPosition = 'top' | 'bottom' | 'left' | 'right';

	let seatContainer: HTMLElement;
	let ticketQuantity = 1000;
	let showWarning = false;
	let frontLabel = 'STAGE';
	let labelPosition: LabelPosition = 'top';

	let section: Section = {
		seatConfig: {
			rows: 3,
			seatsPerRow: 3,
			rowStartChar: 'A',
			seatStartNum: 1,
			rowOrder: 'down',
			seatOrder: 'left',
			rowLabel: 'Show All'
		},
		seats: Array(3)
			.fill(null)
			.map(() => Array(3).fill(false))
	};

	const generateRowLabel = (startChar: string, rowIndex: number) => {
		const alphabetLength = 26;
		const startCharCode = startChar.toUpperCase().charCodeAt(0);
		const totalOffset = rowIndex;

		const cycleNumber = Math.floor(totalOffset / alphabetLength);
		const letterOffset = totalOffset % alphabetLength;
		const letterCode = ((startCharCode - 65 + letterOffset) % alphabetLength) + 65;

		const letter = String.fromCharCode(letterCode);
		return cycleNumber === 0 ? letter : `${letter}${cycleNumber}`;
	};

	const generateSeatLabel = (seatConfig: SeatConfig, rowIndex: number, seatIndex: number) => {
		const rowLabel = generateRowLabel(
			seatConfig.rowStartChar,
			seatConfig.rowOrder === 'down' ? rowIndex : seatConfig.rows - 1 - rowIndex
		);

		const seatNum =
			seatConfig.seatOrder === 'left'
				? seatConfig.seatStartNum + seatIndex
				: seatConfig.seatStartNum + (seatConfig.seatsPerRow - 1 - seatIndex);

		return `${rowLabel}${seatNum}`;
	};

	const handleSeatClick = (rowIndex: number, seatIndex: number) => {
		const newSeats = [...section.seats];
		newSeats[rowIndex][seatIndex] = !newSeats[rowIndex][seatIndex];
		section.seats = newSeats;
	};

	const checkQuantityExceeded = () => {
		const totalSeats = section.seatConfig.rows * section.seatConfig.seatsPerRow;
		showWarning = totalSeats > ticketQuantity;
		return showWarning;
	};

	const handleQuantityChange = (event: Event) => {
		const input = event.target as HTMLInputElement;
		const newValue = Math.max(1, parseInt(input.value) || 1);
		ticketQuantity = newValue;
		checkQuantityExceeded();
	};

	const handleDimensionChange = (dimension: 'rows' | 'seatsPerRow', event: Event) => {
		const input = event.target as HTMLInputElement;
		const newValue = Math.max(1, parseInt(input.value) || 1);

		if (dimension === 'rows') {
			section.seatConfig.rows = newValue;
		} else {
			section.seatConfig.seatsPerRow = newValue;
		}

		checkQuantityExceeded();
		regenerateSeats();
	};

	const regenerateSeats = () => {
		section.seats = Array(section.seatConfig.rows)
			.fill(null)
			.map(() => Array(section.seatConfig.seatsPerRow).fill(false));
		checkQuantityExceeded();
	};

	const handleSaveLayout = () => {
		console.log('Saving layout:', { frontLabel, labelPosition, section });
	};

	onMount(() => {
		if (seatContainer) {
			const panzoomInstance = panzoom(seatContainer, {
				maxZoom: 2,
				minZoom: 0.1,
				bounds: true,
				boundsPadding: 0.1,
				smoothScroll: true,
				zoomDoubleClickSpeed: 1
			});

			return () => {
				panzoomInstance.dispose();
			};
		}
	});
</script>

<div class="mx-auto max-w-4xl space-y-8 p-6">
	<h1 class="mb-6 text-2xl font-bold">Seat Layout Configuration</h1>

	<div class="space-y-2">
		<label class="block font-medium">Ticket Quantity</label>
		<input
			type="number"
			bind:value={ticketQuantity}
			on:change={handleQuantityChange}
			class="w-full rounded border p-2"
			min="1"
			required
		/>
	</div>

	<div class="space-y-2">
		<label class="block font-medium">Front Label</label>
		<div class="grid grid-cols-2 gap-4">
			<input
				type="text"
				bind:value={frontLabel}
				class="w-full rounded border p-2"
				placeholder="STAGE"
			/>
			<select bind:value={labelPosition} class="w-full rounded border p-2">
				<option value="top">Top</option>
				<option value="bottom">Bottom</option>
				<option value="left">Left Side</option>
				<option value="right">Right Side</option>
			</select>
		</div>
	</div>

	<div class="rounded-lg border p-4">
		<div class="grid grid-cols-2 gap-4">
			<div>
				<label class="block font-medium">Number of rows</label>
				<input
					type="number"
					bind:value={section.seatConfig.rows}
					on:change={(e) => handleDimensionChange('rows', e)}
					class="w-full rounded border p-2"
					min="1"
					required
				/>
			</div>
			<div>
				<label class="block font-medium">Seats per row</label>
				<input
					type="number"
					bind:value={section.seatConfig.seatsPerRow}
					on:change={(e) => handleDimensionChange('seatsPerRow', e)}
					class="w-full rounded border p-2"
					min="1"
					required
				/>
			</div>
		</div>

		<div class="mt-4 grid grid-cols-2 gap-4">
			<div>
				<label class="block font-medium">Rows starts with</label>
				<input
					type="text"
					bind:value={section.seatConfig.rowStartChar}
					class="w-full rounded border p-2"
					maxlength="1"
					on:input={(e) => {
						const value = e.currentTarget.value;
						const validValue = value.replace(/[^A-Za-z]/g, '').toUpperCase();
						if (validValue !== value) {
							e.currentTarget.value = validValue;
							section.seatConfig.rowStartChar = validValue;
						}
					}}
				/>
			</div>
			<div>
				<label class="block font-medium">Row level order</label>
				<div class="flex rounded border">
					<button
						class="flex-1 p-2 text-center"
						class:bg-teal-500={section.seatConfig.rowOrder === 'down'}
						class:text-white={section.seatConfig.rowOrder === 'down'}
						on:click={() => (section.seatConfig.rowOrder = 'down')}
					>
						Down
					</button>
					<button
						class="flex-1 p-2 text-center"
						class:bg-teal-500={section.seatConfig.rowOrder === 'up'}
						class:text-white={section.seatConfig.rowOrder === 'up'}
						on:click={() => (section.seatConfig.rowOrder = 'up')}
					>
						Up
					</button>
				</div>
			</div>
		</div>

		<div class="mt-4 grid grid-cols-2 gap-4">
			<div>
				<label class="block font-medium">Seats starts with</label>
				<input
					type="number"
					bind:value={section.seatConfig.seatStartNum}
					class="w-full rounded border p-2"
					min="1"
					on:input={(e) => {
						const value = e.currentTarget.value;
						if (parseInt(value) < 1) {
							e.currentTarget.value = '1';
							section.seatConfig.seatStartNum = 1;
						}
					}}
				/>
			</div>
			<div>
				<label class="block font-medium">Seat level order</label>
				<div class="flex rounded border">
					<button
						class="flex-1 p-2 text-center"
						class:bg-teal-500={section.seatConfig.seatOrder === 'left'}
						class:text-white={section.seatConfig.seatOrder === 'left'}
						on:click={() => (section.seatConfig.seatOrder = 'left')}
					>
						Left
					</button>
					<button
						class="flex-1 p-2 text-center"
						class:bg-teal-500={section.seatConfig.seatOrder === 'right'}
						class:text-white={section.seatConfig.seatOrder === 'right'}
						on:click={() => (section.seatConfig.seatOrder = 'right')}
					>
						Right
					</button>
				</div>
			</div>
		</div>

		<div class="mt-4">
			<label class="block font-medium">Row label</label>
			<select bind:value={section.seatConfig.rowLabel} class="w-full rounded border p-2">
				<option value="Show All">Show All</option>
				<option value="Left Side">Left Side</option>
				<option value="Right Side">Right Side</option>
				<option value="No Label">No Label</option>
			</select>
		</div>

		<div class="mt-4 flex items-center justify-between">
			<span class="font-medium">Configure individual rows</span>
			<button class="text-teal-500 hover:text-teal-600">Manage</button>
		</div>
	</div>

	{#if showWarning}
		<div class="mt-4 rounded-md bg-yellow-50 p-4">
			<div class="flex">
				<div class="flex-shrink-0">
					<svg class="h-5 w-5 text-yellow-400" viewBox="0 0 20 20" fill="currentColor">
						<path
							fill-rule="evenodd"
							d="M8.257 3.099c.765-1.36 2.722-1.36 3.486 0l5.58 9.92c.75 1.334-.213 2.98-1.742 2.98H4.42c-1.53 0-2.493-1.646-1.743-2.98l5.58-9.92zM11 13a1 1 0 11-2 0 1 1 0 012 0zm-1-8a1 1 0 00-1 1v3a1 1 0 002 0V6a1 1 0 00-1-1z"
							clip-rule="evenodd"
						/>
					</svg>
				</div>
				<div class="ml-3">
					<h3 class="text-sm font-medium text-yellow-800">Configuration exceeds ticket quantity</h3>
					<div class="mt-2 text-sm text-yellow-700">
						<p>
							Current configuration creates {section.seatConfig.rows *
								section.seatConfig.seatsPerRow} seats, but ticket quantity is {ticketQuantity}. This
							may cause issues with ticket allocation.
						</p>
					</div>
				</div>
			</div>
		</div>
	{/if}

	<div class="mt-4 rounded-lg border bg-gray-50 p-4">
		<div class="flex items-center justify-between">
			<div class="space-y-1">
				<h3 class="font-medium text-gray-700">Seat Count</h3>
				<p class="text-sm text-gray-600">
					Rows: {section.seatConfig.rows} × Seats per row: {section.seatConfig.seatsPerRow}
				</p>
				<p class="text-sm text-gray-600">
					Maximum allowed: {ticketQuantity} seats
				</p>
			</div>
			<div class="text-right">
				<span
					class="text-2xl font-bold"
					class:text-red-600={section.seatConfig.rows * section.seatConfig.seatsPerRow >
						ticketQuantity}
					class:text-teal-600={section.seatConfig.rows * section.seatConfig.seatsPerRow <=
						ticketQuantity}
				>
					{section.seatConfig.rows * section.seatConfig.seatsPerRow}
				</span>
				<p class="text-sm text-gray-600">Total Seats</p>
			</div>
		</div>
	</div>

	<div class="relative h-[600px] overflow-hidden rounded border">
		<div class="relative h-full w-full overflow-hidden">
			{#if labelPosition === 'top'}
				<div class="sticky top-0 z-10 mb-4 border-b bg-white p-4 text-center">
					{frontLabel}
				</div>
			{/if}

			<div class="flex h-full">
				{#if labelPosition === 'left'}
					<div
						class="writing-vertical sticky left-0 z-10 flex w-24 items-center justify-center border-r bg-white p-4"
					>
						{frontLabel}
					</div>
				{/if}

				<div bind:this={seatContainer} class="inline-block min-w-min p-4">
					<table class="border-separate border-spacing-1">
						<tbody>
							{#each section.seats as row, rowIndex}
								<tr class="h-8">
									{#if section.seatConfig.rowLabel === 'Left Side' || section.seatConfig.rowLabel === 'Show All'}
										<td class="w-8 pr-2 text-right whitespace-nowrap">
											{generateRowLabel(
												section.seatConfig.rowStartChar,
												section.seatConfig.rowOrder === 'down'
													? rowIndex
													: section.seatConfig.rows - 1 - rowIndex
											)}
										</td>
									{/if}

									<td class="px-2">
										<div class="flex gap-1">
											{#each row as seat, seatIndex}
												<button
													class="flex h-8 w-8 items-center justify-center rounded-md border text-xs hover:bg-gray-200"
													class:bg-green-500={seat}
													class:text-white={seat}
													on:click={() => handleSeatClick(rowIndex, seatIndex)}
												>
													{generateSeatLabel(section.seatConfig, rowIndex, seatIndex)}
												</button>
											{/each}
										</div>
									</td>

									{#if section.seatConfig.rowLabel === 'Right Side' || section.seatConfig.rowLabel === 'Show All'}
										<td class="w-8 pl-2 text-left whitespace-nowrap">
											{generateRowLabel(
												section.seatConfig.rowStartChar,
												section.seatConfig.rowOrder === 'down'
													? rowIndex
													: section.seatConfig.rows - 1 - rowIndex
											)}
										</td>
									{/if}
								</tr>
							{/each}
						</tbody>
					</table>
				</div>

				{#if labelPosition === 'right'}
					<div
						class="writing-vertical sticky right-0 z-10 flex w-24 items-center justify-center border-l bg-white p-4"
					>
						{frontLabel}
					</div>
				{/if}
			</div>

			{#if labelPosition === 'bottom'}
				<div class="sticky bottom-0 z-10 mt-4 border-t bg-white p-4 text-center">
					{frontLabel}
				</div>
			{/if}
		</div>
	</div>

	<button
		class="w-full rounded bg-blue-500 py-2 text-white hover:bg-blue-600"
		on:click={handleSaveLayout}
	>
		Save Layout
	</button>
</div>

<style>
	.writing-vertical {
		writing-mode: vertical-rl;
		text-orientation: mixed;
	}
</style>
