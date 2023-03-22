<template>
  <div class="login-page">
    <div class="tips">
      <div v-show="loginType === 'key'">
        <p>
          你可以在<a target="_blank" href="https://platform.openai.com/account/api-keys">platform.openai.com</a>创建你的API KEY, 你应该遵循以下条款:
        </p>
        <ul style="margin-bottom: 20px">
          <li>
            你的API密钥属于个人信息，请妥善保管。
          </li>
          <li>
            要使用 OpenAI 进行身份验证，我们会将您的 API 密钥发送到我们的服务器进行处理，但它永远不会存储在那里。不信？前往检查 <a
            href="https://github.com/jw-12138/davinci-web/blob/main/back-end/main.cjs#L158" target="_blank">source code</a>.
          </li>
          <li>
            确保此设备受信任，我们会将您的API密钥存储在此浏览器中。如果您使用的是公用设备，请记住在使用完后注销。
          </li>
          <li>
            确保您在安全的网络上，以防止您的 API 密钥可能被盗。
          </li>
          <li>
            如果您认为您的 API 密钥已泄露，请尽快撤销它！
          </li>
        </ul>
      </div>

    </div>
    <!--ban掉sso-->
    <!--
    <div v-show="loginType === 'password'">
      <button class="sso" @click="goToSSO">
        <span>用SSO登录</span>
      </button>
    </div>
    -->
    <div class="password" v-show="loginType === 'key'">
      <input type="password" v-model="password" autofocus @keydown="listenForEnter" @focus="passwordFocus = true"
             @blur="passwordFocus = false" placeholder="API key" enterkeyhint="go">
      <button @click="login" :disabled="trying"><i v-show="trying" class="iconfont spin">&#xe676;</i>登录</button>
    </div>
    <div style="font-size: 14px;">
      <br>
      <p v-show="loginType === 'password'">
        <button class="plain" @click="loginType = 'key'">用API Key登录</button>
      </p>
      <!--ban掉sso-->
      <!--
      <p v-show="loginType === 'key'">
        <button class="plain" @click="loginType = 'password'">用SSO登录</button>
      </p>
      -->
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import {nanoid} from 'nanoid'

export default {
  name: 'login',
  mounted() {
    this.id = nanoid(32)
    if (location.hostname === 'localhost') {
      this.url = `https://sso.jw1.dev/#/sign-in?from=chat_local&id=${this.id}&client_id=${USER_POOL_CLIENT_ID}`
    } else {
      this.url = `https://sso.jw1.dev/#/sign-in?from=chat&id=${this.id}&client_id=${USER_POOL_CLIENT_ID}`
    }
  },
  data() {
    return {
      id: '',
      passwordFocus: false,
      password: '',
      trying: false,
      loginType: 'password',
      url: ''
    }
  },
  methods: {
    goToSSO() {
      location.href = this.url
    },
    listenForEnter(e) {
      if (e.key === 'Enter' && this.password !== '' && this.passwordFocus) {
        this.login()
      }
    },
    verifyKey(cb) {
      let key = this.password
      if (!key.startsWith('sk-')) {
        cb && cb(false)
        alert('The token you entered seems to be invalid 🤔')
        return false
      }

      this.trying = true

      axios({
        url: 'https://api.openai.com/v1/moderations',
        headers: {
          'Authorization': 'Bearer ' + key
        },
        method: 'POST',
        data: {
          input: `Hello World!`
        }
      }).then(res => {
        console.log(res)
        cb && cb(true)
      }).catch(err => {
        console.log(err)
        err.response.status === 401 && alert('The key you entered seems to be invalid 🤔')
        cb && cb(false)
      }).finally(() => {
        this.trying = false
        this.password = ''
      })
    },
    login() {
      let _ = this
      if (_.password === '') {
        alert('A token is required')
        return false
      }

      _.verifyKey(function (res) {
        if (res) {
          localStorage.setItem('fromID', 'key_' + _.password)
          _.$emit('logged')
        }
      })
    }
  }
}
</script>
