<script>
  import { onMount } from "svelte";
  import mapboxgl from "mapbox-gl";
  import "mapbox-gl/dist/mapbox-gl.css";
  import bbox from "@turf/bbox";
  import Wherewolf from "wherewolf";

  import * as topojson from "topojson-client";

  import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";
  import "@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css";

  import topo from "../data/topo.json";

  export let overlayInfo = undefined;
  export let marker;

  let map;

  var states = topojson.feature(topo, topo.objects.State);

  var teenWolf = Wherewolf();
  teenWolf.addAll(topo);

  let bounds = bbox(states);

  mapboxgl.accessToken = import.meta.env.VITE_MAPBOX_TOKEN;

  onMount(() => {
    map = new mapboxgl.Map({
      container: "map",
      style: "mapbox://styles/agwaterdesk/clwp3ilyt00ki01qe6skw05px",
    });

    map.fitBounds(bounds, { duration: 0 });
    map.setMaxBounds(bounds);

    const geocoder = new MapboxGeocoder({
      accessToken: mapboxgl.accessToken,
      mapboxgl: mapboxgl,
      countries: "us",
      placeholder: "Zoom to location",
      marker: false,
      bbox: bounds,
    });

    map.addControl(geocoder, "top-left");

    map.addControl(new mapboxgl.NavigationControl());

    // Add hide-in-static class to controls
    document
      .querySelector(".mapboxgl-control-container")
      .classList.add("hide-in-static");



    function getTopoData(coords) {
      return teenWolf.find(coords);
    }

    map.on("load", () => {
      // Add the GeoJSON data as a new source
      map.addSource("states", {
        type: "geojson",
        data: states,
      });

      // Add a layer to visualize the states
      map.addLayer({
        id: "states-layer",
        type: "line",
        source: "states",
        paint: {
          "line-color": "#333", // Black color
          "line-width": 0.5, // 1px stroke
        },
      });

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
</script>

<div>
  <div id="map"></div>
</div>

<style lang="scss">
  #map {
    width: 100%;
    height: 800px;

    @include mq(600px, "max-width") {
      height: 600px;
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
      border-radius: 4px;
      box-shadow: 0 0 0 2px rgba(0, 0, 0, 0.1);

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
