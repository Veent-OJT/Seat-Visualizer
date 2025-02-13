<script lang="ts">
	interface SeatInfo {
		seatName: string;
		REGISTRANTSINFO: Array<Record<string, never>>;
		PAID: boolean;
	}

	let rowStart = 'E';
	let rowEnd = 'J';
	let columnStart = 10;
	let columnEnd = 50;
	let seatData: SeatInfo[] = [];
	let showResults = false;
	let errors = {
		rowStart: '',
		rowEnd: '',
		columnStart: '',
		columnEnd: ''
	};

	interface TicketForm {
		ticketName: string;
		price: number;
		quantity: number;
		validFrom: string;
		validTo: string;
		labelColor: string;
		activePayment: boolean;
	}

	let ticketForm: TicketForm = {
		ticketName: '',
		price: 0,
		quantity: 1,
		validFrom: new Date().toISOString().split('T')[0],
		validTo: '',
		labelColor: '#D12F2B',
		activePayment: false
	};

	let showToast = false;
	let toastMessage = '';

	const colors = [
		{ value: '#0000FF', name: 'Blue' },
		{ value: '#000000', name: 'Black' },
		{ value: '#FFFFFF', name: 'White' },
		{ value: '#FFD700', name: 'Gold' },
		{ value: '#FF4500', name: 'Orange Red' },
		{ value: '#FF00FF', name: 'Magenta' },
		{ value: '#90EE90', name: 'Light Green' }
	];

	function handleRowChange(value: string, field: 'rowStart' | 'rowEnd') {
		const letterOnly = value.replace(/[^A-Za-z]/g, '').toUpperCase();
		if (field === 'rowStart') {
			rowStart = letterOnly;
		} else {
			rowEnd = letterOnly;
		}

		// Clear error when valid input is provided
		if (letterOnly || !value) {
			errors[field] = '';
		} else {
			errors[field] = 'Please enter letters only';
		}
		errors = errors; // Trigger reactivity
	}

	function handleColumnChange(value: string, field: 'columnStart' | 'columnEnd') {
		const numberOnly = value.replace(/[^0-9]/g, '');
		const numberValue = numberOnly ? parseInt(numberOnly) : 0;

		if (field === 'columnStart') {
			columnStart = numberValue;
		} else {
			columnEnd = numberValue;
		}

		// Clear error when valid input is provided
		if (numberOnly || !value) {
			errors[field] = '';
		} else {
			errors[field] = 'Please enter numbers only';
		}
		errors = errors; // Trigger reactivity
	}

	function showNotification(message: string) {
		toastMessage = message;
		showToast = true;
		setTimeout(() => {
			showToast = false;
		}, 3000);
	}

	async function copyToClipboard() {
		try {
			await navigator.clipboard.writeText(JSON.stringify(seatData, null, 2));
			showNotification('JSON copied to clipboard!');
		} catch (err) {
			console.error('Failed to copy text: ', err);
			showNotification('Failed to copy to clipboard');
		}
	}

	function generateAllSeatData() {
		const allSeats: SeatInfo[] = [];
		const rows = [];

		for (let r = rowStart.charCodeAt(0); r <= rowEnd.charCodeAt(0); r++) {
			rows.push(String.fromCharCode(r));
		}

		const columns = Array.from({ length: columnEnd - columnStart + 1 }, (_, i) => i + columnStart);

		for (const row of rows) {
			for (const col of columns) {
				allSeats.push({
					seatName: `${row}${col}`,
					REGISTRANTSINFO: [{}],
					PAID: Math.random() > 0.5
				});
			}
		}

		seatData = allSeats;
		showResults = true;
	}

	function handleSubmit(e: SubmitEvent) {
		e.preventDefault();

		// Validate before submission
		if (!rowStart || !rowEnd || columnStart <= 0 || columnEnd <= 0) {
			return;
		}

		if (rowStart.charCodeAt(0) > rowEnd.charCodeAt(0)) {
			errors.rowEnd = 'End row must be after start row';
			errors = errors; // Trigger reactivity
			return;
		}

		if (columnStart > columnEnd) {
			errors.columnEnd = 'End column must be greater than start column';
			errors = errors; // Trigger reactivity
			return;
		}

		generateAllSeatData();
	}

	$: hasErrors = Object.values(errors).some((error) => error !== '');
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-900 to-indigo-800 p-6">
	{#if showToast}
		<div class="animate-fade-in-up fixed right-4 bottom-4 z-50">
			<div class="rounded-lg bg-gray-800 px-4 py-2 text-sm text-white shadow-lg">
				{toastMessage}
			</div>
		</div>
	{/if}
	<div class="mx-auto max-w-7xl">
		<div class="rounded-xl bg-white p-8 shadow-2xl">
			<h1 class="mb-8 text-center text-3xl font-bold text-gray-800">Add Ticket</h1>

			<form on:submit={handleSubmit} class="mb-8 space-y-8">
				<!-- Ticket Information -->
				<div class="rounded-lg bg-gray-50 p-6">
					<h2 class="mb-6 text-xl font-semibold text-gray-800">Ticket Information</h2>
					<div class="grid grid-cols-1 gap-6 md:grid-cols-2">
						<div class="space-y-4">
							<div>
								<label class="mb-1 block text-sm font-medium text-gray-600">Ticket Name</label>
								<input
									type="text"
									bind:value={ticketForm.ticketName}
									class="w-full rounded-lg border px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-500"
									placeholder="Enter ticket name"
									required
								/>
							</div>

							<div>
								<label class="mb-1 block text-sm font-medium text-gray-600">Price (PHP)</label>
								<div class="relative">
									<span class="absolute top-2 left-3 text-gray-500">₱</span>
									<input
										type="number"
										bind:value={ticketForm.price}
										min="0"
										step="0.01"
										class="w-full rounded-lg border py-2 pr-4 pl-8 focus:border-blue-500 focus:ring-2 focus:ring-blue-500"
										placeholder="0.00"
										required
									/>
								</div>
							</div>

							<div>
								<label class="mb-1 block text-sm font-medium text-gray-600">Quantity</label>
								<input
									type="number"
									bind:value={ticketForm.quantity}
									min="1"
									class="w-full rounded-lg border px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-500"
									required
								/>
							</div>
						</div>

						<div class="space-y-4">
							<div>
								<label class="mb-1 block text-sm font-medium text-gray-600">Valid From</label>
								<input
									type="date"
									bind:value={ticketForm.validFrom}
									class="w-full rounded-lg border px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-500"
									required
								/>
							</div>

							<div>
								<label class="mb-1 block text-sm font-medium text-gray-600">
									Valid To <span class="text-sm text-gray-400"
										>(Leave blank if always available)</span
									>
								</label>
								<input
									type="date"
									bind:value={ticketForm.validTo}
									class="w-full rounded-lg border px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-500"
								/>
							</div>

							<div>
								<label class="mb-1 block text-sm font-medium text-gray-600">Label Color</label>
								<div class="flex flex-wrap gap-3">
									{#each colors as color}
										<button
											type="button"
											class="relative h-8 w-8 rounded-lg border-2 transition-transform hover:scale-110 focus:outline-none"
											style="background-color: {color.value}; border-color: {ticketForm.labelColor ===
											color.value
												? '#3B82F6'
												: 'transparent'}"
											on:click={() => (ticketForm.labelColor = color.value)}
											title={color.name}
										>
											{#if ticketForm.labelColor === color.value}
												<div class="absolute inset-0 flex items-center justify-center">
													<span class="text-{color.value === '#FFFFFF' ? 'black' : 'white'}">✓</span
													>
												</div>
											{/if}
										</button>
									{/each}
								</div>
							</div>
						</div>
					</div>

					<div class="mt-6 flex items-center justify-between rounded-lg bg-white p-4 shadow-sm">
						<label class="text-sm font-medium text-gray-600">Active Payment</label>
						<button
							type="button"
							class="relative h-6 w-11 rounded-full bg-gray-200 transition-colors duration-200 ease-in-out {ticketForm.activePayment
								? 'bg-green-600'
								: ''}"
							on:click={() => (ticketForm.activePayment = !ticketForm.activePayment)}
						>
							<span
								class="absolute top-0.5 left-0.5 h-5 w-5 transform rounded-full bg-white transition-transform duration-200 ease-in-out {ticketForm.activePayment
									? 'translate-x-5'
									: ''}"
							/>
						</button>
					</div>
				</div>

				<!-- Seat Range Selection -->
				<div class="rounded-lg bg-gray-50 p-6">
					<h2 class="mb-6 text-xl font-semibold text-gray-800">Seat Range</h2>
					<!-- Row Range -->
					<div class="space-y-4">
						<h2 class="text-xl font-semibold text-gray-700">Row Range</h2>
						<div class="grid grid-cols-2 gap-4">
							<div>
								<label class="mb-1 block text-sm font-medium text-gray-600"> Start Row </label>
								<input
									type="text"
									bind:value={rowStart}
									on:input={(e) => handleRowChange(e.currentTarget.value, 'rowStart')}
									class="w-full rounded-lg border px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-500 {errors.rowStart
										? 'border-red-500'
										: ''}"
									maxlength={1}
									required
								/>
								{#if errors.rowStart}
									<p class="mt-1 text-sm text-red-500">{errors.rowStart}</p>
								{/if}
							</div>
							<div>
								<label class="mb-1 block text-sm font-medium text-gray-600"> End Row </label>
								<input
									type="text"
									bind:value={rowEnd}
									on:input={(e) => handleRowChange(e.currentTarget.value, 'rowEnd')}
									class="w-full rounded-lg border px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-500 {errors.rowEnd
										? 'border-red-500'
										: ''}"
									maxlength={1}
									required
								/>
								{#if errors.rowEnd}
									<p class="mt-1 text-sm text-red-500">{errors.rowEnd}</p>
								{/if}
							</div>
						</div>
					</div>

					<!-- Column Range -->
					<div class="space-y-4">
						<h2 class="text-xl font-semibold text-gray-700">Column Range</h2>
						<div class="grid grid-cols-2 gap-4">
							<div>
								<label class="mb-1 block text-sm font-medium text-gray-600"> Start Column </label>
								<input
									type="text"
									bind:value={columnStart}
									on:input={(e) => handleColumnChange(e.currentTarget.value, 'columnStart')}
									class="w-full rounded-lg border px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-500 {errors.columnStart
										? 'border-red-500'
										: ''}"
									required
								/>
								{#if errors.columnStart}
									<p class="mt-1 text-sm text-red-500">{errors.columnStart}</p>
								{/if}
							</div>
							<div>
								<label class="mb-1 block text-sm font-medium text-gray-600"> End Column </label>
								<input
									type="text"
									bind:value={columnEnd}
									on:input={(e) => handleColumnChange(e.currentTarget.value, 'columnEnd')}
									class="w-full rounded-lg border px-4 py-2 focus:border-blue-500 focus:ring-2 focus:ring-blue-500 {errors.columnEnd
										? 'border-red-500'
										: ''}"
									required
								/>
								{#if errors.columnEnd}
									<p class="mt-1 text-sm text-red-500">{errors.columnEnd}</p>
								{/if}
							</div>
						</div>
					</div>
				</div>

				<div class="flex gap-4">
					<button
						type="button"
						class="flex-1 rounded-lg border border-gray-300 px-6 py-2 text-gray-700 transition-colors hover:bg-gray-50"
					>
						Cancel
					</button>
					<button
						type="submit"
						disabled={hasErrors}
						class="flex-1 rounded-lg bg-red-600 px-6 py-2 text-white transition-colors hover:bg-red-700 disabled:bg-gray-400"
					>
						Create
					</button>
				</div>
			</form>
			{#if showResults}
				<div class="rounded-lg bg-gray-50 p-4">
					<button
						on:click={copyToClipboard}
						class="flex items-center gap-2 rounded-lg bg-blue-600 px-4 py-2 text-sm text-white transition-all hover:bg-blue-700 active:scale-95"
					>
						Copy JSON
					</button>
					<pre class="overflow-auto text-sm">
      
      {JSON.stringify(seatData, null, 2)}
      
      </pre>
				</div>
			{/if}
		</div>
	</div>
</div>
