<template>
  <div>
    <h1 class="heading">Analyst Price Targets</h1>
    <div id="chart"></div>
  </div>
</template>

<script>
import * as d3 from "d3";
import monthlyMockData from "../mockData/analystPriceTargets.json"; // Adjust the file path accordingly
import dailyMockData from "../mockData/analystPriceTargets2.json"; // Adjust the file path accordingly

export default {
  name: "AnalystPriceTargets",
  mounted() {
    this.drawChart();
  },
  methods: {
    drawChart() {
      const margin = { top: 10, right: 30, bottom: 50, left: 30 };
      const width = 1300 - margin.left - margin.right;
      const height = 600 - margin.top - margin.bottom;

      // Append SVG element to the #chart div
      const svg = d3
        .select("#chart")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

      const monthlyData = monthlyMockData; // Import your monthly data
      const dailyData = dailyMockData; // Import your daily data

      // Merge datasets
      const data = dailyData.map((daily) => {
        const month = d3.timeFormat("%b-%y")(new Date(daily.date)); // Extract month-year format
        const monthlyEntry = monthlyData.find((m) => m.date === month) || {};
        return {
          date: new Date(daily.date),
          share_price: parseFloat(daily.share_price.replace("$", "").trim()), // Remove '$' if present
          average_1Y_Price: monthlyEntry.average_1Y_Price || null,
          high: monthlyEntry.high || null,
          low: monthlyEntry.low || null,
        };
      });

      // Parse the date and format data
      // const parseDate = d3.timeParse("%b-%y");
      // const data = monthlyMockData.map((d) => ({
      //   date: parseDate(d.date),
      //   share_price: +d.share_price,
      //   average_1Y_Price: +d.average_1Y_Price,
      //   high: +d.high,
      //   low: +d.low,
      // }));

      // Define scales
      const x = d3
        .scaleTime()
        .domain(d3.extent(data, (d) => d.date))
        .range([0, width]);
      const y = d3
        .scaleLinear()
        .domain([
          0,
          d3.max(data, (d) =>
            Math.max(d.high, d.share_price, d.average_1Y_Price)
          ),
        ])
        .range([height, 0]);

      // Add X-axis
      svg
        .append("g")
        .attr("transform", `translate(0,${height})`)
        .call(
          d3
            .axisBottom(x)
            .ticks(d3.timeMonth.every(1))
            .tickFormat(d3.timeFormat("%b %Y"))
        )
        .selectAll("text")
        .attr("transform", "translate(-10,10)rotate(-45)")
        .style("text-anchor", "end");

      // Add Y-axis
      svg.append("g").call(d3.axisLeft(y));

      // Create the area generator for the filled region
      const area = d3
        .area()
        .x((d) => x(d.date))
        .y0((d) => y(d.low)) // Bottom edge of the area (low)
        .y1((d) => y(d.high)) // Top edge of the area (high))
        .curve(d3.curveCardinal); // Smooth edges for the area

      // Add the filled area to the SVG
      svg
        .append("path")
        .datum(data)
        .attr("fill", "rgba(128, 0, 128, 0.2)") // Light purple fill color
        .attr("d", area);

      // Create the line generator for `share_price`
      const lineSharePrice = d3
        .line()
        .x((d) => x(d.date))
        .y((d) => y(d.share_price));

      // Create the line generator for `average_1Y_Price`
      const lineAveragePrice = d3
        .line()
        .x((d) => x(d.date))
        .y((d) => y(d.average_1Y_Price))
        .curve(d3.curveCardinal);

      // Add the `share_price` line to the SVG
      svg
        .append("path")
        .datum(data)
        .attr("fill", "none")
        .attr("stroke", "steelblue")
        .attr("stroke-width", 2)
        .attr("d", lineSharePrice);

      // Add the `average_1Y_Price` line to the SVG
      svg
        .append("path")
        .datum(data)
        .attr("fill", "none")
        .attr("stroke", "purple") // Purple color for the second line
        .attr("stroke-width", 2)
        .attr("d", lineAveragePrice);
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

#chart {
  margin-top: 20px;
}
</style>
