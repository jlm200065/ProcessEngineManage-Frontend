<template>
  <el-dialog :title="dialogTitle" :visible.sync="dialogVisible" @close="closeDialog">
    <div>
      <p>著名建筑有：</p>
      <ul>
        <li v-for="item in dialogContent" :key="item.uri">
          <el-button class="item-button" type="text" @click="showDetail(item.uri, item)">{{ item.nameS }}</el-button>
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
</style>
