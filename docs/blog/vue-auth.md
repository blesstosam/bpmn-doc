# Vue权限系统
后台管理系统一般都会有权限模块，用来控制用户能访问哪些页面和哪些数据接口。大多数管理系统的页面都长这样。 

![image](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1547697255453&di=b3726bc2804b4aa20d915bb4f175e29a&imgtype=0&src=http%3A%2F%2Fimg.redocn.com%2Fsheji%2F20151208%2Fheisehoutaiguanlixitongsheji_5512176.jpg)

左边为菜单，分为两级，右边为图表显示区域，有增删改查的按钮。
### 表的结构
```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for t_auth_rule
-- ----------------------------
DROP TABLE IF EXISTS `t_auth_rule`;
CREATE TABLE `t_auth_rule` (
  `id_pk` bigint(20) NOT NULL AUTO_INCREMENT,
  `auth_id` varchar(128) NOT NULL COMMENT '权限Id',
  `pauth_id` varchar(128) DEFAULT NULL COMMENT '父级Id',
  `auth_name` varchar(255) NOT NULL COMMENT '权限名称',
  `auth_icon` varchar(255) NOT NULL COMMENT '权限图标',
  `auth_type` smallint(6) NOT NULL COMMENT '权限类型，BIT表示其属性\r\n            0x00表示可显示的菜单权限节点;\r\n            0x01表示普通节点',
  `auth_condition` text COMMENT '条件',
  `remark` varchar(255) DEFAULT NULL COMMENT '备注',
  `is_menu` smallint(255) DEFAULT '0' COMMENT '是否为菜单，0表示非，1表示是',
  `weight` int(11) NOT NULL DEFAULT '0' COMMENT '权重',
  `rule` varchar(256) DEFAULT NULL COMMENT '规则路径主要对应菜单或方法的路径名称',
  `cr_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `up_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id_pk`),
  UNIQUE KEY `AK_auth_id` (`auth_id`)
) ENGINE=InnoDB AUTO_INCREMENT=264 DEFAULT CHARSET=utf8 COMMENT='权限规则表，记录权限相关的信息，权限以父子关系存在，菜单是权限的一种。';

SET FOREIGN_KEY_CHECKS = 1;


SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for t_role_auth
-- ----------------------------
DROP TABLE IF EXISTS `t_role_auth`;
CREATE TABLE `t_role_auth` (
  `id_pk` bigint(20) NOT NULL AUTO_INCREMENT,
  `role_id_fk` varchar(32) DEFAULT NULL COMMENT '角色id',
  `auth_id_fk` varchar(128) DEFAULT NULL COMMENT '权限id',
  `aa` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id_pk`)
) ENGINE=InnoDB AUTO_INCREMENT=77 DEFAULT CHARSET=utf8 COMMENT='角色与权限的关系表';

SET FOREIGN_KEY_CHECKS = 1;
```
稍微解释一下表结构，t_auth_rule 表用来存储对应的权限菜单，一般来说，菜单分为一级和二级菜单，rule字段对应前端的路由规则；而按钮为第三级，rule对应的是接口url地址。  
t_role_auth 表为角色权限关联表，一个角色拥有哪些权限是通过这张表查出来的。当然还有一个role表，还有一个账号表，账号表里有一个role的外键。  
**这样就是一个账号 --> 角色 --> 权限的关系。**

### 对于菜单的权限，通过路由表匹配
```js
// 本地写好路由列表（需要进行动态匹配的）
export const routerList: Array<RouteConfig> = [
  // 首页
  {
    path: '/',
    name: '_home',
    redirect: '/home',
    component: Layout,
    meta: {
      hideInMenu: true,
      notCache: true
    },
    children: [
      {
        path: 'home',
        name: 'home',
        meta: {
          hideInMenu: true,
          title: '首页',
          notCache: true,
          icon: 'md-home'
        },
        component: () => import('@/views/overview-operations/data-center/DataCenter.vue')
      }
    ]
  }
]

// 负责将后台返回的 菜单列表 转成vue-router所需要的 router list
export function toRouterComponent(menuList: Array<any>) {
  if (!menuList.length) return []

  let routerArr = [];
  for (let j = 0; j < menuList.length; j++) {
    let obj;
    let firstIndex = routerList.findIndex(i => i.path === menuList[j].url); // 一级菜单
    if (firstIndex !== -1) {
      let children: Array<RouteConfig> = []
      obj = {
        path: routerList[firstIndex].path,
        component: routerList[firstIndex].component,
        redirect: routerList[firstIndex].redirect,
        name: routerList[firstIndex].name,
        meta: routerList[firstIndex].meta,
        children
      };

      // 如果有子菜单
      if (menuList[j].children && menuList[j].children.length) {
        for (let k = 0; k < menuList[j].children.length; k++) {
          const _children = routerList[firstIndex].children!
          let secondIndex = _children.findIndex(i => {
            let fullpath = '';
            if (routerList[firstIndex].path === '/') {
              fullpath = `${routerList[firstIndex].path}${i.path}`;
            } else {
              fullpath = `${routerList[firstIndex].path}/${i.path}`;
            }
            return fullpath === menuList[j].children[k].url;
          });
          if (secondIndex !== -1) {
            obj.children.push(_children[secondIndex]);
          }
        }
      }
    }

    if (obj) {
      routerArr.push(obj);
    }
  }

  return routerArr;
}

// 根据菜单权限，获取路由数组
// 本地只保存后台返回的菜单，在页面刷新的时候从本地拿到菜单重新调用toRouterComponent生成 路由数组
export function getRouterList() {
  if (!storage.get('username')) {
    storage.set('menuTree', ''); // 清空菜单权限数据
  }
  let menuList = storage.get('menuTree') ? storage.get('menuTree') : [];

  const routerArr = toRouterComponent(menuList);
  return routerArr;
}
```
menuList，菜单数组（或对象）, 由后台返回； routerList为前端定义的路由表；遍历routerList，如果routerList的path在menuMap里能找到的话，就表示该路由存在。最后生成一个过滤后的路由表，用vue提供的addRoutes方法动态添加到路由中，并把过滤后的路由表存到本地。

在页面刷新的时候，从本地获取路由表，添加到路由表中，代码如下，constRouterArr为基础路由表，比如登录，404等  

**注意这一步有个问题，由于我写的storage库用了JSON.stringify，把路由表中的component（实际为一个函数）丢失了，所以在从本地获取路由的时候，还要重新生成一个新的路由表，重新把component加上去，即把上面的addrouters重新执行一遍**

### 对于按钮的权限
```js
if (res.data.auth_rule_map) {
    let obj = {}
    Object.keys(res.data.auth_rule_map).forEach(i => {
      // 将所有的按钮放到一个obj里 key 为接口地址  
      if (res.data.auth_rule_map[i].is_menu === 0) {  // 如果是按钮
        obj[res.data.auth_rule_map[i].rule] = 1
      }             
    })
    storage.set("btnList", obj);
    storage.set("menuTree", res.data.auth_rule_map);
}
```
auth_rule_map为接口返回权限map，**把按钮的权限过滤出来存到本地**。  
将map添加到每个**路由组件**的data里，(这里有一个问题，怎么判断一个组件是否是路由组件)，目前想到的是通过组件name来判断，把所有的路由组件放到一个数组里做判断。  

在组件内部的按钮上加上v-if，如果this.uri__里的uri在uriMap里存在就显示。  
也可以通过方法来判断，如下面的__isBtnShow，不仅可以控制按钮的显示隐藏，还可以控制其样式，比如颜色等，更加灵活，推荐使用方法来控制
```js
uri = {
    ADD_MEMBER: '/api/add_member'
}

export default function install (Vue) {
  const uriMap = storage.get('btnList')
  //uriMap['/admin/api/auth_rule/update_auth_rule.action'] = 1
  Vue.mixin({
    created() {
      const arr = ['MemberManage', 'PayManage', '...']
      if (arr.indexOf(this.$options.name) !== -1) {
        this.dataUri__ = uriMap
        this.uri__ = uri  
      }
    },
    data() {
      return {
        dataUri__: {}
      }
    },
    methods: {
      __isBtnShow(uri) {
        return uriMap[uri] ? 'display: inline-block' : 'display: none'
      },
    }
  })
}

<Button v-if="dataUri__[uri__.ADD_MEMBER]">添加会员</Button>

// 通过方法来控制，更加灵活
<Button :style="__isBtnShow(uri__.ADD_MEMBER)">添加会员</Button>
```
### 登出的问题
**登出后要清空缓存，routerArr，btnList 等。  
由于之前登录，调用addRouter把权限上个账号的路由表加进去了，所以登出后要location.reload()一次，重新实例化路由表，去掉动态添加的路由，只保留基础路由。  
location.reload()体验不是太好，但是vue-router没有提供动态删除路由的api，比如 deleteRouter。**
### 同时登两个账号，导致刷新页面的时候，前者页面的本地缓存被覆盖，权限菜单等数据发生变化，路由表也发生变化
能想到的解决方法是存一个loginIndex 来表示登录账号的个数，比如第一次登录的时候存一个loginIndex=0， 后面存数据的时候都把这个参数带上；后面登多个账号的时候个loginIndex++，这样localStorage的key就是一个动态的（这样还是不行）  
**最简单的方法是存到localStorage里，只有登出才会清空缓存，只能登一个账号。**

### 关于菜单和按钮的多语言放在前台还是后台
经过思考还是觉得放在前台好，有两点原因
* 放在前台方便前台操作，切换语言时不需要重新从后台获取菜单和按钮的名称
* 如果放在后台，新增语言的时候，后台还得加字段， 放在前台的话只需要更新一下语言包就行了

### 关于menutree 是否存到本地
之前一直是存在localstorage里，但是仔细一想还是有安全问题，用户可以通过api改变本地的menutree，刷新页面后还是会得到想要的权限，所以menutree 不能存到本地，只读到内存中，页面刷新后，再重新获取一次menutree，代价就是会牺牲一点用户体验。
