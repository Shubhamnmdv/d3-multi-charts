<template>
    <div>
      <h1 class="heading">Multi Functional Bar Chart</h1>
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

      }
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
  