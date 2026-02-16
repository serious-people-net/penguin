<script lang="ts">
    import { onMount } from "svelte";

    export let bookPages: string[] = [];

    let currentPage = 0;
    let isTransitioning = false;

    // Group pages: first page alone, then pairs, last page alone if odd
    $: displayPages = (() => {
        if (currentPage === 0) return [bookPages[0]];
        if (
            currentPage === bookPages.length - 1 &&
            bookPages.length % 2 === 1
        ) {
            return [bookPages[currentPage]];
        }
        // Show double spreads
        const leftPage = currentPage % 2 === 0 ? currentPage : currentPage - 1;
        return [bookPages[leftPage], bookPages[leftPage + 1]].filter(Boolean);
    })();

    function nextPage() {
        if (isTransitioning) return;

        if (currentPage === 0) {
            currentPage = 1;
        } else if (currentPage < bookPages.length - 1) {
            currentPage += displayPages.length === 2 ? 2 : 1;
            if (currentPage >= bookPages.length)
                currentPage = bookPages.length - 1;
        }

        isTransitioning = true;
        setTimeout(() => (isTransitioning = false), 300);
    }

    function prevPage() {
        if (isTransitioning) return;

        if (currentPage === 1) {
            currentPage = 0;
        } else if (currentPage > 0) {
            currentPage -= 2;
            if (currentPage < 0) currentPage = 0;
        }

        isTransitioning = true;
        setTimeout(() => (isTransitioning = false), 300);
    }
</script>

<div class="book-carousel">
    <div class="carousel-container max-w-6xl mx-auto px-4">
        <div
            class="carousel-pages flex justify-center items-center gap-4 mb-8 min-h-100 md:min-h-150"
        >
            {#each displayPages as page, index}
                <div
                    class="page-image flex-1 max-w-125"
                    style="transition: opacity 0.3s ease-in-out; opacity: {isTransitioning
                        ? 0.5
                        : 1};"
                >
                    <img
                        src={page}
                        alt="Book page {currentPage + index + 1}"
                        class="w-full h-auto shadow-lg"
                    />
                </div>
            {/each}
        </div>

        <div class="carousel-controls flex justify-center items-center gap-4">
            <button
                on:click={prevPage}
                disabled={currentPage === 0}
                class="px-6 py-3 bg-[#237F8D] text-white rounded disabled:opacity-30 disabled:cursor-not-allowed hover:bg-[#1a6070] transition-colors"
            >
                Previous
            </button>

            <span class="text-gray-600">
                Page {currentPage + 1} of {bookPages.length}
            </span>

            <button
                on:click={nextPage}
                disabled={currentPage >= bookPages.length - 1}
                class="px-6 py-3 bg-[#237F8D] text-white rounded disabled:opacity-30 disabled:cursor-not-allowed hover:bg-[#1a6070] transition-colors"
            >
                Next
            </button>
        </div>
    </div>
</div>

<style>
    .book-carousel {
        padding: 3rem 0;
    }

    .page-image img {
        border-radius: 4px;
    }

    button {
        font-family: "powell", sans-serif;
        font-weight: 400;
        min-width: 100px;
    }
</style>
