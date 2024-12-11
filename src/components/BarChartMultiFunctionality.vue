<template>
    <div>
      <h1 class="heading">Multi Functional Bar Chart</h1>
      <div id="chart"></div>
    </div>
  </template>
  
  <script>
  import * as d3 from "d3";
  import multiFunctionalityMock from "../mockData/multiFunctionalityMock.json";
  
  export default {
    name: "BarChartMultiFunctionality",
    props: {
      selectedBars: {
        type: Array,
        required: true,
      },
    },
    watch: {
      selectedBars: {
        handler() {
          this.drawChart();
        },
        deep: true,
      },
    },
    mounted() {
      this.drawChart();
    },
    methods: {
      drawChart() {
        const margin = { top: 10, right: 30, bottom: 20, left: 50 };
        const width = 1400 - margin.left - margin.right;
        const height = 600 - margin.top - margin.bottom;
  
        // Remove any existing chart
        d3.select("#chart").select("svg").remove();
  
        // Append the SVG object
        const svg = d3
          .select("#chart")
          .append("svg")
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
          .append("g")
          .attr("transform", `translate(${margin.left},${margin.top})`);
  
        const data = multiFunctionalityMock;
  
        // Filter the subgroups based on selectedBars
        const subgroups = this.selectedBars;
  
        // X-axis scale
        const groups = data.map((d) => d.xAxisLabel);
        const x = d3
          .scaleBand()
          .domain(groups)
          .range([0, width])
          .padding(0.2);
  
        svg
          .append("g")
          .attr("transform", `translate(0,${height})`)
          .call(d3.axisBottom(x).tickSize(0))
          .selectAll("text")
          .style("text-anchor", "middle");
  
        // Y-axis scale
        const y = d3.scaleLinear().domain([0, 100]).range([height, 0]);
        svg.append("g").call(d3.axisLeft(y));
  
        // Subgroup scale
        const xSubgroup = d3
          .scaleBand()
          .domain(subgroups)
          .range([0, x.bandwidth()])
          .padding(0.08);
  
        // Color scale
        const color = d3
          .scaleOrdinal()
          .domain(subgroups)
          .range(["#4c58d9", "#48d2a0", "#a889ff",  "#f6b100"]);
  
        // Show bars
        svg
          .append("g")
          .selectAll("g")
          .data(data)
          .enter()
          .append("g")
          .attr("transform", (d) => `translate(${x(d.xAxisLabel)},0)`)
          .selectAll("rect")
          .data((d) =>
            subgroups.map((key) => ({ key: key, value: d[key] }))
          )
          .enter()
          .append("rect")
          .attr("x", (d) => xSubgroup(d.key))
          .attr("y", (d) => y(d.value))
          .attr("width", xSubgroup.bandwidth())
          .attr("height", (d) => height - y(d.value))
          .attr("fill", (d) => color(d.key))
        //   .attr("rx", 10) // Rounded corners
        //   .attr("ry", 10); // Rounded corners
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
    text-align: center;
  }
  
  #chart {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0 auto;
  }
  
  .domain {
    stroke: none !important;
  }
  </style>
  