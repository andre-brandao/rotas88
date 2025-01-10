<script lang="ts">
	import { getContext, onDestroy, onMount } from 'svelte';
	import type { MapContext } from './types';

	type Props = {
		latLng: L.LatLngExpression;
		options: {
			radius: number;
			numPetals: number;
		};
	};

	let { map } = getContext<MapContext>('map');
	let { latLng, options }: Props = $props();
	let circles: L.Circle[] = $state([]);

	function getPointOnCircle(
		centerLat: number,
		centerLng: number,
		radiusInMeters: number,
		angleInDegrees: number
	) {
		const earthRadius = 6371000;
		const angleRad = (angleInDegrees * Math.PI) / 180;
		const radiusRad = radiusInMeters / earthRadius;
		const centerLatRad = (centerLat * Math.PI) / 180;
		const centerLngRad = (centerLng * Math.PI) / 180;

		const newLatRad = Math.asin(
			Math.sin(centerLatRad) * Math.cos(radiusRad) +
				Math.cos(centerLatRad) * Math.sin(radiusRad) * Math.cos(angleRad)
		);

		const newLngRad =
			centerLngRad +
			Math.atan2(
				Math.sin(angleRad) * Math.sin(radiusRad) * Math.cos(centerLatRad),
				Math.cos(radiusRad) - Math.sin(centerLatRad) * Math.sin(newLatRad)
			);

		return {
			lat: (newLatRad * 180) / Math.PI,
			lng: (newLngRad * 180) / Math.PI
		};
	}

	function generateFlowerPattern(
		centerLat: number,
		centerLng: number,
		radiusInMeters: number,
		numPetals: number
	) {
		const circleConfigs = [
			// Center circle
			{
				center: { lat: centerLat, lng: centerLng },
				radius: radiusInMeters
			}
		];

		// Calculate the positions for the surrounding circles
		// The centers of the petals should be at radiusInMeters distance (not 2x)
		// so they touch the inner circle's circumference
		const angleStep = 360 / numPetals;

		for (let i = 0; i < numPetals; i++) {
			const angle = i * angleStep;
			// Changed from radiusInMeters * 2 to just radiusInMeters
			const petalCenter = getPointOnCircle(centerLat, centerLng, radiusInMeters, angle);

			circleConfigs.push({
				center: petalCenter,
				radius: radiusInMeters
			});
		}

		return circleConfigs;
	}

	onMount(() => {
		if (typeof window === 'undefined' || !map) return;

		import('leaflet').then((L) => {
			// Extract lat/lng from the LatLngExpression
			let lat: number, lng: number;

			if (Array.isArray(latLng)) {
				[lat, lng] = latLng;
			} else if (typeof latLng === 'object' && 'lat' in latLng && 'lng' in latLng) {
				lat = typeof latLng.lat === 'function' ? latLng.lat : latLng.lat;
				lng = typeof latLng.lng === 'function' ? latLng.lng : latLng.lng;
			} else {
				console.error('Invalid latLng format');
				return;
			}

			// Generate and add circles
			const circleConfigs = generateFlowerPattern(lat, lng, options.radius, options.numPetals); // 500m radius, 6 petals

			circles = circleConfigs.map((config) =>
				L.circle([config.center.lat, config.center.lng], {
					radius: config.radius,
					color: 'red',
					fillColor: '#304f',
					fillOpacity: 0.2
				}).addTo(map)
			);
		});
	});

	onDestroy(() => {
		circles.forEach((circle) => circle.remove());
	});
</script>
