<script>
import * as d3 from 'd3';
import { onMount } from 'svelte';
export let svg_width, svg_height;


let distribution = 'Geometric';
let parameter = 0.5;
let mounted = false;

let slider_label = "Pick a value for p!";

// Defines the margin that surrounds the svg element
const center_margin = {top: 50, right: 50, bottom: 50, left: 50},
  width = 250 - center_margin.left - center_margin.right,
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

// Define graph limits (in x,y terms)
const xMax = 10;
const yMax = 1;
const geom_yMax = 0.55

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

// The width between units for adding bars to a bar chart

let central_graph, geometric, exponential, mix_model;

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
  .attr("class", "x label")
  .attr("text-anchor", "end")
  .attr("x", width / 2 + 5)
  .attr("y", height + 35)
  .text("x");
central_graph.append("text")
  .attr("class", "y label")
  .attr("text-anchor", "end")
  .attr("y", -35)
  .attr("x", -height / 2)
  .attr("transform", "rotate(-90)")
  .html("P(X=x)")
central_graph.append("text")
.attr("class", "title")
.attr("text-anchor", "title")
.attr("x", width/2 - 75)
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
    console.log(distribution)
    distribution = 'Geometric'
    console.log(distribution)
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

// Add function graph
let line = d3.line()
  .x(d => geom_xScale(d[0]))
  .y(d => geom_yScale(d[1]))
exponential.append("path")
  .datum(graphFunction(0.1, xMax - 0.1, 0.5, exp, 0.5))
  .attr("class", "function")
  .attr("clip-path", "url(#chart-area)")
  .attr("fill", "none")
  .attr("stroke", "blue")
  .attr("stroke-width", 1)
  .attr("d", line);

  d3.select('div.item-ml').on('click', () => {
    console.log(distribution)
    distribution = 'Exponential'
    console.log(distribution)
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
  .datum(graphFunction(0.1, xMax, 0.25, mixture, 1))
  .attr("class", "function")
  .attr("clip-path", "url(#chart-area)")
  .attr("fill", "none")
  .attr("stroke", "magenta")
  .attr("stroke-width", 1)
  .attr("d", line);

d3.select('div.item-bl').on('click', () => {
    console.log(distribution)
    distribution = 'Gaussian MM'
    console.log(distribution)
})
}) // end of onMount

function wipeCentralGraph() {
    central_graph.selectAll("rect").remove()
    central_graph.selectAll("path.function").remove()
}

$: if(mounted) {
    distribution
    wipeCentralGraph()
}q

$: if(mounted && (distribution == "Geometric")) {
    central_graph.selectAll("rect").remove();
    central_graph.selectAll("rect")
    .data(graphFunction(1, xMax, 1, geom, parameter))
    .enter()
    .append("rect")
    .attr("x", d => xScale(d[0]) - (bar_width/2))
    .attr("y", d => yScale(d[1]))
    .attr("width", bar_width)
    .attr("height", d => height - yScale(d[1]))
    .attr("fill", "#DA2A05")
    .attr("stroke", "black")
    .attr("stroke-width", 1);
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

</script>


<div class="container" style="width: {svg_width - 40}px; height: {svg_height - 60}px;">
    <div class="item-tl">
        <svg id="geom-dist"></svg>
        <label>Geometric</label>
    </div>
    <div class="item-ml">
        <svg id="exp-dist"></svg>
        <label>Exponential</label>
    </div>
    <div class="item-bl">
        <svg id="mixture-dist"></svg>
        <label>Gaussian Mixture</label>
    </div>
    <div class="item-middle">
        <svg id="central-graph"></svg>
        <br>
        <label>{slider_label}</label>
        <br>
        <input
            id="slider"
            type="range"
            style="width: 50%; margin: 0 auto;"
            min="0"
            max="1"
            step="0.01"
            bind:value={parameter}
        />
        <br>
        p = {parameter}
    </div>
    <div class="item-right"></div>
</div>


<style>
    .container {
        display: grid;
        grid-template-columns: 1fr 2fr 2fr;
        grid-template-rows: 1fr 1fr 1fr;
        grid-gap:10px;
    }
    .item-tl{
        grid-column: 1 / span 1;
        grid-row: 1 / span 1;
        background-color: lightcoral;
        display: flex;
        flex-direction: column;
        justify-content: center;
    }
    .item-ml{
        grid-column: 1 / span 1;
        grid-row: 2 / span 1;
        background-color: lightblue;
        display: flex;
        flex-direction: column;
        justify-content: center;
    }
    .item-bl{
        grid-column: 1 / span 1;
        grid-row: 3 / span 1;
        background-color: lightpink;
        display: flex;
        flex-direction: column;
        justify-content: center;
    }
    .item-middle{
        grid-column: 2 / span 1;
        grid-row: 1 / span 3;
        background-color: bisque;
        display: flex;
        flex-direction: column;
        justify-content: center;
    }
    .item-right{
        grid-column: 3 / span 1;
        grid-row: 1 / span 3;
        background-color: beige;
        display: flex;
        flex-direction: column;
        justify-content: center;
    }
</style>