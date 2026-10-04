<script>
	import DepthMagnitudePlot from '$lib/DepthMagnitudePlot.svelte';
</script>

<svelte:head>
	<title>World earthquake bubble map</title>
</svelte:head>

<main>
	<p class="eyebrow">CSC316 · Quake Encoding</p>
	<h1>Where do large earthquakes occur?</h1>
	<p>Explore the locations, magnitudes, and depths of M6.0+ earthquakes recorded worldwide in the USGS dataset.</p>
	<DepthMagnitudePlot />
	<p>Position shows longitude and latitude, circle area shows magnitude, and color shows depth. This dataset includes only M6.0+ events; smaller earthquakes are filtered out.</p>
</main>

<style>
	:global(body) { margin: 0; background: #fafbfc; color: #182638; font-family: system-ui, sans-serif; }
	main { max-width: 1500px; margin: 0 auto; padding: 40px 24px; overflow-x: auto; }
	.eyebrow { font-size: 0.85rem; color: #526071; }
	h1 { font-size: clamp(1.6rem, 4vw, 2.3rem); line-height: 1.2; }
	p { line-height: 1.6; max-width: 75ch; }
</style>
