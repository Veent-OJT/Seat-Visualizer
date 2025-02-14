<script lang="ts">
	import panzoom from 'panzoom';
	import { onMount } from 'svelte';
	import "./style/seat-visualizer.css"
	import seats from "$lib/data/seats.json"

	let seatContainer: HTMLElement;

	let selectedSeat: string | null = null;

	function getRowPrefix(seatName: string): string {
		let match = seatName.match(/^[a-zA-Z]+/);
		return match ? match[0] : seatName;
	}

	function getColumnNumbers(seats: Array<any>) {
		let numbers = new Set<number>();
		seats.forEach(seat => {
			let num = parseInt(seat.seatName.replace(/[^0-9]/g, ''), 10);
			if (!isNaN(num)) numbers.add(num);
		});
		return Array.from(numbers).sort((a, b) => a - b);
	}

	let groupedSeats = seats.reduce(
		(acc, seat) => {
			let rowKey = getRowPrefix(seat.seatName);
			if (!acc[rowKey]) acc[rowKey] = [];
			acc[rowKey].push(seat);
			return acc;
		},
		{} as Record<string, typeof seats>
	);

	let columnNumbers = getColumnNumbers(seats);

	function handleSeatClick(seat: any) {
		if (!seat.paid) {
			selectedSeat = selectedSeat === seat.seatName ? null : seat.seatName;
		}
	}

	onMount(() => {
	const panZoomInstance = panzoom(seatContainer, {
		maxZoom: 5, // Allow zooming up to 5x
		minZoom: 1, // Keep minZoom at 1
		bounds: false, // Allow free panning beyond bounds
		boundsPadding: 0,
		zoomDoubleClickSpeed: 1
	});

	// Allow horizontal panning even when not zoomed
	panZoomInstance.on('transform', () => {
		const transform = panZoomInstance.getTransform();

		// Prevent resetting position when scale is 1
		if (transform.scale <= 1) {
			panZoomInstance.moveTo(transform.x, transform.y);
		}
	});

	return () => panZoomInstance.dispose();
});

</script>

<div class="container">
	<div class="content">
		<h1>Select Seats</h1>

		<!-- Venue Image -->
		<div class="venue-container">
			<div class="venue-image">
				<p>
					VENUE FLOOR PLAN IMAGE
					<br />
					<span>refer to the venue image to know your seat</span>
				</p>
			</div>
		</div>

		<!-- Seat Layout -->
		<div class="seat-layout">
			<div  bind:this={seatContainer} class="grid">
				<!-- Column Numbers -->
				<div class="column-numbers">
					{#each columnNumbers as colNum}
						<div class="column-number">
							{colNum}
						</div>
					{/each}
				</div>

				{#each Object.entries(groupedSeats) as [row, seats]}
					<div class="row">
						<div class="row-label">
							{row.toUpperCase()}
						</div>
						<div class="seats">
							{#each seats as seat}
								<button
									class="seat-button {seat.PAID ? 'taken' : selectedSeat === seat.seatName ? 'selected' : 'available'}"
									disabled={seat.PAID}
									on:click={() => handleSeatClick(seat)}
								>
								<!-- the PAID should be lowercase but I made it uppercase to match jie's data -->
									<span>
										{seat.seatName}
									</span>
								</button>
							{/each}
						</div>
					</div>
				{/each}
			</div>
		</div>

		<!-- Legend and Price -->
		{#if selectedSeat}
			<div class="price-info">
				<div class="seat-details">
					<p>Row {getRowPrefix(selectedSeat).toUpperCase()}</p>
					<p>Seat {selectedSeat.replace(/[^0-9]/g, '')}</p>
				</div>
				<div class="price">
					<span>Total Price</span>
					<span class="amount">$149.99</span>
				</div>
				<button class="reserve-button">
					Reserve Seat
				</button>
			</div>
		{:else}
			<div class="legend">
				<div class="legend-item">
					<div class="legend-color available"></div>
					<span>Available</span>
				</div>
				<div class="legend-item">
					<div class="legend-color taken"></div>
					<span>Taken</span>
				</div>
				<div class="legend-item">
					<div class="legend-color selected"></div>
					<span>Selected</span>
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
	:global(body) {
    margin: 0;
    background-color: #000;
    color: #fff;
    min-height: 100vh;
}
</style>