<template>
    <div>
      <h1 class="heading">Line Chart (With tooltip)</h1>
      <!-- Button Group for Time Ranges with Hardcoded Data -->
      <div class="button-group">
        <button 
          v-for="(range, index) in timeRanges" 
          :key="index" 
          :class="{'active': activeRange === index}"
          @click="changeTimeRange(index)">
          <span>{{ range.label }}</span>
          <span :style="{ color: range.change >= 0 ? 'green' : 'red' }">
            {{ range.change >= 0 ? '+' : '' }}{{ range.change }}%
          </span>
        </button>
      </div>
      <div ref="chart" class="line-chart"></div>
    </div>
  </template>

<script>
import * as d3 from "d3";
// import lineChartMock from "../mockData/lineChartMock.json"; // Mock data import

export default {
  name: "LineChartTimeline",
  data() {
    return {
      timeRanges: [
        { label: "1M", change: 0 },
        { label: "6M", change: 24 },
        { label: "1Y", change: 22 },
        { label: "3Y", change: 56 },
        { label: "5Y", change: 256 },
        { label: "10Y", change: 827 },
        { label: "All", change: null },
      ],
      activeRange: 6, // Default to 'All'
    };
  },
  mounted() {
    this.drawChart();
  },
  methods: {
    drawChart() {
      // Set up dimensions and margins
      const margin = { top: 10, right: 30, bottom: 30, left: 60 },
        width = 1200 - margin.left - margin.right,
        height = 400 - margin.top - margin.bottom;

      // Append the svg object to the chart div
      const svg = d3
        .select(this.$refs.chart)
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top})`);

         // Tooltip div setup
      const tooltipDiv = d3
        .select(this.$refs.chart)
        .append("div")
        .attr("class", "tooltip")
        .style("position", "absolute")
        .style("background", "#333")
        .style("color", "#fff")
        .style("padding", "5px 10px")
        .style("border-radius", "4px")
        .style("pointer-events", "none")
        .style("opacity", 0);

      // Load data and render chart
      d3.csv(
        "https://raw.githubusercontent.com/holtzy/data_to_viz/master/Example_dataset/3_TwoNumOrdered_comma.csv",
        function (d) {
          return { date: d3.timeParse("%Y-%m-%d")(d.date), value: +d.value };
        }
      ).then((data) => {
        // Define X axis
        const x = d3
          .scaleTime()
          .domain(d3.extent(data, (d) => d.date))
          .range([0, width]);
        const y = d3
          .scaleLinear()
          .domain([0, d3.max(data, (d) => d.value)])
          .range([height, 0]);

        const xAxis = svg
          .append("g")
          .attr("transform", `translate(0, ${height})`)
          .call(d3.axisBottom(x))
          .call((g) => g.selectAll(".tick line").attr("stroke", "white"))
          .call((g) => g.select(".domain").attr("stroke", "white"));

        const yAxis = svg
          .append("g")
          .call(d3.axisLeft(y))
          .call((g) => g.selectAll(".tick line").attr("stroke", "white"))
          .call((g) => g.select(".domain").attr("stroke", "white"));

        // Add horizontal grid lines
        svg
          .append("g")
          .attr("class", "grid")
          .selectAll("line")
          .data(y.ticks())
          .enter()
          .append("line")
          .attr("x1", 0)
          .attr("x2", width)
          .attr("y1", (d) => y(d))
          .attr("y2", (d) => y(d))
          .attr("stroke", "#ccc")
          .attr("stroke-dasharray", "4")
          .attr("stroke-width", 0.5);

        // Define the line and area generators
        const lineGenerator = d3
          .line()
          .x((d) => x(d.date))
          .y((d) => y(d.value));
        const areaGenerator = d3
          .area()
          .x((d) => x(d.date))
          .y0(height)
          .y1((d) => y(d.value));

        // Append the area and line paths
        svg
          .append("path")
          .datum(data)
          .attr("class", "area")
          .attr("fill", "lightblue")
          .attr("opacity", 0.5)
          .attr("d", areaGenerator);
        svg
          .append("path")
          .datum(data)
          .attr("class", "line")
          .attr("fill", "none")
          .attr("stroke", "steelblue")
          .attr("stroke-width", 1.5)
          .attr("d", lineGenerator);

        // Add brushing (zoom) functionality
        const brush = d3
          .brushX()
          .extent([
            [0, 0],
            [width, height],
          ])
          .on("end", updateChart);
        svg.append("g").attr("class", "brush").call(brush);

        // Add the hover line and dot with a glowing effect
        const hoverLine = svg
          .append("line")
          .attr("class", "hover-line")
          .attr("stroke", "#003366")
          .attr("stroke-width", 1)
          .attr("y1", 0)
          .attr("y2", height)
          .attr("stroke-dasharray", "4")
          .style("opacity", 0);

        const hoverDotGlow = svg
          .append("circle")
          .attr("class", "hover-dot-glow")
          .attr("r", 8) // Outer circle for glow effect
          .attr("fill", "white")
          .attr("opacity", 2) // Lower opacity for glow
          .attr("stroke-width", 2)
          .style("opacity", 0.2);

        const hoverDot = svg
          .append("circle")
          .attr("class", "hover-dot")
          .attr("r", 5) // Smaller inner circle
          .attr("fill", "steelblue") // Change the color for contrast
          .style("opacity", 0);

        // Add a transparent overlay for capturing mouse events for the hover effect
        svg
          .append("rect")
          .attr("width", width)
          .attr("height", height)
          .style("fill", "none")
          .style("pointer-events", "all")
          .on("mousemove", (event) => this.tooltip(event, data, x, y, tooltipDiv, hoverLine, hoverDot, hoverDotGlow))
          .on("mouseenter", () => {
            hoverLine.style("opacity", 1);
            hoverDot.style("opacity", 1);
            hoverDotGlow.style("opacity", 0.3);
          })
          .on("mouseleave", () => {
            hoverLine.style("opacity", 0);
            hoverDot.style("opacity", 0);
            hoverDotGlow.style("opacity", 0);
            tooltipDiv.style("opacity", 0);
          });

        // Define bisector for finding the closest data point
        const bisectDate = d3.bisector((d) => d.date).left;

        function mousemove(event) {
          const mouseX = d3.pointer(event, this)[0];
          const xDate = x.invert(mouseX);
          const i = bisectDate(data, xDate);
          const selectedData = data[i];

          if (selectedData) {
            const xPos = x(selectedData.date);
            const yPos = y(selectedData.value);

            hoverLine.attr("x1", xPos).attr("x2", xPos);
            hoverDot.attr("cx", xPos).attr("cy", yPos);
            hoverDotGlow.attr("cx", xPos).attr("cy", yPos);
          }
        }

        // Update chart based on brush selection (zoom)
        function updateChart(event) {
          const extent = event.selection;
          if (!extent) return; // If no selection, do nothing
          x.domain([x.invert(extent[0]), x.invert(extent[1])]); // Update x domain based on selection

          // Update the axes, line, and area to reflect the zoomed view
          xAxis.transition().duration(1000).call(d3.axisBottom(x));
          svg
            .select(".area")
            .transition()
            .duration(1000)
            .attr("d", areaGenerator);
          svg
            .select(".line")
            .transition()
            .duration(1000)
            .attr("d", lineGenerator);

          // Remove the brush selection area after zoom
          svg.select(".brush").call(brush.move, null);
        }

        // Double-click to reset the zoom
        svg.on("dblclick", function () {
          x.domain(d3.extent(data, (d) => d.date)); // Reset x domain
          xAxis.transition().call(d3.axisBottom(x)); // Update x axis

          // Reset the area and line
          svg.select(".area").transition().attr("d", areaGenerator);
          svg.select(".line").transition().attr("d", lineGenerator);
        });
      });
    },
    tooltip(event, data, x, y, tooltipDiv, hoverLine, hoverDot, hoverDotGlow) {
      const bisectDate = d3.bisector((d) => d.date).left;
      const mouseX = d3.pointer(event, this.$refs.chart)[0];
      const xDate = x.invert(mouseX);
      const i = bisectDate(data, xDate);
      const selectedData = data[i];

      if (selectedData) {
        const xPos = x(selectedData.date);
        const yPos = y(selectedData.value);

        hoverLine.attr("x1", xPos).attr("x2", xPos);
        hoverDot.attr("cx", xPos).attr("cy", yPos);
        hoverDotGlow.attr("cx", xPos).attr("cy", yPos);

        tooltipDiv
          .style("opacity", 1)
          .style("left", `${event.pageX + 60}px`)
          .style("top", `${event.pageY - 30}px`)
          .html(`Date: ${d3.timeFormat("%Y-%m-%d")(selectedData.date)}<br>Value: ${selectedData.value}`);
      }
    },
    changeTimeRange(index) {
      this.activeRange = index;
      // Logic to update the chart based on selected time range
    },
  },
};
</script>

<style scoped>
.button-group {
  display: flex;
  gap: 8px;
  margin-bottom: 16px;
}
.button-group button {
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  background: #f0f0f0;
  color: #333;
  font-size: 14px;
  font-weight: bold;
}
.button-group button.active {
  background: #333;
  color: #fff;
}
.line-chart {
  position: relative;
}
.heading {
  font-size: 40px; /* Adjust font size */
  font-weight: bold;
  color: #003366; /* Dark blue color similar to the reference image */
  margin-bottom: 20px; /* Add some space between the heading and the chart */
}

</style>
