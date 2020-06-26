<template>
  <ul :class="$style.GraphLegend">
    <li v-for="(label, i) in labels" :key="i" @click="onClickLegend(i)">
      <button>
        <div :style="legend.style" />
        <span
          :style="{
            textDecoration: displayLegends[i] ? 'none' : 'line-through',
          }"
        >
          {{ legend.label }}
        </span>
      </button>
    </li>
  </ul>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue'
import { ThisTypedComponentOptionsWithRecordProps } from 'vue/types/options'
import { Legend } from '../ChartLegend.vue'

export interface LegendLabel {
  style: {}
  text: string
}
type Data = {}
type Methods = {
  onClickLegend: (i: number) => void
}
type Computed = {}
type Props = {
  legends: LegendLabel[]
  displayLegends: boolean[]
}

const options: ThisTypedComponentOptionsWithRecordProps<
  Vue,
  Data,
  Methods,
  Computed,
  Props
> = {
  props: {
    legends: {
      type: Array as PropType<LegendLabel[]>,
      required: true,
    },
    displayLegends: {
      type: Array,
      required: true,
    },
  },
  methods: {
    onClickLegend(i) {
      this.$emit('click', i)
    },
  },
}

export default Vue.extend(options)
</script>

<style module lang="scss">
.Graph {
  &Legend {
    text-align: center;
    list-style: none;
    padding: 0 !important;
    li {
      display: inline-block;
      margin: 0 3px;
      div {
        height: 12px;
        margin: 2px 4px;
        width: 40px;
        display: inline-block;
        vertical-align: middle;
        border-width: 1px;
        border-style: solid;
      }
      button {
        color: $gray-3;
        @include font-size(12);
      }
    }
  }
}
</style>
