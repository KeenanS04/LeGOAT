<script>
  import mapboxgl from "mapbox-gl";
  import { onMount } from "svelte";

  mapboxgl.accessToken = "pk.eyJ1IjoiYW11anJhbCIsImEiOiJjbHNqbW9rZzgycHhvMmtzYmp5eTJwNnJ5In0.OA2ad1hWPGAfF7QQOGURJQ";

  let mapContainer;

  const locations = [
    {
      coords: [-81.5241, 41.0814],
      description: `Akron: Birthplace and high school
- 3× Ohio Mr. Basketball (2001–2003)
- 2× Gatorade National Basketball Player of the Year (2002, 2003)
- 2× USA Today All-USA First Team (2002, 2003)
- Parade All-American First Team (2002, 2003)
- McDonald's All-American Game MVP (2003)
- National High School Player of the Year (2003)`,
      imageURL: "images/svsm.png",
      color: "#006400",
      iconSize: 0.15,
    },
    {
      coords: [-81.6944, 41.4993],
      description: `Cleveland Cavaliers (First Stint)
- Rookie of the Year (2004)
- 2× NBA Most Valuable Player (MVP) (2009, 2010)
- NBA Eastern Conference Champion (2007)
- NBA All-Star Game MVP (2006, 2008)
- Multiple NBA All-Star selections
- NBA Scoring Champion (2008)`,
      imageURL: "images/Cleveland_Cavaliers_logo.svg.png",
      color: "#98002E",
      iconSize: 0.05,
    },
    {
      coords: [-80.1918, 25.7617],
      description: `Miami Heat
- 2× NBA Champion (2012, 2013)
- 2× NBA Most Valuable Player (MVP) (2012, 2013)
- 4× NBA Eastern Conference Champion (2011–2014)
- NBA Finals MVP (2012, 2013)
- Selected to the NBA All-Star team each year
- Multiple All-NBA First Team selections`,
      imageURL: "images/Miami-Heat-logo.png",
      color: "#860038",
      iconSize: 0.1
    },
    {
      coords: [-81.6944, 41.4993],
      description: `Return to Cleveland Cavaliers
- NBA Champion (2016)
- NBA Eastern Conference Champion (2015–2018)
- Multiple NBA All-Star selections
- NBA Finals MVP (2016)`,
      imageURL: "images/Cleveland_Cavaliers_logo.svg.png",
      color: "#FDB927",
      iconSize: 0.05,
    },
    {
      coords: [-118.2437, 34.0522],
      description: `Los Angeles Lakers
- NBA Champion (2020)
- NBA Finals MVP (2020)
- Selected to the NBA All-Star team each year
- Multiple All-NBA First Team selections`,
      imageURL: "images/lakers-transparent copy.png",
      color: "#FDB927",
      iconSize: 0.1,
    }
  ];
  
  onMount(() => {
    const map = new mapboxgl.Map({
      container: mapContainer,
      style: 'mapbox://styles/mapbox/light-v10',
      center: [-97.2263, 37.7091],
      zoom: 3.5,
      minZoom: 3.5
    });

    map.scrollZoom.disable();
    map.dragPan.disable();
    map.dragRotate.disable();
    map.keyboard.disable();
    map.doubleClickZoom.disable();
    map.touchZoomRotate.disable();

    map.on('load', async () => {
      for (let i = 0; i < locations.length - 1; i++) {
        const startLocation = locations[i];
        const endLocation = locations[i + 1];
        await animateLineBetweenPoints(map, startLocation, endLocation, i);
      }
      for (let i = 0; i < locations.length; i++) {
        await new Promise((resolve, reject) => {
          map.loadImage(locations[i].imageURL, (error, image) => {
            if (error) {
              console.error('Error loading image:', error);
              reject(error);
            }

            map.addImage(`locationImage${i}`, image);

            map.addSource(`point${i}`, {
              type: 'geojson',
              data: {
                type: 'FeatureCollection',
                features: [{
                  type: 'Feature',
                  properties: {
                    description:locations[i].description,
                  },
                  geometry: {
                    type: 'Point',
                    coordinates: locations[i].coords
                  }
                }]
              }
            });

            map.addLayer({
              id: `pointLayer${i}`,
              type: 'symbol',
              source: `point${i}`,
              layout: {
                'icon-image': `locationImage${i}`,
                'icon-size': location.iconSize || 0.15,
                'icon-allow-overlap': true,
                'icon-ignore-placement': true
              }
            });

            map.on('click', `pointLayer${i}`, (e) => {
              new mapboxgl.Popup()
                .setLngLat(locations[i].coords)
                .setHTML(locations[i].description)
                .addTo(map);
            });

            map.on('mouseenter', `pointLayer${i}`, () => {
              map.getCanvas().style.cursor = 'pointer';
            });

            map.on('mouseleave', `pointLayer${i}`, () => {
              map.getCanvas().style.cursor = '';
            });

            resolve();
            resolve();
          });
        });
      }
    });
  });

  function animateLineBetweenPoints(map, startLocation, endLocation, index) {
    const start = startLocation.coords;
    const end = endLocation.coords;
    const color = startLocation.color;

    const midPoint = [
      (start[0] + end[0]) / 2, 
      (start[1] + end[1]) / 2 + 2
    ];

    let t = 0;
    const speed = 0.05;
    const coordinates = [start];

    function draw() {
      if (t <= 1) {
        t += speed;

        const xt = (1 - t) * (1 - t) * start[0] + 2 * (1 - t) * t * midPoint[0] + t * t * end[0];
        const yt = (1 - t) * (1 - t) * start[1] + 2 * (1 - t) * t * midPoint[1] + t * t * end[1];
        coordinates.push([xt, yt]);

        if (map.getSource(`journeyLine${index}`)) {
          map.getSource(`journeyLine${index}`).setData({
            'type': 'Feature',
            'properties': {},
            'geometry': {
              'type': 'LineString',
              'coordinates': coordinates
            }
          });
        } else {
          map.addSource(`journeyLine${index}`, {
            'type': 'geojson',
            'data': {
              'type': 'Feature',
              'properties': {},
              'geometry': {
                'type': 'LineString',
                'coordinates': coordinates
              }
            }
          });

          map.addLayer({
            'id': `journeyLine${index}`,
            'type': 'line',
            'source': `journeyLine${index}`,
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

        requestAnimationFrame(draw);
      }
    }

    draw();
  }
</script>

<svelte:head>
  <link href="https://api.mapbox.com/mapbox-gl-js/v2.14.0/mapbox-gl.css" rel="stylesheet">
</svelte:head>

<div bind:this={mapContainer} class="map"></div>

<style>
  .map {
    width: 100%;
    height: 500px;
    border: 1px solid black;
    border-radius: 5px;
  }
</style>