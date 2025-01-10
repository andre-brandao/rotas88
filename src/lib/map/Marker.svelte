<script lang="ts">
	import { getContext, onDestroy, onMount } from 'svelte';
	import type { MapContext } from './types';
	// import L from 'leaflet';
	const { map } = getContext<MapContext>('map');
	type Props = {
		latLng: L.LatLngExpression;
        options?: L.MarkerOptions | undefined
	};

	let { latLng, options }: Props = $props();
    console.log(map);

	let marker: L.Marker | undefined = $state(undefined);

	onMount(() => {
        // if(!map) return;
        if (typeof window === 'undefined' || !map) return;
		import('leaflet').then((L) => {
            
			marker = L.marker(latLng, options);
			marker.addTo(map);
            console.log(marker);
            
		});
	});

    onDestroy(()=>{
        if(marker){
            marker.remove();
        }
    })
</script>
