<template>
  <div>
    <h4 :id="`${titleId}-graph`" class="visually-hidden">
      {{ $t(`{title}のグラフ`, { title }) }}
    </h4>
    <div ref="chartWrapper">
      <slot />
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { ThisTypedComponentOptionsWithRecordProps } from 'vue/types/options'

type Data = {}
type Methods = {}
type Computed = {}
type Props = {
  title: string
  titleId: string
}

const options: ThisTypedComponentOptionsWithRecordProps<
  Vue,
  Data,
  Methods,
  Computed,
  Props
> = {
  props: {
    title: {
      type: String,
      required: true,
    },
    titleId: {
      type: String,
      required: true,
    },
  },
  mounted() {
    const chartWrapper = this.$refs.chartWrapper as HTMLElement
    const canvas = chartWrapper.querySelector('canvas')
    const labelledbyId = `${this.titleId}-graph`

    if (canvas) {
      canvas.setAttribute('role', 'img')
      canvas.setAttribute('aria-labelledby', labelledbyId)
    }
  },
}

export default options
</script>
