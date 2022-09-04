<template>
  <div class="main">
    <div class="form">
      <img src="../assets/logo.png" class="logo" alt="logo"/>
      <el-form :model="loginForm"
               :rules="rules"
               ref="loginForm">
        <el-form-item prop="username">
          <el-input type="text"
                    v-model="loginForm.username"
                    autocomplete="off"
                    size="large"
                    :prefix-icon="User"/>
        </el-form-item>
        <el-form-item prop="password">
          <el-input type="password"
                    v-model="loginForm.password"
                    autocomplete="off"
                    size="large"
                    :prefix-icon="Lock"/>
        </el-form-item>
        <el-form-item prop="captchaValue">

        </el-form-item>
        <el-form-item>
          <el-button type="primary"
                     class="button"
                     size="large"
                     @click="submitForm('loginForm')">登 录</el-button>
        </el-form-item>
      </el-form>
    </div>

  </div>
</template>

<script>
import { ElMessage } from 'element-plus'
import { User, Lock, CircleCheck } from '@element-plus/icons-vue'

export default {
  name: "Login",
  setup() {
    return {User, Lock, CircleCheck}
  },
  data() {
    return {
      show: true,
      loginForm: {
        username: 'psikun',
        password: 'psikun',
        captchaId: '',
        captchaValue: '',
      },
      captchaImg: null,
      rules: {
        username: [
          {equired: true, message: '请输入账号', trigger: 'blur'}
        ],
        password: [
          {equired: true, message: '请输入密码', trigger: 'blur'}
        ],
        // captchaValue: [
        //   {required: true, message: '请输入验证码', trigger: 'blur'},
        //   {min: 4, max: 4, message: '长度为 4 个字符', trigger: 'blur'}
        // ],
      }
    };
  },
  mounted() {
    // this.getCaptcha();
  },
  updated() {
    // this.getCaptcha();
  },
  methods: {

    // 用户登录
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.$axios.post('auth/login', {
            username: this.loginForm.username,
            password: this.loginForm.password,
            captchaId: this.loginForm.captchaId,
            captchaValue: this.loginForm.captchaValue,
          }).then((response) => {
            console.log(response)
            localStorage.setItem("token", response.data.data.token)
            localStorage.setItem("sid", response.data.data.user)
            ElMessage({ message: '欢迎回来'+response.data.data.user, type: 'success'})
            this.$router.push('/category');
          }).catch((error) => {
            console.log(error);
          })
        }
      });
      // this.getCaptcha();
    },
    getUserInfo() {
      this.$axios.get('/auth/info').then(response => {
        localStorage.setItem("sid", response.data.data.username)
        console.log(response.data.data)
      })
    }

    // 获取验证码
    // getCaptcha() {
    //   this.$axios.get('/captcha').then(response => {
    //     this.loginForm.captchaId = response.data.data.captchaId
    //     this.captchaImg = response.data.data.captchaImg
    //     this.loginForm.captchaValue = ''
    //   })
    // }
  }
}
</script>

<style scoped>
.main {
  width: 100%;
  height: 100vh;
  display: flex;
  background-image: url("../assets/back.png");
  background-size: 100% 60%;
  background-repeat: no-repeat;
  /* background-color: #e8ecf1; */
  background-color: #F5F7FA;
}
.form {
  width: 25%;
  height: 50%;
  margin: 150px auto;
  text-align: center;
  padding: 20px;
  background-color: white;
  border-radius: 15px;
  box-shadow: 0px 0px 1px #F2F6FC;
}
.logo {
  width: 80px;
  height: 80px;
  margin: 20px;
  border-radius: 15px;
  box-shadow: 0px 0px 12px #ebeff4;
}
.captchaImg {
  width: 45%;
  height: 40px;
  margin-left: 5%;
  float: left;
  border-radius: 5px;
  background-color: #ecf5ff;
}
.button {
  width: 100%;
  margin-top: 5px;
}

.footer{
  bottom: 0;
  width: 100%;
  height: 80px;
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #606266;
  font-size: 10px;
}

.footer a{
  color: #606266;
}
.footer a:hover{
  color: #2d2d2f;
  transition: 0.5s;
}
</style>