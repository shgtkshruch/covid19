<template>
  <div>
    <chart-legend
      v-if="legends.length > 0"
      :legends="legends"
      :display-legends="displayLegends"
      @click="onClickLegend"
    />
    <scrollable :display-data="displayData">
      <template v-slot:chart="{ chartWidth }">
        <aria-labelledby :title="title" :title-id="titleId">
          <bar
            :chart-id="chartId"
            :chart-data="displayData"
            :options="displayOption"
            :display-legends="displayLegends"
            :height="height"
            :width="chartWidth"
          />
        </aria-labelledby>
      </template>
      <template v-slot:sticky-chart>
        <slot name="sticky-chart" />
      </template>
    </scrollable>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue'
import { ThisTypedComponentOptionsWithRecordProps } from 'vue/types/options'

import AriaLabelledby from '@/components/chart/AriaLabelledby.vue'
import ChartLegend, { LegendLabel } from '@/components/chart/Legend.vue'
import Scrollable from '@/components/chart/Scrollable.vue'
import { DisplayData } from '@/plugins/vue-chart'

type Data = {
  displayLegends: boolean[]
}
type Methods = {
  onClickLegend: (i: number) => void
}
type Computed = {}
type Props = {
  displayData: DisplayData
  displayOption: Chart.ChartOptions
  legends: LegendLabel[]
  title: string
  titleId: string
  chartId: string
  height: number
}

const options: ThisTypedComponentOptionsWithRecordProps<
  Vue,
  Data,
  Methods,
  Computed,
  Props
> = {
  components: { AriaLabelledby, ChartLegend, Scrollable },
  props: {
    displayData: {
      type: Object as PropType<DisplayData>,
      required: true,
    },
    displayOption: {
      type: Object as PropType<Chart.ChartOptions>,
      required: true,
    },
    legends: {
      type: Array as PropType<LegendLabel[]>,
      default: () => [],
      required: false,
    },
    title: {
      type: String,
      required: true,
    },
    titleId: {
      type: String,
      required: true,
    },
    chartId: {
      type: String,
      required: true,
    },
    height: {
      type: Number,
      default: 240,
    },
  },
  data() {
    return {
      displayLegends: Array.from(
        { length: this.displayData.datasets.length },
        () => true
      ),
    }
  },
  methods: {
    onClickLegend(i) {
      this.displayLegends[i] = !this.displayLegends[i]
      this.displayLegends = this.displayLegends.slice()
    },
  },
}

export default options
</script>
