<script>
  import Window from "./components/Window.svelte";
  import Map from "./components/Map.svelte";
  import Overlay from "./components/Overlay.svelte";
  import { MousePointer } from "lucide-svelte";

  // Handle responsive iframes for embeds
  import pym from "pym.js";

  new pym.Child({
    polling: 500,
  });

  function getUrlParameter(name) {
    const params = new URLSearchParams(window.location.search);
    return params.get(name);
  }

  let includeCredit = getUrlParameter("credit") != "false";

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
  <h1 class="headline">Subsurface drainage pervades Midwest agriculture</h1>
  <p class="dek">
    The map shows agricultural areas in the Midwest that are likely to have been
    drained for crop production, usually through subsurface tile drainage,
    according to a <a
      href="https://transformingdrainage.org/tools/drained-area-tool/"
      target="_blank">2022 study</a
    >. Soils are classified as <span class="highlight likely">likely</span>,
    <span class="highlight potentially">potentially</span>
    or <span class="highlight unlikely">unlikely</span> to be drained based on
    their natural drainage conditions and the likelihood of having artificial
    drainage to support agriculture.
    <span class="hide-in-static"
      ><MousePointer size="14" />Click on a location to see drainage details by
      watershed, county and state.</span
    >
  </p>

  <p class="sr-only">
    The map shows agricultural areas in the Midwest that are likely to
    have been drained for crop production, typically using subsurface tile
    drainage. It is based on a 2022 study and classifies soils into three
    categories: likely to be drained (blue), potentially drained (purple), and
    unlikely to be drained (white).
  </p>

  <div id="g-viz">
    <Overlay {overlayInfo} {reset} />
    <Map bind:overlayInfo bind:marker />
  </div>

  {#if includeCredit}
    <div class="credit">
      Data: <a
        target="_blank"
        href="https://purr.purdue.edu/publications/3966/1"
        >Purdue University Research Repository</a
      >; Graphic by Jared Whalen /
      <a target="_blank" href="https://agwaterdesk.org/">Ag & Water Desk</a>
    </div>
  {/if}
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
