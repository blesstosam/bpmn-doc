# 前端代码规范

### js规范
1. js, html, css文件命名和文件夹名用小写，多个单词之间用横杠隔开 如
```
/user-center/user-center.html    // 表示 user-center 文件夹下的 user-center.html 文件
/user-center/UserCenter.vue     // .vue 文件还是首字母大写驼峰法
```

2. 不要使用全局变量，有可能被覆盖  
3. 定义变量时，尽量放在最前面  
4. 不要使用 var 定义变量（有变量提升）
5. 拼接字符串尽量用模版字符串语法（不容易出错）
6. 判断条件用全等  
7. 所有的变量尽量做到语义化，方法命名用动词，变量命名用名词，并避免单词长度过长  
* 标准变量：驼峰法  
* 常量：大写，单词中间用下划线  
* dom：小写，单词中间用下划线
* 构造函数或类：首字母大写驼峰法  
* 布尔值类型：以is/has/can/should等开头的驼峰法    
* 类的成员：公有属性和方法同上，私有属性和方法在前面加下划线
```js
let realName = 'sam';   // 标准变量
let isLogin = false;   // 布尔值类型
const PI = 3.14;
let input_dom = document.getElementById('input');  // dom元素变量
let Person = function(name, age) {  // 类名答谢
    this.name = name;
    this.age = age;
    this._test() {       // 私有方法，加下划线
        // do something
    }
}
```
8. 如果if语句中有return，后面不要跟else语句
9. 多个if else 应该用switch，逻辑更清晰
10. 注释：要重视注释，注释也是代码的一部分，需要和写代码一样去写注释。注释不做硬性规定，遵循不滥用注释，以让别人更容易的读懂代码为目标就行。
一般以下情况需要写注释
*  公共调用的方法或模块
*  一些比较复杂的方法或语句
*  比较容易混淆的变量
*  正则表达式
*  逻辑比较复杂的地方
*  方法注释参考以下
11. Prefer early exit
```js
// bad
function someFunction(someCondition) {
  if (someCondition) {
    // Do Something
  }
}

// good 逻辑更清晰，避免过度else if
function someFunction(someCondition) {
  if (!someCondition) {
      return;
  }
  // Do Something
}
```
12. ajax 错误处理不要漏掉
```js
login(param).then(res => {
    if (res.code !== 200) {
        this.$alert(res.msg || 'some message')
        return;
    }
    // do something
})
```



### css 规范  
1. Class或id命名：小写，单词之间用横线隔开 如 input-wrapper
2. 属性尽量采用简写方式
3. 避免选择器嵌套层级过多，尽量少于3级
4. 尽量使用子选择器（>），而不是后代选择器
5. 避免为0指定单位
6. 避免以下写法
```css
.font18 {
    font-size: 18px;
}
.p-0 {
    padding: 0 !important;
}
```
7. 不要用float和绝对定位布局（不方便响应式布局），用flex布局（现在主流框架的响应式都是基于flex）
8. media query 代码放到最后，否则很可能不起作用




### vueJs规范
1. v-for渲染列表时加上key（key必须是唯一的），以便维护内部组件及其子树的状态  
```html
<ul>
  <li v-for="todo in todos" :key="todo.id">
    {{ todo.text }}
  </li>
</ul>
```
2. 不要把v-if和v-for同时用在同一个元素上（会造成重复计算）  
3. 为组件样式设置作用域（避免样式冲突）  
4. 在模板中避免复杂的js表达式（没有高亮，代码堆在一起，阅读困难；使模块过重），复杂的表达式应用computed计算属性或者methods替代  
5. 在必要的地方加上v-cloak，避免数据未渲染而模板已经渲染  
6. 尽量使用vue提供的指令，避免dom操作，如有需要使用this.$refs
7. 指令的缩写应统一(用 : 表示 v-bind: 和用 @ 表示 v-on:) ，应该要么都用要么都不用
8. 不允许在组件内部改变props（会造成数据流动混乱）
9. data 方法里不要的变量定义尽量精简；如果是模板里没有使用，不需要在data里定义，可以直接挂在vue实例上，或者写一个局部变量代替
10. 不要在data里定义下划线开头的变量
```js
data() {
    return {
        _base: ''  // bad
    }
}
```
10. 学会使用计算属性，ex:
```js
data() {
    return {
        arr: [
            {status: true, foo: 1},
            {status: false, foo: 2}
        ]
    }
}
computed {
    // 用计算属性代替，而不是每次手动计算一次
    activeArr() {
        return this.arr.filter(i => i.status)
    }
}
```
11. 组件属性排列顺序（vue+js）
```js
  export default {
    name: "component-name",
    
    // 模板依赖
    components: {
        
    },
    directives: {
        
    },
    filters: {
        
    },
    
    // 一系列钩子函数
    created() {
      
    },
    mounted() {
        
    },
    
    // watch
    watch() {
        
    },
    
    // data相关
    props() {
        
    },
    data() {
      return {

      }
    },
    computed: {
      
    },
    
    // methods
    methods: {

    }
  };
```
12. 组件属性排列顺序（vue+ts）
```js
export default class HelloWorld extends Vue {
    // 全局属性定义
    $util: any;
    
    // 一系列钩子函数
    created() {
      
    }
    mounted() {
        
    }
    
    // 路由钩子
    beforeRouteEnter() {}
    beforeRouteUpdate() {}
    beforeRouteLeave() {}
    
    // watch
    @Watch('msg')
    onChildChanged(val: string, oldVal: string): void {
        console.log(`oldVal: ${oldVal} new val: ${val}`);
    }
    
    // data, computed, props, vuex(state)
    name: string = 'local name';
    get computedName(): string {
        return 'hello ' + this.name;
    }
    @Prop() msg!: string;
    @State('isShowLoading') isShowLoading!: boolean;
    @Getter('username') username!: string;
    
    // methods, $emit, vuex(mutaions, actions )
    toAbout() {
        this.$router.push('/about');
    }
    @Emit('changeMsg')
    handleEmit(): string {
        return this.name;
    }
}
```

### vueJs组件设计规范
1. 一个组件应该只做一件事，如果逻辑过多建议拆分组件  
2. 组件的data必须为一个函数（拷贝一个对象，避免多处使用相同组件时数据共享）
3. 组件命名：遵循W3C规范中的自定义组件名（字母全小写且必须包含一个连字符），避免和当前或者以后的HTML元素相冲突；
基础组件（也就是展示类的、无逻辑的或无状态的组件），应该以一个特定的前缀开头，比如el, mt；（避免出现my-header, his-header这种情况）
```
<el-header></el-header>
<el-footer></el-footer>
```
4. Props验证，要给定其数据类型type，其他验证根据需求 
5. Props尽量使用js原始数据类型（字符串、数字、布尔值），避免复杂的对象  
6. 不允许在组件内部改变props，以免造成数据流动难以理解；如果有需求要改变props，可以在组件内部用一个data属性或者computed属性接收，注意如果传递的是一个数组或者对象，需要深拷贝改数组或对象  
7. 根据业务严格划分data（内部状态）和props（外部状态），如果一个状态完全是由组件内部的逻辑来决定的，应该使用data，如果一个状态是受外部影响的，应该使用props  
8. 当需要操作组件dom的时候，推荐用this.$refs 而不是用jquery  
9. 在.vue文件里面，组件的style应加上scope，避免样式冲突  
10. 组件应写详细注释，特别是数据的定义流向，以便于其他同事理解代码  

### ts规范

---
### 问题
* 大家会写大量重复的代码，不管是js还是css，以后碰到两处以上会调用的代码，都要封装起来
* 变量命名不按照规范，并且不注意语义化。代码不仅仅是写给计算机执行的，同时是写给人看的。好的代码，一看变量，就大概知道这个变量是干嘛的，或者方法是干什么的，而不好的变量命名会让程序出现歧义
* 写代码不注意代码量，一个单文件组件能写到几千行，方法的行数也要注意。这里建议：单文件组件行数不大于1000，超出应该要拆分出来；方法行数不能超出100行，超出拆分
* 代码风格检查：代码规范 + eslint + prettier
* 每个人同时维护开发多个项目的开发方式，这样对代码质量和代码可维护性有更高的要求，因为你写的代码不只是你一个人看，也要让别人看懂
* 不要滥用ui框架提供的栅格布局，大的布局可以用栅格，小的方块布局用flex布局