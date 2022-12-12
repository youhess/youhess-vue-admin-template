<template>
  <div class="login-body">
    <div class="login-container">
      <div class="login-left">
        <div class="login-header">
          <h1>YOUHESS</h1>
        </div>
        <form class="login-form">
          <div class="login-form-content">
            <div class="form-item">
              <label for="username">用户名</label>
              <input id="username" v-model="loginForm.username" type="text" />
            </div>
            <div class="form-item">
              <label for="password">密码</label>
              <input
                id="password"
                v-model="loginForm.password"
                type="password"
              />
            </div>
            <div class="form-item checkbox">
              <input id="rememberMeCheckbox" type="checkbox" />
              <label
                id="checkboxLabel"
                for="rememberMeCheckbox"
                class="checkboxLabel"
                >记住密码</label
              >
            </div>
            <button type="submit" @click="handleLogin">登 录</button>
          </div>
        </form>
      </div>
      <div class="login-right">
        <img
          src="@/assets/images/undraw_newspaper_re_syf5.svg"
          alt="undraw_newspaper"
        />
      </div>
    </div>
  </div>
</template>

<script>
// import { validUsername } from '@/utils/validate'

export default {
  name: "Login",
  data() {
    return {
      loginForm: {
        username: "admin",
        password: "111111",
      },
      loading: false,
      passwordType: "password",
      // redirect: undefined
    };
  },
  watch: {
    $route: {
      handler: function (route) {
        this.redirect = route.query && route.query.redirect;
      },
      immediate: true,
    },
  },
  methods: {
    handleLogin() {
      this.loading = true;
      this.$store
        .dispatch("user/login", this.loginForm)
        .then(() => {
          this.$router.push({ path: this.redirect || "/" });
          this.loading = false;
        })
        .catch(() => {
          this.loading = false;
        });
    },
  },
};
</script>

<style lang="scss" scoped>
$bg: #2d3a4b;
$dark_gray: #889aa4;
$light_gray: #eee;

$primary-color: #a531dc;
$secondary-color: #4300b1;

.login-body {
  height: 100vh;
  font-family: "阿里巴巴普惠体 2.0 55 Regular";
  background: linear-gradient(45deg, $primary-color, $secondary-color);
  padding: 50px;
}

.login-container {
  // min-height: calc(100vh - 40px*2);
  height: 100%;
  display: grid;
  grid-template-columns: 50% 50%;
  border-radius: 15px;
  overflow: hidden;
}

.login-left {
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding-left: 100px;
  background-color: white;
}

.login-header {
  h1 {
    font-size: 40px;
    margin-bottom: 24px;
  }
}

.login-form {
  width: 450px;
}

.checkbox {
  display: flex;
  align-items: center;
  gap: 7px;
}

input[type="checkbox"] {
  width: 20px;
  height: 20px;
  // 确认后的样式
  accent-color: $primary-color;
}

.form-item label:not(.checkboxLabel) {
  display: inline-block;
  background-color: white;
  margin-bottom: 10px;
  position: absolute;
  padding: 0 10px;
  transform: translate(25px, -10px);
  font-size: 14px;
}

input[type="text"],
input[type="password"] {
  border: 1px solid black;
  height: 55px;
  // 光标起始点
  padding: 0 2rem;
  width: 100%;
  transition: background 0.5s;
  font-size: 18px;
  outline: none;
  border-radius: 100px;
}

.login-form-content {
  display: flex;
  flex-direction: column;
  gap: 35px;
}

.login-right {
  background-color: #eee;
  display: flex;
  justify-content: center;
  align-items: center;

  img {
    width: 80%;
  }
}

button {
  border: none;
  background: linear-gradient(45deg, $primary-color, $secondary-color);
  padding: 1rem;
  color: white;
  border-radius: 100px;
  text-align: center;
  // 转化成大写
  // text-transform: uppercase;
  font-size: 18px;
  height: 55px;
  cursor: pointer;
}

/*
* responsive
*/
@media (max-width: 1350px) {
  .login-left {
    padding: 50px;
  }
  .login-form {
    width: 100%;
  }
}
@media (max-width: 768px) {
  .login-body {
    padding: 40px;
  }
  .login-container {
    grid-template-columns: auto;
  }
}
</style>
