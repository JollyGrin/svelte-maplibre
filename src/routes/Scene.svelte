<script lang="ts">
	import { T, useTask, useThrelte } from '@threlte/core';
	import { OrbitControls, Text } from '@threlte/extras';
	import { Color, FogExp2, Vector3 } from 'three';
	import { createEventDispatcher } from 'svelte';
	import { base } from '$app/paths';

	const { scene } = useThrelte();
	const dispatch = createEventDispatcher();

	// Animation parameters for floating text
	let floatY = 0;
	let floatDirection = 0.01;
	let textPosition: [number, number, number] = $state([0, 25, 0]);

	// Animation for floating text
	useTask((delta) => {
		// Simple floating animation
		floatY += floatDirection;
		if (floatY > 1 || floatY < -1) {
			floatDirection *= -1;
		}
		textPosition = [0, 10 + floatY, 0] as [number, number, number];
	});

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

<T.Group position={[0, 10, 0]}>
	<T.Mesh>
		<T.BoxGeometry args={[5, 20, 5]} />
		<T.MeshStandardMaterial
			color="#98FB98"
			roughness={0.9}
			metalness={0.05}
			transparent={true}
			opacity={0.5}
		/>
	</T.Mesh>

	<!-- 3D Floating Text -->
	<Text
		position={[0, 10, 0]}
		text="Hello Map!"
		color="#ff3e00"
		font="{base}/fonts/helvetiker_bold.typeface.json"
		size={20}
		height={0.5}
		curveSegments={12}
		bevelEnabled={true}
		bevelThickness={0.1}
		bevelSize={0.1}
		bevelSegments={3}
		castShadow
	>
		<T.MeshStandardMaterial
			color="#ff3e00"
			metalness={0.8}
			roughness={0.2}
			emissive="#ff3e00"
			emissiveIntensity={0.2}
		/>
	</Text>
</T.Group>
