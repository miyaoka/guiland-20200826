<template>
  <div class="hello">
    <section class="control">
      <InputRange
        label="animFreq"
        v-model:value="animFreq"
        min="0"
        max="3.5"
        :digits="2"
      />
      <InputRange
        label="animPhase"
        v-model:value="animPhase"
        min="0"
        max="1"
        :digits="2"
      />
      <InputRange
        label="bodyFreq"
        v-model:value="bodyFreq"
        min="0"
        max="5"
        :digits="2"
      />
      <InputRange
        label="bodyPhase"
        v-model:value="bodyPhase"
        min="0"
        max="1"
        :digits="2"
      />
      <InputRange
        label="cellCount"
        v-model:value="cellCount"
        min="3"
        max="20"
      />
      <InputRange
        label="bodyAmp"
        v-model:value="bodyAmp"
        min="0"
        max="1"
        :digits="2"
      />
      <InputRange label="bodyHue" v-model:value="bodyHue" min="0" max="360" />
      <InputRange label="eyeHue" v-model:value="eyeHue" min="0" max="360" />
    </section>
    <section class="canvas">
      <svg :width="radius * 3" :height="radius * 3">
        <g transform="translate(300 300)">
          <!-- <path :d="path" fill="transparent" stroke="black" /> -->
          <circle
            v-for="(cell, i) in pointList"
            :key="i"
            :transform="
              `translate(${cell.point[0]} ${cell.point[1]}) rotate(${
                cell.rotation
              }) scale(${1 + animVal * 0.05} ${cell.scale + animVal * 0.05}) `
            "
            :r="cell.cellSize"
            :fill="`hsl(${bodyHue},100%,45%)`"
          />

          <g
            v-for="(cell, i) in pointList.filter(cell => cell.hasEye)"
            :key="i"
            :transform="
              `translate(${cell.point[0] + cell.eyePos.x} ${cell.point[1] +
                +cell.eyePos.y}) rotate(${cell.rotation}) scale(${1 +
                animVal * 0.05} ${cell.scale + animVal * 0.05})`
            "
          >
            <circle :r="cell.eyeSize" fill="white" />
            <circle
              :cx="cell.eyeBallPos.x"
              :cy="cell.eyeBallPos.y"
              :r="cell.eyeBallSize"
              :fill="`hsl(${eyeHue},100%,36%)`"
            />
          </g>
        </g>
      </svg>
    </section>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import InputRange from '@/components/InputRange.vue'
const pi2 = Math.PI * 2

export default defineComponent({
  components: {
    InputRange,
  },
  props: {
    msg: String,
  },
  data() {
    return {
      animFreq: 0.4,
      animPhase: 0,
      bodyFreq: 2.4,
      bodyPhase: 0,
      bodyAmp: 0.5,
      bodyHue: 355,
      eyeHue: 206,
      lastTime: 0,
      radius: 200,
      cellCount: 11,
    }
  },
  computed: {
    cellList(): any[] {
      const count = Math.round(this.cellCount)
      return [...Array(count)]
        .reduce((acc: boolean[]) => {
          return [
            ...acc,
            acc[acc.length - 1] || Math.random() < 0.5 ? false : true,
          ]
        }, [])
        .map((hasEye, i) => {
          const phase =
            ((i + Math.random() * Math.random() * 0.2) / count) * pi2

          const cellSize = 60 + Math.random() * 20
          const eyeSize = (0.3 + Math.random() * 0.4) * cellSize
          const eyeRad = Math.random() * pi2
          const eyeSizeRest = (cellSize - eyeSize) * Math.random() * 0.8
          const eyePos = {
            x: Math.cos(eyeRad) * eyeSizeRest,
            y: Math.sin(eyeRad) * eyeSizeRest,
          }
          const eyeBallRad = Math.random() * pi2

          const eyeBallSize = (0.4 + Math.random() * 0.2) * eyeSize
          const eyeBallPos = {
            x: Math.cos(eyeBallRad) * (eyeSize - eyeBallSize),
            y: Math.sin(eyeBallRad) * (eyeSize - eyeBallSize),
          }

          return {
            phase,
            cellSize,
            hasEye,
            eyeSize,
            eyePos,
            eyeBallSize,
            eyeBallPos,
            scale: 1 - Math.random() * Math.random() * 0.5,
            rotation: Math.random() * 360,
          }
        })
    },
    animVal(): number {
      return Math.sin(this.animPhase * pi2)
    },
    pointList(): any[] {
      const bodyAmp = this.bodyAmp + this.animVal * 0.1
      const bodyPhase = this.bodyPhase + this.animVal * 0.1
      const pointList = this.cellList.map(cell => {
        const amp =
          (Math.sin((cell.phase + bodyPhase * Math.PI) * this.bodyFreq) + 1) / 2

        const rad = this.radius * (1 - bodyAmp + amp * bodyAmp)
        return {
          ...cell,
          point: [Math.cos(cell.phase), -Math.sin(cell.phase)].map(v =>
            Math.round(v * rad)
          ),
        }
      })

      return pointList

      // const reversePointList = pointList
      //   .slice()
      //   .reverse()
      //   .map(point => [point[0], -point[1]])

      // return [...pointList, ...reversePointList]
    },
    path(): string {
      return (
        'M' +
        this.pointList[0].point.join(' ') +
        ' ' +
        this.pointList.map(({ point }) => `L` + point.join(' ')).join(' ') +
        ' Z'
      )
    },
  },
  mounted() {
    this.lastTime = new Date().getTime()
    requestAnimationFrame(this.step)
  },
  methods: {
    step() {
      const time = new Date().getTime()
      const diff = time - this.lastTime
      this.lastTime = time

      this.animPhase = (this.animPhase + (diff * this.animFreq) / 1000) % 1
      requestAnimationFrame(this.step)
    },
  },
})
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
svg {
  border: 1px solid #ccc;
}
</style>
