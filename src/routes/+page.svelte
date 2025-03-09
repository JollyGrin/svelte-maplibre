<script lang="ts">
	import { MapLibre, Marker, Popup } from 'svelte-maplibre-gl';

	let lnglat = $state({ lng: 11.5761, lat: 48.1371 });
	let lngLatText = $derived(`(${lnglat.lat.toFixed(3)}, ${lnglat.lng.toFixed(3)})`);
	let popupOpen = $state(true);
	let offset = $state(24);

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
	center={[11.5761, 48.1371]}
	maxPitch={85}
>
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
