---
id: javascript
title: node-express通过jwt实现带token验证的登录
author: Amos Xu
author_title: Front End Engineer|前端开发
author_url: https://github.com/AmossXu
author_image_url: https://avatars.githubusercontent.com/AmossXu
tags: [javascript, node]
---

# node-express通过jwt实现带token验证的登录

[toc]
## 文件目录
```
│  package-lock.json
│  package.json
│  server.js
│  
├─config//配置文件
│      key.js
│      passport.js
│
├─models//实例
│      Profile.js
│      User.js
│
└─routers
    └─api
        profile.js//产品api
        users.js//用户登录api
```

<!--truncate-->

## 启动项目
在package.json中
```
{
  "name": "node_app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node server.js",//server.js为入口启动文件
    "serve": "nodemon server.js"
  },
  "author": "amos",
  "license": "ISC",
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "body-parser": "^1.19.0",
    "express": "^4.17.1",
    "gravatar": "^1.8.0",
    "jsonwebtoken": "^8.5.1",
    "mongoose": "^5.9.3",
    "nodemon": "^2.0.2",
    "passport": "^0.4.1",
    "passport-jwt": "^4.0.0",
    "tree": "^0.1.3"
  }
}
```
```
npm run serve //通过nodemon启动server.js
```
## 入口文件-server.js
```
const express = require('express');//引入express模块
const app = express();//后续可以通过app.xxx()调用express组件
```
### 连接MongoDB
```
const mongoose = require('mongoose');
mongoose.set('useFindAndModify', false)
//DB config
const db = require('./config/key').mongoURL;
//db connection
mongoose.connect(db, {
        useNewUrlParser: true,
        useUnifiedTopology: true
    })
    .then(() => {
        console.log("MongoDB connected");
    }).catch(() => {
        console.log(err);
    })
```
### 中间件
```
const bodyParser = require('body-parser');
const passport = require('passport');
const users = require('./routers/api/users');
const profile = require('./routers/api/profile');

//body-parser
// parse application/x-www-form-urlencoded
/*
就是application/x-www-from-urlencoded,会将表单内的数据转换为键值对，&分隔。
当form的action为get时，浏览器用x-www-form-urlencoded的编码方式，将表单数据编码为
(name1=value1&name2=value2…)，然后把这个字符串append到url后面，用?分隔，跳转
到这个新的url。
当form的action为post时，浏览器将form数据封装到http body中，然后发送到server。
这个格式不能提交文件。
*/
app.use(bodyParser.urlencoded({
    extended: false
}));
// parse application/json
app.use(bodyParser.json());

//routers中间件
app.use('/api/users', users);
app.use('/api/profile', profile);

//passport初始化
app.use(passport.initialize());
require('./config/passport')(passport);
```
### routers/api/user.js中的login方法
```
// $route  POST api/users/login
// @desc   返回token jwt passport
// @access public
router.post(('/login'), (req, res) => {
    //接收url后面跟的数据
    const email = req.body.email;
    const password = req.body.password;
    //查询数据库
    User.findOne({
            email
        })
        .then(user => {
            if (!user) {
                return res.status(404).json("用户不存在");
            }
            //密码匹配 之前用的genSalt加密 现在使用compare匹配密码
            // Load hash from your password DB.
            bcrypt.compare(password, user.password)
                .then(isMatch => {
                    if (isMatch) {
                        //sign 第一个参数是规则 第二个参数是加密名字 第三个是过期时间 第四个是箭头函数
                        const rule = {
                            id: user.id,
                            name: user.name,
                            avatar: user.avatar,
                            identity: user.identity
                        };
                        jwt.sign(rule, keys.secreteOrKey, {
                            expiresIn: 3600
                        }, (err, token) => {
                            if (err) throw err;
                            res.json({
                                success: true,
                                token: "Bearer " + token
                            })
                        })
                        // res.json({
                        //     msg: "success"
                        // });
                    } else {
                        return res.status(400).json("密码错误");
                    }
                })
        }).catch(err => console.log(err));
})
```
## 前端接收到token之后
```
this.$axios.post('/api/users/login', this.loginUser)
    .then(res => {
        //登录成功，获取tokentoken
        const {
            token
        } = res.data;
        //存储到localStorage
        localStorage.setItem('eleToken', token);

        //解析token
        const decoded = jwt_decode(token);
        // console.log(decoded);

        //token存储到vuex中
        this.$store.dispatch('setAuthenticated', !this.isEmpty(decoded));
        this.$store.dispatch('setUser', decoded);

        // console.log(this.$store);

        this.$router.push('./index')

        this.$message({
            message: '登录成功！',
            type: 'success'
        })
    })
```

## Vue的vuex-actions
```
const actions = {
  setAuthenticated: ({
    commit
  }, isAuthenticated) => {
    commit(types.SET_AUTHENTICATED, isAuthenticated);
  },
  setUser: ({
    commit
  }, user) => {
    commit(types.SET_USER, user);
  },
  clearCurrentState: ({
    commit
  }) => {
    commit(types.SET_AUTHENTICATED, false);
    commit(types.SET_USER, null)
  }
}
```

## 在router的index.js中设置路由守卫
```
router.beforeEach((to, from, next) => {
  const isLogin = localStorage.eleToken ? true : false;
  if (to.path == '/login' || to.path == '/register') {
    next();
  } else {
    isLogin ? next() : next('/login')
  }
})
```
