<template>
  <data-view :title="title" :title-id="titleId" :date="date">
    <template v-slot:description>
      <slot name="description" />
    </template>
    <template v-slot:button>
      <data-selector
        v-model="dataKind"
        :target-id="chartId"
        :style="{ display: canvas ? 'inline-block' : 'none' }"
      />
    </template>
    <scrollable-bar-chart
      v-show="canvas"
      :title="title"
      :title-id="titleId"
      :chart-id="chartId"
      :display-data="displayData"
      :display-option="displayOption"
      :legends="legends"
    />
    <template v-slot:dataTable>
      <client-only>
        <data-view-table :headers="tableHeaders" :items="tableData" />
      </client-only>
    </template>
    <template v-slot:additionalDescription>
      <slot name="additionalDescription" />
    </template>
    <template v-slot:infoPanel>
      <data-view-basic-info-panel
        :l-text="displayInfo.lText"
        :s-text="displayInfo.sText"
        :unit="displayInfo.unit"
      />
    </template>
  </data-view>
</template>

<script lang="ts">
import { Chart } from 'chart.js'
import dayjs from 'dayjs'
import Vue from 'vue'
import { TranslateResult } from 'vue-i18n'
import { ThisTypedComponentOptionsWithRecordProps } from 'vue/types/options'

import { LegendLabel } from '@/components/chart/Legend.vue'
import ScrollableBarChart from '@/components/chart/ScrollableBarChart.vue'
import DataSelector from '@/components/DataSelector.vue'
import DataView from '@/components/DataView.vue'
import DataViewBasicInfoPanel from '@/components/DataViewBasicInfoPanel.vue'
import DataViewTable, {
  TableHeader,
  TableItem,
} from '@/components/DataViewTable.vue'
import { DisplayData, yAxesBgPlugin } from '@/plugins/vue-chart'
import { getGraphSeriesStyle, SurfaceStyle } from '@/utils/colors'
import { getComplementedDate, getDayjsObject } from '@/utils/formatDate'

interface HTMLElementEvent<T extends HTMLElement> extends MouseEvent {
  currentTarget: T
}
type Data = {
  dataKind: 'transition' | 'cumulative'
  canvas: boolean
  colors: SurfaceStyle[]
  isSmall: boolean
}
type Methods = {
  sum: (array: number[]) => number
  cumulative: (array: number[]) => number[]
  pickLastNumber: (chartDataArray: number[][]) => number[]
  cumulativeSum: (chartDataArray: number[][]) => number[]
  eachArraySum: (chartDataArray: number[][]) => number[]
  handleResize: () => void
}

type Computed = {
  displayInfo: {
    lText: string
    sText: string
    unit: string
  }
  displayData: DisplayData
  displayOption: Chart.ChartOptions
  scaledTicksYAxisMax: number
  tableHeaders: TableHeader[]
  tableData: TableItem[]
  legends: LegendLabel[]
}

type Props = {
  title: string
  titleId: string
  chartId: string
  chartData: number[][]
  date: string
  items: string[]
  labels: string[]
  dataLabels: string[] | TranslateResult[]
  tableLabels: string[] | TranslateResult[]
  unit: string
  yAxesBgPlugin: Chart.PluginServiceRegistrationOptions[]
}

const options: ThisTypedComponentOptionsWithRecordProps<
  Vue,
  Data,
  Methods,
  Computed,
  Props
> = {
  created() {
    this.canvas = process.browser
    this.dataKind =
      this.$route.query.embed && this.$route.query.dataKind === 'cumulative'
        ? 'cumulative'
        : 'transition'
  },
  components: {
    DataView,
    DataSelector,
    DataViewTable,
    DataViewBasicInfoPanel,
    ScrollableBarChart,
  },
  props: {
    title: {
      type: String,
      default: '',
    },
    titleId: {
      type: String,
      required: false,
      default: '',
    },
    chartId: {
      type: String,
      default: 'time-stacked-bar-chart',
    },
    chartData: {
      type: Array,
      required: false,
      default: () => [],
    },
    date: {
      type: String,
      required: true,
      default: '',
    },
    items: {
      type: Array,
      default: () => [],
    },
    labels: {
      type: Array,
      default: () => [],
    },
    dataLabels: {
      type: Array,
      default: () => [],
    },
    tableLabels: {
      type: Array,
      default: () => [],
    },
    unit: {
      type: String,
      default: '',
    },
    yAxesBgPlugin: {
      type: Array,
      default: () => yAxesBgPlugin,
    },
  },
  data: () => ({
    dataKind: 'transition',
    colors: getGraphSeriesStyle(2),
    canvas: true,
    isSmall: false,
  }),
  computed: {
    legends() {
      const dataLabels = this.dataLabels as string[]
      return dataLabels.map((label, i) => {
        const style = {
          backgroundColor: this.colors[i].fillColor,
          borderColor: this.colors[i].strokeColor,
        }
        return { label, style }
      })
    },
    displayInfo() {
      const lastDay = this.labels[this.labels.length - 1]
      const date = this.$d(getDayjsObject(lastDay).toDate(), 'dateWithoutYear')

      if (this.dataKind === 'transition') {
        return {
          lText: this.sum(this.pickLastNumber(this.chartData)).toLocaleString(),
          sText: `${this.$t('{date}の合計', { date })}`,
          unit: this.unit,
        }
      }
      return {
        lText: this.sum(this.cumulativeSum(this.chartData)).toLocaleString(),
        sText: `${this.$t('{date}の合計', { date })}`,
        unit: this.unit,
      }
    },
    displayData() {
      const graphSeries = getGraphSeriesStyle(this.chartData.length)
      if (this.dataKind === 'transition') {
        return {
          labels: this.labels,
          datasets: this.chartData.map((item, index) => {
            return {
              label: this.items[index],
              data: item,
              backgroundColor: graphSeries[index].fillColor,
              borderColor: graphSeries[index].strokeColor,
              borderWidth: 1,
            }
          }),
        }
      }
      return {
        labels: this.labels,
        datasets: this.chartData.map((item, index) => {
          return {
            label: this.items[index],
            data: this.cumulative(item),
            backgroundColor: graphSeries[index].fillColor,
            borderColor: graphSeries[index].strokeColor,
            borderWidth: 1,
          }
        }),
      }
    },
    tableHeaders() {
      return [
        { text: this.$t('日付'), value: 'text' },
        ...(this.tableLabels as string[])
          .reduce((arr, text) => {
            arr.push(
              ...[this.$t('日別'), this.$t('累計')].map(
                (label) => `${text} (${label})`
              )
            )
            return arr
          }, [] as string[])
          .map((text, i) => {
            return {
              text,
              value: String(i),
              align: 'end',
              width: this.isSmall ? '6em' : 'auto',
            }
          }),
      ]
    },
    tableData() {
      return this.labels
        .map((label, i) => {
          return Object.assign(
            { text: label },
            ...this.tableHeaders.map((_, j) => {
              const index = j < 2 ? 0 : 1
              const transition = this.chartData[index]
              const cumulative = this.cumulative(transition)
              return {
                [j]:
                  j % 2 === 0
                    ? transition[i].toLocaleString()
                    : cumulative[i].toLocaleString(),
              }
            })
          )
        })
        .sort((a, b) => dayjs(a.text).unix() - dayjs(b.text).unix())
        .reverse()
    },
    displayOption() {
      const self = this
      const unit = this.unit
      const sumArray = this.eachArraySum(this.chartData)
      const data = this.chartData
      const cumulativeData = this.chartData.map((item) => {
        return this.cumulative(item)
      })
      const cumulativeSumArray = this.eachArraySum(cumulativeData)
      const options: Chart.ChartOptions = {
        tooltips: {
          displayColors: false,
          callbacks: {
            label: (tooltipItem) => {
              let casesTotal, cases, label
              if (this.dataKind === 'transition') {
                casesTotal = sumArray[tooltipItem.index!].toLocaleString()
                cases = data[tooltipItem.datasetIndex!][
                  tooltipItem.index!
                ].toLocaleString()
              } else {
                casesTotal = cumulativeSumArray[
                  tooltipItem.index!
                ].toLocaleString()
                cases = cumulativeData[tooltipItem.datasetIndex!][
                  tooltipItem.index!
                ].toLocaleString()
              }

              label = `${
                this.dataLabels[tooltipItem.datasetIndex!]
              } : ${cases} ${unit}`
              if (this.dataKind === 'cumulative') {
                label = `${label} (${this.$t('合計')}: ${casesTotal} ${unit})`
              }
              return label
            },
            title(tooltipItem, data) {
              const label = data.labels![tooltipItem[0].index!] as string
              return self.$d(getComplementedDate(label), 'dateWithoutYear')
            },
          },
        },
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
        scales: {
          xAxes: [
            {
              id: 'day',
              stacked: true,
              gridLines: {
                display: false,
              },
              ticks: {
                fontSize: 9,
                maxTicksLimit: 20,
                fontColor: '#808080',
                maxRotation: 0,
                callback: (label: string) => {
                  return dayjs(label).format('D')
                },
              },
              // #2384: If you set "type" to "time", make sure that the bars at both ends are not hidden.
              // #2384: typeをtimeに設定する時はグラフの両端が見切れないか確認してください
            },
            {
              id: 'month',
              stacked: true,
              gridLines: {
                drawOnChartArea: false,
                drawTicks: true,
                drawBorder: false,
                tickMarkLength: 10,
              },
              ticks: {
                fontSize: 11,
                fontColor: '#808080',
                padding: 3,
                fontStyle: 'bold',
              },
              type: 'time',
              time: {
                unit: 'month',
                displayFormats: {
                  month: 'MMM',
                },
              },
            },
          ],
          yAxes: [
            {
              stacked: true,
              gridLines: {
                display: true,
                color: '#E5E5E5',
              },
              ticks: {
                suggestedMin: 0,
                suggestedMax: this.scaledTicksYAxisMax,
                maxTicksLimit: 8,
                fontColor: '#808080',
              },
            },
          ],
        },
      }
      if (this.$route.query.ogp === 'true') {
        Object.assign(options, { animation: { duration: 0 } })
      }
      return options
    },
    scaledTicksYAxisMax() {
      return Array.from(this.chartData[0].keys())
        .map((i) => this.chartData[0][i] + this.chartData[1][i])
        .reduce((a, b) => Math.max(a, b), 0)
    },
  },
  methods: {
    cumulative(array: number[]): number[] {
      const cumulativeArray: number[] = []
      let patSum = 0
      array.forEach((d) => {
        patSum += d
        cumulativeArray.push(patSum)
      })
      return cumulativeArray
    },
    sum(array: number[]): number {
      return array.reduce((acc, cur) => {
        return acc + cur
      })
    },
    pickLastNumber(chartDataArray: number[][]) {
      return chartDataArray.map((array) => {
        return array[array.length - 1]
      })
    },
    cumulativeSum(chartDataArray: number[][]) {
      return chartDataArray.map((array) => {
        return array.reduce((acc, cur) => {
          return acc + cur
        })
      })
    },
    eachArraySum(chartDataArray: number[][]) {
      const sumArray: number[] = []
      for (let i = 0; i < chartDataArray[0].length; i++) {
        sumArray.push(chartDataArray[0][i] + chartDataArray[1][i])
      }
      return sumArray
    },
    handleResize() {
      this.isSmall = window.innerWidth <= 400
    },
  },
  mounted() {
    window.addEventListener('resize', this.handleResize)
    this.handleResize()
  },
  destroyed() {
    window.removeEventListener('resize', this.handleResize)
  },
}

export default Vue.extend(options)
</script>
