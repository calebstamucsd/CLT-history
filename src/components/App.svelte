<script>
import Scroller from "@sveltejs/svelte-scroller";
import * as d3 from 'd3';
import LLN from "./LLN.svelte"
import CLTHook from "./CLT_hook.svelte"
import Formula from "./Formula.svelte"
import { onMount } from 'svelte';

let count, index, offset, progress;

let LLN_sec, height_LLN, width_LLN;

let sectionHeight = '80vh';

let unlocked = false;

// Extra comment so github actions will activate

function unlockWebpage(event) {
  unlocked = true;

  // Update dimensions when the component is mounted
  width_LLN = LLN_sec.clientWidth;
  height_LLN = LLN_sec.clientHeight;

  const section = document.querySelector('section');
  if (section) {
    sectionHeight = `${section.scrollHeight}px`;
  }

  // Update dimensions when the window is resized
  const updateDimensions = () => {
      width_LLN = LLN_sec.clientWidth;
      height_LLN = LLN_sec.clientHeight;
  };

  window.addEventListener('resize', updateDimensions);

  return () => {
      window.removeEventListener('resize', updateDimensions);
  };
}


</script>

<title> The Central Limit Theorem: A Walk Through History </title>

<Scroller
  top={0.0}
  bottom={1}
  threshold={0.5}
  bind:count
  bind:index
  bind:offset
  bind:progress
>

<div class="background" slot="background" style="height: 100vh;">
  <!-- <div class="progress-bars">
    <p>current section: <strong>{index + 1}/{count}</strong></p>
    <progress value={count ? (index + 1) / count : 0} />

    <p>offset in current section</p>
    <progress value={offset || 0} />

    <p>total progress</p>
    <progress value={progress || 0} />
  </div> -->
</div>

<div class="foreground" slot="foreground">
  <div class="vertical-space"></div>
  <section style="height: auto">
    <h1> The Central Limit Theorem: A Walk Through History </h1>
    <br>
    <h4> By Caleb Stam and Kelo Komesu</h4> 
    <h4 class='date'> March 16, 2024 </h4>
    <br>
    <CLTHook on:message={unlockWebpage} />
    <br>
    <button on:click={unlockWebpage}> Bypass Introduction (for testing)</button>
  </section>
  {#if unlocked}
  <section style="height: auto">
    <h2> 1713: Bernoulli's Law of Large Numbers </h2> <br>
    <p> (Wikipedia Text as Placeholder) In probability theory, the law of large numbers (LLN) is a mathematical theorem that states that the average of the results obtained from a large number of independent and identical random samples converges to the true value, if it exists. More formally, the LLN states that given a sample of independent and identically distributed values, the sample mean converges to. </p>
    <p> The Italian mathematician Gerolamo Cardano (1501–1576) stated without proof that the accuracies of empirical statistics tend to improve with the number of trials. This was then formalized as a law of large numbers. A special form of the LLN (for a binary random variable) was first proved by Jacob Bernoulli. It took him over 20 years to develop a sufficiently rigorous mathematical proof which was published in his Ars Conjectandi (The Art of Conjecturing) in 1713. He named this his "Golden Theorem" but it became generally known as "Bernoulli's theorem". This should not be confused with Bernoulli's principle, named after Jacob Bernoulli's nephew Daniel Bernoulli. In 1837, S. D. Poisson further described it under the name "la loi des grands nombres" ("the law of large numbers"). Thereafter, it was known under both names, but the "law of large numbers" is most frequently used. </p>
    <p> After Bernoulli and Poisson published their efforts, other mathematicians also contributed to refinement of the law, including Chebyshev, Markov, Borel, Cantelli, Kolmogorov and Khinchin. Markov showed that the law can apply to a random variable that does not have a finite variance under some other weaker assumption, and Khinchin showed in 1929 that if the series consists of independent identically distributed random variables, it suffices that the expected value exists for the weak law of large numbers to be true. These further studies have given rise to two prominent forms of the LLN. One is called the "weak" law and the other the "strong" law, in reference to two different modes of convergence of the cumulative sample means to the expected value; in particular, as explained below, the strong form implies the weak. </p>
  </section>
  <section bind:this={LLN_sec} style="height: 80vh">
    <LLN svg_width={width_LLN} svg_height={height_LLN} />
  </section>
  <section>
    <h2> 1733: De Moivre's Coin Flips </h2>
  </section>
  <section>
    <h2> 1812: Laplace's Approximation </h2>
    <Formula />
  </section>
  <section>
    <h2> 1850s-1900s: Cauchy and the Russians </h2>
  </section>
  {/if}
</div>

</Scroller>

<style>
  .background {
    width: 100vw;
    position: relative;
    /* outline: green solid 3px; */
    background: rgba(245, 245, 220, 1);
  }

  .foreground {
    width: 70%;
    margin: 0 auto;
    height: auto;
    position: relative;
    /* outline: red solid 3px; */
  }

  .progress-bars {
    position: absolute;
    background: rgba(170, 51, 120, 0.4);
    visibility: visible;
  }

  * {
    font-family: 'Georgia', serif;
  }

  .vertical-space {
        height: 30vh;
  }

  h1 {
    width: 75%;
    font-family: 'Garamond', serif;
    font-size: 48px;
  }

  h2 {
    width: 75%;
    font-family: 'Garamond', serif;
    font-size: 36px;
  }

  h4 {
    font-family: 'Franklin', sans-serif;
    display: inline;
    font-weight: 700;
  }

  .date {
    font-weight: 50;
    color: rgba(0, 0, 0, 0.5)
  }

  p {
    text-align: justify;
    text-indent: 50px;
  }

  section {
    height: auto;
    /* background-color: rgba(0, 0, 0, 0.2); */
    /* color: white; */
    /* outline: magenta solid 3px; */
    text-align: center;
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
    align-items: center;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  section > * {
            margin: 0em 0; /* Adjust the value to reduce spacing */
        }
</style>