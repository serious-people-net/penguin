<script lang="ts">
    import { onMount } from "svelte";

    const quotes = [
        { text: "Now our kids wake up screaming less", source: "Parent" },
        {
            text: "A sound legal argument for participating in a global crisis",
            source: "Lawyer",
        },
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
        transition:
            opacity 0.6s ease,
            transform 0.6s ease;
        margin: 0;
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: center;
        width: 100%;
    }

    .quote.visible {
        opacity: 1;
        transform: translateY(0);
    }

    blockquote {
        font-style: italic;
        font-size: 1.5rem;
        color: white;
        line-height: 1.35;
        margin: 0;
        text-align: center;
        font-weight: 400;
        max-width: 280px;
        text-wrap: balance;
    }

    figcaption {
        font-size: 0.95rem;
        color: rgba(255, 255, 255, 0.7);
        margin-top: 0.75rem;
    }

    @media (min-width: 500px) {
        blockquote {
            font-size: 1.75rem;
            max-width: 400px;
        }
    }

    @media (min-width: 768px) {
        blockquote {
            font-size: 2rem;
            max-width: 520px;
        }

        figcaption {
            font-size: 1.05rem;
        }
    }

    @media (min-width: 1024px) {
        blockquote {
            font-size: 2.25rem;
            max-width: 600px;
        }

        figcaption {
            font-size: 1.1rem;
        }
    }
</style>
