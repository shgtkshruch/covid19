<template>
  <data-view :title="title" :title-id="titleId" :date="date">
    <template v-slot:description>
      <slot name="description" />
    </template>
    <scrollable-bar-chart
      v-show="canvas"
      :title="title"
      :title-id="titleId"
      :chart-id="chartId"
      :display-data="displayData"
      :display-option="displayOption"
      :legends="legends"
    >
      <template v-slot:sticky-chart>
        <line-chart
          class="sticky-legend"
          :chart-id="`${chartId}-header`"
          :chart-data="displayDataHeader"
          :options="displayOptionHeader"
          :plugins="yAxesBgPlugin"
          :height="240"
        />
      </template>
    </scrollable-bar-chart>
    <template v-slot:attentionNote>
      <slot name="attentionNote" />
    </template>
    <template v-slot:additionalDescription>
      <slot name="additionalDescription" />
    </template>
    <template v-slot:dataTable>
      <client-only>
        <data-view-table :headers="tableHeaders" :items="tableData" />
      </client-only>
    </template>
    <template v-slot:infoPanel>
      <data-view-basic-info-panel
        :l-text="displayInfo[0].lText"
        :s-text="displayInfo[0].sText"
        :unit="displayInfo[0].unit"
      />
    </template>
    <template v-slot:footer>
      <open-data-link v-show="url" :url="url" />
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
import DataView from '@/components/DataView.vue'
import DataViewBasicInfoPanel from '@/components/DataViewBasicInfoPanel.vue'
import DataViewTable, {
  TableHeader,
  TableItem,
} from '@/components/DataViewTable.vue'
import OpenDataLink from '@/components/OpenDataLink.vue'
import { DisplayData, yAxesBgPlugin } from '@/plugins/vue-chart'
import { getGraphSeriesColor, SurfaceStyle } from '@/utils/colors'
import { GraphDataType } from '@/utils/formatGraph'
import { getNumberToLocaleStringFunction } from '@/utils/monitoringStatusValueFormatters'

type Data = {
  dataKind: 'transition'
  canvas: boolean
  colors: SurfaceStyle[]
}
type Methods = {
  formatDayBeforeRatio: (dayBeforeRatio: number) => string
  makeLineData: (value: number) => number[]
}

type Computed = {
  displayTransitionRatio: string
  displayInfo: [
    {
      lText: string
      sText: string
      unit: string
    }
  ]
  displayData: DisplayData
  displayOption: Chart.ChartOptions
  displayDataHeader: DisplayData
  displayOptionHeader: Chart.ChartOptions
  scaledTicksYAxisMax: number
  tableHeaders: TableHeader[]
  tableData: TableItem[]
  legends: LegendLabel[]
}
type Props = {
  title: string
  titleId: string
  chartId: string
  chartData: GraphDataType[]
  formatter: Function
  date: string
  items: string[]
  unit: string
  url: string
  tableLabels: string[] | TranslateResult[]
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
  },
  components: {
    DataView,
    DataViewTable,
    DataViewBasicInfoPanel,
    ScrollableBarChart,
    OpenDataLink,
  },
  props: {
    title: {
      type: String,
      default: '',
    },
    titleId: {
      type: String,
      default: '',
    },
    chartId: {
      type: String,
      default: 'confirmed-cases-increase-ratio-by-week-chart',
    },
    chartData: {
      type: Array,
      default: () => [],
    },
    formatter: {
      type: Function,
      required: false,
      default: getNumberToLocaleStringFunction(),
    },
    date: {
      type: String,
      required: true,
    },
    items: {
      type: Array,
      default: () => [],
    },
    unit: {
      type: String,
      default: '',
    },
    url: {
      type: String,
      default: '',
    },
    tableLabels: {
      type: Array,
      default: () => [],
    },
    yAxesBgPlugin: {
      type: Array,
      default: () => yAxesBgPlugin,
    },
  },
  data: () => ({
    dataKind: 'transition',
    colors: [getGraphSeriesColor('D')],
    canvas: true,
  }),
  computed: {
    legends() {
      return this.items.map((label, i) => {
        let style
        switch (i) {
          case 1:
            style = {
              background: `repeating-linear-gradient(90deg, ${this.colors[1].fillColor}, ${this.colors[1].fillColor} 2px, #fff 2px, #fff 4px)`,
              border: 0,
              height: '2px',
            }
            break
          case 2:
            style = {
              background: this.colors[1].fillColor,
              border: 0,
              height: '2px',
            }
            break
          default:
            style = {
              backgroundColor: this.colors[i].fillColor,
              borderColor: this.colors[i].strokeColor,
            }
        }
        return { label, style }
      })
    },
    displayTransitionRatio() {
      const lastDay = this.chartData.slice(-1)[0].transition
      const lastDayBefore = this.chartData.slice(-2)[0].transition
      return this.formatDayBeforeRatio(lastDay - lastDayBefore)
    },
    displayInfo() {
      return [
        {
          lText: `${this.formatter(this.chartData.slice(-1)[0].transition)}`,
          sText: `${this.chartData.slice(-1)[0].label} ${this.$t(
            'の数値'
          )}（${this.$t('前日比')}: ${this.displayTransitionRatio} ${
            this.unit
          }）`,
          unit: this.unit,
        },
      ]
    },
    displayData() {
      const style = [getGraphSeriesColor('D')]
      return {
        labels: this.chartData.map((d) => d.label),
        datasets: [
          {
            label: this.items[0],
            data: this.chartData.map((d) => d.transition),
            backgroundColor: style[0].fillColor,
            borderColor: style[0].strokeColor,
            borderWidth: 1,
            pointBackgroundColor: 'rgba(0,0,0,0)',
            pointBorderColor: 'rgba(0,0,0,0)',
            lineTension: 0,
            order: 1,
          },
        ],
      }
    },
    displayOption() {
      const unit = this.unit
      const scaledTicksYAxisMax = this.scaledTicksYAxisMax
      const options: Chart.ChartOptions = {
        tooltips: {
          displayColors: false,
          callbacks: {
            label: (tooltipItem) => {
              const cases = `${this.formatter(
                parseFloat(tooltipItem.value!)
              )} ${unit}`
              return `${
                this.items[tooltipItem.datasetIndex!]
              } : ${cases} ${unit}`
            },
            title(tooltipItem, data) {
              if (tooltipItem[0].datasetIndex! < 1) {
                return data.labels![tooltipItem[0].index!] as string[]
              }
              return ''
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
                maxTicksLimit: 15,
                fontColor: '#808080',
                maxRotation: 0,
              },
              type: 'time',
              time: {
                displayFormats: {
                  day: 'D',
                },
                parser: 'M/D',
                unit: 'day',
              },
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
                parser: 'M/D',
                displayFormats: {
                  month: 'MMM',
                },
              },
            },
          ],
          yAxes: [
            {
              gridLines: {
                display: true,
                color: '#E5E5E5',
              },
              ticks: {
                suggestedMin: 0,
                maxTicksLimit: 8,
                fontColor: '#808080',
                suggestedMax: scaledTicksYAxisMax,
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
    displayDataHeader() {
      return {
        labels: ['2020-01-01'],
        datasets: [
          {
            data: [Math.max(...this.chartData.map((d) => d.transition))],
            backgroundColor: 'transparent',
            pointBackgroundColor: 'rgba(0,0,0,0)',
            pointBorderColor: 'rgba(0,0,0,0)',
            borderColor: 'rgba(0,0,0,0)',
            borderWidth: 0,
          },
        ],
      }
    },
    displayOptionHeader() {
      const options: Chart.ChartOptions = {
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
        tooltips: { enabled: false },
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
                maxTicksLimit: 15,
                fontColor: 'transparent', // #808080
                maxRotation: 0,
              },
              type: 'time',
              time: {
                displayFormats: {
                  day: 'D',
                },
                parser: 'M/D',
                unit: 'day',
              },
            },
            {
              id: 'month',
              stacked: true,
              gridLines: {
                drawOnChartArea: false,
                drawTicks: false, // true -> false
                drawBorder: false,
                tickMarkLength: 10,
              },
              ticks: {
                fontSize: 11,
                fontColor: 'transparent', // #808080
                padding: 13, // 3 + 10(tickMarkLength)
                fontStyle: 'bold',
              },
              type: 'time',
              time: {
                unit: 'month',
                parser: 'M/D',
                displayFormats: {
                  month: 'MMM',
                },
              },
            },
          ],
          yAxes: [
            {
              gridLines: {
                display: true,
                drawOnChartArea: false,
                color: '#E5E5E5', // #E5E5E5
              },
              ticks: {
                suggestedMin: 0,
                maxTicksLimit: 8,
                fontColor: '#808080', // #808080
              },
            },
          ],
        },
        animation: { duration: 0 },
      }
      return options
    },
    scaledTicksYAxisMax() {
      const values = this.chartData.map((d) => d.transition)
      return Math.max(...values)
    },
    tableHeaders() {
      return [
        { text: this.$t('日付'), value: 'text' },
        {
          text: this.tableLabels[0],
          value: 'transition',
          align: 'end',
        },
      ]
    },
    tableData() {
      return this.chartData
        .map((d) => {
          return {
            text: d.label,
            transition: this.formatter(d.transition),
          }
        })
        .sort((a, b) => dayjs(a.text).unix() - dayjs(b.text).unix())
        .reverse()
    },
  },
  methods: {
    formatDayBeforeRatio(dayBeforeRatio: number): string {
      const dayBeforeRatioLocaleString = dayBeforeRatio.toLocaleString()
      switch (Math.sign(dayBeforeRatio)) {
        case 1:
          return `+${dayBeforeRatioLocaleString}`
        case -1:
          return `${dayBeforeRatioLocaleString}`
        default:
          return `${dayBeforeRatioLocaleString}`
      }
    },
    makeLineData(value: number): number[] {
      return this.chartData.map((_) => value)
    },
  },
}

export default Vue.extend(options)
</script>
