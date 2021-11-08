<template>
  <div>
    <div>
      <el-button type="success" class="button-wrapper" :disabled="isConnected" @click="connect">Connect</el-button>
      <el-button type="success" class="button-wrapper" :disabled="!isConnected" @click="disconnect">Disonnect</el-button>
    </div>
    <div>
      <el-input class="input-wrapper" :disabled="!isConnected||isJoined" v-model="room" placeholder="Please input the room name." />
      <el-input class="input-wrapper" :disabled="!isConnected||isJoined" v-model="name" placeholder="Please input your name." />
      <el-button @click="join" :disabled="!isConnected||isJoined">Join</el-button>
      <el-button @click="leave" :disabled="!isConnected||!isJoined">Leave</el-button>
    </div>
    <div class="message-panel">
      <msgBox v-for="(item, index) of msgList" :key="index+Math.random()" :uname="item.name" :content="item.msg" :isself="item.isSelf"></msgBox>
    </div>
    <div>
      <el-input :disabled="!isConnected||!isJoined" class="content" v-model="content" placeholder="Type something : " />
      <el-button :disabled="!isConnected||!isJoined" @click="send" type="primary">Send</el-button>
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
      isConnected: false,
      isJoined: false,
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
      this.websocket = new WebSocket(wsurl);
      this.websocket.onopen = this.onWsOpen;
      this.websocket.onerror = this.onWsError;
      this.websocket.onmessage = this.onWsMessage;
      this.websocket.onclose = this.onWsClose;
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
      if (Array.isArray(this.msgList) && this.msgList.length){
        this.msgList = [];
      }
      if (this.name && this.room){
        this.isJoined = true;

        const joinMessage = {
          ClientId: "",
          Action: "Join",
          RoomId: this.room,
          NickName: this.name,
          Content: ""
        }
        this.sendMessageToServer(joinMessage);
      }else{
        alert("Should fill the both room and name info.");
      }
    },
    leave(){
      // unlock the name and room input
      this.msgList = [];
      this.isJoined = false;
    },
    send(){
      // send the message to server
      if (this.content){
        this.msgList.push({name:this.name, msg:this.content, isSelf:true});
        this.content = "";
      }else{
        alert("You can't send empty content.");
      }
    },
    sendMessageToServer(message){
      console.log(JSON.stringify(message));
      this.websocket.send(JSON.stringify(message));
    },
    onWsOpen(){
      alert("Success");
      this.isConnected = true;
    },
    onWsMessage(e){
      const resdata = JSON.parse(e.data);
      console.log(resdata);
    },
    onWsError(){
      alert("Connect failed. Please check the websocket server.");
    },
    onWsClose(){
      alert("Disconnected.");
      // init the data
      this.isConnected = false;
      this.isJoined = false;
      this.name = "";
      this.room = "";
      this.content = "";
      this.clientId = "";
    }
  }
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
