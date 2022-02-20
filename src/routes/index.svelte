<script>
    import Calendar from '$lib/Calendar.svelte'
</script>

<svelte:head>
    <title>Calendar</title>
</svelte:head>

<main>
    <Calendar selected_day={new Date()} />
</main>

<style>
    :global(:root) {
        --color-dark: #222222;
        --color-light: #f2f2f2;
        --color-highlight: #ffd000;
        --color-primary: #00d3e2;
    }

    :global(html, body, *::before, *::after) {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
    }

    :global(html, body) {
        background-color: var(--color-dark);
        color: var(--color-light);
        height: 100%;
        font-family: 'open sans', sans-serif;
    }

    main {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: flex-start;
        padding-top: 2rem;
    }
</style>