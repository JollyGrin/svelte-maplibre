<script lang="ts">
	import { T, useTask, useThrelte } from '@threlte/core';
	import { OrbitControls, Text, Text3DGeometry } from '@threlte/extras';
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
	<T.Mesh position={[-30, -10, -30]}>
		<Text3DGeometry text={'Amsterdam'} size={10} depth={5} />
		<T.MeshStandardMaterial color="#FD3F00" toneMapped={false} metalness={0.2} roughness={0.8} />
	</T.Mesh>
</T.Group>
