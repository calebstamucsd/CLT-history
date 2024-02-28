<script>
import * as d3 from 'd3';
import { onMount } from 'svelte';
export let svg_width, svg_height;


let distribution = 'Geometric';
let parameter = 0.5;
let mounted = false;

let containerWidth = svg_width; // Fixed width for the container
let containerHeight = svg_height; // Fixed height for the container

let slider_label = "Pick a value for p!";

// Defines the margin that surrounds the svg element
const margin = {top: 50, right: 50, bottom: 50, left: 50},
  width = 250 - margin.left - margin.right,
  height = 200 - margin.top - margin.bottom;

// probability density of the geometric distribution
function geom(x) {
  return (((1 - parameter)**(x-1))*parameter);
}

// calculates x, y pairs up to pointNum
function graphFunction(pointNum, f) {
  const data = [];
  for (let x = 1; x <= pointNum; x++) {
    let y = f(x);
    data.push([x, y])
  }
  return data;
}

// Define graph limits (in x,y terms)
const xMax = 10;
const yMax = 1;

// xScale, yScale convert math units to graphic coordinates
let xScale = d3.scaleLinear([0, xMax], [0, width])
let yScale = d3.scaleLinear([0, yMax], [height, 0])

// making axis elements
let xAxis = d3.axisBottom(xScale)
let yAxis = d3.axisLeft(yScale)

// The width between units for adding bars to a bar chart
let bar_width = xScale(1)
let central_graph = null;

// We have to put SVG-related stuff into onMount() because onMount activates AFTER HTML elements are loaded
onMount(() => {
mounted = true;

// select the central graph svg, name it svg
central_graph = d3.select("#central-graph").attr("width", width + margin.left + margin.right)
  .attr("height", height + margin.top + margin.bottom)
  .append("g")
  .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

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

// Add function graph
// let line = d3.line()
//   .x(d => xScale(d[0]))
//   .y(d => yScale(d[1]))
// svg.append("path")
//   .datum(graphFunction())
//   .attr("clip-path", "url(#chart-area)")
//   .attr("fill", "none")
//   .attr("stroke", "teal")
//   .attr("stroke-width", 2)
//   .attr("d", line);


}) // end of onMount

$: if(mounted) {
    parameter
    central_graph.selectAll("rect").remove();
    central_graph.selectAll("rect")
    .data(graphFunction(xMax, geom))
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
</script>


<div class="container" style="width: {containerWidth}px; height: {containerHeight}px;">
    {containerWidth} {containerHeight}
    <div class="item-tl"></div>
    <div class="item-ml"></div>
    <div class="item-bl"></div>
    <div class="item-middle">
        <svg id="central-graph"></svg>

        <label>{slider_label} {svg_width} {svg_height}</label>
        <br>
        <input
            id="slider"
            type="range"
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
    }
    .item-ml{
        grid-column: 1 / span 1;
        grid-row: 2 / span 1;
        background-color: lightblue;
    }
    .item-bl{
        grid-column: 1 / span 1;
        grid-row: 3 / span 1;
        background-color: lightpink;
    }
    .item-middle{
        grid-column: 2 / span 1;
        grid-row: 1 / span 3;
        background-color: bisque;
    }
    .item-right{
        grid-column: 3 / span 1;
        grid-row: 1 / span 3;
        background-color: beige;
    }
</style>