<template>
  <el-dialog :title="dialogTitle" :visible.sync="dialogVisible" @close="closeDialog">
    <div>
      <p>著名建筑有：</p>
      <ul>
        <li v-for="item in dialogContent" :key="item.uri" style="display: flex; align-items: center;">
          <el-button class="item-button" type="text" @click="showDetail(item.uri, item)">{{ item.nameS }}</el-button>
          <el-button
            class="gossip-button"
            type="text"
            icon="el-icon-plus"
            size="mini"
            @click="addToGossip('location', item.nameS)"
            style="margin-left: 10px;"
          >
            加入噶讪胡
          </el-button>
        </li>
      </ul>
      <p>相关事件：</p>
      <ul>
        <li v-for="event in eventList" :key="event.uri">
          <el-button
            class="item-button"
            type="text"
            @click="showEventDetail(event.uri)"
            style="position: relative; z-index: 10;"
          >
            {{ event.title }} ({{ event.begin }} - {{ event.end }})
          </el-button>
        </li>
      </ul>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button @click="closeDialog">关闭</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  props: {
    dialogTitle: String,
    dialogVisible: Boolean,
    dialogContent: Array,
    eventList: Array
  },
  methods: {
    closeDialog() {
      this.$emit('update:dialogVisible', false);
    },
    showDetail(uri, item) {
      this.$emit('show-detail', uri, item);
    },
    showEventDetail(uri) {
      this.$emit('show-event-detail', uri);
    },
    addToGossip(field, value) {
      this.$emit('add-to-gossip', { field, value });
    }
  }
};
</script>

<style scoped>
.item-button {
  border: 1px solid #007bff;
  color: #007bff;
  padding: 5px 10px;
  margin: 5px 0;
  display: inline-block;
  cursor: pointer;
  position: relative;
  z-index: 10; /* 确保按钮位于上层 */
}

.item-button:hover {
  background-color: #007bff;
  color: #fff;
}

.gossip-button {
  margin-left: 10px;
  font-size: 12px; /* 更小的字体 */
  background-color: rgba(70, 130, 180, 0.3); /* 淡蓝色半透明背景 */
  color: #4682b4; /* 淡蓝色字体 */
  padding: 3px 8px; /* 调整按钮大小 */
  border: none;
  border-radius: 3px; /* 添加圆角 */
  cursor: pointer;
  transition: background-color 0.3s ease, color 0.3s ease; /* 添加过渡效果 */
}

.gossip-button:hover {
  background-color: rgba(70, 130, 180, 0.6); /* 鼠标悬停时背景颜色变深 */
  color: #ffffff; /* 鼠标悬停时字体变为白色 */
}
</style>
