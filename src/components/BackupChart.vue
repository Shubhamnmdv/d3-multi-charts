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
      const rawData = forceDirectedGraphMock;
      const width = 928;
      const height = 680;

      const color = d3.scaleOrdinal(d3.schemeCategory10);
      const nodeMap = new Map(rawData.nodes.map(n => [n.id, { ...n, expanded: false }]));
      const childMap = new Map();

      // Build parent -> children map
      rawData.links.forEach(link => {
        if (!childMap.has(link.source)) childMap.set(link.source, []);
        childMap.get(link.source).push(link.target);
      });

      const visibleNodes = new Map();
      const visibleLinks = [];

      // Show only parent root nodes initially
      const rootNodes = rawData.nodes.filter(
        n => !n.group?.rdfs_subClassOf
      );

      rootNodes.forEach(n => visibleNodes.set(n.id, nodeMap.get(n.id)));

      function toggleExpandCollapse(id) {
        const node = nodeMap.get(id);
        if (!node) return;

        if (!node.expanded) {
          node.expanded = true;
          const children = childMap.get(id) || [];
          for (const childId of children) {
            visibleNodes.set(childId, nodeMap.get(childId));
            visibleLinks.push({ source: id, target: childId });
          }
        } else {
          collapseNode(id);
        }
      }

      function collapseNode(id) {
        const node = nodeMap.get(id);
        if (!node) return;

        const children = childMap.get(id) || [];
        node.expanded = false;
        for (const childId of children) {
          collapseNode(childId);
          visibleNodes.delete(childId);
        }

        for (let i = visibleLinks.length - 1; i >= 0; i--) {
          if (visibleLinks[i].source === id) {
            visibleLinks.splice(i, 1);
          }
        }
      }

      const svg = d3.select(this.$refs.chart)
        .attr("width", "100%")
        .attr("height", height)
        .attr("viewBox", [0, 0, width, height])
        .call(
          d3.zoom().scaleExtent([0.1, 8])
            .on("zoom", (event) => g.attr("transform", event.transform))
        );

      const g = svg.append("g");

      let nodes = Array.from(visibleNodes.values());
      let links = visibleLinks;

      const simulation = d3.forceSimulation(nodes)
        .force("link", d3.forceLink(links).id(d => d.id).distance(80))
        .force("charge", d3.forceManyBody().strength(-100))
        .force("center", d3.forceCenter(width / 2, height / 2));

      const link = g.append("g")
        .attr("stroke", "#aaa")
        .attr("stroke-opacity", 0.6)
        .selectAll("line");

      let node = g.append("g")
        .attr("stroke", "#fff")
        .attr("stroke-width", 1.5)
        .selectAll("circle");

      const tooltip = d3.select("#tooltip");

      function updateGraph() {
        nodes = Array.from(visibleNodes.values());
        links = visibleLinks.slice();

        simulation.nodes(nodes);
        simulation.force("link").links(links);

        link.data(links, d => `${d.source}->${d.target}`)
          .join("line")
          .attr("stroke-width", 1.5);

        node = node.data(nodes, d => d.id)
          .join(
            enter => enter.append("circle")
              .attr("r", 10)
              .attr("fill", d => color(d.group?.rdf_Property || "default"))
              .on("click", (event, d) => {
                toggleExpandCollapse(d.id);
                updateGraph();
              })
              .on("mouseover", (event, d) => {
                tooltip.style("opacity", 1).html(`
                  <strong>${d.group?.identifier || d.id.split("#").pop()}</strong><br/>
                  <em>${d.group?.rdfs_comment || d.group?.rdfs_isDefinedBy || "No description."}</em>
                `);
              })
              .on("mousemove", event => {
                tooltip.style("left", (event.pageX + 10) + "px")
                  .style("top", (event.pageY + 10) + "px");
              })
              .on("mouseout", () => tooltip.style("opacity", 0))
              .call(d3.drag()
                .on("start", dragstarted)
                .on("drag", dragged)
                .on("end", dragended)
              )
          );

        simulation.alpha(1).restart();
      }

      simulation.on("tick", () => {
        g.selectAll("line")
          .attr("x1", d => d.source.x)
          .attr("y1", d => d.source.y)
          .attr("x2", d => d.target.x)
          .attr("y2", d => d.target.y);

        g.selectAll("circle")
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

      updateGraph();
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
