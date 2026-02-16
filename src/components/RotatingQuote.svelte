<script lang="ts">
    import { onMount } from "svelte";

    const quotes = [
        { text: "Now our kids wake up screaming less", source: "Parent" },
        { text: "A sound legal argument for participating in a global crisis", source: "Lawyer" },
    ];

    let currentQuoteIndex = 0;
    let visible = true;

    onMount(() => {
        const interval = setInterval(() => {
            visible = false;
            setTimeout(() => {
                currentQuoteIndex = (currentQuoteIndex + 1) % quotes.length;
                visible = true;
            }, 600);
        }, 5000);

        return () => clearInterval(interval);
    });
</script>

<figure class="quote" class:visible>
    <blockquote>"{quotes[currentQuoteIndex].text}"</blockquote>
    <figcaption>— {quotes[currentQuoteIndex].source}</figcaption>
</figure>

<style>
    .quote {
        opacity: 0;
        transform: translateY(4px);
        transition: opacity 0.6s ease, transform 0.6s ease;
        margin: 0;
    }

    .quote.visible {
        opacity: 1;
        transform: translateY(0);
    }

    blockquote {
        font-style: italic;
        font-size: 1.25rem;
        color: white;
        line-height: 1.5;
        margin: 0;
    }

    figcaption {
        font-size: 0.9rem;
        color: rgba(255, 255, 255, 0.7);
        margin-top: 0.5rem;
    }

    @media (min-width: 768px) {
        blockquote {
            font-size: 1.5rem;
        }
    }
</style>
