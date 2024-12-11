<template>
    <div>
      <h1 class="heading">Spark Line Chart Table</h1>
      <div id="chart">
        <table>
          <thead>
            <tr>
              <th>Name</th>
              <th>Data</th>
              <th>Sparkline</th>
            </tr>
          </thead>
          <tbody id="table-body">
            <!-- Rows will be dynamically rendered -->
          </tbody>
        </table>
      </div>
    </div>
  </template>
  
  <script>
  import * as d3 from "d3";
  
  export default {
    name: "SparkLineChart",
    mounted() {
      this.renderTable();
    },
    methods: {
      renderTable() {
        // Sample data for the table
        const tableData = [
          { name: "Row 1", data: [9, 8, 7, 6, 5, 4, 3] },
          { name: "Row 2", data: [3, 8, -1, -5, 2] },
          { name: "Row 3", data: [-6, 4, 3, 5, -3] },
        ];
  
        const tbody = d3.select("#table-body");
  
        // Add rows to the table
        tableData.forEach((row) => {
          const tr = tbody.append("tr");
  
          // Add Name cell
          tr.append("td").text(row.name).style("text-align", "center");
  
          // Add Data cell
          tr.append("td").text(row.data.join(", ")).style("text-align", "center");
  
          // Add Sparkline cell
          const sparklineCell = tr
            .append("td")
            .attr("class", "sparkline-cell")
            .style("text-align", "center");
  
          // Set dimensions for the sparkline
          const width = 100;
          const height = 20;
          const barWidth = width / row.data.length;
  
          // Define scales
          const xScale = d3.scaleLinear().domain([0, row.data.length]).range([0, width]);
          const yScale = d3.scaleLinear()
            .domain([d3.min(row.data), d3.max(row.data)])
            .range([height, 0]);
  
          // Create SVG for sparkline
          const svg = sparklineCell
            .append("svg")
            .attr("width", width)
            .attr("height", height);
  
          // Add bars to the SVG with animations
          svg.selectAll("rect")
            .data(row.data)
            .enter()
            .append("rect")
            .attr("x", (_, i) => xScale(i))
            .attr("y", yScale(0)) // Start bars at the baseline
            .attr("width", barWidth - 2) // Add space between bars
            .attr("height", 0) // Start with zero height
            .attr("fill", (d) => (d < 0 ? "red" : "steelblue"))
            .transition() // Apply animation
            .duration(1000) // Duration of 1 second
            .attr("y", (d) => (d > 0 ? yScale(d) : yScale(0))) // Animate to the final position
            .attr("height", (d) => Math.abs(yScale(d) - yScale(0))); // Animate to the final height
        });
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
  
  table {
    border-collapse: collapse;
    width: 100%;
  }
  
  th, td {
    border: 1px solid #d3d3d3;
    padding: 8px;
    text-align: center; /* Center align headers and data */
    vertical-align: middle; /* Ensure content stays vertically centered */
  }
  
  th, ::v-deep(td) {
    border: 1px solid #d3d3d3;
    padding: 8px;
    text-align: center; /* Center align headers and data */
    vertical-align: middle; /* Ensure content stays vertically centered */
  }
  
  .sparkline-cell svg {
    display: block;
    margin: auto;
  }
  </style>
  