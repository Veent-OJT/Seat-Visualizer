<script lang="ts">
    import panzoom from 'panzoom';
    import { onMount } from 'svelte';

    let seatContainer: any;

    let seats = [
        { seatName: "a1", paid: true, registrantsInfo: null },
        { seatName: "a2", paid: false, registrantsInfo: null },
        { seatName: "a3", paid: false, registrantsInfo: null },
        { seatName: "a4", paid: true, registrantsInfo: null },
        { seatName: "b1", paid: true, registrantsInfo: null },
        { seatName: "b2", paid: false, registrantsInfo: null },
        { seatName: "b3", paid: true, registrantsInfo: null },
        { seatName: "c1", paid: false, registrantsInfo: null },
        { seatName: "c2", paid: true, registrantsInfo: null },
        { seatName: "c3", paid: true, registrantsInfo: null },
        { seatName: "aa1", paid: true, registrantsInfo: null },
        { seatName: "bs2", paid: true, registrantsInfo: null }
    ];

    function getRowPrefix(seatName: string): string {
        let match = seatName.match(/^[a-zA-Z]+/);
        return match ? match[0] : seatName;
    }

    let groupedSeats = seats.reduce((acc, seat) => {
        let rowKey = getRowPrefix(seat.seatName);
        if (!acc[rowKey]) acc[rowKey] = [];
        acc[rowKey].push(seat);
        return acc;
    }, {} as Record<string, typeof seats>);

    onMount(() => {
        const panZoomInstance = panzoom(seatContainer, {
            maxZoom: 3,
            minZoom: 0.5,
            zoomDoubleClickSpeed: 1,
        });

        return () => panZoomInstance.dispose();
    });
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4 md:p-8">
    <div class="max-w-4xl mx-auto">
        <h1 class="text-3xl md:text-4xl font-bold mb-8 text-center text-gray-800">
            Seat Layout
        </h1>
        <div bind:this={seatContainer} class="zoom-container">
            <div class="bg-white rounded-2xl shadow-xl p-6 md:p-8 backdrop-blur-sm bg-opacity-90">
                <div class="grid gap-6">
                    {#each Object.entries(groupedSeats) as [row, seats]}
                        <div class="flex flex-wrap items-center gap-4">
                            <div class="flex items-center justify-center w-10 h-10 rounded-full bg-indigo-600 text-white font-bold">
                                {row.toUpperCase()}
                            </div>
                            <div class="flex flex-wrap gap-3">
                                {#each seats as seat}
                                    <button 
                                        class="relative group w-14 h-14 md:w-16 md:h-16 flex items-center justify-center rounded-lg transition-all duration-300 transform hover:scale-105
                                        {seat.paid ? 
                                            'bg-gradient-to-br from-red-500 to-red-600 text-white cursor-not-allowed' : 
                                            'bg-gradient-to-br from-emerald-400 to-emerald-500 text-white cursor-pointer hover:from-emerald-500 hover:to-emerald-600'}"
                                        disabled={seat.paid}
                                        on:click={() => {
                                            alert("You've selected seat " + seat.seatName);
                                        }}
                                    >
                                        <span class="font-medium text-sm md:text-base">
                                            {seat.seatName}
                                        </span>
                                        {#if seat.paid}
                                            <div class="absolute -top-2 -right-2 w-4 h-4 bg-red-500 rounded-full border-2 border-white"></div>
                                        {/if}
                                        <div class="absolute inset-0 rounded-lg opacity-0 group-hover:opacity-100 transition-opacity duration-300 bg-black bg-opacity-10"></div>
                                    </button>
                                {/each}
                            </div>
                        </div>
                    {/each}
                </div>
            </div>
        </div>

        <div class="mt-8 flex flex-wrap gap-4 justify-center items-center text-sm text-gray-600">
            <div class="flex items-center gap-2">
                <div class="w-4 h-4 rounded-full bg-gradient-to-br from-emerald-400 to-emerald-500"></div>
                <span>Available</span>
            </div>
            <div class="flex items-center gap-2">
                <div class="w-4 h-4 rounded-full bg-gradient-to-br from-red-500 to-red-600"></div>
                <span>Occupied</span>
            </div>
        </div>
    </div>
</div>

<style>
    .zoom-container {
        width: 100%;
        max-height: 500px;
        overflow: hidden;
        border: 1px solid #ddd;
        touch-action: none;
    }
</style>
