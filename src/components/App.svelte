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
    <!-- <button on:click={unlockWebpage}> Bypass Introduction (for testing)</button> -->
  </section>
  {#if unlocked}
  <section style="height: auto">
    <h2> 1713: Jakob Bernoulli's Law of Large Numbers </h2> <br>
    <div class='square'>
      <div><img src='images/jakob_bernoulli.jpg' alt='A Photograph of Jakob Bernoulli' width=200vw/></div>
      <p>The Law of Large numbers states that the average value of a sample, or a set of data, will eventually converge to whatever the true average value is for that distribution. This property was first noted as early as 1501, by Italian mathematician Gerolamo Cordeno. However, the first formal proof of the Law was developed by Jakob Bernoulli over the course of twenty plus years. In 1713, it was published in his pioneering work <i>Ars Conjectandi</i>, or the Art of Conjecturing. </p>
      <p>Bernoulli's version of the Law of Large Numbers applied only to Bernoulli trials, or flips of a weighted coin - a pattern we'll see repeated as we build up to the central limit theorem. In essence, it stated the following: Take a coin, with a probability of landing on heads <i>p</i>. Flip it some number of times, and calculate the percentage of times it landed on heads. Eventually, if you're willing to flip the coin a near-infinite number of times, you can guarantee that the margin of error between the percentage of times it landed heads and the true probability <i>p</i> is as small as you want. For example, you can guarantee that the percentage you compute is only one-millionth away from the true mean - you just might need to flip the coin billions of times.</p>
      <p>Much later, a mathematician named Siméon Denis Poisson, perhaps most famous for the Poisson distribution, would prove that the same property is true for any random variable, not just coin flips. You can try this out in the simulation below: pick a distribution, and start sampling. You'll see that the sample mean always eventually gets close to the true mean. </p>
    </div>
  </section>
  <section bind:this={LLN_sec} style="height: 80vh">
    <LLN svg_width={width_LLN} svg_height={height_LLN} />
  </section>
  <section>
    <h2> 1733: De Moivre's Coin Flips </h2>
    <div class='square'>
    <div><img src='images/abraham_de_moivre.jpg' alt='A Photograph of Abraham De Moivre' width=200vw height=254vw /></div>
     <p>Great! Now we know that, for a infinite number of samples, the true mean converges to the sample mean. Unfortunately, infinite samples is a lot. In fact, it's impossible to take an infinite samples today, so we might just need to settle for "a lot" of samples. But without infinite samples we can't guarantee that our sample mean matches the true one - so how likely is it to at least be "pretty close" to the true mean?</p>
     <p>This is the question that French mathematician Abraham de Moivre sought to answer in his work <i>Approximatio ad summam terminorum binomii (a+b)<sup>n</sup> in seriem expansi</i>. Only in the context of a <b>fair</b> coin flip, he approximated the probability that the number of heads would be <i>i</i> flips away from the expected number of heads based on the probability distribution. In doing so, De Moivre was the first mathematician to allude to the general principles of the CLT.</p>
     <p>Some of the things De Moivre did look shockingly close to today's version of the CLT. His calculation is essentially correct, but because of differences in mathematical notation at the time, all of the integrals and exponentials were written in terms of series expansion. Additionally, De Moivre's goal wasn't to find the "distribution" of sample means at all, especially not at a limit, but just to be able to approximate the probability that the resulting number of heads were close to the expected, true proportion. The real work that defined the CLT would come later.</p>
  </div>
  </section>
  <section>
    <h2> 1800s-1850s: Laplace to Cauchy </h2>
    <div class='square'>
    <img src='images/pierre_simon_laplace.jpg' alt='A Painting of Pierre-Simon Laplace' style='float: left;' width='200vw'/>
    <img src='images/augustin_louis_cauchy.jpg' alt='A Photograph of Augustin-Louis Cauchy'  width='200vw' />
    <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nam dapibus purus vitae nibh auctor, id dapibus magna porta. Praesent efficitur velit eu iaculis consequat. Donec rutrum dui ut justo pretium volutpat. Etiam id dictum sem. Aliquam quis lorem justo. Morbi rutrum sed quam quis sagittis. Fusce tempor volutpat aliquam. Vestibulum fringilla iaculis augue vel ullamcorper. Fusce at pharetra orci. Vivamus id diam dolor. Sed id turpis nec enim tempus porta eget sit amet ligula.</p>
    </div>
    <Formula />
    </section>
  <section>
    <h2> 1850s-1900s: Chebyshev and his students </h2>
    <div class='square'>
      <img src='images/chebyshev.png' alt='A Painting of Chebyshev' style='float:left;' width=200vw/>
      <img src='images/aleksandr_lyapunov.jpg' alt='A Photograph of Aleksandr Lyapunov' width=200vw/>
      <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nullam sed turpis faucibus, ullamcorper dolor non, fringilla metus. Aliquam erat volutpat. Aenean at accumsan sem. Curabitur ac erat dolor. Mauris ut scelerisque sapien. Cras pellentesque vulputate arcu. Ut sodales eros ac purus sagittis, sed sodales lorem tempor.</p>
      <p>Fusce interdum posuere tempor. Maecenas a molestie est. Nulla a sem pharetra, porta tortor eu, tempor quam. Quisque pretium erat arcu, nec rutrum turpis dictum et. Suspendisse interdum erat id lobortis ullamcorper. Proin mi leo, convallis convallis pellentesque a, porttitor quis elit. Vestibulum quam nibh, porttitor et tempor a, consectetur nec mi. Nam id.</p>
    </div>
  </section>
  <section>
    <h2> Acknowledgements </h2>
    <ul>
    <li> Thank you to Sam Lau for teaching and all the TAs for helping to run the course! </li>
    <li> Motivated by Hans Fischer's book "<a href="https://link.springer.com/book/10.1007/978-0-387-87857-7">History of the Central Limit Theorem</a>" (Chapters 1 and 2) </li>
    <li> Inspired by Grant Sanderson's (3Blue1Brown on Youtube) Video "<a href="https://www.youtube.com/watch?v=zeJD6dqJ5lo&t=975s&pp=ygUPM2JsdWUxYnJvd24gY2x0">But What is the Central Limit Theorem?</a>" </li>
    <li> Dataset used for Movie Reviews: <p style='text-indent:-20px; padding-left:20px; margin:0;'> Harper, F. Maxwell, and Joseph A. Konstan. &quot;The movielens datasets: History and context.&quot;&nbsp;<em>Acm transactions on interactive intelligent systems (tiis)</em>&nbsp;5.4 (2015): 1-19.</p></li>
    <li>
      Implementation References: 
      <ul>
      <li> <a href='https://github.com/ErnaneJ/svelte-star-rating'>svelte-star-rating</a> by <a href='https://github.com/ErnaneJ'>Ernane Ferreria</a></li>
      <li> <a href='https://blog.devgenius.io/d3-js-visualize-a-mathematical-function-d0c164ddcf2c'>D3.Js: Visualize a Matematical Function</a> by <a href='https://medium.com/@adyaksa.w'>Adyaksa W</a></li>
     </ul>
    </li>
    </ul>
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

  li {
    text-align: left;
  }

  img {
    float: right;
    margin: 5px;
    padding: 10px;
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