<script>
import * as d3 from 'd3';
import { onMount } from 'svelte';
let mounted = false;

let Hist, CoinFlip;
let inputValue = 50;
let singular_or_plural = 'coins'

// histogram center margin
const Hcenter_margin = {top: 50, right: 50, bottom: 50, left: 50}, // graph size in terms of like display pixels
  width = 450 - Hcenter_margin.left - Hcenter_margin.right,
  height = 250 - Hcenter_margin.top - Hcenter_margin.bottom;


let xMin = 0; // graph limits in terms of axis coordinates
let xMax = inputValue;
let yMin = 0;
let yMax = 0.4;

let xScale = d3.scaleLinear([xMin, xMax], [0, width]) // function that maps math coordinates to display coords
let yScale = d3.scaleLinear([yMin, yMax], [height, 0])
let xAxis = d3.axisBottom(xScale) // svg object for the x-axis
xAxis.ticks(inputValue/2)
let yAxis = d3.axisLeft(yScale) // same as above but for the y-axis
yAxis.ticks(5)

let disallow100Flip = false;

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

let CFUpdatesEnabled = true;

let flip_sets = 0;

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
}

async function flipCoins(iV) {
  let previous_data = [{ label: 'Heads', count: headsCount }, { label: 'Tails', count: tailsCount }];
  headsCount = 0;
  tailsCount = 0;
  for (let i = 0; i < iV; i++) {
    flipCoin();
    // await pause(5000/iV);
  }
  animateUpdateBars(previous_data);
}

function deMoivreMath(k, N) {
  return Math.sqrt(((Math.PI*0.5*N)**(-1)))*(Math.E**(-((k-0.5*N)**2)/(0.5*N)))
}

function graphDeMovirePDF(start_idx, end_idx, step, N) {
  const data = [];
  for (let x = start_idx; x <= end_idx; x += step) {
    let y = deMoivreMath(x, N);
    data.push([x, y])
  }
  return data;
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
  disallow100Flip = true;
  clearhBars();
  flip_sets = 0;
  updateBarsrunning = true
  simheadsCount = 0;
  // run 1000 simulations of iV coin flips
  
  for (let i = 0; i < 1000; i++){
    flip_sets = i+1;
    if (updateBarsrunning){
    simheadsCount = 0;
    // single sim of iV coinflips
    for (let j = 0; j < iV; j++) {
      let outcome = Math.random() < 0.5 ? 'Heads' : 'Tails';
      if (outcome === 'Heads') {
        simheadsCount++;
      }
    }
    } else {
      return
    }

    binDict[simheadsCount] = binDict[simheadsCount]+1
    let freqDict = {};
    for (let key in binDict) {
      if (binDict.hasOwnProperty(key)) {
        freqDict[key] = (binDict[key]/i)
      }
    }
    await pause(5000/(iV*10000));
    updateHistogramBars(freqDict)
  }
  disallow100Flip = false;
}


function updateHistogramBars(source_dict) {
  // Select all existing bars and bind data
  const bars = Hist.selectAll('.hist-bar')
    .data(Object.entries(source_dict));

  // Enter selection: create new bars
  bars.enter()
    .append('rect')
    .attr('class', 'hist-bar')
    .attr('x', d => xScale(parseInt(d[0])) - xScale(0.5))
    .attr('width', xScale(1))
    .attr('y', d => yScale(d[1]))
    .attr('height', d => height - yScale(d[1]))
    .attr('fill', '#CD7F32')
    .attr('fill-opacity', 0.8)

  // Update selection: update existing bars
  bars
    .attr('x', d => xScale(parseInt(d[0])) - xScale(0.5))
    .attr('width', xScale(1))
    .attr('y', d => yScale(d[1]))
    .attr('height', d => height - yScale(d[1]));

  // const maxCount = Math.max(...Object.values(binDict));

  // // Update the y-scale domain
  // yScale.domain([0, maxCount]);

  // // Redraw the y-axis
  // Hist.select("g.y-axis")
  //   .transition()
  //   .duration(5)
  //   .call(yAxis);

  // Exit selection: remove bars not needed anymore
  bars.exit().remove();
}

function clearhBars(){
  updateBarsrunning = false
  const bars = Hist.selectAll('.hist-bar').remove()
}

async function animateUpdateBars(previous_data) {
  let new_data = [{label: 'Heads', count: headsCount}, {label: 'Tails', count: tailsCount}]
  let heads_range = (new_data[0]['count'] - previous_data[0]['count'])
  let tails_range = (new_data[1]['count'] - previous_data[1]['count'])
  let first_update = !(Math.abs(heads_range) == Math.abs(tails_range))
  CFUpdatesEnabled = false;
  for (let i = 1; i <= 50; i+= 1) {
    let loop_percent = i/50;
    let heads_interp = previous_data[0]['count'] + (loop_percent * heads_range)
    let tails_interp = previous_data[1]['count'] + (loop_percent * tails_range)
    let interp_data = [{label: 'Heads', count: heads_interp}, {label: 'Tails', count: tails_interp}]
    updateBars(interp_data, first_update)
    await pause(10)
  }
  CFUpdatesEnabled = true;
  updateBars(new_data, first_update)
}

function updateBars(data, first_update) {
  const hbars = CoinFlip.select('#cfbar-head')
  .data(data);
  
  // Select tail bars and bind data
  const tbars = CoinFlip.select('#cfbar-tail')
  .data(data);

  if(CFUpdatesEnabled) {
    CoinFlip.select("#heads_text").text(data.find(item => item.label === 'Heads').count);
  }

  // Calculate the maximum count for y-axis scale
  let maxCount;
  if(first_update) {
    maxCount = Math.max(headsCount, tailsCount)
  }
  else {
    maxCount = Math.max(data[0]['count'], data[1]['count']);
  }

  // Update y-axis scale domain
  CFyScale.domain([0, maxCount]);
  
  // Redraw y-axis
  CoinFlip.select("g.y-axis").call(CFyAxis);

  // Update existing bars
  try {
    hbars.attr('height',CFheight - CFyScale(data.find(item => item.label === 'Heads').count) - 15);
    tbars.attr('height',CFheight - CFyScale(data.find(item => item.label === 'Tails').count) - 15);
  }
  catch(e){
    null;
  }

  CoinFlip.selectAll(".coin").remove()

  CoinFlip.append("svg:image")
  .attr('class', 'coin')
  .attr('x', 102)
  .attr('y', 250 - (CFheight - CFyScale(data.find(item => item.label === 'Heads').count)))
  .attr('width', 30)
  .attr('height', 30)
  .attr("xlink:href", "images/heads.png")

  CoinFlip.append("svg:image")
  .attr('class', 'coin')
  .attr('x', 218)
  .attr('y', 250 - (CFheight - CFyScale(data.find(item => item.label === 'Tails').count)))
  .attr('width', 30)
  .attr('height', 30)
  .attr("xlink:href", "images/tails.png")
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
  .attr("id", "HyAxis")
  .call(yAxis)

  Hist.append("text")
  .attr("class", "x-label")
  .attr("text-anchor", "middle")
  .attr("x", width / 2)
  .attr("y", height + 35)
  .text("Number of Heads");
  Hist.append("text")
  .attr("class", "y-label")
  .attr("text-anchor", "middle")
  .attr("y", -35)
  .attr("x", -height / 2)
  .attr("transform", "rotate(-90)")
  .html("Density")

    // creating CoinFlip container
  Hist.append('rect')
  .attr('x', width-170)
  .attr('y', -20)
  .attr('width', 170)
  .attr('height', 54)
  .attr('fill-opacity', 0.5)

  Hist.append('rect')
  .attr('x', width-160)
  .attr('y', -10)
  .attr('width', 10)
  .attr('height', 10)
  .attr('fill', '#628575')
  .attr('stroke', 'white')

  // heads count text box
  Hist.append("text")
    .attr("id", "heads_text")
    .attr("x",  width-140)
    .attr("y",  -5)
    .attr("dy", ".35em")
    .attr('text-anchor', 'left')
    .style("fill", "white")
    .text("De Moivre's Guess")
    .attr('color', 'white')

  Hist.append('rect')
  .attr('x', width-160)
  .attr('y', 10)
  .attr('width', 10)
  .attr('height', 10)
  .attr('fill', '#CD7F32')
  .attr('stroke', 'white')

  // heads count text box
  Hist.append("text")
    .attr("id", "heads_text")
    .attr("x",  width-140)
    .attr("y",  15)
    .attr("dy", ".35em")
    .attr('text-anchor', 'left')
    .style("fill", "white")
    .text("Our Results")
    .attr('color', 'white')

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
  CoinFlip.append("text")
  .attr("class", "y-label")
  .attr("text-anchor", "middle")
  .attr("y", -35)
  .attr("x", -CFheight / 2)
  .attr("transform", "rotate(-90)")
  .html("Count")

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
  .attr('fill', '#CD7F32');

  // placing tails bar
  CoinFlip.append('rect')
  .attr('id', 'cfbar-tail')
  .attr('x', -CFxScale(2)-15)
  .attr('y', -250)
  .attr('width', 30)
  .attr('height', 0)
  .attr('transform', 'rotate(-180)')
  .attr('fill', '#a1c7b6');


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

$: if(inputValue == 1) {
  singular_or_plural = 'coin';
}

$: if(inputValue > 1) {
  singular_or_plural = 'coins';
}

$: if(mounted) {
  let line = d3.line()
    .x(d => xScale(d[0]))
    .y(d => yScale(d[1]))

  Hist.selectAll("path.function").remove()
  Hist.selectAll("circle").remove()

  let current_data = graphDeMovirePDF(xMin, xMax, 1, xMax)

  Hist.append("path")
    .datum(current_data)
    .attr("class", "function")
    .attr("fill", "none")
    .attr("stroke", "#628575")
    .attr("stroke-width", 2)
    .attr("d", line);

  Hist.selectAll(".point")
    .data(current_data)
    .enter().append("circle")
    .attr("class", "point")
    .attr("cx", d => xScale(d[0]))
    .attr("cy", d => yScale(d[1]))
    .attr("r", (4-0.8*(((xMax/10)-1)**(0.5))))
    .attr("fill", "#628575");
  }


$: if(mounted) {
  let yScale = d3.scaleLinear([yMin, yMax], [height, 0])
  let yAxis = d3.axisLeft(yScale) // same as above but for the y-axis
  yAxis.ticks(5)

    Hist.append("g")
  .attr("transform", `translate(${0},${yScale(yMax)})`)
  .attr("id", "HyAxis")
  .call(yAxis)
}

$: if(mounted) {
  inputValue == inputValue;
  flip_sets = 0;
  disallow100Flip = false;
}

</script>
<div class='main'>
<!-- placing the svg in the html section -->
  <p>A set of size n coins</p>
  <form>
    <input
            id="n-input"
            type="range"
            style="width: 100%; margin: 0 auto;"
            min={10}
            max={100}
            step="1"
            bind:value={inputValue}
            on:input={clearhBars}
            on:input={updateHxAxis}
        />
  </form>
  <br>
  <button class='button-30' disabled={!CFUpdatesEnabled} on:click={flipCoins(inputValue)}>Flip {inputValue} {singular_or_plural}</button>
  <svg id='coin-flip'></svg>

  <p>Flip {inputValue} {singular_or_plural} 1000 times</p>
  <p>Flips: {flip_sets}</p>
  <button class='button-30' disabled={disallow100Flip} on:click={createBins(inputValue)} on:click={sim1000flipnCoins(inputValue)}>Flip {inputValue} Coins 1000 times</button>
  <br>
  <svg id='histogram-coin'></svg>
</div>


<style>
.main {
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
input[type="range"]::-webkit-slider-thumb:active {
  transform: translateY(1px)
}
</style> 
