<template>
    <div class="ai">
        <div class="showbox" ref="showboxRef">
            <div v-for="msg in message" :key="msg.id"
                :class="{ 'from-user': msg.role === 'user', 'from-backend': msg.role === 'assistant' }">
                <div :class="{ 'user-message': msg.role === 'user', 'backend-message': msg.role === 'assistant' }">
                    <p v-if="msg.role === 'user'" v-text="msg.content"></p>
                    <div v-else v-html="msg.content"></div>
                </div>
            </div>
        </div>
        <div class="inputbox">
            <div class="ipt" @click="focusInput">
                <el-input ref="inputRef" class="eipt" v-model="input" style="width: 505px;height: 26px;
                  margin: 17px 20px ;
                  text-indent: 20px;
                  box-shadow: none;
                  border-radius: 40px;" placeholder="请输入你的想法" clearable />
                <button class="input_btn" @click="sendToBackend">发送</button>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, watch, nextTick } from 'vue';
const inputRef = ref()
const showboxRef = ref()
let input = ref('')

const message = ref([])
const sendToBackend = async() => {
    const inputcontent = {
        role: 'user',
        content: input.value,
    }
    message.value.push(inputcontent)
    input.value = ''
    kimiApi(message.value)
    
    await nextTick()
    showboxRef.value.scrollTop = showboxRef.value.scrollHeight
}
const apiKey = 'sk-GYYRZsaEK0gmFrtAXhUzj7iwRrdMxHPhFrYKg4Gv6gCtJ089'

const kimiApi = async (message) => {
  const index = message.length;
  const response = await fetch("https://api.moonshot.cn/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${apiKey}`,
    },
    body: JSON.stringify({
      model: "moonshot-v1-8k",
      messages: message,
      temperature: 0.3,
      stream: true,
    })
  });
  const reader = response.body.getReader();
  const textDecoder = new TextDecoder();
  message.push({
    role: 'assistant',
    content: ''
  });

  const updateMessage = (text) => {
    message[index].content += text;
    nextTick(() => {
      showboxRef.value.scrollTop = showboxRef.value.scrollHeight;
    });
  };

  while (true) {
    const { done, value } = await reader.read();
    if (done) {
      break;
    }
    const str = textDecoder.decode(value);
    str.split('\n').forEach(line => {
      if (line) {
        if (line === 'data: [DONE]') {
          return;
        }
        line = line.replaceAll('data: ', '');
        const data = JSON.parse(line);
        if (data.choices[0].finish_reason && data.choices[0].finish_reason === 'stop') {
          return;
        }
        if (data.choices[0].delta.content) {
          const text = data.choices[0].delta.content;
          
          updateMessage(text);
        }
      }
    });
  }
}
</script>


<style scoped>
.input_btn {
    color: white;
    border-radius: 50%;
    height: 50px;
    width: 50px;
    background-color: rgb(94, 89, 232);
    margin: 4px 12px;
    border: none;
    outline: none;
}

.ipt:deep(.el-input__inner) {
    border: none;
    box-shadow: none;
}

.ipt:deep(.el-input__inner:hover) {
    border: none;
    box-shadow: none;
}

.ipt:deep(.el-input__inner:focus) {
    border: none;
    box-shadow: none;
}

.ipt:deep(.el-input__wrapper) {
    border: none;
    box-shadow: none;
}

.ipt:deep(.el-input__wrapper:hover) {
    border: none;
    box-shadow: none;
}

.ipt:deep(.el-input__wrapper:focus) {
    border: none;
    box-shadow: none;
}

.ipt {
    display: flex;
    align-self: center;
    width: 90%;
    height: 60px;
    background-color: white;
    border-radius: 40px;
}

.inputbox {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 120px;
    border-radius: 12px;
}

.showbox {
    width: 100%;
    /* 固定宽度 */
    height: 495px;
    overflow-y: auto;
    padding: 10px;
    background-color: rgb(247, 248, 252);
    border-radius: 12px;
}

.from-user .user-message,
.from-backend .backend-message {
    display: block;
    width: 90%;
    padding: 20px;
    margin: 0 auto;
    border-radius: 8px;
    word-wrap: break-word;
    /* 自动换行 */
    white-space: pre-wrap;
    /* 保留空白符和换行符 */
    color: blue;
    margin-bottom: 10px;
    background-color: rgb(224, 223, 255);
}

.from-backend .backend-message {
    color: green;
    background-color: white;
}

.ai {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: 20px; /* 添加内边距 */
    background-color: rgb(247, 248, 252);
    border-radius: 12px;
    overflow: hidden; /* 防止内容溢出 */
}
</style>