<template>
  <div>
    <div class="header-box">
      <h1 class="heading">Assignment</h1>
    </div>

    <div class="dashboard">
      <!-- left chart container -->
      <div class="chart-container">
        <div id="chart"></div>
      </div>

      <!-- info cards inside their own card -->
      <div class="info-container">
        <!-- move select inside its own styled card -->
        <div class="card white">
          <h3>Select Country</h3>
          <select v-model="selectedCountry" @change="drawChart">
            <option v-for="c in countries" :key="c" :value="c">{{ c }}</option>
          </select>
        </div>

        <div class="card green">
          <h3>Country</h3>
          <p>{{ selectedCountry }}</p>
        </div>
        <div class="card tooltip">
          <h3>Year</h3>
          <p>{{ hoverYear || "-" }}</p>
        </div>
        <div class="card orange">
          <h3>GDP</h3>
          <p>{{ hoverGDP || "-" }}</p>
        </div>
        <div class="card purple">
          <h3>Population</h3>
          <p>{{ hoverPopulation || "-" }}</p>
        </div>
      </div>
    </div>
  </div>

  <div class="chart-container">
    <div id="scatterChart"></div>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  data() {
    return {
      selectedCountry: "India",
      countries: [],
      gdpData: {},
      popData: {},
      years: [],
      hoverYear: "",
      hoverGDP: "",
      hoverPopulation: "",
    };
  },
  mounted() {
    this.loadData();
  },
  watch: {
    selectedCountry() {
      this.drawChart();
      this.drawScatterChart();
    },
  },
  methods: {
    async loadData() {
      const gdpRaw = await d3.csv("/GDP.csv");
      const popRaw = await d3.csv("/Population.csv");

      this.gdpData = this.processData(gdpRaw);
      this.popData = this.processData(popRaw);
      this.countries = Object.keys(this.gdpData);
      this.years = this.gdpData[this.selectedCountry].map((d) => d.year);

      this.drawChart();
      this.drawScatterChart();
    },
    processData(data) {
      const structured = {};
      data.forEach((row) => {
        const country = row["Country Name"];
        structured[country] = [];
        Object.keys(row).forEach((key) => {
          if (!isNaN(+key) && +key > 1900) {
            const val = parseFloat(row[key].replace(/,/g, ""));
            structured[country].push({
              year: +key,
              value: isNaN(val) ? 0 : val,
            });
          }
        });
      });
      return structured;
    },
    drawChart() {
      const self = this;
      d3.select("#chart").selectAll("*").remove();

      const width = 1100,
        height = 480,
        margin = { top: 20, right: 60, bottom: 40, left: 70 };
      const svg = d3
        .select("#chart")
        .append("svg")
        .attr("width", width)
        .attr("height", height);

      const x = d3
        .scaleLinear()
        .domain(d3.extent(this.years))
        .range([margin.left, width - margin.right]);

      const yGDP = d3
        .scaleLog()
        .base(10)
        .domain([1e7, 1e14])
        .nice()
        .range([height - margin.bottom, margin.top]);

      const yPOP = d3
        .scaleLinear()
        .domain([0, d3.max(this.popData[this.selectedCountry], (d) => d.value)])
        .nice()
        .range([height - margin.bottom, margin.top]);

      svg
        .append("g")
        .attr("transform", `translate(0,${height - margin.bottom})`)
        .call(
          d3
            .axisBottom(x)
            .tickFormat(d3.format("d"))
            .tickSize(-(height - margin.top - margin.bottom))
            .tickSizeOuter(0)
        )
        .call((g) => {
          g.selectAll(".tick line")
            .attr("stroke", "rgba(0,0,0,0.15)")
            .attr("stroke-dasharray", "2,2");
          g.selectAll(".tick text")
            .attr("fill", "#757575")
            .attr("dy", "1.2em")
            .style("font-weight", "bold");
          g.selectAll(".domain").attr("stroke", "rgba(0,0,0,0.15)");
        });

      svg
        .append("g")
        .attr("transform", `translate(${margin.left},0)`)
        .call(d3.axisLeft(yGDP).ticks(6, "~s"))
        .call((g) => {
          g.selectAll(".tick line").attr("stroke", "rgba(0,0,0,0.15)");
          g.selectAll(".tick text")
            .attr("fill", "#757575")
            .style("font-weight", "bold");
          g.selectAll(".domain").attr("stroke", "rgba(0,0,0,0.15)");
        })
        .append("text")
        .attr("fill", "#6f7288")
        .attr("font-size", "10px")
        .style("font-weight", "bold")
        .attr("transform", "rotate(-90)")
        .attr("x", -height / 2)
        .attr("y", -margin.left + 30)
        .attr("dy", "-1em")
        .attr("text-anchor", "middle")
        .text("GDP");

      svg
        .append("g")
        .attr("transform", `translate(${width - margin.right},0)`)
        .call(d3.axisRight(yPOP).ticks(6).tickFormat(d3.format(".2s")))
        .call((g) => {
          g.selectAll(".tick line").attr("stroke", "rgba(0,0,0,0.15)");
          g.selectAll(".tick text")
            .attr("fill", "#757575")
            .style("font-weight", "bold");
          g.selectAll(".domain").attr("stroke", "rgba(0,0,0,0.15)");
        })
        .append("text")
        .attr("fill", "#6f7288")
        .attr("font-size", "10px")
        .style("font-weight", "bold")
        .attr("transform", "rotate(-90)")
        .attr("x", -height / 2)
        .attr("y", margin.right - 15)
        .attr("dy", "1em")
        .attr("text-anchor", "middle")
        .text("POPULATION");

      // GDP line
      svg
        .append("path")
        .datum(this.gdpData[this.selectedCountry])
        .attr("fill", "none")
        .attr("stroke", "blue")
        .attr("stroke-width", 2)
        .attr(
          "d",
          d3
            .line()
            .x((d) => x(d.year))
            .y((d) => yGDP(d.value))
        );

      // Population line
      svg
        .append("path")
        .datum(this.popData[this.selectedCountry])
        .attr("fill", "none")
        .attr("stroke", "orange")
        .attr("stroke-width", 2)
        .attr(
          "d",
          d3
            .line()
            .x((d) => x(d.year))
            .y((d) => yPOP(d.value))
        );

      // === Hover line and tooltip with triangle ===
      const hoverGroup = svg.append("g").style("opacity", 0);

      // dashed vertical line
      hoverGroup
        .append("line")
        .attr("class", "hover-line")
        .attr("stroke", "#ef4444")
        .attr("stroke-width", 2)
        .attr("stroke-dasharray", "4,4")
        .attr("y1", margin.top)
        .attr("y2", height - margin.bottom);

      // tooltip group
      const label = hoverGroup.append("g").attr("transform", "translate(0,0)");

      // rectangle box
      label
        .append("rect")
        .attr("width", 60)
        .attr("height", 24)
        .attr("x", -30)
        .attr("y", -12)
        .attr("rx", 4)
        .attr("fill", "#ef4444");

      // text inside box
      label
        .append("text")
        .attr("text-anchor", "middle")
        .attr("dy", "0.4em")
        .attr("fill", "#fff")
        .style("font-weight", "bold")
        .style("font-size", "12px")
        .text("");

      // triangle pointer under box
      label
        .append("path")
        .attr("d", "M -5,12 L 5,12 L 0,18 Z")
        .attr("fill", "#ef4444");

      // overlay rect for interaction
      svg
        .append("rect")
        .attr("fill", "transparent")
        .attr("width", width)
        .attr("height", height)
        .on("mousemove", function (event) {
          const [xm] = d3.pointer(event);
          const year = Math.round(x.invert(xm));
          const clampedYear = Math.max(
            Math.min(year, d3.max(self.years)),
            d3.min(self.years)
          );
          const xPos = x(clampedYear);

          // update line and tooltip
          hoverGroup.style("opacity", 1);
          hoverGroup.select(".hover-line").attr("x1", xPos).attr("x2", xPos);

          // move label just below top margin
          label.attr("transform", `translate(${xPos},${margin.top + 5})`);
          label.select("text").text(clampedYear);

          // update info panel
          self.hoverYear = clampedYear;
          const gdpEntry = self.gdpData[self.selectedCountry].find(
            (d) => d.year === clampedYear
          );
          const popEntry = self.popData[self.selectedCountry].find(
            (d) => d.year === clampedYear
          );
          self.hoverGDP = gdpEntry ? d3.format(".2s")(gdpEntry.value) : "-";
          self.hoverPopulation = popEntry
            ? d3.format(".2s")(popEntry.value)
            : "-";
        })
        .on("mouseleave", function () {
          hoverGroup.style("opacity", 0);
          self.hoverYear = self.hoverGDP = self.hoverPopulation = "";
        });
    },
    drawScatterChart() {
      const self = this;
      d3.select("#scatterChart").selectAll("*").remove();

      const width = 1400,
        height = 500,
        margin = { top: 20, right: 60, bottom: 40, left: 70 };
      const svg = d3
        .select("#scatterChart")
        .append("svg")
        .attr("width", width)
        .attr("height", height);

      const gdpData = self.gdpData[self.selectedCountry];
      const popData = self.popData[self.selectedCountry];
      const combined = gdpData.map((d, i) => ({
        year: d.year,
        gdp: d.value,
        pop: popData[i]?.value || 0,
      }));

      const x = d3
        .scaleLinear()
        .domain([0, d3.max(combined, (d) => d.pop)])
        .nice()
        .range([margin.left, width - margin.right]);

      const y = d3
        .scaleLog()
        .base(10)
        .domain([1e7, 1e14])
        .nice()
        .range([height - margin.bottom, margin.top]);

      // Axes
      svg
        .append("g")
        .attr("transform", `translate(0,${height - margin.bottom})`)
        .call(d3.axisBottom(x).tickFormat(d3.format(".2s")))
        .call((g) => {
          g.selectAll(".tick line")
            .attr("stroke", "rgba(0,0,0,0.15)")
            .attr("stroke-dasharray", "2,2");
          g.selectAll(".tick text")
            .attr("fill", "#757575")
            .style("font-weight", "bold");
          g.selectAll(".domain").attr("stroke", "rgba(0,0,0,0.15)");
        });

      svg
        .append("g")
        .attr("transform", `translate(${margin.left},0)`)
        .call(d3.axisLeft(y).ticks(6, "~s"))
        .call((g) => {
          g.selectAll(".tick line").attr("stroke", "rgba(0,0,0,0.15)");
          g.selectAll(".tick text")
            .attr("fill", "#757575")
            .style("font-weight", "bold");
          g.selectAll(".domain").attr("stroke", "rgba(0,0,0,0.15)");
        });

      // Labels
      svg
        .append("text")
        .attr("x", width / 2)
        .attr("y", height - 5)
        .attr("text-anchor", "middle")
        .style("fill", "#6f7288")
        .style("font-size", "10px")
        .style("font-weight", "bold")
        .text("POPULATION");

      svg
        .append("text")
        .attr("transform", "rotate(-90)")
        .attr("x", -height / 2)
        .attr("y", 15)
        .attr("text-anchor", "middle")
        .style("fill", "#6f7288")
        .style("font-size", "10px")
        .style("font-weight", "bold")
        .text("GDP");

      // History line
      const line = svg
        .append("path")
        .attr("fill", "none")
        .attr("stroke", "#6366f1")
        .attr("stroke-width", 2);

      // Animated point
      const circle = svg.append("circle").attr("r", 5).attr("fill", "#ef4444");

      const yearLabel = svg
        .append("text")
        .attr("text-anchor", "middle")
        .attr("fill", "#ef4444")
        .attr("font-weight", "bold")
        .attr("y", margin.top);

      let i = 0;
      function animate() {
        if (i >= combined.length) return;
        const dataSlice = combined.slice(0, i + 1);
        line.attr(
          "d",
          d3
            .line()
            .x((d) => x(d.pop))
            .y((d) => y(d.gdp))(dataSlice)
        );
        const point = dataSlice[dataSlice.length - 1];
        circle.attr("cx", x(point.pop)).attr("cy", y(point.gdp));
        yearLabel
          .attr("x", x(point.pop))
          .attr("y", y(point.gdp) - 10)
          .text(point.year);

        i++;
        setTimeout(animate, 200); // slower animation
      }
      animate();
    },
  },
};
</script>

<style scoped>
.header-box {
  background: #151b42;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}
.heading {
  font-size: 40px;
  font-weight: bold;
  color: #ffffff;
  text-align: center;
}

/* main layout */
.dashboard {
  display: flex;
  gap: 20px;
}

/* chart on left */
.chart-container {
  flex: 3;
  background: #ffffff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.05);
}

/* right side panel */
.info-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
}

/* select country outside the card, styled simply */
.select-country {
  margin-bottom: 15px;
}
.select-country label {
  display: block;
  font-size: 12px;
  text-transform: uppercase;
  font-weight: 600;
  color: #6b7280; /* gray-500 */
  margin-bottom: 4px;
}
.select-country select {
  width: 100%;
  padding: 6px 10px;
  font-size: 16px;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  box-shadow: 0 0 2px rgba(0, 0, 0, 0.05);
}

/* card container */
.info-container {
  background: #ffffff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.05);
  display: flex;
  flex-direction: column;
  gap: 15px;
}

/* cards */
.card {
  background: #fafafa;
  border: 1px solid #e0e0e0;
  border-left-width: 8px;
  border-left-style: solid;
  padding: 12px 15px;
  border-radius: 6px;
}
.card h3 {
  font-size: 12px;
  text-transform: uppercase;
  font-weight: 600;
  color: #6b7280;
  margin: 0 0 5px;
}
.card p {
  font-size: 20px;
  font-weight: 800;
  color: #0f172a;
  margin: 0;
}

.card select {
  width: 100%;
  margin-top: 5px;
  padding: 6px 10px;
  font-size: 16px;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  box-shadow: 0 0 2px rgba(0, 0, 0, 0.05);
}

.card.orange {
  border-left-color: #f59e0b;
}
.card.green {
  border-left-color: #10b981;
}
.card.purple {
  border-left-color: #6366f1;
}
.card.blue {
  border-left-color: #3b82f6;
}
.card.white {
  border-left-color: #4a4a4b;
}

.card.tooltip {
  border-left-color: #ef4444;
}
</style>
