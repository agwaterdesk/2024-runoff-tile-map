<script>
  import { onMount } from "svelte";
  import mapboxgl from "mapbox-gl";
  import "mapbox-gl/dist/mapbox-gl.css";
  import { extent } from "d3-array";
  import { scaleSqrt } from "d3-scale";
  import Wherewolf from "wherewolf";
  import Legend from "./Legend.svelte";

  import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";
  import "@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css";

  import topo from "../data/topo.json";

  export let overlayInfo;

  let map;
  let marker = null;

  var teenWolf = Wherewolf();
  teenWolf.addAll(topo);

  let bounds = [
    [-99.62083, 35.26577], // southwest corner
    [-77.89919, 49.05199], // northeast corner
  ];
  mapboxgl.accessToken = import.meta.env.VITE_MAPBOX_TOKEN;

  onMount(() => {
    map = new mapboxgl.Map({
      container: "map",
      style: "mapbox://styles/agwaterdesk/clweai28a020601qgfxih2hj2/draft",
    });

    map.fitBounds(bounds, { duration: 0 });
    map.setMaxBounds(bounds);

    const geocoder = new MapboxGeocoder({
      accessToken: mapboxgl.accessToken,
      mapboxgl: mapboxgl,
      countries: "us",
      placeholder: "Zoom to location",
      marker: false,
      bbox: bounds.flat(),
    });

    map.addControl(geocoder, "top-left");

    map.addControl(new mapboxgl.NavigationControl());

    function getTopoData(coords) {
      return teenWolf.find(coords);
    }

    map.on("load", () => {
      // Assuming the raster source is already in your style

      map.on("click", (e) => {
        let { lng, lat } = e.lngLat;
        let res = getTopoData([lng, lat]);
        if (Object.values(res).some((value) => value)) {
          overlayInfo = res;
        } else {
          overlayInfo = undefined;
        }
        // Remove the existing marker if it exists
        if (!overlayInfo || marker) {
          marker.remove();
        }

        if (overlayInfo) {
          // Create a new marker and add it to the map
          marker = new mapboxgl.Marker().setLngLat([lng, lat]).addTo(map);
        }
      });
    });

    geocoder.on("result", ({ result }) => {
      let coords = result.geometry.coordinates;
      getTopoData(coords);

      map.flyTo({
        center: coords,
        zoom: 10,
        essential: true,
      });
    });
  });

  function updateMap() {}
</script>

<div>
  <div class="controls">
    <div class="toggle">
      <span class="dek">Size waste water treatment plants by</span>
    </div>
  </div>

  <!-- <div bind:this={geocoderContainer} class="geocoder" /> -->

  <div id="map"></div>
</div>

<style lang="scss">
  #map {
    width: 100%;
    height: 800px;
  }

  .controls {
    display: flex;
    align-items: start;
    gap: 2rem;
    margin-bottom: 1rem;

    .toggle {
    }
  }

  button {
    background: transparent;
    border: 2px solid transparent;
    position: relative;
    padding: 3px 6px;
    font-weight: bold;
    cursor: pointer;

    &:hover {
      &::before {
        opacity: 0.5;
      }
    }

    &.active {
      border: 2px solid;
      &::before {
        opacity: 0.5;
      }
    }

    &::before {
      content: "";
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: var(--color);
      opacity: 0.3;
      z-index: -1;
    }
  }

  :global {
    .mapboxgl-ctrl-geocoder {
      box-shadow: none;
      border: 1px solid #666;

      .mapboxgl-ctrl-geocoder--input {
        font-family: var(--font-sans);
      }
    }

    .mapboxgl-popup .mapboxgl-popup-content {
      filter: drop-shadow(2px 2px 4px #00000050);
      font: var(--font-sans);
    }
  }

  :global {
    .g-popup {
      h3 {
        font-size: 24px;
      }
    }
  }
</style>
