## 目录
##### 1. [登录(login)](#1-登录)
##### 2. [注册(register)](#2-注册)
##### 3. [获取个人信息(getPersonalInfo)](#3-获取个人信息)
##### 4. [修改个人信息(modifyPersonalInfo)](#4-修改个人信息)
##### 5. [修改密码(modifyPassword)](#5-修改密码)
##### 6. [获取用户列表(getClientList)](#6-获取用户列表)
##### 7. [获取用户详情(getClientDetail)](#7-获取用户详情)
##### 8. [获取任务类型列表(getTaskTypeList)](#8-获取任务类型列表)
##### 9. [通过类型获取任务列表(getTaskListByType)](#9-通过类型获取任务列表)
##### 10. [创建单一任务(leaderCreateTask)](#10-创建单一任务)
##### 11. [创建综合任务(secretaryCreateTask)](#11-创建综合任务)
##### 12. [领导获取待审批的任务列表(getToExamineTaskList)](#12-领导获取待审批的任务列表)
##### 13. [根据类型获取的任务列表(getTaskListByType)](#13-根据类型获取的任务列表)
##### 14. [获取单一任务的详情(getSingleTaskDetail)](#14-获取单一任务的详情)
##### 15. [获取群组任务的详情(getGroupTaskDetail)](#15-获取群组任务的详情)
##### 16. [打回任务(rejectPublishTask)](#16-打回任务)
##### 17. [通过任务(agreePublishTask)](#17-通过任务)
##### 18. [创建提醒(createRemind)](#18-创建提醒)
##### 19. [修改提醒(modifyRemind)](#19-修改提醒)
##### 20. [推送提醒通知(pushRemindNotice)](#20-推送提醒通知)
##### 21. [创建日程(createSchedule)](#21-创建日程)
##### 22. [修改日程(modifySchedule)](#22-修改日程)
##### 23. [删除日程(removeSchedule)](#23-删除日程)
##### 24. [获取日程列表(getScheduleList)](#24-获取日程列表)
##### 25. [修改进度(modifyProgress)](#25-修改进度)
##### 26. [获取进度列表(getProgressList)](#26-获取进度列表)

---

### 14. [登录](#14-登录login)
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

### 15. [注册](#15-注册register)
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

### 16. [获取个人信息](#16-获取个人信息getPersonalInfo)
- `getPersonalInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |


```js
{
    "success": true,
    "context": {
        "phone": "手机号码"
        "name": "用户名"
        "head": "头像"
        "position": "职位"
    }
}
```
---

### 17. [修改个人信息](#17-修改个人信息modifyPersonalInfo)
- `modifyPersonalInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
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

### 18. [修改密码](#18-修改密码modifyPassword)
- `modifyPassword`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| oldPassword | String | 旧密码 |
| newPassword | String | 新密码 |

```js
{
    "success": true,
}
```

---

### 19. [获取用户列表](#19-获取用户列表getClientList)
- `getClientList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| keyword | String | 搜索关键字 |
| pageNo | Number | 第几页 |
| pageSize | Number | 每页的数量 |

```js
{
    "success": true,
    "context": {
        "clientList": [
            {
                "phone": "18085192480",
                "email": "42550564@qq.com",
                "name": "方运江",
                "head": "http://localhost:3000/api/image?id=59117df95150c20e08b560af",
                "partment": {
                    "name": "123",
                    "id": "591fa902b526bebf009d92c3"
                },
                "reservePhone": [
                    "18085192480"
                ],
                "id": "59117ddb5150c20e08b560ac"
            }
        ]
    }
}
```

---

### 20. [获取用户详情](#20-获取用户详情getClientDetail)
- `getClientDetail`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| clientId | String | 获取详情的用户Id |

```js
{
    "success": true,
    "context": {
        "phone": "18085192480",
        "email": "42550564@qq.com",
        "name": "方运江",
        "head": "http://localhost:3000/api/image?id=59117df95150c20e08b560af",
        "birthday": "1982-02-25",
        "partment": {
            "name": "123",
            "descript": "123",
            "chargeMan": {
                "phone": "18085192480",
                "name": "方运江",
                "head": "http://localhost:3000/api/image?id=59117df95150c20e08b560af",
                "id": "59117ddb5150c20e08b560ac"
            },
            "superior": {
                "name": "工信部",
                "id": "591cf488f6a0636b34d64985"
            },
            "phoneList": [
                "18085192480"
            ],
            "id": "591fa902b526bebf009d92c3",
            "membersNum": 1,
            "suborsNum": 1
        },
        "registerTime": "2017-05-09 16:29:15",
        "salary": 0,
        "reservePhone": [
            "18085192480"
        ],
        "age": 35,
        "sex": 0,
        "id": "59117ddb5150c20e08b560ac"
    }
}
```

---

### 21. [获取任务类型列表](#21-获取任务类型列表getTaskTypeList)
- `getTaskTypeList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
```js
{
    "success": true,
    "context": {
        "taskTypeList": [
            {
                "name": "类型1",
                "id": "5922b03e9a24539774bc0eec"
            }
        ]
    }
}
```

---

### 22. [通过类型获取任务列表](#22-通过类型获取任务列表getTaskListByType)
- `getTaskListByType`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
```js
{
    "success": true,
    "context": {
        "taskTypeList": [
            {
                "key": "1",
                "name": "综合任务",
            }
        ]
    }
}
```

---

### 23. [创建单一任务](#23-创建单一任务leaderCreateTask)
- `leaderCreateTask`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| executorId | String | 执行人Id |
| supervisorId | String | 监督人Id |
| title | String | 标题 |
| content | String | 内容 |
| audioList | Array | 音频列表 |
| imageList | Array | 图片列表 |
| remindList | Array | 提醒列表 |
| type | Number | 任务类型 |
| expectStartTime | String | 期望开始时间 |
| expectFinishTime | String | 期望结束时间 |
- audioList 格式：

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| url | String | 上传音频文件返回的url |
| duration | Number | 音频文件的时长 |

-  remindList 格式：

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| info | String | 提醒的内容 |
| time | String | 提醒的时间 |

```js
{
    "success": true,
}
```

---

### 24. [创建综合任务](#24-创建综合任务secretaryCreateTask)
- `secretaryCreateTask`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| examinerId | String | 审核人Id |
| title | String | 标题 |
| content | String | 内容 |
| taskList | Array | 任务列表 |

- taskList 格式：

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| executorId | String | 执行人Id |
| supervisorId | String | 监督人Id |
| title | String | 标题 |
| content | String | 内容 |
| audioList | Array | 音频列表 |
| imageList | Array | 图片列表 |
| remindList | Array | 提醒列表 |
| type | Number | 任务类型(0：一般任务， 1：紧急任务，2：加急任务) |
| expectStartTime | String | 期望开始时间 |
| expectFinishTime | String | 期望结束时间 |

- audioList 格式：

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| url | String | 上传音频文件返回的url |
| duration | Number | 音频文件的时长 |

- remindList 格式：

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| info | String | 提醒的内容 |
| time | String | 提醒的时间 |
```js
{
    "success": true,
}
```

---

### 25. [领导获取待审批的任务列表](#25-领导获取待审批的任务列表getToExamineTaskList)
- `getToExamineTaskList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| pageNo | Number | 页号 |
| pageSize | Number | 每页数量 |

```js
{
    "success": true,
    "context": {
        "taskList": [
            {
                "id": "5922b7726df4409ec01c425d",
                "title": "总测试标题",
                "content": "总测试内容",
                "modifyTime": "2017-05-22 18:03:30",
                "expectFinishTime": "2017-05-16 03:00:00",
                "expectStartTime": "2017-05-15 03:00:00",
                "taskNumbers": 1,
                "isSingleTask": true //是否是单任务
            }
        ]
    }
}
```

---

### 26. [根据类型获取的任务列表](#26-根据类型获取的任务列表getTaskListByType)
- `getTaskListByType`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| type | Number | 类型(#9-通过类型获取任务列表) |
| keyword | String | 搜索关键字 |
| pageNo | Number | 页号 |
| pageSize | Number | 每页数量 |

```js
{
    "success": true,
    "context": {
        "taskList": [
            {
                "title": "测试任务",
                "content": "认真测试",
                "modifyTime": "2017-05-23 08:44:58",
                "expectFinishTime": "2017-05-16 03:00:00",
                "id": "5923860a5c1070c1e96defd2"
            }
        ]
    }
}
```

---

### 27. [获取单一任务的详情](#27-获取单一任务的详情getSingleTaskDetail)
- `getSingleTaskDetail`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| taskId | ID | 任务Id |

```js
{
    "success": true,
    "context": {
        "title": "测试标题",
        "content": "测试内容",
        "examineFinishTime": "2017-05-22 18:03:30",
        "applyFinishTime": "2017-05-22 18:03:30",
        "modifyTime": "2017-05-22 18:03:30",
        "startExecTime": "2017-05-22 18:03:30",
        "examineTime": "2017-05-22 18:03:30",
        "publishTime": "2017-05-22 18:03:30",
        "expectFinishTime": "2017-05-16 03:00:00",
        "expectStartTime": "2017-05-15 03:00:00",
        "state": 1,
        "type": 1,
        "imageList": [
            "http://localhost:3000/api/image?id=5922b7726df4409ec01c425b"
        ],
        "audioList": [
            {
                "url": "http://localhost:3000/api/image?id=5922b7726df4409ec01c425c",
                "duration": 10
            }
        ],
        "supervisor": {
            "phone": "18085192480",
            "name": "方运江",
            "id": "59117ddb5150c20e08b560ac"
        },
        "executor": {
            "phone": "18085192480",
            "name": "方运江",
            "id": "59117ddb5150c20e08b560ac"
        },
        "id": "5922b7726df4409ec01c425e"
    }
}
```

---

### 28. [获取群组任务的详情](#28-获取群组任务的详情getGroupTaskDetail)
- `getGroupTaskDetail`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| taskId | ID | 任务Id |

```js
{
    "success": true,
    "context": {
        "title": "总测试标题",
        "content": "总测试内容",
        "taskList": [
            {
                "title": "测试标题",
                "content": "测试内容",
                "examineFinishTime": "2017-05-22 18:03:30",
                "applyFinishTime": "2017-05-22 18:03:30",
                "modifyTime": "2017-05-22 18:03:30",
                "startExecTime": "2017-05-22 18:03:30",
                "examineTime": "2017-05-22 18:03:30",
                "publishTime": "2017-05-22 18:03:30",
                "expectFinishTime": "2017-05-16 03:00:00",
                "expectStartTime": "2017-05-15 03:00:00",
                "state": 1,
                "type": 1,
                "imageList": [
                    "http://localhost:3000/api/image?id=5922b7726df4409ec01c425b"
                ],
                "audioList": [
                    {
                        "url": "http://localhost:3000/api/image?id=5922b7726df4409ec01c425c",
                        "duration": 10
                    }
                ],
                "supervisor": {
                    "phone": "18085192480",
                    "name": "方运江",
                    "id": "59117ddb5150c20e08b560ac"
                },
                "executor": {
                    "phone": "18085192480",
                    "name": "方运江",
                    "id": "59117ddb5150c20e08b560ac"
                },
                "id": "5922b7726df4409ec01c425e"
            }
        ],
        "id": "5922b7726df4409ec01c425d"
    }
}
```

---

### 29. [打回任务](#29-打回任务rejectPublishTask)
- `rejectPublishTask`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| taskId | String | 任务Id |
| reason | String | 被打回的原因 |

```js
{
    "success": true,
}
```

### 30. [通过任务](#30-通过任务agreePublishTask)
- `agreePublishTask`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| taskId | String | 任务Id |

```js
{
    "success": true,
}
```

---

### 31. [创建提醒](#31-创建提醒createRemind)
- `createRemind`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| defaultRemindRule | String | 默认提醒规则 |
| customRemindTime | Date | 自定义提醒时间 |

```js
{
    "success": true,
}
```

---

### 32. [修改提醒](#32-修改提醒modifyRemind)
- `modifyRemind`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| remindId | String | 提醒Id |
| defaultRemindRule | String | 默认提醒规则 |
| customRemindTime | Date | 自定义提醒时间 |

```js
{
    "success": true,
}
```

---

### 33. [推送提醒通知](#33-推送提醒通知pushRemindNotice)
- `pushRemindNotice`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| receiveUserId | String | 接受者用户Id |
| remindId | String | 提醒Id |


```js
{
    "success": true,
}
```

---

### 34. [创建日程](#34-创建日程createSchedule)
- `createSchedule`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| time | String | 日程时间 |
| content | String | 日程内容 |
| state | String | 日程状态 |

```js
{
    "success": true,
}
```

---

### 35. [修改日程](#35-修改日程modifySchedule)
- `modifySchedule`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| id | String | 日程Id |
| time | String | 日程时间 |
| content | String | 日程内容 |
| state | String | 日程状态 |

```js
{
    "success": true,
}
```

---

### 36. [删除日程](#36-删除日程removeSchedule)
- `removeSchedule`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| id | String | 日程Id |

```js
{
    "success": true,
}
```

---

### 37. [获取日程列表](#37-获取日程列表getScheduleList)
- `getScheduleList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| time | String | 日程时间 |
| content | String | 日程内容 |
| state | String | 日程状态 |

```js
{
    "success": true,
    "context": {
        "scheduleList": [
            {
                "id":"123",
                "time":"8:00",
                "content":"要干嘛",
                "state":1
            }
        ]
    }
}
```

---

### 38. [修改进度](#38-修改进度modifyProgress)
- `modifyProgress`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| taskId | String | 任务Id |
| time | String | 修改时间 |
| content | String | 修改内容 |

```js
{
    "success": true,
}
```

---

### 39. [获取进度列表](#39-获取进度列表getProgressList)
- `getProgressList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| taskId | String | 任务Id |

```js
{
    "success": true,
    "context": {
        "progressList": [
            {
                "time":"8:00",
                "content":"状态",
            }
        ]
    }
}
```
