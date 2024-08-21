<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import mapboxgl from 'mapbox-gl';

	let map: mapboxgl.Map;
	let selectedFeature: GeoJSON.Feature<GeoJSON.Geometry> | null = null;
	let maxSpeedFilter: number = 50;
	let maxSpeedInfo: string = 'N/A';

	onMount(() => {
		mapboxgl.accessToken =
			'pk.eyJ1IjoidGFyaWtnYWRvdW1pIiwiYSI6ImNsemd3NG05bzFua2IyanF3Mzkzbmg2NW8ifQ.pTCrY1WgjDRR9gUdTXefBA';

		map = new mapboxgl.Map({
			container: 'map',
			style: 'mapbox://styles/mapbox/streets-v11',
			center: [2.3522, 48.8566], // le centre de paris
			zoom: 11
		});

		map.on('load', () => {
			map.addSource('roads', {
				type: 'vector',
				url: 'http://localhost:8080/data/v3.json',
				minzoom: 0,
				maxzoom: 14
			});

			map.addLayer({
				id: 'roads',
				type: 'line',
				source: 'roads',
				'source-layer': 'transportation',
				layout: {},
				paint: {
					'line-color': [
						'case',
						['>', ['to-number', ['coalesce', ['get', 'maxspeed'], '0']], maxSpeedFilter],
						'#ff0000',
						'#888'
					],
					'line-width': 2
				}
			});

			map.on('click', 'roads', (e) => {
				if (e.features && e.features.length > 0) {
					selectedFeature = e.features[0] as GeoJSON.Feature<GeoJSON.Geometry>;
					maxSpeedInfo = selectedFeature.properties?.maxspeed || 'unknown';

					console.log('Feature properties:', selectedFeature.properties);
				}
			});
		});
	});

	function updateMapFilter() {
		if (map) {
			map.setPaintProperty('roads', 'line-color', [
				'case',
				[
					'>',
					[
						'to-number',
						[
							'coalesce',
							[
								'match',
								// Check if maxspeed is a valid number or "unknown"
								['get', 'maxspeed'],
								['signals', ''],
								'0', // If it is "signals" or empty, return 0
								['get', 'maxspeed'] // Else return the maxspeed value
							],
							'0'
						]
					],
					maxSpeedFilter
				],
				'#ff0000', // Color for roads exceeding the filter
				'#888' // Color for roads below or equal to the filter
			]);
		}
	}

	$: updateMapFilter();

	onDestroy(() => {
		map?.remove();
	});
</script>

<div id="app-container">
	<div id="dashboard">
		<h2>Dashboard</h2>
		<label for="maxSpeed">Max Speed Filter: {maxSpeedFilter} km/h</label>
		<input
			id="maxSpeed"
			type="range"
			min="0"
			max="130"
			step="10"
			bind:value={maxSpeedFilter}
			on:input={updateMapFilter}
		/>
		<div id="feature-info">
			<h3>Selected Feature Info:</h3>
			<p><strong>Max Speed:</strong> {maxSpeedInfo}</p>
		</div>
	</div>
	<div id="map"></div>
</div>

<style>
	#app-container {
		display: flex;
		flex-direction: row;
		align-items: flex-start;
		justify-content: center;
		overflow: hidden;
	}

	#map {
		width: 70%;
		height: 90vh;
		margin: 20px;
	}

	#dashboard {
		width: 25%;
		padding: 20px;
		border: 2px solid black;
		background-color: #f9f9f9;
		margin: 20px;
		border-radius: 10px;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
		display: flex;
		flex-direction: column;
	}

	#feature-info {
		margin-top: 20px;
		padding: 10px;
		border: 1px solid #ddd;
		background-color: #fff;
		border-radius: 5px;
	}
</style>
