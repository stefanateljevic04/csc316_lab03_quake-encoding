<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	import { feature } from 'topojson-client';
	import world from 'world-atlas/land-110m.json';

	const margin = { top: 36, right: 36, bottom: 36, left: 36 };
	const width = 960;
	const height = width / 2;
	const x = d3.scaleLinear().domain([-180, 180]).range([0, width]);
	const y = d3.scaleLinear().domain([-90, 90]).range([height, 0]);
	const radius = d3.scaleSqrt().domain([0, 8.8]).range([0, 30]);
	// Equirectangular projection matches x(longitude), y(latitude) exactly.
	// D3's spherical preclip splits geometry at the antimeridian.
	const projection = d3.geoEquirectangular()
		.scale(width / (2 * Math.PI)).translate([x(0), y(0)])
		.precision(0).clipAngle(null);
	const path = d3.geoPath(projection);
	const landPath = path(feature(world, world.objects.land));
	const graticulePath = path(d3.geoGraticule().step([30, 30])());
	let earthquakes = $state([]);
	let loading = $state(true);
	let error = $state('');
	let depthExtent = $derived(d3.extent(earthquakes, (d) => d.depth));
	let depth95 = $derived(d3.quantile(earthquakes.map((d) => d.depth), 0.95));
	let versions = $derived([
		{ id: 'full', title: 'Full observed depth range', domain: depthExtent,
			color: d3.scaleSequential(d3.interpolateViridis).domain(depthExtent), clamped: false },
		{ id: 'p95', title: 'Depth capped at the 95th percentile', domain: [depthExtent[0], depth95],
			color: d3.scaleSequential(d3.interpolateViridis).domain([depthExtent[0], depth95]).clamp(true), clamped: true }
	]);
	const legendWidth = 360;
	let colorAxes = $state([]);

	onMount(() => {
		let cancelled = false;
		d3.csv('/earthquakes_M6.0_1year.csv', (d) => ({
			...d, depth: Number(d.depth), mag: Number(d.mag),
			longitude: Number(d.longitude), latitude: Number(d.latitude)
		})).then((rows) => {
			if (!cancelled) earthquakes = rows;
		}).catch(() => {
			if (!cancelled) error = 'The earthquake data could not be loaded. Please refresh to try again.';
		}).finally(() => {
			if (!cancelled) loading = false;
		});
		return () => { cancelled = true; };
	});

	$effect(() => {
		versions.forEach((version, index) => {
			if (colorAxes[index]) {
				const position = d3.scaleLinear().domain(version.domain).range([0, legendWidth]);
				d3.select(colorAxes[index]).call(d3.axisBottom(position).ticks(5));
			}
		});
	});
</script>

{#if loading}
	<p role="status">Loading earthquake data…</p>
{:else if error}
	<p role="alert">{error}</p>
{:else}
	{#each versions as version, versionIndex (version.id)}
	<figure>
		<h2 class="version-title">{version.title}</h2>
		<svg class="map" viewBox={`0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`} role="img" aria-labelledby={`${version.id}-map-title ${version.id}-map-description`}>
			<title id={`${version.id}-map-title`}>World map of M6.0+ earthquakes</title>
			<desc id={`${version.id}-map-description`}>North is at the top. Longitude increases to the right and latitude upward. Circle area represents magnitude, and Viridis color represents depth in kilometers. The Pacific is split at plus or minus 180 degrees longitude.</desc>
			<g transform={`translate(${margin.left}, ${margin.top})`}>
				<rect width={width} height={height} fill="#f0f5f8" stroke="#c6d2dc" />
				<path d={landPath} fill="#dce3e5" stroke="#a9b8bc" stroke-width="0.8" />
				<path d={graticulePath} fill="none" stroke="#bbc9d0" stroke-width="0.5" />
				{#each earthquakes as quake (quake.id)}
					<circle cx={x(quake.longitude)} cy={y(quake.latitude)} r={radius(quake.mag)} fill={version.color(quake.depth)} stroke="#fff" stroke-width="0.7">
						<title>{quake.place}: M{quake.mag}, depth {quake.depth} km; {quake.latitude}° latitude, {quake.longitude}° longitude</title>
					</circle>
				{/each}
			</g>
		</svg>
		<div class="legends">
			<section aria-label="Magnitude size legend">
				<h2>Magnitude (M)</h2>
				<svg viewBox="0 0 420 110" role="img" aria-label="Magnitude size legend: M6, M7, and M8">
					{#each [6, 7, 8] as magnitude, index}
						<circle cx={60 + index * 120} cy="38" r={radius(magnitude)} fill="#536779" />
						<text x={60 + index * 120} y="88" text-anchor="middle">M{magnitude}</text>
					{/each}
				</svg>
				<p>Circle area is proportional to the magnitude number. Magnitude is logarithmic: each +1 corresponds to approximately 32× energy. These circles are not proportional to earthquake energy or energy ratios.</p>
			</section>
			<section aria-label="Depth color legend">
				<h2>Depth (km)</h2>
				<svg viewBox="0 0 420 110" role="img" aria-label={`Depth in kilometers, ${version.domain[0]} to ${version.clamped ? "at or above " : ""}${version.domain[1]}`}>
					<defs>
						<linearGradient id={`${version.id}-depth-gradient`}>
							{#each d3.range(0, 1.01, 0.01) as t}
								<stop offset={`${t * 100}%`} stop-color={d3.interpolateViridis(t)} />
							{/each}
						</linearGradient>
					</defs>
					<g transform="translate(30, 20)">
						<rect width={legendWidth} height="20" fill={`url(#${version.id}-depth-gradient)`} />
						<g bind:this={colorAxes[versionIndex]} transform="translate(0, 20)"></g>
						<text x={legendWidth} y="72" text-anchor="end">{version.clamped ? "≥ " : ""}{version.domain[1]?.toFixed(2)} km</text>
					</g>
				</svg>
				{#if version.clamped}
					<p>95th-percentile cutoff: {depth95.toFixed(2)} km. All depths at or above this cutoff share the maximum yellow color. The full data remain on the map.</p>
				{:else}
					<p>Full observed range: {depthExtent[0].toFixed(2)}–{depthExtent[1].toFixed(2)} km below the surface. Purple represents shallower earthquakes; yellow represents deeper earthquakes.</p>
				{/if}
			</section>
		</div>
		<figcaption>M6.0 and larger only. {earthquakes.length} earthquakes · One circle per earthquake. Overlapping circles may hide events. Equirectangular map; the Pacific is split at ±180°. Basemap: World Atlas / Natural Earth, 1:110 million.</figcaption>
	</figure>
	{/each}
{/if}

<style>
	figure { margin: 0 0 48px; }
	.version-title { font-size: 1.3rem; margin: 24px 0 16px; }
	svg { display: block; width: 100%; height: auto; }
	svg :global(text) { font-family: inherit; font-size: 13px; }
	.legends { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 32px; margin: 20px 0; }
	.legends svg { max-width: 420px; }
	h2 { font-size: 1.1rem; margin: 0 0 12px; }
	p, figcaption { color: #526071; font-size: 0.9rem; line-height: 1.5; }
	@media (max-width: 600px) { .legends { grid-template-columns: 1fr; } }
</style>
