<script lang="ts">
	import {
		FillExtrusionLayer,
		GeolocateControl,
		MapLibre,
		Marker,
		Popup
	} from 'svelte-maplibre-gl';

	let lnglat = $state({ lng: 4.902, lat: 52.379 });
	let lngLatText = $derived(`(${lnglat.lat.toFixed(3)}, ${lnglat.lng.toFixed(3)})`);
	let popupOpen = $state(true);
	let offset = $state(24);

	// for geolocation
	let logString = $state('Press the GeolocateControl button.\n');
	function log(s: string) {
		logString += '» ' + s + '\n';
	}

	// Positions the popup beneath the arrow
	let offsets: maplibregl.Offset = $derived({
		top: [0, offset],
		bottom: [0, -offset],
		left: [offset + 12, 0],
		right: [-offset - 12, 0],
		center: [0, 0],
		'top-left': [offset, offset],
		'top-right': [-offset, offset],
		'bottom-left': [offset, -offset],
		'bottom-right': [-offset, -offset]
	});
</script>

<MapLibre
	class="h-screen"
	style="https://basemaps.cartocdn.com/gl/voyager-gl-style/style.json"
	zoom={14}
	center={[4.902, 52.379]}
	maxPitch={85}
>
	<GeolocateControl
		position="top-left"
		positionOptions={{ enableHighAccuracy: true }}
		trackUserLocation={true}
		showAccuracyCircle={true}
		ontrackuserlocationstart={() => log('trackuserlocationstart')}
		ontrackuserlocationend={() => log('trackuserlocationend')}
		ongeolocate={(ev) => log(`geolocate ${JSON.stringify(ev.coords, null, 2)}`)}
	/>
	<FillExtrusionLayer
		source="carto"
		sourceLayer="building"
		minzoom={14}
		filter={['!=', ['get', 'hide_3d'], true]}
		paint={{
			'fill-extrusion-color': [
				'interpolate',
				['linear'],
				['get', 'render_height'],
				0,
				'#aaccbb',
				200,
				'royalblue',
				400,
				'purple'
			],
			'fill-extrusion-height': [
				'interpolate',
				['linear'],
				['zoom'],
				14,
				0,
				15,
				['get', 'render_height']
			],
			'fill-extrusion-base': ['case', ['>=', ['get', 'zoom'], 14], ['get', 'render_min_height'], 0]
		}}
	/>
	<Marker bind:lnglat draggable>
		{#snippet content()}
			<div class="text-center leading-none">
				<div class="text-3xl">🐶</div>
				<div class="font-bold text-white drop-shadow-xs">{lngLatText}</div>
			</div>
		{/snippet}
		<Popup class="text-black" bind:open={popupOpen} offset={offsets}>
			<span class="text-lg">{lngLatText}</span>
			<p>Style this however you want</p>
		</Popup>
	</Marker>
</MapLibre>
