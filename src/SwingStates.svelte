<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
  
    const states = [
      "wisconsin", 
      "pennsylvania", 
      "north_carolina", 
      "arizona", 
      "georgia", 
      "michigan", 
      "nevada"
    ];
  
    let width, height;
    let stateData = {};
    let leftMargin = 30;
    let containerWidth;
  
    async function loadData(state) {
      const url = `https://raw.githubusercontent.com/tomvaillant/us-wahl-data/refs/heads/main/${state}.csv`;
      const data = await d3.csv(url, d => ({
        date: d3.timeParse("%Y-%m-%d")(d.date),
        trump: +d.trump_polling_average,
        harris: +d.harris_polling_average
      }));
      stateData[state] = data;
    }
  
    function updateDimensions() {
      const isMobile = window.innerWidth <= 960;
      containerWidth = document.querySelector('.chart-container').clientWidth;
      width = isMobile ? containerWidth : containerWidth * 0.5;
      height = width * 0.5;
      leftMargin = isMobile ? 50 : 30;
    }
  
    onMount(async () => {
      window.addEventListener('resize', () => {
        updateDimensions();
        redrawCharts();
      });
      updateDimensions();
  
      for (const state of states) {
        await loadData(state);
      }
      redrawCharts();
    });
  
    function drawGraph(svg, data, state) {
      const margin = { top: 20, right: 40, bottom: 50, left: leftMargin };
      const innerWidth = width - margin.left - margin.right;
      const innerHeight = height - margin.top - margin.bottom;
  
      const xScale = d3.scaleTime()
        .domain(d3.extent(data, d => d.date))
        .range([0, innerWidth]);
  
      const yScale = d3.scaleLinear()
        .domain([45, 50]) // Fixed domain for Y-axis (45 to 50)
        .range([innerHeight, 0]);
  
      const lineTrump = d3.line()
        .x(d => xScale(d.date))
        .y(d => yScale(d.trump));
  
      const lineHarris = d3.line()
        .x(d => xScale(d.date))
        .y(d => yScale(d.harris));
  
      const latestData = data[data.length - 1]; 
      const lead = latestData.harris - latestData.trump; // Calculate lead
      const leadingCandidate = lead > 0 ? "Harris" : "Trump";
      const trailingCandidate = lead > 0 ? "Trump" : "Harris";
      const absoluteLead = Math.abs(lead);
  
      const g = svg.append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

        // Title
      svg.append("text")
        .attr("x", margin.left + 10)
        .attr("y", margin.top + 10)
        .attr("class", "state-title-swing")
        .text(state.split('_')
            .map(word => word.charAt(0).toUpperCase() + word.slice(1))
            .join(' ')
        );
  
      // X-axis with CSS class for labels
      g.append("g")
        .attr("class", "x-axis")
        .attr("transform", `translate(0,${innerHeight + 30})`)
        .call(d3.axisBottom(xScale)
            .ticks(2)
            .tickFormat(d3.timeFormat("%b")) // Changed to only show month
            .tickSize(0)) // This removes the tick lines
        .call(g => g.select(".domain").remove()) // This removes the axis line
        .selectAll(".tick text")
        .attr("class", "x-axis-label-swing");
  
      // Y-axis with CSS class for labels, lowering them by 15px
      g.append("g")
        .attr("class", "y-axis")
        .call(d3.axisLeft(yScale)
            .tickValues([45, 50])
            .tickSize(0)
            .tickFormat(d => d + "%")) // Add percentage sign to tick labels
        .selectAll(".tick text")
        .attr("class", "y-axis-label-swing")
        .attr("transform", "translate(-5, 0)"); 
  
      // Remove the axis lines
      g.selectAll(".domain").remove();
  
      // Add Trump line with adjusted opacity
      g.append("path")
        .datum(data)
        .attr("fill", "none")
        .attr("stroke", "#D9312E")
        .attr("stroke-width", 2)
        .attr("opacity", lead > 0 ? 0.5 : 1) // Reduced opacity for non-leading line
        .attr("d", lineTrump);
  
      // Add Harris line with adjusted opacity
      g.append("path")
        .datum(data)
        .attr("fill", "none")
        .attr("stroke", "#4855DC")
        .attr("stroke-width", 2)
        .attr("opacity", lead > 0 ? 1 : 0.5) // Increased opacity for leading line
        .attr("d", lineHarris);
  
      // Add lead label above the leading line
      g.append("text")
        .attr("x", xScale(latestData.date) - 20)
        .attr("y", yScale(latestData[leadingCandidate.toLowerCase()]) - 30)
        .attr("text-anchor", "middle")
        .attr("fill", lead > 0 ? "#4855DC" : "#D9312E")
        .attr("class", "lead-label-swing")
        .style("font-size", "12px")
        .text(`${leadingCandidate} +${absoluteLead.toFixed(1)}`);

      // Add horizontal benchmark line
      const currentDate = new Date();
      const benchmarkY = yScale(latestData[trailingCandidate.toLowerCase()]);
      
      g.append("line")
        .attr("class", "benchmark-line-swing")
        .attr("x1", xScale(currentDate))
        .attr("x2", 0) // Position of y-axis
        .attr("y1", benchmarkY)
        .attr("y2", benchmarkY)
        .attr("stroke", "gray")
        .attr("stroke-width", 1)
        .attr("opacity", 0.8);
    }

    function redrawCharts() {
      states.forEach(state => {
        if (stateData[state]) {
          const svg = d3.select(`#${state} svg`);
          svg.selectAll("*").remove();
          svg.attr("width", width).attr("height", height);
          drawGraph(svg, stateData[state], state);
        }
      });
    }
  
    $: {
      if (containerWidth) {
        redrawCharts();
      }
    }
</script>
  
<div class="chart-container">
    {#each states as state}
      <div id={state} class="chart">
        <svg></svg>
      </div>
    {/each}
</div>
  
<style>
    .chart-container {
      width: 100%;
      margin-top: 5vh;
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 15px;
    }
  
    .chart {
      width: 100%;
      margin-bottom: 20px;
    }
  
    :global(.x-axis-label-swing) {
      font-size: 12px;
      fill: #07184D;
      opacity: 0.8;
    }
  
    :global(.y-axis-label-swing) {
      font-size: 10px;
      fill: #07184D;
      opacity: 0.8;
    }

    :global(.state-title-swing) {
        font-size: 16px;
        font-weight: medium;
        fill: #07184D;
    }

    :global(.lead-label-swing) {
        font-size: 14px;
        font-weight: bold;
    }

    :global(.benchmark-line-swing) {
        stroke-dasharray: 4;
    }
  
    @media (max-width: 960px) {
      .chart-container {
        grid-template-columns: 1fr;
      }
    }
</style>