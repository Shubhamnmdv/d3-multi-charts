<template>
  <div>
    <h1 class="heading">Line Chart (With Zoomable)</h1>
    <!-- Button Group for Time Ranges with Hardcoded Data -->
    <div class="button-group">
      <button
        v-for="(range, index) in timeRanges"
        :key="index"
        :class="{ active: activeRange === index }"
        @click="changeTimeRange(index)"
      >
        <span>{{ range.label }}</span>
        <span :style="{ color: range.change >= 0 ? 'green' : 'red' }">
          {{ range.change >= 0 ? "+" : "" }}{{ range.change }}%
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

        const xAxis = svg
          .append("g")
          .attr("transform", `translate(0, ${height})`)
          .call(d3.axisBottom(x))
          .call((g) => g.selectAll(".tick line").attr("stroke", "white"))
          .call((g) => g.select(".domain").attr("stroke", "white"));

        // Define Y axis
        const y = d3
          .scaleLinear()
          .domain([0, d3.max(data, (d) => d.value)])
          .range([height, 0]);

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

        // Define clipPath to limit drawing area
        const clip = svg
          .append("defs")
          .append("svg:clipPath")
          .attr("id", "clip")
          .append("svg:rect")
          .attr("width", width)
          .attr("height", height)
          .attr("x", 0)
          .attr("y", 0);

        // Create brush and line container
        const lineGroup = svg.append("g").attr("clip-path", "url(#clip)");

        // Define area below the line
        const area = d3
          .area()
          .x((d) => x(d.date))
          .y0(height)
          .y1((d) => y(d.value));

        // Append the area to the SVG with a unique class
        lineGroup
          .append("path")
          .datum(data)
          .attr("class", "area") // Assign a unique class to the area
          .attr("fill", "lightblue")
          .attr("opacity", 0.5)
          .attr("d", area);

        // Add the line on top of the area
        lineGroup
          .append("path")
          .datum(data)
          .attr("class", "line") // Assign a unique class to the line
          .attr("fill", "none")
          .attr("stroke", "steelblue")
          .attr("stroke-width", 1.5)
          .attr(
            "d",
            d3
              .line()
              .x((d) => x(d.date))
              .y((d) => y(d.value))
          );

        // Add brushing
        const brush = d3
          .brushX()
          .extent([
            [0, 0],
            [width, height],
          ])
          .on("end", updateChart);

        lineGroup.append("g").attr("class", "brush").call(brush);

        let idleTimeout;
        function idled() {
          idleTimeout = null;
        }

        // Update chart based on brush selection
        function updateChart(event) {
          const extent = event.selection;
          if (!extent) {
            if (!idleTimeout) return (idleTimeout = setTimeout(idled, 350));
            x.domain(d3.extent(data, (d) => d.date));
          } else {
            x.domain([x.invert(extent[0]), x.invert(extent[1])]);
            lineGroup.select(".brush").call(brush.move, null);
          }
          xAxis.transition().duration(1000).call(d3.axisBottom(x));

          // Update both area and line paths based on the new x-axis domain
          lineGroup.select(".area").transition().duration(1000).attr("d", area); // Update area path

          lineGroup
            .select(".line")
            .transition()
            .duration(1000)
            .attr(
              "d",
              d3
                .line()
                .x((d) => x(d.date))
                .y((d) => y(d.value))
            ); // Update line path
        }

        // Reset on double-click
        svg.on("dblclick", function () {
          x.domain(d3.extent(data, (d) => d.date));
          xAxis.transition().call(d3.axisBottom(x));

          lineGroup.select(".area").transition().attr("d", area); // Reset area path

          lineGroup
            .select(".line")
            .transition()
            .attr(
              "d",
              d3
                .line()
                .x((d) => x(d.date))
                .y((d) => y(d.value))
            ); // Reset line path
        });
      });
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
