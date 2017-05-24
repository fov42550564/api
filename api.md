## 目录
##### 1. [登录(login)](#1-登录)
##### 2. [注册(register)](#2-注册)
##### 3. [获取个人信息(getPersonalInfo)](#3-获取个人信息)
##### 4. [修改个人信息(modifyPersonalInfo)](#4-修改个人信息)
##### 5. [修改密码(modifyPassword)](#5-修改密码)
##### 6. [获取用户列表(getClientList)](#6-获取用户列表)
##### 7. [获取用户详情(getClientDetail)](#7-获取用户详情)
##### 8. [获取任务类型列表(getTaskTypeList)](#8-获取任务类型列表)
##### 9. [创建单一任务(leaderCreateTask)](#9-创建单一任务)
##### 10. [创建综合任务(secretaryCreateTask)](#10-创建综合任务)
##### 11. [领导获取待审批发布的任务列表(getApplyPublishTaskList)](#11-领导获取待审批发布的任务列表)
##### 12. [领导或者综合部获取待审批完成的任务列表(getApplyFinishTaskList)](#12-领导或者综合部获取待审批完成的任务列表)
##### 13. [根据类型获取的任务列表(getTaskListByType)](#13-根据类型获取的任务列表)
##### 14. [获取打回申请发布任务列表(getRejectTaskList)](#14-获取打回申请发布任务列表)
##### 15. [获取最关心任务列表(getMostCaredTaskList)](#15-获取最关心任务列表)
##### 16. [获取单一任务的详情(getSingleTaskDetail)](#16-获取单一任务的详情)
##### 17. [获取群组任务的详情(getGroupTaskDetail)](#17-获取群组任务的详情)
##### 18. [修改任务(modifyTask)](#18-修改任务)
##### 19. [修改群组任务(modifyGroupTask)](#19-修改群组任务)
##### 20. [删除群组任务(removeGroupTask)](#20-删除群组任务)
##### 21. [删除任务(removeGroupTask)](#21-删除任务)
##### 22. [打回申请发布任务(rejectPublishTask)](#22-打回申请发布任务)
##### 23. [通过发布任务的申请(agreePublishTask)](#23-通过发布任务的申请)
##### 24. [申请完成任务(applyFinishTask)](#24-申请完成任务)
##### 25. [同意完成任务的申请(agreeFinishTask)](#25-同意完成任务的申请)
##### 26. [打回完成发布任务(rejectFinishTask)](#26-打回完成发布任务)
##### 27. [创建提醒(createRemind)](#27-创建提醒)
##### 28. [修改提醒(modifyRemind)](#28-修改提醒)
##### 29. [推送提醒通知(pushRemindNotice)](#29-推送提醒通知)
##### 30. [创建日程(createSchedule)](#30-创建日程)
##### 31. [完成日程(finishSchedule)](#31-完成日程)
##### 32. [修改日程(modifySchedule)](#32-修改日程)
##### 33. [删除日程(removeSchedule)](#33-删除日程)
##### 34. [获取日程列表(getScheduleList)](#34-获取日程列表)
##### 35. [提醒任务(remaindTask)](#35-提醒任务)
##### 36. [更新任务进度(updateTaskProgress)](#36-更新任务进度)
##### 37. [获取进度列表(getTaskProgressList)](#37-获取进度列表)

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

### 4. [修改个人信息](#4-修改个人信息modifypersonalinfo)
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

### 5. [修改密码](#5-修改密码modifypassword)
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

### 6. [获取用户列表](#6-获取用户列表getclientlist)
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

### 7. [获取用户详情](#7-获取用户详情getclientdetail)
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

### 8. [获取任务类型列表](#8-获取任务类型列表gettasktypelist)
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
                "key": "1",
                "name": "综合任务",
            }
        ]
    }
}
```

---

### 9. [创建单一任务](#9-创建单一任务leadercreatetask)
- `leaderCreateTask`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| executorId | String | 执行人Id |
| supervisorId | String | 监督人Id |
| title | String | 标题 |
| content | String | 内容 |
| audioList | Array | [音频列表](#audiolist格式) |
| imageList | Array | 图片列表 |
| remindList | Array | [提醒列表](#remindlist格式) |
| type | Number | 任务类型 |
| expectStartTime | String | 期望开始时间 |
| expectFinishTime | String | 期望结束时间 |

###### audioList格式：

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| url | String | 上传音频文件返回的url |
| duration | Number | 音频文件的时长 |

###### remindList格式：

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

### 10. [创建综合任务](#10-创建综合任务secretarycreatetask)
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
| audioList | Array | [音频列表](#audiolist格式) |
| imageList | Array | 图片列表 |
| remindList | Array | [提醒列表](#remindlist格式) |
| type | Number | 任务类型(0：一般任务， 1：紧急任务，2：加急任务) |
| expectStartTime | String | 期望开始时间 |
| expectFinishTime | String | 期望结束时间 |

```js
{
    "success": true,
}
```

---

### 11. [领导获取待审批发布的任务列表](#11-领导获取待审批发布的任务列表getapplypublishtasklist)
- `getApplyPublishTaskList`
- 请求方式：`POST`
- 备注：获取到的是多任务或者单任务

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

### 12. [领导或者综合部获取待审批完成的任务列表](#12-领导或者综合部获取待审批完成的任务列表getapplyfinishtasklist)
- `getApplyFinishTaskList`
- 请求方式：`POST`
- 备注：谁发布的任务谁来审批完成

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
            }
        ]
    }
}
```

---

### 13. [根据类型获取的任务列表](#13-根据类型获取的任务列表gettasklistbytype)
- `getTaskListByType`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| type | Number | [类型](#8-获取任务类型列表) |
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
                "type": 1,
                "id": "5923860a5c1070c1e96defd2"
            }
        ]
    }
}
```

---

### 14. [获取打回申请发布任务列表](#14-获取打回申请发布任务列表getrejecttasklist)
- `getRejectTaskList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| role | String | 角色 = ['`leader`', '`secretary`', '`executor`'] |
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
                "rejectPublishReason": "不合格",
                "rejectFinishReason": "不合格",
                "type": 1,
                "state": 1,
                "id": "5923860a5c1070c1e96defd2"
            }
        ]
    }
}
```

###### 状态说明：
`1：待审批，2：驳回审批，4：通过审批，8：待执行， 16：进行中，32：待完成审核，64：驳回完成审核，128：完成`

---

### 15. [获取最关心任务列表](#15-获取最关心任务列表getmostcaredtasklist)
- `getMostCaredTaskList`
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
                "title": "测试任务",
                "content": "认真测试",
                "modifyTime": "2017-05-23 08:44:58",
                "expectFinishTime": "2017-05-16 03:00:00",
                "rejectPublishReason": "不合格",
                "type": 1,
                "state": 1,
                "id": "5923860a5c1070c1e96defd2"
            }
        ]
    }
}
```

---

### 16. [获取单一任务的详情](#16-获取单一任务的详情getsingletaskdetail)
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

### 17. [获取群组任务的详情](#17-获取群组任务的详情getgrouptaskdetail)
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

### 18. [修改任务](#18-修改任务modifytask)
- `modifyTask`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| taskId | String | 任务Id |
| executorId | String | 执行人Id |
| supervisorId | String | 监督人Id |
| title | String | 标题 |
| content | String | 内容 |
| audioList | Array | [音频列表](#audiolist格式) |
| imageList | Array | 图片列表 |
| remindList | Array | [提醒列表](#remindlist格式) |
| type | Number | 任务类型 |
| expectStartTime | String | 期望开始时间 |
| expectFinishTime | String | 期望结束时间 |

```js
{
    "success": true,
}
```

---

### 19. [修改群组任务](#19-修改群组任务modifygrouptask)
- `modifyGroupTask`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| taskId | ID | 任务Id |
| examinerId | ID | 审核人Id |
| title | String | 标题 |
| content | String | 内容 |

```js
{
    "success": true,
}
```

---

### 20. [删除群组任务](#20-删除群组任务removegrouptask)
- `removeGroupTask`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| taskId | ID | 任务Id |

```js
{
    "success": true,
}
```

---

### 21. [删除任务](#21-删除任务removegrouptask)
- `removeGroupTask`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| taskId | ID | 任务Id |

```js
{
    "success": true,
}
```

---

### 22. [打回申请发布任务](#22-打回申请发布任务rejectpublishtask)
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

---

### 23. [通过发布任务的申请](#23-通过发布任务的申请agreepublishtask)
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

### 24. [申请完成任务](#24-申请完成任务applyfinishtask)
- `applyFinishTask`
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

### 25. [同意完成任务的申请](#25-同意完成任务的申请agreefinishtask)
- `agreeFinishTask`
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

### 26. [打回完成发布任务](#26-打回完成发布任务rejectfinishtask)
- `rejectFinishTask`
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

---

### 27. [创建提醒](#27-创建提醒createremind)
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

### 28. [修改提醒](#28-修改提醒modifyremind)
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

### 29. [推送提醒通知](#29-推送提醒通知pushremindnotice)
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

### 30. [创建日程](#30-创建日程createschedule)
- `createSchedule`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| content | String | 日程内容 |

```js
{
    "success": true,
}
```

---

### 31. [完成日程](#31-完成日程finishschedule)
- `finishSchedule`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| scheduleId | String | 日程Id |

```js
{
    "success": true,
}
```

---

### 32. [修改日程](#32-修改日程modifyschedule)
- `modifySchedule`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| scheduleId | String | 日程Id |
| content | String | 日程内容 |

```js
{
    "success": true,
}
```

---

### 33. [删除日程](#33-删除日程removeschedule)
- `removeSchedule`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| scheduleId | String | 日程Id |

```js
{
    "success": true,
}
```

---

### 34. [获取日程列表](#34-获取日程列表getschedulelist)
- `getScheduleList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | String | 用户Id |
| time | String | 日程时间 |

```js
{
    "success": true,
    "context": {
        "scheduleList": [
            {
                "content": "测试日程",
                "state": 0,
                "id": "59225114d4118c3a27ff4a5d"
            }
        ]
    }
}
```

---

### 35. [提醒任务](#35-提醒任务remaindtask)
- `remaindTask`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 监督人Id |
| taskId | ID | 任务Id |

```js
{
    "success": true,
}
```

---

### 36. [更新任务进度](#36-更新任务进度updatetaskprogress)
- `updateTaskProgress`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 监督人Id |
| taskId | ID | 任务Id |
| content | String | 填写的内容 |

```js
{
    "success": true,
}
```

---

### 37. [获取进度列表](#37-获取进度列表gettaskprogresslist)
- `getTaskProgressList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 监督人Id |
| taskId | ID | 任务Id |

```js
{
    "success": true,
    "context": {
        "ProgressList": [
            {
                "createTime": "2017-05-23 17:09:20",
                "content": "测试进度",
                "userName": "方运江",
                "id": "5923fc408ddca4373159bb66"
            },
            {
                "createTime": "2017-05-23 17:09:18",
                "content": "提醒一次任务",
                "userName": "方运江",
                "id": "5923fc3e8ddca4373159bb65"
            }
        ]
    }
}
```
