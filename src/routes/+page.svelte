<script>
	import Flower from '$lib/map/Flower.svelte';
	import Map from '$lib/map/Map.svelte';
	import Marker from '$lib/map/Marker.svelte';

	let radius = $state(2400);
	let numPetals = $state(6);
	$inspect({ radius, numPetals });
</script>

<div class="bg-white">
	<div>
		Radius <input type="range" bind:value={radius} min="0" max="10000" step="10" /> (radius: {radius /
			1000}
		km) (max_distance: {(radius * 2) / 1000} km)
	</div>
	<div>
		Number of petals <input type="range" bind:value={numPetals} min="0" max="20" step="1" />
	</div>
</div>
<Map options={{ center: [-19.96811611070914, -43.96033258481795], zoom: 13 }}>
			<Marker latLng={[-19.96811611070914, -43.95033258481795]} />
			<Marker latLng={[-19.95811611070914, -43.97033258481795]} />
			<Marker latLng={[-19.97811611070914, -43.95033258681795]} />
			<Marker latLng={[-19.96815611070914, -43.97033258481795]} />
			{#key [radius, numPetals]}
				<Flower
					latLng={[-19.96811611070914, -43.96033258481795]}
					
					options={{
						radius: radius,
						numPetals: numPetals
					}}
				/>
			{/key}
</Map>
