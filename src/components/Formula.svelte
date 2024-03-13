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

const center_margin = {top: 50, right: 50, bottom: 50, left: 50}, // graph size in terms of like display pixels
  width = 300 - center_margin.left - center_margin.right,
  height = 250 - center_margin.top - center_margin.bottom;

let xMin = -4; // graph limits in terms of axis coordinates
let xMax = 4;
let yMin = -1;
let yMax = 4
let function_name = '';

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
    formula_graph.append("path")
          .datum(graphFunction(-4, 4, 0.05, func))
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
      function_name = 'f(x) = e<sup>x</sup>'
    }
    if(function_index == 2) {
      plot_with_func(neg_exponential)
      function_name = 'f(x) = -e<sup>x</sup>'
    }
    if(function_index == 3) {
      plot_with_func(square_neg)
      function_name = 'f(x) = -e<sup>x<sup>2</sup></sup>'
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
    .attr("height", height)

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

</script>

<!-- placing the svg in the html section -->
<svg id='formula-graph'></svg>

<!-- button that calls changeFunction whenver you click it -->
<button class='button-30' on:click={changeFunction}> Update </button>

<style>
/* button is now pretty */
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
</style>