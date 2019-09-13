# Vue-i18n 单独使用

### 在做表单校验的时候，validator通常为一个函数，为了复用这些函数，一般都会将其放到一个单独的js文件里维护
```
**/validator.ts
```
大概长这样
```js
export const validateAccountNumber = function(_: object, value: string, callback: Function) {
  if (!value) {
    return callback(new Error('xxx'));
  } else if (!/^[0-9a-zA-Z]{6,16}$/.test(value)) {
    callback('xxx');
  } else {
    callback();
  }
};
```

如果要加上多语言的话，由于vue-i18n插件必须从vue实例上获取$t来进行翻译，但是在单独的js文件里是拿不到vue实例的，所以不能直接用 $t 或者 this.$t。

---

一开始我想的是把vue实例在每个单文件组件里传过去
```js
import {validateAccountNumber} from 'xxx'
methods: {
    validator(...arg) {
        validateAccountNumber.call(this, ...arg)
    }
}

export const validateAccountNumber = function(_: object, value: string, callback: Function) {
  if (!value) {
    return callback(new Error(this.$t('xxx')));
  } else if (!/^[0-9a-zA-Z]{6,16}$/.test(value)) {
    callback(this.$t('xxx'));
  } else {
    callback();
  }
};
```

但是这样每个validator都要包一层函数，实在繁琐。后来百度了一些方法，比如在里面单独new一个Vue 实例用来挂载i18n实例，这个方法是可行的，但是要注意一点，切换语言后，要重新把vue实例化一次，要不然locale是上一次的不会改变。所以最终代码如下：
```js
import Vue from 'vue'
import VueI18n from 'vue-i18n'
import store from '@/store'

const messages = {
  'zh-CN': {
    cantBeEmpty: '不能为空',
  },
  'en-US': {
    cantBeEmpty: `Can't be empty`,
  }
}

function initI18n() {
  const i18n = new VueI18n({
    locale: (store as any).state.app.local, // 设置地区
    messages, // 设置地区信息
  })
  return new Vue({ i18n });
}

export const validateAccountNumber = function(_: object, value: string, callback: Function) {
  // 在函数里new 保证locale是最新的 
  const vueInstance = initI18n();
  if (!value) {
    return callback(new Error(String(vueInstance.$t('cantBeEmpty'))));
  } else if (!/^[0-9a-zA-Z]{6,16}$/.test(value)) {
    callback(vueInstance.$t('stringLen6To16'));
  } else {
    callback();
  }
};
```
