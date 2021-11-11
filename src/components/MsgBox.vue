<template>
  <div class="msg-wrapper" v-bind:class="getClass()">
    <span v-if="!isFromSystem && !isself" class="uname">{{ uname }}</span>
    <span v-if="!isFromSystem && !isself">：</span>
    <div class="content">{{ content }}</div>
  </div>
</template>

<script>
export default {
  name: "msgBox",
  props: {
    isFromSystem: Boolean,
    uname: String,
    content: String,
    isself: Boolean,
  },
  methods: {
    getClass() {
      if (this.isFromSystem) return "system";
      if (this.isself) return "self";
      return "";
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.msg-wrapper {
  display: flex;
  width: 100%;
  flex-direction: row;
  padding: 10px;
}
.content {
  border-radius: 2px;
  margin-left: 10px;
  margin-right: 10px;
}
.self {
  flex-direction: row-reverse;
}

.system {
  color: gray;
  justify-content: center;
}
</style>    