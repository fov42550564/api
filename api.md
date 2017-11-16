# 目录
## 基础api
##### 1. [登录(login)](#1-登录)
##### 2. [注册(register)](#2-注册)
##### 3. [发送验证码(requestSendVerifyCode)](#3-发送验证码)
##### 4. [修改支付密码(setPaymentPassword)](#4-修改支付密码)
##### 5. [修改登录密码(findPassword)](#5-修改登录密码)
##### 6. [获取账户余额(getRemainAmount)](#6-获取账户余额)
##### 7. [货主账户充值(recharge)](#7-货主账户充值)
##### 8. [货主账户提现(withdraw)](#8-货主账户提现)
##### 9. [获取个人信息(getPersonalInfo)](#9-获取个人信息)
##### 10. [修改个人信息(modifyPersonalInfo)](#10-修改个人信息)
##### 11. [修改密码(modifyPassword)](#11-修改密码)
##### 12. [意见反馈(submitFeedback)](#12-意见反馈)
##### 13. [董事长修改参数(modifySettingInfo)](#13-董事长修改参数)
##### 14. [货主获取交易记录(getBillList)](#14-货主获取交易记录)
##### 15. [获取中国区划地址(getRegionAddress)](#15-获取中国区划地址)
##### 16. [获取发货点地址(getStartPointAddress)](#16-获取发货点地址)
## 货单api
##### 17. [货主预下单(placePreOrder)](#17-货主预下单)
##### 18. [货主获取发货单列表(getSendOrderList)](#18-货主获取发货单列表)
##### 19. [物流公司获取已装货单卡车列表(getTrucks)](#19-物流公司获取已装货单卡车列表)
##### 20. [物流公司获取货车里的货单列表(getOrderListInTruck)](#20-物流公司获取货车里的货单列表)
##### 21. [物流公司通过司机手机号获取司机信息(getDriverInfoByPhone)](#21-物流公司通过司机手机号获取司机信息)
##### 22. [货主获取收货单列表(getReceiveOrderList)](#22-货主获取收货单列表)
##### 23. [货主获取货单详情(getOrderDetail)](#23-货主获取货单详情)
##### 24. [货主修改预下单货单(modifyPreOrder)](#24-货主修改预下单货单)
##### 25. [货主删除预下单货单(removePreOrder)](#25-货主删除预下单货单)
##### 26. [货主通过分店发货线上支付(payForOrderWhenSend)](#26-货主通过分店发货线上支付)
##### 27. [货主通过收货点发货线上支付(payAgentForOrder)](#27-货主通过收货点发货线上支付)
##### 28. [货主结束货单现金支付(finishOrder)](#28-货主结束货单现金支付)
##### 29. [货主结束货单线上支付(payForOrderWhenReceive)](#29-货主结束货单线上支付)
##### 30. [获取订单物流信息(getLogisticsList)](#30-获取订单物流信息)
##### 31. [物流公司开始装车(startScanOrder)](#31-物流公司开始装车)
##### 32. [物流公司扫描货单(scanOrder)](#32-物流公司扫描货单)
##### 33. [物流公司获取入驻分店详情包括担保公司(getBranchShopInfo)](#33-物流公司获取入驻分店详情包括担保公司)
##### 34. [物流公司结束扫描货单(finishScanOrder)](#34-物流公司结束扫描货单)
##### 35. [物流公司继续扫描货单(continueScanOrder)](#35-物流公司继续扫描货单)
##### 36. [库管扫描入库(placeStorage)](#36-库管扫描入库)
## 货物api
##### 37. [物流公司获取竞得货物列表(getNeedSendOrderSummaryList)](#37-物流公司获取竞得货物列表)
##### 38. [物流公司获取竞得货物订单列表(getNeedSendOrderListByEndPoint)](#38-物流公司获取竞得货物订单列表)
##### 39. [物流公司获取成员列表(getMemberList)](#39-物流公司获取成员列表)
##### 40. [物流公司修改成员权限(modifyMemberAuthority)](#40-物流公司修改成员权限)
##### 41. [物流公司通过手机号搜索成员(getMemberByPhone)](#41-物流公司通过手机号搜索成员)
## 搬运队api
##### 42. [搬运队获取待扫描货车(getWaitScanTruckInfo)](#42-搬运队获取待扫描货车)
##### 43. [搬运队获取搬运历史(getCarryList)](#43-搬运队获取搬运历史)
##### 44. [搬运队修改搬运部信息(modifyOwnPartment)](#44-搬运队修改搬运部信息)
##### 45. [搬运部获取当前搬运货车信息(getWorkTruckInfo)](#45-搬运部获取当前搬运货车信息)
##### 46. [物流公司获取货搬运队列表(getCarryPartmentList)](#46-物流公司获取货搬运队列表)
##### 47. [物流公司选择搬运队(selectCarryPartment)](#47-物流公司选择搬运队)
##### 48. [物流公司付搬运费(payForCarryPartment)](#48-物流公司付搬运费)
## 卡车api
##### 49. [物流公司获取担保金额(getBondAmountInfo)](#49-物流公司获取担保金额)
##### 50. [物流公司创建卡车(createTruck)](#50-物流公司创建卡车)
##### 51. [物流公司获取未完成卡车信息(getUnfinishTruck)](#51-物流公司获取未完成卡车信息)
##### 52. [总部确认卡车认证通过(passExamineTruck)](#52-总部确认卡车认证通过)
##### 53. [物流公司获取卡车详情(getTruckDetail)](#53-物流公司获取卡车详情)
##### 54. [物流公司获取卡车列表(getWorkTruckList)](#54-物流公司获取卡车列表)
##### 55. [物流公司获取历史审核过的卡车列表(getHistoryTruckList)](#55-物流公司获取历史审核过的卡车列表)
## 司机api
##### 56. [司机获取当前运输中的货车信息(getOnWayTruck)](#56-司机获取当前运输中的货车信息)
##### 57. [司机上传位置信息(uploadLocation)](#57-司机上传位置信息)
##### 58. [司机确认到达(confirmReach)](#58-司机确认到达)
##### 59. [司机获取位置列表(getLocationList)](#59-司机获取位置列表)
## 路线api
##### 60. [物流公司发布路线(publishRoadmap)](#60-物流公司发布路线)
##### 61. [物流公司修改路线(modifyRoadmap)](#61-物流公司修改路线)
##### 62. [物流公司删除路线(removeRoadmap)](#62-物流公司删除路线)
##### 63. [物流公司修改路线价格(setRoadmapPrice)](#63-物流公司修改路线价格)
##### 64. [物流公司修改路线对应送货上门路线价格(setRoadmapSendDoorPrice)](#64-物流公司修改路线对应送货上门路线价格)
##### 65. [物流公司修改路线对应送货上门路线底格(setRoadmapSendDoorBasePrice)](#65-物流公司修改路线对应送货上门路线底格)
##### 66. [物流公司修改路线是否上线(setRoadmapEnable)](#66-物流公司修改路线是否上线)
##### 67. [物流公司修改路线对应的送货上门路线是否上线(setRoadmapSendDoorEnable)](#67-物流公司修改路线对应的送货上门路线是否上线)
##### 68. [物流公司获取路线列表(getRoadmapList)](#68-物流公司获取路线列表)
##### 69. [物流公司获取路线详情(getRoadmapDetail)](#69-物流公司获取路线详情)
##### 70. [货主获取分店路线列表(getRoadmapListInShop)](#70-货主获取分店路线列表)
##### 71. [货主获取收货点路线列表(getRoadmapListInAgent)](#71-货主获取收货点路线列表)
##### 72. [通过预下单群组id获取路线行情(getRoadmapListWithPreOrderGroup)](#72-通过预下单群组id获取路线行情)
##### 73. [通过预下单id获取路线行情(getRoadmapListWithPreOrder)](#73-通过预下单id获取路线行情)
##### 74. [通过终点获取路线行情(getRoadmapListWithEndPoint)](#74-通过终点获取路线行情)
##### 75. [修改预下单群组详情(modifyPreOrderGroup)](#75-修改预下单群组详情)
##### 76. [获取预下单群组中的预下单列表(getPreOrderListInGroup)](#76-获取预下单群组中的预下单列表)
## 分店api
##### 77. [收货部获取需要上传照片的订单(getLastestOrderByReceivePartment)](#77-收货部获取需要上传照片的订单)
##### 78. [收货部为货单上传照片(setPhotoForOriginOrder)](#78-收货部为货单上传照片)
##### 79. [收货部为货单下中转单(placeTransferOrder)](#79-收货部为货单下中转单)
##### 80. [收货部获取货单详情(getOrderDetailByReceivePartment)](#80-收货部获取货单详情)
##### 81. [收货部获取订单(getOrdersByReceivePartment)](#81-收货部获取订单)
##### 82. [库管部获取订单(getOrdersByWareHousePartment)](#82-库管部获取订单)
##### 83. [库管部获取正在装车的订单列表(getLoadingOrderList)](#83-库管部获取正在装车的订单列表)
##### 84. [库管部扫描入库(placeStorage)](#84-库管部扫描入库)
##### 85. [保安部检查卡车状态(checkTruckPass)](#85-保安部检查卡车状态)
##### 86. [分店获取成员列表(getMemberList)](#86-分店获取成员列表)
##### 87. [分店创建成员(createMember)](#87-分店创建成员)
##### 88. [分店创建成员(modifyMember)](#88-分店创建成员)
##### 89. [分店获取部门信息(getOwnPartmentInfo)](#89-分店获取部门信息)
##### 90. [获取统计信息(getStatistics)](#90-获取统计信息)
## 收货点
##### 91. [收货点通过手机号获取预下单列表(getPreOrderList)](#91-收货点通过手机号获取预下单列表)
##### 92. [收货点下单(placeOrder)](#92-收货点下单)
##### 93. [收货点修改货单(modifyOrder)](#93-收货点修改货单)
##### 94. [收货点打印二维码(printBarCode)](#94-收货点打印二维码)
##### 95. [收货点打印二货单(printOrderListBill)](#95-收货点打印二货单)
##### 96. [收货点代支付货单(confirmCachPayedForOrderList)](#96-收货点代支付货单)
##### 97. [收货点获取未完成货单(getLastestOrder)](#97-收货点获取未完成货单)
##### 98. [收货点获取订单(getOrders)](#98-收货点获取订单)
##### 99. [收货点获取订单详情(getOrderDetail)](#99-收货点获取订单详情)
##### 100. [收货点获取成员列表(getMemberList)](#100-收货点获取成员列表)
##### 101. [收货点修改成员权限(modifyMemberAuthority)](#101-收货点修改成员权限)
##### 102. [收货点通过手机号搜索成员(getMemberByPhone)](#102-收货点通过手机号搜索成员)
##### 103. [收货点获取关联店铺列表(getReferShopList)](#103-收货点获取关联店铺列表)
##### 104. [收货点设置关联店铺(setReferShop)](#104-收货点设置关联店铺)
##### 105. [收货点添加区域的路线提成(addRegionProfit)](#105-收货点添加区域的路线提成)
##### 106. [收货点获取区域的路线提成列表(getRegionProfitList)](#106-收货点获取区域的路线提成列表)
##### 107. [收货点获取区域的路线提成详情(getRegionProfitDetail)](#107-收货点获取区域的路线提成详情)
##### 108. [收货点修改区域的路线提成(modifyRegionProfit)](#108-收货点修改区域的路线提成)
##### 109. [收货点删除区域的路线提成(removeRegionProfit)](#109-收货点删除区域的路线提成)
##### 110. [收货点批量设置区域的路线提成(setRegionProfitWithList)](#110-收货点批量设置区域的路线提成)
## 同城配送
##### 111. [获取分店设置了上门自提竞价的物流公司列表(getClientPickShipperList)](#111-获取分店设置了上门自提竞价的物流公司列表)
##### 112. [短途物流公司扫描入库(placeStorage)](#112-短途物流公司扫描入库)
##### 113. [短途物流公司设置送货上门(setClientPickOrderTransportFee)](#113-短途物流公司设置送货上门)
##### 114. [短途物流公司设置货车(setCityTruck)](#114-短途物流公司设置货车)
##### 115. [短途物流公司获取货车详情(getCityTruckDetail)](#115-短途物流公司获取货车详情)
##### 116. [短途物流公司扫描装车(scanLoadOrderForCityDistribute)](#116-短途物流公司扫描装车)
##### 117. [短途物流公司获取货车的货单列表(getOrderListInCityTruck)](#117-短途物流公司获取货车的货单列表)
## 协议文档
##### 118. [用户协议(user)](#118-用户协议)
##### 119. [获取软件许可协议(software)](#119-获取软件许可协议)
##### 120. [关于(about)](#120-关于)

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
### 3. [发送验证码](#3-发送验证码requestsendverifycode)
- `requestSendVerifyCode`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| phone | String | 登录手机号码 |

```js
{
    "success": true
}
```

---

### 4. [修改支付密码](#4-修改支付密码setpaymentpassword)
- `setPaymentPassword`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| phone | String | 登录手机号码 |
| password | String | 新密码 |
| verifyCode | String | 验证码 |


```js
{
    "success": true
}
```

---
### 5. [修改登录密码](#5-修改登录密码findpassword)
- `findPassword`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| phone | String | 登录手机号码 |
| password | String | 新密码 |
| verifyCode | String | 验证码 |

```js
{
    "success": true
}
```

---

### 6. [获取账户余额](#6-获取账户余额getremainamount)
- `getRemainAmount`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |


```js
{
  "success": true,
  "context": {
    "amount": 98800
  }
}
```


---

### 7. [货主账户充值](#7-货主账户充值recharge)
- `recharge`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| amount | Number | 充值金额 |
| thirdpartyAccount | Number | 充值账户平台（第三方的账号  0：支付宝(A)， 1：财付通(C)，2：工商银行(G)，3：建设银行(J)， 4：中国银行（Z）， 5：农业银行(N)，规则，给定的缩写字母+账号） |


```js
{
  "success": true,
  "context": {
    "amount": 99097
  }
}
```

---

### 8. [货主账户提现](#8-货主账户提现withdraw)
- `withdraw`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| amount | Number | 提现金额 |
| thirdpartyAccount | Number | 提现账户平台（第三方的账号  0：支付宝(A)， 1：财付通(C)，2：工商银行(G)，3：建设银行(J)， 4：中国银行（Z）， 5：农业银行(N)，规则，给定的缩写字母+账号） |


```js
{
  "success": true,
  "context": {
    "amount": 99097
  }
}
```

---

### 9. [获取个人信息](#9-获取个人信息getpersonalinfo)
- `getPersonalInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |


```js
{
  "success": true,
  "context": {
    "phone": "18685192480",
    "email": "42550564@qq.com",
    "registerTime": "2017-08-08 19:43:00",
    "createTime": "2017-08-17T02:46:25.879Z",
    "authority": [
      {
        "20011": "搬货的权限"
      },
      {
        "10008": "修改所在部门信息的权限"
      },
      {
        "10001": "创建成员的权限"
      },
      {
        "10002": "修改成员信息的权限"
      },
      {
        "10003": "删除成员的权限"
      },
      {
        "10005": "充值的权限"
      },
      {
        "10006": "提现的权限"
      }
    ],
    "sex": 0,
    "partment": {
      "shopId": "5995496cd1df03410425a904",
      "name": "王大搬运部",
      "type": 2,
      "descript": "我们活着的意义就是移动",
      "phoneList": "0851-98989000;0851-98989001",
      "isBusy": false,
      "price": 100,
      "modifyTime": "2017-08-21 12:01:26",
      "workTruckId": "59a75bfd4b0e7a4e17cde5da",
      "createTime": "2017-08-18 09:24:02",
      "enable": true,
      "memberCount": 1,
      "chargeMan": {
        "phone": "18385192481",
        "name": "搬运工负责人",
        "id": "599641b27338534997932c44"
      },
      "id": "599641b27338534997932c45"
    },
    "shipper": {
      "name": "广顺达物流公司",
      "logo": "http://192.168.1.111:3000/api/image?id=5989a3c5b48db929e46ad6c4",
      "image": "http://192.168.1.111:3000/api/image?id=5989a3c5b48db929e46ad6c9",
      "sign": "这是一家非常专业的物流公司",
      "phoneList": "0851-86190987;18185192480",
      "address": "贵阳市花果园",
      "legalName": "方运江",
      "legalPhone": "18085192480",
      "createTime": "2017-08-08 19:43:01",
      "legalIDCard": [
        "http://192.168.1.111:3000/api/image?id=5989a3c5b48db929e46ad6cd",
        "http://192.168.1.111:3000/api/image?id=5989a3c5b48db929e46ad6ca"
      ],
      "acountAmount": 99930000,
      "capital": 10000000,
      "descriptList": [
        {
          "img": "http://192.168.1.111:3000/api/image?id=5989a3c5b48db929e46ad6cc",
          "text": "这是我们公司的车队"
        }
      ],
      "chairMan": {
        "phone": "18685192480",
        "id": "5989a3c4b48db929e46ad6b0",
        "phoneList": ""
      },
      "id": "5989a3c5b48db929e46ad6ce",
      "registerShopList": [
        {
          "shop": {
            "name": "华通物流超市",
            "address": {
              "name": "贵州省贵阳市南明区花果园街道花果园L2区"
            },
            "id": "5989a3c0b48db929e46ad68d"
          },
          "id": "5989a3c6b48db929e46ad6d0"
        }
      ]
    },
    "agent": {
      "name": "18684165865收货点",
      "logo": "http://localhost:3000/api/image?id=59f698e10931cf0b4e8b5262",
      "image": "http://localhost:3000/api/image?id=59f698e70931cf0b4e8b5265",
      "sign": "18684165865收货点18684165865收货点",
      "phoneList": "18684165865",
      "address": "贵州省贵阳市云岩区",
      "location": [
        106.709177,
        26.629907
      ],
      "legalName": "1868416586",
      "legalPhone": "18684165865",
      "createTime": "2017-10-30 11:13:59",
      "legalIDCard": [],
      "descriptList": [],
      "id": "59f698f70931cf0b4e8b5266",
      "chairMan": {
        "phone": "18684165865",
        "id": "59df0a26009deb7e9298e18f"
      },
      "referShop": {
        "name": "15555555501",
        "address": "贵州省贵阳市云岩区黔灵山分店",
        "id": "59e426b1b6ee191fe5d4fbf8"
      }
    },
    "phoneList": "",
    "userId": "5989a3c4b48db929e46ad6b0"
  }
}

```
###### authority的说明：
```
authority为用户权限:
(1)总部权限
    0:创建分店的权限
    1:修改分店信息的权限
    2:删除分店的权限
    3:查看分店列表的权限
    4:修改设置的权限
    5:创建收货点的权限
    6:修改收货点的权限
    7:删除收货点的权限
    8:查看收货点的权限
(2)公共权限
    10000:修改所在物流超市信息的权限
    10001:创建成员的权限
    10002:修改成员信息的权限
    10003:删除成员的权限
    10004:查看成员的权限
    10005:充值的权限
    10006:提现的权限
    10007:修改路线的提成的权限
(3)分店权限
    20000:创建物流公司的权限
    20001:修改物流公司的权限
    20002:删除物流公司的权限
    20003:查看物流公司的权限
    20004:创建部门的权限
    20005:修改部门信息的权限
    20006:删除部门的权限
    20007:查看部门的权限
    20008:收货的权限(收货部人员)
    20009:库管的权限(库管部人员)
    20010:搬货的权限(搬运部人员)
    20011:安检的权限(保安部人员)
    20012:配送的权限(配送部人员)
(4)物流公司
    30000:修改物流公司信息的权限
    30001:修改物流公司成员权限的权限
    30002:创建路线的权限
    30003:修改路线的权限
    30004:删除路线的权限
    30005:查看路线的权限
(5)收货点
    40000:修改收货点信息的权限
    40001:修改收货点成员权限的权限
    40002:收货点下单的权限

```
---

### 10. [修改个人信息](#10-修改个人信息modifypersonalinfo)
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

### 11. [修改密码](#11-修改密码modifypassword)
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

### 12. [意见反馈](#12-意见反馈submitfeedback)
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
### 13. [董事长修改参数](#13-董事长修改参数modifysettinginfo)
- `modifySettingInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| sizeWeightRate | number | 方量重量比 |
| insuanceRate | number | 保险费与保价的比 |
| insuanceMountRate | number | 保价与运费的比 |
| insuanceBaseValue | number | 保险的保底值 |
| proxyChargeProfitRate | number | 手续费与代收货款的比 |
| gradientPriceList | array | 阶梯价格的梯度 |
| additionalDeliverTime | number | 代签额外增加时间 |
| noticeShipperStorageWeight | number | 通知物流公司的吨数 |
| bondAmountWeightRate | number | 每吨货需要的保证金 |
| rankedMaskFirstRankWeight | number | 屏蔽物流公司货物的基数 |
| rankedMaskRateList | array | 屏蔽物流公司排名的梯度 |

```js
{
    "success": true,
}
```

---

### 14. [货主获取交易记录](#14-货主获取交易记录getbilllist)
- `getBillList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "count": 1,
    "billList": [
      {
        "tradeAmountYuan": 100000,
        "tradeTime": "2017-08-08 19:43:09",
        "tradeAmount": 10000000,
        "thirdpartyAccount": "A支付宝测试账号",
        "orderId": null,
        "remark": "个人充值"
      }
    ]
  }
}

```
---

### 15. [获取中国区划地址](#15-获取中国区划地址getregionaddress)
- `getRegionAddress`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| parentCode | Number | 上级编码 |
| type | Number | 0:长途地址,1:短途地址 |

```js
{
  "success": true,
  "context": {
    "addressList": [
      {
        "id": 1,
        "code": 11,
        "parentCode": 0,
        "name": "北京市",
        "level": 1
      }
    ]
  }
}
```
---

### 16. [获取发货点地址](#16-获取发货点地址getstartpointaddress)
- `getStartPointAddress`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| parentCode | Number | 上级编码 |
| region | String | 区划地址 |

```js
{
  "success": true,
  "context": {
    "addressList": [
      {
        "name": "华通物流超市",
        "address": {
          "name": "贵阳市石板镇"
        },
        "id": "59841315ad007024a2c0ce9b",
        "isShop": true
      }
    ]
  }
}
```
---

## 货单api

---
### 17. [货主预下单](#17-货主预下单placepreorder)
- `placePreOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 订单号 |
| shopId | ID | 指定的某家分店 |
| agentId | ID | 指定的某家收货点 |
| startPoint | String | 始发地 |
| receiverPhone | String | 收货人电话 |
| receiverName | String | 收货人姓名 |
| endPoint | String | 货到地 |
| totalNumbers | Number | 一票货总的件数 |
| weight | Number | 一票货总的重量 |
| size | Number | 一票货总的方量 |
| isSendToDoor | Bool | 是否送货上门 |
| proxyCharge | Number | 代收货款金额 |
| isInsuance | Bool | 是否保价 |
| isReachPay | Bool | 是否到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)|
| totalDesignatedFee | Number | 指定收款金额 |
| photo | String | 货单照片 |


```js
{
    "success": true,
}
```

---
### 18. [货主获取发货单列表](#18-货主获取发货单列表getsendorderlist)
- `getSendOrderList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| type | String | 类型 (preorder,waitpay,onway,complete)|
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |


```js
{
  "success": true,
  "context": {
    "orderList": [
      {
        "endPoint": "遵义市湄潭县新南镇",
        "createTime": "2017-08-03 10:51:53",
        "needPayInsuanceFee": 0,
        "needPayTransportFee": 600,
        "realFee": 0,
        "payTool": 0,
        "size": 1,
        "weight": 10,
        "totalNumbers": 5,
        "receiver": {
          "phone": "18885192481",
          "id": "59827a100bbe270ec8d9b4ab"
        },
        "id": "59828fc90bbe270ec8d9b4ae"
      }
    ]
  }
}
```

---
### 19. [物流公司获取已装货单卡车列表](#19-物流公司获取已装货单卡车列表gettrucks)
- `getTrucks`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |
| type | String | 类型 (onway,complete)|
| keyword | String | 关键字|
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |


```js
无type拉取所有
{
  "success": true,
  "context": {
    "onway": {
      "count": 1,
      "list": [
        {
          "plateNo": "贵A-778899",
          "drivingLicense": "A71236717",
          "truckType": "4.5米 高栏",
          "driverId": null,
          "locationList": [],
          "insuanceMount": 1000000,
          "driver": null,
          "id": "59954eb8d1df03410425a95f"
        }
      ]
    },
    "complete": {
      "count": 0,
      "list": []
    }
  }
}

如果有type（onway）
{
  "success": true,
  "context": {
    "onway": {
      "count": 1,
      "list": [
        {
          "plateNo": "贵A-778899",
          "drivingLicense": "A71236717",
          "truckType": "4.5米 高栏",
          "driverId": null,
          "locationList": [],
          "insuanceMount": 1000000,
          "driver": null,
          "id": "59954eb8d1df03410425a95f"
        }
      ]
    }
  }
}

```
---
### 20. [物流公司获取货车里的货单列表](#20-物流公司获取货车里的货单列表getorderlistintruck)
- `getOrderListInTruck`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| truckId | ID | 货车Id |


```js
{
  "success": true,
  "context": {
    "orderList": [
      {
        "shopId": "5995496cd1df03410425a904",
        "receiverName": "18885192481",
        "endPoint": "北京市东城区景山街道",
        "roadmapId": "59954975d1df03410425a94c",
        "createTime": "2017-08-18 14:56:08",
        "stateList": [
          {
            "state": 6,
            "count": 15,
            "_id": "599697d9f95a2f61737ef26e"
          },
          {
            "state": 5,
            "count": 49985,
            "_id": "59968fe4133e66604b4bb37b"
          }
        ],
        "needPayInsuanceFee": 0,
        "proxyCharge": 0,
        "totalDesignatedFee": 1200,
        "realFee": 1212,
        "payMode": 2,
        "isReachPay": false,
        "isInsuance": false,
        "isSendToDoor": false,
        "size": 1,
        "weight": 10,
        "totalNumbers": 5,
        "receiver": {
          "phone": "18885192481",
          "id": "59954981d1df03410425a952"
        },
        "sender": {
          "phone": "18885192480",
          "id": "59954981d1df03410425a951"
        },
        "id": "59968f88133e66604b4bb36e"
      },
      {
        "shopId": "5995496cd1df03410425a904",
        "receiverName": "18885192481",
        "endPoint": "北京市东城区景山街道",
        "roadmapId": "59954975d1df03410425a94c",
        "createTime": "2017-08-18 14:38:46",
        "stateList": [
          {
            "state": 6,
            "count": 5,
            "_id": "59968c87833c675ed00b9eb4"
          }
        ],
        "needPayInsuanceFee": 0,
        "proxyCharge": 0,
        "totalDesignatedFee": 1200,
        "realFee": 1212,
        "payMode": 2,
        "isReachPay": false,
        "isInsuance": false,
        "isSendToDoor": false,
        "size": 1,
        "weight": 10,
        "totalNumbers": 5,
        "receiver": {
          "phone": "18885192481",
          "id": "59954981d1df03410425a952"
        },
        "sender": {
          "phone": "18885192480",
          "id": "59954981d1df03410425a951"
        },
        "id": "59968b76833c675ed00b9ea6"
      }
    ]
  }
}

```

---
### 21. [物流公司通过司机手机号获取司机信息](#21-物流公司通过司机手机号获取司机信息getdriverinfobyphone)
- `getDriverInfoByPhone`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| phone | String | 司机手机号 |

```js
{
  "success": true,
  "context": {
    "phone": "18985192480",
    "email": "42550564@qq.com",
    "name": "方运江",
    "head": "http://192.168.1.111:3000/api/image?id=599577a97aad4147a4f845db",
    "birthday": "1982-02-25",
    "address": "贵州省贵阳市",
    "license": "599577a97aad4147a4f845db",
    "licenseNo": "123123123",
    "registerTime": "2017-08-17 18:54:04",
    "sex": 0,
    "age": 35,
    "id": "599575cc277608472d636e97",
    "phoneList": ""
  }
}
```

---
### 22. [货主获取收货单列表](#22-货主获取收货单列表getreceiveorderlist)
- `getReceiveOrderList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| type | String | 类型(all,onway,complete) |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |


```js
{
  "success": true,
  "context": {
    "orderList": [
      {
        "endPoint": "遵义市湄潭县新南镇",
        "embraceTime": "2017-08-03 10:51:53",
        "createTime": "2017-08-03 10:51:53",
        "needPayInsuanceFee": 0,
        "proxyCharge": 20000,
        "realFee": 0,
        "payMode": 2,
        "isSendToDoor": false,
        "size": 1,
        "weight": 10,
        "totalNumbers": 5,
        "receiver": {
          "phone": "18885192481",
          "id": "59827a100bbe270ec8d9b4ab"
        },
        "sender": {
          "phone": "18885192480",
          "id": "59827a100bbe270ec8d9b4aa"
        },
        "id": "59828fc90bbe270ec8d9b4ae"
      }
    ]
  }
}
```

---
### 23. [货主获取货单详情](#23-货主获取货单详情getorderdetail)
- `getOrderDetail`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |

```js
{
  "success": true,
  "context": {
    "shopId": "这个订单隶属于哪家分店（预下单的情况下为指定的某家分店）",
    "senderId": "发货的客户 Id (如果是物流公司，就是物流公司的负责人，如果是收货点，就是收货点的负责人)",
    "endPoint": "终点，精确到镇",
    "receivePartmentId": "收货的部门",
    "receiveMemberId": "具体收货制单的人员的Id",
    "modifyTime": "修改时间",
    "shipperChairManId": "物流公司董事长Id",
    "shipperId": "物流公司 Id",
    "startPoint": "省市（只对预下单有用，如果有 shopId 或 agentId 时，该字段无效）",
    "receiverName": "订单使用的收货人姓名",
    "createTime": "下单时间",
    "stateList": [
      {
        "count": 5,
        "state": 8,
        "_id": "598d495c108bc60cd2237d11"
      }
    ],
    "needPayInsuanceFee": '132,需要现场付的保险',
    "needPayTransportFee": '120,需要现场付的运费',
    "proxyCharge": '0,代收货款金额',
    "designatedFee": '0,指定收款 (只有payMode为到付的情况下，才可能有指定收款)',
    "totalDesignatedFee": '1200,全程总的指定收款(只有payMode为到付的情况下有效，实际为到付时向收货人收的运费)',
    "realFee": '1320,屏幕显示价格 fee+masterProfit+branchProfit',
    "branchProfit": '110,分部的提成',
    "masterProfit": '110,总部的提成',
    "fee": '1100,运输价格（物流公司实际价格，没有经过加价）（自动计算）',
    "payTool": '现付的情况下支付工具 0：现金支付（PT_CASH），1：支付宝（PT_ALIPAY），2：微信（PT_WEIXIN），3：银行卡（PT_BANK）',
    "payMode": '支付方式 0：现付（PM_IMMEDIATE），1：到付（PM_REACH），2：混合支付（PM_MIXED 指定收款的部分到付，其余的现付）',
    "isReachPay": '是否是到付',
    "insuanceFee": '132,担保保险的费用（客户实际付的钱）(必须现付)[初始订单]',
    "insuanceMount": '13200,担保保险额度（需要赔偿用户的钱）[初始订单]',
    "isInsuance": '是否保价 [初始订单]，如果保价，保价个度为运费的setting.insuanceMountRate倍',
    "isSendToDoor": '是否送货上门',
    "size": '方量',
    "weight": '重量',
    "totalNumbers": '总的货物的件数（具体的数量以这个数量为准）',
    "roadmapRankIndex": '该订单选择的路线在选择时的排序名次，交易成功后，需要根据这个名次修改 RoadmapMaskModel 的数据',
    "isSenderRepresentShipper": '发货人是否代表一家物流公司，如果是，需要更新物流公司的账目',
    "roadmap": {
      "shipperId": "物流公司 Id",
      "endPoint": "遵义市湄潭县新南镇",
      "branchDefaultProfitRate": '分部对路线的提成',
      "masterDefaultProfitRate": '总部对路线的提成 ',
      "id": "路线Id"
    },
    "receiver": {
      "phone": "18885192480",
      "email": "42550564@qq.com",
      "name": "方运江",
      "head": "http://192.168.1.111:3000/api/image?id=598aaa2d14573c3611f265fb",
      "id": "5989a3ccb48db929e46ad724",
      "phoneList": ""
    },
    "id": "598a67b8616e822a6131feaa"
  }
}

```
###### stateList.state的说明：
```
货主把货拉倒分店收货部，收货部对其点数，称重，量方量后填单，然后调用 < placeOriginOrder > 成功后为待选线路状态
-1：预下单状态，需要将货拉到分店后不显示（件数，重量，方量），由收货员填写确认后进入待选线路状态
0：待选线路（安装所填的数据，给货主显示路线排名，让货主选择一条路线，付款方式，是否交保险，是否代收货款，是否送货上门，选择后显示运费和其他费用的列表和总的费用，点击确认按钮，调用接口 < confirmSelectRoadmap >后，如果是需要付钱的进入待付款状态，否则进入待库存状态）。
1：待打印条形码
2：待扣款并打印打印货单（用来区别需要付款的情况）
3：待付款状态（1.货主在手机上刷新，使用支付宝等付款，成功后告知收货员，收货员刷新订单，显示收款成功，2.货主将现金给收货员，收货员点击代付款按钮，付款成功后，显示收款成功；之后进入待库存状态）
4：待打印货单
5：待库存（指收款成功后搬运人员将货物运到库管处的状态，交给库管成功后，状态进入待装车状态）
6：待装车（货物送到库管处，每收5吨货就通知物流公司，物流公司自己看有一车的时候会生成一个货车实例<包括司机和货车信息>到检查部审核，审核成功后，将车开到库管处，物流公司会通过竞价联系搬运部，搬运部搬运货物上车，物流公司需要一件一件的扫描，装车完成，需要给库管确认，库管确认之后，进入待出发状态）
7：待出发 （物流公司将车开到门卫处，门卫扫描驾驶员的二维码，点击放行按钮，进入运输中状态）
8：运输中 （运输过程中，驾驶员需要开启app，自动上传位置信息）
9：待交货
10：交货成功 （司机将货物送到目的地之后，会劝说收货人安装app或者扫描小程序，然后让他扫描货物，确认收货的件数，完成交接）
```

---
### 24. [货主修改预下单货单](#24-货主修改预下单货单modifypreorder)
- `modifyPreOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 订单号 |
| shopId | ID | 指定的某家分店 |
| agentId | ID | 指定的某家收货点 |
| startPoint | String | 始发地 |
| receiverPhone | String | 收货人电话 |
| receiverName | String | 收货人姓名 |
| endPoint | String | 货到地 |
| totalNumbers | Number | 一票货总的件数 |
| weight | Number | 一票货总的重量 |
| size | Number | 一票货总的方量 |
| isSendToDoor | Bool | 是否送货上门 |
| proxyCharge | Number | 代收货款金额 |
| isInsuance | Bool | 是否保价 |
| isReachPay | Bool | 是否到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)|
| totalDesignatedFee | Number | 指定收款金额 |
| photo | String | 货单照片 |

```js
{
  "success": true
}
```

---
### 25. [货主删除预下单货单](#25-货主删除预下单货单removepreorder)
- `removePreOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 订单号 |

```js
{
  "success": true
}
```

---
### 26. [货主通过分店发货线上支付](#26-货主通过分店发货线上支付payfororderwhensend)
- `payForOrderWhenSend`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderIdList | Array | 订单Id列表 |
| payPassword | String | 支付密码 |

```js
{
  "success": true
}
```

---
### 27. [货主通过收货点发货线上支付](#27-货主通过收货点发货线上支付payagentfororder)
- `payAgentForOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderIdList | Array | 订单Id列表 |
| payPassword | String | 支付密码 |

```js
{
  "success": true
}
```

---
### 28. [货主结束货单现金支付](#28-货主结束货单现金支付finishorder)
- `finishOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "shipperId": "59954973d1df03410425a946",
    "shopId": "5995496cd1df03410425a904",
    "plateNo": "贵A-778899",
    "drivingLicense": "A71236717",
    "truckType": "4.5米 高栏",
    "driverId": "599575cc277608472d636e97",
    "totalNumber": 0,
    "carryPartmentId": "599641b27338534997932c45",
    "scannerId": "59954972d1df03410425a928",
    "createTime": "2017-08-18 09:37:35",
    "unloadAllOrderList": [],
    "totalSize": 1,
    "totalWeight": 10,
    "totalNumbers": 81,
    "orderCount": 4,
    "orderList": [
      "599a4e0d42b345056bf4ece7",
      "599a4c2042b345056bf4ecda",
      "59968f88133e66604b4bb36e",
      "59968b76833c675ed00b9ea6"
    ],
    "state": 5,
    "locationList": [],
    "insuanceMount": 1000000,
    "carryPrice": 100,
    "id": "599644df7338534997932c4c"
  }
}
```

---
### 29. [货主结束货单线上支付](#29-货主结束货单线上支付payfororderwhenreceive)
- `payForOrderWhenReceive`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 订单Id |

```js
{
  "success": true
}
```

---
### 30. [获取订单物流信息](#30-获取订单物流信息getlogisticslist)
- `getLogisticsList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 订单Id |

```js
{
  "success": true,
  "logisticsList": [
    {
      "shopName": "华通物流超市",
      "address": "贵州省贵阳市南明区花果园街道花果园L2区",
      "locateTime": "2017-08-09T02:17:55.848Z"
    }
  ]
}
```

---
### 31. [物流公司开始装车](#31-物流公司开始装车startscanorder)
- `startScanOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| truckId | ID | 卡车Id |

```js
{
  "success": true
}
```

---
### 32. [物流公司扫描货单](#32-物流公司扫描货单scanorder)
- `scanOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| truckId | ID | 卡车Id |
| orderId | ID | 货单Id |
| count | Number | 件数 |

```js
{
  "success": true,
  "context": {
    "shipperId": "59954973d1df03410425a946",
    "shopId": "5995496cd1df03410425a904",
    "plateNo": "贵A-778899",
    "drivingLicense": "A71236717",
    "truckType": "4.5米 高栏",
    "driverId": "599575cc277608472d636e97",
    "totalNumber": 0,
    "carryPartmentId": "599641b27338534997932c45",
    "scannerId": "59954972d1df03410425a928",
    "createTime": "2017-08-18 09:37:35",
    "unloadAllOrderList": [
      {
        "endPoint": "北京市东城区景山街道",
        "totalNumbers": 5,
        "receiver": {
          "phone": "18885192481"
        },
        "sender": {
          "phone": "18885192480"
        },
        "id": "59968f88133e66604b4bb36e",
        "unloadNumber": 49999
      }
    ],
    "totalSize": 1,
    "totalWeight": 10,
    "totalNumbers": 11,
    "orderCount": 2,
    "state": 4,
    "locationList": [],
    "insuanceMount": 1000000,
    "carryPrice": 100,
    "id": "599644df7338534997932c4c",
    "latestOrder": {
      "endPoint": "北京市东城区景山街道",
      "createTime": "2017-08-18 14:56:08",
      "size": 1,
      "weight": 10,
      "totalNumbers": 5,
      "receiver": {
        "phone": "18885192481",
        "id": "59954981d1df03410425a952"
      },
      "sender": {
        "phone": "18885192480",
        "id": "59954981d1df03410425a951"
      },
      "id": "59968f88133e66604b4bb36e"
    }
  }
}
```

---
### 33. [物流公司获取入驻分店详情包括担保公司](#33-物流公司获取入驻分店详情包括担保公司getbranchshopinfo)
- `getBranchShopInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |

```js
{
  "success": true,
  "context": {
    "shopId": "5995496cd1df03410425a904",
    "shipperId": "59954973d1df03410425a946",
    "registerTime": "2017-08-17 15:44:52",
    "usedBondAmount": 0,
    "remainBondAmount": 1000000,
    "totalBondAmount": 1000000,
    "bondCompanyList": [
      {
        "capital": 100000000,
        "bondAmount": 1000000,
        "name": "华阳通运公司",
        "address": "贵阳市",
        "phoneList": "18085192580;18085192581",
        "legalName": "方运江",
        "legalPhone": "18085192480",
        "legalIDCard": [
          "http://192.168.1.111:3000/api/image?id=59954973d1df03410425a945",
          "http://192.168.1.111:3000/api/image?id=59954973d1df03410425a942"
        ],
        "certificate": [
          "http://192.168.1.111:3000/api/image?id=59954973d1df03410425a941",
          "http://192.168.1.111:3000/api/image?id=59954973d1df03410425a944",
          "http://192.168.1.111:3000/api/image?id=59954973d1df03410425a943",
          "http://192.168.1.111:3000/api/image?id=59954973d1df03410425a940"
        ]
      }
    ],
    "id": "59954974d1df03410425a948"
  }
}
```

---
### 34. [物流公司结束扫描货单](#34-物流公司结束扫描货单finishscanorder)
- `finishScanOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true
}
```

---
### 35. [物流公司继续扫描货单](#35-物流公司继续扫描货单continuescanorder)
- `continueScanOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true
}
```

---
### 36. [库管扫描入库](#36-库管扫描入库placestorage)
- `placeStorage`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 订单Id |
| count | Number | 货物件数 |

```js
{
  "success": true,
  "order": {
    "storeScannerId": "59954970d1df03410425a91a",
    "shopId": "5995496cd1df03410425a904",
    "senderId": "59954981d1df03410425a951",
    "receiverId": "59954981d1df03410425a952",
    "receiverName": "18885192481",
    "endPoint": "北京市东城区景山街道",
    "endPointLastCode": 110101002,
    "receivePartmentId": "59954970d1df03410425a917",
    "receiveMemberId": "59954970d1df03410425a916",
    "roadmapId": "59954975d1df03410425a94c",
    "shipperChairManId": "59954972d1df03410425a928",
    "shipperId": "59954973d1df03410425a946",
    "createTime": "2017-08-17 15:45:05",
    "stateList": [
      {
        "state": 5,
        "count": 1,
        "_id": "59954d7dd1df03410425a95e"
      },
      {
        "state": 4,
        "count": 4,
        "_id": "59954982d1df03410425a95a"
      }
    ],
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 1200,
    "proxyChargeProfit": 0,
    "proxyCharge": 0,
    "designatedFee": 0,
    "totalDesignatedFee": 1200,
    "realFee": 1200,
    "branchProfit": 100,
    "masterProfit": 100,
    "fee": 1000,
    "payTool": 0,
    "payMode": 2,
    "isReachPay": false,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": false,
    "isSendToDoor": false,
    "size": 1,
    "weight": 10,
    "totalNumbers": 5,
    "roadmapRankIndex": 0,
    "isSenderRepresentShipper": false,
    "id": "59954981d1df03410425a953"
  }
}
```
---

## 货物api

---
### 37. [物流公司获取竞得货物列表](#37-物流公司获取竞得货物列表getneedsendordersummarylist)
- `getNeedSendOrderSummaryList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |
| keyword | String | 关键字|
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "count": 1,
    "orderList": [
      {
        "orderCount": 6,
        "totalNumbers": 30,
        "totalSize": 6,
        "totalWeight": 60,
        "totalFee": 6060,
        "endPoint": "北京市东城区景山街道"
      }
    ]
  }
}
```
---
### 38. [物流公司获取竞得货物订单列表](#38-物流公司获取竞得货物订单列表getneedsendorderlistbyendpoint)
- `getNeedSendOrderListByEndPoint`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |
| endPoint | String | 终点|
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "count": 6,
    "orderList": [
      {
        "shopId": "5995496cd1df03410425a904",
        "receiverName": "18885192481",
        "endPoint": "北京市东城区景山街道",
        "createTime": "2017-08-18 14:38:15",
        "stateList": [
          {
            "state": 5,
            "count": 5,
            "_id": "59968b57833c675ed00b9e7d"
          }
        ],
        "needPayInsuanceFee": 0,
        "proxyCharge": 0,
        "totalDesignatedFee": 1200,
        "realFee": 1212,
        "payMode": 2,
        "isReachPay": false,
        "isInsuance": false,
        "isSendToDoor": false,
        "size": 1,
        "weight": 10,
        "totalNumbers": 5,
        "roadmap": {
          "endPoint": "北京市东城区景山街道",
          "branchDefaultProfitRate": null,
          "masterDefaultProfitRate": 0.1,
          "id": "59954975d1df03410425a94c"
        },
        "receiver": {
          "phone": "18885192481",
          "id": "59954981d1df03410425a952"
        },
        "sender": {
          "phone": "18885192480",
          "id": "59954981d1df03410425a951"
        },
        "id": "59968b57833c675ed00b9e74"
      },
      {
        "shopId": "5995496cd1df03410425a904",
        "receiverName": "18885192481",
        "endPoint": "北京市东城区景山街道",
        "createTime": "2017-08-18 14:38:32",
        "stateList": [
          {
            "state": 5,
            "count": 5,
            "_id": "59968b68833c675ed00b9e87"
          }
        ],
        "needPayInsuanceFee": 0,
        "proxyCharge": 0,
        "totalDesignatedFee": 1200,
        "realFee": 1212,
        "payMode": 2,
        "isReachPay": false,
        "isInsuance": false,
        "isSendToDoor": false,
        "size": 1,
        "weight": 10,
        "totalNumbers": 5,
        "roadmap": {
          "endPoint": "北京市东城区景山街道",
          "branchDefaultProfitRate": null,
          "masterDefaultProfitRate": 0.1,
          "id": "59954975d1df03410425a94c"
        },
        "receiver": {
          "phone": "18885192481",
          "id": "59954981d1df03410425a952"
        },
        "sender": {
          "phone": "18885192480",
          "id": "59954981d1df03410425a951"
        },
        "id": "59968b68833c675ed00b9e7e"
      },
      {
        "shopId": "5995496cd1df03410425a904",
        "receiverName": "18885192481",
        "endPoint": "北京市东城区景山街道",
        "createTime": "2017-08-18 14:38:43",
        "stateList": [
          {
            "state": 5,
            "count": 5,
            "_id": "59968b73833c675ed00b9e91"
          }
        ],
        "needPayInsuanceFee": 0,
        "proxyCharge": 0,
        "totalDesignatedFee": 1200,
        "realFee": 1212,
        "payMode": 2,
        "isReachPay": false,
        "isInsuance": false,
        "isSendToDoor": false,
        "size": 1,
        "weight": 10,
        "totalNumbers": 5,
        "roadmap": {
          "endPoint": "北京市东城区景山街道",
          "branchDefaultProfitRate": null,
          "masterDefaultProfitRate": 0.1,
          "id": "59954975d1df03410425a94c"
        },
        "receiver": {
          "phone": "18885192481",
          "id": "59954981d1df03410425a952"
        },
        "sender": {
          "phone": "18885192480",
          "id": "59954981d1df03410425a951"
        },
        "id": "59968b73833c675ed00b9e88"
      }
    ]
  }
}
```

---
### 39. [物流公司获取成员列表](#39-物流公司获取成员列表getmemberlist)
- `getMemberList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "memberList": [
    {
      "phone": "18685192481",
      "email": "42550564@qq.com",
      "registerTime": "2017-08-17 15:44:50",
      "shipperId": "59954973d1df03410425a946",
      "createTime": "2017-08-17T07:44:50.524Z",
      "authority": [
        30001,
        30002,
        30003
      ],
      "sex": 0,
      "id": "59954972d1df03410425a929"
    },
    {
      "phone": "18685192480",
      "email": "42550564@qq.com",
      "registerTime": "2017-08-17 15:44:50",
      "shipperId": "59954973d1df03410425a946",
      "name": "朱元璋",
      "address": "北京市西城区西长安街街道",
      "createTime": "2017-08-17T07:44:50.522Z",
      "authority": [
        30000,
        30001,
        30002,
        30003,
        30004,
        30005,
        30006,
        30007,
        30008,
        30009,
        30010,
        30011,
        10005,
        10006
      ],
      "sex": 1,
      "id": "59954972d1df03410425a928"
    }
  ]
}
```

---
### 40. [物流公司修改成员权限](#40-物流公司修改成员权限modifymemberauthority)
- `modifyMemberAuthority`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| memberId | ID | 成员Id |
| authority | Array | 权限列表Id |

```js
{
  "success": true,
  "context": {
    "phone": "18685192481",
    "email": "42550564@qq.com",
    "registerTime": "2017-08-17 15:44:50",
    "shipperId": "59954973d1df03410425a946",
    "createTime": "2017-08-17T07:44:50.524Z",
    "authority": [
      30001,
      30002,
      30003
    ],
    "sex": 0,
    "id": "59954972d1df03410425a929"
  }
}
```

---
### 41. [物流公司通过手机号搜索成员](#41-物流公司通过手机号搜索成员getmemberbyphone)
- `getMemberByPhone`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| memberPhone | String | 成员手机号 |

```js
{
  "success": true,
  "context": {
    "phone": "13312341234",
    "email": "42550564@qq.com",
    "registerTime": "2017-08-30 19:41:55",
    "createTime": "2017-08-30T11:41:55.543Z",
    "authority": [],
    "sex": 0,
    "id": "59a6a483523fa0838161fd27"
  }
}
```

---

## 搬运队api

---
### 42. [搬运队获取待扫描货车](#42-搬运队获取待扫描货车getwaitscantruckinfo)
- `getWaitScanTruckInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "truckList": [
      {
        "plateNo": "贵A-778899",
        "drivingLicense": "A71236717",
        "createTime": "2017-08-30 17:22:07",
        "totalWeight": 10,
        "orderCount": 1,
        "id": "59a683bf48382f73b5977cb7"
      }
    ]
  }
}
```

---
### 43. [搬运队获取搬运历史](#43-搬运队获取搬运历史getcarrylist)
- `getCarryList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "count": 6,
    "truckList": [
      {
        "plateNo": "贵A-778899",
        "drivingLicense": "A71236717",
        "createTime": "2017-08-30 17:22:59",
        "totalWeight": 10,
        "orderCount": 1,
        "id": "59a683f391f69d7410de75e1"
      },
      {
        "plateNo": "123",
        "drivingLicense": "123",
        "createTime": "2017-08-25 16:12:43",
        "totalWeight": 10,
        "orderCount": 1,
        "id": "599fdbfb0db48253c419d810"
      },
      {
        "plateNo": "1234",
        "drivingLicense": "123456789",
        "createTime": "2017-08-25 16:09:07",
        "totalWeight": 10,
        "orderCount": 1,
        "id": "599fdb23ff2ce453630e6e11"
      }
    ]
  }
}
```

---
### 44. [搬运队修改搬运部信息](#44-搬运队修改搬运部信息modifyownpartment)
- `modifyOwnPartment`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| price | Number | 单价 |
| enable | Bool | 是否在线 |

```js
{
  "success": true,
  "context": {
    "name": "搬运部",
    "descript": "我们活着的意义就是移动",
    "phoneList": "0851-98989000;0851-98989001",
    "price": 100,
    "enable": true,
    "chargeMan": {
      "phone": "18385192480",
      "name": "搬运工负责人",
      "id": "59954970d1df03410425a91c",
      "phoneList": ""
    },
    "id": "59954970d1df03410425a91d"
  }
}
```

---
### 45. [搬运部获取当前搬运货车信息](#45-搬运部获取当前搬运货车信息getworktruckinfo)
- `getWorkTruckInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "shipperId": "59954973d1df03410425a946",
    "shopId": "5995496cd1df03410425a904",
    "plateNo": "贵A-778899",
    "drivingLicense": "A71236717",
    "truckType": "4.5米 高栏",
    "driverId": "599575cc277608472d636e97",
    "carryPartmentId": "599641b27338534997932c45",
    "createTime": "2017-08-31 08:44:45",
    "unloadAllOrderList": [],
    "totalSize": 0,
    "totalWeight": 0,
    "totalNumbers": 0,
    "orderCount": 0,
    "orderList": [],
    "state": 3,
    "locationList": [],
    "insuanceMount": 1000000,
    "carryPrice": 100,
    "id": "59a75bfd4b0e7a4e17cde5da"
  }
}
```

---
### 46. [物流公司获取货搬运队列表](#46-物流公司获取货搬运队列表getcarrypartmentlist)
- `getCarryPartmentList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| truckId | ID | 卡车Id |


```js
{
  "success": true,
  "context": {
    "partmenList": [
      {
        "shopId": "5995496cd1df03410425a904",
        "chargeManId": "59954970d1df03410425a91c",
        "name": "搬运部",
        "type": 2,
        "descript": "我们活着的意义就是移动",
        "phoneList": "0851-98989000;0851-98989001",
        "price": 100,
        "modifyTime": "2017-08-17 16:23:00",
        "createTime": "2017-08-17 15:44:48",
        "isBusy": false,
        "enable": true,
        "id": "59954970d1df03410425a91d"
      }
    ]
  }
}
```

---
### 47. [物流公司选择搬运队](#47-物流公司选择搬运队selectcarrypartment)
- `selectCarryPartment`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| truckId | ID | 卡车Id |
| carryPartmentId | ID | 搬运队Id |


```js
{
  "success": true
}
```

---
### 48. [物流公司付搬运费](#48-物流公司付搬运费payforcarrypartment)
- `payForCarryPartment`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true
}
```

---

## 卡车api

---
### 49. [物流公司获取担保金额](#49-物流公司获取担保金额getbondamountinfo)
- `getBondAmountInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |

```js
{
  "success": true,
  "context": {
    "totalBondAmount": 1000000,
    "remainBondAmount": 1000000
  }
}
```

---
### 50. [物流公司创建卡车](#50-物流公司创建卡车createtruck)
- `createTruck`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |
| plateNo | String | 车牌号 |
| drivingLicense | String | 驾照号 |
| truckType | String | 卡车类型 |
| insuanceMount | Number | 保险 |
| driverId | ID | 司机Id |

```js
{
  "success": true,
  "context": {
    "plateNo": "贵A-778899",
    "drivingLicense": "A71236717",
    "truckType": "4.5米 高栏",
    "driverId": null,
    "locationList": [],
    "insuanceMount": 1000000,
    "driver": null,
    "id": "59954eb8d1df03410425a95f"
  }
}
```


### 51. [物流公司获取未完成卡车信息](#51-物流公司获取未完成卡车信息getunfinishtruck)
- `getUnfinishTruck`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |

```js
{
  "success": true,
  "context": {
    "shipperId": "59954973d1df03410425a946",
    "shopId": "5995496cd1df03410425a904",
    "plateNo": "贵A-778899",
    "drivingLicense": "A71236717",
    "truckType": "4.5米 高栏",
    "driverId": "599575cc277608472d636e97",
    "carryPartmentId": "599641b27338534997932c45",
    "scannerId": "59954972d1df03410425a928",
    "createTime": "2017-08-21 11:59:11",
    "unloadAllOrderList": [],
    "totalSize": 1,
    "totalWeight": 10,
    "totalNumbers": 13,
    "orderCount": 1,
    "state": 5,
    "locationList": [],
    "insuanceMount": 1000000,
    "carryPrice": 100,
    "id": "599a5a8f42b345056bf4ecfe",
    "latestOrder": {
      "endPoint": "北京市东城区景山街道",
      "createTime": "2017-08-21 11:58:32",
      "size": 1,
      "weight": 10,
      "totalNumbers": 5,
      "receiver": {
        "phone": "18885192481",
        "id": "59954981d1df03410425a952"
      },
      "sender": {
        "phone": "18885192480",
        "id": "59954981d1df03410425a951"
      },
      "id": "599a5a6842b345056bf4ecf8"
    }
  }
}
```

---
### 52. [总部确认卡车认证通过](#52-总部确认卡车认证通过passexaminetruck)
- `passExamineTruck`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| truckId | ID | 卡车Id |

```js
{
  "success": true
}

```

---
### 53. [物流公司获取卡车详情](#53-物流公司获取卡车详情gettruckdetail)
- `getTruckDetail`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| truckId | ID | 卡车Id |

```js
{
  "success": true,
  "context": {
    "shipperId": "59954973d1df03410425a946",
    "shopId": "5995496cd1df03410425a904",
    "plateNo": "贵A-778899",
    "drivingLicense": "A71236717",
    "truckType": "4.5米 高栏",
    "driverId": null,
    "createTime": "2017-08-17 16:07:20",
    "unloadAllOrderList": [],
    "totalSize": 0,
    "totalWeight": 0,
    "totalNumber": 0,
    "orderCount": 0,
    "orderList": [],
    "state": 1,
    "locationList": [],
    "insuanceMount": 1000000,
    "carryPrice": 0,
    "driver": null,
    "id": "59954eb8d1df03410425a95f"
  }
}

```
---
### 54. [物流公司获取卡车列表](#54-物流公司获取卡车列表getworktrucklist)
- `getWorkTruckList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |
| keyword | String | 关键字 |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
没开始装车时：
{
  "success": true,
  "context": {
    "count": 2,
    "truckList": [
      {
        "plateNo": "贵A-778899",
        "drivingLicense": "A71236717",
        "truckType": "4.5米 高栏",
        "driverId": null,
        "state": 1,
        "locationList": [],
        "insuanceMount": 1000000,
        "driver": null,
        "id": "59954eb8d1df03410425a95f"
      },
      {
        "plateNo": "贵A-778899",
        "drivingLicense": "A71236717",
        "truckType": "4.5米 高栏",
        "driverId": null,
        "state": 0,
        "locationList": [],
        "insuanceMount": 1000000,
        "driver": null,
        "id": "59954976d1df03410425a94f"
      }
    ]
  }
}

没装完车时：
{
  "success": true,
  "context": {
    "truck": {
      "shipperId": "59954973d1df03410425a946",
      "shopId": "5995496cd1df03410425a904",
      "plateNo": "贵A-778899",
      "drivingLicense": "A71236717",
      "truckType": "4.5米 高栏",
      "driverId": "59954975d1df03410425a94e",
      "carryPartmentId": "59954970d1df03410425a91d",
      "scannerId": "59954972d1df03410425a928",
      "createTime": "2017-08-17 16:07:20",
      "unloadAllOrderList": [
        "59954981d1df03410425a953"
      ],
      "totalSize": 1,
      "totalWeight": 10,
      "totalNumber": 1,
      "orderCount": 1,
      "orderList": [
        "59954981d1df03410425a953"
      ],
      "state": 4,
      "locationList": [],
      "insuanceMount": 1000000,
      "carryPrice": 100,
      "id": "59954eb8d1df03410425a95f"
    }
  }
}

```
###### state的说明：
```
0：待审核
1：待入库
2：待找搬运队
3：待扫描装车
4：装车中
5：待付搬运部的款
6：待出库
7：运输中
8：完成
```

---
### 55. [物流公司获取历史审核过的卡车列表](#55-物流公司获取历史审核过的卡车列表gethistorytrucklist)
- `getHistoryTruckList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |
| keyword | String | 关键字 |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "count": 1,
    "truckList": [
      {
        "plateNo": "贵A-778899",
        "drivingLicense": "A71236717",
        "truckType": "4.5米 高栏",
        "locationList": [],
        "insuanceMount": 1000000,
        "driver": {
          "phone": "18985192480",
          "name": "方运江",
          "licenseNo": "123123123",
          "id": "599575cc277608472d636e97"
        },
        "id": "5997ef0474487466abd0dcac"
      }
    ]
  }
}
```

---

## 司机api

---
### 56. [司机获取当前运输中的货车信息](#56-司机获取当前运输中的货车信息getonwaytruck)
- `getOnWayTruck`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "shipperId": "59954973d1df03410425a946",
    "shopId": "5995496cd1df03410425a904",
    "plateNo": "贵AA1234",
    "drivingLicense": "1234567890123",
    "truckType": "5.5米平板",
    "driverId": "599575cc277608472d636e97",
    "carryPartmentId": "59954970d1df03410425a91d",
    "scannerId": "59954972d1df03410425a928",
    "createTime": "2017-08-25 15:03:56",
    "unloadAllOrderList": [],
    "totalSize": 1.4,
    "totalWeight": 14,
    "totalNumbers": 5,
    "orderCount": 1,
    "orderList": [
      "599e7fabead9861cc2a8e213"
    ],
    "state": 7,
    "locationList": [],
    "insuanceMount": 100000,
    "carryPrice": 100,
    "id": "599fcbdc89a721507f4a990c"
  }
}
```

---
### 57. [司机上传位置信息](#57-司机上传位置信息uploadlocation)
- `uploadLocation`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| address | String | 地址 |
| longitude | ID | 经度 |
| latitude | ID | 维度 |

```js
{
  "success": true
}
```

---
### 58. [司机确认到达](#58-司机确认到达confirmreach)
- `confirmReach`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true
}
```

---
### 59. [司机获取位置列表](#59-司机获取位置列表getlocationlist)
- `getLocationList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "locationList": [
    {
      "time": "2017-09-08 14:32:53",
      "latitude": 26.561979,
      "longitude": 106.684821,
      "address": "贵州省贵阳市南明区花果园街道花果园L1区"
    },
    {
      "time": "2017-09-08 14:29:33",
      "latitude": 26.561979,
      "longitude": 106.684821,
      "address": "贵州省贵阳市南明区花果园街道花果园L1区"
    }
  ]
}
```

---

## 路线api

---
### 60. [物流公司发布路线](#60-物流公司发布路线publishroadmap)
- `publishRoadmap`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |
| endPoint | String | 终点 |
| endPointLastCode | Number | 终点编码 |
| price | Number | 运费单价 |
| basePrice | Number | 运费保底价 |
| minFee | Number | 运费起价 |
| baseMinFee | Number | 起步价保底价 |
| sendToDoorPrice | Number | 送货上门单价 |
| baseSendToDoorPrice | Number | 送货上门单价保底价 |
| sendToDoorMinFee | Number | 送货上门起价 |
| baseSendToDoorMinFee | Number | 送货上门起步价保底价 |

```js
{
  "success": true
}
```
---
### 61. [物流公司修改路线](#61-物流公司修改路线modifyroadmap)
- `modifyRoadmap`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| roadmapId | ID | 路线 |
| endPoint | String | 终点 |
| endPointLastCode | Number | 终点编码 |
| price | Number | 运费单价 |
| basePrice | Number | 运费保底价 |
| minFee | Number | 运费起价 |
| baseMinFee | Number | 起步价保底价 |
| sendToDoorPrice | Number | 送货上门单价 |
| baseSendToDoorPrice | Number | 送货上门单价保底价 |
| sendToDoorMinFee | Number | 送货上门起价 |
| baseSendToDoorMinFee | Number | 送货上门起步价保底价 |
| enable | Bool | 是否打开路线 |

```js
{
  "success": true
}
```

---
### 62. [物流公司删除路线](#62-物流公司删除路线removeroadmap)
- `removeRoadmap`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| roadmapId | ID | 路线 |

```js
{
  "msg": "你没有删除路线的权限"
}
```

---
### 63. [物流公司修改路线价格](#63-物流公司修改路线价格setroadmapprice)
- `setRoadmapPrice`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| roadmapIdList | Array | 路线Id列表 |
| typeList | Array | 修改类型列表（0：长途运费单价  1：长途运费起价 ） |
| mode | Number | 修改模式 （0: 按照价格调整  1: 按照百分比调整  2: 按照名次调整）|
| value | Number | 修改量（如果mode为0和1时，正数为增加，负数为减少） |

```js
{
  "success": true
}
```

---
### 64. [物流公司修改路线对应送货上门路线价格](#64-物流公司修改路线对应送货上门路线价格setroadmapsenddoorprice)
- `setRoadmapSendDoorPrice`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| roadmapId | ID | 路线Id |
| lastCodeList | Array | 送货上门终点lastCode列表 |
| typeList | Array | 数组，0：送货上门单价  1：送货上门起价 |
| mode | Number | 修改模式 （0: 按照价格调整  1: 按照百分比调整  2: 按照名次调整）|
| value | Number | 修改量（如果mode为0和1时，正数为增加，负数为减少） |

```js
{
  "success": true
}
```

---
### 65. [物流公司修改路线对应送货上门路线底格](#65-物流公司修改路线对应送货上门路线底格setroadmapsenddoorbaseprice)
- `setRoadmapSendDoorBasePrice`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| roadmapId | ID | 路线Id |
| lastCodeList | Array | 送货上门终点lastCode列表 |
| typeList | Array | 数组，0：同城配送运费单价的底价  1：同城配送运费起价的底价 |
| mode | Number | 修改模式 （0: 设置底价, 1: 按照价格调整  2: 按照百分比调整）|
| value | Number | 修改量（如果mode为0和1时，正数为增加，负数为减少） |

```js
{
  "success": true
}
```

---
### 66. [物流公司修改路线是否上线](#66-物流公司修改路线是否上线setroadmapenable)
- `setRoadmapEnable`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| roadmapIdList | Array | 路线Id列表 |
| enable | Boolean | 是否上线 |

```js
{
  "success": true
}
```

---
### 67. [物流公司修改路线对应的送货上门路线是否上线](#67-物流公司修改路线对应的送货上门路线是否上线setroadmapsenddoorenable)
- `setRoadmapSendDoorEnable`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| roadmapId | ID | 路线Id |
| lastCodeList | Array | 送货上门终点lastCode列表 |
| enable | Boolean | 是否上线 |

```js
{
  "success": true
}
```

---
### 68. [物流公司获取路线列表](#68-物流公司获取路线列表getroadmaplist)
- `getRoadmapList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "count": 1,
    "roadmapList": [
      {
        "endPoint": "北京市东城区景山街道",
        "endPointLastCode": 110101002,
        "selfSignRate": 0,
        "duration": 0,
        "sendToDoorMinFee": 50,
        "sendToDoorPrice": 100,
        "minFee": 50,
        "price": 100,
        "startPoint": "贵州省贵阳市南明区花果园街道花果园L2区",
        "masterDefaultProfitRate": 0.1,
        "id": "59954975d1df03410425a94c"
      }
    ]
  }
}
```

---
### 69. [物流公司获取路线详情](#69-物流公司获取路线详情getroadmapdetail)
- `getRoadmapDetail`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| roadmapId | ID | 路线 |

```js
{
  "success": true,
  "context": {
    "shipperId": "59954973d1df03410425a946",
    "shipperInBranchShopId": "59954974d1df03410425a948",
    "endPoint": "北京市东城区景山街道",
    "endPointLastCode": null,
    "basePrice": null,
    "baseMinFee": null,
    "baseSendToDoorPrice": 40,
    "baseSendToDoorMinFee": null,
    "modifyTime": "2017-08-21 10:58:35",
    "createTime": "2017-08-19 10:14:07",
    "monthTradeTimes": 0,
    "tradeTimes": 0,
    "selfSignRate": 0,
    "duration": 0,
    "updateSendToDoorMinFeeTime": "2017-08-19T02:14:07.584Z",
    "sendToDoorMinFee": 40,
    "updateSendToDoorPriceTime": "2017-08-19T02:14:07.584Z",
    "sendToDoorPrice": 20,
    "updateMinPriceTime": "2017-08-19T02:14:07.584Z",
    "minFee": 10,
    "updatePriceTime": "2017-08-19T02:14:07.584Z",
    "price": 300,
    "enable": true,
    "shop": {
      "name": "华通物流超市",
      "address": "贵州省贵阳市南明区花果园街道花果园L2区",
      "id": "5995496cd1df03410425a904"
    },
    "sendDoorList": [
        {
            "sendDoorEndPoint": "城关镇",
            "sendDoorEndPointLastCode": 520121100,
            "enable": true,
            "sendDoorPrice": 110,
            "sendDoorMinFee": 60,
            "baseSendDoorPrice": 0,
            "baseSendDoorMinFee": 0,
            "updateSendDoorPriceTime": "2017-11-04 09:40:10",
            "updateSendDoorMinFeeTime": "2017-11-04 09:40:10",
            "_id": "59fd1a7aa5b6e90e5619f1a8"
        }
    ],
    "startPoint": "贵州省贵阳市南明区花果园街道花果园L2区",
    "masterDefaultProfitRate": 0.1,
    "id": "59979eef74487466abd0dc8b"
  }
}
```

---
### 70. [货主获取分店路线列表](#70-货主获取分店路线列表getroadmaplistinshop)
- `getRoadmapListInShop`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| region | String | 地址 |
| branchShopId | String | 分店Id |
| location | Array | 坐标 |
| orderBy | Number | 排序类型 |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "roadmapList": [
      {
        "endPoint": "遵义市湄潭县新南镇",
        "selfSignRate": 0,
        "duration": 0,
        "sendToDoorPrice": 10,
        "price": 20,
        "id": "598965bfbdfa331feb32ce12",
        "shipper": {
          "name": "广顺达物流公司"
        },
        "shop": {
          "_id": "598965b8bdfa331feb32cdb6",
          "name": "华通物流超市",
          "distance": 10.888834907682734
        }
      },
      {
        "endPoint": "遵义市湄潭县新南镇",
        "selfSignRate": 0,
        "duration": 0,
        "sendToDoorPrice": 10,
        "price": 100,
        "id": "598965bdbdfa331feb32cdef",
        "shipper": {
          "name": "广顺达物流公司"
        },
        "shop": {
          "_id": "598965b6bdfa331feb32cda8",
          "name": "华通物流超市",
          "distance": 0.6651347015721002
        }
      }
    ]
  }
}

```

---
### 71. [货主获取收货点路线列表](#71-货主获取收货点路线列表getroadmaplistinagent)
- `getRoadmapListInAgent`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| region | String | 地址 |
| agentId | String | 收货点Id |
| location | Array | 坐标 |
| orderBy | Number | 排序类型 |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "roadmapList": [
      {
        "endPoint": "遵义市湄潭县新南镇",
        "selfSignRate": 0,
        "duration": 0,
        "sendToDoorPrice": 15,
        "price": 150,
        "shipper": {
          "name": "广顺达物流公司",
          "id": "598965bbbdfa331feb32cde9"
        },
        "id": "598965bdbdfa331feb32cdef",
        "distance": 18.138418524923516,
        "agent": {
          "name": "后巢乡收货点"
        }
      },
      {
        "endPoint": "遵义市湄潭县新南镇",
        "selfSignRate": 0,
        "duration": 0,
        "sendToDoorPrice": 15,
        "price": 150,
        "shipper": {
          "name": "广顺达物流公司",
          "id": "598965bbbdfa331feb32cde9"
        },
        "id": "598965bdbdfa331feb32cdef",
        "distance": 18.138418524923516,
        "agent": {
          "name": "后巢乡收货点"
        }
      }
    ]
  }
}
```

---
### 72. [通过预下单群组id获取路线行情](#72-通过预下单群组id获取路线行情getroadmaplistwithpreordergroup)
- `getRoadmapListWithPreOrderGroup`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderGroupId | ID | 货单群组Id |
| location | Array | 坐标 |

```js
{
  "success": true,
  "context": {
    "shopList": [
      {
        "totalTransportFee": 0,
        "name": "华通物流超市",
        "address": "贵州省贵阳市南明区花果园街道花果园L2区",
        "distance": 0.6651347015721002,
        "unfindOrderList": [
          {
            "endPoint": "内蒙古自治区通辽市库伦旗额勒顺镇",
            "proxyCharge": 1,
            "totalDesignatedFee": 1,
            "isReachPay": false,
            "isSendToDoor": false,
            "size": 1,
            "weight": 1,
            "id": "599f8c4a206d103b2a3de444"
          },
          {
            "endPoint": "北京市东城区景山街道",
            "proxyCharge": 0,
            "totalDesignatedFee": 1200,
            "isReachPay": true,
            "isSendToDoor": false,
            "size": 1,
            "weight": 10,
            "id": "599d4d58d4ad6318177f05dc"
          }
        ]
      }
    ]
  }
}
```

---
### 73. [通过预下单id获取路线行情](#73-通过预下单id获取路线行情getroadmaplistwithpreorder)
- `getRoadmapListWithPreOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |
| location | Array | 坐标 |
| orderBy | Number | 排序类型 |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "count": 0,
    "roadmapList": []
  }
}
```

---
### 74. [通过终点获取路线行情](#74-通过终点获取路线行情getroadmaplistwithendpoint)
- `getRoadmapListWithEndPoint`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |
| agentId | ID | 收货点Id |
| startPoint | String | 范围，如贵州省贵阳市，如果有agentId，该字段无效 |
| endPointLastCode | String | 终点code |
| sendDoorEndPointLastCode | String | 送货上门终点code |
| isSendDoor | Bool | 是否送货上门 |
| location | Array | 坐标 |
| orderBy | Number | 排序类型 |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "shop": {
      "count": 1,
      "roadmapList": [
        {
          "endPoint": "北京市东城区交道口街道",
          "selfSignRate": 0,
          "duration": 0,
          "shipperName": "广顺达物流公司",
          "shopName": "华通物流超市",
          "shopAddress": "贵州省贵阳市南明区花果园街道花果园L2区",
          "id": "599f78ec206d103b2a3de434",
          "price": 0,
          "minFee": 0,
          "sendToDoorPrice": 0,
          "sendToDoorMinFee": 0
        }
      ]
    }
  }
}
```

---
### 75. [修改预下单群组详情](#75-修改预下单群组详情modifypreordergroup)
- `modifyPreOrderGroup`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderGroupId | ID | 群组Id |
| shopId | ID | 分店Id（无为null） |
| agentId | ID | 收货点Id（无为null） |
| startPoint | String | 起点 |
| groupName | String | 群组名 |

```js
{
  "success": true
}
```

---
### 76. [获取预下单群组中的预下单列表](#76-获取预下单群组中的预下单列表getpreorderlistingroup)
- `getPreOrderListInGroup`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderGroupId | ID | 货单群组Id |

```js
{
  "success": true,
  "context": {
    "orderList": [
      {
        "startPoint": "贵州省贵阳市华通物流超市",
        "receiverName": "111",
        "endPoint": "内蒙古自治区通辽市库伦旗额勒顺镇",
        "proxyCharge": 1,
        "totalDesignatedFee": 1,
        "isReachPay": false,
        "isInsuance": false,
        "isSendToDoor": false,
        "size": 1,
        "weight": 1,
        "totalNumbers": 1,
        "shop": {
          "name": "华通物流超市",
          "id": "5995496cd1df03410425a904"
        },
        "receiver": {
          "phone": "13333333333",
          "name": "111",
          "id": "599f8c4a206d103b2a3de443"
        },
        "id": "599f8c4a206d103b2a3de444"
      }
    ]
  }
}
```

---

## 分店api

---
### 77. [收货部获取需要上传照片的订单](#77-收货部获取需要上传照片的订单getlastestorderbyreceivepartment)
- `getLastestOrderByReceivePartment`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "shopId": "5995496cd1df03410425a904",
    "receiverName": "18885192481",
    "endPoint": "北京市东城区东华门街道",
    "endPointLastCode": 110101001,
    "receivePartmentId": "59954970d1df03410425a917",
    "receiveMemberId": "59954970d1df03410425a916",
    "createTime": "2017-08-22 16:09:42",
    "stateList": [
      {
        "state": 0,
        "count": 5,
        "_id": "599be6c68e337d17123e9c1f"
      }
    ],
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 0,
    "proxyChargeProfit": 0,
    "proxyCharge": 0,
    "designatedFee": 0,
    "totalDesignatedFee": 1200,
    "realFee": 0,
    "branchProfit": 0,
    "masterProfit": 0,
    "fee": 0,
    "payTool": 0,
    "payMode": 1,
    "isReachPay": true,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": true,
    "isSendToDoor": true,
    "size": 1,
    "weight": 10,
    "totalNumbers": 5,
    "roadmapRankIndex": -1,
    "isSenderRepresentShipper": false,
    "receiver": {
      "phone": "18885192481",
      "id": "59954981d1df03410425a952"
    },
    "sender": {
      "phone": "18885192480",
      "id": "59954981d1df03410425a951"
    },
    "id": "599be6c68e337d17123e9c1e",
    "state": 0
  }
}
```

---
### 78. [收货部为货单上传照片](#78-收货部为货单上传照片setphotofororiginorder)
- `setPhotoForOriginOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |
| photo | String | 图片地址 |

```js
{
  "msg": "没有该订单"
}
```

---
### 79. [收货部为货单下中转单](#79-收货部为货单下中转单placetransferorder)
- `placeTransferOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |
| totalNumbers | Number | 总件数 |
| weight | Number | 重量 |
| size | Number | 方量 |

```js
{
  "success": true,
  "context": {
    "shopId": "5995496fd1df03410425a913",
    "senderId": "59954972d1df03410425a928",
    "receiverId": "59954981d1df03410425a952",
    "receiverName": "18885192481",
    "endPoint": "北京市东城区景山街道",
    "endPointLastCode": 110101002,
    "receivePartmentId": "59954970d1df03410425a919",
    "receiveMemberId": "59954970d1df03410425a918",
    "modifyTime": "2017-09-02 09:20:10",
    "createTime": "2017-09-02 09:20:10",
    "stateList": [
      {
        "state": 0,
        "count": 5,
        "_id": "59aa074ad920502bddbbf7c1"
      }
    ],
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 0,
    "proxyChargeProfit": 0,
    "proxyCharge": 0,
    "designatedFee": 0,
    "totalDesignatedFee": 1200,
    "realFee": 0,
    "branchProfit": 0,
    "masterProfit": 0,
    "fee": 0,
    "payTool": 0,
    "payMode": 0,
    "isReachPay": false,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": false,
    "isSendToDoor": false,
    "size": 1,
    "weight": 50,
    "totalNumbers": 5,
    "roadmapRankIndex": -1,
    "isSenderRepresentShipper": true,
    "isTransferOrder": true,
    "id": "59aa074ad920502bddbbf7c0"
  }
}
```

---
### 80. [收货部获取货单详情](#80-收货部获取货单详情getorderdetailbyreceivepartment)
- `getOrderDetailByReceivePartment`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |

```js
{
  "success": true,
  "context": {
    "shopId": "5995496cd1df03410425a904",
    "receiverName": "18885192481",
    "endPoint": "北京市东城区景山街道",
    "endPointLastCode": 110101002,
    "receivePartmentId": "59954970d1df03410425a917",
    "receiveMemberId": "59954970d1df03410425a916",
    "modifyTime": "2017-08-31 11:29:54",
    "createTime": "2017-08-31 11:29:54",
    "stateList": [
      {
        "state": 5,
        "count": 5,
        "_id": "59a782b298e48164554806ba"
      }
    ],
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 0,
    "proxyChargeProfit": 0,
    "proxyCharge": 0,
    "designatedFee": 0,
    "totalDesignatedFee": 1200,
    "realFee": 0,
    "branchProfit": 0,
    "masterProfit": 0,
    "fee": 0,
    "payTool": 0,
    "payMode": 0,
    "isReachPay": false,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": false,
    "isSendToDoor": false,
    "size": 1,
    "weight": 10,
    "totalNumbers": 5,
    "roadmapRankIndex": -1,
    "isSenderRepresentShipper": false,
    "receiver": {
      "phone": "18885192481",
      "id": "59954981d1df03410425a952"
    },
    "sender": {
      "phone": "18885192480",
      "id": "59954981d1df03410425a951"
    },
    "isTransferOrder": false,
    "id": "59a782b298e48164554806b7"
  }
}
```

---
### 81. [收货部获取订单](#81-收货部获取订单getordersbyreceivepartment)
- `getOrdersByReceivePartment`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| type | String | 类型 (tostore,stored)|
| keyword | String | 关键字|
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "topay": {
      "count": 0,
      "list": []
    },
    "toprint": {
      "count": 0,
      "list": []
    },
    "tostore": {
      "count": 2,
      "list": [
        {
          "endPoint": "北京市东城区景山街道",
          "stateList": [
            {
              "state": 5,
              "count": 5,
              "_id": "599e7fb2ead9861cc2a8e262"
            }
          ],
          "needPayInsuanceFee": 0,
          "needPayTransportFee": 48,
          "size": 1,
          "weight": 10,
          "totalNumbers": 5,
          "receiveMember": {
            "phone": "18185192480",
            "name": "收货部负责人",
            "id": "59954970d1df03410425a916"
          },
          "shipper": {
            "name": "广顺达物流公司",
            "id": "59954973d1df03410425a946"
          },
          "receiver": {
            "phone": "18885192481",
            "id": "59954981d1df03410425a952"
          },
          "sender": {
            "phone": "18885192480",
            "id": "59954981d1df03410425a951"
          },
          "id": "599e7fb2ead9861cc2a8e259"
        },
        {
          "endPoint": "北京市东城区景山街道",
          "stateList": [
            {
              "state": 5,
              "count": 5,
              "_id": "599e7fb1ead9861cc2a8e258"
            }
          ],
          "needPayInsuanceFee": 0,
          "needPayTransportFee": 48,
          "size": 1,
          "weight": 10,
          "totalNumbers": 5,
          "receiveMember": {
            "phone": "18185192480",
            "name": "收货部负责人",
            "id": "59954970d1df03410425a916"
          },
          "shipper": {
            "name": "广顺达物流公司",
            "id": "59954973d1df03410425a946"
          },
          "receiver": {
            "phone": "18885192481",
            "id": "59954981d1df03410425a952"
          },
          "sender": {
            "phone": "18885192480",
            "id": "59954981d1df03410425a951"
          },
          "id": "599e7fb1ead9861cc2a8e24f"
        }
      ]
    },
    "stored": {
      "count": 15,
      "list": [
        {
          "endPoint": "北京市东城区景山街道",
          "stateList": [
            {
              "state": 6,
              "count": 5,
              "_id": "599fea3c0db48253c419d84d"
            }
          ],
          "needPayInsuanceFee": 0,
          "needPayTransportFee": 48,
          "size": 1,
          "weight": 10,
          "totalNumbers": 5,
          "receiveMember": {
            "phone": "18185192480",
            "name": "收货部负责人",
            "id": "59954970d1df03410425a916"
          },
          "shipper": {
            "name": "广顺达物流公司",
            "id": "59954973d1df03410425a946"
          },
          "receiver": {
            "phone": "18885192481",
            "id": "59954981d1df03410425a952"
          },
          "sender": {
            "phone": "18885192480",
            "id": "59954981d1df03410425a951"
          },
          "id": "599e7fb0ead9861cc2a8e245"
        },
        {
          "endPoint": "北京市东城区景山街道",
          "stateList": [
            {
              "state": 7,
              "count": 5,
              "_id": "599fe94a0db48253c419d83f"
            }
          ],
          "needPayInsuanceFee": 0,
          "needPayTransportFee": 48,
          "size": 1,
          "weight": 10,
          "totalNumbers": 5,
          "receiveMember": {
            "phone": "18185192480",
            "name": "收货部负责人",
            "id": "59954970d1df03410425a916"
          },
          "shipper": {
            "name": "广顺达物流公司",
            "id": "59954973d1df03410425a946"
          },
          "receiver": {
            "phone": "18885192481",
            "id": "59954981d1df03410425a952"
          },
          "sender": {
            "phone": "18885192480",
            "id": "59954981d1df03410425a951"
          },
          "id": "599e7fafead9861cc2a8e23b"
        },
        {
          "endPoint": "北京市东城区景山街道",
          "stateList": [
            {
              "state": 7,
              "count": 5,
              "_id": "599fe5e50db48253c419d835"
            }
          ],
          "needPayInsuanceFee": 0,
          "needPayTransportFee": 48,
          "size": 1,
          "weight": 10,
          "totalNumbers": 5,
          "receiveMember": {
            "phone": "18185192480",
            "name": "收货部负责人",
            "id": "59954970d1df03410425a916"
          },
          "shipper": {
            "name": "广顺达物流公司",
            "id": "59954973d1df03410425a946"
          },
          "receiver": {
            "phone": "18885192481",
            "id": "59954981d1df03410425a952"
          },
          "sender": {
            "phone": "18885192480",
            "id": "59954981d1df03410425a951"
          },
          "id": "599e7faeead9861cc2a8e231"
        }
      ]
    }
  }
}

```
---
### 82. [库管部获取订单](#82-库管部获取订单getordersbywarehousepartment)
- `getOrdersByWareHousePartment`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| type | String | 类型 (stored,loaded)|
| keyword | String | 关键字|
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "tostore": {
      "count": 0,
      "list": []
    },
    "stored": {
      "count": 19,
      "list": [
        {
          "endPoint": "北京市东城区景山街道",
          "modifyTime": "2017-08-30 19:07:30",
          "createTime": "2017-08-30 19:07:30",
          "stateList": [
            {
              "state": 6,
              "count": 5,
              "_id": "59a69c8c03819c74dd8d488a"
            }
          ],
          "needPayInsuanceFee": 0,
          "needPayTransportFee": 0,
          "payMode": 0,
          "size": 1,
          "weight": 10,
          "totalNumbers": 5,
          "receiveMember": {
            "phone": "18185192480",
            "name": "收货部负责人",
            "id": "59954970d1df03410425a916"
          },
          "shop": {
            "name": "华通物流超市",
            "address": "贵州省贵阳市南明区花果园街道花果园L2区",
            "id": "5995496cd1df03410425a904"
          },
          "receiver": {
            "phone": "18885192481",
            "id": "59954981d1df03410425a952"
          },
          "sender": {
            "phone": "18885192480",
            "id": "59954981d1df03410425a951"
          },
          "id": "59a69c7203819c74dd8d4885"
        },
        {
          "endPoint": "北京市东城区景山街道",
          "modifyTime": "2017-08-31 08:58:21",
          "createTime": "2017-08-28 19:57:48",
          "stateList": [
            {
              "state": 6,
              "count": 5,
              "_id": "59a4053ca655283c6ff39d04"
            }
          ],
          "needPayInsuanceFee": 0,
          "needPayTransportFee": 0,
          "payMode": 0,
          "size": 1,
          "weight": 10,
          "totalNumbers": 5,
          "receiveMember": {
            "phone": "18185192480",
            "name": "收货部负责人",
            "id": "59954970d1df03410425a916"
          },
          "shop": {
            "name": "华通物流超市",
            "address": "贵州省贵阳市南明区花果园街道花果园L2区",
            "id": "5995496cd1df03410425a904"
          },
          "receiver": {
            "phone": "18885192481",
            "id": "59954981d1df03410425a952"
          },
          "sender": {
            "phone": "18885192480",
            "id": "59954981d1df03410425a951"
          },
          "id": "59a4053ca655283c6ff39cff"
        },
        {
          "endPoint": "北京市东城区景山街道",
          "modifyTime": "2017-08-31 08:58:21",
          "createTime": "2017-08-24 15:26:42",
          "stateList": [
            {
              "state": 6,
              "count": 5,
              "_id": "59a62763477dfaa641c83e89"
            }
          ],
          "needPayInsuanceFee": 0,
          "needPayTransportFee": 48,
          "payMode": 2,
          "size": 1,
          "weight": 10,
          "totalNumbers": 5,
          "receiveMember": {
            "phone": "18185192480",
            "name": "收货部负责人",
            "id": "59954970d1df03410425a916"
          },
          "shipper": {
            "name": "广顺达物流公司",
            "id": "59954973d1df03410425a946"
          },
          "shop": {
            "name": "华通物流超市",
            "address": "贵州省贵阳市南明区花果园街道花果园L2区",
            "id": "5995496cd1df03410425a904"
          },
          "receiver": {
            "phone": "18885192481",
            "id": "59954981d1df03410425a952"
          },
          "sender": {
            "phone": "18885192480",
            "id": "59954981d1df03410425a951"
          },
          "id": "599e7fb2ead9861cc2a8e259"
        }
      ]
    }
  }
}
```
---
### 83. [库管部获取正在装车的订单列表](#83-库管部获取正在装车的订单列表getloadingorderlist)
- `getLoadingOrderList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "shipperId": "59954973d1df03410425a946",
    "shopId": "5995496cd1df03410425a904",
    "plateNo": "贵A-778899",
    "drivingLicense": "A71236717",
    "truckType": "4.5米 高栏",
    "driverId": "599575cc277608472d636e97",
    "carryPartmentId": "599641b27338534997932c45",
    "scannerId": "59954972d1df03410425a928",
    "createTime": "2017-08-31 08:44:45",
    "unloadAllOrderList": [
      {
        "endPoint": "北京市东城区景山街道",
        "totalNumbers": 5,
        "receiver": {
          "phone": "18885192481"
        },
        "sender": {
          "phone": "18885192480"
        },
        "id": "59a69c7203819c74dd8d4885",
        "unloadNumber": 4
      }
    ],
    "totalSize": 1.2,
    "totalWeight": 12,
    "totalNumbers": 6,
    "orderCount": 2,
    "orderList": [
      {
        "endPoint": "北京市东城区景山街道",
        "createTime": "2017-08-31 09:44:41",
        "size": 1,
        "weight": 10,
        "totalNumbers": 5,
        "receiver": {
          "phone": "18885192481",
          "id": "59954981d1df03410425a952"
        },
        "sender": {
          "phone": "18885192480",
          "id": "59954981d1df03410425a951"
        },
        "id": "59a76a0901d61855a2875c12"
      },
      {
        "endPoint": "北京市东城区景山街道",
        "createTime": "2017-08-30 19:07:30",
        "size": 1,
        "weight": 10,
        "totalNumbers": 5,
        "receiver": {
          "phone": "18885192481",
          "id": "59954981d1df03410425a952"
        },
        "sender": {
          "phone": "18885192480",
          "id": "59954981d1df03410425a951"
        },
        "id": "59a69c7203819c74dd8d4885"
      }
    ],
    "state": 4,
    "locationList": [],
    "insuanceMount": 1000000,
    "carryPrice": 100,
    "id": "59a75bfd4b0e7a4e17cde5da"
  }
}
```

---
### 84. [库管部扫描入库](#84-库管部扫描入库placestorage)
- `placeStorage`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |
| count | Number | 数量 |

```js
{
  "success": true,
  "order": {
    "shopId": "5995496cd1df03410425a904",
    "senderId": "59954981d1df03410425a951",
    "receiverId": "59954981d1df03410425a952",
    "receiverName": "18885192481",
    "endPoint": "北京市东城区景山街道",
    "endPointLastCode": 110101002,
    "receivePartmentId": "59954970d1df03410425a917",
    "receiveMemberId": "59954970d1df03410425a916",
    "roadmapId": "599d22fec71183033861fd74",
    "shipperChairManId": "59954972d1df03410425a928",
    "shipperId": "59954973d1df03410425a946",
    "storeScannerId": "59954970d1df03410425a91a",
    "createTime": "2017-08-24 15:26:41",
    "stateList": [
      {
        "state": 6,
        "count": 1,
        "_id": "59a405d0a655283c6ff39d05"
      },
      {
        "state": 5,
        "count": 4,
        "_id": "599e7fb1ead9861cc2a8e258"
      }
    ],
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 48,
    "proxyChargeProfit": 0,
    "proxyCharge": 0,
    "designatedFee": 0,
    "totalDesignatedFee": 1200,
    "realFee": 48,
    "branchProfit": 4,
    "masterProfit": 4,
    "fee": 40,
    "payTool": 0,
    "payMode": 2,
    "isReachPay": false,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": false,
    "isSendToDoor": false,
    "size": 1,
    "weight": 10,
    "totalNumbers": 5,
    "roadmapRankIndex": 0,
    "isSenderRepresentShipper": false,
    "id": "599e7fb1ead9861cc2a8e24f"
  }
}
```

### 85. [保安部检查卡车状态](#85-保安部检查卡车状态checktruckpass)
- `checkTruckPass`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| driverId | ID | 司机Id |

```js
{
  "success": true,
  "context": {
    "isEnter": true,
    "driver": {
      "phone": "18985192480",
      "email": "42550564@qq.com",
      "name": "方运江",
      "head": "http://192.168.1.111:3000/api/image?id=599577a97aad4147a4f845db",
      "birthday": "1982-02-25",
      "address": "贵州省贵阳市",
      "license": "599577a97aad4147a4f845db",
      "licenseNo": "123123123",
      "registerTime": "2017-08-17 18:54:04",
      "sex": 0,
      "age": 35,
      "id": "599575cc277608472d636e97"
    }
  }
}
```

### 86. [分店获取成员列表](#86-分店获取成员列表getmemberlist)
- `getMemberList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| keyword | String | 关键字|
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "count": 1,
    "memberList": [
      {
        "phone": "18085192480",
        "name": "方运江",
        "post": "董事长",
        "authority": [
          0,
          1,
          2,
          3,
          4,
          5,
          6,
          7,
          8,
          10000,
          10001,
          10002,
          10003,
          10004,
          10005,
          10006,
          10007
        ],
        "id": "5995496ad1df03410425a8f2"
      }
    ]
  }
}
```

### 88. [分店创建成员](#88-分店创建成员modifymember)
- `createMember`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| name | String | 成员名|
| phone | String | 成员手机|
| authority | Array | 成员权限 |

```js
{
  "success": true,
  "context": {
    "name": "王小二",
    "phone": "13312341234",
    "authority": [
      10000,
      10001
    ],
    "id": "59a8f7792bdb4d2ef77b63e5"
  }
}
```

### 88. [分店创建成员](#88-分店创建成员modifymember)
- `modifyMember`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| memberId | ID | 成员Id |
| name | String | 成员名|
| phone | String | 成员手机|
| authority | Array | 成员权限 |

```js
{
  "success": true,
  "context": {
    "name": "王小三",
    "phone": "13312341234",
    "authority": [
      10000,
      10002
    ],
    "id": "59a8f7792bdb4d2ef77b63e5"
  }
}
```

### 89. [分店获取部门信息](#89-分店获取部门信息getownpartmentinfo)
- `getOwnPartmentInfo`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "shopId": "5995496cd1df03410425a904",
    "name": "库管部",
    "type": 1,
    "descript": "看家我最在行",
    "phoneList": "0851-98989000;0851-98989001",
    "isBusy": false,
    "createTime": "2017-08-17 15:44:48",
    "enable": false,
    "memberCount": 1,
    "chargeMan": {
      "phone": "18285192480",
      "name": "库管部负责人",
      "id": "59954970d1df03410425a91a"
    },
    "id": "59954970d1df03410425a91b"
  }
}
```

---

### 90. [获取统计信息](#90-获取统计信息getstatistics)
- `getStatistics`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |

```js
{
  "success": true,
  "context": {
    "branchProfit" : [0, 0, 0, 0, 0, 0, 0],
  "count": [1, 0, 0, 0, 0, 0, 0],
  "insuanceFee": [0, 0, 0, 0, 0, 0, 0],
  "insuanceMount": [0, 0, 0, 0, 0, 0, 0],
  "isInsuance": [0, 0, 0, 0, 0, 0, 0],
  "isReachPay": [0, 0, 0, 0, 0, 0, 0],
  "masterProfit": [0, 0, 0, 0, 0, 0, 0],
  "proxyCharge": [0, 0, 0, 0, 0, 0, 0],
  "realFee": [1, 0, 0, 0, 0, 0, 0],
  "size": [1, 0, 0, 0, 0, 0, 0],
  "totalDesignatedFee": [0, 0, 0, 0, 0, 0, 0],
  "weight": [1, 0, 0, 0, 0, 0, 0],
  }
}
```

---
## 收货点

---

### 91. [收货点通过手机号获取预下单列表](#91-收货点通过手机号获取预下单列表getpreorderlist)
- `getPreOrderList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| senderPhone | String | 发货人手机号|
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "count": 2,
    "orderList": [
      {
        "startPoint": "贵州省贵阳市",
        "senderId": "59e416b1d9196a1dc3d5d961",
        "senderPhone": "13984387205",
        "senderName": "1024",
        "receiverId": "59e419d4b6ee191fe5d4fbc6",
        "receiverPhone": "13000000000",
        "receiverName": "Test1",
        "name": "种子",
        "endPoint": "天津",
        "sendDoorEndPoint": "",
        "isSendToDoor": false,
        "createTime": "2017-10-20 10:34:52",
        "stateList": [
          {
            "state": -1,
            "count": 1,
            "_id": "59e960cc482e7a0be7dc7581"
          }
        ],
        "needPayInsuanceFee": 0,
        "needPayTransportFee": 0,
        "proxyChargeProfit": 0,
        "proxyCharge": 0,
        "designatedFee": 0,
        "totalDesignatedFee": 0,
        "realFee": 0,
        "branchProfit": 0,
        "masterProfit": 0,
        "profit": 0,
        "fee": 0,
        "payTool": 0,
        "payMode": 0,
        "isReachPay": false,
        "insuanceFee": 0,
        "insuanceMount": 0,
        "isInsuance": false,
        "size": 1,
        "weight": 1,
        "totalNumbers": 1,
        "needBondAmount": 0,
        "roadmapRankIndex": -1,
        "sendDoorEndPointLastCode": null,
        "isSendDoor": false,
        "endPointLastCode": 12,
        "startPointLastCode": 0,
        "isSenderRepresentShipper": true,
        "isTransferOrder": false,
        "id": "59e960cc482e7a0be7dc7580"
      },
      {
        "shopId": "59df08df009deb7e9298e176",
        "startPoint": "贵州省",
        "senderId": "59e416b1d9196a1dc3d5d961",
        "senderPhone": "13984387205",
        "senderName": "范泽华",
        "receiverId": "59e419d4b6ee191fe5d4fbc6",
        "receiverPhone": "13000000000",
        "receiverName": "伊藤诚",
        "name": "种子",
        "endPoint": "北京市东城区东华门街道",
        "isSendToDoor": false,
        "groupId": "59e5a1889594771ba19eeba7",
        "createTime": "2017-10-17 14:21:39",
        "stateList": [
          {
            "state": -1,
            "count": 1,
            "_id": "59e5a1739594771ba19eeba5"
          }
        ],
        "needPayInsuanceFee": 0,
        "needPayTransportFee": 0,
        "proxyChargeProfit": 0,
        "proxyCharge": 0,
        "designatedFee": 0,
        "totalDesignatedFee": 0,
        "realFee": 0,
        "branchProfit": 0,
        "masterProfit": 0,
        "profit": 0,
        "fee": 0,
        "payTool": 0,
        "payMode": 0,
        "isReachPay": false,
        "insuanceFee": 0,
        "insuanceMount": 0,
        "isInsuance": false,
        "size": 1,
        "weight": 1,
        "totalNumbers": 1,
        "needBondAmount": 0,
        "roadmapRankIndex": -1,
        "sendDoorEndPointLastCode": 0,
        "isSendDoor": false,
        "endPointLastCode": 110101001,
        "startPointLastCode": 0,
        "isSenderRepresentShipper": true,
        "isTransferOrder": false,
        "id": "59e5a1739594771ba19eeba4"
      }
    ]
  }
}
```

---

### 92. [收货点下单](#92-收货点下单placeorder)
- `placeOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| senderPhone | String | 发货人手机号|
| senderName | String | 发货人姓名|
| receiverPhone | String | 收货人手机号|
| receiverName | String | 收货人姓名|
| endPoint | String | 终点|
| endPointLastCode | Number | 编码|
| sendDoorEndPoint | String | 送货上门终点|
| sendDoorEndPointLastCode | Number | 送货上门终点编码|
| name | String | 货物名|
| totalNumbers | Number | 总件数 |
| weight | Number | 重量 |
| size | Number | 方量 |
| isSendDoor | Bool | 是否送货上门 |
| isReachPay | Bool | 是否到付 |
| proxyCharge | Number | 代收货款金额 |
| insuanceMount | Number | 保险 |
| totalDesignatedFee | Number | 指定收款金额 |

```js
{
  "success": true,
  "context": {
    "agentId": "59f698f70931cf0b4e8b5266",
    "senderId": "59df087f009deb7e9298e16f",
    "senderName": "fang1",
    "senderPhone": "18083602719",
    "receiverId": "59df0a26009deb7e9298e18f",
    "receiverPhone": "18684165865",
    "receiverName": "方运江",
    "name": "腊肉",
    "endPoint": "北京",
    "sendDoorEndPoint": "东城区景山街道",
    "placeOrderTime": "2017-10-30 16:35:02",
    "createTime": "2017-10-30 16:35:02",
    "stateList": [
      {
        "state": 0,
        "count": 5,
        "_id": "59f6e4361dc67612a47e3162"
      }
    ],
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 0,
    "proxyChargeProfit": 0,
    "proxyCharge": 0,
    "designatedFee": 0,
    "totalDesignatedFee": 0,
    "realFee": 0,
    "branchProfit": 0,
    "masterProfit": 0,
    "profit": 0,
    "fee": 0,
    "payTool": 0,
    "payMode": 0,
    "isReachPay": false,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": false,
    "size": 1,
    "weight": 1,
    "totalNumbers": 5,
    "needBondAmount": 0,
    "roadmapRankIndex": -1,
    "sendDoorEndPointLastCode": 110101002,
    "isSendDoor": true,
    "endPointLastCode": 11,
    "startPointLastCode": 0,
    "isSenderRepresentShipper": false,
    "isTransferOrder": false,
    "id": "59f6e4361dc67612a47e3161",
    "state": 0
  }
}
```

---

### 93. [收货点修改货单](#93-收货点修改货单modifyorder)
- `modifyOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |
| senderPhone | String | 发货人手机号|
| senderName | String | 发货人姓名|
| receiverPhone | String | 收货人手机号|
| receiverName | String | 收货人姓名|
| endPoint | String | 终点|
| endPointLastCode | Number | 编码|
| sendDoorEndPoint | String | 送货上门终点|
| sendDoorEndPointLastCode | Number | 送货上门终点编码|
| name | String | 货物名|
| totalNumbers | Number | 总件数 |
| weight | Number | 重量 |
| size | Number | 方量 |
| isSendDoor | Bool | 是否送货上门 |
| isReachPay | Bool | 是否到付 |
| proxyCharge | Number | 代收货款金额 |
| insuanceMount | Number | 保险 |
| totalDesignatedFee | Number | 指定收款金额 |

```js
{
  "success": true,
  "context": {
    "agentId": "59f698f70931cf0b4e8b5266",
    "senderId": "59df087f009deb7e9298e16f",
    "senderName": "fang1",
    "senderPhone": "18083602719",
    "receiverId": "59df0a26009deb7e9298e18f",
    "receiverPhone": "18684165865",
    "receiverName": "方运江",
    "name": "腊肉",
    "endPoint": "北京",
    "sendDoorEndPoint": "东城区景山街道",
    "placeOrderTime": "2017-10-30 16:35:02",
    "createTime": "2017-10-30 16:35:02",
    "stateList": [
      {
        "state": 0,
        "count": 5,
        "_id": "59f6e4361dc67612a47e3162"
      }
    ],
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 0,
    "proxyChargeProfit": 0,
    "proxyCharge": 0,
    "designatedFee": 0,
    "totalDesignatedFee": 0,
    "realFee": 0,
    "branchProfit": 0,
    "masterProfit": 0,
    "profit": 0,
    "fee": 0,
    "payTool": 0,
    "payMode": 0,
    "isReachPay": false,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": false,
    "size": 1,
    "weight": 1,
    "totalNumbers": 5,
    "needBondAmount": 0,
    "roadmapRankIndex": -1,
    "sendDoorEndPointLastCode": 110101002,
    "isSendDoor": true,
    "endPointLastCode": 11,
    "startPointLastCode": 0,
    "isSenderRepresentShipper": false,
    "isTransferOrder": false,
    "id": "59f6e4361dc67612a47e3161",
    "state": 0
  }
}
```

---

### 94. [收货点打印二维码](#94-收货点打印二维码printbarcode)
- `printBarCode`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |

```js
{
  "success": true,
  "context": {
    "startPoint": "贵州省贵阳市",
    "senderId": "59df087f009deb7e9298e16f",
    "senderPhone": "18083602719",
    "senderName": "第一物流老大",
    "receiverId": "59df08e5009deb7e9298e179",
    "receiverPhone": "17608566786",
    "receiverName": "收货人2",
    "name": "0000",
    "endPoint": "北京市",
    "isSendToDoor": false,
    "shopId": "59df08df009deb7e9298e176",
    "receivePartmentId": "59df0968009deb7e9298e17e",
    "receiveMemberId": "59df1048009deb7e9298e1eb",
    "placeOrderTime": "2017-10-12 16:06:36",
    "photo": "http://localhost:3000/api/image?id=59df22a4009deb7e9298e2de",
    "modifyTime": "2017-11-02 11:51:16",
    "deductError": false,
    "roadmapId": "59df0d32009deb7e9298e1ca",
    "shipperChairManId": "59df0b1f009deb7e9298e19a",
    "shipperId": "59df0c27009deb7e9298e1a7",
    "storeScannerId": "59df09bf009deb7e9298e185",
    "loadScannerId": "59df0b1f009deb7e9298e19a",
    "truckId": "59df23f4009deb7e9298e2f7",
    "confirmHandOverId": "59df0b1f009deb7e9298e19a",
    "createTime": "2017-10-12 15:50:17",
    "stateList": [
      {
        "state": 3,
        "count": 2,
        "_id": "59fa96346137254a82861339"
      }
    ],
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 1300,
    "proxyChargeProfit": 10,
    "proxyCharge": 1000,
    "designatedFee": 0,
    "totalDesignatedFee": 20,
    "realFee": 1320,
    "branchProfit": 30.799999999999997,
    "masterProfit": 13.200000000000003,
    "profit": 44,
    "fee": 220,
    "payTool": 0,
    "payMode": 2,
    "isReachPay": true,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": false,
    "size": 1,
    "weight": 2,
    "totalNumbers": 2,
    "needBondAmount": 20000,
    "roadmapRankIndex": 1,
    "sendDoorEndPointLastCode": 0,
    "isSendDoor": false,
    "endPointLastCode": 11,
    "isCityDistribute": false,
    "startPointLastCode": 0,
    "isSenderRepresentShipper": true,
    "isTransferOrder": false,
    "id": "59df1eb9009deb7e9298e28f",
    "state": 3
  }
}
```

---

### 95. [收货点打印二货单](#95-收货点打印二货单printorderlistbill)
- `printOrderListBill`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderIdList | Array | 货单Id数组 |

```js
{
  "success": true,
}
```

---

### 96. [收货点代支付货单](#96-收货点代支付货单confirmcachpayedfororderlist)
- `confirmCachPayedForOrderList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderIdList | Array | 货单Id数组 |

```js
{
  "success": true,
}
```

---

### 97. [收货点获取未完成货单](#97-收货点获取未完成货单getlastestorder)
- `getLastestOrder`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "senderId": "59fab1a1c651f44b1d891d28",
    "senderName": "fang",
    "senderPhone": "18085192480",
    "receiverId": "59fab1a1c651f44b1d891d28",
    "receiverPhone": "18085192480",
    "receiverName": "fang",
    "name": "腊肉",
    "endPoint": "贵州遵义市湄潭县",
    "sendDoorEndPoint": "湄江镇",
    "receiveAgentMemberId": "59df0a26009deb7e9298e18f",
    "placeOrderTime": "2017-11-02 13:48:17",
    "createTime": "2017-11-02 13:48:17",
    "stateList": [
      {
        "state": 0,
        "count": 1,
        "_id": "59fab1a1c651f44b1d891d2a"
      }
    ],
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 0,
    "proxyChargeProfit": 0,
    "proxyCharge": 0,
    "designatedFee": 0,
    "totalDesignatedFee": 0,
    "realFee": 0,
    "branchProfit": 0,
    "masterProfit": 0,
    "profit": 0,
    "fee": 0,
    "payTool": 0,
    "payMode": 0,
    "isReachPay": false,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": false,
    "size": 1,
    "weight": 1,
    "totalNumbers": 1,
    "needBondAmount": 0,
    "roadmapRankIndex": -1,
    "sendDoorEndPointLastCode": 520328100,
    "isSendDoor": true,
    "endPointLastCode": 520328,
    "isCityDistribute": false,
    "startPointLastCode": 0,
    "isSenderRepresentShipper": false,
    "isTransferOrder": false,
    "id": "59fab1a1c651f44b1d891d29",
    "agent": {
      "name": "18684165865收货点",
      "address": "贵州省贵阳市云岩区",
      "id": "59f698f70931cf0b4e8b5266"
    },
    "state": 0
  }
}
```

---
### 98. [收货点获取订单](#98-收货点获取订单getorders)
- `getOrders`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| type | String | 类型 (topay(待支付),toprintbill(待打印货单),tosend(待发货),onway(运输中),success(已完成))|
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "topay": {
      "count": 1,
      "list": [
        {
          "senderName": "fang",
          "senderPhone": "18085192480",
          "receiverPhone": "18085192480",
          "receiverName": "fang",
          "name": "腊肉",
          "endPoint": "贵州遵义市湄潭县",
          "sendDoorEndPoint": "湄江镇",
          "placeOrderTime": "2017-11-02 13:48:17",
          "createTime": "2017-11-02 13:48:17",
          "stateList": [
            {
              "state": 0,
              "count": 1,
              "_id": "59fab1a1c651f44b1d891d2a"
            }
          ],
          "needPayInsuanceFee": 0,
          "needPayTransportFee": 0,
          "proxyCharge": 0,
          "totalDesignatedFee": 0,
          "payMode": 0,
          "size": 1,
          "weight": 1,
          "totalNumbers": 1,
          "isSendDoor": true,
          "isCityDistribute": false,
          "id": "59fab1a1c651f44b1d891d29"
        }
      ]
    },
    "toprintbill": {
      "count": 0,
      "list": []
    },
    "tosend": {
      "count": 0,
      "list": []
    },
    "onway": {
      "count": 0,
      "list": []
    },
    "success": {
      "count": 0,
      "list": []
    }
  }
}
```

###### stateList.state的说明：
```
3：待付款状态
2/4：待打印货单
8：待发货
9：运输中
11：已完成
```

---
### 99. [收货点获取订单详情](#99-收货点获取订单详情getorderdetail)
- `getOrderDetail`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |

```js
{
  "success": true,
  "context": {
    "senderId": "59df087f009deb7e9298e16f",
    "senderName": "fang1",
    "senderPhone": "18083602719",
    "receiverId": "59df0a26009deb7e9298e18f",
    "receiverPhone": "18684165865",
    "receiverName": "方运江",
    "name": "腊肉",
    "endPoint": "北京",
    "sendDoorEndPoint": "东城区景山街道",
    "placeOrderTime": "2017-10-30 16:35:02",
    "createTime": "2017-10-30 16:35:02",
    "stateList": [
      {
        "state": 0,
        "count": 5,
        "_id": "59f6e4361dc67612a47e3162"
      }
    ],
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 0,
    "proxyChargeProfit": 0,
    "proxyCharge": 0,
    "designatedFee": 0,
    "totalDesignatedFee": 0,
    "realFee": 0,
    "branchProfit": 0,
    "masterProfit": 0,
    "profit": 0,
    "fee": 0,
    "payTool": 0,
    "payMode": 0,
    "isReachPay": false,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": false,
    "size": 1,
    "weight": 1,
    "totalNumbers": 5,
    "needBondAmount": 0,
    "roadmapRankIndex": -1,
    "sendDoorEndPointLastCode": 110101002,
    "isSendDoor": true,
    "endPointLastCode": 11,
    "isCityDistribute": false,
    "startPointLastCode": 0,
    "isSenderRepresentShipper": false,
    "isTransferOrder": false,
    "id": "59f6e4361dc67612a47e3161",
    "agent": {
      "name": "18684165865收货点",
      "address": "贵州省贵阳市云岩区",
      "id": "59f698f70931cf0b4e8b5266"
    },
    "state": 0
  }
}
```

---
### 100. [收货点获取成员列表](#100-收货点获取成员列表getmemberlist)
- `getMemberList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "memberList": [
      {
        "phone": "18684165865",
        "registerTime": "2017-10-12 14:22:30",
        "shipperId": "59df0b86009deb7e9298e19c",
        "head": "http://localhost:3000/api/image?id=59df35441b989b12d6ac11d1",
        "agentId": "59f698f70931cf0b4e8b5266",
        "createTime": "2017-10-12T06:22:30.345Z",
        "isSetPaymentPassword": 1,
        "authority": [
          40000,
          40001,
          40002,
          10008,
          10005,
          10006,
          10007,
          10009
        ],
        "sex": 0,
        "id": "59df0a26009deb7e9298e18f"
      }
    ]
  }
}
```

---
### 101. [收货点修改成员权限](#101-收货点修改成员权限modifymemberauthority)
- `modifyMemberAuthority`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| memberId | ID | 成员Id |
| authority | Array | 权限列表Id |

```js
{
  "success": true,
  "context": {
    "phone": "15629002829",
    "registerTime": "2017-10-19 14:29:59",
    "createTime": "2017-10-19T06:29:59.222Z",
    "isSetPaymentPassword": 0,
    "authority": [
      40001,
      40002,
      40003
    ],
    "sex": 0,
    "id": "59e846675d12e51039dde2e0"
  }
}
```

---
### 102. [收货点通过手机号搜索成员](#102-收货点通过手机号搜索成员getmemberbyphone)
- `getMemberByPhone`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| memberPhone | String | 成员手机号 |

```js
{
  "success": true,
  "context": {
    "phone": "13312341234",
    "email": "42550564@qq.com",
    "registerTime": "2017-08-30 19:41:55",
    "createTime": "2017-08-30T11:41:55.543Z",
    "authority": [],
    "sex": 0,
    "id": "59a6a483523fa0838161fd27"
  }
}
```

---
### 103. [收货点获取关联店铺列表](#103-收货点获取关联店铺列表getrefershoplist)
- `getReferShopList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "branchShopList": [
      {
        "name": "15555555501",
        "phoneList": "15555555501",
        "address": "贵州省贵阳市云岩区黔灵山分店",
        "location": [
          106.70094,
          26.605311
        ],
        "addressRegion": "贵州贵阳市市辖区",
        "addressRegionLastCode": 520101,
        "profitRate": 0.2,
        "id": "59e426b1b6ee191fe5d4fbf8",
        "chairMan": {
          "phone": "15555555501",
          "name": "程",
          "id": "59e426b1b6ee191fe5d4fbf7"
        }
      },
      {
        "name": "北站分店",
        "phoneList": "16666666601",
        "address": "贵州省贵阳市乌当区贵阳北站",
        "location": [
          106.68309,
          26.625563
        ],
        "profitRate": 0.2,
        "id": "59e41c3cb6ee191fe5d4fbd4",
        "chairMan": {
          "phone": "16666666601",
          "name": "程",
          "id": "59e41c3cb6ee191fe5d4fbd3"
        }
      },
      {
        "name": "分店2",
        "phoneList": "0851-86190987;18185192480",
        "address": "湖北省十堰市竹溪县",
        "location": [
          109.657937,
          32.171422
        ],
        "addressRegion": "湖北十堰市竹溪县",
        "addressRegionLastCode": 420324,
        "profitRate": 0.7,
        "id": "59df4fdc975c7a2578e79674",
        "chairMan": {
          "phone": "15629000001",
          "name": "陈方东",
          "id": "59df4fdc975c7a2578e79673"
        }
      },
      {
        "name": "范家分店",
        "phoneList": "0851-88622803",
        "address": "贵州省贵阳市云岩区",
        "location": [
          106.709177,
          26.629907
        ],
        "profitRate": 0.7,
        "id": "59df08df009deb7e9298e176",
        "chairMan": {
          "phone": "13984387205",
          "name": "范泽华",
          "id": "59df08df009deb7e9298e175"
        }
      }
    ]
  }
}
```

---
### 104. [收货点设置关联店铺](#104-收货点设置关联店铺setrefershop)
- `setReferShop`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 店铺Id |

```js
{
  "success": true
}
```

---
### 105. [收货点添加区域的路线提成](#105-收货点添加区域的路线提成addregionprofit)
- `addRegionProfit`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| region | String | 方向，如：北京 |
| regionLastCode | number | 方向的最后一个code |
| profitRate | number | 提成比例 |
| profitCount | number | 提成价格 |
| type | number | 类型，0：所有，1：长途，2：短途|

```js
{
  "success": true,
  "context": {
    "shopId": null,
    "region": "河北",
    "createTime": "2017-11-02 10:55:43",
    "type": 0,
    "profitCount": 0,
    "profitRate": 0.3,
    "regionLastCode": 14,
    "id": "59fa892fa417a943de84d1a3",
    "shop": null
  }
}
```

---
### 106. [收货点获取区域的路线提成列表](#106-收货点获取区域的路线提成列表getregionprofitlist)
- `getRegionProfitList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| keyword | String | 关键字 |
| pageNo | Number | 页码 |
| pageSize | Number | 每一页大小 |

```js
{
  "success": true,
  "context": {
    "count": 2,
    "regionRateList": [
      {
        "shopId": null,
        "createTime": "2017-11-02 10:53:33",
        "type": 0,
        "profitCount": 0,
        "profitRate": 0.2,
        "regionLastCode": 0,
        "id": "59fa88ada417a943de84d1a1",
        "shop": null
      },
      {
        "shopId": null,
        "region": "河北",
        "createTime": "2017-11-02 10:55:43",
        "type": 0,
        "profitCount": 0,
        "profitRate": 0.3,
        "regionLastCode": 14,
        "id": "59fa892fa417a943de84d1a3",
        "shop": null
      }
    ]
  }
}
```

---
### 107. [收货点获取区域的路线提成详情](#107-收货点获取区域的路线提成详情getregionprofitdetail)
- `getRegionProfitDetail`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| regionId | ID | 路线提成Id |

```js
{
  "success": true,
  "context": {
    "agentId": "59f698f70931cf0b4e8b5266",
    "shopId": null,
    "createTime": "2017-11-02 10:53:33",
    "level": 1,
    "type": 0,
    "profitCount": 0,
    "profitRate": 0.2,
    "regionLastCode": 0,
    "id": "59fa88ada417a943de84d1a1",
    "shop": null
  }
}
```

---
### 108. [收货点修改区域的路线提成](#108-收货点修改区域的路线提成modifyregionprofit)
- `modifyRegionProfit`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| regionId | ID | 路线提成Id |
| region | String | 方向，如：北京 |
| regionLastCode | number | 方向的最后一个code |
| profitRate | number | 提成比例 |
| profitCount | number | 提成价格 |
| type | number | 类型，0：所有，1：长途，2：短途|

```js
{
  "success": true,
  "context": {
    "shopId": null,
    "region": "河北",
    "modifyTime": "2017-11-02 11:20:23",
    "createTime": "2017-11-02 10:53:33",
    "type": 0,
    "profitCount": 4,
    "profitRate": 0,
    "regionLastCode": 15,
    "id": "59fa88ada417a943de84d1a1",
    "shop": null
  }
}
```

---
### 109. [收货点删除区域的路线提成](#109-收货点删除区域的路线提成removeregionprofit)
- `removeRegionProfit`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| regionId | ID | 路线提成Id |

```js
{
  "success": true,
  "context": {
    "shopId": null,
    "region": "河北",
    "modifyTime": "2017-11-02 11:20:23",
    "createTime": "2017-11-02 10:53:33",
    "type": 0,
    "profitCount": 4,
    "profitRate": 0,
    "regionLastCode": 15,
    "id": "59fa88ada417a943de84d1a1",
    "shop": null
  }
}
```

---
### 110. [收货点批量设置区域的路线提成](#110-收货点批量设置区域的路线提成setregionprofitwithlist)
- `setRegionProfitWithList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| regionIdList | Array | 路线提成Id数组 |
| profitRate | number | 提成比例 |
| profitCount | number | 提成价格 |

```js
{
  "success": true,
}
```

---
## 同城配送

---

### 111. [获取分店设置了上门自提竞价的物流公司列表](#111-获取分店设置了上门自提竞价的物流公司列表getclientpickshipperlist)
- `getClientPickShipperList`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| shopId | ID | 分店Id |

```js
{
  "success": true,
  "context": {
    "shipperList": [
      {
        "clientPickEnable": true,
        "clientPickPrice": 10,
        "id": "5a04fa648fbe0257a3922ddb",
        "name": " 13500000000同城物流",
        "shipperType": 1
      }
    ]
  }
}
```

---
### 112. [短途物流公司扫描入库](#112-短途物流公司扫描入库placestorage)
- `placeStorage`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |
| count | Number | 数量 |

```js
{
  "success": true,
  "order": {
    "placeStoreTime": "2017-11-14 10:51:55",
    "shipperStoreScannerId": "5a04f98a8fbe0257a3922dd3",
    "preOrderId": "5a03c19ffd520c16113012cd",
    "shopId": "5a04f68b8fbe0257a3922db5",
    "senderId": "5a04f98a8fbe0257a3922dd3",
    "senderPhone": "18684165865",
    "senderName": "呵呵",
    "receiverId": "59faced36b15a74e0438faee",
    "receiverPhone": "13312341234",
    "receiverName": "后过敏",
    "name": "破破网",
    "endPoint": "云岩区中华中路街道",
    "sendDoorEndPoint": null,
    "receivePartmentId": "5a04f7018fbe0257a3922db9",
    "receiveMemberId": "5a04f7018fbe0257a3922db8",
    "placeOrderTime": "2017-11-10 10:09:47",
    "modifyTime": "2017-11-10 10:24:30",
    "photo": "http://192.168.1.189:80/api/image?id=5a050a7b1d6a5427bc7dcbeb",
    "deductError": false,
    "roadmapId": "5a04fd559d2eb14135c8a3a4",
    "shipperChairManId": "5a04f98a8fbe0257a3922dd3",
    "shipperId": "5a04fa648fbe0257a3922ddb",
    "warehouse": " 13500000000同城物流",
    "nextOrderId": "5a050eff1d6a5427bc7dcbf1",
    "createTime": "2017-11-10 10:09:47",
    "stateList": [
      {
        "state": 6,
        "count": 1,
        "_id": "5a0a5a4bc9b3cd6f647085d5"
      },
      {
        "state": 5,
        "count": 9,
        "_id": "5a050eff1d6a5427bc7dcbf3"
      }
    ],
    "additionalFee": 0,
    "needPayInsuanceFee": 0,
    "needPayTransportFee": 1,
    "proxyChargeProfit": 0,
    "proxyCharge": 0,
    "designatedFee": 0,
    "totalDesignatedFee": 0,
    "realFee": 1,
    "branchProfit": 0,
    "masterProfit": 0,
    "profit": 0,
    "fee": 1,
    "payTool": 0,
    "payMode": 0,
    "isReachPay": false,
    "insuanceFee": 0,
    "insuanceMount": 0,
    "isInsuance": false,
    "size": 0,
    "weight": 1,
    "totalNumbers": 10,
    "sendDoorMinFee": 0,
    "sendDoorPrice": 0,
    "minFee": 0.2,
    "price": 0.2,
    "needBondAmount": 10000,
    "roadmapRankIndex": 0,
    "isClientPick": false,
    "sendDoorEndPointLastCode": 0,
    "isSendDoor": false,
    "endPointLastCode": 520103001,
    "isCityDistribute": false,
    "startPointLastCode": 0,
    "isSenderRepresentShipper": true,
    "isTransferOrder": true,
    "id": "5a050a6b1d6a5427bc7dcbe5"
  }
}
```

---
### 113. [短途物流公司设置送货上门](#113-短途物流公司设置送货上门setclientpickordertransportfee)
- `setClientPickOrderTransportFee`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |
| isTransport | Boolean | 是否要送货上门 |
| endPoint | String | 送货上门地址 |
| transportFee | Number | 送货上门费用 |
| isScan | Boolean | 是否为扫描货单（还可以在货单列表修改送货上门状态） |

```js
{
  "success": true
}
```

---
### 114. [短途物流公司设置货车](#114-短途物流公司设置货车setcitytruck)
- `setCityTruck`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| plateNo | String | 车牌号 |

```js
{
  "success": true
}
```

---
### 115. [短途物流公司获取货车详情](#115-短途物流公司获取货车详情getcitytruckdetail)
- `getCityTruckDetail`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "shipperId": "5a04fa648fbe0257a3922ddb",
    "plateNo": "123456",
    "driverId": "5a04f98a8fbe0257a3922dd3",
    "createTime": "2017-11-14 11:06:52",
    "unloadAllOrderList": [],
    "totalSize": 0,
    "totalWeight": 0,
    "totalNumbers": 0,
    "orderCount": 0,
    "orderList": [],
    "locationList": [],
    "id": "5a0a5dccc9b3cd6f64708718"
  }
}
```

---
### 116. [短途物流公司扫描装车](#116-短途物流公司扫描装车scanloadorderforcitydistribute)
- `scanLoadOrderForCityDistribute`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |
| orderId | ID | 货单Id |
| count | Number | 数量 |

```js
{
  "success": true,
  "context": {
    "shipperId": "5a04fa648fbe0257a3922ddb",
    "plateNo": "123456",
    "driverId": "5a04f98a8fbe0257a3922dd3",
    "createTime": "2017-11-14 11:06:52",
    "unloadAllOrderList": [
      {
        "senderPhone": "18684165865",
        "senderName": "呵呵",
        "receiverPhone": "13312341234",
        "receiverName": "后过敏",
        "endPoint": "张三家门口1",
        "totalNumbers": 10,
        "id": "5a050a6b1d6a5427bc7dcbe5",
        "unloadNumber": 9
      }
    ],
    "totalSize": 0,
    "totalWeight": 0.1,
    "totalNumbers": 1,
    "orderCount": 1,
    "locationList": [],
    "id": "5a0a5dccc9b3cd6f64708718",
    "latestOrder": {
      "senderPhone": "18684165865",
      "senderName": "呵呵",
      "receiverPhone": "13312341234",
      "receiverName": "后过敏",
      "endPoint": "张三家门口1",
      "photo": "http://192.168.1.189:80/api/image?id=5a050a7b1d6a5427bc7dcbeb",
      "createTime": "2017-11-10 10:09:47",
      "size": 0,
      "weight": 1,
      "totalNumbers": 10,
      "id": "5a050a6b1d6a5427bc7dcbe5"
    }
  }
}
```

---
### 117. [短途物流公司获取货车的货单列表](#117-短途物流公司获取货车的货单列表getorderlistincitytruck)
- `getOrderListInCityTruck`
- 请求方式：`POST`

| 参数名称 | 参数类型  | 描述 |
| :- |:-:| :-:|
| userId | ID | 用户Id |

```js
{
  "success": true,
  "context": {
    "orderList": [
      {
        "senderPhone": "18684165865",
        "senderName": "呵呵",
        "receiverId": "59faced36b15a74e0438faee",
        "receiverPhone": "13312341234",
        "receiverName": "后过敏",
        "name": "破破网",
        "endPoint": "张三家门口1",
        "placeOrderTime": "2017-11-10 10:09:47",
        "photo": "http://192.168.1.189:80/api/image?id=5a050a7b1d6a5427bc7dcbeb",
        "roadmapId": "5a04fd559d2eb14135c8a3a4",
        "createTime": "2017-11-10 10:09:47",
        "stateList": [
          {
            "state": 8,
            "count": 1,
            "_id": "5a0a5f2cc9b3cd6f6470871a"
          },
          {
            "state": 6,
            "count": 9,
            "_id": "5a0a5bb3c9b3cd6f647085d7"
          }
        ],
        "needPayInsuanceFee": 0,
        "proxyCharge": 0,
        "totalDesignatedFee": 0,
        "realFee": 1,
        "payMode": 0,
        "isReachPay": false,
        "isInsuance": false,
        "size": 0,
        "weight": 1,
        "totalNumbers": 10,
        "isSendDoor": false,
        "id": "5a050a6b1d6a5427bc7dcbe5",
        "shop": {
          "name": "13300000000贵阳分店",
          "address": "贵州省贵阳市乌当区",
          "id": "5a04f68b8fbe0257a3922db5"
        }
      }
    ]
  }
}
```

---
## 协议文档

---

### 118. [用户协议](#118-用户协议user)
- `user`
- url: `protocals/user.html`

---

### 119. [获取软件许可协议](#119-获取软件许可协议software)
- `software`
- url: `protocals/software.html`

---

### 120. [关于](#120-关于about)
- `about`
- url: `protocals/about.html`
