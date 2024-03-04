<script>
import Scroller from "@sveltejs/svelte-scroller";
import * as d3 from 'd3';
import LLN from "./LLN.svelte"
import CLTHook from "./CLT_hook.svelte"
import { onMount } from 'svelte';

let count, index, offset, progress;

let LLN_sec, height_LLN, width_LLN;

let sectionHeight = '80vh';

// Extra comment so github actions will activate

onMount(() => {
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
    <CLTHook/>
  </section>
  <section style="height: 55vh">
    Section 1: Law of Large Numbers (Caleb) <br>
    <p> (Wikipedia Text as Placeholder) In probability theory, the law of large numbers (LLN) is a mathematical theorem that states that the average of the results obtained from a large number of independent and identical random samples converges to the true value, if it exists. More formally, the LLN states that given a sample of independent and identically distributed values, the sample mean converges to the true mean. </p>
    <p> The Italian mathematician Gerolamo Cardano (1501–1576) stated without proof that the accuracies of empirical statistics tend to improve with the number of trials. This was then formalized as a law of large numbers. A special form of the LLN (for a binary random variable) was first proved by Jacob Bernoulli. It took him over 20 years to develop a sufficiently rigorous mathematical proof which was published in his Ars Conjectandi (The Art of Conjecturing) in 1713. He named this his "Golden Theorem" but it became generally known as "Bernoulli's theorem". This should not be confused with Bernoulli's principle, named after Jacob Bernoulli's nephew Daniel Bernoulli. In 1837, S. D. Poisson further described it under the name "la loi des grands nombres" ("the law of large numbers"). Thereafter, it was known under both names, but the "law of large numbers" is most frequently used. </p>
    <p> After Bernoulli and Poisson published their efforts, other mathematicians also contributed to refinement of the law, including Chebyshev, Markov, Borel, Cantelli, Kolmogorov and Khinchin. Markov showed that the law can apply to a random variable that does not have a finite variance under some other weaker assumption, and Khinchin showed in 1929 that if the series consists of independent identically distributed random variables, it suffices that the expected value exists for the weak law of large numbers to be true. These further studies have given rise to two prominent forms of the LLN. One is called the "weak" law and the other the "strong" law, in reference to two different modes of convergence of the cumulative sample means to the expected value; in particular, as explained below, the strong form implies the weak. </p>
  </section>
  <section bind:this={LLN_sec} style="height: {sectionHeight}">
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

  * {
    font-family: 'Nunito', sans-serif;
  }

  p {
    text-align: justify;
    text-indent: 50px;
  }

  section {
    height: 80vh;
    background-color: rgba(0, 0, 0, 0.2); /* 20% opaque */
    /* color: white; */
    outline: magenta solid 3px;
    text-align: center;
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
  }
</style>