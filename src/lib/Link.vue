<template>
  <svg>
    <path
      v-if="!link.pattern"
      :d="
        `M${source.point.x + source.width / 2} ${source.point.y +
          source.height / 2} Q ${point.x} ${point.y} ${destination.point.x +
          destination.width / 2} ${destination.point.y +
          destination.height / 2}`
      "
      :stroke="link.color || '#ffeaa7'"
      stroke-width="3"
      fill="none"
    />
    <path
      v-if="link.pattern === 'solid'"
      :d="
        `M${source.point.x + source.width / 2} ${source.point.y +
          source.height / 2} Q ${point.x} ${point.y} ${destination.point.x +
          destination.width / 2} ${destination.point.y +
          destination.height / 2}`
      "
      :stroke="link.color || '#ffeaa7'"
      stroke-width="3"
      fill="none"
    />
    <path
      v-if="link.pattern === 'dash'"
      :d="
        `M${source.point.x + source.width / 2} ${source.point.y +
          source.height / 2} Q ${point.x} ${point.y} ${destination.point.x +
          destination.width / 2} ${destination.point.y +
          destination.height / 2}`
      "
      :stroke="link.color || '#ffeaa7'"
      stroke-width="3"
      stroke-dasharray="10"
      fill="none"
    />
    <path
      v-if="link.pattern === 'dot'"
      :d="
        `M${source.point.x + source.width / 2} ${source.point.y +
          source.height / 2} Q ${point.x} ${point.y} ${destination.point.x +
          destination.width / 2} ${destination.point.y +
          destination.height / 2}`
      "
      :stroke="link.color || '#ffeaa7'"
      stroke-width="3"
      fill="none"
      stroke-dasharray="2"
    />
    <g v-if="editable">
      <line
        :x1="source.point.x + source.width / 2"
        :y1="source.point.y + source.height / 2"
        :x2="point.x"
        :y2="point.y"
        stroke="lightgray"
      />
      <line
        :x1="point.x"
        :y1="point.y"
        :x2="destination.point.x + destination.width / 2"
        :y2="destination.point.y + destination.height / 2"
        stroke="lightgray"
      />
      <ellipse
        :id="id"
        :cx="point.x"
        :cy="point.y"
        rx="10"
        ry="10"
        fill="#ff7675"
        stroke-width="2"
        class="grab"
        @click="select"
        @mousedown="mousedown"
        @touchstart="mousedown"
        @mousemove="mousemove"
        @touchmove="mousemove"
        @mouseup="mouseup"
        @touchend="mouseup"
      />
    </g>
    <g>
      <text
        :x="point.x - 15"
        :y="point.y - 20"
        fill="#00b894"
        @click="edit"
        v-if="selected"
        class="button"
      >
        {{ labels.edit || "Edit" }}
      </text>
      <text
        :x="point.x - 15"
        :y="point.y + 30"
        fill="#ff7675"
        @click="remove"
        v-if="selected"
        class="button"
      >
        {{ labels.remove || "Remove" }}
      </text>
    </g>
  </svg>
</template>
<script>
import mouseLocation from "../mouseLocation";
export default {
  mixins: [mouseLocation],
  props: {
    selected: Boolean,
    editable: Boolean,
    source: {
      id: Number,
      x: Number,
      y: Number
    },
    destination: {
      id: Number,
      x: Number,
      y: Number
    },
    link: {
      id: String,
      color: String,
      pattern: {
        type: String,
        default: "solid"
      },
      point: {
        x: Number,
        y: Number
      }
    },
    labels: Object,
    scale: String
  },
  data() {
    return {
      startPosition: null,
      cursorOffset: {
        x: 0,
        y: 0
      },
      id: this.link.id,
      point: this.link.point
    };
  },
  methods: {
    mousedown(e) {
      const [x, y] = this.getLocation(e);
      this.cursorOffset.x = x;
      this.cursorOffset.y = y;
      this.startPosition = { x: this.point.x, y: this.point.y };
      document.addEventListener("mousemove", this.mousemove);
      document.addEventListener("mouseup", this.mouseup);
    },
    mousemove(e) {
      if (this.startPosition) {
        e.preventDefault();
        const [x, y] = this.getLocation(e);
        this.point.x =
          this.startPosition.x +
          (x - this.cursorOffset.x) / parseFloat(this.scale);
        this.point.y =
          this.startPosition.y +
          (y - this.cursorOffset.y) / parseFloat(this.scale);
        this.$emit("updateLocation", {
          id: this.id,
          x: this.point.x,
          y: this.point.y
        });
      }
    },
    mouseup() {
      this.startPosition = null;
      document.removeEventListener("mousemove", this.mousemove);
      document.removeEventListener("mouseup", this.mouseup);
    },
    remove() {
      this.$emit("remove", this.id);
    },
    select() {
      this.$emit("select", this.id);
    },
    edit() {
      this.$emit("editLink", {
        id: this.link.id,
        content: {
          color: this.link.color
        }
      });
    }
  }
};
</script>
<style scoped></style>
