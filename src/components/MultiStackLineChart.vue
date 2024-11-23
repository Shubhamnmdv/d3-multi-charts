<template>
  <div>
    <h1 class="heading">Multi Stack Line Chart</h1>
    <svg id="mini-map"></svg>
    <svg id="Multi-Stack-line-chart"></svg>
    <div id="tooltip" class="tooltip" style="opacity: 0;"></div>
  </div>
</template>

<script>
import * as d3 from "d3";
import multiStackLineChart from "../mockData/multiStackLineChart.json";

export default {
  name: "MultiStackLineChart",
  mounted() {
    this.drawChart();
  },
  methods: {
    drawChart() {
      const margin = { top: 10, right: 30, bottom: 30, left: 60 };
      const width = 1200 - margin.left - margin.right;
      const height = 500 - margin.top - margin.bottom;
      const miniMapHeight = 100;

      // Prepare and format the data
      const data = multiStackLineChart.map((d) => ({
        date: new Date(d.date * 1000), // Convert Unix timestamp to JavaScript Date
        value: d.close, // Using the 'close' value as the main data point
      }));

      // Scales
      const xScale = d3.scaleTime().domain(d3.extent(data, (d) => d.date)).range([0, width]);
      const yScale = d3.scaleLinear().domain([0, d3.max(data, (d) => d.value)]).range([height, 0]);

      const xMiniScale = d3.scaleTime().domain(d3.extent(data, (d) => d.date)).range([0, width]);
      const yMiniScale = d3
        .scaleLinear()
        .domain([0, d3.max(data, (d) => d.value)])
        .range([miniMapHeight, 0]);

      // Main SVG
      const svg = d3
        .select("#Multi-Stack-line-chart")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top})`);

      // Mini-map SVG
      const miniMap = d3
        .select("#mini-map")
        .attr("width", width + margin.left + margin.right)
        .attr("height", miniMapHeight + margin.top + 25)
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top / 2})`);

      // Draw Main Line Chart
      const linePath = svg
        .append("path")
        .datum(data)
        .attr("fill", "none")
        .attr("stroke", "steelblue")
        .attr("stroke-width", 1.5)
        .attr(
          "d",
          d3
            .line()
            .x((d) => xScale(d.date))
            .y((d) => yScale(d.value))
        );

      // Main Axes
      const xAxis = svg
        .append("g")
        .attr("class", "x-axis")
        .attr("transform", `translate(0, ${height})`)
        .call(d3.axisBottom(xScale));

      svg.append("g").call(d3.axisLeft(yScale));

      // Mini-map Area Chart
      const area = d3
        .area()
        .x((d) => xMiniScale(d.date))
        .y0(yMiniScale(0))
        .y1((d) => yMiniScale(d.value));

      miniMap
        .append("path")
        .datum(data)
        .attr("fill", "lightblue")
        .attr("stroke", "steelblue")
        .attr("d", area);

      miniMap
        .append("g")
        .attr("transform", `translate(0, ${miniMapHeight})`)
        .call(d3.axisBottom(xMiniScale));

      // Brush
      const brush = d3
        .brushX()
        .extent([
          [0, 0],
          [width, miniMapHeight],
        ])
        .on("brush end", brushed);

      miniMap.append("g").attr("class", "brush").call(brush).call(brush.move, xMiniScale.range());

      function brushed(event) {
        const selection = event.selection;
        if (selection) {
          const [x0, x1] = selection.map(xMiniScale.invert);
          xScale.domain([x0, x1]);

          // Update the main chart line
          linePath
            .transition()
            .duration(500)
            .attr(
              "d",
              d3
                .line()
                .x((d) => xScale(d.date))
                .y((d) => yScale(d.value))
            );

          // Update x-axis
          xAxis.transition().duration(500).call(d3.axisBottom(xScale));
        }
      }

      // Reset brush and zoom
      svg.on("dblclick", resetZoom);
      document.addEventListener("click", (event) => {
        if (!event.target.closest("svg")) {
          resetZoom();
        }
      });

      function resetZoom() {
        xScale.domain(d3.extent(data, (d) => d.date));

        // Reset the main chart
        linePath
          .transition()
          .duration(500)
          .attr(
            "d",
            d3
              .line()
              .x((d) => xScale(d.date))
              .y((d) => yScale(d.value))
          );

        // Reset x-axis
        xAxis.transition().duration(500).call(d3.axisBottom(xScale));

        // Reset the brush selection
        miniMap.select(".brush").call(brush.move, xMiniScale.range());
      }

      // Tooltip and Vertical Line
      const tooltip = d3.select("#tooltip");
      const verticalLine = svg
        .append("line")
        .attr("class", "vertical-line")
        .attr("y1", 0)
        .attr("y2", height)
        .attr("stroke", "gray")
        .attr("stroke-width", 1)
        .attr("stroke-dasharray", "4")
        .attr("opacity", 0);

      // Mouse Event Handlers
      svg
        .append("rect")
        .attr("width", width)
        .attr("height", height)
        .attr("fill", "none")
        .attr("pointer-events", "all")
        .on("mousemove", function (event) {
          const [mouseX] = d3.pointer(event);
          const xDate = xScale.invert(mouseX);
          const closestData = data.reduce((a, b) => {
            return Math.abs(a.date - xDate) < Math.abs(b.date - xDate) ? a : b;
          });

          // Update Tooltip Position and Text
          tooltip
            .style("opacity", 1)
            .html(`Value: ${closestData.value}<br>Date: ${closestData.date.toLocaleDateString()}`)
            .style("left", `${event.pageX + 15}px`)
            .style("top", `${event.pageY - 30}px`);

          // Update Vertical Line Position
          verticalLine
            .attr("x1", xScale(closestData.date))
            .attr("x2", xScale(closestData.date))
            .attr("opacity", 1);
        })
        .on("mouseleave", function () {
          tooltip.style("opacity", 0);
          verticalLine.attr("opacity", 0);
        });
    },
  },
};
</script>

<style scoped>
.heading {
  font-size: 40px; /* Adjust font size */
  font-weight: bold;
  color: #003366; /* Dark blue color similar to the reference image */
  margin-bottom: 20px; /* Add some space between the heading and the chart */
}
.tooltip {
  position: absolute;
  background-color: #fff;
  border: 1px solid #d3d3d3;
  padding: 10px;
  border-radius: 4px;
  pointer-events: none;
  font-size: 12px;
}
</style>
