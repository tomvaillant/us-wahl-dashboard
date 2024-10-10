<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";

  const url =
    "https://raw.githubusercontent.com/tomvaillant/us-wahl-data/refs/heads/main/national.csv";

  let container;
  let svg;

  async function drawChart() {
    const data = await d3.csv(url, (d) => ({
      date: d3.timeParse("%Y-%m-%d")(d.date),
      trump: +d.trump_polling_average,
      harris: +d.harris_polling_average,
    }));

    const width = container.clientWidth;
    const height = container.clientHeight * 0.75;
    const padding = Math.min(width, height) * 0.1;
    const paddingTop = padding + height * 0.1;
    const paddingLeft = padding + 10;
    const chartWidth = width - 3 * padding;
    const chartHeight = height - 2 * padding;

    d3.select(svg).selectAll("*").remove();

    const svgElement = d3
      .select(svg)
      .attr("width", width)
      .attr("height", height)
      .append("g")
      .attr("transform", `translate(${paddingLeft},${paddingTop})`);

    const lastDataDate = d3.max(data, (d) => d.date);
    const electionDay = new Date(lastDataDate.getFullYear(), 10, 5); // November 5th

    const x = d3
      .scaleTime()
      .domain([d3.min(data, (d) => d.date), electionDay])
      .range([0, chartWidth]);

    const y = d3.scaleLinear().domain([35, 55]).range([chartHeight, 0]);

    // Custom X-axis
    const xAxis = d3
      .axisBottom(x)
      .tickValues([
        new Date(lastDataDate.getFullYear(), 7, 1), // August 1st
        new Date(lastDataDate.getFullYear(), 8, 1), // September 1st
        new Date(lastDataDate.getFullYear(), 9, 1), // October 1st
        electionDay,
      ])
      .tickFormat((d, i) => (i === 3 ? "Election Day" : d3.timeFormat("%B")(d)))
      .tickSize(-chartHeight);

    svgElement
      .append("g")
      .attr("class", "x-axis")
      .attr("transform", `translate(0,${chartHeight})`)
      .call(xAxis)
      .call((g) => g.select(".domain").remove())
      .call((g) =>
        g
          .selectAll(".tick line")
          .attr("stroke", "#e0e0e0")
          .attr("stroke-dasharray", "2,2")
      );

    // Adjust X-axis labels
    svgElement
      .select(".x-axis")
      .selectAll(".tick text")
      .attr("y", 10)
      .attr("dy", ".71em")
      .style("text-anchor", "middle");

    // Custom Y-axis
    const yAxis = d3
      .axisLeft(y)
      .tickValues([40, 50])
      .tickFormat((d) => d + "%")
      .tickSize(-chartWidth);

    svgElement
      .append("g")
      .attr("class", "y-axis")
      .call(yAxis)
      .call((g) => g.select(".domain").remove())
      .call((g) =>
        g
          .selectAll(".tick line")
          .attr("stroke", "#e0e0e0")
          .attr("stroke-dasharray", "2,2")
      );

    const line = d3
      .line()
      .x((d) => x(d.date))
      .y((d) => y(d.value));

    // Create a clip path
    svgElement
      .append("clipPath")
      .attr("id", "clip")
      .append("rect")
      .attr("width", chartWidth)
      .attr("height", chartHeight);

    const trumpData = data.map((d) => ({ date: d.date, value: d.trump }));
    const harrisData = data.map((d) => ({ date: d.date, value: d.harris }));

    // Apply clip path to trend lines
    const trendLines = svgElement.append("g").attr("clip-path", "url(#clip)");

    trendLines
      .append("path")
      .datum(trumpData)
      .attr("fill", "none")
      .attr("stroke", "red")
      .attr("stroke-width", 2)
      .attr("d", line);

    trendLines
      .append("path")
      .datum(harrisData)
      .attr("fill", "none")
      .attr("stroke", "blue")
      .attr("stroke-width", 2)
      .attr("d", line);

    // Add interactive elements
    const verticalLine = svgElement
      .append("line")
      .attr("y1", 0)
      .attr("y2", chartHeight)
      .attr("stroke", "black")
      .attr("stroke-width", 1)
      .style("pointer-events", "none");

    const dateLabel = svgElement
      .append("text")
      .attr("text-anchor", "middle")
      .attr("font-size", "12px")
      .style("pointer-events", "none");

    const trumpLabel = svgElement
      .append("text")
      .attr("fill", "red")
      .attr("font-size", "12px")
      .attr("alignment-baseline", "middle")
      .style("pointer-events", "none");

    const harrisLabel = svgElement
      .append("text")
      .attr("fill", "blue")
      .attr("font-size", "12px")
      .attr("alignment-baseline", "middle")
      .style("pointer-events", "none");

    // Add invisible rect for mouse tracking
    svgElement
      .append("rect")
      .attr("width", chartWidth)
      .attr("height", chartHeight)
      .style("fill", "none")
      .style("pointer-events", "all")
      .on("mousemove", function (event) {
        const [xPos] = d3.pointer(event, this);
        const date = x.invert(xPos);
        const bisect = d3.bisector((d) => d.date).left;
        const index = bisect(data, date);
        const d0 = data[index - 1];
        const d1 = data[index];
        const d = date - d0.date > d1.date - date ? d1 : d0;

        verticalLine.attr("x1", x(d.date)).attr("x2", x(d.date));
        dateLabel
          .attr("x", x(d.date))
          .attr("y", -5)
          .text(d3.timeFormat("%b %d, %Y")(d.date));
        trumpLabel
          .attr("x", x(d.date) + 5)
          .attr("y", y(d.trump))
          .text(`${d.trump.toFixed(1)}%`);
        harrisLabel
          .attr("x", x(d.date) + 5)
          .attr("y", y(d.harris))
          .text(`${d.harris.toFixed(1)}%`);

        // Update clip path
        d3.select("#clip rect").attr("width", x(d.date));
      })
      .on("mouseleave", function () {
        // Reset to last data point when mouse leaves the chart
        const lastData = data[data.length - 1];
        verticalLine.attr("x1", x(lastData.date)).attr("x2", x(lastData.date));
        dateLabel
          .attr("x", x(lastData.date))
          .attr("y", -5)
          .text(d3.timeFormat("%b %d, %Y")(lastData.date));
        trumpLabel
          .attr("x", x(lastData.date) + 5)
          .attr("y", y(lastData.trump))
          .text(`${lastData.trump.toFixed(1)}%`);
        harrisLabel
          .attr("x", x(lastData.date) + 5)
          .attr("y", y(lastData.harris))
          .text(`${lastData.harris.toFixed(1)}%`);
        d3.select("#clip rect").attr("width", chartWidth);
      });

    // Initialize with last data point
    const lastData = data[data.length - 1];
    verticalLine.attr("x1", x(lastData.date)).attr("x2", x(lastData.date));
    dateLabel
      .attr("x", x(lastData.date))
      .attr("y", -5)
      .text(d3.timeFormat("%b %d, %Y")(lastData.date));
    trumpLabel
      .attr("x", x(lastData.date) + 5)
      .attr("y", y(lastData.trump))
      .text(`${lastData.trump.toFixed(1)}%`);
    harrisLabel
      .attr("x", x(lastData.date) + 5)
      .attr("y", y(lastData.harris))
      .text(`${lastData.harris.toFixed(1)}%`);
  }

  onMount(() => {
    drawChart();

    const resizeObserver = new ResizeObserver(() => {
      drawChart();
    });

    resizeObserver.observe(container);

    return () => resizeObserver.disconnect();
  });
</script>

<div class="chart-container">
  <div class="chart-wrapper" bind:this={container}>
    <svg bind:this={svg}></svg>
  </div>
</div>

<style>
  .chart-container {
    width: 100%;
    font-family: "BatonTurbo", Inter, system-ui, Avenir, Helvetica, Arial,
      sans-serif;
  }

  .chart-wrapper {
    width: 100%;
    height: 40vh;
    background-color: #f5f5f5;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
  }

  svg {
    width: 100%;
    height: 100%;
  }
  :global(.chart-wrapper text) {
    font-family: "BatonTurbo", Inter, system-ui, Avenir, Helvetica, Arial,
      sans-serif;
  }

  /* :global(.x-axis text),
:global(.y-axis text) {
  font-family: 'BatonTurbo', Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
  font-weight: 400;
}

:global(.date-label),
:global(.value-label) {
  font-family: 'BatonTurbo', Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
  font-weight: 600;
} */
</style>
