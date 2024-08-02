<template>
  <el-dialog :visible.sync="dialogVisible" width="50%" @close="closeDialog">
    <div class="chat-container">
      <div class="messages-list">
        <div v-for="(message, index) in messages" :key="index" :class="messageClass(message.sender)">
          {{ message.content }}
        </div>
        <div v-if="isTyping" class="typing-indicator">对方正在输入...</div>
      </div>
      <div class="input-container">
        <el-input
          type="textarea"
          rows="2"
          placeholder="Type your message..."
          v-model="userMessage"
          @keypress.enter.native.prevent="sendMessage"
        ></el-input>
        <el-button type="primary" @click="sendMessage">发送</el-button>
      </div>
    </div>
  </el-dialog>
</template>

<script>
import axios from 'axios';

export default {
  props: {
    visible: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      dialogVisible: this.visible,
      userMessage: '',
      messages: [],
      isTyping: false
    };
  },
  watch: {
    visible(val) {
      this.dialogVisible = val;
    },
    dialogVisible(val) {
      this.$emit('update:visible', val);
    }
  },
  methods: {
    closeDialog() {
      this.dialogVisible = false;
    },
    sendMessage() {
      if (this.userMessage.trim()) {
        const message = {
          sender: 'user',
          content: this.userMessage
        };
        this.messages.push(message);
        this.userMessage = '';
        this.scrollToBottom();
        this.sendToApi(message.content);
      }
    },
    async sendToApi(prompt) {
      this.isTyping = true;
      try {
        const response = await axios.post('http://localhost:8090/api/rag', {
          appId: "404bf78d45714e4a818dce493509a4a8",
          prompt: prompt
        });
        const aiMessage = {
          sender: 'ai',
          content: response.data.data
        };
        this.messages.push(aiMessage);
      } catch (error) {
        console.error(error);
      } finally {
        this.isTyping = false;
        this.scrollToBottom();
      }
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const container = this.$el.querySelector('.messages-list');
        container.scrollTop = container.scrollHeight;
      });
    },
    messageClass(sender) {
      return sender === 'user' ? 'user-message' : 'ai-message';
    }
  }
};
</script>

<style scoped>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 400px;
}

.messages-list {
  flex-grow: 1;
  overflow-y: auto;
  margin-bottom: 10px;
  padding: 10px;
  background-color: #f5f5f5;
  display: flex;
  flex-direction: column;
}

.input-container {
  display: flex;
  align-items: center;
}

.el-input {
  flex-grow: 1;
}

.user-message {
  align-self: flex-end;
  background-color: #1890ff;
  color: #fff;
  padding: 10px;
  border-radius: 5px;
  margin: 5px 0;
  max-width: 60%;
  word-break: break-word;
}

.ai-message {
  align-self: flex-start;
  background-color: #e5e5e5;
  color: #000;
  padding: 10px;
  border-radius: 5px;
  margin: 5px 0;
  max-width: 60%;
  word-break: break-word;
}

.typing-indicator {
  align-self: flex-start;
  color: #888;
  font-style: italic;
  margin: 5px 0;
}
</style>
