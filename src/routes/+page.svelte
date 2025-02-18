<!-- <script>
    let rows = ['A', 'B', 'C', 'D'];
    let cols = 10;
</script>

<h1 class="text-xl font-bold mb-4">Seat Layout</h1>
<div class="grid grid-cols-[50px_repeat(10,50px)] gap-2 text-center">
    <div></div>
    {#each Array(cols).fill(0).map((_, i) => i + 1) as col}
        <div class="font-bold">{col}</div>
    {/each}

    {#each rows as row}
        <div class="font-bold flex items-center justify-center">{row}</div>
        {#each Array(cols).fill(0).map((_, i) => i + 1) as col}
            <div class="w-12 h-12 bg-gray-300 flex items-center justify-center border border-black cursor-pointer">
                {row.toLowerCase() + col}
            </div>
        {/each}
    {/each}

    <input type="text" />
</div> -->

<!-- <script lang="ts">
    let rows = ['A', 'B', 'C', 'D'];
    let cols = 10;
    let takenSeats = ['a1'];

    const handleClick = (seat: string) => {
        if (!takenSeats.includes(seat)) {
            takenSeats = [...takenSeats, seat];
        }
    };
</script>

<h1 class="text-xl font-bold mb-4">Seat Layout</h1>
<div class="grid grid-cols-[50px_repeat(10,50px)] gap-2 text-center">
    <div></div>
    {#each Array(cols).fill(0).map((_, i) => i + 1) as col}
        <div class="font-bold">{col}</div>
    {/each}

    {#each rows as row}
        <div class="font-bold flex items-center justify-center">{row}</div>
        {#each Array(cols).fill(0).map((_, i) => i + 1) as col}
            {#if takenSeats.includes(row.toLowerCase() + col)}
                <button class="w-12 h-12 bg-red-500 text-white flex items-center justify-center border border-black cursor-not-allowed">
                    {row.toLowerCase() + col}
                </button>
            {:else}
                <button class="w-12 h-12 bg-gray-300 flex items-center justify-center border border-black cursor-pointer" on:click={() => handleClick(row.toLowerCase() + col)}>
                    {row.toLowerCase() + col}
                </button>
            {/if}
        {/each}
    {/each}
</div> -->

<!-- rows + col = a1 -->

<div class="flex flex-col items-center justify-center h-screen">
    <a href="/seat-visualizer">Seat Visualizer</a>
    <a href="/add-ticket">Add ticket</a>
</div>