<script setup lang="ts">
import {ref} from "vue";
import {openConfig, getConfig, saveConfig} from "../cli/config";
import {proxySelectedOption,githubSelectedOption,languageOptions} from "../cli/default";
import {useToast} from "vue-toastification";
import {t, i18nSetLocale} from "../cli/i18n";
import {router} from "../cli/router";

const config = ref(getConfig())
const languageSelected = ref(config.value.languageSelected ?? "auto")
const proxyCustom = ref(config.value.proxyCustom ?? "")
const githubCustom = ref(config.value.githubCustom ?? "")
const proxyCustomIsOpen = ref(false)
const githubCustomIsOpen = ref(false)
const proxySelected = ref(config.value.proxySelected ?? "none")
const githubSelected = ref(config.value.githubSelected ?? "none")
console.log('proxyCustom', proxyCustom.value)

function reGetConfig() {
  languageSelected.value = config.value.languageSelected ?? "auto"
  proxyCustom.value = config.value.proxyCustom ?? "http://127.0.0.1:10809"
  githubCustom.value = config.value.githubCustom ?? "https://mirror.ghproxy.com"
  proxyCustomIsOpen.value = false
  githubCustomIsOpen.value = false
  proxySelected.value = config.value.proxySelected ?? "none"
  githubSelected.value = config.value.githubSelected ?? "none"
}

function languageSelectedUpdate(value: string) {
  languageSelected.value = value
  config.value.languageSelected = value
  config.value = saveConfig(config.value)
  reGetConfig()
  if (value === "auto") {
    value = navigator.language
  }
  i18nSetLocale(value)
  router.go(0)
}

function proxySelectedUpdate(key: string, open: boolean, custom: boolean) {
  console.log(`proxySelectedUpdate key [${key}] value [${proxyCustom.value}]`)
  proxySelected.value = key
  config.value.proxySelected = proxySelected.value
  if (custom) {
    proxyCustomIsOpen.value = open
    return
  }
  console.log(key)
  if (key === "custom") {
    proxyCustom.value = proxyCustom.value.trim()
    console.log(`proxySelectedUpdate custom key [${key}] value [${proxyCustom.value}]`)
    if (proxyCustom.value === "") {
      proxyCustom.value = "http://127.0.0.1:1080"
    }
    setTimeout(() => {
      proxyCustomIsOpen.value = open
    }, 500)
  } else {
    setTimeout(() => {
      proxyCustomIsOpen.value = false
    }, 500)
  }
  config.value.proxyCustom = proxyCustom.value
  config.value = saveConfig(config.value)
  console.log('proxySelectedUpdate save', config.value)
  reGetConfig()
}

function githubSelectedUpdate(key: string, open: boolean, custom: boolean) {
  console.debug(`githubSelectedUpdate key [${key}] value [${githubCustom.value}]`)
  githubSelected.value = key
  config.value.githubSelected = githubSelected.value
  if (custom) {
    githubCustomIsOpen.value = open
    return
  }
  if (key === "custom") {
    githubCustom.value = githubCustom.value.trim()
    console.log(`githubSelectedUpdate custom key [${key}] value [${proxyCustom.value}]`)
    if (githubCustom.value === "") {
      githubCustom.value = "https://mirror.ghproxy.com"
    }
    setTimeout(() => {
      githubCustomIsOpen.value = open
    }, 500)
  } else {
    setTimeout(() => {
      githubCustomIsOpen.value = false
    }, 500)
  }
  config.value.githubCustom = githubCustom.value
  config.value = saveConfig(config.value)
  console.log('githubSelectedUpdate save', config.value)
  reGetConfig()
}

</script>

<template>
  <div class="page-container">
    <div class="page">
      <header class="header">
        <h1 class="md:text-xl">{{ t("setting") }}</h1>
        <div class="flex flex-auto items-center justify-end">
          <div class="text-shadow-primary text-primary-600 cursor-pointer text-sm">
            <div @click="openConfig">{{ t("configFile") }}</div>
          </div>
        </div>
      </header>
      <div class="card settings-card">
        <div class="flex flex-wrap">
          <div class="flex w-full items-center justify-between py-3 px-8 md:w-1/2">
            <span class="label font-bold">{{ t('language') }}</span>
            <div class="button-select">
              <button v-for="item in languageOptions"
                      :key="item.key"
                      :value="item.key"
                      :class="{ 'button-select-options': true, 'actived': languageSelected === item.key }"
                      @click="languageSelectedUpdate(item.key)"
              > {{ item.text }}
              </button>
            </div>
          </div>
        </div>
        <!-- TODO APP 更新加速渠道应该只设置一个 可以考虑 CDN JS 更新 也可以考虑使用 加速代理 也可以考虑使用 软件代理 -->
        <div class="flex flex-wrap">
          <div class="flex w-full items-center justify-between py-3 px-8 md:w-1/2">
            <span class="label font-bold">{{ t('proxy') }}</span>
            <div class="button-select">
              <button v-for="item in proxySelectedOption"
                      :key="item.key"
                      :value="item.key"
                      :class="{ 'button-select-options': true, 'actived': proxySelected === item.key }"
                      @click="proxySelectedUpdate(item.key,!proxyCustomIsOpen,item.key==='custom')">
                {{ item.text }}
              </button>
              <input class="Selected" type="url"
                     v-model="proxyCustom"
                     v-show="proxyCustomIsOpen"
                     style="margin-left: 5px"
                     @blur="proxySelectedUpdate('custom',false,false)"
                     @keydown.enter="proxySelectedUpdate('custom',false,false)"
                     :placeholder="proxyCustom??'https://'">
            </div>
          </div>
        </div>
        <div class="flex flex-wrap">
          <div class="flex w-full items-center justify-between py-3 px-8 md:w-1/2">
            <span class="label font-bold">{{ t('githubProxy') }}</span>
            <div class="button-select">
              <button v-for="item in githubSelectedOption"
                      :key="item.key"
                      :value="item.key"
                      :class="{ 'button-select-options': true, 'actived': githubSelected === item.key }"
                      @click="githubSelectedUpdate(item.key,!githubCustomIsOpen,item.key==='custom')">
                {{ item.text }}
              </button>
              <input class="Selected" type="url"
                     v-model="githubCustom"
                     v-show="githubCustomIsOpen"
                     style="margin-left: 5px"
                     @blur="githubSelectedUpdate('custom',false,false)"
                     @keydown.enter="githubSelectedUpdate('custom',false,false)"
                     :placeholder="githubCustom??'https://'">
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.header {
  margin: 10px 0;
  width: 100%;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  user-select: none
}

.header > h1 {
  flex-shrink: 0;
  font-size: 24px;
  color: #2c8af8;
  font-weight: 500;
  text-shadow: 0 2px 6px rgba(44, 138, 248, .4);
  line-height: 32px
}

.switch {
  display: inline-block;
  width: 32px;
  height: 16px;
  border-radius: 8px;
  background-color: #d8dee2;
  transition: background-color .3s ease;
  position: relative;
  cursor: pointer
}

.switch.checked {
  background-color: #57befc
}

.switch.checked:after {
  transform: translate(16px)
}

.switch:after {
  content: "";
  position: absolute;
  top: -1px;
  height: 18px;
  width: 18px;
  border-radius: 9px;
  background-color: #fff;
  box-shadow: 0 0 8px #2c8af866;
  transition: transform .3s ease;
  transform: translate(-2px)
}

.switch-icon {
  position: absolute;
  transform: translate(-1px) scale(.4);
  color: #fff;
  line-height: 16px
}

.card {
  padding: 15px;
  box-shadow: 2px 5px 20px -3px #2c8af82e;
  background-color: #fff;
  border-radius: 4px
}

.button-select {
  display: flex;
  flex-direction: row
}

.button-select .button-select-options {
  height: 30px;
  padding: 0 15px;
  color: #54759a;
  font-size: 12px;
  line-height: 30px;
  background: #fff;
  border: 1px solid #e4eaef;
  border-right: none;
  transition: all .3s ease;
  cursor: pointer;
  outline: 0;
  display: block
}

.button-select .button-select-options:hover {
  border: #FF6900 1px solid;
}

.button-select .button-select-options:first-child {
  border-radius: 3px 0 0 3px
}

.button-select .button-select-options:last-child {
  border-radius: 0 3px 3px 0;
  border-right: 1px solid #e4eaef
}

.button-select .button-select-options.actived {
  background: #57befc;
  color: #fff;
  border-color: #57befc;
  box-shadow: 0 2px 5px #57befc80
}

.button-select .button-select-options.actived:active {
  box-shadow: none
}

.button-select > input {
  background: #ffffff;
}

.input {
  display: inline-block;
  height: 30px;
  width: 100%;
  padding: 0 10px;
  font-size: 14px;
  color: #54759a;
  border-radius: 3px;
  border: 1px solid #e4eaef;
  transition: all .3s;
  transition-property: border-color, color, box-shadow
}

.input:focus {
  outline: 0;
  border-color: #57befc;
  color: #2c8af8;
  box-shadow: 0 2px 5px #57befc80
}

@keyframes spinner {
  0% {
    transform: rotate(0)
  }

  to {
    transform: rotate(360deg)
  }
}

.settings-card {
  margin-top: 10px;
  padding: .75rem 0
}

.settings-card .label {
  font-size: 14px;
  color: #54759a
}

.settings-card .external-controller {
  font-size: 14px;
  color: #54759a;
  display: flex;
  justify-content: flex-end;
  font-weight: 400;
  line-height: 17px
}

*, * :before, * :after {
  margin: 0;
  padding: 0;
  box-sizing: inherit
}

.page-container {
  width: 100%;
  height: 100vh;
  padding-left: 10px;
  overflow-y: scroll
}

.page {
  padding: 20px 35px 30px 20px;
  width: 100%;
  min-height: 100vh;
  margin: 0 auto;
  display: flex;
  flex-direction: column
}

input {
  -webkit-appearance: none
}

@media (max-width: 768px) {

  .page-container {
    width: 100%;
    padding: 0 10px;
    height: calc(100vh - 60px);
    -webkit-overflow-scrolling: touch
  }

  .page-container::-webkit-scrollbar {
    display: none
  }

  .page {
    padding: 0 0 20px;
    height: 100%;
    min-height: unset
  }
}

@font-face {
  font-family: clash-iconfont;
  src: url(//at.alicdn.com/t/font_841708_ok9czskbhel.ttf?t=1576162884356) format("truetype")
}

.clash-iconfont {
  font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Helvetica Neue, Arial, Noto Sans, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", Segoe UI Symbol, "Noto Color Emoji";
  font-size: 14px;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c8af8
}

.icon-check:before {
  content: "\e606"
}

button, input {
  font-family: inherit;
  font-size: 100%;
  line-height: 1.15;
  margin: 0;
  padding: 0;
  line-height: inherit;
  color: inherit
}

button {
  text-transform: none;
  background-color: transparent;
  background-image: none
}

button, [type=button] {
  -webkit-appearance: button
}

button {
  cursor: pointer
}

h1 {
  font-size: inherit;
  font-weight: inherit
}

input::placeholder {
  opacity: 1;
  color: #9ca3af
}


input::-moz-placeholder {
  opacity: 1;
  color: #9ca3af
}

input:-ms-input-placeholder {
  opacity: 1;
  color: #9ca3af
}

input::-ms-input-placeholder {
  opacity: 1;
  color: #9ca3af
}

h1 {
  margin: 0
}

.flex {
  display: -webkit-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex
}

.flex-wrap {
  -ms-flex-wrap: wrap;
  -webkit-flex-wrap: wrap;
  flex-wrap: wrap
}

.items-center {
  -webkit-box-align: center;
  -ms-flex-align: center;
  -webkit-align-items: center;
  align-items: center
}

.justify-end {
  -webkit-box-pack: end;
  -ms-flex-pack: end;
  -webkit-justify-content: flex-end;
  justify-content: flex-end
}

.justify-between {
  -webkit-box-pack: justify;
  -ms-flex-pack: justify;
  -webkit-justify-content: space-between;
  justify-content: space-between
}

.flex-auto {
  -webkit-box-flex: 1;
  -ms-flex: 1 1 auto;
  -webkit-flex: 1 1 auto;
  flex: 1 1 auto
}

.font-bold {
  font-weight: 700
}

.py-3 {
  padding-top: .75rem;
  padding-bottom: .75rem
}

.px-8 {
  padding-left: 2rem;
  padding-right: 2rem
}

.text-center {
  text-align: center
}

.w-full {
  width: 100%
}

.w-28 {
  width: 7rem
}

.Selected {
  background-color: #57befc;
  color: #fff;
  border: #e5e7eb 1px solid;
  border-radius: 3px;
  padding: 0 15px;
  height: 30px;
  color: #54759a;
  line-height: 30px;
  font-size: 14px;
  cursor: pointer;
  transition: all .15s ease
}

input:focus {
  outline: #2c8af8 1px solid;
}

.text-shadow-primary {
  text-shadow: 0 0 6px rgb(44 138 248 / 40%);
}

.text-primary-600 {
  color: rgba(44, 138, 248, var(--tw-text-opacity));
  --tw-text-opacity: 1;
}

.cursor-pointer {
  cursor: pointer;
}

.text-sm {
  font-size: .875rem;
  line-height: 1.25rem;
}

.checkbox {
  position: relative;
  display: flex;
  padding-left: 26px;
  line-height: 18px;
  cursor: pointer;
}

.checkbox:before {
  position: absolute;
  top: 50%;
  left: 0;
  display: inline-block;
  width: 18px;
  height: 18px;
  border: 1px solid #e4eaef;
  border-radius: 3px;
  background-color: #fff;
  content: "";
  transition: background-color .3s ease;
  transform: translateY(-9px);
}
</style>