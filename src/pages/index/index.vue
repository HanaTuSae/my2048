<template>
  <div class="container">
    <image class='logo' src='../../static/logo.png'></image>
    <progress class="progress" :percent="progressValue" active stroke-width="3" activeColor=''/>
  </div>
</template>

<script>
export default {
  data () {
    return {
      progressValue: 100
    }
  },

  mounted () {
    let time0 = setTimeout(() => {
      clearTimeout(time0)
      wx.redirectTo({
        url: '../2048/main'
      })
    }, 3500)
  },

  methods: {
  },

  created () {

  }
}
</script>

<style scoped>
.container {
  background-color: #faf8ee;
}
.logo {
  width:100px;
  height:100px;
  border-radius: 20px;
  margin-top: 35%;
}
.progress {
  width:90%;
  margin-top: 10px;
}
</style>
