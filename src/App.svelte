<script>
  import Window from "./components/Window.svelte";
  import Map from "./components/Map.svelte";
  import Overlay from "./components/Overlay.svelte";

  // Handle responsive iframes for embeds
  import pym from "pym.js";

  new pym.Child({
    polling: 500,
  });

  let overlayInfo;
  let marker = null;

  function reset() {
    overlayInfo = undefined;
    marker.remove();
  }
</script>

<Window />
<!-- Outer div must have class 'chart-container' don't change -->
<div class="chart-container">
  <h1 class="headline">
    Mapping likely subsurface drainage in Midwest agriculture
  </h1>
  <p class="dek">
    The map shows agricultural areas in the Midwest that are likely to have been
    drained for crop production, usually through subsurface tile drainage. Soils
    are classified as
    <span class="highlight likely">likely</span>,
    <span class="highlight potentially">potentially</span>
    or <span class="highlight unlikely">unlikely</span> to be drained based on their
    natural drainage conditions and the necessity for artificial drainage in agriculture.
  </p>
  <!-- <p class="sr-only">[altText]</p> -->

  <div id="g-viz">
    <Overlay {overlayInfo} {reset} />
    <Map bind:overlayInfo bind:marker />
  </div>
</div>

<style lang="scss">
  .chart-container {
    max-width: 800px;
    width: 100%;
    padding: 1rem;
  }

  :root {
    --color-likely: #7fb6d9;
    --color-potentially: #dda3fa;
  }

  #g-viz {
    position: relative;
    overflow: hidden;
  }

  :global {
    .highlight {
      $size: 12px;
      margin-left: calc($size + 4px);
      position: relative;

      &::after {
        width: $size;
        height: $size;
        content: "";
        border: 1px;
        position: absolute;
        left: calc($size * -1 - 2px);
        top: 50%;
        transform: translateY(-50%);
        border: 1px solid;
        border-radius: 100%;
      }

      &.unlikely {
        &::after {
          background: #fff;
          border-color: #999;
        }
      }
      &.potentially {
        &::after {
          background: var(--color-potentially);
          border-color: var(--color-potentially);
        }
      }
      &.likely {
        &::after {
          background: var(--color-likely);
          border-color: var(--color-likely);
        }
      }
    }
  }
</style>
