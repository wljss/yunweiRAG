<template>
  <div style="padding: 10px; margin-bottom: 50px">
    <el-row>

      <!--右半部分-->
      <el-col :span="16">
        <div
          style="width: 970px; margin: 0 auto; background-color: white;
                        border-radius: 5px; box-shadow: 0 0 10px #ccc"
        >
          <!--聊天框顶部显示的信息-->
          <div style="text-align: center; line-height: 50px;">
            RAG智能机器人
          </div>

          <!--显示消息的框-->
          <div style="height: 350px; overflow:auto; border-top: 1px solid #ccc" v-html="content" />

          <!--发送消息的框-->
          <div style="height: 200px">
            <textarea
              v-model="userInput"
              style="height: 200px; width: 100%; padding: 20px; border: none; border-top: 1px solid #ccc;
                 border-bottom: 1px solid #ccc; outline: none"
              placeholder="此处输入问题"
            />

            <!--设置提示词的按钮-->
            <div style="text-align: left; padding-left: 1px ;padding-top: 1px">
              <el-button type="primary" size="huge" @click="getDetails">设置提示词</el-button>
            </div>

            <!--发送消息的按钮-->
            <div style="text-align: right; padding-right: 0px; margin-top:-135px;">
              <el-button type="primary" size="huge" @click="send">发送问题</el-button>
            </div>

          </div>
        </div>
      </el-col>
    </el-row>

    <el-dialog :visible.sync="seeDetails">

      <el-divider content-position="left" style="font-size:30px;"><p style="font-size:15px;color: rgb(255, 0, 0);;">提示词</p></el-divider>

      <div class="listItem">
        <!--输入框 v-model是双向绑定 -->
        <el-input
          ref="prompt"
          v-model="prompt"
          placeholder="填写提示词"
          name="prompt"
          type="textarea"
          tabindex="5"
          auto-complete="on"
          :rows="8"
        />
      </div>

      <div class="listItem">
        <el-button
          type="primary"
          style="margin-bottom:30px;margin-top:25px; "
          @click.native.prevent="FaSong"
        >确认</el-button>
      </div>

    </el-dialog>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import CryptoJS from 'crypto-js'
import { Message } from 'element-ui'
import axios from 'axios';

let socket
export default {
  name: 'Chat',
  components: { },
  props: {
    className: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      appId: '1b42a456',
      user: {},
      userInput: '',
      finalChat: [],
      content: '',
      lastContent: '',
      duihua: '',

      seeDetails: false,
      prompt: '',
      nowState: '0'
    }
  },
  computed: {
    ...mapGetters([
      'name',
      'avatar'
    ])
  },
  watch: {
    async className(val, valOld) { // 这里也可以在 data 定义变量 并将 father 赋值给他， 在这里监控这个变量// 这里做如果 father 变量变化了，子组件需要处理的逻辑
      this.$http.get('/prompt/findByClassName', {
        params: {
          className: this.className
        }
      }).then((response) => {
        this.prompt = response.data[0].content
        console.log('this.prompt')
        console.log(this.prompt)
      })
    }
  },
  created() {
    this.init()
  },
  methods: {
    getDetails(rows, index) {
      this.seeDetails = true
    },
    FaSong() {
      console.log('this.prompt')
      console.log(this.prompt)
      this.$http.get('/prompt/updateContent', {
        params: {
          className: this.className,
          content: this.prompt
        }
      }).then((response) => {
        Message.success('修改成功')
      })
    },
    send() { // 发送消息
      console.log('现在在send()1')
      //this.createContent(null, this.name, this.userInput)

      this.loading = true
      this.totalRes = ''
      this.aiContentRequest = ''
      
      console.log('现在在send()2')
      axios.post('http://localhost:7861/chat/kb_chat', {
            query: '你是谁？',
            mode: 'local_kb',
            kb_name: 'yunwei',
        })
        .then(response => {
          console.log(response)
        })
        .catch(error => {
          // 处理错误
        });

    },
    createContent(remoteUser, nowUser, text) { // 这个方法是用来将 聊天消息数据转换，显示出来
      let html
      this.user = localStorage.getItem('user') ? JSON.parse(localStorage.getItem('user')) : {}// 注意需要运行...mapGetters
      if (nowUser) { // 如果是自己发的消息，绿色气泡
        html = '<div class="el-row" style="padding: 5px 0">\n' +
                '  <div class="el-col el-col-22" style="text-align: right; padding-right: 10px">\n' +
                '    <div class="tip left">' + '<el-col style="white-space: pre-wrap;">' + text + '</el-col>' + '</div>\n' +
                '  </div>\n' +
                '  <div class="el-col el-col-2">\n' +
                '  <span class="el-avatar el-avatar--circle" style="height: 40px; width: 40px; line-height: 40px;">\n' +
                '    <img src="' + this.avatar + '" style="object-fit: cover;">\n' +
                '  </span>\n' +
                '  </div>\n' +
                '</div>'

        this.content += html
        this.lastContent = this.content

        this.$http.get('/dialogueHistory/add', {
          params: {
            sender: this.name,
            avatar: this.avatar,
            content: text,
            className: this.className
          }
        }).then((response) => {
        })
      } else if (remoteUser) { // 接收到别人发的消息，蓝色的气泡
        this.duihua += text
        html = '<div class="el-row" style="padding: 5px 0">\n' +
                '  <div class="el-col el-col-2" style="text-align: right">\n' +
                '  <span class="el-avatar el-avatar--circle" style="height: 40px; width: 40px; line-height: 40px;">\n' +
                '    <img src="https://tse1-mm.cn.bing.net/th/id/OIP-C.N7IbUPCOlwLy157xlFMPDAHaHa?w=178&h=180&c=7&r=0&o=5&dpr=1.5&pid=1.7" style="object-fit: cover;">\n' +
                '  </span>\n' +
                '  </div>\n' +
                '  <div class="el-col el-col-22" style="text-align: left; padding-left: 10px">\n' +
                '    <div class="tip right">' + '<el-col style="white-space: pre-wrap;">' + this.duihua + '</el-col>' + '</div>\n' +
                '  </div>\n' +
                '</div>'

        this.content = this.lastContent + html
      }
    },
    init() { // 初始化，记录用户名
      this.user = localStorage.getItem('user') ? JSON.parse(localStorage.getItem('user')) : {}
      const username = this.name
      console.log(this.name)
    },

    getWebsocketUrl() {
      return new Promise((resolve, reject) => {
        const apiKey = 'c3808e318cc1679ffd3a7333d7068aa6'
        const apiSecret = 'MzgwZDNmODExNDdkZjYwM2U5YjE4ZGE2'
        const url = 'wss://spark-api.xf-yun.com/v3.5/chat' // 这里使用的是星火大模型1.x版本
        const host = window.location.host

        const date = new Date().toGMTString()
        const algorithm = 'hmac-sha256'
        const headers = 'host date request-line'
        const signatureOrigin = `host: ${host}\ndate: ${date}\nGET /v3.5/chat HTTP/1.1`
        const signatureSha = CryptoJS.HmacSHA256(signatureOrigin, apiSecret)
        const signature = CryptoJS.enc.Base64.stringify(signatureSha)
        const authorizationOrigin = `api_key="${apiKey}", algorithm="${algorithm}", headers="${headers}", signature="${signature}"`
        const authorization = window.btoa(authorizationOrigin)
        const finalUrl = `${url}?authorization=${authorization}&date=${date}&host=${host}`
        console.log(finalUrl)
        resolve(finalUrl)
      })
    },
    connectWebSocket() {
      this.getWebsocketUrl().then((url) => {
        let ttsWS
        if ('WebSocket' in window) {
          ttsWS = new WebSocket(url)
        } else if ('MozWebSocket' in window) {
          ttsWS = new MozWebSocket(url)
        } else {
          alert('浏览器不支持WebSocket')
          return
        }
        this.ttsWS = ttsWS
        ttsWS.onopen = (e) => {
          this.webSocketSend()
        }
        ttsWS.onmessage = (e) => {
          this.result(e.data)
        }
        ttsWS.onerror = (e) => {
          clearTimeout(this.playTimeout)
          alert('WebSocket报错，请f12查看详情')
          console.error(`详情查看：${encodeURI(url.replace('wss:', 'https:'))}`)
        }
        ttsWS.onclose = (e) => {
          console.log(e)
        }
      })
    },
    webSocketSend() {
      const that = this
      const params = {
        header: {
          app_id: this.appId,
          uid: '随意'
        },
        parameter: {
          chat: {
            domain: 'generalv3.5', // 如果是chat2这里也需要进行相应修改
            temperature: 0.5,
            max_tokens: 1024
          }
        },
        payload: {
          message: {
            text: [
              { 'role': 'user', 'content': that.userInput + '请你仅使用文字回答，回答不要用加粗字体。' + this.prompt + (this.nowStatestate === '2' ? '本次回答请仅提供详细思路，并提供代码，并带上中文注释' : (this.nowStatestate === '1' ? '本次回答请仅提供详细思路，不要提供代码' : '本次回答请仅提供简要思路，最好用问句来进行启发，不要提供代码，要求200字以内')) }
            ]
          }
        }
      }
      if (this.nowStatestate === '0') this.nowStatestate = '1'
      else if (this.nowStatestate === '1') this.nowStatestate = '2'
      else this.nowStatestate = '0'
      console.log(JSON.stringify(params))
      this.ttsWS.send(JSON.stringify(params))
      this.userInput = ''
    },

    requestHandle(requestData) { // 处理request
      this.aiContentRequest = this.aiContentRequest + requestData.payload.choices.text[0].content
      console.log(requestData.payload.choices.text[0].content)
      this.createContent(this.name, null, requestData.payload.choices.text[0].content)
    },
    result(resultData) {
      const jsonData = JSON.parse(resultData)
      // console.log(jsonData)
      this.totalRes += resultData
      // this.$refs.outputText.value = this.totalRes;
      // 加入到ai回答中
      if (jsonData.header.status !== 2) { // 不为结束就进行添加
        this.requestHandle(jsonData)
        this.userInput = ''
      } else {
        const contentSomething = {
          ai: this.aiContentRequest,
          user: this.userInput
        }
        this.finalChat.push(contentSomething)

        this.loading = false

        this.$http.get('/dialogueHistory/add', { // 将对话记录存入数据库，以便老师检查
          params: {
            sender: 'AI',
            avatar: 'https://tse1-mm.cn.bing.net/th/id/OIP-C.N7IbUPCOlwLy157xlFMPDAHaHa?w=178&h=180&c=7&r=0&o=5&dpr=1.5&pid=1.7',
            content: this.duihua,
            className: this.className
          }
        }).then((response) => {
        })

        this.duihua = ''
        this.lastContent = this.content
      }
      if (jsonData.header.code !== 0) {
        alert(`提问失败: ${jsonData.header.code}:${jsonData.header.message}`)
        console.error(`${jsonData.header.code}:${jsonData.header.message}`)
        return
      }
      if (jsonData.header.code === 0 && jsonData.header.status === 2) {
        this.ttsWS.close()
      }
    }
  }
}
</script>
  <style>
  .tip {
    color: white;

    border-radius: 10px;
    font-family: sans-serif;
    padding: 10px;
    width:auto;
    display:inline-block !important;
    display:inline;
  }
  .right {
    text-align: left;
    background-color: deepskyblue;
    margin-right:100px;
  }
  .left {
    text-align: right;
    background-color: forestgreen;
    margin-left:100px;
  }
  </style>
