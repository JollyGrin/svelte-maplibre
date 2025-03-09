<script lang="ts">
	import {
		FillExtrusionLayer,
		GeolocateControl,
		MapLibre,
		Marker,
		Popup,
		CustomLayer
	} from 'svelte-maplibre-gl';
	import maplibregl from 'maplibre-gl';
	import * as THREE from 'three';
	import { Canvas } from '@threlte/core';
	import Scene from './Scene.svelte';

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

	// Custom layer implementation for Threlte integration
	class ThrelteLayerImpl implements Omit<maplibregl.CustomLayerInterface, 'id' | 'type'> {
		renderingMode = '3d' as const;
		private camera = new THREE.Camera();
		private scene = new THREE.Scene();
		private renderer: THREE.WebGLRenderer | null = null;
		private map: maplibregl.Map | null = null;
		private threlteScene: THREE.Scene | null = null;

		onAdd(map: maplibregl.Map, gl: WebGL2RenderingContext) {
			this.map = map;

			// Use the MapLibre GL JS map canvas for three.js
			this.renderer = new THREE.WebGLRenderer({
				canvas: map.getCanvas(),
				context: gl,
				antialias: true
			});
			this.renderer.autoClear = false;
		}

		// Method to set the Threlte scene
		setThrelteScene(scene: THREE.Scene) {
			this.threlteScene = scene;
		}

		render(
			_gl: WebGL2RenderingContext | WebGLRenderingContext,
			args: maplibregl.CustomRenderMethodInput
		) {
			if (!this.threlteScene || !this.map) return;

			// Use the marker's position for the 3D object
			const modelOrigin: [number, number] = [lnglat.lng, lnglat.lat];
			const modelAltitude = 0;
			const scaling = 10.0; // Adjust scaling as needed

			// Get the correct model matrix for the current projection
			const modelMatrix = this.map.transform.getMatrixForModel(modelOrigin, modelAltitude);
			const m = new THREE.Matrix4().fromArray(args.defaultProjectionData.mainMatrix);
			const l = new THREE.Matrix4()
				.fromArray(modelMatrix)
				.scale(new THREE.Vector3(scaling, scaling, scaling));

			this.camera.projectionMatrix = m.multiply(l);
			this.renderer!.resetState();
			this.renderer!.render(this.threlteScene, this.camera);
			this.map.triggerRepaint();
		}
	}

	const threlteLayerImpl = new ThrelteLayerImpl();
	let sceneRef: { scene: THREE.Scene } | null = null;

	// Function to connect the Threlte scene to our custom layer
	function connectThrelteScene(scene: THREE.Scene) {
		if (threlteLayerImpl) {
			threlteLayerImpl.setThrelteScene(scene);
		}
	}

	// When the Threlte scene is created, connect it to our custom layer
	function handleSceneCreated(event: CustomEvent) {
		sceneRef = event.detail;
		if (sceneRef && sceneRef.scene) {
			connectThrelteScene(sceneRef.scene);
		}
	}
</script>

<div class="fixed top-3 right-3 z-50 flex w-fit items-center gap-2">
	<p class="rounded-lg bg-gray-200 p-2 text-xs">
		hold <span class="rounded-xs border-b-2 border-black/50 bg-white p-1">ctrl</span> & drag to tilt
		camera
	</p>
	<a
		href="https://github.com/JollyGrin/svelte-maplibre"
		class="rounded-full px-3 py-2 transition-all hover:scale-110 hover:bg-orange-200"
	>
		git
	</a>
</div>

<!-- Hidden Canvas with Threlte Scene -->
<div class="hidden">
	<Canvas>
		<Scene on:sceneCreated={handleSceneCreated} />
	</Canvas>
</div>

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
				<div class="text-3xl">📍</div>
				<div class="font-bold text-white drop-shadow-xs">{lngLatText}</div>
			</div>
		{/snippet}
		<Popup class="text-black" bind:open={popupOpen} offset={offsets}>
			<span class="text-lg">{lngLatText}</span>
			<p>Style this however you want</p>
		</Popup>
	</Marker>

	<!-- Add the custom Threlte layer -->
	<CustomLayer implementation={threlteLayerImpl} />
</MapLibre>
