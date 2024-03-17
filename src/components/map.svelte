<script>
  import mapboxgl from "mapbox-gl";
  import { onMount } from "svelte";
  export let index;

  mapboxgl.accessToken =
    "pk.eyJ1IjoiYW11anJhbCIsImEiOiJjbHNqbW9rZzgycHhvMmtzYmp5eTJwNnJ5In0.OA2ad1hWPGAfF7QQOGURJQ";

  let container;
  let map;
  let zoomLevel;

  const locations = [
    {
      name: "Akron",
      coords: [-81.5241, 41.0814],
      description: "Birthplace of LeBron James",
      color: "#006400",
    },
    {
      name: "Akron",
      coords: [-81.5241, 41.0814],
      description: "Birthplace of LeBron James",
      color: "#006400",
    },
    {
      name: "Cleveland",
      coords: [-81.6944, 41.4993],
      description: "First NBA Team",
      color: "#860038",
    },
    {
      name: "Miami",
      coords: [-80.1918, 25.7617],
      description: "Second NBA Team",
      color: "#98002E",
    },
    {
      name: "Cleveland",
      coords: [-81.6944, 41.4993],
      description: "Third NBA Team",
      color: "#860038",
    },
    {
      name: "Los Angeles",
      coords: [-118.2437, 34.0522],
      description: "Fourth NBA Team",
      color: "#FDB927",
    }
  ];

  let currentLocation = locations[0];

  function updateZoomLevel() {
    const screenWidth = window.innerWidth/2;
    zoomLevel = screenWidth <= 600 ? 4 : 5.85;
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
        center: locations[0].coords,
        zoom: zoomLevel,
        attributionControl: true,
      });

      window.addEventListener("resize", handleResize);

      map.on("load", () => {
        if (index && index % 2 == 0) {
          drawPoint(currentLocation);
        }
      });
    });

  function drawPoint(location) {
    new mapboxgl.Marker().setLngLat(location.coords).addTo(map);
  }

  function drawJourneyLine() {
    if (index <= 1) return;
    index = Math.min(index, locations.length - 1);
    const from = locations[index - 1].coords;
    const to = locations[index].coords;
    let color = locations[index].color;

    const midPointLat = (from[1] + to[1]) / 2;
    const midPointLng = (from[0] + to[0]) / 2;

    const curveAdjustment = 2;
    const midPointLngAdjusted =
      midPointLng + (index % 2 === 0 ? curveAdjustment : -curveAdjustment);

    const curvedCoordinates = [];
    for (let i = 0; i <= 100; i++) {
      const t = i / 100;
      const lat =
        from[1] * (1 - t) * (1 - t) +
        midPointLat * 2 * (1 - t) * t +
        to[1] * t * t;
      const lng =
        from[0] * (1 - t) * (1 - t) +
        midPointLngAdjusted * 2 * (1 - t) * t +
        to[0] * t * t;
      curvedCoordinates.push([lng, lat]);
    }

    const lineData = {
      type: "Feature",
      properties: {},
      geometry: {
        type: "LineString",
        coordinates: [],
      },
    };

    if (map.getSource("journeyLine")) {
      map.getSource("journeyLine").setData(lineData);
    } else {
      map.addSource("journeyLine", {
        type: "geojson",
        data: lineData,
      });

      map.addLayer({
        id: "journeyLine",
        type: "line",
        source: "journeyLine",
        layout: {
          "line-join": "round",
          "line-cap": "round",
        },
        paint: {
          "line-color": color,
          "line-width": 4,
        },
      });
    }

    map.setPaintProperty("journeyLine", "line-color", color);

    let i = 0;
    const speed = 3;
    function animateLine() {
      if (i < curvedCoordinates.length) {
        for (let j = 0; j < speed && i < curvedCoordinates.length; j++, i++) {
          lineData.geometry.coordinates.push(curvedCoordinates[i]);
        }
        map.getSource("journeyLine").setData(lineData);
        i++;
        requestAnimationFrame(animateLine);
      }
    }

    animateLine();
  }

  $: if (map && map.isStyleLoaded() && index) {
    if (index > locations.length - 1) {
      index = locations.length - 1;
    }
    currentLocation = locations[index];
    map.flyTo({ center: currentLocation.coords, zoom: zoomLevel });
    drawJourneyLine();
    drawPoint(currentLocation);
  }
</script>

<svelte:head>
  <link
    rel="stylesheet"
    href="https://api.mapbox.com/mapbox-gl-js/v2.14.0/mapbox-gl.css"
  />
</svelte:head>

<div class="map" bind:this={container} />

<style>
  .map {
    width: 50vw;
    height: 40vh; 
    position: absolute; 
    transform: translate(3%, 140%);
    border: black 2px solid;
    border-radius: 5px;
  }
</style>
