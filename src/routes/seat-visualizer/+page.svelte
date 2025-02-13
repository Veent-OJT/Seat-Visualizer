<script lang="ts">
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
</script>

<h1 class="text-2xl font-bold mb-6 text-center">Seat Layout</h1>

<div class="border border-gray-400 p-4 rounded-lg shadow-lg bg-white">
    {#each Object.entries(groupedSeats) as [row, seats]}
        <div class="flex gap-2 mb-2">
            <div class="font-bold text-lg">{row.toUpperCase()}</div> 
            {#each seats as seat}
                <button 
                    class="w-12 h-12 flex items-center justify-center border border-black rounded-md shadow-md 
                    {seat.paid ? 'bg-red-500 text-white cursor-not-allowed' : 'bg-gray-400 hover:bg-green-500 text-white cursor-pointer transition-all'}">
                    {seat.seatName}
                </button>
            {/each}
        </div>
    {/each}
</div>
