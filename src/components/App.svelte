<script>
import Scroller from "@sveltejs/svelte-scroller";
import * as d3 from 'd3';
import LLN from "./LLN.svelte"
import { onMount } from 'svelte';

let count, index, offset, progress;

let LLN_sec, height_LLN, width_LLN

onMount(() => {
        // Update dimensions when the component is mounted
        width_LLN = LLN_sec.clientWidth;
        height_LLN = LLN_sec.clientHeight;

        // Update dimensions when the window is resized
        const updateDimensions = () => {
            width_LLN = LLN_sec.clientWidth;
            height_LLN = LLN_sec.clientHeight;
        };

        window.addEventListener('resize', updateDimensions);

        return () => {
            window.removeEventListener('resize', updateDimensions);
        };
    });
</script>

<Scroller
  top={0.0}
  bottom={1}
  threshold={0.5}
  bind:count
  bind:index
  bind:offset
  bind:progress
>

<div class="background" slot="background">
  <div class="progress-bars">
    <p>current section: <strong>{index + 1}/{count}</strong></p>
    <progress value={count ? (index + 1) / count : 0} />

    <p>offset in current section</p>
    <progress value={offset || 0} />

    <p>total progress</p>
    <progress value={progress || 0} />
  </div>
</div>

<div class="foreground" slot="foreground">
  <section>
    Section 0: Introduction and CLT Demo (Caleb) 
  </section>
  <section bind:this={LLN_sec}>
    Section 1: Law of Large Numbers (Caleb) 
    {width_LLN} pixels by {height_LLN} pixels
    <LLN svg_width={width_LLN} svg_height={height_LLN} />
  </section>
  <section>Section 2: De Moivre Coin Flip Stuff (Kelo) </section>
  <section>Section 3: Laplace pi in the formula demo (Kelo) </section>
</div>

</Scroller>


<style>
  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    outline: green solid 3px;
  }

  .foreground {
    width: 70%;
    margin: 0 auto;
    height: auto;
    position: relative;
    outline: red solid 3px;
  }

  .progress-bars {
    position: absolute;
    background: rgba(170, 51, 120, 0.4);
    visibility: visible;
  }

  section {
    height: 80vh;
    background-color: rgba(0, 0, 0, 0.2); /* 20% opaque */
    /* color: white; */
    outline: magenta solid 3px;
    text-align: center;
    max-width: 750px; /* adjust at will */
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
  }
</style>