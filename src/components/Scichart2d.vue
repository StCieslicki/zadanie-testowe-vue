<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
     <div id="scichart-root" style="width: 800px; height: 600px; margin: auto;"></div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import {
  SciChartSurface,
  NumericAxis,
  FastLineRenderableSeries,
  XyDataSeries,
  SweepAnimation,
  SciChartJsNavyTheme,
  NumberRange,
  MouseWheelZoomModifier,
  ZoomPanModifier,
  ZoomExtentsModifier, DateTimeNumericAxis, CategoryAxis, FastColumnRenderableSeries
} from "scichart";
import axios from "axios";

async function initSciChart(xValues, prices, amounts) {
  // LICENSING
  // Commercial licenses set your license code here
  // Purchased license keys can be viewed at https://www.scichart.com/profile
  // How-to steps at https://www.scichart.com/licensing-scichart-js/
  // SciChartSurface.setRuntimeLicenseKey("YOUR_RUNTIME_KEY");

  // Initialize SciChartSurface. Don't forget to await!
  const { sciChartSurface, wasmContext } = await SciChartSurface.create("scichart-root", {
    theme: new SciChartJsNavyTheme(),
    title: "SciChart.js First Chart",
    titleStyle: { fontSize: 22 }
  });

  // Create an XAxis and YAxis with growBy padding
  const growBy = new NumberRange(0.01, 0.01);
  sciChartSurface.xAxes.add(new DateTimeNumericAxis(wasmContext, { axisTitle: "X Axis", growBy }));
  sciChartSurface.yAxes.add(new NumericAxis(wasmContext, { axisTitle: "Y Axis", growBy }));

  // add a secondary axis amounts
  sciChartSurface.yAxes.add(new NumericAxis(wasmContext, { id: "VolumeAxisId", isVisible: false, growBy: new NumberRange(0.01,1) }));

  // Create a line series with some initial data
  sciChartSurface.renderableSeries.add(new FastLineRenderableSeries(wasmContext, {
    stroke: "steelblue",
    strokeThickness: 3,
    dataSeries: new XyDataSeries(wasmContext, {
      xValues,
      yValues: prices,
    }),
    animation: new SweepAnimation({ duration: 300, fadeEffect: true })
  }));

  sciChartSurface.renderableSeries.add(new FastColumnRenderableSeries(wasmContext, {
    dataSeries: new XyDataSeries(wasmContext, { xValues, yValues: amounts }),
    yAxisId: "VolumeAxisId",
    strokeThickness: 0,
    dataPointWidth: 0.7,
    opacity: 0.47
  }));


  // Add some interaction modifiers to show zooming and panning
  sciChartSurface.chartModifiers.add(new MouseWheelZoomModifier(), new ZoomPanModifier(), new ZoomExtentsModifier());

  return sciChartSurface;
}

export default defineComponent({
  // Best practise in Vue.js is to ensure that sciChartSurface is deleted on component unmount.
  // Here's one way to do this
  data() {
    return {
      chartInitializationPromise: undefined,
      data: null
    };
  },
  mounted() {
    const username = 'frontend2024';
    const password = 'test';

    const token = `${username}:${password}`;
    // const encodedToken = Buffer.from(token).toString('base64');
    const encodedToken = btoa(token);

    axios
        .get('https://rest.statica.pl/rest/stockquotes/gpw:PLKGHM000017?type=trades&dt_from=2010-01-01&limit=10000', {
          headers: { 'Authorization': 'Basic ' + encodedToken }
        })
        .then(response => {
          this.data = response.data;

          console.log("SciChart2d.vue onMounted");

          // const dts = this.data.map(item => new Date(item.dt).toTimeString().slice(3,9));
          // const dts = this.data.map(item => new Date(item.dt/1000).toTimeString().slice(3,9));
          const dts = this.data.map(item => item.dt);
          const prices = this.data.map(item => item.price);
          const amounts = this.data.map(item => item.amount);

          console.log(new Date(dts[0]).toTimeString().slice(3,9), prices[0], amounts[0]);

          this.chartInitializationPromise = initSciChart(dts, prices, amounts);
        })

    // console.log("SciChart2d.vue onMounted");
    // console.log(this.msg);
    // console.log(this.data);
    // this.chartInitializationPromise = initSciChart();
  },
  beforeUnmount() {
    console.log("SciChart2d.vue beforeUnmount");
    this.chartInitializationPromise.then((sciChartSurface) => {
      console.log("..deleting sciChartSurface");
      sciChartSurface.delete();
    });
  },
  name: "scichart2d",
  props: {
    msg: String,
  }
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>