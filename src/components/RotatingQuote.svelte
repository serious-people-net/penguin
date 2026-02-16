<script lang="ts">
    import { onMount } from "svelte";

    const quotes = [
        "Now our kids wake up screaming less",
        "A sound legal argument for participating in a global crisis",
    ];

    let currentQuoteIndex = 0;
    let opacity = 1;

    onMount(() => {
        const interval = setInterval(() => {
            opacity = 0;
            setTimeout(() => {
                currentQuoteIndex = (currentQuoteIndex + 1) % quotes.length;
                opacity = 1;
            }, 500);
        }, 5000);

        return () => clearInterval(interval);
    });
</script>

<div
    class="quote-container"
    style="opacity: {opacity}; transition: opacity 0.5s ease-in-out;"
>
    <p
        class="text-lg md:text-xl italic text-gray-700 text-center max-w-2xl mx-auto mb-8"
    >
        "{quotes[currentQuoteIndex]}"
    </p>
</div>
