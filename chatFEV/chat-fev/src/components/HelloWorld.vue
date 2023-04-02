<script setup>
import { ref, reactive } from "vue";

defineProps({
  msg: String,
});

const msg = reactive([]);
const input = ref("");
const img = ref("");

function getImg(text) {
  fetch('https://api.openai.com/v1/images/generations', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer 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'
    },
    body: JSON.stringify({
      prompt: text,
      n: 2,
      size: "1024x1024",
    })
  })
  .then(response => response.json())
  .then(data => {
    img.value = data.data[0].url;
  })
  .catch(error => console.error(error))
};

function showMsg() {
  msg.push(input.value);
  getChat(msg.join("\n"));
};

function showImg() {
  getImg(input.value)
}

function noThing() {

}

function resultFilter(text) {
  console.log(text);
};

async function handleBlob(typedBuffer) {
  const str = await new Response(typedBuffer, { headers: { 'Content-Type': 'text/html;charset=utf-8' } }).text();
  return str;
}

function handleRespon(text) {
  const start = text.indexOf("{");
  const end = text.lastIndexOf("}");
  console.log(JSON.parse(text.slice(start, Number(end)-1)+']}'));
};

function getChat(text) {
fetch('https://api.openai.com/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer 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'
  },
  body: JSON.stringify({
    messages: [{role: "user", content: text}],
    model: "gpt-3.5-turbo",
    temperature: 0,
    top_p: 1,
    stream: true,
    frequency_penalty: 0.0,
    presence_penalty: 0.0,
    stop: ["\n"],
  })
})
  .then(response => {
    return response.body
  })
  .then(async (rb) => {
    const reader = rb.getReader();
    return new ReadableStream({
      start(controller) {
        // The following function handles each data chunk
        function push() {
          // "done" is a Boolean and value a "Uint8Array"
          reader.read().then(async ({ done, value }) => {
            // If there is no more data to read
            if (done) {
              controller.close();
              return;
            }
            // Get the data and send it to the browser via the controller
            controller.enqueue(value);
            const text = await handleBlob(value);
            console.log(handleRespon(text));
            push();
          });
        }
        push();
      },
    });
  })
  // .then((stream) =>
  //   // Respond with our stream
  //   new Response(stream, { headers: { 'Content-Type': 'text/html;charset=utf-8' } })
  // )
  // .then((result) => {
  //   console.log(resultFilter(result));
  // });
}
</script>

<template>
  <img :src="img" alt="">
  <div class="container2" v-infinite-scroll="noThing">
    <h3 class="talk" v-for="item in msg">{{ item }}</h3>
    <el-row class="input">
      <el-col :span="12">
        <el-input v-model="input"></el-input>
      </el-col>
      <el-col :span="12">
        <el-button @click="showMsg">button</el-button>
        <el-button @click="showImg">getImg</el-button>
      </el-col>
    </el-row>
  </div>
</template>

<style scoped>

.container2 {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  height: 100vh;
}
.talk {
  flex: 0.8
}
.input {
  flex: 0.2
}
</style>
