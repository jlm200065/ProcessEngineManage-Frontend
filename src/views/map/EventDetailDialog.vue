<template>
  <el-dialog :title="title" :visible.sync="visible" @close="closeDialog">
    <div v-if="content">
      <div class="detail-item">
        <span class="label">日期:</span>
        <span class="value">{{ content.dateLabel }}</span>
      </div>
      <div class="detail-item">
        <span class="label">标题:</span>
        <span class="value">{{ content.title }}</span>
      </div>
      <div class="detail-item">
        <span class="label">描述:</span>
        <span class="value">{{ content.description }}</span>
      </div>
      <div v-if="content.imageList.length">
        <div v-for="image in content.imageList" :key="image.eventImage">
          <img :src="image.eventImagePath" :alt="image.des" class="event-image" />
          <div>{{ image.des }}</div>
        </div>
      </div>
      <div class="person-list">
        <span class="label">人物:</span>
        <div v-for="person in content.personList" :key="person.uri" class="person-item" @click="handlePersonClick(person.uri, person.label)">
          {{ person.label }}
        </div>
      </div>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button @click="closeDialog">关闭</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  props: {
    visible: {
      type: Boolean,
      required: true
    },
    title: {
      type: String,
      required: true
    },
    content: {
      type: Object,
      required: true
    }
  },
  methods: {
    closeDialog() {
      this.$emit('update:visible', false);
    },
    handlePersonClick(uri, label) {
      this.$emit('show-person-introduction', uri, label);
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

.event-image {
  width: 100%;
  max-width: 500px;
  margin: 10px 0;
}

.person-list {
  margin: 10px 0;
}

.person-item {
  display: inline-block;
  padding: 5px 10px;
  margin: 5px;
  border: 1px solid #007bff;
  color: #007bff;
  cursor: pointer;
  border-radius: 5px;
}

.person-item:hover {
  background-color: #007bff;
  color: #fff;
}
</style>
