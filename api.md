# 目录
## 基础api
##### 1. [登录(login)](#1-登录)
##### 2. [注册(register)](#2-注册)
##### 3. [获取个人信息(getPersonalInfo)](#3-获取个人信息)
##### 4. [修改个人信息(modifyPersonalInfo)](#4-修改个人信息)
##### 5. [修改密码(modifyPassword)](#5-修改密码)
##### 6. [意见反馈(submitFeedback)](#6-意见反馈)
## 协议文档
##### 7. [用户协议(user)](#7-用户协议)
##### 8. [获取软件许可协议(software)](#8-获取软件许可协议)
##### 9. [关于(about)](#9-关于)

---

## 基础api

---

### 1. [登录](#1-登录login)
- `login`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| phone | String | 登录手机号码 |
| password | String | 登录密码 |

```js
{
    "success": true,
    "context": {
        "userId": "10000"
    }
}
```

---

### 2. [注册](#2-注册register)
- `register`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| phone | String | 登录手机号码 |
| email | String | 找回密码的邮箱 |
| password | String | 登录密码 |


```js
{
    "success": true
}
```

---

### 3. [获取个人信息](#3-获取个人信息getpersonalinfo)
- `getPersonalInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |


```js
{
    "success": true,
    "context": {
        "phone": "手机号码"
        "name": "用户名"
        "head": "头像"
        "position": "职位",
        "authority": 1
    }
}
```
###### authority的说明：
```
authority为用户权限， 1：普通权限 2：拥有领导权限，4：拥有综合部权限，8：拥有监督者权限
```
---

### 4. [修改个人信息](#4-修改个人信息modifypersonalinfo)
- `modifyPersonalInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| phone | String | 登录手机号码 |
| name | String | 用户名 |
| head | String | 头像 |
| position | String | 职位 |


```js
{
    "success": true,
}
```

---

### 5. [修改密码](#5-修改密码modifypassword)
- `modifyPassword`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| oldPassword | String | 旧密码 |
| newPassword | String | 新密码 |

```js
{
    "success": true,
}
```

---

### 6. [意见反馈](#6-意见反馈submitfeedback)
- `submitFeedback`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| content | String | 内容 |
| email | String | 联系邮箱 |

```js
{
    "success": true,
}
```

---

## 协议文档

---

### 7. [用户协议](#7-用户协议user)
- `user`
- url: `protocals/user.html`

---

### 8. [获取软件许可协议](#8-获取软件许可协议software)
- `software`
- url: `protocals/software.html`

---

### 9. [关于](#9-关于about)
- `about`
- url: `protocals/about.html`
