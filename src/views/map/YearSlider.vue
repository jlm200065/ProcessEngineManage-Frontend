<template>
  <div class="year-slider-container">
    <div class="year-slider" ref="slider" @click="onSliderClick($event)">
      <div
        class="year-marker"
        v-for="year in years"
        :key="year"
        :style="{ left: `${(year - 1800) * stepWidth}px` }"
      >
        <span v-if="year % 10 === 0">{{ year }}</span>
        <span v-else class="tooltip">{{ year }}</span>
      </div>
      <div
        class="year-ball"
        :style="{ left: `${(currentYear - 1800) * stepWidth}px` }"
        @mousedown.stop="startDrag($event)"
      >
        <div class="ball-tooltip">{{ currentYear }}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    value: Number // 接收父组件传递的年份
  },
  data() {
    return {
      currentYear: this.value || 1800, // 初始化为父组件传递的年份
      isDragging: false,
      years: Array.from({ length: 2024 - 1800 + 1 }, (_, i) => 1800 + i),
      stepWidth: 40, // 每年的宽度，可以根据需要调整
    };
  },
  watch: {
    value(newYear) {
      this.currentYear = newYear;
    }
  },
  methods: {
    startDrag(event) {
      this.isDragging = true;
      document.addEventListener('mousemove', this.onDrag);
      document.addEventListener('mouseup', this.stopDrag);
    },
    onDrag(event) {
      if (this.isDragging) {
        const newYear = this.getYearFromPosition(event.clientX);
        this.currentYear = newYear;
        this.$emit('input', newYear); // 通知父组件年份变化
      }
    },
    stopDrag() {
      this.isDragging = false;
      document.removeEventListener('mousemove', this.onDrag);
      document.removeEventListener('mouseup', this.stopDrag);
    },
    getYearFromPosition(position) {
      const sliderRect = this.$refs.slider.getBoundingClientRect();
      const relativePosition = position - sliderRect.left;
      let year = Math.round(relativePosition / this.stepWidth) + 1800;
      year = Math.max(1800, Math.min(2024, year));
      return year;
    },
    onSliderClick(event) {
      const newYear = this.getYearFromPosition(event.clientX);
      this.currentYear = newYear;
      this.$emit('input', newYear); // 通知父组件年份变化
    }
  }
};
</script>

<style scoped>
.year-slider-container {
  position: fixed;
  top: 10px;
  right: 10px;
  width: 1200px; /* 宽度可以根据需要调整 */
  height: 50px;
  background-color: rgba(0, 0, 0, 0.5);
  border-radius: 5px;
  padding: 10px;
  display: flex;
  align-items: center;
  overflow-x: scroll;
  overflow-y: hidden;
}

.year-slider {
  position: relative;
  width: 9000px; /* 调整宽度以适应所有年份 */
  height: 30px;
  background-color: #ccc;
  border-radius: 10px;
  cursor: pointer;
}

.year-marker {
  position: absolute;
  top: 0;
  width: 40px; /* 增加宽度以显示完整年份 */
  height: 20px;
  background-color: #007bff;
  color: #fff;
  text-align: center;
  line-height: 20px;
}

.year-marker .tooltip {
  display: none;
}

.year-marker:hover .tooltip {
  display: block;
  position: absolute;
  top: -25px;
  left: 50%;
  transform: translateX(-50%);
  background-color: #000;
  color: #fff;
  padding: 5px;
  border-radius: 3px;
  white-space: nowrap;
}

.year-ball {
  position: absolute;
  top: 0px; /* 调整小球的位置 */
  width: 20px;
  height: 20px;
  background-color: #ff5722;
  border-radius: 50%;
  cursor: pointer;
}

.year-ball .ball-tooltip {
  position: absolute;
  top: 20px; /* 调整小球提示的位置 */
  left: 50%;
  transform: translateX(-50%);
  background-color: #000;
  color: #fff;
  padding: 5px;
  border-radius: 3px;
  white-space: nowrap;
}
</style>
