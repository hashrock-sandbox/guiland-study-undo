<template>
  <svg :width="width" :height="height" ref="canvas">
    <!-- add blur filter -->
    <defs>
      <filter id="blur">
        <feGaussianBlur in="SourceGraphic" stdDeviation="5" />
      </filter>
      <!-- add drop shadow filter -->
      <filter id="dropshadow">
        <feGaussianBlur in="SourceAlpha" stdDeviation="3" />
        <feOffset dx="4" dy="4" result="offsetblur" />
        <feComponentTransfer>
          <feFuncA type="linear" slope="0.1" />
        </feComponentTransfer>
        <feMerge>
          <feMergeNode />
          <feMergeNode in="SourceGraphic" />
        </feMerge>
      </filter>
    </defs>
    <!-- create Miro like app -->
    <Card
      @pointerdown="onPointerDown($event, item)"
      @pointerup="onPointerUp"
      @pointermove="onPointerMove"
      :key="item.id"
      v-for="item in items"
      :item="item"
    />
  </svg>
</template>

<script lang="ts">
interface Card {
  text: string;
  x: number;
  y: number;
  width: number;
  height: number;
  color: string;
  id: string;
}

interface Point {
  x: number;
  y: number;
}
function screenToSvg(
  point: Point,
  el: SVGGraphicsElement,
  svg: SVGSVGElement
): Point {
  const pt = svg.createSVGPoint();
  pt.x = point.x;
  pt.y = point.y;
  return pt.matrixTransform(el?.getScreenCTM()?.inverse());
}

const fancyColors = [
  "#FCD34D",
  "#6EE7B7",
  "#93C5FD",
  "#A5B4FC",
  "#C4B5FD",
  "#F9A8D4",
  "#FCA5A5",
]

import Card from "./Card.vue";
export default {
  name: "HelloWorld",
  data() {
    return {
      width: 500,
      height: 500,
      offset: null as Point | null,
      items: [
        {
          text: "Card 1",
          x: 0,
          y: 0,
          width: 300,
          height: 300,
          color: fancyColors[0],
          id: "1",
        },
        {
          text: "Card 2",
          x: 100,
          y: 200,
          width: 300,
          height: 300,
          color: fancyColors[1],
          id: "2",
        },
        {
          text: "Card 3",
          x: 200,
          y: 300,
          width: 300,
          height: 300,
          color: fancyColors[2],
          id: "3",
        },

] as Card[],
      currentItem: null as Card | null,
      count: 0,
    };
  },
  components: {
    Card,
  },
  methods: {
    onPointerDown(ev: PointerEvent, item: Card) {
      const target = ev.target as SVGGraphicsElement;
      target.setPointerCapture(ev.pointerId);
      this.offset = screenToSvg(
        {
          x: ev.clientX,
          y: ev.clientY,
        },
        target,
        this.$refs.canvas as SVGSVGElement
      );
      this.currentItem = item;
    },
    onPointerMove(ev: PointerEvent) {
      if (this.offset) {
        if (this.currentItem) {
          this.currentItem.x = ev.clientX - this.offset.x;
          this.currentItem.y = ev.clientY - this.offset.y;
        }
      }
    },
    onPointerUp(ev: PointerEvent) {
      this.offset = null;
      this.currentItem = null;
    },
    onResize() {
      this.width = window.innerWidth;
      this.height = window.innerHeight;
    },
  },
  mounted() {
    document.addEventListener("resize", this.onResize);
    this.onResize();
  },
};
</script>


<style>
body {
  background: #fafafa;
}

.card {
  filter: url(#dropshadow);
}
</style>
