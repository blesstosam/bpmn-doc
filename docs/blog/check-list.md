# 代码审查检查项


#### 1. 结合 gitlab 做code review，之前公司没有这种做法，但是 review 作为一种能发现很多bug（根据《代码大全》统计15%左右的bug在review阶段出现）的手段，review是代码生产上必不可少的一环，一定要做起来。  
目前来看还是以我一个人review为主，后面慢慢走向互相review。互相review代码也是让团队每个人都能提高的一个手段。

---
#### 2. Code review checklist
* 检查文件命名，文件夹和文件命名用横杠连接，vue组件用首字母驼峰法
```
/api-user/my-user.js
/components/HelloWorld.vue
```

* 检查变量命名  
>标准变量：驼峰法  
常量：大写，单词中间用下划线  
构造函数或类：首字母大写驼峰法  
布尔值类型：以is/has/can/should等开头的驼峰法  
类的成员：公有属性和方法同上，私有属性和方法在前面加下划线

* 避免回调地狱
```js
// bad
function submitForm(values, cb) {
  validateForm(values, (err) => {
    if (err) {
      return cb(err, null);
    }

    makeRequest(values, (err, resp) => {
      if (err) {
        return cb(err, null);
      }

      makeAnotherRequest(resp, (err, anotherResp) => {
        if (err) {
          return cb(err, null);
        }

        return cb(null, wrapResult(anotherResp));
      })
    })
  })
}

// good
const validateFormPromise = promisify(validateForm);
const makeRequestPromise = promisify(makeRequest);
const makeAnotherRequestPromise = promisify(makeAnotherRequest);

async function submitForm(values) {
  try {
    await validateFormPromise(values);
    const resp = await makeRequestPromise(values);
    const anotherResp = await makeAnotherRequestPromise(resp);
    return wrapResult(anotherResp);
  } catch (e) {
    throw e;
  }
}
```
* 尽量先用return 避免过多if else 判断 
```js
// bad
function someFunction(someCondition) {
  if (someCondition) {
      // Do Something
  }
}
// good
function someFunction(someCondition) {
  if (!someCondition) {
      return;
  }
  // Do Something
}

```
* 避免过多层三元表达式
```js
// bad
const sampleComponent = () => {
    return {flag && flag2 && !flag3
        ? flag4
        ? 'foo'
        : flag5
        ? 'foo1'
        : 'foo2'
        : 'foo3'
      }
};
// good
const sampleComponent = () => {
    if (flag && flag2 && !flag3) {
        if (flag4) {
            return 'foo'
        } else if (flag5) {
            return 'foo1'
        } else {
            return 'foo2'
        }
    } else {
        return 'foo3'
    }
};
```
* 如果是需要多语言支持的话，从一开始就加上i18n的翻译
```html
// bad
<h1>My Awesome Project</h1>
<p>Create like a god. Command like a king. Work like a slave.</p>

// good
<h1>$t('title')</h1>
<p>$t('subTitle')</p>
```
* 避免使用全局css，比如 *选择器，或样式文件
* 一些操作可能会导致程序挂掉的地方，必须使用try catch
```js
// bad
const resp = await ajax()
let obj = JSON.parse(resp.data)

// good
const resp = await ajax()
try {
  let obj = JSON.parse(resp.data)  
} catch(err) {
    console.log(err)
}
```

* 过多的 if esle 用 switch 替代，代码更加易懂
* Dont repeat yourself, 碰到多处使用的代码一定要封装起来
* 在写代码的过程中碰到不合理或者代码数量庞大的要重构

