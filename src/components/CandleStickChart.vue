<template>
  <div>
    <h1 class="heading">Candle Stick Chart</h1>
    <svg id="candlestick-chart"></svg>
    <svg id="mini-map"></svg>
    <div id="tooltip" class="tooltip" style="display: none"></div>
  </div>
</template>

<script>
import * as d3 from "d3";
import candleStickMock from "../mockData/candleStickMock.json";

export default {
  name: "CandleStickChart",
  mounted() {
    this.drawChart();
  },
  methods: {
    drawChart() {
      const data = candleStickMock.map((d) => ({
        Date: new Date(d.date * 1000), // Convert UNIX timestamp to JavaScript Date
        Open: +d.open,
        High: +d.high,
        Low: +d.low,
        Close: +d.close,
        flag: d.flag || false, // Handle flag field
      }));

      const margin = { top: 15, right: 65, bottom: 60, left: 50 },
        width = 1200 - margin.left - margin.right,
        height = 500 - margin.top - margin.bottom,
        miniMapHeight = 100;

      const svg = d3
        .select("#candlestick-chart")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top})`);

      const miniMap = d3
        .select("#mini-map")
        .attr("width", width + margin.left + margin.right)
        .attr("height", miniMapHeight + margin.top + 25)
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top / 2})`);

      const tooltip = d3
        .select("#tooltip")
        .style("position", "absolute")
        .style("background-color", "white")
        .style("border", "1px solid black")
        .style("padding", "5px")
        .style("border-radius", "5px")
        .style("font-size", "12px");

      const dates = data.map((d) => d.Date);
      const xScale = d3.scaleTime().domain(d3.extent(dates)).range([0, width]);
      const xMiniScale = d3
        .scaleTime()
        .domain(d3.extent(dates))
        .range([0, width]);

      const yScale = d3
        .scaleLinear()
        .domain([d3.min(data, (d) => d.Low), d3.max(data, (d) => d.High)])
        .range([height, 0])
        .nice();

      const yMiniScale = d3
        .scaleLinear()
        .domain([d3.min(data, (d) => d.Low), d3.max(data, (d) => d.High)])
        .range([miniMapHeight, 0])
        .nice();

      const xAxis = d3.axisBottom(xScale).ticks(10);
      const yAxis = d3.axisLeft(yScale);
      const xMiniAxis = d3.axisBottom(xMiniScale).ticks(10); // Mini-map x-axis

      svg
        .append("g")
        .attr("class", "x-axis")
        .attr("transform", `translate(0, ${height})`)
        .call(xAxis);

      svg.append("g").attr("class", "y-axis").call(yAxis);

      // Add Y-axis grid lines
      svg
        .append("g")
        .attr("class", "grid")
        .selectAll(".grid-line")
        .data(yScale.ticks()) // Generate tick values for the Y-axis
        .enter()
        .append("line")
        .attr("class", "grid-line")
        .attr("x1", 0)
        .attr("x2", width) // Full width of the chart
        .attr("y1", (d) => yScale(d))
        .attr("y2", (d) => yScale(d))
        .attr("stroke", "#e0e0e0")
        .attr("stroke-dasharray", "3,3") // Optional: Dashed lines for grid
        .attr("stroke-width", 2);

      // Draw candles
      svg
        .selectAll(".candle")
        .data(data)
        .enter()
        .append("rect")
        .attr("class", "candle")
        .attr("x", (d) => xScale(d.Date) - 2.5)
        .attr("y", (d) => yScale(Math.max(d.Open, d.Close)))
        .attr("width", 5)
        .attr("height", (d) => Math.abs(yScale(d.Open) - yScale(d.Close)))
        .attr("rx", 1)
        .attr("fill", (d) => (d.Open > d.Close ? "red" : "green"))
        .on("mouseover", (event, d) => {
          tooltip
            .style("display", "block")
            .html(
              `<strong>Date:</strong> ${d3.timeFormat("%Y-%m-%d")(d.Date)}<br>
                    <strong>Open:</strong> ${d.Open}<br>
                    <strong>High:</strong> ${d.High}<br>
                    <strong>Low:</strong> ${d.Low}<br>
                    <strong>Close:</strong> ${d.Close}`
            )
            .style("left", `${event.pageX + 10}px`)
            .style("top", `${event.pageY - 30}px`);
        })
        .on("mousemove", (event) => {
          tooltip
            .style("left", `${event.pageX + 10}px`)
            .style("top", `${event.pageY - 30}px`);
        })
        .on("mouseout", () => {
          tooltip.style("display", "none");
        });

      // Draw stems
      svg
        .selectAll(".stem")
        .data(data)
        .enter()
        .append("line")
        .attr("class", "stem")
        .attr("x1", (d) => xScale(d.Date))
        .attr("x2", (d) => xScale(d.Date))
        .attr("y1", (d) => yScale(d.High))
        .attr("y2", (d) => yScale(d.Low))
        .attr("stroke", (d) => (d.Open > d.Close ? "red" : "green"));

      // / Add dashed vertical lines and flag labels
      const flaggedData = data.filter((d) => d.flag); // Filter data with flags
      flaggedData.forEach((d) => {
        const xPosition = xScale(d.Date);
        const yStart = 30; // Start of the line (adjust as necessary)

        // Dashed vertical line
        svg
          .append("line")
          .attr("class", "flag-line")
          .attr("x1", xPosition)
          .attr("x2", xPosition)
          .attr("y1", yStart) // Start slightly below the top
          .attr("y2", height) // End slightly above the bottom
          .attr("stroke", "blue")
          .attr("stroke-dasharray", "4,2");

        // Square box for the flag label
        const labelSize = 40; // Square size
        svg
          .append("rect")
          .attr("class", "flag-label-box")
          .attr("x", xPosition - labelSize / 2)
          .attr("y", yStart - labelSize - 5) // Position above the top of the line
          .attr("width", labelSize)
          .attr("height", labelSize)
          .attr("fill", "blue")
          .attr("rx", 4) // Rounded corners
          .attr("ry", 4);

        // Text inside the square box
        svg
          .append("text")
          .attr("class", "flag-label-text")
          .attr("x", xPosition)
          .attr("y", yStart - labelSize / 2 - 5) // Align with the box
          .attr("text-anchor", "middle")
          .attr("fill", "white")
          .attr("font-size", "12px")
          .text("Flag"); // Display "Flag" text
      });

      // Mini-map and brush logic remains unchanged
      // Draw mini-map area chart
      const area = d3
        .area()
        .x((d) => xMiniScale(d.Date))
        .y0(yMiniScale(d3.min(data, (d) => d.Low)))
        .y1((d) => yMiniScale(d.High));

      miniMap
        .append("path")
        .datum(data)
        .attr("class", "mini-map-area")
        .attr("d", area)
        .attr("fill", "lightblue")
        .attr("stroke", "steelblue");

      // Add x-axis for the mini-map
      miniMap
        .append("g")
        .attr("class", "x-mini-axis")
        .attr("transform", `translate(0, ${miniMapHeight})`)
        .call(xMiniAxis);

      // Add brush
      const brush = d3
        .brushX()
        .extent([
          [0, 0],
          [width, miniMapHeight],
        ])
        .on("brush end", brushed);

      miniMap.append("g").attr("class", "brush").call(brush);

      function brushed({ selection }) {
        if (selection) {
          const [x0, x1] = selection.map(xMiniScale.invert);
          xScale.domain([x0, x1]);

          svg
            .selectAll(".candle")
            .attr("x", (d) => xScale(d.Date) - 2.5)
            .attr("width", 5);

          svg
            .selectAll(".stem")
            .attr("x1", (d) => xScale(d.Date))
            .attr("x2", (d) => xScale(d.Date));

          svg.select(".x-axis").call(d3.axisBottom(xScale).ticks(10));
        }
      }

      // Reset zoom on click outside
      document.addEventListener("click", (event) => {
        const isClickInside = event.target.closest("svg");
        if (!isClickInside) {
          resetZoom();
        }
      });

      function resetZoom() {
        xScale.domain(d3.extent(dates));

        svg
          .selectAll(".candle")
          .attr("x", (d) => xScale(d.Date) - 2.5)
          .attr("width", 5);

        svg
          .selectAll(".stem")
          .attr("x1", (d) => xScale(d.Date))
          .attr("x2", (d) => xScale(d.Date));

        svg.select(".x-axis").call(d3.axisBottom(xScale).ticks(10));
        miniMap.select(".brush").call(brush.move, null);
      }
    },
  },
};
</script>

<style scoped>
.candle {
  stroke: black;
}
.stem {
  stroke-width: 1px;
}
.mini-map-area {
  opacity: 0.8;
}
.tooltip {
  pointer-events: none;
  z-index: 10;
}
.grid-line {
  stroke: #756b6b;
  stroke-dasharray: 2, 2; /* Dashed lines */
  stroke-width: 0.5;
}
.heading {
  font-size: 40px; /* Adjust font size */
  font-weight: bold;
  color: #003366; /* Dark blue color similar to the reference image */
  margin-bottom: 20px; /* Add some space between the heading and the chart */
}
</style>
