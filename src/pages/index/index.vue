<!-- 使用 type="home" 属性设置首页，其他页面不需要设置，默认为page；推荐使用json5，更强大，且允许注释 -->
<route lang="json5" type="home">
{
  style: {
    navigationStyle: 'custom',
    navigationBarTitleText: '首页',
  },
}
</route>
<template>
  <view class="bg-white overflow-hidden pt-2 px-4">
    <view>
      <!-- <snapshot mode="view" id="targetSnapshot"> -->
      <video id="myVideo" :src="tempFile" enable-danmu @timeupdate="timeUpdateHandle" danmu-btn controls></video>
      <!-- </snapshot> -->
      <button @click="doshot">截图</button>
      <button @click="getCircleCapture">渲染视频到canvas</button>
      <view>截图指定时间（读取视频的当前时间）</view>
      <button @click="chooseMedia">上传视频</button>
      <!-- <img :src="shot" class="w-full% h-200px" mode='aspectFit'> -->

      <canvas id="tempCanvas" canvas-id="tempCanvas" type="2d"></canvas>
    </view>
  </view>
</template>

<script lang="ts" setup>

import { nextTick, ref } from "vue"



const _this = getCurrentInstance();

const tempFile = ref("");

const height = ref(100);
let videoContext = null;
let shot = ref("");
let currentTime = 0;

const timeUpdateHandle = async (event) => {
  currentTime = event.detail.currentTime
}

const chooseMedia = async () => {
  uni.chooseMedia({
    count: 9,
    mediaType: ['video'],
    sourceType: ['album'],
    success(res) {
      console.log(res.tempFiles)
      tempFile.value = res.tempFiles[0].tempFilePath
    }
  })
}

const doshot = () => {
  videoContext = uni.createVideoContext('myVideo', _this); //创建视频实例指向video
  videoContext.pause();
  nextTick(() => {
    getCapture();
  });
}

const getCapture2 = () => {
  let pages = getCurrentPages();
  console.log(pages)
  let page = pages[pages.length - 1];
  let ws = page.$getAppWebview();
  var bitmap = new plus.nativeObj.Bitmap('test');
  // 将webview内容绘制到Bitmap对象中
  ws.draw(
    bitmap,
    () => {
      console.log('截屏绘制图片成功');
      // 将原生Bitmap转换成Base64字符串
      shot.value = bitmap.toBase64Data();
      videoContext = uni.createVideoContext('myVideo'); //创建视频实例指向video
      videoContext.play();
    },
    (e) => {
      console.log('截屏绘制图片失败：', e);
    },
    {
      check: true, // 设置为检测白屏
      clip: { top: "0px", left: '0px', height: '266px', width: '100%' } // 设置截屏区域
    }
  );
}


const getCapture3 = () => {
  uni.createSelectorQuery()
    .in(_this)
    .select("#targetSnapshot")
    .node((res) => {
      const node = res.node
      node.takeSnapshot({
        type: 'file',
        format: 'png',
        success: (ress) => {
          console.log('截屏绘制图片成功', ress);
          // 将原生Bitmap转换成Base64字符串
          shot.value = ress.toBase64Data();
        },
        fail(err) {
          console.log(err)
        }
      })
    })
    .exec(res => { })
}

const getTempPath = (url) => {
  return new Promise((r, j) => {
    wx.downloadFile({
      url: url,
      success: r,
      fail: j
    })
  })
}

const getCircleCapture = async () => {

  videoContext = uni.createVideoContext('myVideo', _this); //创建视频实例指向video
  console.log(videoContext);

  const query = uni.createSelectorQuery().in(_this);
  query.select("#tempCanvas").node(async (res) => {
    console.log("22", res)

    const canvas = res.node

    const decoder = wx.createVideoDecoder()

    // const context = uni.createCanvasContext('tempCanvas');
    const context = canvas.getContext('2d')
    // await new Promise(resolve => {
    // decoder.on('start', resolve)


    // decoder.on("start", async (res) => {
    //   // const sog = await decoder.getFrameData();
    //   console.log(res)
    //   decoder.seek(currentTime);
    // })
    await decoder.start({
      source: tempFile.value,
      mode: 0
      // abortAudio: false, // 不需要音频
    })

    let ended = false

    decoder.on('ended', () => {
      ended = true
    })


    let frameData

    // 等待开始解码
    do {
      // await new Promise(resolve => canvas.requestAnimationFrame(resolve))
      frameData = decoder.getFrameData()
    } while (!frameData)


    // 设置 canvas 宽高

    const windowInfo = uni.getWindowInfo();
    height.value = (windowInfo.windowWidth * frameData.height) / frameData.width;



    const render = ({ data, width, height }) => {
      const imageData = canvas.createImageData(data, width, height)
      // const base64 = uni.arrayBufferToBase64(frameData)
      context.drawImage(imageData, 0, 0)
    }

    const startTime = Date.now()

    // 循环绘制解码结果
    do {
      if (frameData && frameData.pkPts <= Date.now() - startTime) {
        // 控制在到达 pts 后显示
        render(frameData)
        frameData = null
      } else {
        await new Promise(resolve => canvas.requestAnimationFrame(resolve))
        if (!frameData) frameData = decoder.getFrameData()
      }
    } while (!ended)

    console.log('ended')
    // })
    // decoder.on("seek", async () => {
    //   const sog = decoder.getFrameData();
    //   console.log(sog)
    //   const sog2 = await decoder.getFrameData();
    //   console.log(sog2)
    // })


    // 解码结束
    await decoder.remove()
  })
    .exec(res => {
      console.log("11", res)

    })



}

const getCapture = async () => {

  videoContext = uni.createVideoContext('myVideo', _this); //创建视频实例指向video
  console.log(videoContext);

  const query = uni.createSelectorQuery().in(_this);
  query.select("#tempCanvas").node(async (res) => {
    console.log("22", res)

    const canvas = res.node

    const decoder = wx.createVideoDecoder()

    // const context = uni.createCanvasContext('tempCanvas');
    const context = canvas.getContext('2d')
    // await new Promise(resolve => {
    // decoder.on('start', resolve)


    // decoder.on("start", async (res) => {
    //   // const sog = await decoder.getFrameData();
    //   console.log(res)
    //   decoder.seek(currentTime);
    // })
    await decoder.start({
      source: tempFile.value,
      mode: 0
      // abortAudio: false, // 不需要音频
    })

    let ended = false

    decoder.on('ended', () => {
      ended = true
    })

    decoder.on('seek', async () => {

      // 等待开始解码
      do {
        // await new Promise(resolve => canvas.requestAnimationFrame(resolve))
        frameData = decoder.getFrameData()
      } while (!frameData)

      // 循环绘制解码结果
      do {
        if (frameData && frameData.pkPts <= Date.now() - startTime) {
          // 控制在到达 pts 后显示
          render(frameData)
          frameData = null
          break;
        } else {
          await new Promise(resolve => canvas.requestAnimationFrame(resolve))
          if (!frameData) frameData = decoder.getFrameData()
        }
      } while (!ended)

      // 解码结束
      await decoder.remove()
    })


    let frameData

    // 等待开始解码
    do {
      // await new Promise(resolve => canvas.requestAnimationFrame(resolve))
      frameData = decoder.getFrameData()
    } while (!frameData)


    // 设置 canvas 宽高

    const windowInfo = uni.getWindowInfo();
    height.value = (windowInfo.windowWidth * frameData.height) / frameData.width;

    await decoder.seek(currentTime);


    const render = ({ data, width, height }) => {
      const imageData = canvas.createImageData(data, width, height)
      // const base64 = uni.arrayBufferToBase64(frameData)
      context.drawImage(imageData, 0, 0)
    }

    const startTime = Date.now()

    console.log('ended')
    // })
    // decoder.on("seek", async () => {
    //   const sog = decoder.getFrameData();
    //   console.log(sog)
    //   const sog2 = await decoder.getFrameData();
    //   console.log(sog2)
    // })


  })
    .exec(res => {
    })
}

</script>

<style>
.main-title-color {
  color: #d14328;
}
</style>
