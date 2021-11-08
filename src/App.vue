<template>
  <div>
    <div>
      <el-button type="success" class="button-wrapper" :disabled="websocket" @click="connect">Connect</el-button>
      <el-button type="success" class="button-wrapper" :disabled="!websocket" @click="disconnect">Disonnect</el-button>
    </div>
    <div>
      <el-input class="input-wrapper" v-model="room" placeholder="Please input the room name." />
      <el-input class="input-wrapper" v-model="name" placeholder="Please input your name." />
      <el-button @click="join">Join</el-button>
      <el-button @click="leave">Leave</el-button>
    </div>
    <div class="message-panel">
      <msgBox v-for="(item, index) of msgList" :key="index+Math.random()" :uname="item.name" :content="item.msg" :isself="item.isSelf"></msgBox>
    </div>
    <div>
      <el-input class="content" v-model="content" placeholder="Type something : " />
      <el-button @click="send" type="primary">Send</el-button>
    </div>
  </div>
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import msgBox from '../src/components/MsgBox.vue';
export default {
  name: 'App',
  data(){
    return{
      websocket: null,
      msgList: [],
      room: "",
      name: "",
      clientId: "",
      content: ""
    }
  },
  components: {
    // HelloWorld
    msgBox
  },
  methods:{
    connect(){
      const wsurl = 'ws://localhost:8080/websocket';
      console.log(wsurl)
      this.websocket = new WebSocket(wsurl, "http");
      this.websocket.onopen = this.success;
      this.websocket.onerror = this.success;
      this.websocket.onmessage = this.success;
      this.websocket.onclose = this.success;
    },
    disconnect(){
      if (this.websocket){
        this.websocket.close();
        this.websocket = null;
      }
    },
    join(){
      // check the name and room content
      // lock the name and room input

    },
    leave(){
      // unlock the name and room input
      this.msgList = []
    },
    send(){
      this.msgList.push({name:"kj", msg:"aaa", isSelf:true})
    },
    success(){
      alert("success");
    }
  },
}
</script>

<style scoped>
  .button-wrapper{
    margin: 1%;
  }

  .input-wrapper{
    width: 20%;
    margin: 1%;
  }

  .message-panel{
    width: 90%;
    height: 550px;
    border: 1px #572d2d solid;
    overflow: scroll;
    overflow-x: hidden;
    padding: 10px;
    margin: 1%;
  }

  .content{
    margin: 1%;
    width: 90%;
  }

  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
