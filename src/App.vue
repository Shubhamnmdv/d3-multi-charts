<script setup>
import sankeychart from "./components/SankeyChart.vue";
import lineChartTimeline from "./components/LineChartTimeline.vue";
import candleStickChart from "./components/CandleStickChart.vue";
import MultiStackLineChart from "./components/MultiStackLineChart.vue";
import backupChart from "./components/BackupChart.vue";
import AnalystPriceTargets from "./components/AnalystPriceTargets.vue";
import MiniBarChart from "./components/MiniBarChart.vue";
import SparkLineChart from "./components/SparkLineChart.vue";
import MultiFunctionalityChart from "./components/MultiFunctionalityChart.vue";
import FootballFieldChart from "./components/FootballFieldChart.vue";
import LineChartWithTorch from "./components/LineChartWithTorch.vue";
import InsiderTrandingVolume from "./components/InsiderTrandingVolume.vue";
import ForceDirectedGraph from "./components/ForceDirectedGraph.vue";
import DragAndDropUpload from "./components/DragAndDropUpload.vue";
import LineChartHightChart from "./components/Highcharts/LineChartHightChart.vue";
import InvertedTreegraphHighchart from "./components/Highcharts/InvertedTreegraphHighchart.vue";


</script>

<template>
  <!-- <div class="wrapper"> -->
    <!-- <h1 class="heading">All Charts</h1> -->
    <!-- <div class="card card-sankey">
      <LineChartHightChart></LineChartHightChart>
        <DragAndDropUpload></DragAndDropUpload>
    </div> -->
    <div class="card card-sankey">
        <!-- <ForceDirectedGraph></ForceDirectedGraph> -->
        <InvertedTreegraphHighchart></InvertedTreegraphHighchart>
    </div>
    <!-- <div class="card card-sankey">
        <sankeychart></sankeychart>
      </div>
    <div class="card card-line-timeline">
      <lineChartTimeline></lineChartTimeline>
    </div>
    <div class="card card-line-chart-backup">
      <backupChart></backupChart>
    </div>
    <div class="card card-candlestick">
      <candleStickChart></candleStickChart>
    </div>
    <div class="card card-candlestick">
      <MultiStackLineChart></MultiStackLineChart>
    </div>
    <div class="card card-analyst-price-targets">
        <AnalystPriceTargets></AnalystPriceTargets>
    </div>
    <div class="card card-analyst-price-targets">
        <MiniBarChart></MiniBarChart>
    </div>
     <SparkLineChart></SparkLineChart>
     <MultiFunctionalityChart></MultiFunctionalityChart>
    <div class="card card-candlestick">
        <FootballFieldChart></FootballFieldChart>
    </div>
    <div class="card card-line-chart-backup">
      <LineChartWithTorch></LineChartWithTorch>
    </div>
    <div class="card card-line-chart-backup">
      <InsiderTrandingVolume></InsiderTrandingVolume>
    </div> -->
  <!-- </div> -->
</template>

<style scoped>
.wrapper {
  display: flex;
  flex-direction: column;
  gap: 20px; /* Space between the cards */
}

.card {
  width: auto; /* Full width of the container */
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); /* Subtle shadow */
  overflow: auto;
}

.heading {
  font-size: 40px; /* Adjust font size */
  font-weight: bold;
  color: #422a43; /* Dark blue color similar to the reference image */
  margin-bottom: 10px; /* Add some space between the heading and the chart */
}

/* Custom styles for individual cards */
.card-sankey {
  background-color: #ffe6e6; /* Light red */
}

.card-line-timeline {
  background-color: #e6ffe6; /* Light green */
}

.card-line-chart-backup {
  background-color: #e6f7ff; /* Light blue */
}

.card-candlestick {
  background-color: #fff3e6; /* Light orange */
}
</style>