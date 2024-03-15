<script>
import * as d3 from 'd3';
import { onMount } from 'svelte';
let mounted = false;

// this will be heavily inspired by the 3blue1brown video, clicking button will switch between the functions that "build up" the
// formula for the CLT starting at 15:54
// TODOS
// 1. add all the functions from the 3blue1brown video, and modify changeFunction so that clicking the button switches
// between them. will need to remove previous function before adding next.
// 2. add annotations (on or below the graph) that explain the stuff they talk abt in the 3blue1brown video
// OPTIONAL/BONUS (maybe do these after video deadline)
// 3. i think there's a way to crop the graph so functions don't go offscreen, but I forgot how - something with clipPath?
// 4. make switching between functions smooth and animated
//      I don't think you can do this with just d3.js transitions - might need to iterations of calculating new x,y, coordinates
//      removing the previous frame of graph and plotting the new one

// The below chunk of variables are all things that affect the graph's display
let formula_graph;

const center_margin = {top: 0, right: 50, bottom: 50, left: 50}, // graph size in terms of like display pixels
  width = 300 - center_margin.left - center_margin.right,
  height = 250 - center_margin.top - center_margin.bottom;

let xMin = -4; // graph limits in terms of axis coordinates
let xMax = 4;
let yMin = -1;
let yMax = 4

let cVal = 1;
let muVal = 0;
let cActive = false;
let sigActive = false;
let normActive = false;
let muActive = false;

let slider_min;
let slider_max;
let button_text = 'Update'

let xScale = d3.scaleLinear([xMin, xMax], [0, width]) // function that maps math coordinates to display coords
let yScale = d3.scaleLinear([yMin, yMax], [height, 0])
let xAxis = d3.axisBottom(xScale) // svg object for the x-axis
xAxis.ticks(8) // specify 8 tick marks (default was too many)
let yAxis = d3.axisLeft(yScale) // same as above but for the y-axis
yAxis.ticks(5)

let line = d3.line() // I think this defines like the physical line object that gets manipulated by functions later on
  .x(d => xScale(d[0]))
  .y(d => yScale(d[1]))

// variable that defines which function we're plotting
let function_index = 0;

function plot_with_func(func) {
  formula_graph.selectAll('path.function').remove()
  let xMin = -4;
  let xMax = 4;
  if(func == exponential) {
    xMax = Math.log(4)
  }
  if(func == neg_exponential) {
    xMin = -Math.log(4)
  }
  formula_graph.append("path")
        .datum(graphFunction(xMin, xMax, 0.05, func))
        .attr("class", "function")
        .attr("fill", "none")
        .attr("stroke", "black")
        .attr("stroke-width", 2)
        .attr("d", line);
}

// the "main" function that's called whenver the button is pressed. Will change the graph to the next item
function changeFunction() {
    function_index += 1;
    if(function_index == 1) {
      plot_with_func(exponential)
    }
    if(function_index == 2) {
      plot_with_func(neg_exponential)
    }
    if(function_index == 3) {
      plot_with_func(square_neg)
    }
    if(function_index == 4) {
      slider_min = 0.2
      slider_max = 5
      cActive = true;
      plot_with_func(square_neg_c)
    }
    if(function_index == 5) {
      cActive = false;
      sigActive = true;
      slider_min = 0.1
      slider_max = 2.5
      cVal = (1/(2*cVal))
      plot_with_func(square_neg_sigma)
    }
    if(function_index == 7) {
      sigActive = false;
      normActive = true;
      plot_with_func(normalized_square_neg_sigma)
    }
    if(function_index == 8) {
      normActive = false;
      muActive = true;
      plot_with_func(normal_distribution)
      button_text = 'Reset'
    }
    if(function_index == 9) {
      muActive = false;
      function_index = 0
      button_text = 'Update'
      formula_graph.selectAll('path.function').remove()
    }
}

// takes in one x coordinate, outputs the y coordinate
// we want to have a bunch of these for each function from the 3blue1brown video
function exponential(x) {
    return Math.E ** x - 1
}
function neg_exponential(x) {
  return (Math.E ** (-x)) - 1
}
function square_neg(x) {
  return (Math.E ** -(x**2)) - 1
}
function square_neg_c(x) {
  return (Math.E ** -(cVal*x**2)) - 1
}
function square_neg_sigma(x) {
  return (Math.E ** -((x**2)/(2*cVal))) - 1
}
function normalized_square_neg_sigma(x) {
  return ((Math.E ** -((x**2)/(2*cVal)))/(Math.sqrt(cVal*Math.PI*2))) - 1
}
function normal_distribution(x) {
  return ((Math.E ** -(((x-muVal)**2)/(2*cVal)))/(Math.sqrt(cVal*Math.PI*2))) - 1
}


// calculates f(x) for every x in range(stard_idx, end_idx, step)
function graphFunction(start_idx, end_idx, step, f) {
  const data = [];
  for (let x = start_idx; x <= end_idx; x += step) {
    let y = f(x);
    data.push([x, y])
  }
  return data;
}

onMount(() => {
    mounted = true;

    // everything starting with a formula_graph... is rendering the graph onscreen
    formula_graph = d3.select("#formula-graph").attr("width", width + center_margin.left + center_margin.right)
    .attr("height", height + center_margin.top + center_margin.bottom)
    .append("g")
    .attr("transform", "translate(" + center_margin.left + "," + center_margin.top + ")");

    formula_graph
    .append("clipPath")
    .attr("id", "chart-area")
    .append("rect")
    .attr("x", 0)
    .attr("y", 0)
    .attr("width", width)
    .attr("height", height-30)

    formula_graph.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(xAxis)
    formula_graph.append("g")
    .attr("transform", `translate(${width/2},${yScale(3)})`)
    .call(yAxis)

    // got rid of the tick marks for zero because they looked bad (comment this out to see what I mean)
    formula_graph.selectAll('g.tick text').filter(function() {
            return d3.select(this).text() === "0";
        }).remove()

})

$: if(cActive) {
  cVal == cVal;
  plot_with_func(square_neg_c)
}

$: if(sigActive) {
  cVal == cVal;
  plot_with_func(square_neg_sigma)
}
$: if(normActive) {
  cVal == cVal;
  plot_with_func(normalized_square_neg_sigma)
}
$: if(muActive) {
  cVal == cVal;
  muVal == muVal;
  plot_with_func(normal_distribution)
}

</script>

<div class='container'>
  <div class='item'>
    {#if function_index == 1}
    <p>f(x) = e<sup>x</sup></p>
    {/if}
    {#if function_index == 2}
    <p>f(x) = -e<sup>x</sup></p>
    {/if}
    {#if function_index == 3}
    <p>f(x) = -e<sup>x<sup>2</sup></sup></p>
    {/if}
    {#if function_index == 4}
    <p>f(x) = -e<sup>cx<sup>2</sup></sup></p>
    {/if}
    {#if (function_index == 5) || (function_index == 6)}
    <p>f(x) = -e<sup>-(<sup>x<sup>2</sup></sup>/<sub>2σ<sup>2</sup></sub>)</sup></p>
    {/if}
    {#if function_index == 7}
    <p>f(x) = (<sup>1</sup>/<sub>2πσ<sup>2</sup></sub>)-e<sup>-(<sup>x<sup>2</sup></sup>/<sub>2σ<sup>2</sup></sub>)</sup></p>
    {/if}
    {#if function_index == 8}
    <p>f(x) = (<sup>1</sup>/<sub>2πσ<sup>2</sup></sub>)-e<sup>-(<sup>(x-μ)<sup>2</sup></sup>/<sub>2σ<sup>2</sup></sub>)</sup></p>
    {/if}
  </div>

  <div class='item'>
    <svg id='formula-graph' style='overflow:hidden;'></svg>
  </div>

  <div class='item'>
    {#if (cActive || sigActive || normActive || muActive)}
      <input
      id="n-input"
      type="range"
      name="cVal"
      min={slider_min}
      max={slider_max}
      step="0.01"
      bind:value={cVal}
      />
      <br>
      {#if cActive}
        <label for="cVal">c = {(Math.round(cVal*100)/100)}</label>
      {/if}
      {#if sigActive || normActive || muActive}
        <label for="cVal">σ<sup>2</sup> = {(Math.round(cVal*100)/100)}</label>
      {/if}
      <br>
    {/if}
    {#if muActive}
      <input
      id="n-input"
      type="range"
      name="muVal"
      min={-3}
      max={3}
      step="0.01"
      bind:value={muVal}
      />
      <br>
      <label for="muVal">μ = {muVal}</label>
      <br>
    {/if}
  </div>

  <div class='filler'/>

  <div class='graph-text'>
    {#if function_index == 0}
    <p> We're going to walk through how the formula for the Normal Distribution is constructed. </p>
    <p>Start by clicking the button to plot an exponential function.</p>
    {/if}
    {#if function_index == 1}
    <p> Great! This graph shows exponential growth. It's low for negative values, but explodes for positive ones.</p> 
    <p> Click the button again to flip it.</p>
    {/if}
    {#if function_index == 2}
    <p> This graph shows exponential decay. Now it's low for positive values, but huge for negative ones. </p>
    <p> We know that our probability distribution should be the highest at a single point, then decay on both sides. But what can we do to make this happen? </p>
    <p> Take a guess, then continue.</p>
    {/if}
    {#if function_index == 3}
    <p> If you guessed right - great job! By exponentiating the value of x, we make the decay symmetric. </p>
    <p> Our next step is going to be to customize the rate of decay. </p>
    {/if}
    {#if function_index == 4}
    <p> Play around with the value of c! By adjusting it, you can make the decay thinner or steeper. </p>
    <p> The simple term "c" does seem awfully convenient, though... </p>
    {/if}
    {#if function_index == 5}
    <p>Instead of c, we'll use the value σ<sup>2</sup>, which represents the function's variance. As you can see, this works the exact same way that the constant c did. </p> 
    <p> However, σ<sup>2</sup> makes a little bit more sense to use if you dig into the statistics behind the distribution (just trust us on this one!)</p>
    {/if}
    {#if function_index == 6}
    <p>Unfortunately, I'm afraid we have a problem. This is a cool function, but it's just not a probability distribution. For a function to be a probability distribution, the area under the curve must be exactly one - you can't have more than 100% probability! </p> 
    <p> Right now, the area under the curve is about {Math.round(200*cVal*Math.PI)/100} square units, or (2πσ<sup>2</sup>).</p>
    {/if}
    {#if function_index == 7}
    <p>All we have to do to fix this is divide by 2πσ<sup>2</sup>. Now, the area under the curve is one, like we want. </p>
    <p> For our final step, we want to be able to customize the center of the distribution. The average isn't always zero!</p>
    {/if}
    {#if function_index == 8}
    <p>Great! Now we can make the average, μ, whatever our heart desires. What's more, we've reached the final formula for the normal distribution!</p> 
    <p> If you set μ to zero and σ<sup>2</sup> to one, you'll see what's called the standard normal distribution, the most fundamental case of this function.</p>
    {/if}
  </div>

  <div><br><br><button class='button-30' on:click={changeFunction}> {button_text} </button></div>

</div>


<!-- button that calls changeFunction whenver you click it -->


<style>
/* button is now pretty */
.container {
    display: grid;
    width: 50vw;
    height: 80vh;
    grid-template-columns: 1fr 1fr;
    grid-template-rows: 1fr 4fr 2fr;
    grid-gap:10px;
}

.item {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  grid-column: 1 / span 1;
}

.filler {
  grid-column: 2 / span 1;
  grid-row: 1 / span 1;
}

.graph-text {
  grid-column: 2 / span 1;
  grid-row: 2 / span 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color:#FCFCFD;
  background-color: rgba(0,0,0,0.8);
  border-radius:10px;
  padding: 20px
}

.button-30 {
    align-items: center;
    appearance: none;
    background-color: #FCFCFD;
    border-radius: 4px;
    border-width: 0;
    box-shadow: rgba(54, 52, 2, 0.4) 0 2px 4px,rgba(54, 52, 2, 0.3) 0 7px 13px -3px,#e7e3d6 0 -3px 0 inset;
    color: #000000;
    cursor: pointer;
    display: inline-flex;
    font-family: 'Garamond', 'serif';
    font-size: 18px;
    height: 40px;
    justify-content: center;
    transition: box-shadow .15s,transform .15s;
    will-change: box-shadow,transform;
}

.button-30:focus {
  box-shadow: #e7e3d6 0 0 0 1.5px inset, rgba(54, 52, 2, 0.4) 0 2px 4px, rgba(54, 52, 2, 0.3) 0 7px 13px -3px, #e7e3d6 0 -3px 0 inset;
}

.button-30:hover:enabled {
  box-shadow: rgba(54, 52, 2, 0.4) 0 4px 8px, rgba(54, 52, 2, 0.3) 0 7px 13px -3px, #e7e3d6 0 -3px 0 inset;
  transform: translateY(-2px);
}

.button-30:active:enabled {
  box-shadow: #e7e3d6 0 3px 7px inset;
  transform: translateY(2px);
}

.button-30:disabled {
    background-color: rgba(181, 181, 181, 0.2);
    cursor: default;
    box-shadow: none;
}

input[type='range'] {
  -webkit-appearance: none;
  appearance: none;
  cursor: pointer;
  border-radius: 10px;
  width: 15rem;
  height: 0.5rem;
  background: #FCFCFD;
  box-shadow: rgba(54, 52, 2, 0.4) 0 2px 4px,rgba(54, 52, 2, 0.3) 0 7px 13px -3px,#e7e3d6 0 -2px 0 inset;
  width: 50%; 
  margin: 0 auto;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none; /* Override default look */
  appearance: none;
  border-radius: 10px;
  height: 1rem;
  width: 1rem;    
  background: #FCFCFD;
  box-shadow: rgba(54, 52, 2, 0.4) 0 2px 4px,rgba(54, 52, 2, 0.3) 0 7px 13px -3px,#e7e3d6 0 -2px 0 inset;
  transition: box-shadow .15s,transform .15s;
  will-change: box-shadow,transform;
}
/* input[type="range"]::-webkit-slider-thumb:hover {
  transform: translateY(-2px)
} */
input[type="range"]::-webkit-slider-thumb:active {
  transform: translateY(1px)
}
</style>