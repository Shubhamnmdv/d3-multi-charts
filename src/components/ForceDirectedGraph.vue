<template>
    <div>
      <div id="tooltip" class="tooltip" style="opacity: 0;"></div>
      <svg ref="chart"></svg>
    </div>
  </template>
  
  <script>
  import * as d3 from "d3";
  import forceDirectedGraphMock from "../mockData/forceDirectedGraphMock.json";
  
  export default {
    name: "ForceDirectedGraphMock",
    mounted() {
      this.drawChart();
    },
    methods: {
      drawChart() {
        const data = forceDirectedGraphMock;
        const width = 928;
        const height = 680;
  
        const color = d3.scaleOrdinal(d3.schemeCategory10);
        const nodeIds = new Set(data.nodes.map(d => d.id));
        const links = data.links
        .filter(d => nodeIds.has(d.source) && nodeIds.has(d.target))
        .map(d => ({ ...d }));
        const nodes = data.nodes.map(d => ({ ...d }));
  
          const svg = d3.select(this.$refs.chart)
            .attr("width", "100%")
            .attr("height", height)
            .attr("viewBox", [0, 0, width, height])
            .call(
              d3.zoom()
                .scaleExtent([0.1, 8])
                .on("zoom", (event) => {
                  g.attr("transform", event.transform);
                })
            );

        const g = svg.append("g"); // Container for nodes + links

        const simulation = d3.forceSimulation(nodes)
          .force("link", d3.forceLink(links).id(d => d.id).distance(120))
          .force("charge", d3.forceManyBody().strength(-200))
          .force("center", d3.forceCenter(width / 2, height / 2));

        const link = g.append("g")
          .attr("stroke", "#999")
          .attr("stroke-opacity", 0.6)
          .selectAll("line")
          .data(links)
          .join("line")
          .attr("stroke-width", 1.5);

        const node = g.append("g")
          .attr("stroke", "#fff")
          .attr("stroke-width", 1.5)
          .selectAll("circle")
          .data(nodes)
          .join("circle")
          .attr("r", 12)
          .attr("fill", d => color(d.group?.rdf_Property || "default"))
          .call(
            d3.drag()
              .on("start", dragstarted)
              .on("drag", dragged)
              .on("end", dragended)
          );

  
        node.append("title")
          .text(d => d.group?.identifier || d.id.split("#").pop());
  
        simulation.on("tick", () => {
          link
            .attr("x1", d => d.source.x)
            .attr("y1", d => d.source.y)
            .attr("x2", d => d.target.x)
            .attr("y2", d => d.target.y);
  
          node
            .attr("cx", d => d.x)
            .attr("cy", d => d.y);
        });
  
        function dragstarted(event) {
          if (!event.active) simulation.alphaTarget(0.3).restart();
          event.subject.fx = event.subject.x;
          event.subject.fy = event.subject.y;
        }
  
        function dragged(event) {
          event.subject.fx = event.x;
          event.subject.fy = event.y;
        }
  
        function dragended(event) {
          if (!event.active) simulation.alphaTarget(0);
          event.subject.fx = null;
          event.subject.fy = null;
        }

        const tooltip = d3.select("#tooltip");

        node.on("mouseover", (event, d) => {
            tooltip
              .style("opacity", 1)
              .html(`
                <strong>${d.group?.identifier || d.id.split("#").pop()}</strong><br/>
                <em>${d.group?.rdfs_comment || d.group?.rdfs_isDefinedBy || "No description."}</em>
              `);
          })
          .on("mousemove", (event) => {
            tooltip
              .style("left", (event.pageX + 10) + "px")
              .style("top", (event.pageY + 10) + "px");
          })
          .on("mouseout", () => {
            tooltip.style("opacity", 0);
          });

      }
    }
  };
  </script>
  
  <style scoped>
  svg {
    background: #fafafa;
  }

  .tooltip {
    position: absolute;
    background-color: white;
    padding: 6px 10px;
    border: 1px solid #999;
    border-radius: 4px;
    pointer-events: none;
    font-size: 12px;
    color: #333;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
}

  </style>
  