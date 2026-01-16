<script>

	import '../../assets/global-styles.css';

	import { onMount } from "svelte";

	import maplibregl from "maplibre-gl";
	import "maplibre-gl/dist/maplibre-gl.css";
	import * as pmtiles from "pmtiles";

	let map;

	let buildings = "/toronto-buildings/buildings-2025.pmtiles.gz";

	const heightColors = [
		"#1E3765",
		"#17608A",
		"#007FA3",
		"#2ACFC0",
		"#B8F0A8",
		"#e2c55a",
		"#F27C36",
		"#D84034",
		"#F10E77"
		];

	const buildingHeights = [0, 12, 25, 50, 75, 100, 150, 200, 400];

	onMount(async () => {

		const protocol = new pmtiles.Protocol();
		maplibregl.addProtocol("pmtiles", protocol.tile);
		
		
		map = new maplibregl.Map({
			container: "map",
			style: {
				version: 8,
				glyphs: "https://schoolofcities.github.io/fonts/fonts/{fontstack}/{range}.pbf",
				sources: {
					osm: {
						type: 'vector',
						tiles: [
						'https://vector.openstreetmap.org/shortbread_v1/{z}/{x}/{y}.mvt'
						]
					}
				},
				layers: [
					{
						id: 'background',
						type: 'background',
						paint: {
							'background-color': '#141414'
						}
					},
					{
						id: 'oceans',
						type: 'fill',
						source: 'osm',
						'source-layer': 'oceans',
						paint: {
							'fill-color': '#2c2f33'
						}
					},
					{
						id: 'water_polygons',
						type: 'fill',
						source: 'osm',
						'source-layer': 'water_polygons',
						paint: {
							'fill-color': '#2c2f33'
						}
					},
					{
						id: 'water_lines',
						type: 'line',
						source: 'osm',
						'source-layer': 'water_lines',
						paint: {
							'line-color': '#2c2f33',
							'line-width': [
							'interpolate',
							['linear'],
							['zoom'],
							10, 1,
							16, 3
							],
							'line-opacity': 1
						}
					}
				]
			},
			// center: [-79.37674, 43.67998],
			center: [-79.386783, 43.670203],
			zoom: 13,
			bearing: 0, 
			pitch: 0, 
			scrollZoom: true,
			dragRotate: false,
			touchPitch: false, 
			dragPan: true,
			touchZoomRotate: true,
			boxZoom: true,
			keyboard: true,
			doubleClickZoom: true,
			scrollZoom: true,
			minZoom: 10,
			maxZoom: 17,
			projection: "mercator",
			attributionControl: false,
		});

		const zoomInSVG = encodeURIComponent(`<svg xmlns="http://www.w3.org/2000/svg" width="29" height="29" fill="#fff" viewBox="0 0 29 29"><path d="M14.5 8.5c-.75 0-1.5.75-1.5 1.5v3h-3c-.75 0-1.5.75-1.5 1.5S9.25 16 10 16h3v3c0 .75.75 1.5 1.5 1.5S16 19.75 16 19v-3h3c.75 0 1.5-.75 1.5-1.5S19.75 13 19 13h-3v-3c0-.75-.75-1.5-1.5-1.5"/></svg>`);
		const zoomOutSVG = encodeURIComponent(`<svg xmlns="http://www.w3.org/2000/svg" width="29" height="29" fill="#fff" viewBox="0 0 29 29"><path d="M10 13c-.75 0-1.5.75-1.5 1.5S9.25 16 10 16h9c.75 0 1.5-.75 1.5-1.5S19.75 13 19 13z"/></svg>`);

		const nav = new maplibregl.NavigationControl({ showCompass: false });
		map.addControl(nav, 'top-right');

		setTimeout(() => {
			const navGroup = document.querySelector('.maplibregl-ctrl-group');
			if (!navGroup) return;

			// Force parent container black
			navGroup.style.backgroundColor = '#000';
			navGroup.style.border = 'none';
			navGroup.style.padding = '0';

			const buttons = [
				{ selector: '.maplibregl-ctrl-zoom-in .maplibregl-ctrl-icon', svg: zoomInSVG },
				{ selector: '.maplibregl-ctrl-zoom-out .maplibregl-ctrl-icon', svg: zoomOutSVG },
			];

			buttons.forEach(({ selector, svg }, index) => {
				const icon = navGroup.querySelector(selector);
				if (!icon) return;

				// Replace symbol
				icon.style.backgroundImage = `url("data:image/svg+xml;charset=utf-8,${svg}")`;

				// Style parent button
				const btn = icon.parentElement;
				btn.style.backgroundColor = '#000'; // black background
				btn.style.border = '1px solid #555'; // dark gray border
				btn.style.borderRadius = '3px';
				btn.style.padding = '5px';
				btn.style.cursor = 'pointer';
				btn.style.transition = 'opacity 0.2s';

				// Spacing between buttons
				if (index === 1) btn.style.marginTop = '2px';

				// Hover effect
				btn.addEventListener('mouseenter', () => { btn.style.opacity = '0.75'; });
				btn.addEventListener('mouseleave', () => { btn.style.opacity = '1'; });
			});
		}, 0);

		const scale = new maplibregl.ScaleControl({ unit: 'metric', maxWidth: 120 });
		map.addControl(scale, 'bottom-right');

		const scaleEl = document.querySelector('.maplibregl-ctrl-scale');
		if (scaleEl) {
			scaleEl.style.backgroundColor = 'rgba(0,0,0,0.42)';
			scaleEl.style.color = '#fff';
			scaleEl.style.fontFamily = 'OpenSans, sans-serif';
			scaleEl.style.fontSize = '11px';
			scaleEl.style.padding = '2px 4px';
			scaleEl.style.borderRadius = '3px';
			scaleEl.style.boxSizing = 'border-box';
			scaleEl.style.whiteSpace = 'nowrap';
			scaleEl.style.borderTop = '0px solid #fff';
			scaleEl.style.borderBottom = '1px solid #fff';
			scaleEl.style.borderLeft = '1px solid #fff';
			scaleEl.style.borderRight = '1px solid #fff';
			scaleEl.style.borderRadius = '0';
			scaleEl.style.marginBottom = '40px';
		}
		map.on("load", () => {

			map.addSource("buildings", {
				type: "vector",
				url: "pmtiles://" + buildings,
			});

			map.addLayer({
				id: "buildings-fill",
				type: "fill",
				source: "buildings",
				"source-layer": "buildings2025",
				paint: {
					"fill-color": [
						"step",
						["get", "AVG_HEIGHT"],
						heightColors[0],
						buildingHeights[1], heightColors[1],
						buildingHeights[2], heightColors[2], 
						buildingHeights[3], heightColors[3],
						buildingHeights[4], heightColors[4],
						buildingHeights[5], heightColors[5],
						buildingHeights[6], heightColors[6],
						buildingHeights[7], heightColors[7],
						buildingHeights[8], heightColors[8]
					],
					"fill-opacity": 1,
				}
			});

			map.addLayer({
				id: "buildings-line",
				type: "line",
				source: "buildings",
				"source-layer": "buildings2025",
				paint: {
					"line-color": "#fff",
					"line-width": [
						"interpolate",
						["linear"],
						["zoom"],
						15, 0,
						18, 0.3,
    				],
					"line-opacity": 0.7
				}
			});

			map.addLayer({
				id: "buildings-line-tall",
				type: "line",
				source: "buildings",
				"source-layer": "buildings2025",
				filter: [">", "AVG_HEIGHT", 150], 
				paint: {
					"line-color": "#fff",
					"line-width": [
						"interpolate",
						["linear"],
						["zoom"],
						14, 0.25,
						17, 1.25,
					],
					"line-opacity": 1
				}
			});

			map.addLayer({
				id: "buildings-hover-outline",
				type: "line",
				source: "buildings",
				"source-layer": "buildings2025",
				paint: {
					"line-color": "#fff",
					"line-width": [
						"case",
						["boolean", ["feature-state", "hover"], false],
						2.5,
						0
					],
					"line-opacity": [
						"case",
						["boolean", ["feature-state", "hover"], false],
						1,
						0
					]
				}
			});

			const tooltip = document.getElementById("tooltip");
			let hoveredBuildingId = null;

			map.on("mousemove", "buildings-fill", (e) => {
				const feature = e.features?.[0];
				if (!feature || map.getZoom() < 14) {
					tooltip.style.display = "none";
					if (hoveredBuildingId !== null) {
						map.setFeatureState(
							{ source: "buildings", sourceLayer: "buildings2025", id: hoveredBuildingId },
							{ hover: false }
						);
						hoveredBuildingId = null;
					}
					map.getCanvas().style.cursor = "";
					return;
				}

				// Tooltip content
				const height = feature.properties.AVG_HEIGHT;
				if (height > 400) {
					tooltip.innerHTML = `CN TOWER :)`; // Custom HTML for tall building
				} else {
					tooltip.innerHTML = `AVG_HEIGHT: ${Math.round(height)} m`;
				}

				tooltip.style.display = "block";
				tooltip.style.left = e.point.x + "px";
				tooltip.style.top = e.point.y - 12 + "px";
				tooltip.style.transform = "translate(-50%, -100%)";

				// Cursor pointer
				map.getCanvas().style.cursor = "pointer";

				// Hover outline using feature-state
				if (hoveredBuildingId !== null && hoveredBuildingId !== feature.id) {
					map.setFeatureState(
						{ source: "buildings", sourceLayer: "buildings2025", id: hoveredBuildingId },
						{ hover: false }
					);
				}
				hoveredBuildingId = feature.id;
				map.setFeatureState(
					{ source: "buildings", sourceLayer: "buildings2025", id: hoveredBuildingId },
					{ hover: true }
				);
			});

			map.on("mouseleave", "buildings-fill", () => {
				tooltip.style.display = "none";
				map.getCanvas().style.cursor = "";
				if (hoveredBuildingId !== null) {
					map.setFeatureState(
						{ source: "buildings", sourceLayer: "buildings2025", id: hoveredBuildingId },
						{ hover: false }
					);
					hoveredBuildingId = null;
				}
			});

		});

	});

</script>



<svelte:head>

	<title>Toronto Building Heights | School of Cities</title>

	<meta
		name="viewport"
		content="width=device-width, initial-scale=1, minimum-scale=1"
	/>

	<meta name="description" content="">
	<meta name="author" content="Jeff Allen">
    
	<meta property="og:title" content="Toronto Building Heights" />
	<meta property="og:description" content="Interactive map of building massing data across Toronto" />
	<meta property="og:type" content="website" />
	<meta property="og:url" content="https://schoolofcities.github.io/toronto-buildings/heights-2025" />
	<meta property="og:image" content="https://schoolofcities.github.io/toronto-buildings/web-card.png" />
	<meta property="og:locale" content="en_CA">

	<meta name="twitter:card" content="summary_large_image" />
	<meta name="twitter:site" content="https://schoolofcities.github.io/toronto-buildings/heights-2025" />
	<meta name="twitter:title" content="Toronto Building Heights" />
	<meta name="twitter:description" content="Interactive map of building massing data across Toronto" />
	<meta name="twitter:image" content="https://schoolofcities.github.io/toronto-buildings/web-card.png" /> 


</svelte:head>



<main>

	<div id="map-legend">
		<div id="map-title">Toronto Building Heights</div>
		<div id="legend">
			{#each heightColors as color, i}
				<div class="legend-item">
					{#if i < heightColors.length - 1}
						<span class="legend-color" style="background-color: {color}"></span>
						<span class="legend-label">
						{#if i === 0}
							&lt; {buildingHeights[i+1]} m
						{:else if i === heightColors.length - 2}
							≥ {buildingHeights[i]} m
						{:else if i === heightColors.length - 1}
							553 m
						{:else}
							{buildingHeights[i]}–{buildingHeights[i+1]-1} m
						{/if}
					</span>
					{/if}
				</div>
			{/each}
		</div>
	</div>

	<div id="map-credits">
		Map by <a href="https://jamaps.github.io/" target="_blank">Jeff Allen</a> at the <a href="https://schoolofcities.utoronto.ca/" target="_blank">School of Cities</a> built using <a href="https://github.com/felt/tippecanoe" target="_blank">tippecanoe</a>, <a href="https://github.com/protomaps/PMTiles" target="_blank">PMTiles</a>, <a href="https://github.com/maplibre/maplibre-gl-js" target="_blank">MapLibre</a>, and data from the <a href="https://open.toronto.ca/dataset/3d-massing/" target="_blank">City of Toronto</a> (2025).
	</div>

	<div id="map"></div>

	<div id="tooltip"></div>

</main>



<style>

	#map {
		height: 100dvh;
		width: 100dvw;
		background-color: #141414;
		z-index: 0;
	}

	#tooltip {
		position: absolute;
		pointer-events: none;
		background: rgba(20, 20, 20, 0.9);
		border: solid 0.5px rgb(134, 134, 134);
		color: #fff;
		padding: 6px 8px;
		font-size: 12px;
		border-radius: 1px;
		white-space: nowrap;
		display: none;
		z-index: 10;
	}

	#map-legend {
		position: absolute;
		top: 10px;
		left: 10px;
		background: rgba(20, 20, 20, 0.85);
		color: #fff;
		padding-right: 7px;
		padding-left: 15px;
		padding-top: 10px;
		padding-bottom: 10px;
		border-radius: 4px;
		font-family: "OpenSans", sans-serif;
		font-size: 13px;
		z-index: 20;
		box-shadow: 0 0 5px rgba(0,0,0,0.5);
		width: 100px;
		word-wrap: break-word;
	}

	#map-title {
		font-family: "OpenSansBold", sans-serif;
		font-size: 16px;
		margin-bottom: 6px;
	}

	#legend {
		display: flex;
		flex-direction: column;
		gap: 4px;
	}

	.legend-item {
		display: flex;
		align-items: center;
		gap: 6px;
	}

	.legend-color {
		width: 16px;
		height: 16px;
		border-radius: 2px;
		border: 1px solid #fff;
	}

	.legend-label {
		font-family: "OpenSans", sans-serif;
	}

	#map-credits {
		position: absolute;
		bottom: 2px;       /* flush to bottom */
		right: 2px;        /* flush to right */
		font-family: "OpenSans", sans-serif;
		font-size: 11px;
		line-height: 1.3;
		color: #ffffff;
		background: rgba(20, 20, 20, 0.65); /* optional subtle background for readability */
		padding: 4px 6px;
		border-radius: 3px;
		z-index: 20;
	}

	#map-credits a {
		color: inherit;             /* same as text */
		text-decoration: underline;
		transition: color 0.2s;
	}

	#map-credits a:hover {
		color: var(--brandLightBlue); /* your brand light blue */
	}

	

</style>