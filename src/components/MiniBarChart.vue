<template>
  <div>
    <h1 class="heading">Mini Bar Chart</h1>
    <div id="chart"></div>
  </div>
</template>

<script>
import * as d3 from "d3";
import miniBarChartMock from "../mockData/miniBarChartMock.json"; // Adjust the file path accordingly

export default {
  name: "MiniBarChart",
  mounted() {
    this.drawChart();
  },
  methods: {
    drawChart() {
      // Set the dimensions and margins of the graph
      const margin = { top: 20, right: 60, bottom: 90, left: 60 };
      const width = 1800 - margin.left - margin.right;
      const height = 700 - margin.top - margin.bottom;

      // Append the SVG object to the div with id "chart"
      const svg = d3
        .select("#chart")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

      // Add a background rectangle for the grid area
      svg
        .append("rect")
        .attr("x", 0)
        .attr("y", 0)
        .attr("width", width)
        .attr("height", height)
        .attr("fill", "white"); // Light grey background for grid area

      // Format the data
      const data = miniBarChartMock.map((d) => ({
        date: d.date.split("#")[0], // Extract date from your format
        value: +d.value, // Ensure value is numeric
      }));

      // X axis
      const x = d3
        .scaleBand()
        .range([0, width])
        .domain(data.map((d) => d.date))
        .padding(0.2);

      const xAxis = svg
        .append("g")
        .attr("transform", `translate(0,${height})`)
        .call(d3.axisBottom(x));

      // Format X-axis tick labels
      xAxis
        .selectAll("text")
        .attr("transform", "translate(-10,0)rotate(-45)")
        .style("text-anchor", "end");

        svg.selectAll("path.domain").attr("stroke", "white");

        

      // Y axis
      const y = d3
        .scaleLinear()
        .domain([
          d3.min(data, (d) => d.value) * 1.1, // Extend the domain slightly for aesthetics
          d3.max(data, (d) => d.value) * 1.1,
        ])
        .range([height, 0]);


      // Add Y-axis grid lines (BEFORE adding bars)
      svg
        .append("g")
        .attr("class", "grid")
        .call(
          d3
            .axisLeft(y)
            .tickSize(-width) // Extend grid lines across the chart
            .tickFormat("") // Remove tick labels from the grid
        )
        .selectAll("line")
        .style("stroke", "#d3d3d3") // Light grey grid lines
        .style("stroke-dasharray", "4 4");

      // Remove Y-axis domain line
      svg.selectAll(".grid .domain").remove();

      // Add Y-axis ticks on the right side
      svg
        .append("g")
        .attr("transform", `translate(${width},0)`) // Move Y-axis to the right side
        .call(
          d3.axisRight(y).tickFormat((d) => `$${(d / 1_000_000).toFixed(1)}M`) // Format ticks as dollar millions
        )
        .selectAll("path.domain")
        .attr("stroke", "white"); // Change the color of the domain line to white

      // Add a zero line
      svg
        .append("line")
        .attr("x1", 0)
        .attr("x2", width)
        .attr("y1", y(0))
        .attr("y2", y(0))
        .attr("stroke", "black")
        .attr("stroke-width", 1)
        .style("stroke-dasharray", "4 2"); // Dashed line for zero axis

      // Bars with animation
      svg
        .selectAll("mybar")
        .data(data)
        .enter()
        .append("rect")
        .attr("x", (d) => x(d.date))
        .attr("width", x.bandwidth())
        .attr("y", y(0)) // Start at zero for animation
        .attr("height", 0) // Start with height 0 for animation
        .attr("fill", (d) =>
          d.value >= 0
            ? "rgba(122,141,156,0.6274509803921569)"
            : "rgba(200,35,51,0.6274509803921569)"
        ) // Use different colors for positive and negative values
        .transition() // Add animation
        .duration(300) // Duration of animation in ms
        .attr("y", (d) => (d.value >= 0 ? y(d.value) : y(0))) // Final position of bars
        .attr("height", (d) => Math.abs(y(d.value) - y(0))) // Final height of bars
        .delay((d, i) => i * 10); // Staggered animation for each bar

      // Tooltip (Optional)
      const tooltip = d3
        .select("body")
        .append("div")
        .style("position", "absolute")
        .style("background", "lightgray")
        .style("padding", "5px")
        .style("border-radius", "5px")
        .style("display", "none");

      svg
        .selectAll("rect")
        .on("mouseover", (event, d) => {
          tooltip
            .style("display", "block")
            .html(`Value: $${(d.value / 1_000_000).toFixed(1)}M`);
        })
        .on("mousemove", (event) => {
          tooltip
            .style("top", event.pageY - 10 + "px")
            .style("left", event.pageX + 10 + "px");
        })
        .on("mouseout", () => tooltip.style("display", "none"));
    },
  },
};
</script>

<style scoped>
.heading {
  font-size: 40px;
  font-weight: bold;
  color: #003366;
  margin-bottom: 20px;
}

.domain {
  stroke: none !important;
}
</style>
