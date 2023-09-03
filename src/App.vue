<template>
  <div>
    <div v-if="show">
      <select :value="i18nState.locale || 'zh'" @change="onSelectChange">
        <option v-for="item in Object.entries(locales)" :value="item[0]" :key="item[0]">
          {{ item[1] }}
        </option>
      </select>
      <div class="title">{{ t('基础示例') }}</div>
      <div>{{ t('简单的一段描述') }}</div>
      <div>{{ t('{0}是世界上最好的语言？我不信', bestProgramLang) }}</div>
      <div>
        {{ t(
          '这个男人叫{0}，意外获得了超能力，这个女人叫{1}，意外被{2}追杀，这个小孩叫{3}，意外遭遇了意外',
          '小帅',
          '小美',
          'FBI',
          '小白',
        ) }}
      </div>

      <div class="title">{{ t('格式化数字') }}</div>
      <div>{{ t('GitHub全球开发者数量达到了{n0}', 83000000) }}</div>

      <div class="title">{{ t('格式化金额') }}</div>
      <div>{{ t('售价{c0}', 123456.78) }}</div>

      <div class="title">{{ t('格式化日期') }}</div>
      <div>{{ t('今天的日期是{d0}', date) }}</div>

      <div class="title">{{ t('格式化时间') }}</div>
      <div>{{ t('当前时间：{t0}', date) }}</div>

      <div class="title">{{ t('格式化复数') }}</div>
      <div>{{ t('我有{p0个苹果}', 0) }}</div>
      <div>{{ t('我有{p0个苹果}', 1) }}</div>
      <div>{{ t('我有{p0个苹果}', 5) }}</div>
    </div>
    <div v-if="loading" class="loading">
      loading...
    </div>
  </div>
</template>

<script>

export default {
  data() {
    return {
      show: false,
      loading: false,
      bestProgramLang: ['JavaScript', 'Java', 'C', 'C++', 'Python', 'PHP'][
        Math.round(Math.random() * 5)],
      date: new Date(),
      locales: {
        en: 'English',
        zh: '简体中文',
        cht: '繁體中文',
        jp: '日本語',
      },
    }
  },
  methods: {
    async resolveI18n(localeProp) {
      let locale = undefined

      if (localeProp) {
        locale = localeProp
      } else {
        const params = new URLSearchParams(
          new URLSearchParams(window.location.search.slice(1)),
        )
        locale = params.get('locale') || 'en'
      }

      let lang = {}
      if (locale != 'zh') {
        this.loading = true
        let req = await fetch(`../i18n/${locale}.json`)
        try {
          lang = await req.json()
        } catch (error) {
          console.error(error)
        }
        this.loading = false
      }
      this.setI18n({
        langs: {
          [locale]: lang,
        },
        locale,
      })

      this.show = true
    },
    onSelectChange(e) {
      const locale = e.target.value
      if (this.i18nState?.langs?.[locale] || locale == 'zh') {
        this.setI18n({ locale })
      } else {
        this.resolveI18n(locale)
      }
      history.replaceState(null, '', `?locale=${locale}`)
    }
  },
  mounted() {
    this.resolveI18n()
    this.timer = setInterval(() => {
      this.date = new Date();
    }, 1000);
  },
  beforeDestroy() {
    clearInterval(this.timer);
  },
} 
</script>

<style>
.title {
  font-size: larger;
  font-weight: bolder;
  margin: 10px 0 5px 0;
}

.loading {
  position: fixed;
  width: 100vw;
  height: 100vh;
  top: 0;
  left: 0;
  display: flex;
  background-color: rgba(0, 0, 0, 0.3);
  color: aqua;
  justify-content: center;
  align-items: center;
}
</style>
