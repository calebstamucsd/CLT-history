<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
    import { createEventDispatcher } from 'svelte';
    import StarRating from "@ernane/svelte-star-rating";
    import { base } from '$app/paths'

    // comment for github pages update

    const config = {
        readOnly: false,
        countStars: 5,
        range: {
            min: 0, 
            max: 5, 
            step: 0.5
        },
        score: 0.0,
        showScore: true,
        scoreFormat: function(){
            user_score = Math.max(((this.score * 2).toFixed(0) / 2), 0.5);
            return `(${user_score}/${this.countStars})`;
        },
        name: "",
        starConfig: {
            size: 30,
            fillColor: '#ffcf30',
            strokeColor: '#000000',
            unfilledColor: '#FFF',
            strokeUnfilledColor: '#000000'
        }
        }

    let dialog_0, dialog_1, dialog_2, dialog_3, dialog_4;
    let dialog_list = [dialog_0, dialog_1, dialog_2, dialog_3, dialog_4];
    let movie_choice, movie_data;
    let user_score = 0.5;
    let user_mean;

    const center_margin = {top: 50, right: 50, bottom: 50, left: 50},
    width = 250 - center_margin.left - center_margin.right,
    height = 200 - center_margin.top - center_margin.bottom;
    let xMax, yMax, xScale, yScale, xAxis, yAxis, bar_width, mean_bar_width;
    let meanxMin, meanxMax, meanxScale, meanyScale, meanxAxis, meanyAxis;
    let meanyMax = 5;
    let histogram, mean_hist;

    let samplingCount = 0;
    let slide_2_text_0 = '';
    let slide_2_text_1 = '';
    let slide_2_text_2 = '';
    let slide_2_text_3 = '';
    let slide_2_text_4 = '';
    let slide_2_button = false;

    let slide_3_text_1 = '';
    let slide_3_text_2 = '';
    let slide_3_text_3 = '';
    let gridSize = 6;

    let isMounted = false;

    const dispatch = createEventDispatcher();
    let show = false;

    function swapModal(idx_1, idx_2) {
        console.log(base)
        dialog_list[idx_1].close();
        dialog_list[idx_2].show();
        if(idx_2 == 2) {
            plot_histogram(49)
        }
        if(idx_2 == 3) {
            slide_3()
        }
    }

    function closeOut() {
        dialog_list[4].close()
        show=true;
        dispatch('message', show);
    }

    async function slide_3() {
        let mean_list = [user_mean];
        inner_plot(mean_hist, binList(mean_list, 0.1), mean_bar_width, meanxScale, meanyScale);
        await wait(1000);
        slide_3_text_1 = 'Now, we\'ll add more observations. Remember - each addition to this plot represents the AVERAGE of 50 ratings, not just one person\'s review.'
        for(let i = 5; i < 501; i += 5) {
            mean_hist.selectAll("g").remove()
            meanyMax = i;
            inner_plot(mean_hist, binList(mean_list, 0.1), mean_bar_width, meanxScale, meanyScale);
            await wait(50);
        }
        await wait(3000);
        for(let i = 0; i < 1000; i++) {
            mean_list.push(sample_mean(50))
            inner_plot(mean_hist, binList(mean_list, 0.1), mean_bar_width, meanxScale, meanyScale);
            await(wait(10));
        }
        slide_3_text_2 = "What an interesting curve! One might even say it looks bell-shaped..."
    }
    
    function binList(list, step) {
        let bin_dict = {}
        for (let i = 0; i < list.length; i += 1) {
            let data_val = list[i];
            let bin = (Math.floor(data_val / step) * step);
            if (bin_dict.hasOwnProperty(bin)) {
                bin_dict[bin] += 1
            }
            else {
                bin_dict[bin] = 1
            }
        }
        return Object.entries(bin_dict).map(([key, value]) => [key, value]);;
    }

    function sampleOne() {
        let data_values = Object.entries(movie_data).map(([key, value]) => value)
        let random_index = Math.floor(d3.randomUniform(0, data_values.reduce((acc, curr) => acc + curr, 0))())
        for (const key of Object.keys(movie_data).sort()) {
            const value = movie_data[key];
            random_index -= value;
            if(random_index < 0) {
                return parseFloat(key);
            }
        }
    }

    function clear_histogram(hist) {
        hist.selectAll("rect").remove()
    }

    function wait(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    function inner_plot(hist, data_list, my_bar_width, myXScale, myYScale) {
        clear_histogram(hist)
        hist.selectAll("rect")
                .data(data_list)
                .enter()
                .append("rect")
                .attr("x", d => myXScale(d[0]) - (my_bar_width/2))
                .attr("y", d => myYScale(d[1]))
                .attr("width", my_bar_width)
                .attr("height", d => height - myYScale(d[1]))
                .attr("fill", "#DA2A05")
                .attr("stroke", "black")
                .attr("stroke-width", 1);
    }


    function sample_mean(n) {
        let data_dict = {0.5:0, 1:0, 1.5:0, 2:0, 2.5:0, 3:0, 3.5:0, 4:0, 4.5:0, 5:0}
        for(let i = 0; i<n; i++) {
            let samp = sampleOne()
            data_dict[samp]++;
        }

        let mean;
        let sum = 0;
        let samps = 0;
        for (const key of Object.keys(data_dict).sort()) {
            const value = data_dict[key];
            sum += (key * value);
            samps += (value);
        }
        mean = (sum / samps);

        return mean;
    }

    async function plot_histogram(n) {
        await wait(800);
        slide_2_text_0 = 'Let\'s see what some other movie-goers thought about it.';
        let data_dict = {0.5:0, 1:0, 1.5:0, 2:0, 2.5:0, 3:0, 3.5:0, 4:0, 4.5:0, 5:0}
        for(let i = 0; i<n; i++) {
            let samp = sampleOne()
            data_dict[samp]++;

            let data_list = Object.entries(data_dict).map(([key, value]) => [key,value])
            data_list.unshift(0);
            inner_plot(histogram, data_list, bar_width, xScale, yScale);

            samplingCount += 1;
            await wait(50);
        }
        await wait(800);
        slide_2_text_1 = "And now...";
        await wait(1500);
        slide_2_text_2 = "we'll add you!";
        await wait(1000);
        data_dict[user_score]++;
        samplingCount++;

        let data_list = Object.entries(data_dict).map(([key, value]) => [key,value])
        data_list.unshift(0);
        inner_plot(histogram, data_list, bar_width, xScale, yScale);
        let sum = 0;
        let samps = 0;
        for (const key of Object.keys(data_dict).sort()) {
            const value = data_dict[key];
            sum += (key * value);
            samps += (value);
        }
        user_mean = (sum / samps)
        await wait(1500);
        slide_2_text_3 = `The average rating for the movie is ${user_mean.toFixed(2)} stars.`
        await wait(2500);
        slide_2_text_4 = "But wait. Doesn't our sample seem a little small? Let's take some more to get a better picture."
        await wait(2500);
        slide_2_button = false;
    }

    onMount(() => {
        d3.selectAll('img').on('click', function(event) {
            swapModal(0, 1)
            movie_choice = event.target.name;
            fetch(`${base}/src/components/assets/movie_ratings.json`)
                .then(response => response.json())
                .then(data => {
                    movie_data = data[movie_choice];
                })
                .then(() => {
                    xMax = 5;
                    yMax = 25;
                    xScale = d3.scaleLinear([0, xMax], [0, width])
                    yScale = d3.scaleLinear([0, yMax], [height, 0])
                    xAxis = d3.axisBottom(xScale)
                    yAxis = d3.axisLeft(yScale)
                    bar_width = xScale(0.5)
                    histogram = d3.select("#histogram").attr("width", width + center_margin.left + center_margin.right)
                        .attr("height", height + center_margin.top + center_margin.bottom)
                        .append("g")
                        .attr("transform", "translate(" + center_margin.left + "," + center_margin.top + ")");
                    histogram.append("clipPath")
                        .attr("id", "chart-area")
                        .append("rect")
                        .attr("x", 0)
                        .attr("y", 0)
                        .attr("width", width)
                        .attr("height", height)
                    histogram.append("g")
                        .attr("transform", `translate(0,${height})`)
                        .call(xAxis)
                    histogram.append("g")
                        .attr("transform", `translate(0,0)`)
                        .call(yAxis)
                    isMounted = true;
                }
            )
        })
    })

    $: show = show;
    $: if(isMounted) {
        meanxMin = 3;
        meanxMax = 5;
        meanxScale = d3.scaleLinear([meanxMin, meanxMax], [0, width])
        meanyScale = d3.scaleLinear([0, meanyMax], [height, 0])
        meanxAxis = d3.axisBottom(meanxScale)
        meanyAxis = d3.axisLeft(meanyScale)
        mean_bar_width = meanxScale(3.1)

        mean_hist = d3.select('#mean-hist').attr("width", width + center_margin.left + center_margin.right)
            .attr("height", height + center_margin.top + center_margin.bottom)
            .append("g")
            .attr("transform", "translate(" + center_margin.left + "," + center_margin.top + ")");
        mean_hist.append("clipPath")
            .attr("id", "chart-area")
            .append("rect")
            .attr("x", 0)
            .attr("y", 0)
            .attr("width", width)
            .attr("height", height)
        mean_hist.append("g")
            .attr("transform", `translate(0,${height})`)
            .call(meanxAxis)
        mean_hist.append("g")
            .attr("transform", `translate(0,0)`)
            .call(meanyAxis)
                    
    }
</script>

{#if !show}
<button class="button-30" on:click={() => (dialog_list[0].show())}> Start! </button>
{/if}

<dialog
	bind:this={dialog_list[0]}
>
<p>Pick a movie you've seen from the choices below. If you haven't seen any of them, that's okay - just play along!</p>
<div class='movie-container'>
    <div>
        The Lion King (1994)
        <br>
        <br>
        <img name="Lion King, The (1994)" src="images/the_lion_king.jpg" alt="The Lion King Movie Poster" width=140 height=210>
    </div>
    <div>
        Titanic (1997)
        <br>
        <br>
        <img name="Titanic (1997)" src="images/titanic.jpg" alt="Titanic Movie Poster" width=140 height=210>
    </div>
    <div>
        Spirited Away (2001)
        <br>
        <br>
        <img name="Spirited Away (Sen to Chihiro no kamikakushi) (2001)" src="images/spirited_away.jpg" alt="Spirited Away Movie Poster" width=140 height=210>
    </div>
    <div>
        Harry Potter and the Sorcerer's Stone (2001)
        <br>
        <img name="Harry Potter and the Sorcerer's Stone (a.k.a. Harry Potter and the Philosopher's Stone) (2001)" src="images/harry_potter.jpg" alt="Harry Potter and the Philosopher's Stone Movie Poster" width=140 height=210>
    </div>
    <div>
        The Avengers (2012)
        <br>
        <br>
        <img name="Avengers, The (2012)" src="images/the_avengers.jpg" alt="The Avengers Movie Poster" width=140 height=210>
    </div>
    <div>
        Parasite (2019)
        <br>
        <br>
        <img name="Parasite (2019)" src="images/parasite.jpg" alt="Parasite Movie Poster" width=140 height=210>
    </div>
</div>
<br>
<button class="button-30" on:click={() => swapModal(0,1)}>Next</button>
</dialog>

<dialog
	bind:this={dialog_list[1]}
>
<p>Give it a rating!</p>
<StarRating {config}/>
<br>
<button class="button-30" on:click={() => swapModal(1,2)}>Next</button>
</dialog>

<dialog
	bind:this={dialog_list[2]}
    style='height: auto'
>
<p>Great! You weren't the only person who reviewed this movie, however. </p>
<p>{slide_2_text_0}</p>
<p>N = {samplingCount}</p>
<svg id="histogram"></svg>
<p>{slide_2_text_1}{slide_2_text_2}</p>
<br>
<p>{slide_2_text_3}</p> <br>
<p>{slide_2_text_4}</p>
<button class="button-30" disabled={slide_2_button} on:click={() => swapModal(2,3)}>Next</button>
</dialog>

<dialog
	bind:this={dialog_list[3]}
>
<p>Here's the mean we just got.</p>
<svg id="mean-hist" />
<p>{slide_3_text_1}</p>
<p>{slide_3_text_2}</p>
<button class="button-30" on:click={() => swapModal(3,4)}>Next</button>
</dialog>

<dialog
	bind:this={dialog_list[4]}
>
<p>We're going to walk through the discovery, analysis, and proof of this mysterious bell shaped curve.</p>
<br>
<p>When you're ready, click the button below.</p>
<button class="button-30" on:click={() => closeOut()}>continue</button>
</dialog>

<style>
dialog {
    position: fixed;
    top: 50%;
    left: 50%;
    background: rgba(255, 255, 230, 1);
    transform: translate(-50%, -50%);
    width: 40vw;
    height: auto;
}

.movie-container {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    grid-template-rows: 1fr 1fr;
    grid-gap: 10px
}

.button-30 {
  align-items: center;
  appearance: none;
  background-color: #FCFCFD;
  border-radius: 4px;
  border-width: 0;
  box-shadow: rgba(54, 52, 2, 0.4) 0 2px 4px,rgba(54, 52, 2, 0.3) 0 7px 13px -3px,#e7e3d6 0 -3px 0 inset;
  box-sizing: border-box;
  color: #000000;
  cursor: pointer;
  display: inline-flex;
  font-family: 'Garamond', 'serif';
  font-weight:500;
  height: 48px;
  justify-content: center;
  line-height: 1;
  list-style: none;
  overflow: hidden;
  padding-left: 16px;
  padding-right: 16px;
  position: relative;
  text-align: left;
  text-decoration: none;
  transition: box-shadow .15s,transform .15s;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  white-space: nowrap;
  will-change: box-shadow,transform;
  font-size: 18px;
}

.button-30:focus {
  box-shadow: #e7e3d6 0 0 0 1.5px inset, rgba(54, 52, 2, 0.4) 0 2px 4px, rgba(54, 52, 2, 0.3) 0 7px 13px -3px, #e7e3d6 0 -3px 0 inset;
}

.button-30:hover {
  box-shadow: rgba(54, 52, 2, 0.4) 0 4px 8px, rgba(54, 52, 2, 0.3) 0 7px 13px -3px, #e7e3d6 0 -3px 0 inset;
  transform: translateY(-2px);
}

.button-30:active {
  box-shadow: #e7e3d6 0 3px 7px inset;
  transform: translateY(2px);
}

</style>