<template>
  <myCanvas @edit="onEdit" @change="takeSnapshot" :items="items" />
  <textarea
    @keydown.stop
    :style="textareaStyle"
    v-model="editingText"
    @blur="onBlur"
    ref="editingText"
  />
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

interface TextareaStyle {
  width: string;
  height: string;
  top: string;
  left: string;
  display: string;
}
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
      snapshots: [] as CardItem[][],
      redoSnapshots: [] as CardItem[][],
    };
  },
  components: {
    myCanvas: Canvas,
  },
  computed: {
    textareaStyle(): TextareaStyle {
      if (this.editingItem == null) {
        return {
          display: "none",
          left: "0px",
          top: "0px",
          width: "0px",
          height: "0px",
        };
      }
      return {
        left: `${this.textareaPosition.x}px`,
        top: `${this.textareaPosition.y}px`,
        width: `${this.editingItem.width}px`,
        height: `${this.editingItem.height}px`,
        display: "block",
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
      this.$nextTick(() => {
        (this.$refs.editingText as HTMLTextAreaElement).focus();
      });
    },
    onBlur() {
      if (this.editingItem == null) {
        return;
      }
      this.redoSnapshots = [];
      this.snapshots.push(JSON.parse(JSON.stringify(this.items)));
      this.editingItem.text = this.editingText;
      this.editingItem = null;
    },
    onUndo() {
      if (this.snapshots.length === 0) {
        console.log("no more undo");
        return;
      }
      this.redoSnapshots.push(JSON.parse(JSON.stringify(this.items)));
      const snapshot = this.snapshots.pop();
      if (snapshot) {
        this.items = snapshot;
      }
    },
    onRedo() {
      if (this.redoSnapshots.length === 0) {
        console.log("no more redo");
        return;
      }
      this.snapshots.push(JSON.parse(JSON.stringify(this.items)));
      const snapshot = this.redoSnapshots.pop();
      if (snapshot) {
        this.items = snapshot;
      }
    },
    onKeydown(e: KeyboardEvent) {
      if (e.key === "Escape") {
        this.onBlur();
      }
      //undo
      if (e.keyCode == 90 && e.ctrlKey && !e.shiftKey) {
        this.onUndo();
      }
      //redo
      if (
        (e.keyCode == 89 && e.ctrlKey) ||
        (e.keyCode == 90 && e.ctrlKey && e.shiftKey)
      ) {
        this.onRedo();
      }
    },
    takeSnapshot(snapshot: CardItem[]) {
      this.redoSnapshots = [];
      this.snapshots.push(JSON.parse(JSON.stringify(snapshot)));
    },
  },
  mounted() {
    document.addEventListener("keydown", this.onKeydown);
  },
  unmounted() {
    document.removeEventListener("keydown", this.onKeydown);
  },
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
textarea {
  position: absolute;
  border: none;
  resize: none;
  font-size: 1.5rem;
  background: rgba(255, 255, 255, 0.8);
}
</style>
