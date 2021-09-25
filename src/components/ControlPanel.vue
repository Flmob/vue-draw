<template>
  <div>
    <div>
      <span>{{ pencil.x }}, {{ pencil.y }}</span>
    </div>
    <span>{{ pencil.lineWidth }}, {{ pencil.currentColor }}</span>
    <div>
      <div>
        <button @click="setPencilWidth(+pencil.lineWidth - 1)">-</button>
        <input
          type="range"
          min="1"
          max="200"
          step="1"
          :value="pencil.lineWidth"
          @input="onPencilWidthChange"
        />
        <button @click="setPencilWidth(+pencil.lineWidth + 1)">+</button>
      </div>

      <div>
        <button @click="setOpacity(+opacity - 0.1)">-</button>
        <input
          type="range"
          min="0"
          max="1"
          step="0.1"
          :value="opacity"
          @input="onOpacityChange"
        />
        <button @click="setOpacity(+opacity + 0.1)">+</button>
      </div>

      <button @click="undoLastAction()">Undo</button>
      <button @click="fillWholeScreen()">Fill screen</button>
    </div>
    <div>
      <div
        :class="['color-btn', { selected: pencil.currentColor === color }]"
        v-for="color in colors"
        :key="color"
        @click="setColor(color)"
        :style="{ backgroundColor: color }"
      />
    </div>
    <button @click="saveImage">Convert to PNG</button>
  </div>
</template>

<script>
import { colors } from "../constants";

export default {
  data() {
    return {
      colors,
    };
  },
  props: {
    opacity: Number,
    pencil: Object,
    setColor: Function,
    saveImage: Function,
    undoLastAction: Function,
    fillWholeScreen: Function,
    setOpacity: Function,
    setPencilWidth: Function,
  },
  methods: {
    onOpacityChange(e) {
      this.setOpacity(e.target.value);
    },
    onPencilWidthChange(e) {
      this.setPencilWidth(e.target.value);
    },
  },
};
</script>

<style>
</style>