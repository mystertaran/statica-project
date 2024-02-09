<template>
  <div class="chart-container bg-gray-100 p-4">
    <div ref="sciChartParent" class="sciChartWrapper"></div>
  </div>
</template>

<script>
import { defineComponent, ref, onMounted, nextTick, watchEffect } from "vue";
import * as scichart from "scichart";

export default defineComponent({
  name: "ChartComponent",
  setup() {
    const sciChartParent = ref(null);
    let chartData = [];

    const fetchChartData = async () => {
      const url =
        "https://rest.statica.pl/rest/stockquotes/gpw:PLKGHM000017?type=trades&dt_from=2010-01-01&limit=10000";
      try {
        const response = await fetch(url, {
          headers: new Headers({
            Authorization: "Basic " + btoa("frontend2024:test"),
          }),
        });

        if (!response.ok) {
          throw new Error("Network response was not ok");
        }

        const data = await response.json();

        chartData = data.map((item) => ({
          x: new Date(item.date),
          y: item.close,
          volume: item.volume,
        }));
      } catch (error) {
        console.error("Fetch error:", error);
      }
    };

    const renderChart = async () => {
      try {
        const { sciChartSurface, wasmContext } =
          await scichart.SciChartSurface.create(sciChartParent.value);
        const xAxis = new scichart.CategoryDateAxis(wasmContext);
        const yAxis = new scichart.NumericAxis(wasmContext);

        const lineSeries = new scichart.FastLineRenderableSeries(wasmContext, {
          dataSeries: new scichart.XyDataSeries(wasmContext),
          stroke: "blue",
        });

        const columnSeries = new scichart.FastColumnRenderableSeries(
          wasmContext,
          {
            dataSeries: new scichart.XyDataSeries(wasmContext),
            fill: "blue",
            stroke: "blue",
          }
        );

        sciChartSurface.xAxes.add(xAxis);
        sciChartSurface.yAxes.add(yAxis);
        sciChartSurface.renderableSeries.add(lineSeries);
        sciChartSurface.renderableSeries.add(columnSeries);

        lineSeries.dataSeries.appendRange(
          chartData.map((d) => d.x),
          chartData.map((d) => d.y)
        );
        columnSeries.dataSeries.appendRange(
          chartData.map((d) => d.x),
          chartData.map((d) => d.volume)
        );
      } catch (error) {
        console.error("RenderChart error:", error);
      }
    };

    watchEffect(() => {
      if (sciChartParent.value !== null) {
        renderChart();
      }
    });

    onMounted(async () => {
      await fetchChartData();
      nextTick(async () => {
        await renderChart();
      });
    });
  },
});
</script>
