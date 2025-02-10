<template>
  <div>
    <h1 class="heading">Line Chart With Torch</h1>
    <div ref="chart" class="line-chart"></div>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "LineChartTimeline",
  data() {
    return {};
  },
  mounted() {
    this.drawChart();
  },
  methods: {
    drawChart() {
      const csvUrl =
        "https://raw.githubusercontent.com/holtzy/data_to_viz/master/Example_dataset/3_TwoNumOrdered_comma.csv";

      const margin = { top: 50, right: 100, bottom: 50, left: 60 };
      const width = 800 - margin.left - margin.right;
      const height = 400 - margin.top - margin.bottom;

      // Create SVG container
      const svg = d3
        .select(this.$refs.chart)
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top})`);

      // Fetch CSV data
      d3.csv(csvUrl, d3.autoType).then((data) => {
        data.forEach((d) => {
          d.date = d3.timeParse("%Y-%m-%d")(d.date);
        });

        const xScale = d3.scaleTime().range([0, width]);
        const yScale = d3.scaleLinear().range([height, 0]);

        xScale.domain(d3.extent(data, (d) => d.date));
        yScale.domain([0, d3.max(data, (d) => d.value) * 1.1]);

        // Line generator
        const line = d3
          .line()
          .x((d) => xScale(d.date))
          .y((d) => yScale(d.value));

        const area = d3
          .area()
          .x((d) => xScale(d.date))
          .y0(height)
          .y1((d) => yScale(d.value));

        // Add area
        svg
          .append("path")
          .datum(data)
          .attr("fill", "lightblue")
          .attr("opacity", 0.5)
          .attr("d", area);

        // Add line
        svg
          .append("path")
          .datum(data)
          .attr("fill", "none")
          .attr("stroke", "steelblue")
          .attr("stroke-width", 2)
          .attr("d", line);

        // Add x-axis
        svg
          .append("g")
          .attr("transform", `translate(0, ${height})`)
          .call(d3.axisBottom(xScale).ticks(10));

        // Add y-axis
        svg.append("g").call(d3.axisLeft(yScale).ticks(5));

        // Forecast points
        const forecastData = [
          { date: new Date("2018-05-01"), value: 4000.66, label: "Current" },
          { date: new Date("2018-06-01"), value: 296.13, label: "Avg" },
          { date: new Date("2018-07-01"), value: 528, label: "Max" },
          { date: new Date("2018-07-01"), value: 120, label: "Min" },
        ];

        forecastData.forEach((forecast) => {
          svg
            .append("line")
            .attr("x1", xScale(data[data.length - 1].date))
            .attr("x2", xScale(forecast.date))
            .attr("y1", yScale(data[data.length - 1].value))
            .attr("y2", yScale(forecast.value))
            .attr("stroke", "red")
            .attr("stroke-dasharray", "4,4")
            .attr("stroke-width", 1.5);

          svg
            .append("circle")
            .attr("cx", xScale(forecast.date))
            .attr("cy", yScale(forecast.value))
            .attr("r", 4)
            .attr("fill", "red");

          svg
            .append("text")
            .attr("x", xScale(forecast.date) + 5)
            .attr("y", yScale(forecast.value) - 5)
            .text(`${forecast.label}: ${forecast.value}`)
            .style("font-size", "12px")
            .style("fill", "red");
        });
      });
    },
  },
};
</script>

<style scoped>
.line-chart {
  position: relative;
  width: 100%;
  height: 400px;
}
.heading {
  font-size: 40px;
  font-weight: bold;
  color: #003366;
  margin-bottom: 20px;
}
</style>
