<!-- 改change函数位置 -->
<template>
  <div class="root">

    <!-- 聊天室 -->
    <div class="background2">


      <div class="pd">
        <div style="display: flex;flex-direction: column;align-items: center">
          <div style="display: flex;flex-direction: row;justify-content: space-between;align-items: center;width:100% ">


            <el-select v-model="filtrateValue" placeholder="请选择" :disabled="disabled">
              <el-option
                  v-for="item in options"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value">
              </el-option>
            </el-select>
            <el-button type="primary" @click="onClickStart" :disabled="disabled">开始</el-button>

          </div>
          <video class="video" src="../assets/171_1729420090.mp4"
                 controls autoplay></video>
          <video class="video_cam" id="video" autoplay ref="videos"
                 muted></video>
        </div>
        <div style="padding-bottom: 310px;" v-if="num==2"></div>
        <div v-if="result_num==2">
          <p style="width: 460px;color: #893218;padding: 10px 0">{{ yy_result_analysis }}{{ progress }}，您的状态是：{{depressed_state}}</p>
          <section>
            <div class="box">
              <div class="mo" :style="{'--progress':-progress+100+'%'}"></div>
              <i></i>
            </div>
          </section>
          <p style="margin-top: 20px; padding-left: 50px;width: 400px; font-size: 16px;margin-bottom: 100px;">
            {{ yy_result_tips }}</p>

          <div style="margin-top: 200px;"></div>

          <p style="margin-top: 20px; padding-left: 50px;width: 400px; font-size: 16px;margin-bottom: 100px;">
           视频分段得分： {{ depressed_score_list }}</p>

          <div>
            <ul>
              <li v-for="item in detect_list" :key="item.id">
                <p>{{ item.class_name }}测试结果: {{ item.description }}</p>
              </li>
            </ul>
          </div>
        </div>

      </div>
    </div>
  </div>


</template>
<script>
import axios from 'axios'
import fixWebmDuration from 'webm-duration-fix'

export default {
  name: "Test",
  data() {
    return {
      options: [{value: 10, label: '快速筛选'}, {value: 35, label: '普通筛选'}],
      filtrateValue: 10,
      progress: 0,
      depressed_state: '',
      depressed_score_list: '',
      disabled: false,
      replayVideo: false,
      recordtype: "BEGIN",
      showReplay: true,
      timer: 0,
      recordtime: 0,
      second: 0,
      minute: 0,
      hour: 0,
      playtime: 0,
      playtimer: 0,
      yy_score: 0,
      id: this.GLOBAL.id,
      test_title: '请根据自身最近两周情况回答下列问题',
      title: '聊天室',
      description1: '来这儿坐坐，敞开心扉',
      description2: '忘记过去的痛楚，一切都会好的',
      content1: '没有人会一直顺利，愿你更加强大',
      content2: '然后有一天，你可以笑着讲述那些曾让你哭的瞬间',
      result: '加油加油加油加油加油加油加油加油加油加油加油加油加油加油加油加油加油加油加油加油加油加油加油加油',
      questionnaire_one:
          ["1、对做事没有兴趣或乐趣？",
            "2、感到沮丧、沮丧或绝望？",
            "3、入睡或保持睡眠困难，或睡眠过多？",
            "4、感到疲倦或精力不足？",
            "5、食欲不振或暴饮暴食？",
            "6、自我感觉不好，感觉自己是个失败者，让家人或自己失望了？",
            "7、注意力不能集中在一些事情上，比如读书或看电视？",
            "8、移动或说话的速度很慢，以至于其他人可能会注意到？或者是相反的情况--焦躁不安或坐立不安，以至于你比平时多四处走动？",
            "9、认为你最好死了或者以某种方式伤害自己？",
          ],
      questionnaire_two:
          ["1、过去数月，你是否多次突然感到很害怕或惊恐，而每次通常持续数分钟至数小时？",
            "2、经常检查灯和水龙头关好没有？",
            "3、有时担心会给自己或所爱的人带来伤害？",
            "4、经常想到亲人会有不幸？",
            "5、你是否总是尽量想提前离开有可能使你遭遇尴尬的境地？",
            "6、惊恐的时候，会感觉呼吸困难、担心不能自控？",
            "7、经常觉得身上衣服有些不对劲？",
            "8、是否老是对自己和自己所干的事不满意，尽管已经努力想干好？",
            "9、经历惊恐后，你是否持续一个月或以上，担忧以后再出现惊恐？",
          ],
      questions:
          ["你好呀，为了更好的进行测试，接下来我将问你几个问题，谈话内容完全保密，让我们开始吧！",
            "你最近心情怎么样",
            "是因为什么事情让你拥有这样的心情呢",
            "你最近做的最开心的一件事情是什么呢",
            "你最近有去旅行吗，如果有可以和我讲讲旅行中发生的事情吗",
            "你做过最后悔的一件事情是什么，可以和我分享一下吗",
            "你曾经被诊断出患有抑郁症吗",
            "你想改变自己现在的生活吗？如果想，你希望怎么改变",
            "好吧，我想我已经问了所有需要问的问题，谢谢你和我分享你的想法",
            "请耐心等待分析结果",
            "请耐心等待分析结果",
            "请耐心等待分析结果",
            "请耐心等待分析结果",
            "请耐心等待分析结果",
            "请耐心等待分析结果",
          ],
      // questions:
      //   ["你好呀，为了更好的进行测试，接下来我将问你几个问题，谈话内容完全保密，让我们开始吧！",
      //     "你最近感觉怎么样",
      //     "告诉我你最近做的一些你非常喜欢的事情",
      //     "你最好的朋友会如何形容你",
      //     "我很想听听你的一次旅行",
      //     "有什么让你后悔的事情吗",
      //     "你曾经被诊断出患有抑郁症吗",
      //     "你希望改变自己的哪些方面",
      //     "你一生中最自豪的是什么",
      //     "好吧，我想我已经问了所有需要问的问题，谢谢你和我分享你的想法"
      //   ],
      title_test: '抑郁症测试',
      this_question_num: 0,
      description_test1: '伤心、难过、失眠……你真的是抑郁症吗',
      description_test2: '如果不是，大胆放下顾虑，向阳而生',
      description_test3: '如果是，请努力走向出口，我们永远在你身后',
      num: 0,
      radio: [0, 0, 0, 0, 0, 0, 0, 0, 0,],
      result_num: 0,
      other_result_num: this.GLOBAL.other_result_num,
      // yy_result_analysis: '评估得分范围在0-23，分数越高越可能抑郁。您的得分为:',
      yy_result_analysis: '您的得分为:',
      // yy_result_analysis: '根据已综合的诊断标准，初步显示你可能已经患上抑郁症，建议你尽快向专业人士寻求评估。',
      yy_result_tips: '请注意，抑郁症自我测试绝对不能代替专业临床评估。如果你对抑郁症有任何疑问，请向临床心理学家或有关医生寻求专业协助。',
      other_result_analysis: "根据已综合的诊断标准，初步显示你可能已经患上" + this.GLOBAL.OTHER + "，建议你尽快向专业人士寻求评估。",
      other_result_analysis: "根据已综合的诊断标准，初步显示你没有患上" + this.GLOBAL.OTHER + "。",
      // other_result_analysis: '根据已综合的诊断标准，初步显示你可能已经患上，建议你尽快向专业人士寻求评估。',
      other_result_tips: "请注意，" + this.GLOBAL.OTHER + "自我测试绝对不能代替专业临床评估。如果你对" + this.GLOBAL.OTHER + "有任何疑问，请向临床心理学家或有关医生寻求专业协助。",
      audio: ['audio1', 'audio2', 'audio3', 'audio4', 'audio5', 'audio6', 'audio7', 'audio8', 'audio9', 'audio10', 'audio11', 'audio12', 'audio13', 'audio14', 'audio15', 'audio16'],
      cnt_sum: 0,
      ansMaxTime: 0,
      ansBeginTime: 0,
      ansMaxBeginTime: 0,

    }
  },
  created() {
    if (this.GLOBAL.IS_LOGIN == 0) {
      // this.$router.replace({ path: '/Login' });
      this.$router.push({path: '/Login'})
    }
    // console.log(parseInt('2.345'))
    this.id = this.GLOBAL.id
    if (this.GLOBAL.id == 2) {
      this.title_test = "情绪病测试"
    }
    this.other_result_num = 0
    console.log(this.GLOBAL.id)

  },
  methods: {
    onClickStart(){
      this.yy_score = 0
      this.progress = 0
      this.result_num = 0
      this.this_question_num = 0
      this.disabled  =true
      this.recordtime = 0;
      this.recordtype = "BEGIN";
      this.next()
    },
    getHistory() {
      // this.$router.replace({ path: '/HistoryView' });
      this.$router.push({path: '/HistoryView'})
    },
    // 调用摄像头
    callCamera() {
      let _this = this;
      MediaUtils.getUserMedia(true, true, function (err, stream) {
        if (err) {
          throw err;
        } else {
          // 通过 MediaRecorder 记录获取到的媒体流
          const mimeType = 'video/webm;codecs=vp8,opus';
          mediaRecorder = new MediaRecorder(stream, {
            // mimeType: "video/webm;codecs=vp9",
            mimeType: mimeType,
          });
          mediaStream = stream;
          var chunks = []
          // startTime = 0;
          var video = _this.$refs.videos;
          video["srcObject"] = stream;
          video.play();// 播放实时画面
          mediaRecorder.ondataavailable = function (e) {
            mediaRecorder.blobs.push(e.data);
            chunks.push(e.data);
          };
          mediaRecorder.blobs = [];

          mediaRecorder.onstop = async () => {
            // var duration = Date.now() - startTime;
            // console.log(duration)
            // c
            recorderFile = await fixWebmDuration(new Blob(chunks, {type: mimeType}));
            // ysFixWebmDuration(recorderFile, duration, function (fixedBlob) {
            //   recorderFile = fixedBlob
            //   console.log(recorderFile)
            // });
            console.log('=====', recorderFile);
            var url = URL.createObjectURL(recorderFile)
            // var b = URL.readAsDataURL(recorderFile)
            var videosreplay = _this.$refs.videosreplay;
            videosreplay.setAttribute("src", url);
            console.log('url', url)
            // console.log('base64', b)
            chunks = [];
            if (null != stopRecordCallback) {
              stopRecordCallback();
            }
          };
          _this.record()
        }
      });
    },
    record() {
      if (this.recordtype == "ING") {
        this.stopRecord(() => {
          console.log("结束录制");
        });
      } else if (this.recordtype == "BEGIN") {
        console.log("开始录制");
        this.startAudio();
        mediaRecorder.start();
        startTime = Date.now();
        this.recordtype = "ING";
      }
    },

    // 对录像时长进行记录
    startAudio() {
      this.timer = setInterval(() => {
        this.recordtime += 1000;
        if (this.recordtime == 1000 * this.filtrateValue) {
          this.stopRecord();
        }
        this.second++;
        if (this.second >= 60) {
          this.second = 0;
          this.minute = this.minute + 1;
        }

        if (this.minute >= 60) {
          this.minute = 0;
          this.hour = this.hour + 1;
        }
        console.log('==recordtime==recordtime', this.recordtime)
      }, 1000);
    },

    // 停止录像时终止录制器，关闭媒体流并清除时长记录定时器
    stopRecord(callback) {
      console.log('=====', 'stopRecord')
      this.recordtype = "END";
      this.showReplay = true;

      stopRecordCallback = callback;
      clearInterval(this.timer);
      // 终止录制器
      mediaRecorder.stop();
      // 关闭媒体流
      MediaUtils.closeStream(mediaStream);

      this.this_question_num = 8
      this.next()

      var videosreplay = this.$refs.videosreplay;
      videosreplay.onended = () => {
        this.playtime = 0;
        this.replayVideo = false;
        clearInterval(this.playtimer);
      };
      videosreplay.onclick = () => {
        this.showReplay = !this.showReplay;
      };


    },
    // 回放
    toggleReplayVideo() {
      console.log('播放中...')
      this.replayVideo = !this.replayVideo;
      this.showReplay = false;
      var videosreplay = this.$refs.videosreplay;
      if (this.replayVideo) {
        videosreplay.play().catch(err => {
          this.$message.error(err.message);
          console.log(err);
        });
        this.playtimer = setInterval(() => {
          this.playtime += 1000;
        }, 1000);
      } else {
        videosreplay.pause();
        clearInterval(this.playtimer);
      }
    },
 submit() {

      let that = this;
      console.log('cnt_sum111111111111', that.cnt_sum)
      console.log('recorderFile==',recorderFile)
      setTimeout(() => {
        console.log('recorderF1500ile==',recorderFile)
        var url = URL.createObjectURL(recorderFile)
        // var b = URL.readAsDataURL(recorderFile)
        console.log('url', url)
        // console.log('base64', b)
        let file = new File(
            [recorderFile],
            "msr-" + new Date().toISOString().replace(/:|\./g, "-") + ".mp4",
            {
              type: "video/mp4",
            }
        );
        let config = {
          headers: {"Content-Type": "multipart/form-data"}
        }
        console.log('file', file)
        const formdata = new FormData()
        formdata.append("file", file);
        console.log("video")
        console.log(formdata.get('file'))
        // axios.get('/video', { params: { vedio: formdata } }).then(function (result) {
        //   console.log(result)
        // }).catch(error => (console.log(error)))
        alert('分析需要一点时间，听首歌等等吧~~')
        // document.getElementById(that.audio[8]).pause()
        // document.getElementById(that.audio[9]).play()
        axios.post('/video', formdata, {headers: {'Content-Type': 'application/x-www-form-urlencoded'}},) //请求头要为表单
            .then(response => {
              console.log('video', response.data);
              this.yy_score = response.data.data.detect.depressed_index;
              that.progress = response.data.data.detect.depressed_score;
              that.depressed_state = response.data.data.detect.depressed_state;
              that.depressed_score_list = response.data.data.detect.depressed_score_list;
              that.detect_list = response.data.data.detect.detect_list;
              that.result_num = 2
              this.other_result_num = 0
              that.$message({
                message: '分析完成，请查看结果~~',
                type: 'success'
              });
              this.disabled = false;
            })
            .catch(function (error) {
              this.disabled = false;
              // document.getElementById(that.audio[9]).pause()
              // document.getElementById(that.audio[10]).play()
              that.$message({
                message: error,
                type: 'error'
              });
              console.log(error);
            })
      }, 1500);

    },
    next() {
      console.log(this.this_question_num)
      this.this_question_num = this.this_question_num + 1
      if (this.this_question_num == 1) {
        this.callCamera()
      }
      if (this.this_question_num == 8) {
        this.record()
      }
      if (this.this_question_num > 1 && this.this_question_num <= 8) {
        if (this.recordtime - this.ansBeginTime > this.ansMaxTime) {
          this.ansMaxTime = this.recordtime - this.ansBeginTime
          this.ansMaxBeginTime = this.ansBeginTime
        }
        this.ansBeginTime = this.recordtime
        console.log(this.ansMaxBeginTime, this.ansMaxTime)
      }

      if (this.this_question_num == 9) {

        axios.get('/Time', {
          params: {
            timeBegin: this.ansMaxBeginTime,
            timeEnd: this.ansMaxBeginTime + this.ansMaxTime
          }
        }).then(result => {
          console.log(result)
          if (this.this_question_num == 9) {
            this.submit()
          }
        }).catch(error => (console.log(error)))
      }
      if (this.this_question_num > 9) {
        this.$message({
          message: '请耐心等等分析结果',
          type: 'warning'
        });
      }
      // if (this.this_question_num > 0) {
      if (this.this_question_num < 9) {
        console.log('audio', this.audio[this.this_question_num])
        // document.getElementById(this.audio[this.this_question_num - 1]).pause()
        // document.getElementById(this.audio[this.this_question_num]).play()
        // document.getElementById(this.audio[11]).play()
      }

      // }


    },
    goChatRoom() {
      this.num = 2
      document.getElementById(this.audio[0]).play()
      console.log(this.num)
      this.cnt_sum = 0
      for (var i = 0; i <= 8; i++) {
        console.log(this.radio[i])
        console.log(this.cnt_sum)
        this.cnt_sum = this.cnt_sum + this.radio[i]
      }
      if (this.cnt_sum >= 13) {
        // this.yy_result_analysis = "根据已综合的诊断标准，初步显示你可能已经患上抑郁症，建议你尽快向专业人士寻求评估。"
        this.other_result_analysis = "根据已综合的诊断标准，初步显示你可能已经患上" + this.GLOBAL.OTHER + "，建议你尽快向专业人士寻求评估。"
      } else {
        // this.yy_result_analysis = "根据已综合的诊断标准，初步显示你没有患上抑郁症。"
        this.other_result_analysis = "根据已综合的诊断标准，初步显示你没有患上" + this.GLOBAL.OTHER + "。"
      }
      console.log('cnt', this.cnt_sum)
      // var message = JSON.stringify({ Score: this.cnt_sum });
      // var message = {  }
      axios.get('/Score', {params: {Score: this.cnt_sum}}).then(result => {
        console.log(result)
      }).catch(error => (console.log(error)))
    },
    getOtherResult() {
      this.cnt_sum = 0
      this.other_result_num = 2
      for (var i = 0; i <= 8; i++) {
        console.log(this.radio[i])
        console.log(this.cnt_sum)
        this.cnt_sum = this.cnt_sum + this.radio[i]
      }
      if (this.cnt_sum >= 15) {
        // this.yy_result_analysis = "根据已综合的诊断标准，初步显示你可能已经患上抑郁症，建议你尽快向专业人士寻求评估。"
        this.other_result_analysis = "根据已综合的诊断标准，初步显示你可能已经患上" + this.GLOBAL.OTHER + "，建议你尽快向专业人士寻求评估。"
      } else {
        // this.yy_result_analysis = "根据已综合的诊断标准，初步显示你没有患上抑郁症。"
        this.other_result_analysis = "根据已综合的诊断标准，初步显示你没有患上" + this.GLOBAL.OTHER + "。"
      }
      console.log('cnt', this.cnt_sum)
    }


  }

}
var MediaUtils = {
  /**
   * 获取用户媒体设备(处理兼容的问题)
   * @param videoEnable {boolean} - 是否启用摄像头
   * @param audioEnable {boolean} - 是否启用麦克风
   * @param callback {Function} - 处理回调
   */
  getUserMedia: function (videoEnable, audioEnable, callback) {
    navigator.getUserMedia =
        navigator.getUserMedia ||
        navigator.webkitGetUserMedia ||
        navigator.mozGetUserMedia ||
        navigator.msGetUserMedia ||
        window.getUserMedia;
    var constraints = {video: videoEnable, audio: audioEnable};
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      navigator.mediaDevices
          .getUserMedia(constraints)
          .then(function (stream) {
            callback(false, stream);
          })
          ["catch"](function (err) {
        callback(err);
      });
    } else if (navigator.getUserMedia) {
      navigator.getUserMedia(
          constraints,
          function (stream) {
            callback(false, stream);
          },
          function (err) {
            callback(err);
          }
      );
    } else {
      callback(new Error("Not support userMedia"));
    }
  },

  /**
   * 关闭媒体流
   * @param stream {MediaStream} - 需要关闭的流
   */
  closeStream: function (stream) {
    if (typeof stream.stop === "function") {
      stream.stop();
    } else {
      let trackList = [stream.getAudioTracks(), stream.getVideoTracks()];

      for (let i = 0; i < trackList.length; i++) {
        let tracks = trackList[i];
        if (tracks && tracks.length > 0) {
          for (let j = 0; j < tracks.length; j++) {
            let track = tracks[j];
            if (typeof track.stop === "function") {
              track.stop();
            }
          }
        }
      }
    }
  },
};
var startTime, mediaRecorder, mediaStream, stopRecordCallback, recorderFile;
</script>
<style scoped>
.box {
  position: relative;
  width: 500px;
  height: 20px;
  overflow: hidden;
  background: linear-gradient(270deg, #ff0000 10%, #ffc801 70%, #00ffa2 100%);
}

.mo {
  position: absolute;
  z-index: 1;
  right: 0;
  height: 100%;
  width: var(--progress, 0%);
  background-color: #ccc;
}

.root {
  display: flex;
  justify-content: center;
  align-items: center;
}

.background2 {

  width: 40%;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}


.pd {
  padding: 0;

  .video {
    width: 100%;
  }
}

p {
  font-size: 18px;
  line-height: 22px;
}

.video_cam {
  width: 100%;
  height: 100%;
}
</style>
