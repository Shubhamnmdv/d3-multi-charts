<template>
    <div>
      <svg ref="chart"></svg>
    </div>
  </template>
  
  <script>
  import * as d3 from "d3";
  import footballFieldMock from "../mockData/footballFieldMock.json";
  
  export default {
    name: "FootballFieldChart",
    mounted() {
      this.drawChart();
    },
    methods: {
      drawChart() {
        const data = footballFieldMock;
  
        const margin = { top: 20, right: 30, bottom: 40, left: 250 };
        const width = 1200 - margin.left - margin.right;
        const height = 400 - margin.top - margin.bottom;
  
        const svg = d3
          .select(this.$refs.chart)
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
          .append("g")
          .attr("transform", `translate(${margin.left},${margin.top})`);
  
        // Define scales
        const x = d3
          .scaleLinear()
          .domain([
            d3.min(data, (d) => d.min) - 2, // Add some padding
            d3.max(data, (d) => d.max) + 2,
          ])
          .range([0, width]);
  
        const y = d3
          .scaleBand()
          .domain(data.map((d) => d.label))
          .range([0, height])
          .padding(0.2);
  
        // Add x-axis with "$" symbol
        svg
          .append("g")
          .attr("transform", `translate(0,${height})`)
          .call(
            d3
              .axisBottom(x)
              .ticks(10)
              .tickFormat((d) => `$${d.toFixed(2)}`)
          );
  
        // Add y-axis
        svg
          .append("g")
          .call(d3.axisLeft(y))
          .selectAll("text")
          .style("font-size", "12px");
  
        // Add horizontal grid lines for each y-axis label
        svg
          .selectAll(".grid-line")
          .data(data)
          .enter()
          .append("line")
          .attr("x1", 0)
          .attr("x2", width)
          .attr("y1", (d) => y(d.label) + y.bandwidth() / 2)
          .attr("y2", (d) => y(d.label) + y.bandwidth() / 2)
          .attr("stroke", "grey")
          .attr("stroke-width", 0.5)
          .attr("stroke-dasharray", "3,3");
  
        // Draw bars
        svg
          .selectAll(".bar")
          .data(data)
          .enter()
          .append("rect")
          .attr("class", "bar")
          .attr("x", (d) => x(d.min))
          .attr("y", (d) => y(d.label))
          .attr("width", (d) => x(d.max) - x(d.min))
          .attr("height", y.bandwidth())
          .style("fill", "#4682B4");
  
        // Add text for min and max values with "$" symbol
        svg
          .selectAll(".min-text")
          .data(data)
          .enter()
          .append("text")
          .attr("x", (d) => x(d.min) - 5)
          .attr("y", (d) => y(d.label) + y.bandwidth() / 2)
          .attr("dy", ".35em")
          .attr("text-anchor", "end")
          .style("font-size", "10px")
          .text((d) => `$${d.min.toFixed(2)}`);
  
        svg
          .selectAll(".max-text")
          .data(data)
          .enter()
          .append("text")
          .attr("x", (d) => x(d.max) + 5)
          .attr("y", (d) => y(d.label) + y.bandwidth() / 2)
          .attr("dy", ".35em")
          .attr("text-anchor", "start")
          .style("font-size", "10px")
          .text((d) => `$${d.max.toFixed(2)}`);
  
        // Calculate the average of all (min + max) / 2 values
        const averageValue = d3.mean(data, (d) => (d.min + d.max) / 2);
  
        // Add a vertical dashed line for the average value
        svg
          .append("line")
          .attr("x1", x(averageValue))
          .attr("x2", x(averageValue))
          .attr("y1", 0)
          .attr("y2", height)
          .attr("stroke", "black")
          .attr("stroke-width", 1)
          .attr("stroke-dasharray", "4,4");
  
        // Add label for the average value with "$" symbol
        svg
          .append("text")
          .attr("x", x(averageValue) + 5)
          .attr("y", -5)
          .attr("text-anchor", "start")
          .style("font-size", "12px")
          .style("font-weight", "bold")
          .text(`Average: $${averageValue.toFixed(2)}`);
      },
    },
  };
  </script>
  
  <style scoped>
  .bar {
    fill: steelblue;
  }
  </style>
  