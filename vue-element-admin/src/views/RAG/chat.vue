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

            <!--发送消息的按钮-->
            <div style="text-align: right; padding-right: 0px; margin-top:-135px;">
              <el-button type="primary" size="huge" @click="send">发送问题</el-button>
            </div>

          </div>
        </div>
      </el-col>
    </el-row>

  </div>
</template>

<script>
import { mapGetters } from 'vuex'

// import CryptoJS from 'crypto-js'
// import { Message } from 'element-ui'

// let socket
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
      //user: {},
      userInput: '',
      content: '',
      lastContent: '',
      duihua: '',

    }
  },
  computed: {
    ...mapGetters([
      'name',
      'avatar'
    ])
  },
  watch: {
  },
  created() {
    this.init()
  },
  methods: {
    async send() { // 发送消息
      // this.createContent(null, this.name, this.userInput)

      this.loading = true
      this.totalRes = ''
      this.aiContentRequest = ''
      let new_prompt=''
      let llm_ans=''

      console.log('现在在send() 修改部分')
      const data1 = {
        'query': '请你依据你的知识重新表述以下问题,将代词替换为具体代指的信息，不要有多余的回复：如何正确安装R5410 G11服务器的CPU？',
        'mode': 'local_kb',
        'kb_name': 'yunwei',
        'top_k': 3,
        'score_threshold': 2,
        'history': [
          {
            'content': '假如你是一位网络运维工程师',
            'role': 'user'
          },
          {
            'content': '嗯我是一位网络运维工程师',
            'role': 'assistant'
          }
        ],
        'stream': 'true',
        'model': 'qwen2.5:3b',
        'temperature': 0.7,
        'max_tokens': 0,
        'prompt_name': 'default',
        'return_direct': 'false'
      }
      console.log(data1)
      try {
        const response = await fetch('/chat/kb_chat', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json' // 设置请求头为JSON格式
          },
          body: JSON.stringify(data1)
        })
        console.log(`响应状态码: ${response.status}`)
        const reader = response.body.getReader()
        const decoder = new TextDecoder()
        while (true) {
          const { done, value } = await reader.read()
          if (done) break
          const chunk = decoder.decode(value)
          // 处理每个数据块
          if (chunk.startsWith('data:')) {
            const tmp = JSON.parse(chunk.slice(5).trim()).choices[0].delta.content
            new_prompt += tmp
            console.log(tmp)
          }
        }
      } catch (error) {
        console.error('请求失败:', error)
      }
      console.log(new_prompt)

      console.log('现在在send() 询问部分')
      const data2 = {
        'query': new_prompt,
        'mode': 'local_kb',
        'kb_name': 'yunwei',
        'top_k': 3,
        'score_threshold': 2,
        'history': [
          {
            'content': '假如你是一位网络运维工程师',
            'role': 'user'
          },
          {
            'content': '嗯我是一位网络运维工程师',
            'role': 'assistant'
          }
        ],
        'stream': 'true',
        'model': 'qwen2.5:3b',
        'temperature': 0.7,
        'max_tokens': 0,
        'prompt_name': 'default',
        'return_direct': 'false'
      }
      console.log(data2)
      try {
        const response = await fetch('/chat/kb_chat', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json' // 设置请求头为JSON格式
          },
          body: JSON.stringify(data2)
        })
        console.log(`响应状态码: ${response.status}`)
        const reader = response.body.getReader()
        const decoder = new TextDecoder()
        while (true) {
          const { done, value } = await reader.read()
          if (done) break
          const chunk = decoder.decode(value)
          // 处理每个数据块
          if (chunk.startsWith('data:')) {
            const tmp = JSON.parse(chunk.slice(5).trim()).choices[0].delta.content
            llm_ans += tmp
            console.log(tmp)
          }
        }
      } catch (error) {
        console.error('请求失败:', error)
      }
      console.log(llm_ans)
    },

    createContent(remoteUser, nowUser, text) { // 这个方法是用来将 聊天消息数据转换，显示出来
      let html
      //this.user = localStorage.getItem('user') ? JSON.parse(localStorage.getItem('user')) : {}// 注意需要运行...mapGetters
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
      // this.user = localStorage.getItem('user') ? JSON.parse(localStorage.getItem('user')) : {}
      // const username = this.name
      // console.log(this.name)
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
