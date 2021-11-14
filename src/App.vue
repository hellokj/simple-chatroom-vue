<template>
  <div>
    <div>
      <el-button
        type="success"
        class="button-wrapper"
        :disabled="isConnected"
        @click="connect"
        >Connect</el-button
      >
      <el-button
        type="success"
        class="button-wrapper"
        :disabled="!isConnected"
        @click="disconnect"
        >Disonnect</el-button
      >
    </div>
    <div>
      <el-input
        class="input-wrapper"
        :disabled="!isConnected || isJoined"
        v-model="room"
        placeholder="room name"
      />
      <el-input
        class="input-wrapper"
        :disabled="!isConnected || isJoined"
        v-model="name"
        placeholder="your name"
      />
      <el-button @click="join" :disabled="!isConnected || isJoined"
        >Join</el-button
      >
      <el-button @click="leave" :disabled="!isConnected || !isJoined"
        >Leave</el-button
      >
    </div>
    <div class="message-panel">
      <msgBox
        v-for="(item, index) of msgList"
        :key="index + Math.random()"
        :uname="item.name"
        :content="item.msg"
        :isself="item.isSelf"
        :isFromSystem="item.isFromSystem"
      ></msgBox>
    </div>
    <div>
      <el-input
        :disabled="!isConnected || !isJoined"
        class="content"
        v-model="content"
        placeholder="Type something : "
      />
      <el-button
        :disabled="!isConnected || !isJoined"
        @click="send"
        type="primary"
        >Send</el-button
      >
    </div>
  </div>
</template>

<script>
import config from "../public/config"
import msgBox from "../src/components/MsgBox.vue";
export default {
  name: "App",
  data() {
    return {
      websocket: null,
      msgList: [],
      room: "",
      name: "",
      isConnected: false,
      isJoined: false,
      clientId: "",
      content: ""
    };
  },
  components: {
    // HelloWorld
    msgBox,
  },
  methods: {
    connect() {
      const wsurl = config.WS_SERVER;
      this.websocket = new WebSocket(wsurl);
      this.websocket.onopen = this.onWsOpen;
      this.websocket.onerror = this.onWsError;
      this.websocket.onmessage = this.onWsMessage;
      this.websocket.onclose = this.onWsClose;
    },
    disconnect() {
      if (this.websocket) {
        // before disconnect, it should be leave the room first.
        if (this.isJoined){
          console.log(this.isJoined)
          this.leave();
        }
        this.websocket.close();
        this.websocket = null;
        this.isConnected = false;
        this.isJoined = false;
      }
    },
    join() {
      // check the name and room content
      // lock the name and room input
      if (Array.isArray(this.msgList) && this.msgList.length) {
        this.msgList = [];
      }
      if (this.name && this.room) {
        this.isJoined = true;

        const joinMessage = {
          ClientId: this.clientId,
          Action: "Join",
          RoomId: this.room,
          NickName: this.name,
          Content: this.content,
        };
        this.sendMessageToServer(joinMessage);
      } else {
        alert("Should fill the both room and name info.");
      }
    },
    leave() {
      const leaveMessage = {
        ClientId: this.clientId,
        Action: "Leave",
        RoomId: this.room,
        NickName: this.name,
        Content: this.content,
      };
      this.sendMessageToServer(leaveMessage);
      this.isJoined = false;
    },
    send() {
      // send the message to server
      if (this.content) {
        const broadcastMessage = {
          ClientId: this.clientId,
          Action: "Broadcast",
          RoomId: this.room,
          NickName: this.name,
          Content: this.content,
        };
        this.sendMessageToServer(broadcastMessage);
        this.content = "";
      } else {
        alert("You can't send empty content.");
      }
    },
    sendMessageToServer(message) {
      console.log(JSON.stringify(message));
      this.websocket.send(JSON.stringify(message));
    },
    onWsOpen() {
      alert("Success");
      this.isConnected = true;
    },
    onWsMessage(e) {
      const resdata = JSON.parse(e.data);
      console.log(resdata);
      this.clientId = resdata.ClientId;
      this.msgList.push({
        name: resdata.From,
        msg: resdata.Content,
        isFromSystem: resdata.IsFromSystem,
        isSelf: resdata.FromId == this.clientId,
      });
    },
    onWsError() {
      alert("Connect failed. Please check the websocket server.");
    },
    onWsClose() {
      alert("Disconnected.");
      // init the data
      this.isConnected = false;
      this.isJoined = false;
      this.name = "";
      this.room = "";
      this.content = "";
      this.clientId = "";
    },
  },
};
</script>

<style scoped>
.button-wrapper {
  margin: 1%;
}

.input-wrapper {
  width: 20%;
  margin: 1%;
}

.message-panel::-webkit-scrollbar {
  display: none;
}

.message-panel {
  width: 90%;
  height: 550px;
  border: 1px #572d2d solid;
  overflow: scroll;
  overflow-x: hidden;
  scrollbar-width: none;
  padding: 10px;
  margin: 10px;
}

.content {
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
