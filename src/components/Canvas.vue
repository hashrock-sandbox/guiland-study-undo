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
      @dblclick="onDblClick($event, item)"
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
import { Point, CardItem } from "../type";
import Card from "./Card.vue";

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

export default {
  name: "Canvas",
  props: {
    items: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      width: 500,
      height: 500,
      offset: null as Point | null,

      currentItem: null as CardItem | null,
      count: 0,
    };
  },
  components: {
    Card,
  },
  methods: {
    onPointerDown(ev: PointerEvent, item: CardItem) {
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
    onDblClick(ev: PointerEvent, item: CardItem) {
      this.$emit("edit", item);
    },
  },
  mounted() {
    document.addEventListener("resize", this.onResize);
    this.onResize();
  },
  beforeUnmount() {
    document.removeEventListener("resize", this.onResize);
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
