<script>
export default {
  created () {
    // 调用API从本地缓存中获取数据
    const logs = wx.getStorageSync('logs') || []
    logs.unshift(Date.now())
    wx.setStorageSync('logs', logs)

    console.log('app created and cache logs by setStorageSync')
  }
}
</script>

<style>
html,body {
  height:100%;
}
.container {
  width:100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  /*justify-content: center;*/
  /*padding: 200rpx 0;*/
  box-sizing: border-box;
  background-color: #faf8ee;
}
/* this rule will be remove */
* {
  transition: width 2s;
  -moz-transition: width 2s;
  -webkit-transition: width 2s;
  -o-transition: width 2s;
}
</style>
