<template>
  <video ref="remoteVideo" data-v-app="" style="object-fit:cover" webkit-playsinline="true" playsinline="true" width="960" height="540"></video>
  <div>
    <div>
      <button @click="initRemote">获取推流</button>
      <button @click="releaseHive">释放推流</button>
    </div>
    <div>
      <input type="text" placeholder="请输入文字 按回车发送" autocomplete="off" v-model="question">
      <button @click="broadcast">播报</button>
      <button @click="answer">回答</button>
      <button @click="camera">相机角度</button>
    </div>
    <div>
      <button @click="doRecOpen">打开录音权限</button>
      <button @click="doRecStart">录制</button>
      <button @click="doRecStop">停止</button>
      <button @click="doRecClose">释放录音资源</button>
    </div>
  </div>
  <!-- 扣除背景展示容器 -->
  <div>
    <!-- 视频截图 -->
    <canvas ref="c1" width="960" height="540" style="display:none"></canvas>
    <!-- 处理绿色像素为透明 -->
    <canvas ref="c2" width="960" height="540"></canvas>
  </div>
</template>

<script>
import * as Remote from 'motion-remote'
import Recorder from 'recorder-core/recorder.wav.min'

export default {
  name: 'App',
  data() {
    return {
      question: '',
      rec: null
    }
  },
  methods: {
    initRemote() {
      Remote.initRemote({
        video: this.$refs.remoteVideo, // 视频容器
        serverUserId: 15001191540, // userId
        password: 15001191540 // password
        // roleIndex: 1, // 角色编号
        // curActionStyle: 1, // 动作编号
        // screenMode: 1, // 推流视频横竖屏 0-横屏 1-竖屏
        // dpiWidth: 1080, // 推流视频宽
        // dpiHeight: 1920, // 推流视频高
        // backgroundIndex: 1, // 推流视频背景 0:默认 1:自定义背景
        // cutBackground: {
        //   video: this.$refs.remoteVideo, // 视频标签
        //   c1: this.$refs.c1, // 获取视频截图
        //   c2: this.$refs.c2, // 处理绿色背景
        //   scale: 1 // 视频尺寸与输出尺寸的比例
        // }
      })
    },
    broadcast() {
      Remote.getBroadcast(this.question)
    },
    answer() {
      Remote.getAnswer(this.question)
    },
    camera() {
      Remote.switchCamera()
    },
    releaseHive() {
      Remote.releaseHive()
    },
    doRecOpen() {
      this.rec = null
      let newRec = Recorder({
        type: 'wav',
        sampleRate: 16000,
        bitRate: 24
      })
      newRec.open(
        () => {
          this.rec = newRec
        },
        msg => {
          alert('录音权限已拒绝：' + msg)
        }
      )
    },
    doRecClose() {
      if (this.rec) {
        this.rec.close()
        this.rec = null
        console.log('资源已释放')
      } else {
        console.log('未打开录音')
      }
    },
    doRecStart() {
      if (this.rec && Recorder.IsOpen()) {
        console.log('开始录音')
        this.rec.start()
      }
    },
    doRecStop() {
      if (this.rec == null || !Recorder.IsOpen()) {
        console.log('未打开录音')
        return
      }
      this.rec.stop((blob, duration) => {
        console.log(`已录制${duration}ms，${blob.size}字节`)
        let audioRes = { blob, size: blob.size }
        Remote.getAnswerAccordAudio(audioRes)
      })
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
