<template>
  <g :style="{'transform' : transform}">
    <rect class="card" x="0" y="0" :width="item.width" :height="item.height" :fill="item.color" filter="url(#dropshadow)" />
    <text
      :x="item.width / 2"
      :y="item.height / 2"
      text-anchor="middle"
      alignment-baseline="middle"
      font-size="20"
      fill="black"
    >{{item.text}}</text>
  </g>
</template>
<script lang="ts">

export default {
  name: 'Card',
  props: {
    item: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      text: 'text',
    }
  },
  computed: {
    transform(): string {
      return `translate(${this.item.x}px, ${this.item.y}px)`
    }
  }
}
</script>

<style>
.card{
  cursor: grab;
}
.card:active{
  cursor: grabbing;
}

</style>
