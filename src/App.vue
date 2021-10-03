<template>
  <myCanvas @edit="onEdit" :items="items" />
  <textarea :style="textareaStyle" v-model="editingText" @blur="onBlur" ref="editingText" />
</template>

<script lang="ts">
import { CardItem, Point } from "./type";
const fancyColors = [
  "#FCD34D",
  "#6EE7B7",
  "#93C5FD",
  "#A5B4FC",
  "#C4B5FD",
  "#F9A8D4",
  "#FCA5A5",
];
import Canvas from "./components/Canvas.vue";
export default {
  name: "App",
  data() {
    return {
      editingText: "",
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
      ] as CardItem[],
      textareaPosition: {
        x: 0,
        y: 0,
      } as Point,
      editingItem: null as CardItem | null,
    };
  },
  components: {
    myCanvas: Canvas,
  },
  computed: {
    textareaStyle() {
      if(this.editingItem == null){
        return {
          display: "none",
        };
      }
      return {
        left: `${this.textareaPosition.x}px`,
        top: `${this.textareaPosition.y}px`,
        width: `${this.editingItem.width}px`,
        height: `${this.editingItem.height}px`,
      };
    },
  },
  methods: {
    onEdit(target: CardItem) {
      this.textareaPosition = {
        x: target.x,
        y: target.y,
      };
      this.editingItem = target;
      this.editingText = target.text;
      this.$nextTick(()=>{
        (this.$refs.editingText as HTMLTextAreaElement).focus();
      })
    },
    onBlur() {
      if (this.editingItem == null) {
        return;
      }
      this.editingItem.text = this.editingText;
      this.editingItem = null;
    },
  }
};
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
}
#app {
  margin: 0;
  height: 100%;
}
textarea{
  position: absolute;
  border: none;
  resize: none;
  font-size: 1.5rem;
  background: rgba(255,255,255,0.8);
}
</style>
