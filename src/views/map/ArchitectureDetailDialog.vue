<template>
  <el-dialog :title="detailDialogTitle" :visible.sync="detailDialogVisible" @close="closeDetailDialog">
    <div v-if="detailContent">
      <div class="detail-item">
        <span class="label">建筑名称:</span>
        <span class="value">{{ detailContent.nameS }}</span>
      </div>
      <div class="detail-item">
        <span class="label">地址:</span>
        <span class="value">{{ detailContent.address }}</span>
      </div>
      <div class="detail-item">
        <span class="label">类型:</span>
        <span class="value">{{ detailContent.type }}</span>
      </div>
      <div class="detail-item">
        <span class="label">描述:</span>
        <span class="value">{{ currentItem.des }}</span>
      </div>
      <div class="detail-item">
        <span class="label">地区:</span>
        <span class="value">{{ currentItem.placeValue }}</span>
      </div>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button @click="closeDetailDialog">关闭</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  props: {
    detailDialogTitle: String,
    detailDialogVisible: Boolean,
    detailContent: Object,
    currentItem: Object
  },
  methods: {
    closeDetailDialog() {
      this.$emit('update:detailDialogVisible', false);
    }
  }
};
</script>

<style scoped>
.detail-item {
  margin: 10px 0;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.label {
  font-weight: bold;
  color: #333;
}

.value {
  color: #555;
}
</style>
