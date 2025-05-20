<template>
  <div class="highcharts-figure">
    <div id="container"></div>
  </div>
</template>

<script>
export default {
  name: "InvertedTreegraphHighchart",
  mounted() {
    if (window.Highcharts) {
      this.drawChart();
    } else {
      console.error("Highcharts is not loaded.");
    }
  },
  methods: {
    drawChart() {
      const data = [
        { id: "0.0", parent: "", name: "Disease" },
        { id: "1.1", parent: "0.0", name: "Parkinson" },
        { id: "1.2", parent: "0.0", name: "Dystonia" },
        { id: "1.3", parent: "0.0", name: "ALS" },
        { id: "1.4", parent: "0.0", name: "Huntington's Disease" },

        { id: "2.1", parent: "1.1", name: "Tremor" },
        { id: "2.2", parent: "1.1", name: "Rigidity" },
        { id: "2.3", parent: "1.1", name: "Bradykinesia" },
        { id: "2.4", parent: "1.1", name: "Postural Instability" },

        { id: "2.5", parent: "1.2", name: "Blepharospasm" },
        { id: "2.6", parent: "1.2", name: "Cervical Dystonia" },
        { id: "2.7", parent: "1.2", name: "Oromandibular Dystonia" },
        { id: "2.8", parent: "1.2", name: "Spasmodic Dysphonia" },

        { id: "2.9", parent: "1.3", name: "Muscle Weakness" },
        { id: "2.10", parent: "1.3", name: "Speech Difficulty" },
        { id: "2.11", parent: "1.3", name: "Breathing Difficulty" },

        { id: "2.12", parent: "1.4", name: "Chorea" },
        { id: "2.13", parent: "1.4", name: "Cognitive Decline" },
        { id: "2.14", parent: "1.4", name: "Psychiatric Symptoms" }
      ];

      Highcharts.chart("container", {
        chart: {
          inverted: true,
          marginBottom: 170
        },
        title: {
          text: "Inverted Treegraph - Disease Hierarchy",
          align: "left"
        },
        series: [
          {
            type: "treegraph",
            data,
            tooltip: {
              pointFormat: "{point.name}"
            },
            dataLabels: {
              pointFormat: "{point.name}",
              style: {
                whiteSpace: "nowrap",
                color: "#000000",
                textOutline: "3px contrast"
              },
              crop: false
            },
            marker: {
              radius: 6
            },
            levels: [
              {
                level: 1,
                dataLabels: {
                  align: "left",
                  x: 20
                }
              },
              {
                level: 2,
                colorByPoint: true,
                dataLabels: {
                  verticalAlign: "bottom",
                  y: -20
                }
              },
              {
                level: 3,
                colorVariation: {
                  key: "brightness",
                  to: -0.5
                },
                dataLabels: {
                  verticalAlign: "top",
                  rotation: 90,
                  y: 20
                }
              }
            ]
          }
        ]
      });
    }
  }
};
</script>

<style scoped>
.highcharts-figure {
  max-width: 900px;
  margin: 1rem auto;
}

#container {
  min-width: 360px;
  height: 600px;
}
</style>
