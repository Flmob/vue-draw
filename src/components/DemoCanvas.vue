<template>
  <div class="wrapper">
    <canvas
      id="myCanvas"
      :width="width"
      :height="height"
      @mousemove="draw"
      @mousedown="beginDrawing"
      @mouseup="stopDrawing"
      @mouseleave="stopDrawing"
      @touchmove.prevent="touchDraw"
      @touchstart="beginDrawing"
      @touchend="stopDrawing"
      :style="{
        cursor: cursorsUrls.PENCIL,
        opacity,
      }"
    >
      Your browser does not support canvas. Please, use chrome.
    </canvas>
    <control-panel
      :pencil="pencil"
      :setColor="setColor"
      :undoLastAction="undoLastAction"
      :fillWholeScreen="fillWholeScreen"
      :saveImage="saveImage"
      :opacity="opacity"
      :setOpacity="setOpacity"
      :setPencilWidth="setPencilWidth"
    />
  </div>
</template>

<script scoped>
import ControlPanel from "./ControlPanel";

import { colors } from "../constants";

const cursorsUrls = {
  PENCIL:
    "url('data:image/x-icon;base64,AAACAAEAICAQAAIADQDoAgAAFgAAACgAAAAgAAAAQAAAAAEABAAAAAAAAAIAAAAAAAAAAAAAEAAAAAAAAAAAAAAABXUdAFjG/ADone0AAHj3AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAwAAAAAAAAAAAAAAAAAAADMwAAAAAAAAAAAAAAAAAAETMwAAAAAAAAAAAAAAAABEETAAAAAAAAAAAAAAAAAEREEAAAAAAAAAAAAAAAAARERAAAAAAAAAAAAAAAAABEREAAAAAAAAAAAAAAAAAEREQAAAAAAAAAAAAAAAAARERAAAAAAAAAAAAAAAAABEREAAAAAAAAAAAAAAAAACJEQAAAAAAAAAAAAAAAAAAiJAAAAAAAAAAAAAAAAAACIiAAAAAAAAAAAAAAAAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAD///////////////////////3////4////8H///+D////B////g////wf///4P///8H///+D////B////w////4f///+f//////////////////////////////////////////////////////////////////////////////w==')",
  ERASER:
    "url('data:image/x-icon;base64,AAACAAEAICAQAAIADQDoAgAAFgAAACgAAAAgAAAAQAAAAAEABAAAAAAAAAIAAAAAAAAAAAAAEAAAAAAAAAAAAAAABXUdAFjG/ADone0AAHj3AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAwAAAAAAAAAAAAAAAAAAADMwAAAAAAAAAAAAAAAAAAMzEQAAAAAAAAAAAAAAAAAAMRRAAAAAAAAAAAAAAAAAAAFERAAAAAAAAAAAAAAAAAAARERAAAAAAAAAAAAAAAAAAARERAAAAAAAAAAAAAAAAAAARERAAAAAAAAAAAAAAAAAAARERAAAAAAAAAAAAAAAAAAARERAAAAAAAAAAAAAAAAAAAREIgAAAAAAAAAAAAAAAAAAQiIAAAAAAAAAAAAAAAAAAAIiIAAAAAAAAAAAAAAAAAAAAgAAAAAAAAAAD////////////////////////////////////////////////////////////////////////////////////////////////3////4////8H////g////8H////g////8H////g////8H////g////8H////h////8P////z//w==')",
};

const actionTypes = {
  FILL_SCREEN: "fillScreen",
  DRAW_LINE: "drawLine",
  DRAW_LINE_WITH_EDGES: "drawLineWithEdges",
  DRAW_FILLED_CIRCLE: "drawFilledCircle",
};

const history = [];

export default {
  components: { ControlPanel },
  data() {
    const pencil = {
      x: 0,
      y: 0,
      isDrawing: false,
      lineWidth: 10,
      radius: 5,
      currentColor: colors.BLACK,
      lastColor: colors.WHITE,
    };

    return {
      canvas: null,
      width: 560,
      height: 360,
      cursorsUrls,
      colors,
      opacity: 1,
      history,
      actionTypes,
      pencil,
    };
  },
  methods: {
    historyActions() {
      const self = this; //to allow use 'this' in arrow functions
      const actionTypes = this.actionTypes;

      return {
        [actionTypes.FILL_SCREEN](actionData) {
          self.fillWholeScreen(actionData.color, false);
        },
        [actionTypes.DRAW_LINE_WITH_EDGES](actionData) {
          const { width, color, from, to } = actionData;
          self.pencil.lineWidth = width;
          self.setColor(color);
          self.drawLineWithRoundEdges(from.x, from.y, to.x, to.y, false);
        },
        [actionTypes.DRAW_FILLED_CIRCLE](actionData) {
          const { width, color, from } = actionData;
          self.pencil.lineWidth = width;
          self.setColor(color);
          self.drawCircle(from.x, from.y);
        },
      };
    },
    undoLastAction() {
      if (this.history.length <= 1) return;

      const pencilData = this.pencil;

      this.history.pop();
      this.redrawAllHistory();

      this.pencil = pencilData;
    },
    redrawAllHistory() {
      this.history.forEach((action) => {
        this.historyActions()[action.type](action);
      });
    },
    fillWholeScreen(color, withHistory = true) {
      if (color) {
        this.setColor(color);
      }

      this.canvas.fillRect(0, 0, this.width, this.height);

      if (withHistory) {
        this.history.push({
          type: this.actionTypes.FILL_SCREEN,
          color: this.pencil.currentColor,
        });
      }

      if (color) {
        this.setColor(color);
      }
    },
    drawLine(x1, y1, x2, y2, withHistory = false) {
      this.canvas.beginPath();
      this.canvas.moveTo(x1, y1);
      this.canvas.lineTo(x2, y2);
      this.canvas.stroke();
      this.canvas.closePath();

      if (withHistory) {
        this.history.push({
          type: this.actionTypes.DRAW_LINE,
          color: this.pencil.currentColor,
          from: { x: x1, y: y1 },
          to: { x: x2, y: y2 },
          width: this.pencil.lineWidth,
        });
      }
    },
    drawLineWithRoundEdges(x1, y1, x2, y2, withHistory = true) {
      if (x1 === x2 && y1 === y2) {
        this.drawCircle(x1, y1, true);
        return;
      }

      this.drawCircle(x1, y1);
      this.drawLine(x1, y1, x2, y2);
      this.drawCircle(x2, y2);

      if (withHistory) {
        this.history.push({
          type: this.actionTypes.DRAW_LINE_WITH_EDGES,
          color: this.pencil.currentColor,
          from: { x: x1, y: y1 },
          to: { x: x2, y: y2 },
          width: this.pencil.lineWidth,
        });
      }
    },
    drawCircle(x, y, withHistory = false) {
      this.canvas.beginPath();
      this.canvas.arc(x, y, this.pencil.radius, 0, 2 * Math.PI);
      this.canvas.fill();
      this.canvas.closePath();

      if (withHistory) {
        this.history.push({
          type: this.actionTypes.DRAW_FILLED_CIRCLE,
          color: this.pencil.currentColor,
          from: { x, y },
          width: this.pencil.lineWidth,
        });
      }
    },
    clearCircle(x, y) {
      this.canvas.beginPath();
      this.canvas.arc(x, y, this.pencil.radius, 0, 2 * Math.PI, false);
      this.canvas.clip();
      this.canvas.clearRect(
        x - this.pencil.radius - 1,
        y - this.pencil.radius - 1,
        this.pencil.radius * 2 + 2,
        this.pencil.radius * 2 + 2
      );
      this.canvas.clip();
    },
    touchDraw(e) {
      const touch = e.touches[0] || e.changedTouches[0];
      const realTarget = this.canvas.canvas;
      e.offsetX = touch.clientX - realTarget.getBoundingClientRect().x;
      e.offsetY = touch.clientY - realTarget.getBoundingClientRect().y;

      this.draw(e);
    },
    draw(e) {
      if (this.pencil.isDrawing) {
        this.drawLineWithRoundEdges(
          this.pencil.x,
          this.pencil.y,
          e.offsetX,
          e.offsetY
        );

        this.pencil.x = e.offsetX || 0;
        this.pencil.y = e.offsetY || 0;
      }
    },
    beginDrawing(e) {
      this.pencil.x = e.offsetX;
      this.pencil.y = e.offsetY;
      this.pencil.isDrawing = true;
    },
    stopDrawing(e) {
      if (this.pencil.isDrawing) {
        this.drawLineWithRoundEdges(
          this.pencil.x,
          this.pencil.y,
          e.offsetX,
          e.offsetY
        );
        this.pencil.x = 0;
        this.pencil.y = 0;
        this.pencil.isDrawing = false;
      }
    },
    setColor(color) {
      this.pencil.lastColor = this.pencil.currentColor;
      this.pencil.currentColor = color;
      this.canvas.strokeStyle = this.canvas.fillStyle =
        this.pencil.currentColor;
    },
    saveImage() {
      const mimeType = "image/png";
      const imageURL = this.canvas.canvas.toDataURL(mimeType);
      const linkTag = document.createElement("a");

      linkTag.download = "demo_canvas_image";
      linkTag.href = imageURL;
      linkTag.dataset.downloadurl = [
        mimeType,
        linkTag.download,
        linkTag.href,
      ].join(":");

      document.body.appendChild(linkTag);
      linkTag.click();
      document.body.removeChild(linkTag);
    },
    setOpacity(newOpacity) {
      const opacity = newOpacity > 1 ? 1 : newOpacity < 0 ? 0 : newOpacity;
      this.opacity = opacity;
    },
    setPencilWidth(newPencilWidth) {
      const pencilWidth =
        newPencilWidth > 200 ? 200 : newPencilWidth < 1 ? 1 : newPencilWidth;
      this.pencil.lineWidth = pencilWidth;
    },
  },
  mounted() {
    const c = document.getElementById("myCanvas");
    this.canvas = c.getContext("2d");

    this.canvas.fillStyle = this.colors.WHITE;
    this.fillWholeScreen(this.colors.WHITE);

    this.setColor(this.colors.BLACK);
    this.canvas.lineWidth = this.pencil.lineWidth;
  },
  watch: {
    pencil: {
      deep: true,
      handler: function () {
        this.canvas.lineWidth = this.pencil.lineWidth;
        this.pencil.radius = Math.floor(this.pencil.lineWidth / 2);
      },
    },
  },
};
</script>

<style>
.wrapper {
  display: flex;
  flex-wrap: wrap;
}

#myCanvas {
  border: 1px solid grey;
}

.color-btn {
  display: inline-block;
  width: 20px;
  height: 20px;
  cursor: pointer;

  margin: 2px;
}

.color-btn.selected {
  -webkit-box-shadow: 0px 0px 2px 1px rgba(0, 0, 0, 1);
  -moz-box-shadow: 0px 0px 2px 1px rgba(0, 0, 0, 1);
  box-shadow: 0px 0px 2px 1px rgba(0, 0, 0, 1);
}
</style>