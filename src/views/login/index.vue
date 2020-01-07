<template>
  <div class="login-container">
    <div class="login-box">
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules">
        <img src="./logo_index.png" alt />
        <el-form-item prop="mobile">
          <el-input v-model="loginForm.mobile" placeholder="请输入手机号码"></el-input>
        </el-form-item>
        <el-form-item prop="code">
          <el-input v-model="loginForm.code" placeholder="请输入验证码"></el-input>
        </el-form-item>
        <el-form-item style="text-align:left;" prop="xieyi">
          <el-checkbox v-model="loginForm.xieyi" style="margin-right:10px;"></el-checkbox>
          <span>
            我已阅读并同意
            <a href="#">用户协议</a>和
            <a href="#">隐私条款</a>
          </span>
        </el-form-item>
        <el-form-item>
          <el-button style="width:100%;" type="primary" @click="login()">登录</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>

import './gt'
export default {
  data () {
    var xieyiTest = function (rule, value, callback) {
      //   if (value) {
      //     callback()
      //   } else {
      //     callback(new Error('请遵守协议'))
      //   }
      value ? callback() : callback(new Error('请遵守协议'))
    }
    return {
      loginForm: {
        mobile: '13704475685',
        code: '246810',
        xieyi: false
      },
      loginFormRules: {
        mobile: [
          { required: true, message: '手机号码必填' },
          { pattern: /^1[3-9]\d{9}$/, message: '手机号码格式不对' }
        ],
        code: [{ required: true, message: '验证码必填' }],
        xieyi: [{ validator: xieyiTest }]
      }
    }
  },
  methods: {
    login () {
      this.$refs.loginFormRef.validate(valid => {
        if (!valid) {
          return false
        }
        // this.$router.push('/home')
        // this.$router.push({ name: 'home' })
        let pro = this.$http({
          url: '/mp/v1_0/captchas/' + this.loginForm.mobile,
          method: 'GET'
        })
        pro.then(result => {
          console.log(result)
          let { data } = result.data
          window.initGeetest({
            // 以下配置参数来自服务端 SDK
            gt: data.gt,
            challenge: data.challenge,
            offline: !data.success,
            new_captcha: true,
            product: 'bind'
          }, captchaObj => {
            // 这里可以调用验证实例 captchaObj 的实例方法
            captchaObj.onReady(() => {
              // 验证码ready之后才能调用verify方法显示验证码
              captchaObj.verify() // 显示验证码
            }).onSuccess(() => {
              // your code
              this.loginAct()
            }).onError(() => {
              // your code
            })
            // 按钮提交事件

            // some code
          })
        })
          .catch((err) => {
            console.log(err)

            return this.$message.error('整错了')
          })
      })
    },
    loginAct () {
      let pro = this.$http({
        url: '/mp/v1_0/authorizations',
        method: 'POST',
        data: this.loginForm
      })
      pro
        .then(result => {
          window.sessionStorage.setItem('userinfo', JSON.stringify(result.data.data))
          this.$router.push({ name: 'home' })
        })
        .catch(() => {
          this.$message.error('电话号码或验证码错误')
        })
    }
  }
}
</script>

<style lang="less" scoped>
.login-container {
  //   background-color: gray;
  background-image: url("./login_bg.jpg");
  background-size: cover;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.login-box {
  background-color: #fff;
  height: 340px;
  width: 410px;
  display: flex;
  justify-content: center;
  align-items: center;
  .el-form {
    width: 75%;
    text-align: center;
  }
  img {
    width: 50%;
    margin: 20px auto;
  }
  a {
    text-decoration: none;
  }
}
</style>
