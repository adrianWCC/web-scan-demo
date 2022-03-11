<template>
  <transition name="tip">
    <div class="tips" v-if="msg.show" :class="[msg.type]">{{msg.text}}</div>
  </transition>
  <div class="page">
    <div class="video-box">
      <video id="video" class="video" autoplay></video>
    </div>
    <h6>扫描结果：</h6>
    <p class="result-box">{{result}}</p>
    <div class="btn-box">
      <button @click="startScan">开始扫描</button>
      <button @click="close">关闭摄像头</button>
    </div>
    <div class="btn-box">
      <button @click="e => tips('success', 'success')">成功</button>
      <button @click="e => tips('error', 'error')">失败</button>
      <button @click="e => tips('warning', 'warning')">警告</button>
    </div>
  </div>
</template>

<script>
import { BrowserMultiFormatReader } from '@zxing/library';
export default {
  data() {
    return {
      reader: null,
      deviceId: null,
      result: '',
      msg: {
        show: false,
        type: 'success',
        text: ''
      },
      timer: null
    }
  },
  mounted() {
    this.reader = new BrowserMultiFormatReader()
    this.openCamera()
  },
  methods: {
    tips(type, text) {
      this.timer && clearTimeout(this.timer)
      this.msg = {
        show: true,
        type,
        text
      }
      this.timer = setTimeout(_ => {
        this.msg.show = false
      }, 1500)
    },
    close() {
      this.reader.reset()
    },
    startScan() {
      this.reader.reset()
      if(this.deviceId) {
        this.scan(this.deviceId)
      }
    },
    async openCamera() {
      const devices = await this.reader.listVideoInputDevices().catch(err => this.tips('error','interface error'))
      if(devices.length) {
        const deviceId = devices[0].deviceId
        devices.forEach(device => {
          if(device.label.indexOf('back') !== -1) {
            // if has backCamera use it
            deviceId = device.deviceId
            return
          }
        })
        this.deviceId = deviceId
      }else{
        // no cameras
        this.tips('warning', '未发现摄像头，请启用')
      }
    },
    async scan(deviceId) {
      const res = await this.reader.decodeOnceFromVideoDevice(deviceId, 'video').catch(err => this.tips('error', '扫码失败'))
      this.result = res && res.text
    }
  }
}
</script>

<style scoped>
.video-box{
  width: 100vw;
  height: 280px;
  margin: 0 auto;
  background-color: #000;
}
#video{
  width: 100vw;
  height: 280px;
}
.result-box{
  height: 200px;
  border: 1px solid #ddd;
}
.tips{
  height: 40px;
  line-height: 40px;
  width: 100vw;
  position: fixed;
}
.tips.success{background-color: #40c463;}
.tips.warning{background-color: #ffc501}
.tips.error{background-color: #fa4549}

.tip-enter-from, .tip-leave-to{
  transform: translateY(-40px);
}
.tip-enter-to, .tip-leave-from{
  transform: translateX(0);
}
.tip-enter-active, .tip-leave-active{
  transition: all .6s;
}
.btn-box{
  display: flex;
  margin: 10px 0;
  justify-content: space-around;
}
button{
  padding: 5px 10px;
  margin: 3px 5px;
}
</style>
