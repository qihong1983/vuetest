<template>
  <div id="app">
    <!-- <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div>-->

    <van-row>
      <van-col span="24">
        <router-view />
      </van-col>
    </van-row>
  </div>
</template>
<script>
import { Col, Row } from "vant";
export default {
  name: "HelloWorld",

  mounted() {},
  components: {
    [Col.name]: Col,
    [Row.name]: Row
  },
  methods: {},
  props: {
    msg: String
  }
};
</script>
<style lang="less">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  // text-align: center;
  color: #2c3e50;
  width: 100vw;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
