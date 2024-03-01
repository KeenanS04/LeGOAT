<script>
    import mapboxgl from "mapbox-gl";
    import { onMount } from "svelte";
    export let index;
    export let geoJsonToFit;
    
    mapboxgl.accessToken = "pk.eyJ1IjoiYW11anJhbCIsImEiOiJjbHNqbW9rZzgycHhvMmtzYmp5eTJwNnJ5In0.OA2ad1hWPGAfF7QQOGURJQ";
  
    let container;
    let map;
  
    let zoomLevel;

    const locations = [
        { name: "Akron", coords: [-81.519, 41.081], description: "LeBron's Hometown", color: "#000000" },
        // { name: "Cleveland", coords: [-81.6944, 41.4993], description: "First & Third NBA Team" },
        { name: "Miami", coords: [-80.1918, 25.7617], description: "Second NBA Team", color: "#98002E" },
        { name: "Cleveland", coords: [-81.6944, 41.4993], description: "First & Third NBA Team", color: "#860038"},
        { name: "Los Angeles", coords: [-118.2437, 34.0522], description: "Fourth NBA Team", color: "#FDB927"},
    ];

    let currentLocation = locations[0]; // Default to Akron
  
    function updateZoomLevel() {
      const screenWidth = window.innerWidth;
      zoomLevel = screenWidth <= 600 ? 4 : 5.85; // Adjust these values as needed
    }
  
    function handleResize() {
      updateZoomLevel();
      map.setZoom(zoomLevel);
    }
  
    onMount(() => {
      updateZoomLevel();
      map = new mapboxgl.Map({
        container,
        style: "mapbox://styles/mapbox/light-v11",
        center: [31.3, 55],
        zoom: zoomLevel,
        attributionControl: true, // removes attribution from the bottom of the map
      });
  
      window.addEventListener("resize", handleResize);
  
      function hideLabelLayers() {
        const labelLayerIds = map
          .getStyle()
          .layers.filter(
            (layer) =>
              layer.type === "symbol" && /label|text|place/.test(layer.id)
          )
          .map((layer) => layer.id);
  
        for (const layerId of labelLayerIds) {
          map.setLayoutProperty(layerId, "visibility", "none");
        }
      }
  
      map.on("load", () => {
        hideLabelLayers();
        updateBounds();
        map.on("zoom", updateBounds);
        map.on("drag", updateBounds);
        map.on("move", updateBounds);
      });
    });

    function drawJourneyLine() {
        if (index === 0) return; // No line to draw for the first index (Akron)
    
        const from = locations[index - 1].coords;
        const to = locations[index].coords;
        const color = locations[index].color; // Color of the destination team
        
        // Calculate intermediate point for the curve
        const midPointLat = (from[1] + to[1]) / 2;
        const midPointLng = (from[0] + to[0]) / 2;
        
        // Adjust midPointLng to create curvature
        // This adjustment factor controls the curve's "strength" and direction
        const curveAdjustment = 2; // Adjust this value based on desired curvature
        const midPointLngAdjusted = midPointLng + (index % 2 === 0 ? curveAdjustment : -curveAdjustment);
        
        const curvedCoordinates = [from, [midPointLngAdjusted, midPointLat], to];

        const lineData = {
            'type': 'Feature',
            'properties': {},
            'geometry': {
                'type': 'LineString',
                'coordinates': curvedCoordinates
            }
        };

        // Add or update the source and layer as before
        if (map.getSource('journeyLine')) {
            map.getSource('journeyLine').setData(lineData);
        } else {
            map.addSource('journeyLine', {
                'type': 'geojson',
                'data': lineData
            });

            map.addLayer({
                'id': 'journeyLine',
                'type': 'line',
                'source': 'journeyLine',
                'layout': {
                    'line-join': 'round',
                    'line-cap': 'round'
                },
                'paint': {
                    'line-color': color,
                    'line-width': 4
                }
            });
        }

        // Ensure the line color updates for each team
        map.setPaintProperty('journeyLine', 'line-color', color);
    }
    
    function updateBounds() {
        const bounds = map.getBounds();
        geoJsonToFit.features[0].geometry.coordinates = [
          bounds._ne.lng,
          bounds._ne.lat,
        ];
        geoJsonToFit.features[1].geometry.coordinates = [
          bounds._sw.lng,
          bounds._sw.lat,
      ];
    }
    let isVisible = true;
  
    $: if (map && locations[index]) {
        currentLocation = locations[index];
        map.flyTo({ center: currentLocation.coords, zoom: zoomLevel });
        drawJourneyLine();
        // Optional: Add markers or other visual elements here
    }
</script>
  
<svelte:head>
<link
    rel="stylesheet"
    href="https://api.mapbox.com/mapbox-gl-js/v2.14.0/mapbox-gl.css"
/>
</svelte:head>

<div class="map" class:visible={isVisible} bind:this={container} />

<style>
.map {
    width: 100%;
    height: 100vh; /* check problem when setting width */
    position: absolute;
    opacity: 0;
    visibility: hidden;
    transition: opacity 2s, visibility 2s;
    outline: blue solid 3px;
}

.map.visible {
    opacity: 1;
    visibility: visible;
}
</style>
  
  