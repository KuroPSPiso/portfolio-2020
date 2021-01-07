<template>
  <div>
    <canvas id="view" ref="canvasview" :color="fillColor" resize />
  </div>
</template>

<script>
// TODO: move all of this logic to master
// packages
const paper = require('paper')

export default {
  name: 'Canvas',
  props: ['canvasId', 'fillColor'],
  data: () => ({
    path: null,
    width: null,
    height: null,
    center: null,
    points: 11,
    smooth: true,
    paper: null
  }),
  created () {
    // Create and store the Paper instance in a Vue variable (this.paper)
    this.paper = paper.setup(this.$refs.canvasview)
  },
  methods: {
    initializePath () {
      this.center = this.paper.view.center
      this.width = this.paper.view.size.width * 2
      this.height = this.paper.view.size.height / 2
      this.path.segments = []
      this.path.add(this.paper.view.bounds.bottomLeft)
      for (let i = 1; i < this.points; i++) {
        const point = new paper.Point((0 - this.paper.view.size.width / 2) + (this.width / this.points * i), this.center.y)
        this.path.add(point)
      }
      this.path.add(this.paper.view.bounds.bottomRight)
      this.path.fullySelected = false
    },
    tick () {
      const now = Date.now()

      if (this.lastUpdate === null) {
        this.lastUpdate = now
      }

      let dt = now - this.lastUpdate
      dt /= 1000
      this.lastUpdate = now

      this.pathHeight += (this.paper.center.y - this.pathHeight) - (this.paper.view.size.height / 4)
      for (let i = 1; i < this.points; i++) {
        const sinSeed = dt + (i + i % 10) * 40
        const sinHeight = Math.sin(sinSeed / 300) * this.pathHeight
        const yPos = Math.sin(sinSeed / 300) * sinHeight + this.height
        this.path.segments[i].point.y = yPos
      }
      if (this.smooth) {
        this.path.smooth({ type: 'continuous' })
      }
    }
  },
  mounted () {
    this.path = new paper.Path()
    this.initializePath()
    setInterval(this.tick, 0)
  }
}
</script>

<style scoped>
  .canvas-style {
    position: absolute;
    cursor: crosshair;
    width: 100vw !important;
    height: 100vh !important;
    display: block;
    margin: auto;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 0;
  }
</style>
