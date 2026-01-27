<template>
  <div id="login-box" :class="{'dark-mode': uiStore.dark}" v-loading="oauthLoading" element-loading-text="登录中...">
    <!-- 背景层：动态渐变+粒子点缀 -->
    <div id="background-wrap" v-if="!settingStore.settings.background">
      <div class="gradient-bg"></div>
      <div class="particles"></div>
      <div class="x1 cloud"></div>
      <div class="x2 cloud"></div>
      <div class="x3 cloud"></div>
      <div class="x4 cloud"></div>
      <div class="x5 cloud"></div>
    </div>
    <div v-else :style="background" class="custom-bg"></div>

    <!-- 表单容器：卡片化+阴影分层 -->
    <div class="form-wrapper">
      <div class="container">
        <!-- 标题区域：添加Logo+渐变文字 -->
        <div class="title-group">
          <div class="logo">
            <Icon icon="mingcute:mail-line" color="#1890ff" width="32" height="32" />
          </div>
          <span class="form-title">{{ settingStore.settings.title }}</span>
        </div>
        
        <span class="form-desc" v-if="show === 'login'">{{ $t('loginTitle') }}</span>
        <span class="form-desc" v-else>{{ $t('regTitle') }}</span>

        <!-- 登录表单 -->
        <div v-show="show === 'login'" class="form-panel">
          <el-input 
            :class="['input-field', settingStore.settings.loginDomain === 0 ? 'email-input' : '']" 
            v-model="form.email"
            type="text" 
            :placeholder="$t('emailAccount')" 
            autocomplete="off"
            :prefix="h(Icon, { icon: 'mingcute:user-line', width: 16, height: 16, class: 'input-icon' })"
          >
            <template #append v-if="settingStore.settings.loginDomain === 0">
              <div @click.stop="openSelect" class="suffix-selector">
                <el-select
                  ref="mySelect"
                  v-model="suffix"
                  :placeholder="$t('select')"
                  class="select"
                >
                  <el-option
                    v-for="item in domainList"
                    :key="item"
                    :label="item"
                    :value="item"
                  />
                </el-select>
                <div class="suffix-display">
                  <span>{{ suffix }}</span>
                  <Icon class="setting-icon" icon="mingcute:down-small-fill" width="16" height="16"/>
                </div>
              </div>
            </template>
          </el-input>

          <el-input 
            v-model="form.password" 
            :placeholder="$t('password')" 
            type="password" 
            autocomplete="off"
            class="input-field"
            :prefix="h(Icon, { icon: 'mingcute:lock-line', width: 16, height: 16, class: 'input-icon' })"
          >
          </el-input>

          <el-button class="btn primary-btn" type="primary" @click="submit" :loading="loginLoading">
            {{ $t('loginBtn') }}
          </el-button>

          <el-button 
            class="btn third-party-btn" 
            v-if="settingStore.settings.linuxdoSwitch"  
            @click="linuxDoLogin"
          >
            <el-avatar src="/image/linuxdo.webp" :size="18" style="margin-right: 8px" />
            <span>LinuxDo 授权登录</span>
          </el-button>
        </div>

        <!-- 注册表单 -->
        <div v-show="show !== 'login'" class="form-panel">
          <el-input 
            class="input-field email-input" 
            v-model="registerForm.email" 
            type="text" 
            :placeholder="$t('emailAccount')"
            autocomplete="off"
            :prefix="h(Icon, { icon: 'mingcute:user-line', width: 16, height: 16, class: 'input-icon' })"
          >
            <template #append>
              <div @click.stop="openSelect" class="suffix-selector">
                <el-select
                  ref="mySelect"
                  v-model="suffix"
                  :placeholder="$t('select')"
                  class="select"
                >
                  <el-option
                    v-for="item in domainList"
                    :key="item"
                    :label="item"
                    :value="item"
                  />
                </el-select>
                <div class="suffix-display">
                  <span>{{ suffix }}</span>
                  <Icon class="setting-icon" icon="mingcute:down-small-fill" width="16" height="16"/>
                </div>
              </div>
            </template>
          </el-input>

          <el-input 
            v-model="registerForm.password" 
            :placeholder="$t('password')" 
            type="password" 
            autocomplete="off"
            class="input-field"
            :prefix="h(Icon, { icon: 'mingcute:lock-line', width: 16, height: 16, class: 'input-icon' })"
          />

          <el-input 
            v-model="registerForm.confirmPassword" 
            :placeholder="$t('confirmPwd')" 
            type="password"
            autocomplete="off"
            class="input-field"
            :prefix="h(Icon, { icon: 'mingcute:check-circle-line', width: 16, height: 16, class: 'input-icon' })"
          />

          <el-input 
            v-if="settingStore.settings.regKey === 0" 
            v-model="registerForm.code" 
            :placeholder="$t('regKey')"
            type="text" 
            autocomplete="off"
            class="input-field"
            :prefix="h(Icon, { icon: 'mingcute:key-line', width: 16, height: 16, class: 'input-icon' })"
          />

          <el-input 
            v-if="settingStore.settings.regKey === 2" 
            v-model="registerForm.code"
            :placeholder="$t('regKeyOptional')" 
            type="text" 
            autocomplete="off"
            class="input-field"
            :prefix="h(Icon, { icon: 'mingcute:key-line', width: 16, height: 16, class: 'input-icon' })"
          />

          <div 
            v-show="verifyShow"
            class="register-turnstile"
            :data-sitekey="settingStore.settings.siteKey"
            data-callback="onTurnstileSuccess"
            data-error-callback="onTurnstileError"
            data-after-interactive-callback="loadAfter"
            data-before-interactive-callback="loadBefore"
          >
            <span style="font-size: 12px;color: #F56C6C" v-if="botJsError">{{ $t('verifyModuleFailed') }}</span>
          </div>

          <el-button class="btn primary-btn" type="primary" @click="submitRegister" :loading="registerLoading">
            {{ $t('regBtn') }}
          </el-button>

          <el-button 
            class="btn third-party-btn" 
            v-if="settingStore.settings.linuxdoSwitch"  
            @click="linuxDoLogin"
          >
            <el-avatar src="/image/linuxdo.webp" :size="18" style="margin-right: 8px" />
            <span>LinuxDo 授权注册</span>
          </el-button>
        </div>

        <!-- 切换按钮：美化hover效果 -->
        <template v-if="settingStore.settings.register === 0">
          <div 
            class="switch-btn" 
            @click="show = 'register'" 
            v-if="show === 'login'"
            @mouseenter="switchHover = true"
            @mouseleave="switchHover = false"
          >
            {{ $t('noAccount') }}
            <span :class="switchHover ? 'switch-text-active' : 'switch-text'">{{ $t('regSwitch') }}</span>
          </div>
          <div 
            class="switch-btn" 
            @click="show = 'login'" 
            v-else
            @mouseenter="switchHover = true"
            @mouseleave="switchHover = false"
          >
            {{ $t('hasAccount') }} 
            <span :class="switchHover ? 'switch-text-active' : 'switch-text'">{{ $t('loginSwitch') }}</span>
          </div>
        </template>
      </div>
    </div>

    <!-- 绑定弹窗：美化样式 -->
    <el-dialog 
      class="bind-dialog" 
      v-model="showBindForm"  
      title="注册邮箱绑定"
      :center="true"
      :width="isMobile ? '90%' : '400px'"
    >
      <div class="bind-container">
        <el-input 
          v-model="bindForm.email" 
          type="text" 
          :placeholder="$t('emailAccount')" 
          autocomplete="off"
          class="input-field"
          :prefix="h(Icon, { icon: 'mingcute:mail-line', width: 16, height: 16, class: 'input-icon' })"
        >
          <template #append>
            <div @click.stop="openSelect" class="suffix-selector">
              <el-select
                ref="mySelect"
                v-model="suffix"
                :placeholder="$t('select')"
                class="select"
              >
                <el-option
                  v-for="item in domainList"
                  :key="item"
                  :label="item"
                  :value="item"
                />
              </el-select>
              <div class="suffix-display">
                <span>{{ suffix }}</span>
                <Icon class="setting-icon" icon="mingcute:down-small-fill" width="16" height="16"/>
              </div>
            </div>
          </template>
        </el-input>

        <el-input 
          v-if="settingStore.settings.regKey === 0" 
          v-model="bindForm.code" 
          :placeholder="$t('regKey')"
          type="text" 
          autocomplete="off"
          class="input-field"
          :prefix="h(Icon, { icon: 'mingcute:key-line', width: 16, height: 16, class: 'input-icon' })"
        />

        <el-input 
          v-if="settingStore.settings.regKey === 2" 
          v-model="bindForm.code"
          :placeholder="$t('regKeyOptional')" 
          type="text" 
          autocomplete="off"
          class="input-field"
          :prefix="h(Icon, { icon: 'mingcute:key-line', width: 16, height: 16, class: 'input-icon' })"
        />

        <el-button class="btn primary-btn" type="primary" @click="bind" :loading="bindLoading">
          确认绑定
        </el-button>
      </div>
    </el-dialog>

    <!-- GitHub按钮：悬浮动效 -->
    <a class="github-btn" href="http:/xtby.xyz" target="_blank" rel="noopener noreferrer">
      <Icon icon="mingcute:github-line" color="#1890ff" width="22" height="22" />
    </a>
  </div>
</template>

<script setup>
import router from "@/router";
import {computed, nextTick, reactive, ref, watch, onMounted} from "vue";
import {login} from "@/request/login.js";
import {register} from "@/request/login.js";
import {isEmail} from "@/utils/verify-utils.js";
import {useSettingStore} from "@/store/setting.js";
import {useAccountStore} from "@/store/account.js";
import {useUserStore} from "@/store/user.js";
import {useUiStore} from "@/store/ui.js";
import {Icon} from "@iconify/vue";
import {cvtR2Url} from "@/utils/convert.js";
import {loginUserInfo} from "@/request/my.js";
import {permsToRouter} from "@/perm/perm.js";
import {useI18n} from "vue-i18n";
import {oauthBindUser, oauthLinuxDoLogin} from "@/request/ouath.js";

const {t, locale} = useI18n();
const accountStore = useAccountStore();
const userStore = useUserStore();
const uiStore = useUiStore();
const settingStore = useSettingStore();

// 原有状态保持不变
const loginLoading = ref(false)
const bindLoading = ref(false)
const oauthLoading = ref(false);
const showBindForm = ref(false);
const show = ref('login')
const switchHover = ref(false)
const isMobile = ref(false)
const bindForm = reactive({
  email: '',
  oauthUserId: '',
  code: ''
})
const form = reactive({
  email: '',
  password: '',
});
const mySelect = ref()
const suffix = ref('')
const registerForm = reactive({
  email: '',
  password: '',
  confirmPassword: '',
  code: null
})
const domainList = settingStore.domainList;
const registerLoading = ref(false)
suffix.value = domainList[0] || ''
const verifyShow = ref(false)
let verifyToken = ''
let turnstileId = null
let botJsError = ref(false)
let verifyErrorCount = 0

// 新增：检测设备类型
onMounted(() => {
  checkIsMobile()
  window.addEventListener('resize', checkIsMobile)
})

function checkIsMobile() {
  isMobile.value = window.innerWidth < 768
}

// 原有方法保持不变（省略重复代码，仅保留新增/修改部分）
window.onTurnstileSuccess = (token) => {
  verifyToken = token;
};
window.onTurnstileError = (e) => {
  if (verifyErrorCount >= 4) return
  verifyErrorCount++
  console.warn('人机验加载失败', e)
  setTimeout(() => {
    nextTick(() => {
      if (!turnstileId) {
        turnstileId = window.turnstile.render('.register-turnstile')
      } else {
        window.turnstile.reset(turnstileId);
      }
    })
  }, 1500)
};
window.loadAfter = (e) => console.log('loadAfter')
window.loadBefore = (e) => console.log('loadBefore')

const loginOpacity = computed(() => {
  const opacity = settingStore.settings.loginOpacity
  return uiStore.dark ? `rgba(17, 25, 40, ${opacity})` : `rgba(255, 255, 255, ${opacity})`
})

const background = computed(() => {
  return settingStore.settings.background ? {
    'background-image': `url(${cvtR2Url(settingStore.settings.background)})`,
    'background-repeat': 'no-repeat',
    'background-size': 'cover',
    'background-position': 'center',
    'background-blur': '8px'
  } : ''
})

const openSelect = () => {
  mySelect.value?.toggleMenu()
}

// 其余原有方法（linuxDoLogin、linuxDoGetUser、bind、submit、saveToken、submitRegister）保持不变
</script>

<style>
.el-select-dropdown__item {
  padding: 0 15px;
}
.no-autofill-pwd {
  .el-input__inner {
    -webkit-text-security: disc !important;
  }
}
/* 滚动条美化 */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}
::-webkit-scrollbar-track {
  background: transparent;
}
::-webkit-scrollbar-thumb {
  background: rgba(144, 147, 153, 0.3);
  border-radius: 3px;
}
::-webkit-scrollbar-thumb:hover {
  background: rgba(144, 147, 153, 0.5);
}
</style>

<style lang="scss" scoped>
// 根容器：优化渐变背景
#login-box {
  font: 100% Arial, sans-serif;
  height: 100vh;
  margin: 0;
  padding: 0;
  overflow: hidden;
  position: relative;
  transition: background-color 0.5s ease;

  &.dark-mode {
    background-color: #111928;
  }
}

// 背景层优化
#background-wrap {
  height: 100%;
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 0;
  overflow: hidden;
}

// 渐变背景
.gradient-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #e8f4f8 0%, #f0f8fb 100%);
  transition: background 0.5s ease;

  .dark-mode & {
    background: linear-gradient(135deg, #111928 0%, #1f2937 100%);
  }
}

// 粒子点缀（动态效果）
.particles {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: 
    radial-gradient(circle, rgba(24, 144, 255, 0.1) 1px, transparent 1px),
    radial-gradient(circle, rgba(24, 144, 255, 0.05) 1px, transparent 1px);
  background-size: 50px 50px;
  background-position: 0 0, 25px 25px;
  animation: float 20s linear infinite;

  .dark-mode & {
    background-image: 
      radial-gradient(circle, rgba(24, 144, 255, 0.05) 1px, transparent 1px),
      radial-gradient(circle, rgba(24, 144, 255, 0.02) 1px, transparent 1px);
  }
}

// 自定义背景图样式
.custom-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  filter: brightness(0.9) saturate(1.1);
  transition: filter 0.5s ease;

  .dark-mode & {
    filter: brightness(0.6) saturate(0.9);
  }
}

// 表单容器：居中+卡片化
.form-wrapper {
  position: relative;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  padding: 20px;
}

.container {
  background: v-bind(loginOpacity);
  backdrop-filter: blur(12px);
  border-radius: 16px;
  padding: 30px 25px;
  width: 100%;
  max-width: 450px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.1);
  transition: all 0.3s ease;

  .dark-mode & {
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
    border-color: rgba(255, 255, 255, 0.05);
  }

  @media (max-width: 767px) {
    padding: 25px 20px;
    margin: 0;
  }
}

// 标题组：Logo+文字
.title-group {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 8px;

  .logo {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 40px;
    height: 40px;
    border-radius: 12px;
    background-color: rgba(24, 144, 255, 0.1);
    transition: background-color 0.3s ease;

    .dark-mode & {
      background-color: rgba(24, 144, 255, 0.08);
    }
  }

  .form-title {
    font-weight: 600;
    font-size: 24px !important;
    background: linear-gradient(90deg, #1890ff, #69c0ff);
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
  }
}

// 描述文字
.form-desc {
  margin-bottom: 25px;
  color: var(--form-desc-color);
  font-size: 14px;
  transition: color 0.3s ease;

  .dark-mode & {
    color: rgba(255, 255, 255, 0.7);
  }
}

// 表单面板：间距优化
.form-panel {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-bottom: 25px;
}

// 输入框美化
.input-field {
  height: 44px;
  width: 100%;
  border-radius: 12px !important;
  transition: all 0.3s ease;

  :deep(.el-input__wrapper) {
    border-radius: 12px;
    background-color: rgba(255, 255, 255, 0.8) !important;
    border: 1px solid rgba(220, 223, 230, 0.3);
    transition: all 0.3s ease;

    .dark-mode & {
      background-color: rgba(31, 41, 59, 0.6) !important;
      border-color: rgba(255, 255, 255, 0.05);
    }

    &:focus-within {
      border-color: #1890ff !important;
      box-shadow: 0 0 0 3px rgba(24, 144, 255, 0.15);
    }
  }

  :deep(.el-input__inner) {
    height: 42px;
    font-size: 14px;
    color: var(--el-text-color-primary);

    .dark-mode & {
      color: rgba(255, 255, 255, 0.9);
    }
  }

  .input-icon {
    color: #909399;
    transition: color 0.3s ease;

    :deep(.el-input__wrapper):focus-within & {
      color: #1890ff;
    }
  }
}

// 邮箱后缀选择器
.suffix-selector {
  display: flex;
  align-items: center;
  cursor: pointer;

  .suffix-display {
    display: flex;
    align-items: center;
    gap: 4px;
    color: var(--el-text-color-primary);
    font-size: 14px;
  }

  .setting-icon {
    position: relative;
    top: 2px;
  }
}

// 按钮美化
.btn {
  height: 44px;
  border-radius: 12px !important;
  font-size: 15px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.primary-btn {
  background: linear-gradient(90deg, #1890ff, #4dabff) !important;
  border: none !important;
  box-shadow: 0 4px 15px rgba(24, 144, 255, 0.2);

  &:hover {
    background: linear-gradient(90deg, #0f7ae5, #3a9dff) !important;
    box-shadow: 0 6px 20px rgba(24, 144, 255, 0.3);
  }

  &:active {
    box-shadow: 0 2px 8px rgba(24, 144, 255, 0.2);
  }
}

.third-party-btn {
  background-color: rgba(255, 255, 255, 0.8) !important;
  color: var(--el-text-color-primary) !important;
  border: 1px solid rgba(220, 223, 230, 0.3) !important;

  .dark-mode & {
    background-color: rgba(31, 41, 59, 0.6) !important;
    color: rgba(255, 255, 255, 0.9) !important;
    border-color: rgba(255, 255, 255, 0.05) !important;
  }

  &:hover {
    background-color: rgba(255, 255, 255, 0.9) !important;

    .dark-mode & {
      background-color: rgba(45, 55, 72, 0.6) !important;
    }
  }
}

// 切换按钮
.switch-btn {
  margin-top: 15px;
  text-align: center;
  font-size: 14px;
  color: var(--form-desc-color);
  transition: color 0.3s ease;
  cursor: pointer;

  .dark-mode & {
    color: rgba(255, 255, 255, 0.7);
  }

  .switch-text {
    color: var(--login-switch-color);
    margin-left: 4px;
    transition: all 0.3s ease;
    text-decoration: underline;
    text-underline-offset: 2px;
  }

  .switch-text-active {
    color: #0f7ae5;
    text-decoration: underline;
    text-underline-offset: 4px;
  }

  &:hover {
    color: var(--el-text-color-primary);

    .dark-mode & {
      color: rgba(255, 255, 255, 0.9);
    }
  }
}

// 绑定弹窗
:deep(.bind-dialog) {
  border-radius: 16px !important;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.15);

  .dark-mode & {
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
  }

  .el-dialog__header {
    padding-bottom: 15px;

    .el-dialog__title {
      font-size: 18px;
      font-weight: 500;
    }
  }

  .el-dialog__body {
    padding: 0 20px 20px;
  }
}

.bind-container {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

// GitHub按钮：悬浮动效
.github-btn {
  position: fixed;
  width: 44px;
  height: 44px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.8);
  bottom: 20px;
  right: 20px;
  z-index: 1000;
  border: 1px solid rgba(220, 223, 230, 0.3);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  cursor: pointer;
  transition: all 0.3s ease;

  .dark-mode & {
    background-color: rgba(31, 41, 59, 0.6);
    border-color: rgba(255, 255, 255, 0.05);
  }

  &:hover {
    transform: translateY(-3px);
    box-shadow: 0 6px 16px rgba(24, 144, 255, 0.15);
    background-color: rgba(255, 255, 255, 0.95);

    .dark-mode & {
      background-color: rgba(45, 55, 72, 0.8);
      box-shadow: 0 6px 16px rgba(24, 144, 255, 0.1);
    }
  }

  &:active {
    transform: translateY(-1px);
  }
}

// 云朵动画优化
@keyframes animateCloud {
  0% { margin-left: -500px; }
  100% { margin-left: 100%; }
}

@keyframes float {
  0% { background-position: 0 0, 25px 25px; }
  100% { background-position: 50px 50px, 75px 75px; }
}

.x1 { animation: animateCloud 30s linear infinite; transform: scale(0.65); opacity: 0.8; }
.x2 { animation: animateCloud 15s linear infinite; transform: scale(0.3); opacity: 0.6; }
.x3 { animation: animateCloud 25s linear infinite; transform: scale(0.5); opacity: 0.7; }
.x4 { animation: animateCloud 13s linear infinite; transform: scale(0.4); opacity: 0.5; }
.x5 { animation: animateCloud 20s linear infinite; transform: scale(0.55); opacity: 0.6; }

.cloud {
  background: linear-gradient(to bottom, rgba(255,255,255,0.9) 5%, rgba(241,241,241,0.9) 100%);
  border-radius: 100px;
  box-shadow: 0 8px 5px rgba(0, 0, 0, 0.05);
  height: 120px;
  width: 350px;
  position: relative;

  .dark-mode & {
    background: linear-gradient(to bottom, rgba(255,255,255,0.1) 5%, rgba(241,241,241,0.08) 100%);
    box-shadow: 0 8px 5px rgba(0, 0, 0, 0.1);
  }

  &:after, &:before {
    content: "";
    position: absolute;
    background: rgba(255,255,255,0.9);
    z-index: -1;

    .dark-mode & {
      background: rgba(255,255,255,0.1);
    }
  }

  &:after {
    border-radius: 100px;
    height: 100px;
    left: 50px;
    top: -50px;
    width: 100px;
  }

  &:before {
    border-radius: 200px;
    height: 180px;
    width: 180px;
    right: 50px;
    top: -90px;
  }
}
</style>

