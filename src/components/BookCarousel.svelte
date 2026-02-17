<script lang="ts">
    import { onMount } from "svelte";

    export let bookPages: string[] = [];
    export let pdfUrl: string = "";

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
    let imagesLoaded = false;

    // Preload all images with proper loading
    onMount(() => {
        if (typeof window === 'undefined') return;
        
        const promises = bookPages.map((page) => {
            return new Promise<void>((resolve) => {
                const img = new window.Image();
                img.onload = () => resolve();
                img.onerror = () => resolve(); // Resolve even on error to not block
                img.src = page;
            });
        });
        
        Promise.all(promises).then(() => {
            imagesLoaded = true;
        });
    });

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

<!-- Hidden preload images -->
<div style="display: none;">
    {#each bookPages as page}
        <img src={page} alt="" />
    {/each}
</div>

<div class="reader">
    {#if !imagesLoaded}
        <div class="reader-loading">
            <p>Loading book...</p>
        </div>
    {:else}
        <div class="reader-viewport" class:single={isSinglePage}>
            {#each views[currentView] as page, i}
                <img
                    src={page}
                    alt={pageLabel}
                    class="reader-page"
                    loading="eager"
                    decoding="sync"
                />
            {/each}
        </div>
    {/if}

    {#if imagesLoaded}
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

        {#if pdfUrl}
            <div class="reader-download">
                <a href={pdfUrl} download class="btn-download">
                    <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
                        <polyline points="7 10 12 15 17 10"></polyline>
                        <line x1="12" y1="15" x2="12" y2="3"></line>
                    </svg>
                    Download PDF
                </a>
            </div>
        {/if}
    {/if}
</div>

<style>
    .reader {
        max-width: 960px;
        margin: 0 auto;
        padding: 3rem 1rem;
    }

    .reader-loading {
        text-align: center;
        padding: 4rem 2rem;
        color: #656d70;
        font-size: 1.1rem;
    }

    .reader-viewport {
        display: flex;
        justify-content: center;
        align-items: stretch;
        gap: 0;
        margin-bottom: 2rem;
        border-radius: 4px;
        overflow: hidden;
        box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
        background: white;
    }

    .reader-viewport.single {
        max-width: 480px;
        margin-left: auto;
        margin-right: auto;
        margin-bottom: 2rem;
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

    .reader-download {
        display: flex;
        justify-content: center;
        margin-top: 2rem;
    }

    .btn-download {
        display: inline-flex;
        align-items: center;
        gap: 0.5rem;
        padding: 0.75rem 1.5rem;
        background: #237f8d;
        color: white;
        text-decoration: none;
        border-radius: 4px;
        font-family: "powell", sans-serif;
        font-size: 1rem;
        transition: background 0.2s ease;
    }

    .btn-download:hover {
        background: #1a5f6a;
    }

    .btn-download svg {
        flex-shrink: 0;
    }
</style>
