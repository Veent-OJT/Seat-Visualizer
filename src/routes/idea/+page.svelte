<script lang="ts">
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
		id: number;
		seatConfig: SeatConfig;
		seats: boolean[][];
		visible: boolean;
	};

	let frontLabel = 'STAGE';
	let sections: Section[] = [];
	let sectionCounter = 1;

	const createDefaultSeatConfig = (): SeatConfig => ({
		rows: 3,
		seatsPerRow: 3,
		rowStartChar: 'A',
		seatStartNum: 1,
		rowOrder: 'down',
		seatOrder: 'left',
		rowLabel: 'Show All'
	});

	const generateSeats = (config: SeatConfig): boolean[][] => {
		return Array(config.rows)
			.fill(null)
			.map(() => Array(config.seatsPerRow).fill(false));
	};

	const addSection = () => {
		sections = [
			...sections,
			{
				id: sectionCounter,
				seatConfig: createDefaultSeatConfig(),
				seats: generateSeats(createDefaultSeatConfig()),
				visible: true
			}
		];
		sectionCounter++;
	};

	const removeSection = (id: number) => {
		sections = sections.filter((section) => section.id !== id);
	};

	const toggleSectionVisibility = (sectionIndex: number) => {
		sections = sections.map((section, idx) => {
			if (idx === sectionIndex) {
				return { ...section, visible: !section.visible };
			}
			return section;
		});
	};

	const generateSeatLabel = (seatConfig: SeatConfig, rowIndex: number, seatIndex: number) => {
		const startCharCode = seatConfig.rowStartChar.toUpperCase().charCodeAt(0);
		const rowChar = String.fromCharCode(
			seatConfig.rowOrder === 'down'
				? startCharCode + rowIndex
				: startCharCode + (seatConfig.rows - 1 - rowIndex)
		);

		const seatNum =
			seatConfig.seatOrder === 'left'
				? seatConfig.seatStartNum + seatIndex
				: seatConfig.seatStartNum + (seatConfig.seatsPerRow - 1 - seatIndex);

		return `${rowChar}${seatNum}`;
	};

	const handleSeatClick = (sectionIndex: number, rowIndex: number, seatIndex: number) => {
		sections = sections.map((section, idx) => {
			if (idx === sectionIndex) {
				const newSeats = [...section.seats];
				newSeats[rowIndex][seatIndex] = !newSeats[rowIndex][seatIndex];
				return { ...section, seats: newSeats };
			}
			return section;
		});
	};

	const regenerateSeats = (sectionIndex: number) => {
		sections = sections.map((section, idx) => {
			if (idx === sectionIndex) {
				return {
					...section,
					seats: generateSeats(section.seatConfig)
				};
			}
			return section;
		});
	};

	const handleSaveLayout = () => {
		console.log('Saving layout:', { frontLabel, sections });
	};

	onMount(() => {
		if (sections.length === 0) {
			addSection();
		}
	});
</script>

<div class="mx-auto max-w-4xl space-y-8 p-6">
	<h1 class="mb-6 text-2xl font-bold">Seat Layout Configuration</h1>

	<!-- Section Controls -->
	<div class="flex items-center gap-4">
		<span class="font-medium">Section</span>
		<button
			class="rounded-md border px-3 py-1 hover:bg-gray-100"
			on:click={() => removeSection(sections[sections.length - 1].id)}
			disabled={sections.length <= 1}
		>
			-
		</button>
		<span>{sections.length}</span>
		<button class="rounded-md border px-3 py-1 hover:bg-gray-100" on:click={addSection}> + </button>
	</div>

	<!-- Front Label -->
	<div class="space-y-2">
		<label class="block font-medium">Front Label</label>
		<input
			type="text"
			bind:value={frontLabel}
			class="w-full rounded border p-2"
			placeholder="STAGE"
		/>
	</div>

	<!-- Sections -->
	{#each sections as section, sectionIndex}
		<div class="rounded-lg border p-4">
			<div class="flex items-center justify-between p-4">
				<h2 class="text-xl font-semibold">SECTION {section.id}</h2>
				<button
					class="flex items-center gap-2 text-gray-600 hover:text-gray-900"
					on:click={() => toggleSectionVisibility(sectionIndex)}
				>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-6 w-6"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
					>
						{#if section.visible}
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
							/>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"
							/>
						{:else}
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.88 9.88l-3.29-3.29m7.532 7.532l3.29 3.29M3 3l3.59 3.59m0 0A9.953 9.953 0 0112 5c4.478 0 8.268 2.943 9.543 7a10.025 10.025 0 01-4.132 5.411m0 0L21 21"
							/>
						{/if}
					</svg>
					<span>{section.visible ? 'DISPLAY' : 'HIDDEN'}</span>
				</button>
			</div>

			{#if section.visible}
				<div class="grid grid-cols-2 gap-4">
					<div>
						<label class="block font-medium">Number of rows</label>
						<input
							type="number"
							bind:value={section.seatConfig.rows}
							on:change={() => regenerateSeats(sectionIndex)}
							class="w-full rounded border p-2"
							min="1"
						/>
					</div>
					<div>
						<label class="block font-medium">Seats per row</label>
						<input
							type="number"
							bind:value={section.seatConfig.seatsPerRow}
							on:change={() => regenerateSeats(sectionIndex)}
							class="w-full rounded border p-2"
							min="1"
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
					<button class="text-teal-500 hover:text-teal-600"> Manage </button>
				</div>
			{/if}
		</div>
	{/each}

	<!-- Seat Layout Preview -->
	<div class="rounded border p-4">
		<div class="mb-4 border-b p-4 text-center">{frontLabel}</div>
		{#each sections.filter((s) => s.visible) as section, sectionIndex}
			<table class="mx-auto mb-8">
				<tbody>
					{#each section.seats as row, rowIndex}
						<tr class="h-10">
							{#if section.seatConfig.rowLabel === 'Left Side' || section.seatConfig.rowLabel === 'Show All'}
								<td class="w-8 pr-2 text-right">
									{String.fromCharCode(
										section.seatConfig.rowStartChar.toUpperCase().charCodeAt(0) +
											(section.seatConfig.rowOrder === 'down'
												? rowIndex
												: section.seatConfig.rows - 1 - rowIndex)
									)}
								</td>
							{/if}

							<td class="px-2">
								<div class="flex justify-center gap-2">
									{#each row as seat, seatIndex}
										<button
											class="flex h-8 w-8 items-center justify-center rounded-md border text-xs"
											class:bg-green-500={seat}
											class:text-white={seat}
											on:click={() => handleSeatClick(sectionIndex, rowIndex, seatIndex)}
										>
											{generateSeatLabel(section.seatConfig, rowIndex, seatIndex)}
										</button>
									{/each}
								</div>
							</td>

							{#if section.seatConfig.rowLabel === 'Right Side' || section.seatConfig.rowLabel === 'Show All'}
								<td class="w-8 pl-2 text-left">
									{String.fromCharCode(
										section.seatConfig.rowStartChar.toUpperCase().charCodeAt(0) +
											(section.seatConfig.rowOrder === 'down'
												? rowIndex
												: section.seatConfig.rows - 1 - rowIndex)
									)}
								</td>
							{/if}
						</tr>
					{/each}
				</tbody>
			</table>
		{/each}
	</div>

	<button
		class="w-full rounded bg-blue-500 py-2 text-white hover:bg-blue-600"
		on:click={handleSaveLayout}
	>
		Save Layout
	</button>
</div>
