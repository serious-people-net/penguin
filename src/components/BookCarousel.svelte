<script lang="ts">
    import { onMount } from "svelte";

    export let bookPages: string[] = [];
    export let pdfUrl: string = "";
    // Set to true to hide publishing/copyright pages (pages 02–05, indices 1–4)
    export let skipPublishingPages: boolean = false;

    // Filter pages if needed
    $: filteredPages = skipPublishingPages
        ? [bookPages[0], ...bookPages.slice(5)]
        : bookPages;

    let currentView = 0;
    let imagesLoaded = false;
    let isMobile = false;

    onMount(() => {
        // Detect mobile for single-page layout
        const mq = window.matchMedia("(max-width: 639px)");
        isMobile = mq.matches;
        mq.addEventListener("change", (e) => {
            isMobile = e.matches;
        });

        // Preload all images
        const promises = bookPages.map((page) => {
            return new Promise<void>((resolve) => {
                const img = new window.Image();
                img.onload = () => resolve();
                img.onerror = () => resolve();
                img.src = page;
            });
        });
        Promise.all(promises).then(() => {
            imagesLoaded = true;
        });
    });

    // Build views: single pages on mobile, double spreads on desktop
    $: views = (() => {
        const v: string[][] = [];
        if (filteredPages.length === 0) return v;
        if (isMobile) {
            for (const page of filteredPages) v.push([page]);
            return v;
        }
        // Cover
        v.push([filteredPages[0]]);
        // Double spreads
        for (let i = 1; i < filteredPages.length - 1; i += 2) {
            const spread = [filteredPages[i]];
            if (i + 1 < filteredPages.length) spread.push(filteredPages[i + 1]);
            v.push(spread);
        }
        // Back cover if last page wasn't included in a spread
        if (filteredPages.length > 1 && filteredPages.length % 2 === 0) {
            v.push([filteredPages[filteredPages.length - 1]]);
        }
        return v;
    })();

    function next() {
        if (currentView < views.length - 1) currentView++;
    }

    function prev() {
        if (currentView > 0) currentView--;
    }

    function handleKeydown(e: KeyboardEvent) {
        if (e.key === "ArrowRight") next();
        if (e.key === "ArrowLeft") prev();
        if (e.key === "Escape" && isExpanded) toggleExpand();
    }

    $: isSinglePage = views[currentView]?.length === 1;

    // Page number display
    $: pageLabel = (() => {
        if (isMobile) {
            if (currentView === 0) return "Cover";
            if (currentView === filteredPages.length - 1) return "Back Cover";
            return `Page ${currentView + 1}`;
        }
        if (currentView === 0) return "Cover";
        if (currentView === views.length - 1 && views[currentView].length === 1)
            return "Back Cover";
        const startPage = 1 + (currentView - 1) * 2 + 1;
        return `Pages ${startPage}–${startPage + 1}`;
    })();
    let isExpanded = false;

    function toggleExpand() {
        isExpanded = !isExpanded;
        if (typeof document !== "undefined") {
            document.body.style.overflow = isExpanded ? "hidden" : "";
        }
    }
</script>

<svelte:window on:keydown={handleKeydown} />

<!-- Hidden preload images -->
<div style="display: none;">
    {#each bookPages as page}
        <img src={page} alt="" />
    {/each}
</div>

<div class="reader" class:expanded={isExpanded}>
    {#if isExpanded}
        <button
            class="expand-close"
            on:click={toggleExpand}
            aria-label="Exit fullscreen"
        >
            <svg
                width="24"
                height="24"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
                ><line x1="18" y1="6" x2="6" y2="18" /><line
                    x1="6"
                    y1="6"
                    x2="18"
                    y2="18"
                /></svg
            >
        </button>
    {/if}
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
        {#if isMobile && !isExpanded}
            <p class="rotate-hint">
                <svg
                    width="16"
                    height="16"
                    viewBox="0 0 24 24"
                    fill="none"
                    stroke="currentColor"
                    stroke-width="2"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    ><path d="M23 4v6h-6" /><path d="M1 20v-6h6" /><path
                        d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"
                    /></svg
                >
                Rotate for a better view
            </p>
        {/if}
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

            <button
                on:click={toggleExpand}
                class="expand-btn"
                aria-label={isExpanded ? "Exit fullscreen" : "View fullscreen"}
            >
                {#if isExpanded}
                    <svg
                        width="20"
                        height="20"
                        viewBox="0 0 24 24"
                        fill="none"
                        stroke="currentColor"
                        stroke-width="2"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        ><polyline points="4 14 10 14 10 20" /><polyline
                            points="20 10 14 10 14 4"
                        /><line x1="14" y1="10" x2="21" y2="3" /><line
                            x1="3"
                            y1="21"
                            x2="10"
                            y2="14"
                        /></svg
                    >
                {:else}
                    <svg
                        width="20"
                        height="20"
                        viewBox="0 0 24 24"
                        fill="none"
                        stroke="currentColor"
                        stroke-width="2"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        ><polyline points="15 3 21 3 21 9" /><polyline
                            points="9 21 3 21 3 15"
                        /><line x1="21" y1="3" x2="14" y2="10" /><line
                            x1="3"
                            y1="21"
                            x2="10"
                            y2="14"
                        /></svg
                    >
                {/if}
            </button>
        </div>

        {#if pdfUrl}
            <div class="reader-download">
                <a href={pdfUrl} download class="btn-download">
                    <svg
                        width="20"
                        height="20"
                        viewBox="0 0 24 24"
                        fill="none"
                        stroke="currentColor"
                        stroke-width="2"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                    >
                        <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"
                        ></path>
                        <polyline points="7 10 12 15 17 10"></polyline>
                        <line x1="12" y1="15" x2="12" y2="3"></line>
                    </svg>
                    Download
                </a>
            </div>
        {/if}
    {/if}
</div>

<style>
    .reader {
        max-width: 1200px;
        margin: 0 auto;
        padding: 3rem 1rem;
    }

    .reader.expanded {
        position: fixed;
        inset: 0;
        z-index: 10000;
        max-width: none;
        background: rgba(0, 0, 0, 0.95);
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        padding: 2rem;
    }

    .reader-loading {
        text-align: center;
        padding: 4rem 2rem;
        color: #656d70;
        font-size: 1.1rem;
    }

    .reader-viewport {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 0;
        margin-bottom: 2rem;
        border-radius: 4px;
        overflow: hidden;
        box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
        background: white;
        max-width: 1200px;
        margin-left: auto;
        margin-right: auto;
    }

    .reader-viewport.single {
        grid-template-columns: 1fr;
        max-width: 600px;
    }

    .expanded .reader-viewport {
        /* Constrain by height so both grid cells shrink together */
        max-height: 80vh;
        width: auto;
        max-width: 95vw;
        box-shadow: none;
        border-radius: 0;
        background: transparent;
    }

    .expanded .reader-viewport.single {
        max-width: calc(80vh * 1.02);
    }

    .expanded .reader-page {
        display: block;
        width: 100%;
        height: 100%;
        object-fit: contain;
    }

    @media (max-width: 639px) {
        .expanded .reader-viewport {
            max-height: 75vh;
        }

        .expanded .reader-viewport.single {
            max-width: calc(75vh * 1.02);
        }
    }

    .reader-page {
        display: block;
        width: 100%;
        height: auto;
        object-fit: cover;
        vertical-align: top;
    }

    .reader-viewport.single .reader-page {
        width: 100%;
    }

    .rotate-hint {
        text-align: center;
        font-size: 1.1rem;
        color: #656d70;
        opacity: 0.6;
        margin: -0.75rem 0 1.25rem;
        display: flex;
        gap: 0.4rem;
        align-items: center;
        justify-content: center;
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
        flex-shrink: 0;
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

    .expand-btn {
        margin-left: 1rem;
    }

    .expand-close {
        position: absolute;
        top: 1rem;
        right: 1rem;
        width: 44px;
        height: 44px;
        border-radius: 50%;
        border: 2px solid rgba(255, 255, 255, 0.5);
        background: transparent;
        color: rgba(255, 255, 255, 0.7);
        cursor: pointer;
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 10001;
        transition: all 0.2s ease;
    }

    .expand-close:hover {
        border-color: white;
        color: white;
        background: rgba(255, 255, 255, 0.1);
    }

    .expanded .reader-controls button {
        border-color: rgba(255, 255, 255, 0.5);
        color: rgba(255, 255, 255, 0.7);
    }

    .expanded .reader-controls button:hover:not(:disabled) {
        background: rgba(255, 255, 255, 0.15);
        color: white;
        border-color: white;
    }

    .expanded .reader-controls button:disabled {
        border-color: rgba(255, 255, 255, 0.15);
        color: rgba(255, 255, 255, 0.2);
    }

    .expanded .reader-label {
        color: rgba(255, 255, 255, 0.7);
    }

    .expanded .reader-download .btn-download {
        background: rgba(255, 255, 255, 0.15);
    }

    .expanded .reader-download .btn-download:hover {
        background: rgba(255, 255, 255, 0.25);
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
