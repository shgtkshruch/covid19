<template>
  <data-view :title="title" :title-id="titleId" :date="date">
    <scrollable-bar-chart
      v-show="canvas"
      :title="title"
      :title-id="titleId"
      :chart-id="chartId"
      :display-data="displayData"
      :display-option="displayOption"
      :legends="legends"
      :bar-count="25"
      :height="280"
    />
    <template v-slot:dataTable>
      <client-only>
        <data-view-table :headers="tableHeaders" :items="tableData" />
      </client-only>
    </template>
    <template v-slot:additionalDescription>
      <slot name="additionalDescription" />
    </template>
  </data-view>
</template>

<script lang="ts">
import { ChartOptions } from 'chart.js'
import Vue from 'vue'
import VueI18n from 'vue-i18n'
import { ThisTypedComponentOptionsWithRecordProps } from 'vue/types/options'

import { LegendLabel } from '@/components/chart/Legend.vue'
import ScrollableBarChart from '@/components/chart/ScrollableBarChart.vue'
import DataView from '@/components/DataView.vue'
import DataViewTable, {
  TableHeader,
  TableItem,
} from '@/components/DataViewTable.vue'
import AgencyData from '@/data/agency.json'
import { DataSets, DisplayData } from '@/plugins/vue-chart'
import { getGraphSeriesStyle, SurfaceStyle } from '@/utils/colors'

interface AgencyDataSets extends DataSets {
  label: string
}
interface AgencyDisplayData extends DisplayData {
  datasets: AgencyDataSets[]
}

type Data = {
  canvas: boolean
  agencies: VueI18n.TranslateResult[]
  colors: SurfaceStyle[]
}
type Methods = {}
type Computed = {
  displayData: AgencyDisplayData
  displayOption: ChartOptions
  tableHeaders: TableHeader[]
  tableData: TableItem[]
  legends: LegendLabel[]
}
type Props = {
  title: string
  titleId: string
  chartId: string
  chartData: typeof AgencyData
  date: string
  unit: string
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
  components: { DataView, DataViewTable, ScrollableBarChart },
  props: {
    title: {
      type: String,
      required: false,
      default: '',
    },
    titleId: {
      type: String,
      required: false,
      default: '',
    },
    chartId: {
      type: String,
      required: false,
      default: 'agency-bar-chart',
    },
    chartData: Object,
    date: {
      type: String,
      default: '',
    },
    unit: {
      type: String,
      required: false,
      default: '',
    },
  },
  data() {
    const agencies = [
      this.$t('第一庁舎計'),
      this.$t('第二庁舎計'),
      this.$t('議事堂計'),
    ]

    return {
      canvas: true,
      agencies,
      colors: getGraphSeriesStyle(3),
    }
  },
  computed: {
    legends() {
      const dataLabels = this.agencies as string[]
      return dataLabels.map((label, i) => {
        const style = {
          backgroundColor: this.colors[i].fillColor,
          borderColor: this.colors[i].strokeColor,
        }
        return { label, style }
      })
    },
    displayData() {
      return {
        labels: this.chartData.labels as string[],
        datasets: this.chartData.datasets.map((item, index) => {
          return {
            label: this.agencies[index] as string,
            data: item.data,
            backgroundColor: this.colors[index].fillColor,
            borderColor: this.colors[index].strokeColor,
            borderWidth: 1,
          }
        }),
      }
    },
    displayOption() {
      const self = this
      const options: ChartOptions = {
        tooltips: {
          displayColors: false,
          callbacks: {
            title(tooltipItem) {
              const dateString = tooltipItem[0].label
              return self.$t('期間: {duration}', {
                duration: dateString!,
              }) as string
            },
            label(tooltipItem, data) {
              const index = tooltipItem.datasetIndex!
              const title = self.$t(data.datasets![index].label!)
              const num = parseInt(tooltipItem.value!).toLocaleString()
              const unit = self.$t(self.unit)
              return `${title}: ${num} ${unit}`
            },
          },
        },
        legend: {
          display: false,
        },
        maintainAspectRatio: false,
        scales: {
          xAxes: [
            {
              stacked: true,
              gridLines: {
                display: false,
              },
              ticks: {
                fontSize: 9,
                fontColor: '#808080',
              },
            },
          ],
          yAxes: [
            {
              stacked: true,
              gridLines: {
                display: true,
              },
              ticks: {
                fontSize: 9,
                fontColor: '#808080',
                maxTicksLimit: 10,
                callback(label) {
                  return `${label}${self.unit}`
                },
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
    tableHeaders() {
      return [
        { text: this.$t('日付'), value: 'text' },
        ...this.displayData.datasets.map((text, value) => {
          return { text: text.label, value: String(value), align: 'end' }
        }),
      ]
    },
    tableData() {
      return this.displayData.datasets[0].data
        .map((_, i) => {
          return Object.assign(
            { text: this.displayData.labels![i] },
            ...this.displayData.datasets!.map((_, j) => {
              return {
                [j]: this.displayData.datasets[j].data[i].toLocaleString(),
              }
            })
          )
        })
        .reverse()
    },
  },
}

export default Vue.extend(options)
</script>
