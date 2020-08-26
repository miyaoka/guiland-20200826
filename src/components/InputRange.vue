<template>
  <div class="inputRange">
    <div class="label">
      {{ label }}
    </div>
    <input
      type="range"
      step="any"
      v-bind="$attrs"
      :value="value"
      @input="$emit('update:value', Number($event.target.value))"
    />
    <div class="value">
      {{ Number(value).toFixed(digits) }}
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  props: {
    label: {
      type: String,
      default: '',
    },
    digits: {
      type: Number,
      default: 0,
    },
    value: {
      type: Number,
      required: true,
    },
  },
  // computed: {
  //   listeners(): any {
  //     const a = {
  //       ...this.$attrs,
  //       input: (event: InputEvent) =>
  //         this.$emit('input', (event.target as HTMLInputElement).value),
  //     }
  //     return a
  //   },
  // },
})
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.inputRange {
  position: relative;
  margin-top: 1em;

  input {
    width: 100%;
  }
  .label,
  .value {
    position: absolute;
    bottom: 100%;
    user-select: none;
  }
  .value {
    right: 0;
  }
}
</style>
