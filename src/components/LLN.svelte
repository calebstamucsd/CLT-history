<script>
import * as d3 from 'd3';
import { onMount } from 'svelte';
export let svg_width, svg_height;


let distribution = 'Geometric';
let parameter = 0.5;
let min_param = 0;
let max_param = 1;
let param_name = "p";
let mounted = false;
let mean, niceMean, currentSamp;
currentSamp = "...";
let sampleList = [];
let meanList = [];

let theme_light = '#DA2A05'; 
let theme_dark = '#831e0a';

let slider_label = `Pick a value for ${param_name}!`;

// Defines the margin that surrounds the svg element
const center_margin = {top: 50, right: 50, bottom: 50, left: 50},
  width = 260 - center_margin.left - center_margin.right,
  height = 200 - center_margin.top - center_margin.bottom;

// Defines the margin that surrounds the svg element
const mini_margin = {top: 15, right: 15, bottom: 15, left: 30},
  mini_width = 140 - mini_margin.left - mini_margin.right,
  mini_height = 120 - mini_margin.top - mini_margin.bottom;

// probability density of the geometric distribution
function geom(x, p) {
  return (((1 - p)**(x-1))*p);
}

function exp(x, lambda) {
    return lambda * (Math.E ** (-lambda * x))
}

function mixture(x, sigma) {
    let gauss_1 = (1/(sigma * Math.sqrt(2 * Math.PI))) * (Math.E**((-1/2)*(((x-2.5)/sigma)**2)))
    let gauss_2 = (1/(sigma * Math.sqrt(2 * Math.PI))) * (Math.E**((-1/2)*(((x-7.5)/sigma)**2)))
    return (gauss_1 + gauss_2) / 2
}

// calculates x, y pairs up to pointNum
function graphFunction(start_idx, end_idx, step, f, param) {
  const data = [];
  for (let x = start_idx; x <= end_idx; x += step) {
    let y = f(x, param);
    data.push([x, y])
  }
  return data;
}

function sampleOne() {
    if (distribution == "Geometric") {
        currentSamp = d3.randomGeometric(parameter)()
    }
    else if (distribution == "Exponential") {
        currentSamp = d3.randomExponential(parameter)()
    }
    else if (distribution == "Gaussian MM") {
        let which_gaussian = d3.randomBernoulli(0.5)()
        if(which_gaussian == 0) {
            currentSamp = d3.randomNormal(2.5, parameter)()
        }
        else {
            currentSamp = d3.randomNormal(7.5, parameter)()
        }
    }
    sampleList.push(currentSamp);
    sampleList = sampleList;
    meanList.push((sampleList.reduce((total, current) => total + current, 0) / sampleList.length).toFixed(2));
    meanList = meanList;
}

// Define graph limits (in x,y terms)
const xMax = 10;
const yMax = 1;
const geom_yMax = 0.55

let line_xMax, line_yMin, line_yMax, line_xScale, line_yScale, line_xAxis, line_yAxis;

// xScale, yScale convert math units to graphic coordinates making axis elements
let xScale = d3.scaleLinear([0, xMax], [0, width])
let yScale = d3.scaleLinear([0, yMax], [height, 0])
let xAxis = d3.axisBottom(xScale)
let yAxis = d3.axisLeft(yScale)
let bar_width = xScale(1)

// xScale, yScale convert math units to graphic coordinates making axis elements
let geom_xScale = d3.scaleLinear([0, xMax], [0, mini_width])
let geom_yScale = d3.scaleLinear([0, geom_yMax], [mini_height, 0])
let geom_xAxis = d3.axisBottom(geom_xScale).tickSize(0).tickFormat("");
let geom_yAxis = d3.axisLeft(geom_yScale).tickSize(0).tickFormat("");
let geom_bar_width = geom_xScale(1)

let intervalId = null;
let isRunning = false;
let buttonText = "Start Sampling!"
let inverseSpeed = (1/500);
let speed = (1/inverseSpeed)

function toggleInterval() {
    if (isRunning) {
      clearInterval(intervalId);
      buttonText = "Start Sampling!"
    } else {
      intervalId = setInterval(sampleOne, speed);
      buttonText = "Pause"
    }
    isRunning = !isRunning;
  }

let central_line = d3.line()
    .x(d => xScale(d[0]))
    .y(d => yScale(d[1]))

let line = d3.line()
  .x(d => geom_xScale(d[0]))
  .y(d => geom_yScale(d[1]))

let mean_line;

let central_graph, geometric, exponential, mix_model, mean_graph;

// We have to put SVG-related stuff into onMount() because onMount activates AFTER HTML elements are loaded
onMount(() => {
mounted = true;

// select the central graph svg, name it svg
central_graph = d3.select("#central-graph").attr("width", width + center_margin.left + center_margin.right)
  .attr("height", height + center_margin.top + center_margin.bottom)
  .append("g")
  .attr("transform", "translate(" + center_margin.left + "," + center_margin.top + ")");

// Define chart area
central_graph
  .append("clipPath")
  .attr("id", "chart-area")
  .append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", width)
  .attr("height", height)

// adding them to the actual graphic
central_graph.append("g")
  .attr("transform", `translate(0,${height})`)
  .call(xAxis)
central_graph.append("g")
  .attr("transform", `translate(0,0)`)
  .call(yAxis)

// Axes labels
central_graph.append("text")
  .attr("class", "x-label")
  .attr("text-anchor", "middle")
  .attr("x", width / 2)
  .attr("y", height + 35)
  .text("x");
central_graph.append("text")
  .attr("class", "y-label")
  .attr("text-anchor", "middle")
  .attr("y", -35)
  .attr("x", -height / 2)
  .attr("transform", "rotate(-90)")
  .html("P(X=x)")
central_graph.append("text")
.attr("class", "title")
.attr("text-anchor", "center")
.attr("x", width/2)
.attr("y", -10)
.text(distribution + " Distribution")

// select the central graph svg, name it svg
geometric = d3.select("#geom-dist").attr("width", mini_width + mini_margin.left + mini_margin.right)
  .attr("height", mini_height + 20 + mini_margin.top + mini_margin.bottom)
  .append("g")
  .attr("transform", "translate(" + mini_margin.left + "," + mini_margin.top + ")");

// Define chart area
geometric
  .append("clipPath")
  .attr("id", "chart-area")
  .append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", mini_width)
  .attr("height", mini_height)

geometric.append("g")
  .attr("transform", `translate(0,${mini_height})`)
  .call(geom_xAxis)
geometric.append("g")
  .attr("transform", `translate(0,0)`)
  .call(geom_yAxis)

geometric.selectAll("rect")
.data(graphFunction(0, xMax, 1, geom, 0.5))
.enter()
.append("rect")
.attr("x", d => geom_xScale(d[0]) - (geom_bar_width/2))
.attr("y", d => geom_yScale(d[1]))
.attr("width", geom_bar_width)
.attr("height", d => mini_height - geom_yScale(d[1]))
.attr("fill", "#DA2A05")
.attr("stroke", "black")
.attr("stroke-width", 1);

d3.select('div.item-tl').on('click', () => {
    distribution = 'Geometric'
    min_param = 0.01
    parameter = 0.5
    max_param = 1
    param_name = "p"
})

// select the central graph svg, name it svg
exponential = d3.select("#exp-dist").attr("width", mini_width + mini_margin.left + mini_margin.right)
  .attr("height", mini_height + 20 + mini_margin.top + mini_margin.bottom)
  .append("g")
  .attr("transform", "translate(" + mini_margin.left + "," + mini_margin.top + ")");

// Define chart area
exponential
  .append("clipPath")
  .attr("id", "chart-area")
  .append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", mini_width)
  .attr("height", mini_height)

exponential.append("g")
  .attr("transform", `translate(0,${mini_height})`)
  .call(geom_xAxis)
  .style("font-size", `8px`);
exponential.append("g")
  .attr("transform", `translate(0,0)`)
  .call(geom_yAxis)
  .style("font-size", `8px`);

exponential.append("path")
  .datum(graphFunction(0.1, xMax - 0.1, 0.2, exp, 0.5))
  .attr("class", "function")
  .attr("fill", "none")
  .attr("stroke", "blue")
  .attr("stroke-width", 1)
  .attr("d", line);

  d3.select('div.item-ml').on('click', () => {
    distribution = 'Exponential'
    min_param = 0.01
    parameter = 0.5
    max_param = 10
    param_name = "λ"
})

// select the central graph svg, name it svg
mix_model = d3.select("#mixture-dist").attr("width", mini_width + mini_margin.left + mini_margin.right)
  .attr("height", mini_height + 20 + mini_margin.top + mini_margin.bottom)
  .append("g")
  .attr("transform", "translate(" + mini_margin.left + "," + mini_margin.top + ")");

// Define chart area
mix_model
  .append("clipPath")
  .attr("id", "chart-area")
  .append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", mini_width)
  .attr("height", mini_height)

mix_model.append("g")
  .attr("transform", `translate(0,${mini_height})`)
  .call(geom_xAxis)
  .style("font-size", `8px`);
mix_model.append("g")
  .attr("transform", `translate(0,0)`)
  .call(geom_yAxis)
  .style("font-size", `8px`);

mix_model.append("path")
  .datum(graphFunction(0.1, xMax, 0.05, mixture, 1))
  .attr("class", "function")
  .attr("fill", "none")
  .attr("stroke", "magenta")
  .attr("stroke-width", 1)
  .attr("d", line);

d3.select('div.item-bl').on('click', () => {
    distribution = 'Gaussian MM'
    min_param = 0.2
    parameter = 1
    max_param = 3
    param_name = "σ"
})

mean_graph = d3.select("#mean-graph").attr("width", width + center_margin.left + center_margin.right)
  .attr("height", height + center_margin.top + center_margin.bottom)
  .append("g")
  .attr("transform", "translate(" + center_margin.left + "," + center_margin.top + ")");

// Define chart area
mean_graph
  .append("clipPath")
  .attr("id", "chart-area")
  .append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", width)
  .attr("height", height)

mean_graph.append("text")
  .attr("class", "x-label")
  .attr("text-anchor", "middle")
  .attr("x", width / 2)
  .attr("y", height + 35)
  .text("N");
mean_graph.append("text")
  .attr("class", "y-label")
  .attr("text-anchor", "middle")
  .attr("y", -35)
  .attr("x", -height / 2)
  .attr("transform", "rotate(-90)")
  .html("Running Mean")
}) // end of onMount

function wipeCentralGraph() {
    central_graph.selectAll("rect").remove()
    central_graph.selectAll("path.function").remove()
    wipeMeanGraph()
}

function wipeMeanGraph() {
  currentSamp = '...';
  sampleList = [];
  meanList = [];
  mean_graph.selectAll("path.function").remove()
  mean_graph.selectAll("line.mean").remove()
  mean_graph.selectAll("circle").remove()
  clearInterval(intervalId);
  buttonText = "Start Sampling!"
}

function swap_density_label(mass) {
  if(mass) {
    central_graph.selectAll('text.y-label').remove()
    central_graph.append("text")
    .attr("class", "y-label")
    .attr("text-anchor", "middle")
    .attr("y", -35)
    .attr("x", -height / 2)
    .attr("transform", "rotate(-90)")
    .html("P(X=x)")
  }
  else {
    central_graph.selectAll('text.y-label').remove()
    central_graph.append("text")
    .attr("class", "y-label")
    .attr("text-anchor", "middle")
    .attr("y", -35)
    .attr("x", -height / 2)
    .attr("transform", "rotate(-90)")
    .html("P(X)")
  }
}

$: if(mounted) {
    wipeCentralGraph()
    if(distribution == 'Geometric') {
        theme_light = '#DA2A05'
        theme_dark = '#831e0a'
        central_graph.selectAll("rect").remove();
        central_graph.selectAll("rect")
        .data(graphFunction(1, xMax, 1, geom, parameter))
        .enter()
        .append("rect")
        .attr("x", d => xScale(d[0]) - (bar_width/2))
        .attr("y", d => yScale(d[1]))
        .attr("width", bar_width)
        .attr("height", d => height - yScale(d[1]))
        .attr("fill", theme_light)
        .attr("stroke", "black")
        .attr("stroke-width", 1);
        mean = (1 / parameter);
        swap_density_label(true);
    }
    else if(distribution == 'Exponential') {
        theme_light = '#375DEF'
        theme_dark = '#071D70'
        central_graph.append("path")
        .datum(graphFunction(0, xMax, 0.25, exp, parameter))
        .attr("class", "function")
        .attr("fill", "none")
        .attr("stroke", theme_light)
        .attr("stroke-width", 1)
        .attr("d", central_line);
        mean = (1 / parameter)
        swap_density_label(false);
    }
    else if(distribution == 'Gaussian MM') {
        theme_light = '#CC00B3'
        theme_dark = '#6D0D61'
        central_graph.append("path")
        .datum(graphFunction(0, xMax, 0.1, mixture, parameter))
        .attr("class", "function")
        .attr("fill", "none")
        .attr("stroke", theme_light)
        .attr("stroke-width", 1)
        .attr("d", central_line);
        mean = 5
        swap_density_label(false);
    }
    niceMean = Number.isNaN(parseFloat(mean)) ? "Invalid number" : parseFloat(mean).toFixed(2);
}

$: if(mounted) {
    central_graph.select("text.title").remove();
    central_graph.append("text")
    .attr("class", "title")
    .attr("text-anchor", "title")
    .attr("x", width/2 - 75)
    .attr("y", -10)
    .text(distribution + " Distribution")
}

$: slider_label = `Pick a value for ${param_name}!`;

$: if(mounted) {
  line_xMax = Math.max(10, meanList.length)
  line_yMin = Math.min(0, Math.min(...meanList), mean)
  line_yMax = Math.max(5, Math.max(...meanList), (mean*2))

  line_xScale = d3.scaleLinear([0, line_xMax], [0, width])
  line_yScale = d3.scaleLinear([line_yMin, line_yMax], [height, 0])
  line_xAxis = d3.axisBottom(line_xScale)
  line_yAxis = d3.axisLeft(line_yScale)

  mean_graph.selectAll('g').remove()
  // adding them to the actual graphic
  mean_graph.append("g")
    .attr("transform", `translate(0,${height})`)
    .call(line_xAxis)
  mean_graph.append("g")
    .attr("transform", `translate(0,0)`)
    .call(line_yAxis)

  mean_graph.selectAll("path.function").remove()
  mean_graph.selectAll("line.mean").remove()
  mean_graph.selectAll("circle").remove()

  mean_line = d3.line()
    .x(d => line_xScale(d[0]))
    .y(d => line_yScale(d[1]))

  mean_graph.append("line")
    .attr("class", "mean")
    .attr("x1", 0) // Start of the line (x-coordinate)
    .attr("y1", line_yScale(mean)) // Start of the line (y-coordinate)
    .attr("x2", width) // End of the line (x-coordinate)
    .attr("y2", line_yScale(mean)) // End of the line (y-coordinate)
    .attr("stroke", theme_light) // Line color
    .attr("stroke-width", 2); // Line width

  mean_graph.append("path")
    .datum(meanList.map((value, index) => [index + 1, value]))
    .attr("class", "function")
    .attr("fill", "none")
    .attr("stroke", "black")
    .attr("stroke-width", 1)
    .attr("d", mean_line);

  mean_graph.selectAll(".point")
    .data(meanList.map((value, index) => [index + 1, value]))
    .enter().append("circle")
    .attr("class", "point")
    .attr("cx", d => line_xScale(d[0]))
    .attr("cy", d => line_yScale(d[1]))
    .attr("r", 2) // Set the radius of the circles
    .attr("fill", "black"); // Set the fill color of the circles
}

$: if(mounted) {
    speed = (1/inverseSpeed);
    toggleInterval();
    toggleInterval();
  }

</script>


<div class="container" style="width: {svg_width}px; height: {svg_height - 60}px;">
    <div class="left-row item-tl">
        <svg id="geom-dist" name='geom'></svg>
        <label class="dist-pick" for='geom'>Geometric</label>
    </div>
    <div class="left-row item-ml">
        <svg id="exp-dist" name='exp'></svg>
        <label class="dist-pick" for='exp'>Exponential</label>
    </div>
    <div class="left-row item-bl">
        <svg id="mixture-dist" name="GMM"></svg>
        <label class="dist-pick" for="GMM">Gaussian Mixture</label>
    </div>
    <div class="item-middle">
        <svg id="central-graph"></svg>
        <br>
        <label for="parameter">{slider_label}</label>
        <br>
        <input
            id="slider"
            type="range"
            name="parameter"
            style="width: 50%; margin: 0 auto;"
            min={min_param}
            max={max_param}
            step="0.01"
            bind:value={parameter}
        />
        <br>
        {param_name} = {parameter}
        <br>
        <br>
        The true mean of this <br> distribution is {niceMean}.
        <br>
    </div>
    <div class="item-right">

      <p> Sample: {Number.isNaN(parseFloat(currentSamp)) ? "..." : parseFloat(currentSamp).toFixed(2)} </p>
      <p> Number of Samples: {sampleList.length} </p>
      <p> Running Mean: {(meanList.length < 1) ? "..." : (meanList[meanList.length - 1])} </p>
      <svg id='mean-graph'></svg>
      <button class='button-30' on:click={toggleInterval} style="width: fit-content; padding: 3px; margin: 0 auto;">
        {buttonText}
      </button>
      <br>
      <label for='speed'>Sampling Speed:</label>
      <br>
      <input
        id="slider"
        type="range"
        name="speed"
        min={(1/1000)}
        max={(1/75)}
        step="0.0001"
        
        bind:value={inverseSpeed}
      />
      <br>
      <button class='button-30' on:click={wipeMeanGraph} name='reset' style="width: fit-content; padding: 3px; margin: 0 auto;">Reset</button>
    </div>
</div>


<style>
p {
  margin: 0; /* Set margin to 0 */
  padding: 5px; /* Add some padding for spacing */
}
.dist-pick {
  transform: translateY(-15px);
}

.container {
    display: grid;
    grid-template-columns: 1fr 2fr 2fr;
    grid-template-rows: 1fr 1fr 1fr;
    grid-gap:20px;
}

.left-row{
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  transition: box-shadow .30s,transform .30s;
  will-change: box-shadow,transform;
}
.left-row:hover{
  transform:translateY(-2px)
}
.left-row:active{
  transform:translateY(2px)
}

.item-tl{
    grid-column: 1 / span 1;
    grid-row: 1 / span 1;
    box-shadow: rgba(54, 52, 2, 0.4) 0 2px 4px,rgba(54, 52, 2, 0.3) 0 7px 13px -3px,rgba(240, 128, 128, 0.8) 0 -3px 0 inset;
    background-color: rgba(240, 128, 128, 0.5);
}
.item-tl:hover{box-shadow: rgba(54, 52, 2, 0.4) 0 4px 8px,rgba(54, 52, 2, 0.3) 0 7px 13px -3px,rgba(240, 128, 128, 0.8) 0 -3px 0 inset;}
.item-tl:focus {box-shadow: rgba(240, 128, 128, 0.8) 0 0 0 1.5px inset, rgba(54, 52, 2, 0.4) 0 2px 4px, rgba(54, 52, 2, 0.3) 0 7px 13px -3px, rgba(240, 128, 128, 0.8) 0 -3px 0 inset;}
.item-ml{
    grid-column: 1 / span 1;
    grid-row: 2 / span 1;
    box-shadow: rgba(54, 52, 2, 0.4) 0 2px 4px,rgba(54, 52, 2, 0.3) 0 7px 13px -3px,rgba(172, 216, 230, 0.8) 0 -3px 0 inset;
    background-color: rgba(172, 216, 230, 0.5);
}
.item-bl{
    grid-column: 1 / span 1;
    grid-row: 3 / span 1;
    box-shadow: rgba(54, 52, 2, 0.4) 0 2px 4px,rgba(54, 52, 2, 0.3) 0 7px 13px -3px,rgba(255, 182, 193, 0.8) 0 -3px 0 inset;
    background-color: rgba(255, 182, 193, 0.5);
}

.item-middle{
    grid-column: 2 / span 1;
    grid-row: 1 / span 3;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}
.item-right{
    grid-column: 3 / span 1;
    grid-row: 1 / span 3;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
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

.button-30:hover {
  box-shadow: rgba(54, 52, 2, 0.4) 0 4px 8px, rgba(54, 52, 2, 0.3) 0 7px 13px -3px, #e7e3d6 0 -3px 0 inset;
  transform: translateY(-2px);
}

.button-30:active {
  box-shadow: #e7e3d6 0 3px 7px inset;
  transform: translateY(2px);
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