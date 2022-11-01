<script>
  import { onMount, afterUpdate } from "svelte";

  import { scaleDivergingLog, scaleLinear, scaleLog } from "d3-scale";
  import { interpolateRgbBasis } from "d3-interpolate";

  const d3 = { interpolateRgbBasis, scaleDivergingLog, scaleLinear, scaleLog };

  export let colors = ["#ff0000", "#0000ff"];
  export let domain = [-1, 1];
  export let barHeight = 20;
  // export let hoveredRatio = null

  onMount(() => {});

  afterUpdate(() => {});

  // let sl = d3.scaleDivergingLog().domain(domain)
  // $: vertTick = hoveredRatio ? sl.(hoveredRatio) * 100 : null
</script>

<div class="rounded overflow-hidden">
  <svg width="100%" height={barHeight}>
    <defs>
      <linearGradient id="myGradient">
        {#each colors as col, i}
          <!-- this calculation is NOT!! right -->
          <stop offset="{i * (100 / colors.length + 2)}%" stop-color={col} />
        {/each}
      </linearGradient>
    </defs>
    <g>
      <rect
        x="0"
        y="0"
        width="100%"
        height={barHeight}
        fill="url('#myGradient')"
      />
      <!-- {#if vertTick}
					<rect
						x="{vertTick}%"
						y="0"
						width="10px"
						height={barHeight}
						fill="black" />
				{/if} -->
    </g>
  </svg>
</div>

<style lang="scss"></style>
