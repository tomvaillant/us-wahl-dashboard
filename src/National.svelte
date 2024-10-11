<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
  
    const url =
      "https://raw.githubusercontent.com/tomvaillant/us-wahl-data/refs/heads/main/national.csv";
  
    let container;
    let svg;
  
    const trumpColor = "#D9312E";
    const harrisColor = "#4855DC";
  
    function createLabel(svgElement, color, candidate) {
      const label = svgElement
        .append("g")
        .attr("class", `label-group ${candidate.toLowerCase()}-label`)
        .style("pointer-events", "none");
  
      label.append("rect")
        .attr("fill", color)
        .attr("rx", 3)
        .attr("ry", 3)
        .attr("height", 25)
        .attr("y", -13);
  
      label.append("text")
        .attr("class", "label-text")
        .attr("fill", "white")
        .attr("alignment-baseline", "middle")
        .attr("font-weight", "bold")
        .attr("x", 5);
  
      label.append("text")
        .attr("class", "candidate-name")
        .attr("fill", color)
        .attr("alignment-baseline", "middle")
        .attr("font-weight", "bold")
        .text(candidate);
  
      return label;
    }
  
    function setLabelPositions(d, trumpLabel, harrisLabel, x, y) {
      const trumpY = y(d.trump);
      const harrisY = y(d.harris);
      const labelHeight = 20;
      const padding = 5;
  
      if (trumpY < harrisY) {
        trumpLabel.attr("transform", `translate(${x(d.date) + 5}, ${trumpY - labelHeight - padding})`);
        harrisLabel.attr("transform", `translate(${x(d.date) + 5}, ${harrisY + padding + 5})`);
      } else {
        trumpLabel.attr("transform", `translate(${x(d.date) + 5}, ${trumpY + padding + 8})`);
        harrisLabel.attr("transform", `translate(${x(d.date) + 5}, ${harrisY - labelHeight - padding + 5})`);
      }
    }
  
    async function drawChart() {
      const data = await d3.csv(url, (d) => ({
        date: d3.timeParse("%Y-%m-%d")(d.date),
        trump: +d.trump_polling_average,
        harris: +d.harris_polling_average,
      }));
  
      const width = container.clientWidth;
      const height = container.clientHeight * 0.9;
      const padding = Math.min(width, height) * 0.3;
      const paddingTop = padding * 0.8;
      const paddingLeft = padding * 0.4;
      const chartWidth = width - padding * 0.9;
      const chartHeight = height - padding * 1.2;
  
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
        .tickFormat((d, i) => (i === 3 ? "Election Day" : d3.timeFormat("%b")(d))) // %b for abbreviated month name
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
            .attr("class", "grid-line")
            .attr("stroke", "#e0e0e0")
            .attr("stroke-dasharray", "2,2")
        )
        .call((g) =>
          g
            .selectAll(".tick text")
            .attr("class", "x-axis-label")
            .attr("font-size", "12px")
            .style("opacity", 0.6)
        );
  
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
            .attr("class", "grid-line")
            .attr("stroke", "#e0e0e0")
            .attr("stroke-dasharray", "2,2")
        )
        .call((g) =>
          g
            .selectAll(".tick text")
            .attr("class", "y-axis-label")
            .attr("font-size", "14px")
            .style("opacity", 0.6)
        );
  
      const line = d3
        .line()
        .x((d) => x(d.date))
        .y((d) => y(d.value));
  
      const trumpData = data.map((d) => ({ date: d.date, value: d.trump }));
      const harrisData = data.map((d) => ({ date: d.date, value: d.harris }));
  
      svgElement
        .append("path")
        .datum(trumpData)
        .attr("fill", "none")
        .attr("stroke", trumpColor)
        .attr("stroke-width", 2)
        .attr("d", line);
  
      svgElement
        .append("path")
        .datum(harrisData)
        .attr("fill", "none")
        .attr("stroke", harrisColor)
        .attr("stroke-width", 2)
        .attr("d", line);
  
      // Add interactive elements
      const verticalLine = svgElement
        .append("line")
        .attr("class", "vertical-line")
        .attr("y1", 0)
        .attr("y2", chartHeight)
        .attr("stroke", "black")
        .attr("stroke-width", 1)
        .style("pointer-events", "none");
  
      const dateLabel = svgElement
        .append("text")
        .attr("class", "date-label")
        .attr("text-anchor", "middle")
        .style("pointer-events", "none");
  
      // Add lead indicator
      const leadIndicator = svgElement
        .append("text")
        .attr("class", "lead-indicator")
        .attr("text-anchor", "middle")
        .attr("font-weight", "bold")
        .style("pointer-events", "none");
  
      // Create labels
      const trumpLabel = createLabel(svgElement, trumpColor, "Trump");
      const harrisLabel = createLabel(svgElement, harrisColor, "Harris");
  
      const overlay = svgElement.append("rect")
        .attr("class", "overlay")
        .attr("width", 0)
        .attr("height", chartHeight)
        .attr("fill", "#f5f5f5")
        .attr("opacity", 0.7);
  
      trumpLabel.raise();
      harrisLabel.raise();
  
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
            .attr("y", -30)
            .text(d3.timeFormat("%b %d")(d.date));
  
          const lead = Math.abs(d.harris - d.trump).toFixed(1);
          const leadCandidate = d.harris > d.trump ? "Harris" : "Trump";
          const leadColor = d.harris > d.trump ? harrisColor : trumpColor;
  
          leadIndicator
            .attr("x", x(d.date))
            .attr("y", -10)
            .attr("fill", leadColor)
            .text(`${leadCandidate} +${lead}`);
  
          setLabelPositions(d, trumpLabel, harrisLabel, x, y);
  
          trumpLabel.select(".label-text").text(`${d.trump.toFixed(1)}%`);
          trumpLabel.select("rect")
            .attr("width", trumpLabel.select(".label-text").node().getBBox().width + 10);
          trumpLabel.select(".candidate-name")
            .attr("x", trumpLabel.select("rect").attr("width"))
            .attr("dx", 5);
  
          harrisLabel.select(".label-text").text(`${d.harris.toFixed(1)}%`);
          harrisLabel.select("rect")
            .attr("width", harrisLabel.select(".label-text").node().getBBox().width + 10);
          harrisLabel.select(".candidate-name")
            .attr("x", harrisLabel.select("rect").attr("width"))
            .attr("dx", 5);
  
          // Update overlay rectangle
          overlay
            .attr("x", x(d.date))
            .attr("width", chartWidth - x(d.date));
        })
        .on("mouseleave", function () {
          // Reset to last data point when mouse leaves the chart
          const lastData = data[data.length - 1];
          verticalLine.attr("x1", x(lastData.date)).attr("x2", x(lastData.date));
          dateLabel
            .attr("x", x(lastData.date))
            .attr("y", -30)
            .text(d3.timeFormat("%b %d")(d.date));
          
          const lead = Math.abs(lastData.harris - lastData.trump).toFixed(1);
          const leadCandidate = lastData.harris > lastData.trump ? "Harris" : "Trump";
          const leadColor = lastData.harris > lastData.trump ? harrisColor : trumpColor;
  
          leadIndicator
            .attr("x", x(lastData.date))
            .attr("y", -10)
            .attr("fill", leadColor)
            .text(`${leadCandidate} +${lead}`);
  
          setLabelPositions(lastData, trumpLabel, harrisLabel, x, y);
  
          trumpLabel.select(".label-text").text(`${lastData.trump.toFixed(1)}%`);
          trumpLabel.select("rect")
            .attr("width", trumpLabel.select(".label-text").node().getBBox().width + 10);
          trumpLabel.select(".candidate-name")
            .attr("x", trumpLabel.select("rect").attr("width"))
            .attr("dx", 5);
  
          harrisLabel.select(".label-text").text(`${lastData.harris.toFixed(1)}%`);
          harrisLabel.select("rect")
            .attr("width", harrisLabel.select(".label-text").node().getBBox().width + 10);
          harrisLabel.select(".candidate-name")
            .attr("x", harrisLabel.select("rect").attr("width"))
            .attr("dx", 5);
          
          // Reset overlay rectangle
          overlay.attr("x", x(lastData.date)).attr("width", chartWidth - x(lastData.date));
        });
  
      // Initialize with last data point
      const lastData = data[data.length - 1];
      verticalLine.attr("x1", x(lastData.date)).attr("x2", x(lastData.date));
      dateLabel
        .attr("x", x(lastData.date))
        .attr("y", -30)
        .text(d3.timeFormat("%b %d")(lastData.date));
  
      const lead = Math.abs(lastData.harris - lastData.trump).toFixed(1);
      const leadCandidate = lastData.harris > lastData.trump ? "Harris" : "Trump";
      const leadColor = lastData.harris > lastData.trump ? harrisColor : trumpColor;
  
      leadIndicator
        .attr("x", x(lastData.date))
        .attr("y", -10)
        .attr("fill", leadColor)
        .text(`${leadCandidate} +${lead}`);
  
      setLabelPositions(lastData, trumpLabel, harrisLabel, x, y);
  
      trumpLabel.select(".label-text").text(`${lastData.trump.toFixed(1)}%`);
      trumpLabel.select("rect")
        .attr("width", trumpLabel.select(".label-text").node().getBBox().width + 10);
      trumpLabel.select(".candidate-name")
        .attr("x", trumpLabel.select("rect").attr("width"))
        .attr("dx", 5);
  
      harrisLabel.select(".label-text").text(`${lastData.harris.toFixed(1)}%`);
      harrisLabel.select("rect")
        .attr("width", harrisLabel.select(".label-text").node().getBBox().width + 10);
      harrisLabel.select(".candidate-name")
        .attr("x", harrisLabel.select("rect").attr("width"))
        .attr("dx", 5);
  
      overlay.attr("x", x(lastData.date)).attr("width", chartWidth - x(lastData.date));
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
      font-family: "BatonTurbo", Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
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
  
    :global(.chart-wrapper .axis-label) {
      font-family: "BatonTurbo", Inter, system-ui, Avenir, Helvetica,
    }

    @media (max-width: 768px) {
    :global(.y-axis-label) {
      font-size: 12px;
    }

    :global(.x-axis-label) {
      font-size: 10px;
    }

    :global(.label-text) {
      font-size: 10px;
    }

    :global(.candidate-name) {
      font-size: 10px;
    }

    :global(.date-label) {
      font-size: 12px;
      font-weight: bold;
    }

    :global(.lead-indicator) {
      font-size: 12px;
    }
  }
</style>