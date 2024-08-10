<template>
  <transition name="fade">
    <div v-if="visible" class="region-display">
      <div class="region-name">{{ regionName }}</div>
    </div>
  </transition>
</template>

<script>
export default {
  props: {
    regionName: {
      type: String,
      required: true
    },
    visible: {
      type: Boolean,
      default: false
    }
  }
};
</script>

<style scoped>
.region-display {
  position: fixed;
  top: 40%;
  right: 0%;
  transform: translateY(-50%);
  width: 400px;
  height: 350px;
  background-color: rgba(0, 0, 0, 0.5);
  padding: 10px;
  border-radius: 5px;
  color: white;
  font-size: 14px;
  z-index: 2000;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
}

.region-name {
  font-size: 18px;
  margin-bottom: 10px;
}

/* 添加淡入淡出的动画效果 */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0;
}
</style>

