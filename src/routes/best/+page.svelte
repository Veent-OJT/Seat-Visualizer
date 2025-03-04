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
		seats: SeatStatus[][]; // Changed from boolean[][] to SeatStatus[][]
	};

	type LabelPosition = 'top' | 'bottom' | 'left' | 'right';
	// Define seat status type
	type SeatStatus = 'available' | 'unavailable' | 'sold';

	// Add a type for custom seat names
	type CustomSeatNames = {
		[key: string]: string;
	};

	// Define type for selected seat
	type SelectedSeat = { rowIndex: number; seatIndex: number };

	let seatContainer: HTMLElement;
	let ticketQuantity = 1000;
	let showWarning = false;
	let frontLabel = 'STAGE';
	let labelPosition: LabelPosition = 'top';
	let section: Section = {
		seatConfig: {
			rows: 0,
			seatsPerRow: 0,
			rowStartChar: 'A',
			seatStartNum: 1,
			rowOrder: 'down',
			seatOrder: 'left',
			rowLabel: 'Show All'
		},
		seats: []
	};

	// Add state to track the selected seat for single selection mode
	let selectedSeat: { rowIndex: number; seatIndex: number } | null = null;
	let originalSeatName = '';
	let seatName = '';

	// Add state for multiple selection mode
	let multipleSeatSelection = false;
	let selectedSeats: { rowIndex: number; seatIndex: number }[] = [];

	// Store custom seat names
	let customSeatNames: CustomSeatNames = {};

	const handleToggleChange = () => {
		multipleSeatSelection = !multipleSeatSelection;

		// Clear selections when switching modes
		if (multipleSeatSelection) {
			// Switching to multiple selection mode, clear single selection

			selectedSeat = null;
		} else {
			selectedSeats = []; // Clear multiple selections
		}
	};

	const handleStatusChange = (status: SeatStatus) => {
		if (!multipleSeatSelection && selectedSeat) {
			// Single seat mode
			section.seats[selectedSeat.rowIndex][selectedSeat.seatIndex] = status;
			selectedSeat = null; // Clear selection after changing status
		} else if (multipleSeatSelection && selectedSeats.length > 0) {
			// Multiple seats mode
			selectedSeats.forEach((seat) => {
				section.seats[seat.rowIndex][seat.seatIndex] = status;
			});
			selectedSeats = []; // Clear selections after changing status
		}
	};

	// Update handleRenameSave
	const handleRenameSave = () => {
		if (!selectedSeat) return;

		// Extract the number part from the original seat name
		const numberPart = originalSeatName.replace(/[^0-9]/g, '');

		// Combine the new row name with the original number part
		const newSeatName = `${seatName}${numberPart}`;

		console.log('Seat renamed from:', originalSeatName, 'to:', newSeatName);

		// Store the custom name mapping
		customSeatNames[originalSeatName] = newSeatName;

		// Clear the selection
		selectedSeat = null;
	};

	// Add cancel handler
	const handleRenameCancel = () => {
		selectedSeat = null;
	};

	// Check if a seat is selected in multiple selection mode
	const isSeatSelected = (rowIndex: number, seatIndex: number) => {
		return selectedSeats.some((seat) => seat.rowIndex === rowIndex && seat.seatIndex === seatIndex);
	};

	const generateRowLabel = (startChar: string, rowIndex: number) => {
		const alphabetLength = 26;
		const startCharCode = startChar.toUpperCase().charCodeAt(0);
		const totalOffset = rowIndex;

		// Calculate how many complete alphabet cycles we've gone through
		const cycleNumber = Math.floor(totalOffset / alphabetLength);

		// Calculate which letter in the current cycle
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

		// Format seat number with leading zero for single digits
		const formattedSeatNum = seatNum < 10 ? `0${seatNum}` : `${seatNum}`;

		// Generate the standard seat label
		const standardLabel = `${rowLabel}${formattedSeatNum}`;

		// Return custom name if it exists, otherwise return standard label
		return customSeatNames[standardLabel] || standardLabel;
	};

	// Modify the handleSeatClick function to handle both selection modes
	const handleSeatClick = (rowIndex: number, seatIndex: number) => {
		if (multipleSeatSelection) {
			// Multiple selection mode
			const index = selectedSeats.findIndex(
				(seat) => seat.rowIndex === rowIndex && seat.seatIndex === seatIndex
			);

			if (index !== -1) {
				// If seat is already selected, remove it
				selectedSeats = [...selectedSeats.slice(0, index), ...selectedSeats.slice(index + 1)];
			} else {
				// Add seat to selection
				selectedSeats = [...selectedSeats, { rowIndex, seatIndex }];
			}
		} else {
			// Single selection mode
			// If seat is already selected, toggle selection off
			if (
				selectedSeat &&
				selectedSeat.rowIndex === rowIndex &&
				selectedSeat.seatIndex === seatIndex
			) {
				selectedSeat = null;
				return;
			}

			// Select the seat
			selectedSeat = { rowIndex, seatIndex };

			// Generate the standard seat label (without custom name)
			const standardLabel = generateStandardSeatLabel(section.seatConfig, rowIndex, seatIndex);

			// Set the original seat name
			originalSeatName = standardLabel;

			// Extract only the row part (letters) from the seat name
			const rowPart = originalSeatName.replace(/[0-9]/g, '');

			// Initialize the rename field with the row part
			seatName = rowPart;
		}
	};

	// Helper function to generate standard seat label without custom name lookup
	const generateStandardSeatLabel = (
		seatConfig: SeatConfig,
		rowIndex: number,
		seatIndex: number
	) => {
		const rowLabel = generateRowLabel(
			seatConfig.rowStartChar,
			seatConfig.rowOrder === 'down' ? rowIndex : seatConfig.rows - 1 - rowIndex
		);

		const seatNum =
			seatConfig.seatOrder === 'left'
				? seatConfig.seatStartNum + seatIndex
				: seatConfig.seatStartNum + (seatConfig.seatsPerRow - 1 - seatIndex);

		// Format seat number with leading zero for single digits
		const formattedSeatNum = seatNum < 10 ? `0${seatNum}` : `${seatNum}`;

		// No delimiter, just concatenate the row label and formatted seat number
		return `${rowLabel}${formattedSeatNum}`;
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
		// Initialize with 'available' status instead of boolean false
		section.seats = Array(section.seatConfig.rows)
			.fill(null)
			.map(() => Array(section.seatConfig.seatsPerRow).fill('available'));

		checkQuantityExceeded();
	};

	const handleSaveLayout = () => {
		// Create a structured output object with complete configuration
		const layoutData = {
			config: {
				frontLabel,
				labelPosition,
				ticketQuantity,
				seatConfig: { ...section.seatConfig }
			},
			customSeatNames, // Add custom seat names to the saved layout
			seats: {},
			summary: {
				totalSeats: section.seatConfig.rows * section.seatConfig.seatsPerRow,
				availableSeats: section.seats.flat().filter((status) => status === 'available').length,
				unavailableSeats: section.seats.flat().filter((status) => status === 'unavailable').length,
				soldSeats: section.seats.flat().filter((status) => status === 'sold').length
			}
		};

		// Build the detailed seat data
		section.seats.forEach((row, rowIndex) => {
			row.forEach((seatStatus, seatIndex) => {
				const standardLabel = generateStandardSeatLabel(section.seatConfig, rowIndex, seatIndex);
				const displayLabel = customSeatNames[standardLabel] || standardLabel;

				layoutData.seats[standardLabel] = {
					displayName: displayLabel,
					status: seatStatus
				};
			});
		});

		console.log('Saving layout:', layoutData);

		// Create a JSON string with pretty formatting (2-space indentation)
		const jsonString = JSON.stringify(layoutData, null, 2);

		// Create a blob with the JSON data
		const blob = new Blob([jsonString], { type: 'application/json' });

		// Create a download URL
		const url = URL.createObjectURL(blob);

		// Create a temporary anchor element to trigger download
		const a = document.createElement('a');
		a.href = url;
		a.download = `seat-layout-${new Date().toISOString().split('T')[0]}.json`;

		// Trigger the download
		document.body.appendChild(a);
		a.click();

		// Clean up
		setTimeout(() => {
			document.body.removeChild(a);
			URL.revokeObjectURL(url);
		}, 0);
	};

	onMount(() => {
		// Initialize with some default values for better UX
		section.seatConfig.rows = 5;
		section.seatConfig.seatsPerRow = 10;
		regenerateSeats();

		if (seatContainer) {
			const panzoomInstance = panzoom(seatContainer, {
				maxZoom: 2,
				minZoom: 0.1,
				bounds: true,
				boundsPadding: 0.1,
				smoothScroll: true,
				zoomDoubleClickSpeed: 1,
				zoomSpeed: 0.1
			});

			return () => {
				panzoomInstance.dispose();
			};
		}
	});
</script>

<div class="mx-auto max-w-7xl p-6">
	<h1 class="mb-6 text-2xl font-bold">Seat Layout Configuration</h1>

	<div class="flex gap-8">
		<!-- Left Column -->
		<div class="w-[400px] flex-shrink-0 space-y-6">
			<div class="space-y-2">
				<label for="ticket-quantity" class="block font-medium">Ticket Quantity</label>

				<input
					type="number"
					bind:value={ticketQuantity}
					on:change={handleQuantityChange}
					class="w-full rounded border p-2"
					min="1"
					required
				/>
			</div>

			<div class="space-y-3 rounded-lg border bg-white p-4">
				<div class="col-span-2 space-y-2">
					<label for="front-label" class="block font-medium">Front Label</label>

					<div class="flex gap-4">
						<input
							type="text"
							bind:value={frontLabel}
							class="flex-1 rounded border p-2"
							placeholder="STAGE"
						/>

						<select bind:value={labelPosition} class="flex-1 rounded border p-2">
							<option value="top">Top</option>
							<option value="bottom">Bottom</option>
							<option value="left">Left Side</option>
							<option value="right">Right Side</option>
						</select>
					</div>
				</div>
				<div class="grid grid-cols-2 gap-4">
					<div>
						<label for="number-of-rows" class="block font-medium">Number of rows</label>

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
						<label for="seats-per-row" class="block font-medium">Seats per row</label>

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
						<label for="rows-starts-with" class="block font-medium">Rows starts with</label>

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
						<label for="row-level-orders" class="block font-medium">Row level order</label>

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
						<label for="seats-starts-with" class="block font-medium">Seats starts with</label>

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
						<label for="seat-level-order" class="block font-medium">Seat level order</label>

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
					<label for="row-label" class="block font-medium">Row label</label>

					<select bind:value={section.seatConfig.rowLabel} class="w-full rounded border p-2">
						<option value="Show All">Show All</option>
						<option value="Left Side">Left Side</option>
						<option value="Right Side">Right Side</option>
						<option value="No Label">No Label</option>
					</select>
				</div>
			</div>

			<!-- Rename Seat UI - Only show when a seat is selected in single selection mode -->
			{#if selectedSeat && !multipleSeatSelection}
				<div class="space-y-4 rounded-lg border bg-white p-4">
					<h2 class="text-lg font-semibold">Rename Seat: {originalSeatName}</h2>
					<div class="space-y-2">
						<label for="seat-name" class="block font-medium">New Prefix</label>
						<input
							type="text"
							bind:value={seatName}
							class="w-full rounded border p-2"
							placeholder="VIP"
						/>
						<p class="text-xs text-gray-500">
							Original seat: {originalSeatName} → New: {seatName}{originalSeatName.replace(
								/[^0-9]/g,
								''
							)}
						</p>
					</div>
					<div class="flex justify-end gap-2">
						<button
							class="rounded bg-[#DF4D60] px-4 py-2 text-white hover:bg-red-400"
							on:click={handleRenameSave}
						>
							Save
						</button>
						<button
							class="rounded bg-gray-200 px-4 py-2 text-gray-700 hover:bg-gray-300"
							on:click={handleRenameCancel}
						>
							Cancel
						</button>
					</div>
				</div>
			{/if}

			<button
				class="w-full rounded bg-[#DF4D60] py-2 text-white hover:bg-red-400"
				on:click={handleSaveLayout}
			>
				Save Layout
			</button>
		</div>

		<!-- Right Column -->
		<div class="min-w-0 flex-1 space-y-6">
			{#if showWarning}
				<div class="rounded-md bg-yellow-50 p-4">
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
							<h3 class="text-sm font-medium text-yellow-800">
								Configuration exceeds ticket quantity
							</h3>

							<div class="mt-2 text-sm text-yellow-700">
								<p>
									Current configuration creates {section.seatConfig.rows *
										section.seatConfig.seatsPerRow} seats, but ticket quantity is {ticketQuantity}.
									This may cause issues with ticket allocation.
								</p>
							</div>
						</div>
					</div>
				</div>
			{/if}

			<div class="relative h-[600px] overflow-hidden rounded border">
				<div class="absolute inset-0 overflow-auto bg-gray-100 p-8">
					{#if labelPosition === 'top'}
						<div class="sticky top-0 z-10 mb-4 border-b bg-rose-400 p-4 text-center text-white">
							{frontLabel}
						</div>
					{/if}

					<div class="flex h-full">
						{#if labelPosition === 'left'}
							<div
								class="writing-vertical sticky left-0 z-10 flex w-24 items-center justify-center border-r bg-rose-400 p-4 text-white"
							>
								{frontLabel}
							</div>
						{/if}

						<div class="relative flex-1 rounded-lg border-2 border-gray-300 bg-white p-8">
							<div bind:this={seatContainer} class="max-w-full overflow-auto">
								<table class="border-separate border-spacing-1">
									<tbody>
										{#each section.seats as row, rowIndex}
											<tr class="h-8">
												{#if section.seatConfig.rowLabel === 'Left Side' || section.seatConfig.rowLabel === 'Show All'}
													<td class="sticky left-0 z-10 w-8 bg-white pr-2 text-right font-medium">
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
														{#each row as seatStatus, seatIndex}
															<button
																class="flex h-8 w-8 items-center justify-center rounded-md border text-xs transition-colors hover:bg-gray-100"
																class:bg-green-500={seatStatus === 'available'}
																class:bg-gray-300={seatStatus === 'unavailable'}
																class:bg-red-500={seatStatus === 'sold'}
																class:text-white={true}
																class:ring-2={multipleSeatSelection
																	? isSeatSelected(rowIndex, seatIndex)
																	: selectedSeat &&
																		selectedSeat.rowIndex === rowIndex &&
																		selectedSeat.seatIndex === seatIndex}
																class:ring-blue-500={true}
																on:click={() => handleSeatClick(rowIndex, seatIndex)}
															>
																{generateSeatLabel(section.seatConfig, rowIndex, seatIndex)}
															</button>
														{/each}
													</div>
												</td>

												{#if section.seatConfig.rowLabel === 'Right Side' || section.seatConfig.rowLabel === 'Show All'}
													<td class="sticky right-0 z-10 w-8 bg-white pl-2 text-left font-medium">
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
						</div>

						{#if labelPosition === 'right'}
							<div
								class="writing-vertical sticky right-0 z-10 flex w-24 items-center justify-center border-l bg-rose-400 p-4 text-white"
							>
								{frontLabel}
							</div>
						{/if}
					</div>

					{#if labelPosition === 'bottom'}
						<div class="sticky bottom-0 z-10 mt-4 border-t bg-rose-400 p-4 text-center text-white">
							{frontLabel}
						</div>
					{/if}
				</div>
			</div>

			<!-- Multiple Seat Selection and Status Controls -->
			<div class="flex items-center justify-between rounded-lg border bg-white p-4">
				<div class="flex items-center gap-4">
					<label class="flex items-center">
						<span class="mr-2 font-medium">Multiple Seat Selection</span>
						<input
							type="checkbox"
							bind:checked={multipleSeatSelection}
							on:change={handleToggleChange}
							class="toggle-checkbox"
						/>
					</label>

					<!-- Show selection count in multiple selection mode -->
					{#if multipleSeatSelection && selectedSeats.length > 0}
						<span class="ml-2 rounded-md bg-blue-100 px-2 py-1 text-sm font-medium text-blue-700">
							{selectedSeats.length} seats selected
						</span>
					{/if}
				</div>

				<div class="flex gap-2">
					<button
						class="rounded bg-gray-300 px-4 py-2 text-gray-700 transition hover:bg-gray-400"
						on:click={() => handleStatusChange('unavailable')}
						disabled={(!multipleSeatSelection && !selectedSeat) ||
							(multipleSeatSelection && selectedSeats.length === 0)}
					>
						Unavailable
					</button>
					<button
						class="rounded bg-green-500 px-4 py-2 text-white transition hover:bg-green-600"
						on:click={() => handleStatusChange('available')}
						disabled={(!multipleSeatSelection && !selectedSeat) ||
							(multipleSeatSelection && selectedSeats.length === 0)}
					>
						Available
					</button>
					<button
						class="rounded bg-red-500 px-4 py-2 text-white transition hover:bg-red-600"
						on:click={() => handleStatusChange('sold')}
						disabled={(!multipleSeatSelection && !selectedSeat) ||
							(multipleSeatSelection && selectedSeats.length === 0)}
					>
						Sold
					</button>
				</div>
			</div>

			<div class="rounded-lg border bg-white p-4">
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
		</div>
	</div>
</div>

<style>
	.writing-vertical {
		writing-mode: vertical-rl;
		text-orientation: mixed;
	}

	.toggle-checkbox {
		appearance: none;
		width: 40px;
		height: 20px;
		background-color: #ccc;
		border-radius: 10px;
		position: relative;
		outline: none;
		cursor: pointer;
		transition: background-color 0.2s;
	}

	.toggle-checkbox:checked {
		background-color: #4caf50;
	}

	.toggle-checkbox:before {
		content: '';
		position: absolute;
		width: 18px;
		height: 18px;
		background-color: white;
		border-radius: 50%;
		top: 1px;
		left: 1px;
		transition: transform 0.2s;
	}

	.toggle-checkbox:checked:before {
		transform: translateX(20px);
	}

	/* Add disabled button styling */
	button:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}
</style>
