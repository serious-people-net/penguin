<script lang="ts">
    import { onMount } from "svelte";

    export let bookPages: string[] = [];

    // Build views: cover alone, then double spreads, back cover alone
    // Pages: [0] = cover, [1,2], [3,4], ... [29,30], [31] = back cover
    $: views = (() => {
        const v: string[][] = [];
        if (bookPages.length === 0) return v;
        // Cover (page 1)
        v.push([bookPages[0]]);
        // Double spreads from page 2 onwards
        for (let i = 1; i < bookPages.length - 1; i += 2) {
            const spread = [bookPages[i]];
            if (i + 1 < bookPages.length) spread.push(bookPages[i + 1]);
            v.push(spread);
        }
        // Back cover if the last page wasn't already included in a spread
        if (bookPages.length > 1 && bookPages.length % 2 === 0) {
            v.push([bookPages[bookPages.length - 1]]);
        }
        return v;
    })();

    let currentView = 0;

    function next() {
        if (currentView < views.length - 1) currentView++;
    }

    function prev() {
        if (currentView > 0) currentView--;
    }

    function handleKeydown(e: KeyboardEvent) {
        if (e.key === "ArrowRight") next();
        if (e.key === "ArrowLeft") prev();
    }

    $: isSinglePage = views[currentView]?.length === 1;

    // Page number display
    $: pageLabel = (() => {
        if (currentView === 0) return "Cover";
        if (currentView === views.length - 1 && views[currentView].length === 1)
            return "Back Cover";
        const startPage = 1 + (currentView - 1) * 2 + 1;
        return `Pages ${startPage}–${startPage + 1}`;
    })();
</script>

<svelte:window on:keydown={handleKeydown} />

<div class="reader">
    <div class="reader-viewport" class:single={isSinglePage}>
        {#each views[currentView] as page, i}
            <img
                src={page}
                alt={pageLabel}
                class="reader-page"
                loading="eager"
                decoding="async"
            />
        {/each}
    </div>

    <div class="reader-controls">
        <button
            on:click={prev}
            disabled={currentView === 0}
            aria-label="Previous page"
        >
            <svg
                width="20"
                height="20"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"><path d="M15 18l-6-6 6-6" /></svg
            >
        </button>

        <span class="reader-label">{pageLabel}</span>

        <button
            on:click={next}
            disabled={currentView >= views.length - 1}
            aria-label="Next page"
        >
            <svg
                width="20"
                height="20"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"><path d="M9 18l6-6-6-6" /></svg
            >
        </button>
    </div>
</div>

<style>
    .reader {
        max-width: 960px;
        margin: 0 auto;
        padding: 0 1rem;
    }

    .reader-viewport {
        display: flex;
        justify-content: center;
        align-items: stretch;
        gap: 0;
        margin-bottom: 1.5rem;
        border-radius: 4px;
        overflow: hidden;
        box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
        background: white;
    }

    .reader-viewport.single {
        max-width: 480px;
        margin-left: auto;
        margin-right: auto;
        margin-bottom: 1.5rem;
    }

    .reader-page {
        display: block;
        width: 50%;
        height: auto;
        object-fit: cover;
    }

    .reader-viewport.single .reader-page {
        width: 100%;
    }

    .reader-controls {
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 1.5rem;
    }

    .reader-label {
        font-size: 0.95rem;
        color: #656d70;
        min-width: 120px;
        text-align: center;
    }

    button {
        font-family: "powell", sans-serif;
        display: flex;
        align-items: center;
        justify-content: center;
        width: 44px;
        height: 44px;
        border-radius: 50%;
        border: 2px solid #237f8d;
        background: transparent;
        color: #237f8d;
        cursor: pointer;
        transition: all 0.2s ease;
    }

    button:hover:not(:disabled) {
        background: #237f8d;
        color: white;
    }

    button:disabled {
        opacity: 0.25;
        cursor: not-allowed;
        border-color: #656d70;
        color: #656d70;
    }
</style>
