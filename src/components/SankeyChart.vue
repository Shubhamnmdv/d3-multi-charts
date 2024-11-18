<template>
  <div ref="chart" class="sankey-chart"></div>
</template>

<script>
import * as d3 from "d3";
import { sankey, sankeyLinkHorizontal } from "d3-sankey"; // Sankey functions from d3-sankey
import sankeyChartMock from "../mockData/sankeyChartMock.json"; // Mock data import

export default {
  name: "SankeyChart",
  mounted() {
    this.drawChart();   
  },
  methods: {
    drawChart() {
      // Define margins and larger dimensions for a spacious layout
      const margin = { top:60, right: 80, bottom: 20, left: 20 },
        width = 1100 - margin.left - margin.right, 
        height = 1000 - margin.top - margin.bottom; 

      // Create the SVG element
      const svg = d3
        .select(this.$refs.chart)
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

      // Define a color scale or custom color mapping
      const color = d3.scaleOrdinal(d3.schemeCategory10);
      const colors = {
        "Data Center": "#FF6600",
        Gaming: "#00CC66",
        "Professional Visualization": "#CC0066",
        Automotive: "#3399FF",
        "OEM & Other": "#9933FF",
        "Gross profit": "#00FF66",
        "Operating profit": "#FF3366",
        "Net profit": "#6666FF",
        "Cost of revenue": "#FF66FF",
        "Operating expenses": "#666666",
        "Research & Development": "#99CC33",
        "Sales, General & Admin": "#66CCFF",
        Tax: "#003399",
        Other: "#FF9966",
        Revenue: "#000000",
      };

      // Create a map from node ID to index
      const nodeMap = {};
      sankeyChartMock.nodes.forEach((node, i) => {
        nodeMap[node.id] = i;
      });

      // Transform the source and target in the links to indices
      const linksWithIndices = sankeyChartMock.links.map((link) => ({
        source: nodeMap[link.source],
        target: nodeMap[link.target],
        value: link.value,
      }));

      // Create the Sankey generator
      const sankeyGenerator = sankey()
        .nodeWidth(36)
        .nodePadding(60) // Increased padding between nodes
        .extent([
          [0, 0],
          [width, height],
        ]);

      // Bind the nodes and transformed links (with indices) to the Sankey generator
      const { nodes, links } = sankeyGenerator({
        nodes: sankeyChartMock.nodes.map((d) => Object.assign({}, d)),
        links: linksWithIndices, // Use transformed links with indices
      });

      nodes.forEach( node => {
          if(node.id == "Data Center") {
            node.x0 = node.x0 - 1100;
            node.x1 = node.x1 - 1100;
          }

      });

      // Add the links to the chart
      const link = svg
        .append("g")
        .selectAll("path")
        .data(links)
        .join("path")
        .attr("class", "link")
        .attr("d", sankeyLinkHorizontal())
        .attr("stroke", (d) => colors[d.source.name] || "#000")
        .attr("stroke-opacity", 0.4)
        .attr("stroke-width", (d) => Math.max(2, d.width))
        .attr("fill", "none")
        .on("mouseover", function () {
          d3.select(this).attr("stroke-opacity", 0.6);
        })
        .on("mouseout", function () {
          d3.select(this).attr("stroke-opacity", 0.4);
        });

      // Add the nodes to the chart
      const node = svg
        .append("g")
        .selectAll("rect")
        .data(nodes)
        .join("rect")
        .attr("x", (d) => d.x0)
        .attr("y", (d) => d.y0)
        .attr("height", (d) => d.y1 - d.y0)
        .attr("width", sankeyGenerator.nodeWidth())
        .attr("fill", (d) => colors[d.id] || color(d.id)) // Apply colors to nodes
        .attr("stroke", (d) => d3.rgb(color(d.id)).darker(2))
        .append("title")
        .text((d) => `${d.id}\n${d.value}`);

      // Add the node labels (text), value, and subValue above the rectangles
      svg
        .append("g")
        .selectAll("g") 
        .data(nodes)
        .join("g")
        .attr(
          "transform",
          (d) => `translate(${(d.x0 + d.x1) / 2},${d.y0 - 20})`
        ) // Align horizontally to the center of the rect and move above
        .each(function (d) {
          const group = d3.select(this);

          // Main label (id)
          group
            .append("text")
            .attr("dy", "-1.8em")
            .attr("text-anchor", "middle") 
            .attr("font-size", "14px")
            .attr("font-weight", "bold")
            .attr("fill", d.color)
            .text(d.id);

          group
            .append("text")
            .attr("dy", "-0.4em") 
            .attr("text-anchor", "middle") 
            .attr("font-size", "13px")
            .attr("fill", "#888")
            .text((d) => d.value);

          group
            .append("text")
            .attr("dy", "1em") 
            .attr("text-anchor", "middle")
            .attr("font-size", "12px")
            .attr("fill", "#666")
            .text((d) => d.subValue);
        });
    },
  },
};
</script>

<style scoped>
.sankey-chart {
  width: 960px;
  height: 500px;
  margin: 0 auto;
}

.link {
  fill: none;
  stroke: #000;
  stroke-opacity: 0.2;
}

.link:hover {
  stroke-opacity: 0.5;
}
</style>
