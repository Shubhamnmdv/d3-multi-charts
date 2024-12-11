<template>
    <div>
      <div class="container">
        <div class="chart-container">
          <!-- Pass selectedBars as a prop -->
          <BarChartMultiFunctionality :selectedBars="selectedBars" />
        </div>
        <div class="filter-container">
          <!-- Filters Section -->
          <h2>Filters</h2>
          <div class="filter-options">
            <h3>Toggle Bars</h3>
            <div v-for="(key, index) in barKeys" :key="index" class="checkbox-group">
              <input
                type="checkbox"
                :id="key"
                :value="key"
                v-model="selectedBars"
              />
              <label :for="key">{{ key }}</label>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import * as d3 from "d3";
  import multiFunctionalityMock from "../mockData/multiFunctionalityMock.json";
  import BarChartMultiFunctionality from "./BarChartMultiFunctionality.vue";
  
  export default {
    name: "MultiFunctionalityChart",
    components: {
      BarChartMultiFunctionality,
    },
    data() {
      return {
        barKeys: [], // To hold bar properties dynamically
        selectedBars: ["barFirst", "barSecond", "barThird"], // Default selected bars
      };
    },
    mounted() {
      this.extractBarKeys();
    },
    methods: {
      extractBarKeys() {
        const keys = Object.keys(multiFunctionalityMock[0]);
        this.barKeys = keys.filter((key) => key !== "xAxisLabel");
      },
    },
  };
  </script>
  
  
  <style scoped>
/* Reset body margin and padding */
body,
html {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

.container {
  display: flex;
  flex-direction: row;
  width: 100%; /* Ensures the container spans the full width */
  box-sizing: border-box;
  padding: 10px;
}

.chart-container { 
  flex-basis: 80%;
  /* flex: 7; */
  padding: 10px;
  background-color: #f8faff; /* Optional for visualization */
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); /* Subtle shadow */
  overflow: auto;
  
}

.filter-container {
flex-basis: 20%; 
  padding: 20px;
  background-color: #f8faff; /* Soft blue background color */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Subtle shadow for better separation */
  border-radius: 8px; /* Rounded corners */
}

h2 {
  font-size: 16px;
  font-weight: bold;
  color: #4a4a4a; /* Grayish text color */
  text-transform: uppercase;
  margin-bottom: 20px;
  border-bottom: 1px solid #ddd; /* Add a subtle border under the heading */
  padding-bottom: 10px;
}

.filter-options {
  display: flex;
  flex-direction: column;
  gap: 15px; /* Add spacing between filter options */
}

h3 {
  font-size: 14px;
  color: #4a4a4a; /* Grayish text color */
  margin-bottom: 10px;
}

.checkbox-group {
  display: flex;
  align-items: center;
  gap: 10px; /* Space between checkbox and label */
}

.checkbox-group input[type="checkbox"] {
  width: 16px;
  height: 16px;
  border: 2px solid #ff6f61; /* Red border for the checkbox */
  border-radius: 4px; /* Rounded corners for the checkbox */
  outline: none;
  appearance: none; /* Removes default styling */
  cursor: pointer;
  position: relative;
  background-color: #fff; /* White background */
  transition: background-color 0.2s, border-color 0.2s;
}

.checkbox-group input[type="checkbox"]:checked {
  background-color: #ff6f61; /* Red fill for selected state */
  border-color: #ff6f61;
}

.checkbox-group input[type="checkbox"]:checked::after {
  content: "";
  position: absolute;
  width: 8px;
  height: 8px;
  background-color: white;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  border-radius: 2px; /* Small rounded inner checkmark */
}

.checkbox-group label {
  font-size: 14px;
  color: #4a4a4a; /* Grayish text color */
  cursor: pointer; /* Cursor changes to pointer for better usability */
}

.domain {
  stroke: none !important;
}
</style>
