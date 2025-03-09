<script lang="ts">
	import { T, useTask, useThrelte } from '@threlte/core';
	import { OrbitControls } from '@threlte/extras';
	import { Color, FogExp2 } from 'three';
	import { createEventDispatcher } from 'svelte';

	const { scene } = useThrelte();
	const dispatch = createEventDispatcher();

	// Dispatch the scene to the parent component
	useTask(() => {
		dispatch('sceneCreated', { scene });
	});
</script>

<T.PerspectiveCamera makeDefault position={[0, 10, 40]} fov={40}>
	<OrbitControls
		autoRotate
		autoRotateSpeed={0.3}
		maxPolarAngle={(Math.PI / 2) * 0.9}
		enableDamping
		enableZoom
		maxDistance={70}
	/>
</T.PerspectiveCamera>

<!-- Main sunlight - warm directional light -->
<T.DirectionalLight
	position={[5, 15, 10]}
	intensity={1.2}
	color="#ffd700"
	castShadow
	shadow.mapSize.width={2048}
	shadow.mapSize.height={2048}
	shadow.camera.far={50}
	shadow.camera.near={0.1}
/>

<!-- Ambient light for overall scene brightness -->
<T.AmbientLight intensity={0.1} color="#ffffff" castShadow />

<!-- Hemisphere light for sky/ground interaction -->
<T.HemisphereLight skyColor="#87ceeb" groundColor="#8b4513" intensity={0.2} />

<!-- Ground plane to receive shadows -->
<T.Mesh position={[0, 10, 0]}>
	<T.BoxGeometry args={[5, 20, 5]} />
	<T.MeshStandardMaterial
		color="#98FB98"
		roughness={0.9}
		metalness={0.05}
		transparent={true}
		opacity={0.5}
	/>
</T.Mesh>
