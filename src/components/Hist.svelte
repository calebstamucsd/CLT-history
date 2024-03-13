<script>
import * as d3 from 'd3';
import { onMount } from 'svelte';
let mounted = false;

let Hist, CoinFlip;
let inputValue = '';

// histogram center margin
const Hcenter_margin = {top: 50, right: 50, bottom: 50, left: 50}, // graph size in terms of like display pixels
  width = 450 - Hcenter_margin.left - Hcenter_margin.right,
  height = 350 - Hcenter_margin.top - Hcenter_margin.bottom;


let xMin = 0; // graph limits in terms of axis coordinates
let xMax = inputValue;
let yMin = 0;
let yMax = 1;

let xScale = d3.scaleLinear([xMin, xMax], [0, width]) // function that maps math coordinates to display coords
let yScale = d3.scaleLinear([yMin, yMax], [height, 0])
let xAxis = d3.axisBottom(xScale) // svg object for the x-axis
xAxis.ticks(inputValue/2)
let yAxis = d3.axisLeft(yScale) // same as above but for the y-axis
yAxis.ticks(5)

// updating the Hist xaxis based on n number of coins input
function updateHxAxis(ivalue) {
  xMax = Number(ivalue.target.value);
  xScale = d3.scaleLinear([xMin, xMax], [0, width])
  let xAxis = d3.axisBottom(xScale)
  Hist.select("#HxAxis")
  .attr("transform", `translate(0,${height})`)
  .call(xAxis)
}


// CoinFlip center margin
const CFcenter_margin = {top: 50, right: 50, bottom: 50, left: 50}, // graph size in terms of like display pixels
  CFwidth = 450 - CFcenter_margin.left - CFcenter_margin.right,
  CFheight = 350 - CFcenter_margin.top - CFcenter_margin.bottom;

// CoinFlip axis limits
let CFxMin = 0;
let CFxMax = 3;
let CFyMin = 0;
let CFyMax = 100;

let CFxScale = d3.scaleLinear([CFxMin, CFxMax], [0, CFwidth]) // function that maps math coordinates to display coords
let CFyScale = d3.scaleLinear([CFyMin, CFyMax], [CFheight, 0])
let CFxAxis = d3.axisBottom(CFxScale) // svg object for the x-axis
CFxAxis.ticks(3)
let ticklabels = ['', 'Heads', 'Tails']
CFxAxis.tickFormat((d,i) => ticklabels[i])
CFxAxis.tickSizeOuter(); // Adjust the length of tick lines
let CFyAxis = d3.axisLeft(CFyScale) // same as above but for the y-axis
CFyAxis.ticks(5)

// Log the tick values to the console

let headsCount = 0;
let tailsCount = 0;
let simheadsCount = 0;

function flipCoin() {
  const outcome = Math.random() < 0.5 ? 'Heads' : 'Tails';
  if (outcome === 'Heads') {
    headsCount++;
  } else {
    tailsCount++;
  }
  updateBars();
}

async function flipCoins(iV) {
  headsCount = 0;
  tailsCount = 0;
  for (let i = 0; i < iV; i++) {
    flipCoin();
    await pause(5000/iV);
  }
}

function pause(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

let binDict = {};
let numBins = [];
async function createBins(iV) {
  numBins = [];
  for (var i = 0; i <= iV; i++) {
    numBins.push(i);
  }


  let binsScale = d3.scaleLinear([0, iV], [0, CFwidth])

  for (var i = 0; i <= iV; i++) {
    binDict[i] = 0;
  }
}

let updateBarsrunning = false

// creating bins for histogram
async function sim1000flipnCoins(iV) {
  updateBarsrunning = true
  simheadsCount = 0;
  // run 1000 simulations of iV coin flips
  
  for (let i = 0; i < 10000; i++){
    if (updateBarsrunning){
    simheadsCount = 0;
    // single sim of iV coinflips
    for (let j = 0; j < iV; j++) {
      const outcome = Math.random() < 0.5 ? 'Heads' : 'Tails';
      if (outcome === 'Heads') {
        simheadsCount++;
      }
    }
    } else {
      return
    }

    binDict[simheadsCount] = binDict[simheadsCount]+1
    await pause(5000/(iV*10000));
    updateHistogramBars()
  }
  console.log(binDict)

}


function updateHistogramBars() {
  // Select all existing bars and bind data
  const bars = Hist.selectAll('.hist-bar')
    .data(Object.entries(binDict));

  // Enter selection: create new bars
  bars.enter()
    .append('rect')
    .attr('class', 'hist-bar')
    .attr('x', d => xScale(parseInt(d[0])))
    .attr('width', width / Object.keys(binDict).length)
    .attr('y', d => yScale(d[1]))
    .attr('height', d => height - yScale(d[1]))
    .attr('fill', 'steelblue');

  // Update selection: update existing bars
  bars
    .attr('x', d => xScale(parseInt(d[0])))
    .attr('width', width / Object.keys(binDict).length)
    .attr('y', d => yScale(d[1]))
    .attr('height', d => height - yScale(d[1]));

  const maxCount = Math.max(...Object.values(binDict));

  // Update the y-scale domain
  yScale.domain([0, maxCount]);

  // Redraw the y-axis
  Hist.select("g.y-axis")
    .transition()
    .duration(5)
    .call(yAxis);

  // Exit selection: remove bars not needed anymore
  bars.exit().remove();
}

function clearhBars(){
  updateBarsrunning = false
  const bars = Hist.selectAll('.hist-bar').remove()
}



//



function updateBars() {
  // Update data
  const data = [{ label: 'Heads', count: headsCount }, { label: 'Tails', count: tailsCount }];

  // Select head bar and bind data
  const hbars = CoinFlip.select('#cfbar-head')
  .data(data);
  
  // Select tail bars and bind data
  const tbars = CoinFlip.select('#cfbar-tail')
  .data(data);

  // Update existing bars
  hbars.attr('height',CFheight - CFyScale(data.find(item => item.label === 'Heads').count));
  tbars.attr('height',CFheight - CFyScale(data.find(item => item.label === 'Tails').count));

  CoinFlip.select("#heads_text").text(data.find(item => item.label === 'Heads').count);

  // Calculate the maximum count for y-axis scale
  const maxCount = Math.max(headsCount, tailsCount);

  // Update y-axis scale domain
  CFyScale.domain([0, maxCount]);
  
  // Redraw y-axis
  CoinFlip.select("g.y-axis").call(CFyAxis);
}


onMount(() => {
  mounted = true;

  // creating histogram container
  Hist = d3.select("#histogram-coin")
  .attr("width", width + Hcenter_margin.left + Hcenter_margin.right)
  .attr("height", height + Hcenter_margin.top + Hcenter_margin.bottom)
  .append("g")
  .attr("transform", "translate(" + Hcenter_margin.left + "," + Hcenter_margin.top + ")");

  // clips any svg's which are outside of this region
  Hist.append("clipPath")
  .attr("id", "chart-area")
  .append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", width)
  .attr("height", height)

  // location of x and y axis bars
  Hist.append("g")
  .attr("transform", `translate(0,${height})`)
  .attr("id", "HxAxis")
  .call(xAxis)
  Hist.append("g")
  .attr("transform", `translate(${0},${yScale(yMax)})`)
  .call(yAxis)

  // got rid of the tick marks for zero because they looked bad (comment this out to see what I mean)
  Hist.selectAll('g.tick text').filter(function() {
        return d3.select(this).text() === "0";
    }).remove()
  


  // creating CoinFlip container
  CoinFlip = d3.select("#coin-flip")
  .attr("width", CFwidth + CFcenter_margin.left + CFcenter_margin.right)
  .attr("height", CFheight + CFcenter_margin.top + CFcenter_margin.bottom)
  .append("g")
  .attr("transform", "translate(" + CFcenter_margin.left + "," + CFcenter_margin.top + ")");

  // clips any svg's which are outside of this region
  CoinFlip.append("clipPath")
  .attr("id", "chart-area")
  .append("rect")
  .attr("x", 0)
  .attr("y", 0)
  .attr("width", CFwidth)
  .attr("height", CFheight)

  // location of x and y axis bars
  CoinFlip.append("g")
  .attr("class", "x-axis")
  .attr("transform", `translate(0,${CFheight})`)
  .call(CFxAxis)
  CoinFlip.append("g")
  .attr("class", "y-axis")
  .attr("transform", `translate(${0},${CFyScale(CFyMax)})`)
  .call(CFyAxis)

  // got rid of the tick marks for zero because they looked bad (comment this out to see what I mean)
  CoinFlip.selectAll('g.tick text').filter(function() {
        return d3.select(this).text() === "0";
    }).remove()
  
  // placing heads bar
  CoinFlip.append('rect')
  .attr('id', 'cfbar-head')
  .attr('x', -CFxScale(1)-15)
  .attr('y', -250)
  .attr('width', 30)
  .attr('height', 0)
  .attr('transform', 'rotate(-180)')
  .attr('fill', 'green');

  // placing tails bar
  CoinFlip.append('rect')
  .attr('id', 'cfbar-tail')
  .attr('x', -CFxScale(2)-15)
  .attr('y', -250)
  .attr('width', 30)
  .attr('height', 0)
  .attr('transform', 'rotate(-180)');


  // creating CoinFlip container
  CoinFlip.append('rect')
  .attr('id', 'meansbox')
  .attr('x', CFwidth-50)
  .attr('y', 0)
  .attr('width', 150)
  .attr('height', 60)
  .attr('fill-opacity', 0.5)

  // heads count text box
  CoinFlip.append("text")
    .attr("id", "heads_text")
    .attr("x",  CFwidth)
    .attr("y",  40)
    .attr("dy", ".35em")
    .attr('text-anchor', 'middle')
    .style("fill", "white")
    .text("0")
    .attr('color', 'white')

  // heads count text box text
  CoinFlip.append("text")
    .attr("id", "heads_text")
    .attr("x",  CFwidth)
    .attr("y",  25)
    .attr("dy", ".35em")
    .attr('text-anchor', 'middle')
    .style("fill", "white")
    .text("Heads Count")
    .attr('color', 'white')

})

</script>

<!-- placing the svg in the html section -->
<p>A set of size n coins</p>
<form>
	<input id='n-input' type="number" min="1" max="100" step="1" bind:value={inputValue} on:input={clearhBars} on:input={updateHxAxis}>
</form>
<button on:click={flipCoins(inputValue)}>Flip {inputValue} Coins</button>
<svg id='coin-flip'></svg>

<p>Flip a set of size {inputValue} coins 1000 times</p>
<button on:click={createBins(inputValue)} on:click={sim1000flipnCoins(inputValue)}>Flip {inputValue} Coins 1000 times</button>
<svg id='histogram-coin'></svg>


<style>

</style> 
