<template>
  <bar
    class="sticky-legend"
    chart-id=""
    :chart-data="chartData"
    :options="chartOption"
    :display-legends="displayLegends"
    :plugins="yAxesBgPlugin"
    :height="height"
  />
</template>

<script lang="ts">
import { Chart } from 'chart.js'
import Vue, { PropType } from 'vue'
import { ThisTypedComponentOptionsWithRecordProps } from 'vue/types/options'

import { DisplayData } from '@/plugins/vue-chart'
const clone = require('rfdc')()

type Data = {
  chartOption: Chart.ChartOptions
}
type Methods = {}
type Computed = {
  chartData: DisplayData
  yAxesBgPlugin: []
}
type Props = {
  displayData: DisplayData
  displayOption: Chart.ChartOptions
  displayLegends: boolean[]
  height: number
}

const options: ThisTypedComponentOptionsWithRecordProps<
  Vue,
  Data,
  Methods,
  Computed,
  Props
> = {
  props: {
    displayData: {
      type: Object as PropType<DisplayData>,
      required: true,
    },
    displayOption: {
      type: Object as PropType<Chart.ChartOptions>,
      required: true,
    },
    displayLegends: {
      type: Array,
      required: false,
    },
    height: {
      type: Number,
      default: 240,
    },
  },
  data() {
    return {
      chartOption: {},
    }
  },
  computed: {
    chartData() {
      return {
        labels: this.displayData.labels!,
        datasets: this.displayData.datasets.map((dataset) => {
          return {
            type: dataset.type,
            yAxisID: dataset.yAxisID,
            data: dataset.data,
            backgroundColor: 'transparent',
            borderColor: 'transparent',
            borderWidth: 0,
          }
        }),
      }
    },
    yAxesBgPlugin() {
      return []
    },
  },
  created() {
    const opt: Chart.ChartOptions = clone(this.displayOption)
    // 凡例を非表示にする
    opt.legend!.display = false

    // ツールチップを無効化
    opt.tooltips!.enabled = false

    // y軸のグラフ描画エリア内のグリッド線を非表示にする
    opt.scales!.yAxes!.forEach((yAxe) => {
      yAxe.gridLines!.drawOnChartArea = false
    })

    // x軸の文字列を非表示にする
    opt.scales!.xAxes!.forEach((xAxe) => {
      xAxe.gridLines!.display = false
      xAxe.ticks!.fontColor = 'transparent'
    })
    this.chartOption = opt
  },
}
export default options
</script>
