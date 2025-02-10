<template>
  <div>
    <h1 class="heading">Insider Trading Volume</h1>
    <div id="chart"></div>
  </div>
</template>

<script>
import * as d3 from "d3";
import insiderTrandingVolumeMock from "../mockData/insiderTrandingVolumeMock.json"; // Adjust the file path accordingly

export default {
  name: "InsiderTradingVolume",
  mounted() {
    const formattedData = this.calculateData(insiderTrandingVolumeMock);
    this.drawChart(formattedData);
  },
  methods: {
    calculateData(data) {
      return data.map((entry) => {
        const sharesSoldIndividualsTotal =
          entry.sharesSold.soldBy.individuals.reduce(
            (acc, cur) => acc + cur.shares,
            0
          );
        const sharesSoldCompaniesTotal =
          entry.sharesSold.soldBy.companies.reduce(
            (acc, cur) => acc + cur.shares,
            0
          );
        const sharesBoughtIndividualsTotal =
          entry.sharesBought.boughtBy.individuals.reduce(
            (acc, cur) => acc + cur.shares,
            0
          );
        const sharesBoughtCompaniesTotal =
          entry.sharesBought.boughtBy.companies.reduce(
            (acc, cur) => acc + cur.shares,
            0
          );

        return {
          timePeriod: entry.timePeriod,
          sharesSoldTotal: {
            individualsTotal: sharesSoldIndividualsTotal,
            companiesTotal: sharesSoldCompaniesTotal,
          },
          sharesBoughtTotal: {
            individualsTotal: sharesBoughtIndividualsTotal,
            companiesTotal: sharesBoughtCompaniesTotal,
          },
        };
      });
    },
    drawChart(data) {
      const margin = { top: 20, right: 20, bottom: 40, left: 120 };
      const width = 900 - margin.left - margin.right;
      const height = data.length * 80;

      const svg = d3
        .select("#chart")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`);

      const xScale = d3
        .scaleLinear()
        .domain([
          0,
          d3.max(data, (d) =>
            Math.max(
              d.sharesSoldTotal.individualsTotal +
                d.sharesSoldTotal.companiesTotal,
              d.sharesBoughtTotal.individualsTotal +
                d.sharesBoughtTotal.companiesTotal
            )
          ),
        ])
        .range([0, width / 2]);

      const yScale = d3
        .scaleBand()
        .domain(data.map((d) => d.timePeriod))
        .range([0, height])
        .padding(0.1);


      // Adding labels for shares sold and bought
      svg.append("text")
        .attr("x", width / 4 + 132) // Position it in the middle of the left half
        .attr("y", -5) // Position above the chart
        .attr("text-anchor", "middle")
        .attr("fill", "#F6AA51")
        .style("font-size", "16px")
        .style("font-weight", "bold")
        .text("Shares sold");

      svg.append("text")
        .attr("x", 2 * width / 4 + 60)  // Position it in the middle of the right half
        .attr("y", -5) // Position above the chart
        .attr("text-anchor", "middle")
        .attr("fill", "#2DC981")
        .style("font-size", "16px")
        .style("font-weight", "bold")
        .text("Shares bought");
      
      svg.append("g")
        .call(d3.axisLeft(yScale))
        .selectAll("text")
        .style("font-size", "32") // Increase the font size
        .style("font-weight", "bold")
        .style("fill", "#ffffff"); // Change the text color to tomato red

         // Add "months" below each tick
         svg
          .selectAll(".y-axis-months")
          .data(data)
          .enter()
          .append("text")
          .attr("x", -10)
          .attr("y", (d) => yScale(d.timePeriod) + yScale.bandwidth() / 2 + 32)
          .attr("text-anchor", "end")
          .attr("font-size", "14px")
          .style("font-weight", "bold")
          .attr("fill", "#6A6A9F")
          .text("months");

      // Apply the solid background color to the right side
      svg.append("rect")
        .attr("x", width/2) // start drawing from the middle of the SVG
        .attr("width", width/2)
        .attr("height", height)
        .attr("fill", "#1E1E2F"); // Change this color as needed

      // Apply the gradient as background to the left side
      const defs = svg.append("defs");
      const gradient = defs.append("linearGradient")
        .attr("id", "svgGradient")
        .attr("x1", "0%")
        .attr("x2", "0%")
        .attr("y1", "57%")
        .attr("y2", "57%");
      gradient.append("stop")
        .attr("offset", "57")
        .attr("stop-color", "#AF966C") // Start of gradient color
        .attr("stop-opacity", 1);
      gradient.append("stop")
        .attr("offset", "57%")
        .attr("stop-color", "#1E3336") // End of gradient color
        .attr("stop-opacity", 1);

      svg.append("rect")
        .attr("width", width/2)
        .attr("height", height)
        .attr("fill", "#262626"); // Adjust this color to differentiate from the left side
      
      // Draw axes
      svg
        .append("g")
        .attr("transform", `translate(0,${height})`)
        .call(d3.axisBottom(xScale.copy().range([width / 2, 0])).ticks(5));

      svg
        .append("g")
        .attr("transform", `translate(${width / 2}, ${height})`)
        .call(d3.axisBottom(xScale).ticks(5));

      // Add central vertical axis line at x = 0
      svg
        .append("line")
        .attr("x1", width / 2)
        .attr("x2", width / 2)
        .attr("y1", 0)
        .attr("y2", height)
        .attr("stroke", "black")
        .attr("stroke-width", 1);
        
         // Add horizontal lines starting above the ticks
        svg
          .selectAll(".grid-line")
          .data(data)
          .enter()
          .append("line")
          .attr("x1", 0)
          .attr("x2", width)
          .attr("y1", (d) => yScale(d.timePeriod))
          .attr("y2", (d) => yScale(d.timePeriod))
          .attr("stroke", "#363661")
          .attr("stroke-width", 1);

      // Drawing bars for shares sold and bought
      data.forEach((d) => {
        const soldIndWidth = xScale(d.sharesSoldTotal.individualsTotal);
        const soldCompWidth = xScale(d.sharesSoldTotal.companiesTotal);
        const boughtIndWidth = xScale(d.sharesBoughtTotal.individualsTotal);
        const boughtCompWidth = xScale(d.sharesBoughtTotal.companiesTotal);

        // Adjust the vertical positioning to keep the bars centered
        const yOffset = yScale(d.timePeriod) + (yScale.bandwidth() - yScale.bandwidth() / 4) / 2;

        // Shares sold
        svg
          .append("rect")
          .attr("x", width / 2 - soldIndWidth - soldCompWidth)
          .attr("y", yOffset)
          .attr("width", soldIndWidth)
          .attr("height", yScale.bandwidth() / 4)
          .attr("fill", "#F6AA51");

        svg
          .append("rect")
          .attr("x", width / 2 - soldCompWidth)
          .attr("y", yOffset)
          .attr("width", soldCompWidth)
          .attr("height", yScale.bandwidth() / 4)
          .attr("fill", "#FF9213");

        // Shares bought
        svg
          .append("rect")
          .attr("x", width / 2)
          .attr("y", yOffset)
          .attr("width", boughtIndWidth)
          .attr("height", yScale.bandwidth() / 4)
          .attr("fill", "#2DC981");

        svg
          .append("rect")
          .attr("x", width / 2 + boughtIndWidth)
          .attr("y", yOffset)
          .attr("width", boughtCompWidth)
          .attr("height", yScale.bandwidth() / 4)
          .attr("fill", "#86B8A1");

         // Position icons and text for sold shares at the beginning of each bar
        svg.append("text")
          .attr("x", width / 2 - soldIndWidth - soldCompWidth) // slightly left to the bar start
          .attr("y", yOffset + yScale.bandwidth() / 2) // below the bar
          .attr("class", "fa-regular fa-user")
          .attr("text-anchor", "start")
          .html(() => `<tspan>${'\uf007'}</tspan> ${d.sharesSoldTotal.individualsTotal}`) // Icon and text
          .attr("fill", "#ffffff")
          .attr("font-size", "10px");

        svg.append("text")
          .attr("x", width / 2 - soldCompWidth) // slightly left to the bar start
          .attr("y", yOffset + yScale.bandwidth() / 2) // below the bar
          .attr("class", "fa-regular fa-building")
          .attr("text-anchor", "start")
          .html(() => `<tspan>${'\uf1ad'}</tspan> ${d.sharesSoldTotal.companiesTotal}`) // Icon and text
          .attr("fill", "#ffffff")
          .attr("font-size", "10px");

        // Position icons and text for bought shares at the beginning of each bar
        svg.append("text")
          .attr("x", width / 2 ) // slightly right to the bar start
          .attr("y", yOffset + yScale.bandwidth() / 2) // below the bar
          .attr("class", "fa-regular fa-user")
          .html(() => `<tspan>${'\uf007'}</tspan> ${d.sharesBoughtTotal.individualsTotal}`) // Icon and text
          .attr("fill", "#ffffff")
          .attr("font-size", "10px");

        svg.append("text")
          .attr("x", width / 2 + boughtIndWidth) // slightly right to the bar start
          .attr("y", yOffset + yScale.bandwidth() / 2) // below the bar
          .attr("class", "fa-regular fa-building")
          .html(() => `<tspan>${'\uf1ad'}</tspan> ${d.sharesBoughtTotal.companiesTotal}`) // Icon and text
          .attr("fill", "#ffffff")
          .attr("font-size", "10px");
      });
    },
  },
};
</script>

<style scoped>
.heading {
  font-size: 40px;
  font-weight: bold;
  color: #ffffff;
  margin-bottom: 20px;
}

#chart {
  margin-top: 20px;
  background-color: #05050F;
  padding: 20px;
  border-radius: 8px;
}
</style>
