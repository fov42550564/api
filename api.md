* PDShopServer/project/App/index.js

```js
import bodyParser from 'body-parser';
import express from 'express';
import multer from 'multer';
import mongoose from 'mongoose';
import path from 'path';
import xmlparser from 'express-xml-bodyparser';

import routers from './routers';

const storage = {
    _handleFile (req, file, cb) {
        const { originalname, stream } = file;
        const writestream = mongoose.gfs.createWriteStream({
            filename: originalname,
        }).on('close', grid => {
            cb(null, { grid });
        }).on('error', cb);

        stream.pipe(writestream);
    },
    _removeFile (req, file, cb) {
        if (file.grid) {
            mongoose.gfs.remove({ _id: file.grid._id }, cb);
        } else {
            cb(null);
        }
    },
};

const app = express();
// Multer
app.use(multer({
    storage,
    limits: {
        fileSize: 100000000,
    },
    onFileSizeLimit: function (file) {
        mongoose.gfs.remove({ _id: file.id });
    },
}).single('file'));

// Json
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json({ limit: 100000000 }));
app.use(xmlparser({ explicitArray: false, ignoreAttrs: true }));

// Static files
app.use(express.static(path.resolve('public')));
app.use(express.static(path.resolve('download')));

app.all('*', (req, res, next) => {
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Headers', 'Content-Type');
    next();
});

// Routers
app.use(routers);

export default app;

```

* PDShopServer/project/App/constants/account.js

```js
// 账户类型
export const AT_BASIC = 0; // 基础账号类型（包括10000，10001，10002）
export const AT_BRANCH_SHOP = 1; // 分店账号类型
export const AT_BRANCH_SHOP_PARTMENT = 2; // 分店部门账号类型
export const AT_CLIENT = 3; // 货主账号类型（物流公司的账号为物流公司负责人的账号，收货点的账号为收货点的账号）

/*
 * 银行对应账户对应银行卡里面的钱。对账的时候，需要看银行对应账户和银行卡的钱是否一致（利息另算）
 * 安全账户用来存放不可动用的部分（各种押金，当交易成功后才将这些钱划到相应的账户上）
 * 总部账户用户存放总部的提成。
 */
export const AC_BANK_ACCOUNT = 10000; // 银行对应账户
export const AC_SECURITY_ACCOUNT = 10001; // 安全账户
export const AC_MASTER_ACCOUNT = 10002; // 总部账户，总部可以支配的金额

// 对账方法：
// 银行对应账户 = 10001以上的账号总和 =  充值总额 - 提现总额

```

* PDShopServer/project/App/constants/authority.js

```js
/*
*
* 规则，只能添加，不能修改
*
*/

// 总部权限
export const AH_CREATE_BRANCH_SHOP = 0; // 创建分店的权限
export const AH_MODIFY_BRANCH_SHOP = 1; // 修改分店信息的权限
export const AH_REMOVE_BRANCH_SHOP = 2; // 删除分店的权限
export const AH_LOOK_BRANCH_SHOP = 3; // 查看分店列表的权限
export const AH_MODIFY_SETTING = 4; // 修改设置的权限
export const AH_CREATE_AGENT = 5; // 创建收货点的权限
export const AH_MODIFY_AGENT = 6; // 修改收货点的权限
export const AH_REMOVE_AGENT = 7; // 删除收货点的权限
export const AH_LOOK_AGENT = 8; // 查看收货点的权限

// 公共权限
export const AH_MODIFY_OWN_SHOP = 10000; // 修改所在物流超市信息的权限
export const AH_CREATE_MEMBER = 10001; // 创建成员的权限
export const AH_MODIFY_MEMBER = 10002; // 修改成员信息的权限
export const AH_REMOVE_MEMBER = 10003; // 删除成员的权限
export const AH_LOOK_MEMBER = 10004; // 查看成员的权限
export const AH_RECHARGE = 10005; // 充值的权限
export const AH_WITHDRAW = 10006; // 提现的权限
export const AH_LOOK_AMOUNT = 10007; // 查看余额的权限
export const AH_MODIFY_ROADMAP_PROFIT = 10008; // 修改路线的提成的权限
export const AH_LOOK_STATISTICS = 10009; // 查看统计信息的权限
export const AH_LOOK_ORDERS = 10010; // 查看综合货单的权限

// 分店权限
export const AH_CREATE_SHIPPER = 20000; // 创建物流公司的权限
export const AH_MODIFY_SHIPPER = 20001; // 修改物流公司的权限
export const AH_REMOVE_SHIPPER = 20002; // 删除物流公司的权限
export const AH_LOOK_SHIPPER = 20003; // 查看物流公司的权限
export const AH_CREATE_PARTMENT = 20004; // 创建部门的权限
export const AH_MODIFY_PARTMENT = 20005; // 修改部门信息的权限
export const AH_REMOVE_PARTMENT = 20006; // 删除部门的权限
export const AH_LOOK_PARTMENT = 20007; // 查看部门的权限
export const AH_MODIFY_OWN_PARTMENT = 20008; // 修改所在部门信息的权限
export const AH_RECEIVE_PARTMENT = 20009; // 收货的权限（收货部人员）
export const AH_WARE_HOUSE_PARTMENT = 20010; // 库管的权限（库管部人员）
export const AH_CARRY_PARTMENT = 20011; // 搬货的权限（搬运部人员）
export const AH_SECURITY_CHECK_PARTMENT = 20012; // 安检的权限（保安部人员）
export const AH_DISTRIBUTION_PARTMENT = 20013; // 配送的权限（配送部人员）
export const AH_TO_EXAMINE_TRUCK = 20014; // 审核货车的权限
export const AH_CREATE_WAREHOUSE = 20015; // 创建仓库的权限
export const AH_MODIFY_WAREHOUSE = 20016; // 修改仓库信息的权限
export const AH_REMOVE_WAREHOUSE = 20017; // 删除仓库的权限
export const AH_LOOK_WAREHOUSE = 20018; // 查看仓库的权限

// 物流公司
export const AH_MODIFY_SHIPPER_INFO = 30000; // 修改物流公司信息的权限
export const AH_MODIFY_SHIPPER_MEMBER_AUTHORITY = 30001; // 修改物流公司成员权限的权限
export const AH_CREATE_ROADMAP = 30002; // 创建路线的权限
export const AH_MODIFY_ROADMAP = 30003; // 修改路线的权限
export const AH_REMOVE_ROADMAP = 30004; // 删除路线的权限
export const AH_LOOK_ROADMAP = 30005; // 查看路线的权限 (分店共享该权限)
export const AH_CREATE_SEND_DOOR_ROADMAP = 30006; // 创建同城配送路线的权限
export const AH_MODIFY_SEND_DOOR_ROADMAP = 30007; // 修改同城配送路线的权限
export const AH_REMOVE_SEND_DOOR_ROADMAP = 30008; // 删除同城配送路线的权限
export const AH_LOOK_SEND_DOOR_ROADMAP = 30009; // 查看同城配送路线的权限 (分店共享该权限)
export const AH_CREATE_TRUCK = 30010; // 创建货车的权限
export const AH_MODIFY_TRUCK = 30011; // 修改货车的权限
export const AH_REMOVE_TRUCK = 30012; // 删除货车的权限
export const AH_LOOK_TRUCK = 30013; // 查看货车的权限
export const SELECT_CARRY_PARTMENT = 30014; // 选择搬运队的权限
export const AH_SCAN_LOAD_TRUCK = 30015; // 扫描装车的权限
export const AH_CITY_DISTRIBUTE = 30016; // 同城配送的权限

// 收货点
export const AH_MODIFY_AGENT_INFO = 40000; // 修改收货点信息的权限
export const AH_MODIFY_AGENT_MEMBER_AUTHORITY = 40001; // 修改收货点成员权限的权限
export const AH_PLACE_ORDER = 40002; // 收货点下单的权限

```

* PDShopServer/project/App/constants/common.js

```js
// 支付方式
export const PM_IMMEDIATE = 0; // 现付
export const PM_REACH = 1; // 到付
export const PM_MIXED = 2; // 混合支付（指定收款的部分到付，其余的现付）

// 支付工具
export const PT_CASH = 0; // 现金支付
export const PT_ALIPAY = 1; // 支付宝
export const PT_WEIXIN = 2; // 微信
export const PT_BANK = 3; // 银行

// 物流公司类型
export const ST_LONG = 0; // 长途物流公司
export const ST_CITY = 1; // 同城配送物流公司

// 方向设置的类型
export const RRT_ALL = 0; // 所有
export const RRT_LONG = 1; // 长途
export const RRT_CITY = 2; // 同城配送

// 公里与弧度的计算比
export const KILOMETRE_RADIAN_RATE = 111.12;

```

* PDShopServer/project/App/constants/index.js

```js
import * as common from './common';
import * as authority from './authority';
import * as account from './account';
import * as state from './state';
import * as post from './post';
import * as partment from './partment';

export default {
    ...common,
    ...authority,
    ...account,
    ...state,
    ...post,
    ...partment,
};

```

* PDShopServer/project/App/constants/partment.js

```js
// 分店
export const SP_RECEIVE_PARTMENT = 0; // 收货部
export const SP_WARE_HOUSE_PARTMENT = 1; // 库管部
export const SP_CARRY_PARTMENT = 2; // 搬运部
export const SP_SECURITY_CHECK_PARTMENT = 3; // 保安部

```

* PDShopServer/project/App/constants/post.js

```js
// 职位
export const PO_CHAIR_MAN = '董事长';
export const PO_PARTMENT_CHARGE_MAN = '部门经理'; // 五大部门的经理
export const PO_CEO = '总经理';
export const PO_PARTMENT_LEADER = '部门主管'; // 五大部门的经理
export const PO_STAFF = '员工';

```

* PDShopServer/project/App/constants/state.js

```js
// 货单的状态
/*
===========
长途运输状态图:(当物流公司确认卸货之后，那么它之前的货单就在待交货状态)
(0->)1->2->5    |           |->11
                |->6->7->8  |
(0->)1->3->4->5 |           |->14->15
===========
收货点状态图:
0->2->9    |
           |(->10)->11->14->15
0->3->4->9 |
===========
同城配送状态图: (当同城配送的在运输中，那么它之前的货单就在待交货状态)
(0->)1->2->5    |
                |->6->8->15
(0->)1->3->4->5 |
===========
上门自提状态图:(当上门自提的在运输中或者等待上门自提，那么它之前的货单就在待交货状态)
         |->13->15
1->5->12 |
         |->6->8->15
*/
export const OS_PREORDER = -1; // 预下单状态，需要将货拉到分店后不显示（件数，重量，方量），由收货员填写确认后进入待选线路状态
export const OS_READY_PRINT_BAR_CODE = 0; // 待打印二维码
export const OS_READY_SELECT_ROADMAP = 1; // 待选物流公司
export const OS_READY_DEDUCT_AND_PRINT_ORDER = 2; // 待扣款并打印打印货单（用来区别需要付款的情况）
export const OS_READY_PAYMENT = 3; // 待付款
export const OS_READY_PRINT_ORDER = 4; // 待打印货单
export const OS_READY_STOCK = 5; // 待入库
export const OS_READY_LOAD = 6; // 待装车
export const OS_READY_START_OFF = 7; // 待出发
export const OS_ON_THE_WAY = 8; // 运输中
export const OS_READY_SEND = 9; // 待发货 (收货点专用)
export const OS_ON_SENDING = 10; // 发货中 (收货点专用)
export const OS_ON_TRANSFER = 11; // 中转中
export const OS_READY_PHONE_CONFIRM = 12; // 待打电话确认
export const OS_WAIT_CLIENT_PICK = 13; // 等待上门自提
export const OS_READY_RECEIVE = 14; // 待交货
export const OS_RECEIVE_SUCCESS = 15; // 交货成功

// 货车的状态
export const TS_READY_EXAMINE = 0; // 待审核
export const TS_READY_ENTER_WAREHOUSE = 1; // 待入库
export const TS_READY_SEARCH_CARRY_PARTMENT = 2; // 待找搬运队
export const TS_READY_SCAN_LOAD = 3; // 待扫描装车
export const TS_SCAN_LOADING = 4; // 装车中
export const TS_READY_PAY_FOR_CARRY_PARTMENT = 5; // 待付搬运部的款
export const TS_READY_EXIT_WAREHOUSE = 6; // 待出库
export const TS_ON_THE_WAY = 7; // 运输中
export const TS_RECEIVE_SUCCESS = 8; // 驾驶员或者物流公司的人点击确认到达后的状态

// 支付状态
export const PS_READY_PAY = 0; // 待支付
export const PS_READY_SAVE = 1; // 待同步数据
export const PS_SUCCESS = 2; // 支付完成

```

* PDShopServer/project/App/manager/index.js

```js
import _settingMgr from './settingMgr';

export const settingMgr = _settingMgr;

export const setting = _settingMgr.value;

export function init () {
    return Promise.all([
        _settingMgr.init(),
    ]);
}

```

* PDShopServer/project/App/manager/settingMgr.js

```js
import { SettingModel } from '../models';
import { _N } from '../utils';
import _ from 'lodash';

class Manager {
    constructor () {
        this.value = {};
    }
    init () {
        return new Promise(async resolve => {
            const data = await SettingModel.findOne();
            Object.assign(this.value, data ? data.toObject() : null);
            return resolve();
        });
    }
    set (data) {
        Object.assign(this.value, data.toObject());
    }
    getRealWeight (weight, size) {
        return _N(Math.max(weight, size / this.value.sizeWeightRate));
    }
    getFloatWeight (weight, size) {
        weight = this.getRealWeight(weight, size);
        return _.findLast(this.value.gradientPriceList, o => o.min <= weight).rate * weight;
    }
    getBondAmount (weight, size) {
        weight = this.getRealWeight(weight, size);
        return this.value.bondAmountWeightRate * weight;
    }
}

module.exports = new Manager();

```

* PDShopServer/project/App/models/AdminModel.js

```js
import mongoose from 'mongoose';
import passportLocalMongoose from 'passport-local-mongoose';
import { virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    phone: { type: String, required: true, unique: true }, // 注册电话
    email: { type: String }, //注册邮箱，用来找回密码
});

schema.plugin(passportLocalMongoose, { usernameField: 'phone' });

virtualObjectById(schema);

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatObjectById(ret);
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Admin', schema);

```

* PDShopServer/project/App/models/AgentModel.js

```js
import mongoose from 'mongoose';
import { formatMedia, formatTime, getMediaPath, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

/*
 * 当发货人将货物拉到收货点的时候，收货点的人员通过填写重量和方量后，显示出路线排名，依据是对应的referShopId的路线，显示价格为该收货点设置的提成比*实际价格
 * 如果收货点对该条路线设置了单独的提成比，优先使用，否则使用默认的提成比
 * 当客户使用预下单和查看行情时，也需要显示提价后的价格
 */

const schema = new Schema({
    chairManId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 董事长(法人)
    // 基础信息
    name: { type: String }, // 收货点名称
    image: { type: String }, // 收货点背景图片
    sign: { type: String }, // 收货点签名
    addressRegion: { type: String }, // 物流超市所在城市
    addressRegionLastCode: { type: Number }, // 物流超市所在城市的lastCode
    address: { type: String }, // 收货点地址
    location: { type: [ Number ], index: '2d' }, // 经纬度定位(必须使用地图来选择)
    phoneList: { type: String }, // 收货点电话列表

    // 法人信息
    legalName: { type: String }, // 法人姓名
    legalPhone: { type: String }, // 法人电话
    legalIDCard: [{ type: Schema.Types.ObjectId, ref: 'Media' }], // 法人身份证

    // 参考店铺
    referShopId: { type: Schema.Types.ObjectId, ref: 'Shop' },

    createTime: { type: Date, default: Date.now }, //创建时间
});

virtualObjectById(schema, 'chairManId', 'referShopId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        ret.name = ret.name || ret.phone;
        formatMedia(ret, 'image');
        formatTime(ret, 'createTime');
        formatObjectById(ret, 'chairManId', 'referShopId');
        ret.legalIDCard && (ret.legalIDCard = ret.legalIDCard.map(o => getMediaPath(o)));
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Agent', schema);

```

* PDShopServer/project/App/models/AgentRegionProfitModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import { formatTime, virtualObjectById, formatObjectById } from '../utils';

const schema = new Schema({
    agentId: { type: Schema.Types.ObjectId, ref: 'Agent' }, // 所属收货点
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 所属分店 Id， 如果为空，则为所有的分店

    region: { type: String }, // 方向，如：北京（必须使用地图选择）
    regionLastCode: { type: Number, default: 0 },  // 方向的最后一个code

    profitRate: { type: Number, default: 0 }, // 收货点对路线的提成比例
    profitCount: { type: Number, default: 0 }, // 收货点对路线的提成价格 （优先于profitRate）

    type: { type: Number, default: 0 }, // 类型，0：所有，1：长途，2：短途

    level: { type: Number, default: 0 },  // 内部使用，用来排序
    createTime: { type: Date, default: Date.now }, // 创建时间
    modifyTime: { type: Date }, //修改时间
});

virtualObjectById(schema, 'shopId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'createTime', 'modifyTime');
        formatObjectById(ret, 'shopId');

        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('AgentRegionProfit', schema);

```

* PDShopServer/project/App/models/CityTruckModel.js

```js
import mongoose from 'mongoose';
import { formatTime, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    shipperId: { type: Schema.Types.ObjectId, ref: 'Shipper' }, // 所属物流公司 Id
    driverId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 对应的司机(同城配送的物流公司员工)

    enable: { type: Boolean, default: true }, // 是否可用，当董事长添加一个成员权限的时候，默认是可用的，当删除权限的时候，车辆关系不删除，设置为不可用的状态
    plateNo: { type: String }, // 车牌号码
    locationList: [ { address: String, longitude: Number, latitude: Number, time: Date } ], // 定位列表

    // 货单
    orderList: [ { type: Schema.Types.ObjectId, ref: 'Order' } ], // 货单列表
    orderCount: { type: Number, default: 0 }, // 总单数
    totalNumbers: { type: Number, default: 0 }, // 总件数
    totalWeight: { type: Number, default: 0 }, // 总重量
    totalSize: { type: Number, default: 0 }, // 总方量
    unloadAllOrderList: [ { type: Schema.Types.ObjectId, ref: 'Order' } ], // 只上了一部分货单列表

    createTime: { type: Date, default: Date.now }, // 创建时间
    modifyTime: { type: Date }, // 修改时间
});

virtualObjectById(schema, 'driverId', 'shipperId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'createTime', 'modifyTime');
        formatObjectById(ret, 'driverId', 'shipperId');
        ret.locationList && (ret.locationList = ret.locationList.map(item => { formatTime(item, 'time'); delete item._id; return item; }));
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('CityTruck', schema);

```

* PDShopServer/project/App/models/ClientModel.js

```js
import mongoose from 'mongoose';
import passportLocalMongoose from 'passport-local-mongoose';
import { _T, formatTime, formatMedia, formatPhoneList, virtualObjectById, formatObjectById } from '../utils';
import moment from 'moment';
import { AccountModel } from '../mysql';
import CONSTANTS from '../constants';
const Schema = mongoose.Schema;

const schema = new Schema({
    phone: { type: String, required: true, unique: true }, // 注册电话
    email: { type: String }, // 注册邮箱，用来找回密码
    name: { type: String }, // 姓名
    sex: { type: Number, default: 0 }, // 性别 0:男   1:女
    head: { type: Schema.Types.ObjectId, ref: 'Media' }, // 用户头像
    birthday: { type: String }, // 生日
    address: { type: String }, // 住址
    phoneList: { type: String }, // 预备电话列表

    authority: [ { type: Number } ], // 拥有的权限
    isSetPaymentPassword: { type: Number, default: 0 }, // 性别 0:没设置   1:已设置

    shipperId: { type: Schema.Types.ObjectId, ref: 'Shipper' }, // 所属物流公司
    truckId: { type: Schema.Types.ObjectId, ref: 'CityTruck' }, // 同城配送的人员对应的货车
    agentId: { type: Schema.Types.ObjectId, ref: 'Agent' }, // 所属收货点
    // 时间
    createTime: { type: Date, default: Date.now }, // 创建时间
    registerTime: { type: Date }, // 注册时间（只有实际注册的用户才有这个时间）
});
schema.plugin(passportLocalMongoose, { usernameField: 'phone' });

schema.virtual('age').get(function () {
    return this.birthday && Math.floor(moment().diff(moment(this.birthday)) / 31536000000);
});

virtualObjectById(schema, 'shipperId', 'truckId', 'agentId');

schema.statics.checkExistElseCreate = async function (phone, name) {
    let user = await this.findOne({ phone });
    if (!user) {
        user = new this({ phone, name });
        const account = await AccountModel.create({ targetId: _T(user.id), type: CONSTANTS.AT_CLIENT });
        if (!account) {
            return undefined;
        }
        await user.save();
    } else if (!user.registerTime && !user.name) { // 如果对方还没有注册，可以修改对方的用户名
        user.name = name;
        await user.save();
    }
    return user.id;
};

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'registerTime');
        formatMedia(ret, 'head');
        formatObjectById(ret, 'shipperId', 'truckId', 'agentId');
        formatPhoneList(ret);
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Client', schema);

```

* PDShopServer/project/App/models/CollectionModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import { formatTime, virtualObjectById, formatObjectById } from '../utils';

const schema = new Schema({
    clientId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 客户 Id
    roadmapId: { type: Schema.Types.ObjectId, ref: 'Roadmap' }, // 路线 Id
    collectionTime: { type: Date, default: Date.now }, //收藏时间
});

virtualObjectById(schema);

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'collectionTime');
        formatObjectById(ret);
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Collection', schema);

```

* PDShopServer/project/App/models/CommentModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import { getMediaPath, formatTime, virtualObjectById, formatObjectById } from '../utils';

const schema = new Schema({
    clientId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 客户 Id
    roadmapId: { type: Schema.Types.ObjectId, ref: 'Roadmap' }, // 线路 Id
    commentTime: { type: Date, default: Date.now }, // 评论时间
    content: { type: String }, // 评论内容
    imgList: [{ type: Schema.Types.ObjectId, ref: 'Media' }], //图片列表
});

virtualObjectById(schema, 'clientId');

const transform = {
    virtuals: true,
    depopulate: ['clientId'],
    transform: function (doc, ret, options) {
        formatTime(ret, 'commentTime');
        formatObjectById(ret, 'clientId');
        ret.imgList && (ret.imgList = ret.imgList.map(item => getMediaPath(item)));
        ret.userId = ret.clientId;
        delete ret.clientId;
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Comment', schema);

```

* PDShopServer/project/App/models/DriverModel.js

```js
import mongoose from 'mongoose';
import passportLocalMongoose from 'passport-local-mongoose';
import moment from 'moment';
import { formatTime, formatMedia, formatPhoneList, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
  // 基本信息
    phone: { type: String, required: true, unique: true }, // 注册电话
    email: { type: String }, // 注册邮箱，用来找回密码
    name: { type: String }, // 姓名
    sex: { type: Number, default: 0 }, // 性别 0:男   1:女
    head: { type: Schema.Types.ObjectId, ref: 'Media' }, // 用户头像
    birthday: { type: String }, // 生日
    address: { type: String }, // 住址
    phoneList: { type: String }, // 预备电话列表

    license: { type: Schema.Types.ObjectId, ref: 'Media' }, // 驾驶证图片
    licenseNo: { type: String }, // 驾驶证号
    insuranceEndTime: { type: Date }, // 保险到期时间

    registerTime: { type: Date, default: Date.now }, //注册时间
});
schema.plugin(passportLocalMongoose, { usernameField: 'phone' });

schema.virtual('age').get(function () {
    return this.birthday && Math.floor(moment().diff(moment(this.birthday)) / 31536000000);
});

virtualObjectById(schema);

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'registerTime');
        formatMedia(ret, 'head');
        formatObjectById(ret);
        formatPhoneList(ret);
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Driver', schema);

```

* PDShopServer/project/App/models/FeedbackModel.js

```js
import mongoose from 'mongoose';
import { formatTime, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    clientId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 客户Id
    shopMemberId: { type: Schema.Types.ObjectId, ref: 'ShopMember' }, // 商户Id
    driverId: { type: Schema.Types.ObjectId, ref: 'Driver' }, // 司机Id
    content: { type: String, required: true }, // 反馈内容
    email: { type: String }, // 联系邮箱
    state: { type: Number, default: 0 }, // 申请发卡的状态， 0：未处理，1：处理中，2：已处理
    submitTime: { type: Date, default: Date.now }, // 反馈时间
    acceptTime: { type: Date }, // 接收时间
    finishTime: { type: Date }, //处理时间
});

virtualObjectById(schema);

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'submitTime', 'acceptTime', 'finishTime');
        formatObjectById(ret);
        ret.state >= 1 || (ret.acceptTime = undefined);
        ret.state === 2 || (ret.finishTime = undefined);
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Feedback', schema);

```

* PDShopServer/project/App/models/LogModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;

const schema = new Schema({
    operator: { id: { type: String }, model: { type: String } }, // 操作人
    target: { id: { type: String }, model: { type: String } }, // 操作对象
    targetList: [{ id: { type: String }, model: { type: String } }], // 操作对象
    action: { type: String }, // 行为
    remark: { type: String }, // 备注
    createTime: { type: Date, default: Date.now }, //创建时间
});

export default mongoose.model('Log', schema);

```

* PDShopServer/project/App/models/MediaModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import _ from 'lodash';

const schema = new Schema({
    ref: { type: Number, default: 0 }, // 文件引用次数，初次上传的时候，ref为1，当文件被使用一次，ref+1, 被弃用一次，ref-1，当ref为0的时候删除文件和记录
    type: { type: Number, default: 0 }, // 文件类型，0：图片，1，音频，2，视频
    gridId: { type: Schema.Types.ObjectId }, //在gridfs中的id
});

schema.statics._updateRef = function (...params) {
    let obj = _.mergeWith(...params, (x, y) => (x || 0) + (y || 0));
    _.map(obj, (value, key) => {
        if (key !== 'undefined' && value !== 0) {
            this.findByIdAndUpdate(key, { $inc: { ref: value } }).exec();
        }
    });
};

export default mongoose.model('Media', schema);

```

* PDShopServer/project/App/models/NotifyModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import { formatTime, virtualObjectById, formatObjectById } from '../utils';

const schema = new Schema({
    adminId: { type: Schema.Types.ObjectId, ref: 'Admin' }, // 发布通知的后台管理员Id
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 物流超市的Id
    memberId: { type: Schema.Types.ObjectId, ref: 'ShopMember' }, // 具体发通知的人的Id
    shipperId: { type: Schema.Types.ObjectId, ref: 'Shipper' }, // 发布通知的物流公司
    type: { type: String, default: 'news' }, // 通知类型  news, publicity, policy, notice
    source: { type: String, required: true }, // 通知来源
    title: { type: String, required: true }, // 通知标题
    content: { type: String, required: true }, // 通知内容
    time: { type: Date, default: Date.now }, //发布时间
});

virtualObjectById(schema);

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'time');
        formatObjectById(ret);
        delete ret.adminId;
        delete ret.type;
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Notify', schema);

```

* PDShopServer/project/App/models/OrderGroupModel.js

```js
import mongoose from 'mongoose';
import { formatTime, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    name: { type: String }, // 名称
    senderId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 发货的客户 Id (如果是物流公司，就是物流公司的负责人，如果是收货点，就是收货点的负责人)
    // 始发地
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 这个货单隶属于哪家分店（预下单的情况下为指定的某家分店）
    agentId: { type: Schema.Types.ObjectId, ref: 'Agent' }, // 这个货单隶属于哪家收货点预下单的情况下为指定的某家收货点）
    startPoint: { type: String }, // 省市（只对预下单有用，如果有 shopId 或 agentId 时，该字段无效）
    startPointLastCode: { type: Number }, // 始发地的lastCode

    // 货单列表
    orderIdList: [{ type: Schema.Types.ObjectId, ref: 'Order' }], // 货单列表
    orderCount: { type: Number, default: 0 }, // 货单的单数
    totalNumbers: { type: Number, default: 0 }, // 总的货物的件数
    totalWeight: { type: Number, default: 0 }, // 重量
    totalSize: { type: Number, default: 0 }, // 方量

    createTime: { type: Date, default: Date.now }, // 下单时间
    modifyTime: { type: Date }, // 修改时间
});

virtualObjectById(schema, 'shopId', 'agentId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'createTime', 'modifyTime');
        formatObjectById(ret, 'shopId', 'agentId');
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('OrderGroup', schema);

```

* PDShopServer/project/App/models/OrderModel.js

```js
import mongoose from 'mongoose';
import { _T, formatTime, formatMedia, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    isTransferOrder: { type: Boolean, default: false }, // 是否是中转单
    preOrderId: { type: Schema.Types.ObjectId, ref: 'Order' }, // 上一个货单的 Id
    nextOrderId: { type: Schema.Types.ObjectId, ref: 'Order' }, // 下一个货单的 Id

    groupId: { type: Schema.Types.ObjectId, ref: 'OrderGroup' }, // 批次的 Id

    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 这个货单隶属于哪家分店（预下单的情况下为指定的某家分店）
    agentId: { type: Schema.Types.ObjectId, ref: 'Agent' }, // 这个货单隶属于哪家收货点预下单的情况下为指定的某家收货点）

    senderId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 发货的客户 Id (如果是物流公司，就是物流公司的负责人，如果是收货点，就是收货点的负责人)
    senderName: { type: String }, // 货单使用的发货人姓名
    senderPhone: { type: String }, // 货单使用的发货人姓名
    isSenderRepresentShipper: { type: Boolean, default: false }, // 发货人是否代表一家物流公司，如果是，需要更新物流公司的账目
    receiverId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 发货的客户 Id
    receiverName: { type: String }, // 货单使用的收货人姓名
    receiverPhone: { type: String }, // 货单使用的收货人姓名
    name: { type: String }, // 货物名称
    photo: { type: Schema.Types.ObjectId, ref: 'Media' }, // 货物拍照
    startPoint: { type: String }, // 始发地省市县
    startPointLastCode: { type: Number, default: 0 },  // 始发地的lastCode（只对预下单有用，如果有 shopId 或 agentId 时，该字段无效）

    isCityDistribute: { type: Boolean, default: false }, // 是否是同城配送
    endPoint: { type: String }, // 长途终点，精确到镇
    endPointLastCode: { type: Number, default: 0 },  // 送货上门终点的lastCode

    isSendDoor: { type: Boolean, default: false }, // 是否送货上门
    sendDoorEndPoint: { type: String }, // 送货上门终点，精确到镇
    sendDoorEndPointLastCode: { type: Number, default: 0 },  // 送货上门终点的lastCode

    isClientPick: { type: Boolean, default: false }, // 是否上门自提

    roadmapId: { type: Schema.Types.ObjectId, ref: 'Roadmap' }, // 线路 Id
    roadmapRankIndex: { type: Number, default: -1 }, // 该货单选择的路线在选择时的排序名次，交易成功后，需要根据这个名次修改 RoadmapMaskModel 的数据
    shipperId: { type: Schema.Types.ObjectId, ref: 'Shipper' }, // 物流公司 Id
    needBondAmount: { type: Number, default: 0 }, // 需要担保的金额
    shipperChairManId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 物流公司董事长Id
    truckId: { type: Schema.Types.ObjectId, ref: 'Truck' }, // 货车 Id

    // 路线的价格信息
    price: { type: Number, default: 0 }, // 成交的价格
    minFee: { type: Number, default: 0 }, // 成交的底价
    sendDoorPrice: { type: Number, default: 0 }, // 成交的送货上门价格
    sendDoorMinFee: { type: Number, default: 0 }, // 成交的送货上门底价

    receivePartmentId: { type: Schema.Types.ObjectId, ref: 'ShopPartment' }, // 收货的部门
    receiveMemberId: { type: Schema.Types.ObjectId, ref: 'ShopMember' }, // 具体收货制单的人员的Id
    receiveAgentMemberId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 具体收货制单的收货点人员的Id
    placeOrderTime: { type: Date }, // 下单时间
    warehouse: { type: String }, // 仓库（一家物流公司可能有多个仓库，都显示出来，用分号分开，拉过去由库管决定, 如果没有找到仓库，则显示该物流公司名称）
    storeScannerId: { type: Schema.Types.ObjectId, ref: 'ShopMember' }, // 入库扫描的库管员 Id
    shipperStoreScannerId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 入库扫描的物流公司员工 Id
    placeStoreTime: { type: Date }, // 入库时间
    loadScannerId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 上车扫描的物流公司员工 Id (同城配送的物流公司为同城配送的司机)
    loadScanTime: { type: Date }, // 装车时间
    confirmHandOverId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 确认交货的流公司员工 Id（只有长途运输公司）

    // 必须由称货员填写的信息
    totalNumbers: { type: Number, default: 0 }, // 总的货物的件数（具体的数量以这个数量为准）
    weight: { type: Number, default: 0 }, // 重量
    size: { type: Number, default: 0 }, // 方量

    // 保险
    isInsuance: { type: Boolean, default: false }, // 是否保价 [初始货单]，如果保价，保价个度为运费的setting.insuanceMountRate倍
    insuanceMount: { type: Number, default: 0 }, // 担保保险额度（需要赔偿用户的钱）[初始货单]
    insuanceFee: { type: Number, default: 0 }, // 担保保险的费用（客户实际付的钱）(必须现付)[初始货单]

    /*
     * 如果是到付，当创建中转货单时，还不知道需要多少运费，不能填写指定收款，当选定路线确定价格后，指定收款 = 总运费 - 屏幕呈现价格 fee（物流公司价格+分店提成+总店提成）
     * 验证方式： fee + masterProfit + branchProfit 为呈现给用户的价格
     * designatedFee = totalFee - 下一家物流公司的屏幕报价(下一家物流公司的实际报价+分店提成+总店提成)
     */
    // 付款方式
    isReachPay: { type: Boolean, default: false }, // 是否是到付
    payMode: { type: Number, default: 0 }, // 支付方式 0：现付（PM_IMMEDIATE），1：到付（PM_REACH），2：混合支付（PM_MIXED 指定收款的部分到付，其余的现付）
    payTool: { type: Number, default: 0 }, // 现付的情况下的支付工具 0：现金支付（PT_CASH），1：支付宝（PT_ALIPAY），2：微信（PT_WEIXIN），3：银行卡（PT_BANK）

    fee: { type: Number, default: 0 }, // 运输价格（物流公司实际价格，没有经过加价）（自动计算）
    // 提成金额
    profit: { type: Number, default: 0 }, // 总的提成
    masterProfit: { type: Number, default: 0 }, // 总部的提成
    branchProfit: { type: Number, default: 0 }, // 分部的提成
    realFee: { type: Number, default: 0 }, // 屏幕显示价格 fee+masterProfit+branchProfit

    totalDesignatedFee: { type: Number, default: 0 }, // 全程总的指定收款(只有payMode为到付的情况下有效，实际为到付时向收货人收的运费)
    designatedFee: { type: Number, default: 0 }, // 指定收款 (只有payMode为到付的情况下，才可能有指定收款)

    // 代收货款
    proxyCharge: { type: Number, default: 0 }, // 代收货款金额
    proxyChargeProfit: { type: Number, default: 0 }, // 代收货款手续费（交易成功后，从代收货款的金额中扣除，只给总部）[初始货单]

    // 该货单需要现付的项目
    needPayTransportFee: { type: Number, default: 0 }, // 需要现场付的运费
    needPayInsuanceFee: { type: Number, default: 0 }, // 需要现场付的保险

    // 附加费用 (上门自提需要送的这种货的收货人谈定的价格)
    additionalFee: { type: Number, default: 0 }, // 附加费用

    /* 状态：
    * 货主把货拉倒分店收货部，收货部对其点数，称重，量方量后填单，然后调用 < placeOriginOrder > 成功后为待选线路状态
    * 0：待选线路（安装所填的数据，给货主显示路线排名，让货主选择一条路线，付款方式，是否交保险，是否代收货款，是否送货上门，选择后显示运费和其他费用的列表和总的费用，点击确认按钮，调用接口 < confirmSelectRoadmap >后，如果是需要付钱的进入待付款状态，否则进入待入库状态）。
    * 1：待付款状态（1.货主在手机上刷新，使用支付宝等付款，成功后告知收货员，收货员刷新货单，显示收款成功，2.货主将现金给收货员，收货员点击代付款按钮，付款成功后，显示收款成功；之后进入待入库状态）
    * 2：待入库（指收款成功后搬运人员将货物运到库管处的状态，交给库管成功后，状态进入待装车状态）
    * 3：待装车（货物送到库管处，每收5吨货就通知物流公司，物流公司自己看有一车的时候会生成一个货车实例<包括司机和货车信息>到检查部审核，审核成功后，将车开到库管处，物流公司会通过竞价联系搬运部，搬运部搬运货物上车，物流公司需要一件一件的扫描，装车完成，需要给库管确认，库管确认之后，进入待出发状态）
    * 4：待出发 （物流公司将车开到门卫处，门卫扫描驾驶员的二维码，点击放行按钮，进入运输中状态）
    * 5：运输中 （运输过程中，驾驶员需要开启app，自动上传位置信息）
    * 6：交货成功 （司机将货物送到目的地之后，会劝说收货人安装app或者扫描小程序，然后让他扫描货物，确认收货的件数，完成交接）
    * 不能包含相同的状态
    */
    stateList: [ { state: Number, count: Number } ],
    deductError: { type: Boolean }, // 扣款失败（该字段只针对 state 为 OS_READY_DEDUCT_AND_PRINT_ORDER 和 OS_READY_PAYMENT 的货单有效）

    createTime: { type: Date, default: Date.now }, // 创建时间
    modifyTime: { type: Date }, // 修改时间

    remark: { type: String }, //备注
});

virtualObjectById(schema, 'senderId', 'receiverId', 'shopId', 'agentId', 'roadmapId', 'shipperId', 'receiveMemberId', 'receiveAgentMemberId');

schema.statics.getOrderFromOriginOrder = async function (targetId, orderId, type = 0) { // type值， 0：分店，1：物流公司，2：发货人，3：收货人
    while (orderId) {
        const order = await this.findById(orderId);
        if (!order) {
            return undefined;
        }
        if (type === 0 && _T(order.shopId) === _T(targetId)) {
            return order;
        } else if (type === 1 && _T(order.shipperId) === _T(targetId)) {
            return order;
        } else if (type === 2 && _T(order.senderId) === _T(targetId)) {
            return order;
        } else if (type === 3 && _T(order.receiverId) === _T(targetId)) {
            return order;
        }
        orderId = order.nextOrderId;
    }
    return undefined;
};
schema.statics.getOrderIdListFromNode = async function (order, includeSelf) {
    const orderIdList = includeSelf ? [order.id] : [];
    while (order.preOrderId) {
        order = await this.findById(order.preOrderId);
        order && orderIdList.unshift(order.id);
    }
    return orderIdList;
};
schema.statics.getLatestOrderFromOriginOrder = async function (orderId) {
    let order = await this.findById(orderId);
    while (order && order.nextOrderId) {
        order = await this.findById(order.nextOrderId);
    }
    return order;
};
schema.statics.getOriginOrderFromNode = async function (orderId) {
    let order = await this.findById(orderId);
    while (order && order.preOrderId) {
        order = await this.findById(order.preOrderId);
    }
    return order;
};

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatMedia(ret, 'photo');
        formatTime(ret, 'createTime', 'modifyTime', 'placeOrderTime', 'placeStoreTime', 'loadScanTime');
        formatObjectById(ret, 'senderId', 'receiverId', 'shopId', 'agentId', 'roadmapId', 'shipperId', 'receiveMemberId', 'receiveAgentMemberId');
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Order', schema);

```

* PDShopServer/project/App/models/PaymentModel.js

```js
import mongoose from 'mongoose';
import passportLocalMongoose from 'passport-local-mongoose';
import { virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    userId: { type: String }, // 对应的用户和店员的Id
    modifyTime: { type: Date, default: Date.now }, // 修改时间
});

schema.plugin(passportLocalMongoose, { usernameField: 'userId' });

virtualObjectById(schema);

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatObjectById(ret);
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Payment', schema);

```

* PDShopServer/project/App/models/RoadmapAddressModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;

const schema = new Schema({
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 所属分店 Id

    code: { type: Number }, // 地址code
    parentCode: { type: Number }, // 父级地址code
    name: { type: String }, // 名称
    level: { type: Number }, // 级别
});

schema.virtual('isLeaf').get(function () {
    return (
        this.level === 0 ||
        this.level === 4 ||
        this.level === 5 ||
        this.level === 6 ||
        this.level === 7 ||
        this.level === 10
    );
});

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        delete ret.__v;
        delete ret._id;
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('RoadmapAddress', schema);

```

* PDShopServer/project/App/models/RoadmapMaskModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import { setting } from '../manager';
import _ from 'lodash';
/*
* 屏蔽规则：
* 开始屏蔽时，获取该路线的statusList中上次收货时间最小的一个，如果该名次的收货量小于最大收货量，则屏蔽该名次之前的名次的路线；否则查找收货时间第二小的名次，如果该名次的收货量小于最大收货量，则屏蔽该名次之前的名次的路线；否则继续查找，如果都没有查找到。调整 receivedWeight 为 receivedWeight-maxWeight，重复以上动作。
* 如果
*/
const schema = new Schema({
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 所属分店的Id
    endPointLastCode: { type: Number }, // 终点（精确到镇级别），判断只有终点相同的为同意路线 (如果endPointLastCode为0，则代表上门自提的过滤)
    statusList: [{
        receivedWeight: Number, // 序号名次的已经收到的货物重量，单位（吨）
        maxWeight: Number, // 最大可以收货的量
        lastReceivedTime: Date, //序号名次的上次收货的时间
    }],
});

// maxCount 为总的路线条数，如果总路线的数量小于rankedMaskRateList的长度，取 maxCount
schema.statics.getMaskNumber = async function (shopId, endPointLastCode, maxCount) {
    const doc = await this.findOne({ shopId, endPointLastCode });
    let statusList = doc ? doc.statusList : [];
    const rankLength = Math.min(setting.rankedMaskRateList.length, maxCount);

    if (!rankLength) {
        return 0;
    }
    if (statusList.length < rankLength) {
        return statusList.length;
    }
    if (statusList.length > rankLength) {
        statusList = statusList.slice(0, rankLength);
    }
    const list = _.orderBy(_.map(_.cloneDeep(statusList), (m, i) => { m.index = i; return m; }), o => o.lastReceivedTime);
    const listItem = _.find(list, o => o.receivedWeight < o.maxWeight);
    if (listItem) {
        return listItem.index;
    }
    for (const item of doc.statusList) {
        item.receivedWeight -= item.maxWeight;
    }
    await doc.save();

    return list[0].index;
};

schema.statics.updateStatusList = async function (shopId, endPointLastCode, index, weight) {
    if (index >= setting.rankedMaskRateList.length) {
        return;
    }
    let doc = await this.findOne({ shopId, endPointLastCode });
    if (!doc) {
        const statusList = [];
        const maxWeight = (setting.rankedMaskFirstRankWeight * setting.rankedMaskRateList[index]) || 0;
        statusList[index] = { receivedWeight: weight, maxWeight, lastReceivedTime: Date.now() };
        for (let i = 0; i < index; i++) { // 防止数组为空
            !statusList[i] && (statusList[i] = {
                receivedWeight: 0,
                maxWeight: (setting.rankedMaskFirstRankWeight * setting.rankedMaskRateList[i]) || 0,
                lastReceivedTime: 0,
            });
        }
        doc = new this({ shopId, endPointLastCode, statusList });
    } else {
        const item = doc.statusList[index];
        if (!item) {
            const maxWeight = (setting.rankedMaskFirstRankWeight * setting.rankedMaskRateList[index]) || 0;
            doc.statusList[index] = { receivedWeight: weight, maxWeight, lastReceivedTime: Date.now() };
            for (let i = 0; i < index; i++) { // 防止数组为空
                !doc.statusList[i] && (doc.statusList[i] = {
                    receivedWeight: 0,
                    maxWeight: (setting.rankedMaskFirstRankWeight * setting.rankedMaskRateList[i]) || 0,
                    lastReceivedTime: 0,
                });
            }
        } else {
            item.receivedWeight += weight;
            item.lastReceivedTime = Date.now();
        }
    }
    await doc.save();
};

export default mongoose.model('RoadmapMask', schema);

```

* PDShopServer/project/App/models/RoadmapModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import { formatTime, getFormatTime, virtualObjectById, formatObjectById } from '../utils';

const schema = new Schema({
    // 基础信息
    shipperId: { type: Schema.Types.ObjectId, ref: 'Shipper' }, // 物流公司 Id
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 所属分店 Id
    shipperInBranchShopId: { type: Schema.Types.ObjectId, ref: 'ShipperInBranchShop' }, // 所属入驻分店模型分店 Id

    profitRate: { type: Number }, // 对路线的提成 = 方向.profitRate < 路线.profitRate

    sendDoorEnable: { type: Boolean, default: false }, // 送货上门是否全部设置（内部使用）
    enable: { type: Boolean, default: false }, // 是否可用
    endPoint: { type: String }, // 终点（精确到镇级别）
    endPointLastCode: { type: Number },  // 终点的最后一个code

    price: { type: Number, default: 0 }, // 每吨的价格，单位为元/吨 (差价必须为整数，相同的路线不能设置和别人相同的价格)
    basePrice: { type: Number, default: 0 }, // 价格保底价（为0的时候无效）
    updatePriceTime: { type: Date, default: Date.now }, // 修改价格的时间，如果价格相同，使用updateXXTime排序，谁先修改谁排前面
    minFee: { type: Number, default: 0 }, // 起步价，在按照重量来计算不足起步价的情况下按照起步价计算。
    baseMinFee: { type: Number, default: 0 }, // 起步价保底价（为0的时候无效）
    updateMinFeeTime: { type: Date, default: Date.now }, // 修改起步价的时间

    isCityDistribute: { type: Boolean, default: false }, // 是否是同城配送的路线（如果为false，表示长途路线，需要配置 sendDoorList， 否则不需要配置）
    sendDoorList: [{ // 送货上门的列表
        enable: { type: Boolean }, // 该同城配送是否可用
        sendDoorEndPoint: { type: String },  // 送货上门的终点
        sendDoorEndPointLastCode: { type: Number },  // 送货上门的终点的最后一个code
        sendDoorPrice: { type: Number }, // 送货上门价格（元/吨）
        baseSendDoorPrice: { type: Number }, // 送货上门价格保底价
        updateSendDoorPriceTime: { type: Date }, // 修改送货上门价格的时间
        sendDoorMinFee: { type: Number }, // 送货上门起步价，在按照重量来计算不足起步价的情况下按照起步价计算。
        baseSendDoorMinFee: { type: Number }, // 送货上门起步价保底价
        updateSendDoorMinFeeTime: { type: Date }, // 修改送货上门起步价的时间
    }],

    // 时长信息
    duration: { type: Number, default: 0 }, // 经历时长(h)(通过统计获得)
    selfSignRate: { type: Number, default: 0 }, // 收货人亲自签收率

    // 统计信息
    tradeTimes: { type: Number, default: 0 }, // 交易次数
    monthTradeTimes: { type: Number, default: 0 }, // 月交易次数

    remark: { type: String }, // 备注 (添加一些特殊说明)

    createTime: { type: Date, default: Date.now }, // 创建时间
    modifyTime: { type: Date }, // 修改时间
    setProfitRateTime: { type: Date }, //总部对路线设置提成的时间
});

schema.virtual('startPoint').get(function () {
    return this.shop ? this.shop.name : undefined;
});

virtualObjectById(schema, 'shipperId', 'shopId', 'shipperInBranchShopId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'createTime', 'modifyTime', 'setProfitRateTime', 'updatePriceTime', 'updateMinFeeTime');
        formatObjectById(ret, 'shipperId', 'shopId', 'shipperInBranchShopId');
        ret.sendDoorList && (ret.sendDoorList = ret.sendDoorList.map(item => {
            item.updateSendDoorPriceTime = getFormatTime(item.updateSendDoorPriceTime);
            item.updateSendDoorMinFeeTime = getFormatTime(item.updateSendDoorMinFeeTime);
            return item;
        }));

        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Roadmap', schema);

```

* PDShopServer/project/App/models/RoadmapRegionProfitRateModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import { formatTime, virtualObjectById, formatObjectById } from '../utils';

const schema = new Schema({
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 所属分店 Id， 如果为空，则为所有的分店
    region: { type: String, default: '' }, // 方向，如：北京（必须使用地图选择）
    regionLastCode: { type: Number, default: 0 },  // 方向的最后一个code
    profitRate: { type: Number, default: 0 }, // 对路线的提成 = 方向.profitRate < 路线.profitRate

    type: { type: Number, default: 0 }, // 类型，0：所有，1：长途，2：短途

    level: { type: Number, default: 0 },  // 内部使用，用来排序
    createTime: { type: Date, default: Date.now }, // 创建时间
    modifyTime: { type: Date }, //修改时间
});

virtualObjectById(schema, 'shopId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'createTime', 'modifyTime');
        formatObjectById(ret, 'shopId');

        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('RoadmapRegionProfitRate', schema);

```

* PDShopServer/project/App/models/SettingModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import { formatObjectById } from '../utils';

const schema = new Schema({
    sizeWeightRate: { type: Number, default: 3 }, // 方量和重量的计算比
    insuanceRate: { type: Number, default: 0.002 }, // 担保保险的费用（客户实际付的钱）= 担保保险额度（需要赔偿用户的钱）* insuanceRate
    insuanceMountRate: { type: Number, default: 5 }, // 保额相对于运费的多少倍，默认为10
    insuanceBaseValue: { type: Number, default: 10 }, // 保险的最低值（默认为最低10元）
    proxyChargeProfitRate: { type: Number, default: 0.01 }, // 手续费 = 代收货款 * proxyChargeProfitRate
    // 设置min必须从0开始，默认： [ {rate: 1.1, min: 0}, {rate: 1, min: 10}, {rate: 0.9, min: 20 } ] 如果物流公司设置的是100元/吨，表示0-10为110元/吨，10-20为100元/吨，20以上为90元/吨
    gradientPriceList: [ { rate: Number, min: Number } ], // 每吨的价格，单位为元/吨（采用阶梯价格）
    additionalDeliverTime: { type: Number, default: 240 }, // 物流公司没有让货主确认附加的交易成功时间，单位小时(h)
    noticeShipperStorageWeight: { type: Number, default: 5 }, // 没多少吨货物放置后通知物流公司
    bondAmountWeightRate: { type: Number, default: 10000 }, // 每吨货需要的保证金
    rankedMaskFirstRankWeight: { type: Number, default: 5 }, // 用来判断排名第一的物流公司的收购多少吨货被屏蔽的依据
    rankedMaskRateList: [{ type: Number }], //为了防止物流公司垄断，排在前3（数组的长度+1）名的需要轮流收货，第一名收货的量为rankedMaskFirstRankWeight*1，第二名为rankedMaskFirstRankWeight*0.8，第三名为rankedMaskFirstRankWeight*0.6。默认： [ 1, 0.8, 0.6 ]，屏蔽规则：见 RoadmapMaskModel
});

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        delete ret.id;
        formatObjectById(ret);
        ret.gradientPriceList && (ret.gradientPriceList = ret.gradientPriceList.map(item => {
            delete item._id;
            return item;
        }));
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Setting', schema);

```

* PDShopServer/project/App/models/ShipperInBranchShopModel.js

```js
import mongoose from 'mongoose';
import { formatTime, formatMedia, getMediaPath, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    shipperId: { type: Schema.Types.ObjectId, ref: 'Shipper' }, // 物流公司
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 所属分店

    // 担保公司
    bondCompanyList: [{ // 担保公司列表
        capital: { type: Number }, // 担保公司注册资金
        bondAmount: { type: Number }, // 担保金额
        name: { type: String }, // 担保公司名称
        address: { type: String }, // 担保公司地址
        phoneList: { type: String }, // 担保公司电话
        legalName: { type: String }, // 法人姓名
        legalPhone: { type: String }, // 法人电话
        certificate: [{ type: Schema.Types.ObjectId, ref: 'Media' }], // 担保公司资格证书
        legalIDCard: [{ type: Schema.Types.ObjectId, ref: 'Media' }], //法人身份证
    }],
    totalBondAmount: { type: Number, default: 0 }, // 总的担保金额
    remainBondAmount: { type: Number, default: 0 }, // 剩余可用的担保金额
    usedBondAmount: { type: Number, default: 0 }, // 已使用的担保金额

    // 上门自提的信息
    clientPickPrice: { type: Number }, // 自提价格 (只有 shipperType === 1 的物流公司需要设置)
    clientPickEnable: { type: Boolean, default: false }, // 是否可用 (默认为不可用，只有物流公司设置了价格后才生效)

    registerTime: { type: Date, default: Date.now }, //注册时间
});

virtualObjectById(schema, 'shopId', 'shipperId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'registerTime');
        formatMedia(ret, 'head');
        formatObjectById(ret, 'shopId', 'shipperId');
        ret.bondCompanyList && (ret.bondCompanyList = ret.bondCompanyList.map(item => {
            item.certificate = item.certificate.map(o => getMediaPath(o));
            item.legalIDCard = item.legalIDCard.map(o => getMediaPath(o));
            delete item._id;
            return item;
        }));
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('ShipperInBranchShop', schema);

```

* PDShopServer/project/App/models/ShipperModel.js

```js
import mongoose from 'mongoose';
import { formatMedia, formatTime, getMediaPath, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    chairManId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 董事长(法人)
    // 基础信息
    name: { type: String }, // 物流公司名称
    image: { type: String }, // 物流公司背景图片
    sign: { type: String }, // 物流公司签名
    address: { type: String }, // 商铺地址
    phoneList: { type: String }, // 物流公司电话列表

    capital: { type: Number, default: 0 }, // 注册资金
    acountAmount: { type: Number, default: 0 }, // 账户金额

    legalName: { type: String }, // 法人姓名
    legalPhone: { type: String }, // 法人电话
    legalIDCard: [{ type: Schema.Types.ObjectId, ref: 'Media' }], // 法人身份证

    // 物流公司类型
    shipperType: { type: Number, default: 0 }, // 0：长途物流公司 1：同城配送物流公司（默认为0）

    createTime: { type: Date, default: Date.now }, //创建时间
});

virtualObjectById(schema, 'chairManId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        ret.name = ret.name || ret.phone;
        formatMedia(ret, 'image');
        formatTime(ret, 'createTime');
        formatObjectById(ret, 'chairManId');
        ret.legalIDCard && (ret.legalIDCard = ret.legalIDCard.map(o => getMediaPath(o)));
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Shipper', schema);

```

* PDShopServer/project/App/models/ShopMemberModel.js

```js
import mongoose from 'mongoose';
import passportLocalMongoose from 'passport-local-mongoose';
import { formatTime, formatMedia, formatPhoneList, getAgeFromBirthday, virtualObjectById, formatObjectById } from '../utils';
import CONSTANTS from '../constants';
const Schema = mongoose.Schema;

const schema = new Schema({
    phone: { type: String, required: true, unique: true }, // 注册电话
    name: { type: String }, // 姓名
    head: { type: Schema.Types.ObjectId, ref: 'Media' }, // 用户头像
    sex: { type: Number, default: 0 }, // 性别 0:男   1:女
    birthday: { type: String }, // 生日
    address: { type: String }, // 住址
    email: { type: String }, // 邮箱
    salary: { type: Number, default: 0 }, // 工资
    post: { type: String, derault: CONSTANTS.PO_STAFF }, // 职位， 默认普通员工
    phoneList: { type: String }, // 预备电话列表

    isSetPaymentPassword: { type: Number, default: 0 }, // 性别 0:没设置   1:已设置

    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 所属店铺
    partmentId: { type: Schema.Types.ObjectId, ref: 'ShopPartment' }, // 所属部门 (这个部门指的是分店的专属部门)
    authority: [ { type: Number } ], // 拥有的权限

    registerTime: { type: Date, default: Date.now }, //注册时间
});

schema.plugin(passportLocalMongoose, { usernameField: 'phone' });

schema.virtual('age').get(function () {
    return getAgeFromBirthday(this.birthday);
});

virtualObjectById(schema, 'shopId', 'partmentId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'registerTime');
        formatMedia(ret, 'head');
        formatObjectById(ret, 'shopId', 'partmentId');
        formatPhoneList(ret);
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('ShopMember', schema);

```

* PDShopServer/project/App/models/ShopModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import { formatMedia, formatTime, virtualObjectById, formatObjectById } from '../utils';

const schema = new Schema({
    chairManId: { type: Schema.Types.ObjectId, ref: 'ShopMember' }, // 董事长(法人)
    // 基础信息
    name: { type: String }, // 物流超市名称
    image: { type: String }, // 物流超市背景图片
    sign: { type: String }, // 物流超市签名
    addressRegion: { type: String }, // 物流超市所在城市
    addressRegionLastCode: { type: Number }, // 物流超市所在城市的lastCode
    address: { type: String }, // 收货点地址
    location: { type: [ Number ], index: '2d' }, // 经纬度定位(必须使用地图来选择)
    phoneList: { type: String }, // 物流超市电话列表

    // 分类
    isMasterShop: { type: Boolean, default: false }, // 是否是主店

    // 提成信息
    profitRate: { type: Number, default: 0 }, // 该分店对路线的默认提成比例  分店的提成 = 总的提成 * profitRate，该参数由总部设置 (只对分店有效)

    createTime: { type: Date, default: Date.now }, //创建时间
});

virtualObjectById(schema, 'chairManId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        ret.name = ret.name || ret.phone;
        formatMedia(ret, 'image');
        formatTime(ret, 'createTime');
        formatObjectById(ret, 'chairManId');
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Shop', schema);

```

* PDShopServer/project/App/models/ShopPartmentModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import { formatTime, virtualObjectById, formatObjectById } from '../utils';

const schema = new Schema({
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 所属店铺
    chargeManId: { type: Schema.Types.ObjectId, ref: 'ShopMember' }, // 负责人

    name: { type: String }, // 部门名称
    type: { type: Number }, // 部门类型(见constants.partment)
    descript: { type: String }, // 部门描述
    phoneList: { type: String }, // 联系电话
    memberCount: { type: Number, default: 1 }, // 部门成员数量

    price: { type: Number }, // 价格 (如果为搬运部，为搬运每吨的价格，如果为货运部，为同城配送每吨的价格)
    enable: { type: Boolean, default: false }, // 是否可用 (默认为不可用，只有部门设置了价格后才生效)
    workTruckId: { type: Schema.Types.ObjectId, ref: 'Truck' }, // （搬运队再在上车的货车Id, 该字段只对搬运队有效）

    createTime: { type: Date, default: Date.now }, // 创建时间
    modifyTime: { type: Date }, //修改时间
});

virtualObjectById(schema, 'chargeManId', 'workTruckId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'createTime', 'modifyTime');
        formatObjectById(ret, 'chargeManId', 'workTruckId');
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('ShopPartment', schema);

```

* PDShopServer/project/App/models/StartPointAddressModel.js

```js
import mongoose from 'mongoose';
const Schema = mongoose.Schema;

const schema = new Schema({
    code: { type: Number }, // 地址code
    parentCode: { type: Number }, // 父级地址code
    name: { type: String }, // 名称
    level: { type: Number }, // 级别
});

const transform = {
    virtuals: false,
    transform: function (doc, ret, options) {
        delete ret.__v;
        delete ret._id;
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('StartPointAddress', schema);

```

* PDShopServer/project/App/models/TruckModel.js

```js
import mongoose from 'mongoose';
import { formatTime, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    shipperId: { type: Schema.Types.ObjectId, ref: 'Shipper' }, // 所属物流公司 Id
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 所属分店 Id（物流公司创建货车时需要制定为哪家分店）
    driverId: { type: Schema.Types.ObjectId, ref: 'Driver' }, // 对应的司机
    carryPartmentId: { type: Schema.Types.ObjectId, ref: 'ShopPartment' }, // 选择的搬运部
    scannerId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 扫描员的 Id

    carryPrice: { type: Number, default: 0 }, // 搬运价格

    plateNo: { type: String }, // 车牌号码
    drivingLicense: { type: String }, // 行驶证编号
    truckType: { type: String }, // 车型
    insuanceMount: { type: Number, default: 0 }, // 保险
    locationList: [ { address: String, longitude: Number, latitude: Number, time: Date } ], // 定位列表

    state: { type: Number, default: 0 }, // 申请发卡的状态， 0：待审核，1：待入库，2：待找搬运队，3：待扫描装车，4：装车中，5：待付搬运部的款，6：待出库，7：运输中，8：完成

    // 货单
    orderList: [ { type: Schema.Types.ObjectId, ref: 'Order' } ], // 货单列表
    orderCount: { type: Number, default: 0 }, // 总单数
    totalNumbers: { type: Number, default: 0 }, // 总件数
    totalWeight: { type: Number, default: 0 }, // 总重量
    totalSize: { type: Number, default: 0 }, // 总方量
    unloadAllOrderList: [ { type: Schema.Types.ObjectId, ref: 'Order' } ], // 只上了一部分货单列表

    createTime: { type: Date, default: Date.now }, // 创建时间
    modifyTime: { type: Date }, // 修改时间
});

virtualObjectById(schema, 'driverId', 'shopId', 'shipperId', 'carryPartmentId', 'scannerId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'createTime', 'modifyTime');
        formatObjectById(ret, 'driverId', 'shopId', 'shipperId', 'carryPartmentId', 'scannerId');
        ret.locationList && (ret.locationList = ret.locationList.map(item => { formatTime(item, 'time'); delete item._id; return item; }));
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Truck', schema);

```

* PDShopServer/project/App/models/VerifyCodeModel.js

```js
import mongoose from 'mongoose';
import { virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    phone: { type: String, required: true, unique: true }, // 注册电话
    codeList: [{ code: String, deadTime: Date }], //验证码的列表
});

virtualObjectById(schema);

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatObjectById(ret);
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('VerifyCode', schema);

```

* PDShopServer/project/App/models/VersionModel.js

```js
import mongoose from 'mongoose';
import { formatTime, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    platform: { type: String }, // platform类型， android, ios
    versionName: { type: String }, // 版本号
    versionCode: { type: String }, // 版本 code
    minVersion: { type: Number }, // 小版本号（js版本号）
    passed: [{ type: String }], // 通过审核的渠道，ios只有default(代表AppStore), android有default和其他渠道名
    description: [{ type: String }], // 版本描述
    time: { type: Date, default: Date.now }, //发布时间
});

schema.virtual('verName').get(function () {
    return this.versionName + '.' + this.minVersion;
});

virtualObjectById(schema);

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatObjectById(ret);
        formatTime('time');
        delete ret.platform;
        delete ret.versionName;
        delete ret.minVersion;
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Version', schema);

```

* PDShopServer/project/App/models/WarehouseModel.js

```js
import mongoose from 'mongoose';
import { formatTime, virtualObjectById, formatObjectById } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    shopId: { type: Schema.Types.ObjectId, ref: 'Shop' }, // 所属分店 Id（物流公司创建货车时需要制定为哪家分店）
    shipperList: [ { type: Schema.Types.ObjectId, ref: 'Shipper' } ], // 所属物流公司 Id

    // 基本信息
    houseManId: { type: Schema.Types.ObjectId, ref: 'ShopMember' }, // 对应的库管员
    houseNo: { type: String }, // 仓库编号
    maxStoreWeight: { type: Number, default: 0 }, // 可以容纳的总重量
    maxStoreSize: { type: Number, default: 0 }, // 可以容纳的总方量

    // 已经入库的信息
    orderCount: { type: Number, default: 0 }, // 已入库的总单数
    totalNumbers: { type: Number, default: 0 }, // 已入库的总件数
    totalWeight: { type: Number, default: 0 }, // 已入库的总重量
    totalSize: { type: Number, default: 0 }, // 已入库的总方量

    createTime: { type: Date, default: Date.now }, // 创建时间
    modifyTime: { type: Date }, // 修改时间
});

virtualObjectById(schema, 'houseManId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'createTime', 'modifyTime');
        formatObjectById(ret, 'houseManId');
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('Warehouse', schema);

```

* PDShopServer/project/App/models/WeixinPayModel.js

```js
import mongoose from 'mongoose';
import { virtualObjectById, formatObjectById, formatTime } from '../utils';
const Schema = mongoose.Schema;

const schema = new Schema({
    // 用户信息
    clientId: { type: Schema.Types.ObjectId, ref: 'Client' }, // 创建货单的人的 Id （用户端）
    shopMemberId: { type: Schema.Types.ObjectId, ref: 'ShopMember' }, // 创建货单的人的 Id （店面端）
    spbill_create_ip: { type: String }, // 支付的人的ip
    openid: { type: String }, // 支付的人的微信 openid

    // 金额信息
    body: { type: String }, // 商品描述
    trade_type: { type: String }, // 交易类型
    total_fee: { type: Number }, // 总金额

    // 回包信息
    prepay_id: { type: String }, // 预支付交易会话 Id
    appid: { type: String }, // App Id
    mch_id: { type: String }, // 商户 Id
    nonce_str: { type: String }, // 随机字符串
    package: { type: String }, // 扩展字段
    timestamp: { type: Number }, // 时间戳

    // 支付信息
    bank_type: { type: String }, // 付款银行
    cash_fee: { type: String }, // 现金支付金额
    transaction_id: { type: String }, // 微信支付货单号
    code_url: { type: String }, // 微信支付二维码（扫码支付）

    // 状态
    state: { type: Number, default: 0 }, // 未支付
    payedTime: { type: Date, default: Date.now }, // 支付完成时间，使用通知接口的 time_end 赋值
    createTime: { type: Date, default: Date.now }, //创建时间
});

schema.virtual('out_trade_no').get(function () {
    return this._id;
});
virtualObjectById(schema, 'clientId');

const transform = {
    virtuals: true,
    transform: function (doc, ret, options) {
        formatTime(ret, 'createTime', 'payedTime');
        formatObjectById(ret, 'clientId');
        delete ret.id;
        return ret;
    },
};
schema.options.toJSON = transform;
schema.options.toObject = transform;

export default mongoose.model('WeixinPay', schema);

```

* PDShopServer/project/App/models/index.js

```js
export AdminModel from './AdminModel';
export AgentModel from './AgentModel';
export ClientModel from './ClientModel';
export CollectionModel from './CollectionModel';
export CommentModel from './CommentModel';
export FeedbackModel from './FeedbackModel';
export MediaModel from './MediaModel';
export NotifyModel from './NotifyModel';
export OrderModel from './OrderModel';
export OrderGroupModel from './OrderGroupModel';
export RoadmapModel from './RoadmapModel';
export RoadmapMaskModel from './RoadmapMaskModel';
export RoadmapRegionProfitRateModel from './RoadmapRegionProfitRateModel';
export RoadmapAddressModel from './RoadmapAddressModel';
export ShipperModel from './ShipperModel';
export ShipperInBranchShopModel from './ShipperInBranchShopModel';
export ShopModel from './ShopModel';
export ShopMemberModel from './ShopMemberModel';
export ShopPartmentModel from './ShopPartmentModel';
export TruckModel from './TruckModel';
export CityTruckModel from './CityTruckModel';
export VersionModel from './VersionModel';
export SettingModel from './SettingModel';
export StartPointAddressModel from './StartPointAddressModel';
export LogModel from './LogModel';
export AgentRegionProfitModel from './AgentRegionProfitModel';
export DriverModel from './DriverModel';
export PaymentModel from './PaymentModel';
export VerifyCodeModel from './VerifyCodeModel';
export WeixinPayModel from './WeixinPayModel';
export WarehouseModel from './WarehouseModel';

```

* PDShopServer/project/App/mysql/AccountModel.js

```js
import Sequelize from 'sequelize';
import sequelize from './sequelize';
import CONSTANTS from '../constants';
import { _Y } from '../utils';

const model = sequelize.define('Account', {
    targetId: { type: Sequelize.STRING(24) }, // mongodb中的对应的 分店的Id，分店的部门的Id，客户的Id
    amount: { type: Sequelize.BIGINT, defaultValue: 0 }, // 账户总额，单位(分)
    type: Sequelize.INTEGER, //账户类型，AT_BASIC：总部, AT_BRANCH_SHOP： 分店， AT_BRANCH_SHOP_PARTMENT： 分店的部门， AT_CLIENT：客户账号 (详情见constans/account)
}, {
    getterMethods: {
        amountYuan: function () { return _Y(this.amount); },
    },
    timestamps: false,
    initialAutoIncrement: CONSTANTS.AC_BANK_ACCOUNT, //10000为总部账号
});

export default model;

```

* PDShopServer/project/App/mysql/BillModel.js

```js
import Sequelize from 'sequelize';
import sequelize from './sequelize';
import { _Y, getFormatTime } from '../utils';

// 如果 thirdpartyAccount 不为空， 说明是充值和提现的账单，否则是内部账单，内部账单必须针对于某个货单 orderId
const model = sequelize.define('Bill', {
    account: Sequelize.INTEGER, // 对应的账户Id
    tradeAmount: Sequelize.BIGINT, // 交易金额 (单位分) (大于0表示进账，小于0表示出账)
    thirdpartyAccount: Sequelize.STRING, // 第三方的货单号  0：支付宝(A)， 1：财付通(C)，2：工商银行(G)，3：建设银行(J)， 4：中国银行（Z）， 5：农业银行(N)，规则：缩写字母#第三方的货单号#第三方账号
    orderId: Sequelize.STRING(24), // 交易对应的货单号
    truckId: Sequelize.STRING(24), // 交易对应的货车号
    remark: Sequelize.STRING, // 备注
    tradeTime: { // 注册时间
        type: Sequelize.DATE,
        defaultValue: Sequelize.NOW,
        get () {
            return getFormatTime(this.getDataValue('tradeTime'));
        },
    },
}, {
    getterMethods: {
        tradeAmountYuan: function () { return _Y(this.tradeAmount); },
    },
    timestamps: false,
});

export default model;

```

* PDShopServer/project/App/mysql/OrderPaymentModel.js

```js
import Sequelize from 'sequelize';
import sequelize from './sequelize';
import { getFormatTime } from '../utils';

const model = sequelize.define('OrderPayment', {
    orderId: { type: Sequelize.STRING(24), primaryKey: true, unique: true },
    isPayed: { type: Sequelize.BOOLEAN, defaultValue: false }, // 是否已经付款（如果是现付，只是否已经付运费+保险，如果是到付，是否扣除物流公司和支付运费）
    isRebated: { type: Sequelize.BOOLEAN, defaultValue: false }, // 是否返利，(交易成功后所有的资金返回)
    payedTime: { // 付款时间
        type: Sequelize.DATE,
        get () {
            return getFormatTime(this.getDataValue('payedTime'));
        },
    },
    rebatedTime: { // 返利时间
        type: Sequelize.DATE,
        get () {
            return getFormatTime(this.getDataValue('rebatedTime'));
        },
    },
}, {
    timestamps: false,
});

export default model;

```

* PDShopServer/project/App/mysql/RegionModel.js

```js
import Sequelize from 'sequelize';
import sequelize from './sequelize';
// 内部交易
const model = sequelize.define('Region', {
    code: Sequelize.INTEGER(10), // 对应的代码
    parentCode: Sequelize.INTEGER(8), // 父级代码
    name: Sequelize.STRING(50), // 名称
    level: Sequelize.STRING, //级别 (1,2,3,4)
}, {
    getterMethods: {
        isLeaf: function () {
            return (
                this.level === 0 ||
                this.level === 4 ||
                this.level === 5 ||
                this.level === 6 ||
                this.level === 7 ||
                this.level === 10
            );
        },
    },
    timestamps: false,
});

export default model;

```

* PDShopServer/project/App/mysql/TruckPaymentModel.js

```js
import Sequelize from 'sequelize';
import sequelize from './sequelize';
import { getFormatTime } from '../utils';

const model = sequelize.define('TruckPayment', {
    truckId: { type: Sequelize.STRING(24), primaryKey: true, unique: true },
    isPayedForCarryPartment: { type: Sequelize.BOOLEAN, defaultValue: false }, // 是否已经给搬运部付钱
    payedForCarryPartmentTime: { // 付款时间
        type: Sequelize.DATE,
        get () {
            return getFormatTime(this.getDataValue('payedForCarryPartmentTime'));
        },
    },
}, {
    timestamps: false,
});

export default model;

```

* PDShopServer/project/App/mysql/index.js

```js
export sequelize from './sequelize';
export AccountModel from './AccountModel';
export BillModel from './BillModel';
export OrderPaymentModel from './OrderPaymentModel';
export TruckPaymentModel from './TruckPaymentModel';
export RegionModel from './RegionModel';

```

* PDShopServer/project/App/mysql/sequelize.js

```js
import Sequelize from 'sequelize';
import config from '../../config';

export default new Sequelize(config.mysql.database, config.mysql.username, config.mysql.password, config.mysql);

```

* PDShopServer/project/App/routers/index.js

```js
import { Router } from 'express';
import posts from './posts';
import gets from './gets';
import _ from 'lodash';
import { timeout, apiRoot } from '../../config';

const router = new Router();
const LOG_FILTERS = [
    'getRegionAddress',
    'getRegionAddressFromLastCode',
    'getSendDoorRegionAddressFromLastCode',
    'getStartPointAddress',
    'getStartPoint',
    'getRegionAddressWithOrder',
    'getRegionAddressWithRegion',
];

async function DEBUG (func, ...params) {
    try {
        return await func(...params);
    } catch (e) {
        console.log(e);
    }
}

function registerPostRouter (root, posts) {
    _.map(posts, (func, api) => {
        const url = root + '/' + api;
        if (typeof func === 'object') {
            registerPostRouter(url, func);
        } else {
            // console.log('register:', url);
            router.post(url, async (req, res) => {
                if (process.env.NODE_ENV === 'production') {
                    console.log(url + ' recv:', req.body, req.file || '');
                    let result = await func(req.body, req);
                    _.includes(LOG_FILTERS, api) || console.log(url + ' send:', JSON.stringify(result, null, 2));
                    if (!result || typeof result === 'number') {
                        res.sendStatus(result || 500);
                    } else {
                        res.send(result);
                    }
                } else {
                    console.log(url + ' recv:', req.body, req.file || '');
                    let result = await DEBUG(func, req.body, req);
                    _.includes(LOG_FILTERS, api) || console.log(url + ' send:', JSON.stringify(result, null, 2));
                    setTimeout(() => {
                        if (!result || typeof result === 'number') {
                            res.sendStatus(result || 500);
                        } else {
                            res.send(result);
                        }
                    }, timeout);
                }
            });
        }
    });
}

function registerGetRouter (root, gets) {
    _.map(gets, (func, api) => {
        const url = root + '/' + api;
        if (typeof func === 'object') {
            registerGetRouter(url, func);
        } else {
            // console.log('register:', url);
            router.get(url, async (req, res) => {
                if (process.env.NODE_ENV === 'production') {
                    console.log(url + ' recv:', req.query);
                    let result = await func(req.query, req);
                    _.includes(LOG_FILTERS, api) || console.log(url + ' send:', result.readable ? 'image' : result);
                    if (!result || typeof result === 'number') {
                        res.sendStatus(result || 500);
                    } else if (result.readable) {
                        result.pipe(res);
                    } else {
                        res.send(result);
                    }
                } else {
                    console.log(url + ' recv:', req.query);
                    let result = await DEBUG(func, req.query, req);
                    _.includes(LOG_FILTERS, api) || console.log(url + ' send:', result.readable ? 'image' : result);
                    setTimeout(() => {
                        if (!result || typeof result === 'number') {
                            res.sendStatus(result || 500);
                        } else if (result.readable) {
                            result.pipe(res);
                        } else {
                            res.send(result);
                        }
                    }, timeout);
                }
            });
        }
    });
}

registerPostRouter(apiRoot, posts);
registerGetRouter(apiRoot, gets);

export default router;

```

* PDShopServer/project/App/utils/account.js

```js
import _ from 'lodash';
import mongoose from 'mongoose';
import { ShopMemberModel, ClientModel, PaymentModel, VerifyCodeModel } from '../models';
import config from '../../config';

export function registerUser (model, user, password) {
    return new Promise(resolve => {
        model.register(user, password, error => {
            if (error) {
                return resolve(error.name);
            }
            return resolve();
        });
    });
}
export function setPassword (user, password) {
    return new Promise(resolve => {
        user.setPassword(password, (err, thisModel, passwordErr) => {
            if (passwordErr) {
                return resolve(passwordErr);
            }
            return resolve();
        });
    });
}
export function checkVerifyCode (phone, verifyCode) {
    if (config.hasNoVerifyCode) {
        return true;
    }
    return new Promise(async resolve => {
        const vc = await VerifyCodeModel.findOne({ phone });
        if (!vc) {
            return resolve();
        }
        const codeList = vc.codeList;
        const now = Date.now();
        if (!_.find(codeList, o => o.code === verifyCode && o.deadTime.getTime() > now)) {
            return resolve();
        }
        vc.codeList = [];
        await vc.save();
        resolve(true);
    });
}
export function authenticatePassword (user, password) {
    return new Promise(resolve => {
        user.authenticate(password, (err, thisModel, passwordErr) => {
            if (passwordErr) {
                return resolve(passwordErr);
            }
            return resolve();
        });
    });
}
export function authenticatePaymentPassword (userId, password) {
    return new Promise(async resolve => {
        const payment = await PaymentModel.findOne({ userId });
        if (!payment) {
            return resolve(true);
        }
        payment.authenticate(password, (err, thisModel, passwordErr) => {
            if (passwordErr) {
                return resolve(passwordErr);
            }
            return resolve();
        });
    });
}
export function getDefaultPassWord (phone, password) {
    return password || phone.substr(-6, 6);
}
export function genRandomPassword () {
    let pwd = '';
    let ranges = [[48, 57], [65, 90], [97, 122]];
    for (let i = 0; i < 6; i++) {
        let range = ranges[_.random(0, 2)];
        pwd += String.fromCharCode(_.random(range[0], range[1]));
    }
    return pwd;
}
export function getOwnShop (userId) {
    return new Promise(async resolve => {
        const member = await ShopMemberModel.findById(userId);
        return resolve(member && member.shopId);
    });
}
export function getOwnShipper (userId) {
    return new Promise(async resolve => {
        const member = await ClientModel.findById(userId);
        return resolve(member && member.shipperId);
    });
}
export function getPartmentIdByMember (userId) {
    return new Promise(async resolve => {
        const member = await ShopMemberModel.findById(userId);
        return resolve(member ? member.partmentId + '' : '');
    });
}
export function hasAuthority (member, authority, isClient) {
    authority = _.isArray(authority) ? authority : [authority];
    if (mongoose.Types.ObjectId.isValid(member)) {
        return new Promise(async resolve => {
            const model = isClient ? ClientModel : ShopMemberModel;
            const _member = await model.findById(member);
            return resolve(_member && !!_.intersection(_member.authority, authority).length);
        });
    } else {
        return member && !!_.intersection(member.authority, authority).length;
    }
}
export function checkAuthorizationCode (authorizationCode) {
    return authorizationCode === '__smt_8_8_6_2_2_8_0_3__';
}

```

* PDShopServer/project/App/utils/check.js

```js
import _ from 'lodash';

function _R (keyword) {
    return new RegExp('.*' + (keyword || '') + '.*', 'gim');
}

export function getKeywordCriteriaForShipper (keyword, criteria = {}) {
    if (keyword) {
        const regex = _R(keyword);
        criteria = { ...criteria, $or: [{ name: regex }, { phone: regex }] };
    }
    return _.omitBy(criteria, _.isNil);
}
export function getKeywordCriteriaForRoadmap (keyword, criteria = {}) {
    if (keyword) {
        criteria = { ...criteria, endPoint: _R(keyword) };
    }
    return _.omitBy(criteria, _.isNil);
}
export function getKeywordCriteriaForOrder (keyword, criteria = {}) {
    if (keyword) {
        const regex = _R(keyword);
        criteria = { ...criteria, endPoint: regex };
    }
    return _.omitBy(criteria, _.isNil);
}
export function getKeywordCriteriaForUser (keyword, criteria = {}) {
    if (keyword) {
        const regex = _R(keyword);
        criteria = { ...criteria, $or: [{ name: regex }, { phone: regex }, { email: regex }] };
    }
    return _.omitBy(criteria, _.isNil);
}
export function getKeywordCriteriaForTruck (keyword, criteria = {}) {
    if (keyword) {
        criteria = { ...criteria, plateNo: _R(keyword) };
    }
    return _.omitBy(criteria, _.isNil);
}
export function getKeywordCriteriaForPartment (keyword, criteria = {}) {
    if (keyword) {
        const regex = _R(keyword);
        criteria = { ...criteria, $or: [{ name: regex }, { plateNo: regex }] };
    }
    return _.omitBy(criteria, _.isNil);
}
export function getKeywordCriteriaForWarehouse (keyword, criteria = {}) {
    if (keyword) {
        const regex = _R(keyword);
        criteria = { ...criteria, houseNo: regex };
    }
    return _.omitBy(criteria, _.isNil);
}
export function getKeywordCriteriaForRoadmapRegion (keyword, criteria = {}) {
    if (keyword) {
        const regex = _R(keyword);
        criteria = { ...criteria, region: regex };
    }
    return _.omitBy(criteria, _.isNil);
}

```

* PDShopServer/project/App/utils/common.js

```js
import _ from 'lodash';
import moment from 'moment';

export function _T (id) {
    return String(id);
}
export function _F (yuan) {
    return Math.floor(yuan * 100);
}
export function _Y (fen) {
    return fen / 100;
}
export function _N (num, rank = 2) {
    return parseFloat(num.toFixed(rank));
}
export function omitNil (obj) {
    return _.omitBy(obj, _.isNil);
}
export function testPhone (phone) {
    return /^1\d{10}$/.test(phone);
}
export function getAgeFromBirthday (birthday) {
    if (!birthday) {
        return undefined;
    }
    return Math.floor(moment().diff(moment(birthday)) / 31536000000);
}

```

* PDShopServer/project/App/utils/email.js

```js
import nodemailer from 'nodemailer';

export function sendFindPasswordMail (address, password) {
    const name = '刷我的卡';
    const serverName = '刷我的卡官网';
    const serverUrl = 'mailto:cocos@chukong-inc.com';
    const transporter = nodemailer.createTransport({
        host: 'smtp.163.com',   // 支持列表 https://github.com/andris9/nodemailer-wellknown#supported-services
        port: 25, // SMTP 端口
        secureConnection: true, // 使用 SSL
        auth: {
            user: 'use_myself_card@163.com',
            pass: 'card123456',
        },
    });
    const options = {
        from: 'use_myself_card@163.com', // 发件地址
        to: address, // 收件列表
        subject: `【${name}】请在30分钟内重设您的密码`, // 标题
        html: `
        <div style="border-top:1px solid #808080;margin:0px 0 10px;padding:20px 0">
                <p style="font-size:14px;font-weight:bold;font-family:'Helvetica Neue',Helvetica,STheiti,Arial,Tahoma,微软雅黑,sans-serif,serif;color:#222;padding:0;margin:0">${address}</span>，感谢您使用刷我的卡帐号</p>
                <div style="font-size:14px;font-family:'Helvetica Neue',Helvetica,STheiti,Arial,Tahoma,微软雅黑,sans-serif,serif;color:#464646;padding:0;margin:20px 0 5px;line-height:1.6">您正在申请重新设置您的${name}帐号密码，请确认是本人操作</div>
                <div style="font-size:14px;font-family:'Helvetica Neue',Helvetica,STheiti,Arial,Tahoma,微软雅黑,sans-serif,serif;color:#464646;padding:0;margin:20px 0 5px;line-height:1.6">你的重置密码为：${password}，请在30分钟内在${name}App中修改密码。</div>
                <p style="font-size:12px;font-family:'Helvetica Neue',Helvetica,STheiti,Arial,Tahoma,微软雅黑,sans-serif,serif;color:#666;padding:0;margin:0;line-height:1.6">如您有任何问题，请联系：<a href="${serverUrl}" target="_blank">${serverName}</a></p>
                <p style="font-size:12px;font-family:'Helvetica Neue',Helvetica,STheiti,Arial,Tahoma,微软雅黑,sans-serif,serif;color:#666;padding:0;margin:0;line-height:1.6">${name}帐号系统</p>
        </div>
        `, //内容
    };
    return new Promise(async resolve => {
        transporter.sendMail(options, (error, info) => {
            console.log(error, info);
            if (error) {
                return resolve(false);
            }
            return resolve(true);
        });
    });
}

```

* PDShopServer/project/App/utils/format.js

```js
import mongoose from 'mongoose';
import moment from 'moment';
import _ from 'lodash';
import { host, port, apiRoot } from '../../config';

export function getMediaPath (id) {
    const imgPort = process.env.NODE_ENV === 'production' ? 80 : port;
    return id ? 'http://' + host + ':' + imgPort + apiRoot + '/image?id=' + id : undefined;
}
export function getMediaId (url) {
    return url ? url.replace(/.*id=/, '') : undefined;
}
export function formatMedia (obj, ...keys) {
    for (var key of keys) {
        obj[key] = getMediaPath(obj[key]);
    }
}
export function getFormatTime (time) {
    return time ? moment(time).format('YYYY-MM-DD HH:mm:ss') : undefined;
}
export function formatTime (obj, ...keys) {
    for (var key of keys) {
        obj[key] && (obj[key] = moment(obj[key]).format('YYYY-MM-DD HH:mm:ss'));
    }
}
export function formatPhoneList (obj) {
    let phoneList = [];
    if (obj.phoneList) {
        phoneList = _.without(obj.phoneList.split(/;|；/), obj.phone);
    }
    obj.phoneList = _.uniq(phoneList).join(';') || undefined;
}
export function virtualObjectById (schema, ...keys) {
    schema.virtual('id').get(function () {
        return this._id;
    });
    _.forEach(keys, (key) => {
        schema.virtual(key.replace(/Id$/, '')).get(function () {
            return mongoose.Types.ObjectId.isValid(this[key]) ? undefined : this[key];
        });
    });
}
export function formatObjectById (obj, ...keys) {
    delete obj.__v;
    delete obj._id;
    for (var key of keys) {
        obj[key.replace(/Id$/, '')] && delete obj[key];
    }
}

```

* PDShopServer/project/App/utils/index.js

```js
export * from './account';
export * from './check';
export * from './common';
export * from './email';
export * from './format';
export * from './sms';
export * from './log';

```

* PDShopServer/project/App/utils/log.js

```js
import { LogModel } from '../models';

export function actionLog (operatorId, operatorModel, targetId, targetModel, action, remark) {
    const log = new LogModel({
        operator: { id: operatorId, model: operatorModel },
        target: { id: targetId, model: targetModel },
        action,
        remark,
    });
    log.save();
}
export function actionLogWithList (operatorId, operatorModel, targetList, action, remark) {
    const log = new LogModel({
        operator: { id: operatorId, model: operatorModel },
        targetList: targetList,
        action,
        remark,
    });
    log.save();
}

```

* PDShopServer/project/App/utils/sms.js

```js
import request from 'superagent';

export function sendSms (phone, code) {
    return new Promise(async resolve => {
        request.get('http://v.juhe.cn/sms/send')
        .query({
            mobile: phone,
            tpl_id: 45242,
            tpl_value: `#code#=${code}`,
            key: '61234234f2760791609002be4d14508f',
        })
        .end((error, res) => {
            if (error) {
                return resolve(false);
            } else {
                const { error_code } = res.body;
                if (error_code === 0) {
                    return resolve(true);
                } else {
                    return resolve(false);
                }
            }
        });
    });
}

```

* PDShopServer/project/App/routers/gets/index.js

```js
import * as common from './common';
import * as test from './test';

export default {
    ...common,
    ...test,
};

```

* PDShopServer/project/App/routers/posts/index.js

```js
import * as common from './common';
import * as admin from './admin';
import * as client from './client';
import * as shop from './shop';
import * as shipper from './shipper';
import * as agent from './agent';
import * as driver from './driver';

export default {
    ...common,
    admin,
    client,
    shipper,
    shop,
    agent,
    driver,
};

```

* PDShopServer/project/App/routers/gets/common/audio.js

```js
import mongoose from 'mongoose';
import { MediaModel } from '../../../models';

export default ({
    id,
}) => {
    return new Promise(async resolve => {
        const { gfs } = mongoose;
        let doc = await MediaModel.findById(id);
        if (!doc) {
            return resolve(404);
        } else {
            return resolve(gfs.createReadStream({ _id: doc.gridId }));
        }
    });
};

```

* PDShopServer/project/App/routers/gets/common/image.js

```js
import mongoose from 'mongoose';
import { MediaModel } from '../../../models';

export default ({
    id,
}) => {
    return new Promise(async resolve => {
        const { gfs } = mongoose;
        let doc = await MediaModel.findById(id);
        if (!doc) {
            return resolve(404);
        } else {
            return resolve(gfs.createReadStream({ _id: doc.gridId }));
        }
    });
};

```

* PDShopServer/project/App/routers/gets/common/index.js

```js
export image from './image';
export audio from './audio';

```

* PDShopServer/project/App/routers/posts/admin/index.js

```js
// common
export login from './personal/login'; // 登录
export register from './personal/register'; // 注册
export findPassword from './personal/findPassword'; // 找回密码
export modifyPassword from './personal/modifyPassword'; // 修改密码
export removeUnusedMedia from './personal/removeUnusedMedia'; // 删除无用的media文件

// version
export addVersion from './version/addVersion'; // 发布版本
export getVersionList from './version/getVersionList'; // 获取发布版本的列表

// feedback
export getFeedbackList from './feedback/getFeedbackList'; // 获取反馈意见的列表
export acceptDeelFeedback from './feedback/acceptDeelFeedback'; // 开始反馈意见的处理
export finishDeelFeedback from './feedback/finishDeelFeedback'; // 完成反馈意见的处理

// shop
export createMasterShop from './shop/createMasterShop'; // 创建总部和董事长账号

// authority
export resetMasterChairManAuthority from './authority/resetMasterChairManAuthority'; // 重置总部董事长的权限
export resetBranchChairManAuthority from './authority/resetBranchChairManAuthority'; // 重置分店董事长的权限
export resetPartmentChargeManAuthority from './authority/resetPartmentChargeManAuthority'; // 重置部门负责人的权限
export resetShipperChairManAuthority from './authority/resetShipperChairManAuthority'; // 重置部门物流公司董事长的权限
export modifyMemberAuthority from './authority/modifyMemberAuthority'; // 修改成员的权限
export modifyClientAuthority from './authority/modifyClientAuthority'; // 修改客户的权限（物流公司 和 收货点）

// test
export test from './test/test'; // 测试接口

```

* PDShopServer/project/App/routers/gets/test/getVerifyCode.js

```js
import { VerifyCodeModel } from '../../../models';

export default async ({
    phone,
}) => {
    let doc = await VerifyCodeModel.findOne({ phone });
    if (!doc) {
        return { msg: '没有验证码' };
    }
    return { code: (doc.codeList[0] || {}).code };
};

```

* PDShopServer/project/App/routers/gets/test/index.js

```js
export getVerifyCode from './getVerifyCode';

```

* PDShopServer/project/App/routers/posts/agent/index.js

```js
// member
export modifyMemberAuthority from './member/modifyMemberAuthority'; // 修改成员的权限
export getMemberList from './member/getMemberList'; // 获取成员列表
export getMemberDetail from './member/getMemberDetail'; // 获取成员详情
export getMemberByPhone from './member/getMemberByPhone'; // 通过电话获取成员详情

// regionProfit
export addRegionProfit from './regionProfit/addRegionProfit'; // 添加区域的路线提成
export removeRegionProfit from './regionProfit/removeRegionProfit'; // 删除区域的路线提成
export modifyRegionProfit from './regionProfit/modifyRegionProfit'; // 修改区域的路线提成
export getRegionProfitDetail from './regionProfit/getRegionProfitDetail'; // 获取区域的路线提成详情
export getRegionProfitList from './regionProfit/getRegionProfitList'; // 获取区域的路线提成列表
export setRegionProfitWithList from './regionProfit/setRegionProfitWithList'; // 批量设置提成
export getRegionAddressWithRegion from './regionProfit/getRegionAddressWithRegion'; // 通过region获取地址列表

// referShop
export setReferShop from './referShop/setReferShop'; // 设置关联店铺
export getReferShopList from './referShop/getReferShopList'; // 获取关联店铺的列表

// order
export placeOrder from './order/placeOrder'; // 下单
export modifyOrder from './order/modifyOrder'; // 修改货单
export placeOrderWithPreOrder from './order/placeOrderWithPreOrder'; // 根据预下单下初始单
export getOrders from './order/getOrders'; // 获取货单列表
export getOrderDetail from './order/getOrderDetail'; // 获取货单详情
export getLastestOrder from './order/getLastestOrder'; // 获取待打印货单
export printBarCode from './order/printBarCode'; // 打印二维码
export confirmCachPayed from './order/confirmCachPayed'; // 确认现金支付
export confirmCachPayedForOrderList from './order/confirmCachPayedForOrderList'; // 确认现金支付多个货单
export printOrderBill from './order/printOrderBill'; // 打印货单
export printOrderListBill from './order/printOrderListBill'; // 打印货单列表的货单
export getPreOrderList from './order/getPreOrderList'; // 获取预下单列表
export getRegionAddressWithOrder from './order/getRegionAddressWithOrder'; // 通过货单获取地址
export getRegionSendDoorAddressWithOrder from './order/getRegionSendDoorAddressWithOrder'; // 通过货单获取送货上门地址
export removeOrder from './order/removeOrder'; // 删除货单

```

* PDShopServer/project/App/routers/posts/client/index.js

```js
// common
export login from './personal/login';// 登录
export register from './personal/register';// 注册
export findPassword from './personal/findPassword';// 找回密码
export modifyPassword from './personal/modifyPassword';// 修改密码
export getPersonalInfo from './personal/getPersonalInfo';// 获取个人信息
export modifyPersonalInfo from './personal/modifyPersonalInfo'; // 修改个人信息
export getShipperInfo from './personal/getShipperInfo';// 获取物流公司信息
export modifyShipperInfo from './personal/modifyShipperInfo'; // 修改物流公司信息
export getAgentInfo from './personal/getAgentInfo';// 获取收货点信息
export modifyAgentInfo from './personal/modifyAgentInfo'; // 修改收货点信息
export getUserNameByPhone from './personal/getUserNameByPhone'; // 通过电话号码获取用户名
export getBranchShopInfo from './personal/getBranchShopInfo'; // 获取物流公司入住分店的详情
export submitFeedback from './personal/submitFeedback'; // 提交反馈意见

// account
export recharge from './account/recharge'; // 充值
export withdraw from './account/withdraw'; // 提现
export getBillList from './account/getBillList'; // 获取账单
export getRemainAmount from './account/getRemainAmount'; // 获取余额值
export setPaymentPassword from './account/setPaymentPassword'; // 设置支付密码
export getBills from './account/getBills'; // 获取

// weixinPay
export weixinPayGetUnifiedOrder from './weixinPay/getUnifiedOrder'; // 统一下单(App支付)
export weixinPayGetUnifiedOrderForPC from './weixinPay/getUnifiedOrderForPC'; // 统一下单(扫码支付)

// roadmap
export getRoadmapListWithPreOrder from './roadmap/getRoadmapListWithPreOrder'; // 通过预下单获取收货点和分店路线列表
export getRoadmapListWithPreOrderGroup from './roadmap/getRoadmapListWithPreOrderGroup'; // 通过预下单批次获取收货点和分店路线列表
export getRoadmapListWithEndPoint from './roadmap/getRoadmapListWithEndPoint'; // 通过终点获取收货点和分店路线列表
export getRoadmapDetail from './roadmap/getRoadmapDetail'; // 获取路线详情

// order
export finishOrder from './order/finishOrder'; // 确认收货
export payForOrderWhenReceive from './order/payForOrderWhenReceive'; // 付款后确认收货
export payForOrderWhenSend from './order/payForOrderWhenSend'; // 货主使用app支付
export payForAgentOrder from './order/payForAgentOrder'; // 支付收货点的费用
export getSendOrderList from './order/getSendOrderList'; // 获取发生货单列表
export getReceiveOrderList from './order/getReceiveOrderList'; // 获取接收货单列表
export getOrderDetail from './order/getOrderDetail'; // 获取货单详情
export getLogisticsList from './order/getLogisticsList'; // 获取物流信息
export getSendOrders from './order/getSendOrders'; // 获取发生货单综合列表
export getReceiveOrders from './order/getReceiveOrders'; // 获取接收货单综合列表

// preOrder
export placePreOrder from './preOrder/placePreOrder'; // 预下单
export modifyPreOrder from './preOrder/modifyPreOrder'; // 修改预下单
export removePreOrder from './preOrder/removePreOrder'; // 删除预下单
export getPreOrderList from './preOrder/getPreOrderList'; // 获取预下单列表
export getPreOrderGroupList from './preOrder/getPreOrderGroupList'; // 获取预下单批次列表
export createPreOrderGroup from './preOrder/createPreOrderGroup'; // 创建预下单批次
export modifyPreOrderGroup from './preOrder/modifyPreOrderGroup'; // 修改预下单批次
export removePreOrderFromGroup from './preOrder/removePreOrderFromGroup'; // 删除货单批次中的一个货单
export getPreOrderListInGroup from './preOrder/getPreOrderListInGroup'; // 获取批次中的预下单列表

// client
export getClientList from './client/getClientList'; // 获取客户列表

```

* PDShopServer/project/App/routers/posts/common/index.js

```js
// notify
export getNotifies from './notify/getNotifies'; // 获取通知的综合列表
export sendNotify from './notify/sendNotify'; // 发送通知

// upload
export uploadFile from './upload/uploadFile'; // 上传文件

// address
export getRegionAddress from './address/getRegionAddress'; // 通过parentCode获取地址列表
export getRegionAddressFromLastCode from './address/getRegionAddressFromLastCode'; // 通过lastCode获取地址列表
export getRegionSendDoorAddressFromLastCode from './address/getRegionSendDoorAddressFromLastCode'; // 通过lastCode获取送货上门地址列表
export getStartPointAddress from './address/getStartPointAddress'; // 通过parentCode获取始发地地址列表
export getStartPointAddressFromLastCode from './address/getStartPointAddressFromLastCode'; // 通过lastCode获取始发地地址列表

// client
export getClientNameByPhone from './client/getClientNameByPhone'; // 通过电话号码获取用户名

// verifyCode
export requestSendVerifyCode from './verifyCode/requestSendVerifyCode'; // 请求发送验证码

// weixinPay
export weixinNotifyCallBack from '../libs/weixinPay/notifyCallBack'; // 微信支付通知回调

```

* PDShopServer/project/App/routers/posts/driver/index.js

```js
// personal
export register from './personal/register'; // 注册
export login from './personal/login'; // 登录
export findPassword from './personal/findPassword'; // 找回密码
export modifyPassword from './personal/modifyPassword'; // 修改密码
export getPersonalInfo from './personal/getPersonalInfo'; // 获取个人信息
export modifyPersonalInfo from './personal/modifyPersonalInfo'; // 修改个人信息
export getUserNameByPhone from './personal/getUserNameByPhone'; // 通过电话号码获取用户名
export submitFeedback from './personal/submitFeedback'; // 提交反馈意见

// truck
export getOnWayTruck from './truck/getOnWayTruck'; // 获取当前车辆
export getLocationList from './truck/getLocationList'; // 获取当前车辆的位置列表
export uploadLocation from './truck/uploadLocation'; // 上传位置
export confirmReach from './truck/confirmReach'; // 确认到达

```

* PDShopServer/project/App/routers/posts/shipper/index.js

```js
// member
export modifyMemberAuthority from './member/modifyMemberAuthority'; // 修改成员的权限
export getMemberList from './member/getMemberList'; // 获取成员列表
export getMemberDetail from './member/getMemberDetail'; // 获取成员详情
export getMemberByPhone from './member/getMemberByPhone'; // 通过电话获取成员详情

// account
export getBondAmountInfo from './account/getBondAmountInfo'; // 获取在某一家分店的担保金额信息

// roadmap
export publishRoadmap from './roadmap/publishRoadmap'; // 发布物流路线
export modifyRoadmap from './roadmap/modifyRoadmap'; // 修改物流路线
export removeRoadmap from './roadmap/removeRoadmap'; // 删除物流路线
export getRoadmapList from './roadmap/getRoadmapList'; // 获取物流路线列表
export getRoadmapDetail from './roadmap/getRoadmapDetail'; // 获取物流路线详情
export setRoadmapPrice from './roadmap/setRoadmapPrice'; // 设置物流路线价格
export setRoadmapEnable from './roadmap/setRoadmapEnable'; // 设置物流路线是否停运
export setRoadmapSendDoorEnable from './roadmap/setRoadmapSendDoorEnable'; // 设置物流路线送货上门是否停运
export setRoadmapSendDoorPrice from './roadmap/setRoadmapSendDoorPrice'; // 设置物流路线送货上门价格
export setRoadmapSendDoorBasePrice from './roadmap/setRoadmapSendDoorBasePrice'; // 设置物流路线送货上门底价

// order
export getNeedSendOrderSummaryList from './order/getNeedSendOrderSummaryList'; // 获取竞得货物汇总列表
export getNeedSendOrderListByEndPoint from './order/getNeedSendOrderListByEndPoint'; // 通过终点获取竞得货物列表
export confirmHandOverOrder from './order/confirmHandOverOrder'; // 确认交货
export getOrders from './order/getOrders'; // 获取货单综合列表

// truck
export createTruck from './truck/createTruck'; // 创建货车
export modifyTruck from './truck/modifyTruck'; // 修改货车
export removeTruck from './truck/removeTruck'; // 删除货车
export getWorkTruckList from './truck/getWorkTruckList'; // 获取待工作货车列表
export getHistoryTruckList from './truck/getHistoryTruckList'; // 获取历史货车列表
export getTruckDetail from './truck/getTruckDetail'; // 获取货车详情
export getTruckType from './truck/getTruckType'; // 获取货车类型
export getDriverInfoByPhone from './truck/getDriverInfoByPhone'; // 通过司机手机号获取司机信息
export getOrderListInTruck from './truck/getOrderListInTruck'; // 获取货车的货单列表
export getTrucks from './truck/getTrucks'; // 获取货车综合列表
export getUnfinishTruck from './truck/getUnfinishTruck'; // 获取未装完货车信息

// loadCargo
export getCarryPartmentList from './loadCargo/getCarryPartmentList'; // 获取搬运部列表
export selectCarryPartment from './loadCargo/selectCarryPartment'; // 选择搬运部
export startScanOrder from './loadCargo/startScanOrder'; // 开始扫描装车 （在列表的时候点击调用）
export scanLoadOrder from './loadCargo/scanLoadOrder'; // 扫描货单装车
export scanOrder from './loadCargo/scanLoadOrder'; // 扫描货单装车
export finishScanOrder from './loadCargo/finishScanOrder'; // 结束扫描
export continueScanOrder from './loadCargo/continueScanOrder'; // 继续扫描装车 （在结束扫描的后点击继续扫描时调用）
export payForCarryPartment from './loadCargo/payForCarryPartment'; // 给搬运队付钱

// cityDistribute
export getClientPickShipperList from './cityDistribute/getClientPickShipperList'; // 获取设置了上门自提竞价的物流公司列表
export placeStorage from './cityDistribute/placeStorage'; // 入库
export setClientPickOrderTransportFee from './cityDistribute/setClientPickOrderTransportFee'; // 打电话确认后设置地址和价格
export setCityTruck from './cityDistribute/setCityTruck'; // 设置同城配送的货车
export getCityTruckDetail from './cityDistribute/getCityTruckDetail'; // 获取同城配送的货车的详情
export getOrderListInCityTruck from './cityDistribute/getOrderListInCityTruck'; // 获取同城配送的货车的货单列表
export scanLoadOrderForCityDistribute from './cityDistribute/scanLoadOrderForCityDistribute'; // 获取同城配送扫描装车
export getCityDistributeOrders from './cityDistribute/getOrders'; // 获取同城配送货单列表

// statistics
export getStatistics from './statistics/getStatistics'; // 获取统计信息

```

* PDShopServer/project/App/routers/posts/shop/index.js

```js
// common
export login from './personal/login'; // 登录
export findPassword from './personal/findPassword'; // 找回密码
export modifyPassword from './personal/modifyPassword'; // 修改密码
export getPersonalInfo from './personal/getPersonalInfo'; // 获取个人信息
export modifyPersonalInfo from './personal/modifyPersonalInfo'; // 修改个人信息
export submitFeedback from './personal/submitFeedback'; // 提交反馈意见

// account
export recharge from './account/recharge'; // 部门充值
export withdraw from './account/withdraw'; // 提现
export getRemainAmount from './account/getRemainAmount'; // 获取余额
export setPaymentPassword from './account/setPaymentPassword'; // 设置支付密码
export getBillList from './account/getBillList'; // 获取账单列表
export getBills from './account/getBills'; // 获取综合账单

// weixinPay
export weixinPayGetUnifiedOrder from './weixinPay/getUnifiedOrder'; // 统一下单(App支付)
export weixinPayGetUnifiedOrderForPC from './weixinPay/getUnifiedOrderForPC'; // 统一下单(扫码支付)

// member
export createMember from './member/createMember'; // 创建公司成员
export getMemberList from './member/getMemberList'; // 获取公司成员列表
export getMemberDetail from './member/getMemberDetail'; // 获取公司成员详情
export modifyMember from './member/modifyMember'; // 修改公司成员信息
export removeMember from './member/removeMember'; // 删除公司成员信息
export getWarehouseMemberList from './member/getWarehouseMemberList'; // 获取库管成员

// partment
export createPartment from './partment/createPartment'; // 创建部门
export getPartmentList from './partment/getPartmentList'; // 获取部门列表
export getPartmentDetail from './partment/getPartmentDetail'; // 获取部门详情
export modifyPartment from './partment/modifyPartment'; // 修改部门信息
export removePartment from './partment/removePartment'; // 删除部门信息
export getOwnPartmentInfo from './partment/getOwnPartmentInfo'; // 获取所在部门信息
export modifyOwnPartment from './partment/modifyOwnPartment'; // 修改所在部门信息

// masterShop
export createBranchShop from './masterShop/createBranchShop'; // 创建分部
export getBranchShopList from './masterShop/getBranchShopList'; // 获取分部列表
export getBranchShopDetail from './masterShop/getBranchShopDetail'; // 获取分部详情
export modifyBranchShop from './masterShop/modifyBranchShop'; // 修改分部信息
export removeBranchShop from './masterShop/removeBranchShop'; // 删除分部
export getSettingInfo from './masterShop/getSettingInfo'; // 获取设置的信息
export modifySettingInfo from './masterShop/modifySettingInfo'; // 获取设置的信息

// branchShop
export createAndRegisterShipper from './branchShop/createAndRegisterShipper'; // 创建并注册物流公司
export registerShipper from './branchShop/registerShipper'; // 注册物流物流公司
export getShipperByChairManPhone from './branchShop/getShipperByChairManPhone'; // 通过董事长电话获取物流公司信息
export getToExamineTruckList from './branchShop/getToExamineTruckList'; // 获取待审核货车的列表

// order
export getOrders from './order/getOrders'; // 获取货单列表
export getOrderDetail from './order/getOrderDetail'; // 获取货单详情

// shop
export getOwnShopDetail from './shop/getOwnShopDetail'; // 获取所在店的信息
export modifyOwnShop from './shop/modifyOwnShop'; // 修改所在分店和总部的信息

// receivePartment
export placeOriginOrder from './receivePartment/placeOriginOrder'; // 收货员下初始单
export placeOriginOrderWithPreOrder from './receivePartment/placeOriginOrderWithPreOrder'; // 收货员根据预下单下初始单
export placeTransferOrder from './receivePartment/placeTransferOrder'; // 收货员下中转单
export modifyOriginOrder from './receivePartment/modifyOriginOrder'; // 收货员修改初始货单
export getLastestOrderByReceivePartment from './receivePartment/getLastestOrder'; // 获取收货部最近下的货单
export getOrdersByReceivePartment from './receivePartment/getOrders'; // 获取货单列表
export getRoadmapListWithOrderByReceivePartment from './receivePartment/getRoadmapListWithOrder'; // 根据货单获取路线列表
export selectRoadmap from './receivePartment/selectRoadmap'; // 选择路线
export selectRoadmapForOrderList from './receivePartment/selectRoadmapForOrderList'; // 批量选择路线
export proxyPay from './receivePartment/proxyPay'; // 代支付
export proxyPayForOrderList from './receivePartment/proxyPayForOrderList'; // 代支付多个货单
export printOrderBill from './receivePartment/printOrderBill'; // 打印货单
export printOrderListBill from './receivePartment/printOrderListBill'; // 打印货单列表的货单
export printBarCode from './receivePartment/printBarCode'; // 打印二维码
export removeOrder from './receivePartment/removeOrder'; // 删除货单
export getRegionAddressWithOrder from './receivePartment/getRegionAddressWithOrder'; // 通过货单获取地址
export getRegionSendDoorAddressWithOrder from './receivePartment/getRegionSendDoorAddressWithOrder'; // 通过货单获取送货上门地址
export setPhotoForOriginOrder from './receivePartment/setPhotoForOriginOrder'; // 为初始单设置图片
export getPreOrderList from './receivePartment/getPreOrderList'; // 收货部获取用户预下单列表

// warehousePartment
export placeStorage from './warehousePartment/placeStorage'; // 库管放置货物
export getOrdersByWareHousePartment from './warehousePartment/getOrders'; // 库管获取货单列表
export getLoadingOrderList from './warehousePartment/getLoadingOrderList'; // 库管获取正在装车的信息

// carryPartment
export getWorkTruckInfo from './carryPartment/getWorkTruckInfo'; // 获取当前正在搬运的货车的信息
export getCarryPartmentList from './carryPartment/getCarryPartmentList'; // 获取搬运队的列表查看行情
export getCarryList from './carryPartment/getCarryList'; // 获取搬运队的搬运历史列表
export getWaitScanTruckInfo from './carryPartment/getWaitScanTruckInfo'; // 获取搬运队的待扫描货车
export getOrderListInTruck from './carryPartment/getOrderListInTruck'; // 获取货车中的货单列表

// securityCheckPartment
export checkTruckPass from './securityCheckPartment/checkTruckPass'; // 保安部扫描是否可以通行

// shipper
export getShipperList from './shipper/getShipperList'; // 获取物流公司列表
export getShipperDetail from './shipper/getShipperDetail'; // 获取物流公司详情
export modifyShipper from './shipper/modifyShipper'; // 修改物流公司信息
export setBondCompany from './shipper/setBondCompany'; // 设置物流公司担保公司
export removeBondCompany from './shipper/removeBondCompany'; // 删除物流公司担保公司
export getNeedExamineTruckList from './shipper/getNeedExamineTruckList'; // 获取需要审核的货车
export passExamineTruck from './shipper/passExamineTruck'; // 通过审核货车

// agent
export createAgent from './agent/createAgent'; // 创建收货点
export modifyAgent from './agent/modifyAgent'; // 修改收货点
export removeAgent from './agent/removeAgent'; // 删除收货点
export getAgentList from './agent/getAgentList'; // 获取收货点列表
export getAgentDetail from './agent/getAgentDetail'; // 获取收货点详情
export checkAgentByChairManPhone from './agent/checkAgentByChairManPhone'; // 检测收货点董事长

// roadmap
export getRoadmapList from './roadmap/getRoadmapList'; // 获取物流路线列表
export getRoadmapDetail from './roadmap/getRoadmapDetail'; // 获取物流路线详情
export setRoadmapProfit from './roadmap/setRoadmapProfit'; // 修改路线的提成
export addRoadmapRegionProfitRate from './roadmap/addRoadmapRegionProfitRate'; // 添加区域的路线提成
export removeRoadmapRegionProfitRate from './roadmap/removeRoadmapRegionProfitRate'; // 删除区域的路线提成
export modifyRoadmapRegionProfitRate from './roadmap/modifyRoadmapRegionProfitRate'; // 修改区域的路线提成
export getRoadmapRegionProfitRateDetail from './roadmap/getRoadmapRegionProfitRateDetail'; // 获取区域的路线提成详情
export getRoadmapRegionProfitRateList from './roadmap/getRoadmapRegionProfitRateList'; // 获取区域的路线提成列表
export setRegionRateProfit from './roadmap/setRegionRateProfit'; // 批量修改区域的路线提成

// warehouse
export createWarehouse from './warehouse/createWarehouse'; // 创建仓库
export getWarehouseList from './warehouse/getWarehouseList'; // 获取仓库列表
export getWarehouseDetail from './warehouse/getWarehouseDetail'; // 获取仓库详情
export modifyWarehouse from './warehouse/modifyWarehouse'; // 修改仓库信息
export removeWarehouse from './warehouse/removeWarehouse'; // 删除仓库信息

// client
export getClientList from './client/getClientList'; // 获取用户列表
export getClientDetail from './client/getClientDetail'; // 获取用户详情

// statistics
export getStatistics from './statistics/getStatistics'; // 获取统计信息 (总部和分部)

```

* PDShopServer/project/App/routers/posts/admin/authority/modifyClientAuthority.js

```js
import { ClientModel } from '../../../../models';
import _ from 'lodash';

export default async ({
    userId,
    clientId,
    authority,
}) => {
    if (authority) {
        authority = _.sortBy(_.uniq(authority));
    }
    const doc = await ClientModel.findByIdAndUpdate(clientId, { authority }, { new: true }).select({
        phone: 1,
        email: 1,
        name: 1,
        head: 1,
        post: 1,
        partmentId: 1,
        authority: 1,
        phoneList: 1,
    }).populate({
        path: 'partmentId',
        select: { name: 1 },
    });
    if (!doc) {
        return { msg: '修改失败' };
    }
    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/admin/authority/modifyMemberAuthority.js

```js
import { ShopMemberModel } from '../../../../models';
import _ from 'lodash';

export default async ({
    userId,
    memberId,
    authority,
}) => {
    if (authority) {
        authority = _.sortBy(_.uniq(authority));
    }
    const doc = await ShopMemberModel.findByIdAndUpdate(memberId, { authority }, { new: true }).select({
        phone: 1,
        email: 1,
        name: 1,
        head: 1,
        post: 1,
        partmentId: 1,
        authority: 1,
        phoneList: 1,
    }).populate({
        path: 'partmentId',
        select: { name: 1 },
    });
    if (!doc) {
        return { msg: '修改失败' };
    }
    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/admin/authority/resetAgentChairManAuthority.js

```js
import { ClientModel, AgentModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    const agents = await AgentModel.find();
    for (const agent of agents) {
        await ClientModel.findByIdAndUpdate(agent.chairManId, {
            authority: [
                CONSTANTS.AH_MODIFY_AGENT_INFO, // 修改收货点信息的权限
                CONSTANTS.AH_MODIFY_AGENT_MEMBER_AUTHORITY, // 修改收货点成员权限的权限
                CONSTANTS.AH_PLACE_ORDER, // 收货下单的权限
                // 公共权限
                CONSTANTS.AH_MODIFY_ROADMAP_PROFIT, // 修改路线的提成的权限
                CONSTANTS.AH_RECHARGE, // 充值的权限
                CONSTANTS.AH_WITHDRAW, // 提现的权限
                CONSTANTS.AH_LOOK_AMOUNT, // 查看余额的权限
                // 统计
                CONSTANTS.AH_LOOK_ORDERS, // 查看综合货单的权限
                CONSTANTS.AH_LOOK_STATISTICS, // 查看统计信息的权限
            ],
        });
    }
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/authority/resetBranchChairManAuthority.js

```js
import { ShopMemberModel, ShopModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    const shops = await ShopModel.find({ isMasterShop: false });
    const chairManIds = shops.map(o => o.chairManId);
    await ShopMemberModel.update({ _id: { $in: chairManIds } }, {
        authority: [
            // 物流公司
            CONSTANTS.AH_CREATE_SHIPPER, // 创建物流公司
            CONSTANTS.AH_MODIFY_SHIPPER, // 修改物流公司
            CONSTANTS.AH_REMOVE_SHIPPER, // 删除物流公司
            CONSTANTS.AH_LOOK_SHIPPER, // 查看物流公司
            // 部门
            CONSTANTS.AH_CREATE_PARTMENT, // 创建部门的权限
            CONSTANTS.AH_MODIFY_PARTMENT, // 修改部门信息的权限
            CONSTANTS.AH_REMOVE_PARTMENT, // 删除部门的权限
            CONSTANTS.AH_LOOK_PARTMENT, // 查看部门的权限
            // 路线
            CONSTANTS.AH_LOOK_ROADMAP, // 查看路线的权限
            CONSTANTS.AH_TO_EXAMINE_TRUCK, // 审核货车
            // 公共权限
            CONSTANTS.AH_MODIFY_OWN_SHOP, // 修改自身分店的权限
            // 成员
            CONSTANTS.AH_CREATE_MEMBER, // 创建成员的权限
            CONSTANTS.AH_MODIFY_MEMBER, // 修改成员信息的权限
            CONSTANTS.AH_REMOVE_MEMBER, // 删除成员的权限
            CONSTANTS.AH_LOOK_MEMBER, // 查看成员的权限
            // 仓库
            CONSTANTS.AH_CREATE_WAREHOUSE, // 创建仓库的权限
            CONSTANTS.AH_MODIFY_WAREHOUSE, // 修改仓库信息的权限
            CONSTANTS.AH_REMOVE_WAREHOUSE, // 删除仓库的权限
            CONSTANTS.AH_LOOK_WAREHOUSE, // 查看仓库的权限
            // 提现，余额
            CONSTANTS.AH_WITHDRAW, // 提现的权限
            CONSTANTS.AH_LOOK_AMOUNT, // 查看余额的权限
            // 统计
            CONSTANTS.AH_LOOK_STATISTICS, // 查看统计信息的权限
            CONSTANTS.AH_LOOK_ORDERS, // 查看综合货单的权限
        ],
    }, { multi: true });

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/authority/resetMasterChairManAuthority.js

```js
import { ShopMemberModel, ShopModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    const shop = await ShopModel.findOne({ isMasterShop: true });
    await ShopMemberModel.findByIdAndUpdate(shop.chairManId, {
        authority: [
            // 分店
            CONSTANTS.AH_CREATE_BRANCH_SHOP, // 创建分店的权限
            CONSTANTS.AH_MODIFY_BRANCH_SHOP, // 修改分店的权限
            CONSTANTS.AH_REMOVE_BRANCH_SHOP, // 删除分店的权限
            CONSTANTS.AH_LOOK_BRANCH_SHOP, // 查看分店的权限
            // 设置
            CONSTANTS.AH_MODIFY_SETTING, // 修改设置的权限
            // 物流公司
            CONSTANTS.AH_LOOK_SHIPPER, // 查看物流公司
            // 收货点
            CONSTANTS.AH_CREATE_AGENT, // 创建收货点的权限
            CONSTANTS.AH_MODIFY_AGENT, // 修改收货点的权限
            CONSTANTS.AH_REMOVE_AGENT, // 删除收货点的权限
            CONSTANTS.AH_LOOK_AGENT, // 查看收货点的权限
            // 公共权限
            CONSTANTS.AH_MODIFY_OWN_SHOP, // 修改所在总部信息的权限
            // 成员
            CONSTANTS.AH_CREATE_MEMBER, // 创建成员的权限
            CONSTANTS.AH_MODIFY_MEMBER, // 修改成员信息的权限
            CONSTANTS.AH_REMOVE_MEMBER, // 删除成员的权限
            CONSTANTS.AH_LOOK_MEMBER, // 查看成员的权限
            // 提现，余额
            CONSTANTS.AH_WITHDRAW, // 提现的权限
            CONSTANTS.AH_LOOK_AMOUNT, // 查看余额的权限
            CONSTANTS.AH_MODIFY_ROADMAP_PROFIT, // 修改路线的提成
            // 统计
            CONSTANTS.AH_LOOK_STATISTICS, // 查看统计信息的权限
            CONSTANTS.AH_LOOK_ORDERS, // 查看综合货单的权限
        ],
    });

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/authority/resetPartmentChargeManAuthority.js

```js
import { ShopMemberModel, ShopPartmentModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    const authorities = {
        [CONSTANTS.SP_RECEIVE_PARTMENT]: [CONSTANTS.AH_RECEIVE_PARTMENT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW, CONSTANTS.AH_LOOK_AMOUNT],
        [CONSTANTS.SP_WARE_HOUSE_PARTMENT]: [CONSTANTS.AH_WARE_HOUSE_PARTMENT],
        [CONSTANTS.SP_CARRY_PARTMENT]: [CONSTANTS.AH_CARRY_PARTMENT, CONSTANTS.AH_WITHDRAW, CONSTANTS.AH_LOOK_AMOUNT],
        [CONSTANTS.SP_SECURITY_CHECK_PARTMENT]: [CONSTANTS.AH_SECURITY_CHECK_PARTMENT],
    };
    const partments = await ShopPartmentModel.find();
    for (const partment of partments) {
        await ShopMemberModel.findByIdAndUpdate(partment.chargeManId, {
            authority: [
                ...(authorities[partment.type] || []), // 部门的权限
                // 部门
                CONSTANTS.AH_MODIFY_OWN_PARTMENT, // 修改所在部门信息的权限
                // 成员
                CONSTANTS.AH_CREATE_MEMBER, // 创建成员的权限
                CONSTANTS.AH_MODIFY_MEMBER, // 修改成员信息的权限
                CONSTANTS.AH_REMOVE_MEMBER, // 删除成员的权限
                CONSTANTS.AH_LOOK_MEMBER, // 查看成员的权限
                // 统计
                CONSTANTS.AH_LOOK_STATISTICS, // 查看统计信息的权限
            ],
        });
    }
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/authority/resetShipperChairManAuthority.js

```js
import { ClientModel, ShipperModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    const shippers = await ShipperModel.find();
    for (const shipper of shippers) {
        const shipperType = shipper.shipperType;
        const commonAuthority = [
            CONSTANTS.AH_MODIFY_SHIPPER_INFO, // 修改物流公司信息的权限
            CONSTANTS.AH_MODIFY_SHIPPER_MEMBER_AUTHORITY, // 修改物流公司成员权限的权限

            // 充值，提现，余额
            CONSTANTS.AH_RECHARGE, // 充值的权限
            CONSTANTS.AH_WITHDRAW, // 提现的权限
            CONSTANTS.AH_LOOK_AMOUNT, // 查看余额的权限
            // 统计
            CONSTANTS.AH_LOOK_STATISTICS, // 查看统计信息的权限
            CONSTANTS.AH_LOOK_ORDERS, // 查看综合货单的权限
        ];

        await ClientModel.findByIdAndUpdate(shipper.chairManId, {
            authority: shipperType === CONSTANTS.ST_LONG ? [
                ...commonAuthority,
                // 路线
                CONSTANTS.AH_CREATE_ROADMAP, // 创建路线的权限
                CONSTANTS.AH_MODIFY_ROADMAP, // 修改路线的权限
                CONSTANTS.AH_REMOVE_ROADMAP, // 删除路线的权限
                CONSTANTS.AH_LOOK_ROADMAP, // 查看路线的权限
                // 货车
                CONSTANTS.AH_CREATE_TRUCK, // 创建货车的权限
                CONSTANTS.AH_MODIFY_TRUCK, // 修改货车的权限
                CONSTANTS.AH_REMOVE_TRUCK, // 删除货车的权限
                CONSTANTS.AH_LOOK_TRUCK, // 查看货车的权限
                // 装车
                CONSTANTS.SELECT_CARRY_PARTMENT, // 选择搬运队的权限
                CONSTANTS.AH_SCAN_LOAD_TRUCK, // 扫描装车的权限
            ] : [
                ...commonAuthority,
                // 同城配送路线
                CONSTANTS.AH_CREATE_SEND_DOOR_ROADMAP, // 创建同城配送路线的权限
                CONSTANTS.AH_MODIFY_SEND_DOOR_ROADMAP, // 修改同城配送路线的权限
                CONSTANTS.AH_REMOVE_SEND_DOOR_ROADMAP, // 删除同城配送路线的权限
                CONSTANTS.AH_LOOK_SEND_DOOR_ROADMAP, // 查看同城配送路线的权限
                // 同城配送
                CONSTANTS.AH_CITY_DISTRIBUTE, // 同城配送的权限
            ],
        });
    }
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/feedback/acceptDeelFeedback.js

```js
import { FeedbackModel } from '../../../../models';

export default async ({
    userId,
    feedbackId,
}) => {
    await FeedbackModel.findByIdAndUpdate(feedbackId, { state: 1, finishTime: Date.now() });
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/feedback/finishDeelFeedback.js

```js
import { FeedbackModel } from '../../../../models';

export default async ({
    userId,
    feedbackId,
    reply,
}) => {
    await FeedbackModel.findByIdAndUpdate(feedbackId, { state: 2, acceptTime: Date.now() });
    // email reply
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/feedback/getFeedbackList.js

```js
import { FeedbackModel } from '../../../../models';
import _ from 'lodash';

async function getPageData (userId, state, keyword, fromPC, pageNo, pageSize) {
    let criteria = state < 0 ? {} : { state };
    if (keyword) {
        const regex = new RegExp('.*' + (keyword || '') + '.*', 'gim');
        criteria = { ...criteria, content: regex };
    }
    const sortCriteria = state === 1 ? { acceptTime: 'desc' } : state === 2 ? { finishTime: 'desc' } : { submitTime: 'desc' };
    const count = (fromPC && pageNo === 0) ? await FeedbackModel.count(criteria) : undefined;
    const query = FeedbackModel.find(criteria).sort(sortCriteria).skip(pageNo * pageSize).limit(pageSize);
    const list = await query
    .select({
        feedbackId: 1,
        email: 1,
        content: 1,
        state: 1,
        finishTime: 1,
        acceptTime: 1,
        submitTime: 1,
    });
    return {
        count,
        list,
    };
}

const TYPES = ['all', 'untreated', 'processing', 'finish'];
export default async ({ userId, type, keyword, fromPC, pageNo, pageSize }) => {
    const index = _.indexOf(TYPES, type);
    if (index !== -1) {
        return {
            success: true,
            context: {
                [type]: await getPageData(userId, index - 1, keyword, fromPC, pageNo, pageSize),
            },
        };
    } else {
        return {
            success: true,
            context: {
                all: await getPageData(userId, -1, keyword, fromPC, pageNo, pageSize),
                untreated: await getPageData(userId, 0, keyword, fromPC, pageNo, pageSize),
                processing: await getPageData(userId, 1, keyword, fromPC, pageNo, pageSize),
                finish: await getPageData(userId, 2, keyword, fromPC, pageNo, pageSize),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/admin/personal/findPassword.js

```js
import { AdminModel } from '../../../../models';
import { setPassword, genRandomPassword, sendFindPasswordMail } from '../../../../utils';

export default async ({ phone, email }) => {
    const user = await AdminModel.findOne({ phone });
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    if (user.email !== email) {
        return { msg: '邮箱和注册的时候的邮箱不一致' };
    }
    let newPassword = genRandomPassword();
    let ret = await sendFindPasswordMail(email, newPassword);
    if (!ret) {
        return { msg: '发送邮件失败' };
    }
    const error = await setPassword(user, newPassword);
    if (error) {
        return { msg: '服务器错误' };
    }
    await user.save();
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/personal/login.js

```js
import { AdminModel } from '../../../../models';
import { authenticatePassword } from '../../../../utils';

export default async ({ phone, password }) => {
    const user = await AdminModel.findOne({ phone });
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    const error = await authenticatePassword(user, password);
    if (error) {
        return { msg: '密码错误' };
    } else {
        return { success: true, context: { userId: user.id } };
    }
};

```

* PDShopServer/project/App/routers/posts/admin/personal/modifyPassword.js

```js
import { AdminModel } from '../../../../models';
import { setPassword, authenticatePassword } from '../../../../utils';

export default async ({ userId, oldPassword, newPassword }) => {
    const user = await AdminModel.findById(userId);
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    let error = await authenticatePassword(user, oldPassword);
    if (error) {
        return { msg: '密码错误' };
    }
    error = await setPassword(user, newPassword);
    if (error) {
        return { msg: '设置密码失败' };
    }
    await user.save();
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/personal/register.js

```js
import { AdminModel } from '../../../../models';
import { registerUser } from '../../../../utils';

export default async ({ phone, password, email }) => {
    const user = new AdminModel({
        phone,
        email,
    });
    const error = await registerUser(AdminModel, user, password);
    if (!error) {
        return { success: true };
    } else if (error === 'UserExistsError') {
        return { msg: '该账号已经被占用' };
    } else {
        return { msg: '注册失败' };
    }
};

```

* PDShopServer/project/App/routers/posts/admin/personal/removeUnusedMedia.js

```js
import mongoose from 'mongoose';
import { MediaModel } from '../../../../models';

export default async ({
    userId,
}) => {
    const docs = await MediaModel.find({ ref: 0 });
    docs.forEach(doc => {
        mongoose.gfs.remove({ _id: doc.gridId });
        doc.remove();
    });
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/shop/createMasterShop.js

```js
import { ShopMemberModel, ShopModel, SettingModel, RoadmapRegionProfitRateModel, MediaModel } from '../../../../models';
import { sequelize, AccountModel } from '../../../../mysql';
import { settingMgr } from '../../../../manager';
import { _T, getMediaId, registerUser, getDefaultPassWord, checkAuthorizationCode } from '../../../../utils';
import CONSTANTS from '../../../../constants';

// 创建基础账号（银行对应账户/安全账户/总部账户）
function createBasicAccount (masterShopId) {
    return new Promise(async resolve => {
        sequelize.transaction(async (transaction) => {
            await AccountModel.create({ targetId: masterShopId, type: CONSTANTS.AT_BASIC }, { transaction });
            await AccountModel.create({ targetId: masterShopId, type: CONSTANTS.AT_BASIC }, { transaction });
            await AccountModel.create({ targetId: masterShopId, type: CONSTANTS.AT_BASIC }, { transaction });
        }).then((result) => {
            return resolve(true);
        }).catch(err => {
            return resolve(false);
        });
    });
}

export default async ({
    userId,
    authorizationCode,
    chairManPhone,  // 总部董事长登录电话
    chairManName,  // 总部董事长姓名
    chairManPassword, // 分店董事长登录密码（不传入该参数，默认为手机号码后6位）
    name, // 总部名称
    image, // 总部背景图片
    sign, // 总部签名
    phoneList, // 总部联系电话
    address, // 总部地址
    location, // 经纬度定位
}) => { // 创建一个全局配置
    if (authorizationCode) {
        if (!checkAuthorizationCode(authorizationCode)) {
            return { msg: '没有权限' };
        }
    }
    const masterShop = await ShopModel.findOne({ isMasterShop: true });
    if (masterShop) {
        return { msg: '总部已经存在' };
    }

    // 注册董事长账号
    const chairMan = new ShopMemberModel({
        phone: chairManPhone,
        name: chairManName,
        post: CONSTANTS.PO_CHAIR_MAN,
        authority: [
            // 分店
            CONSTANTS.AH_CREATE_BRANCH_SHOP, // 创建分店的权限
            CONSTANTS.AH_MODIFY_BRANCH_SHOP, // 修改分店的权限
            CONSTANTS.AH_REMOVE_BRANCH_SHOP, // 删除分店的权限
            CONSTANTS.AH_LOOK_BRANCH_SHOP, // 查看分店的权限
            // 设置
            CONSTANTS.AH_MODIFY_SETTING, // 修改设置的权限
            // 物流公司
            CONSTANTS.AH_LOOK_SHIPPER, // 查看物流公司
            // 收货点
            CONSTANTS.AH_CREATE_AGENT, // 创建收货点的权限
            CONSTANTS.AH_MODIFY_AGENT, // 修改收货点的权限
            CONSTANTS.AH_REMOVE_AGENT, // 删除收货点的权限
            CONSTANTS.AH_LOOK_AGENT, // 查看收货点的权限
            // 公共权限
            CONSTANTS.AH_MODIFY_OWN_SHOP, // 修改所在总部信息的权限
            // 成员
            CONSTANTS.AH_CREATE_MEMBER, // 创建成员的权限
            CONSTANTS.AH_MODIFY_MEMBER, // 修改成员信息的权限
            CONSTANTS.AH_REMOVE_MEMBER, // 删除成员的权限
            CONSTANTS.AH_LOOK_MEMBER, // 查看成员的权限
            // 提现，余额
            CONSTANTS.AH_WITHDRAW, // 提现的权限
            CONSTANTS.AH_LOOK_AMOUNT, // 查看余额的权限
            CONSTANTS.AH_MODIFY_ROADMAP_PROFIT, // 修改路线的提成
            // 统计
            CONSTANTS.AH_LOOK_STATISTICS, // 查看统计信息的权限
            CONSTANTS.AH_LOOK_ORDERS, // 查看综合货单的权限
        ],
    });

    // 创建总部
    let _image = getMediaId(image);
    const shop = new ShopModel({
        chairManId: chairMan.id,
        isMasterShop: true,
        name,
        image: _image,
        sign,
        phoneList,
        address,
        location,
    });
    chairMan.shopId = shop.id;

    // 保存
    const error = await registerUser(ShopMemberModel, chairMan, getDefaultPassWord(chairManPhone, chairManPassword));
    if (error === 'UserExistsError') {
        return { msg: '董事长账号已经被占用' };
    } else if (error) {
        return { msg: '董事长账号注册失败' };
    }

    // 创建总部账号
    const success = await createBasicAccount(_T(shop.id));
    if (!success) {
        return { msg: '创建总部账号失败' };
    }
    await shop.save();

    // 创建一个全局配置
    const setting = new SettingModel({
        gradientPriceList: [{ min: 0, rate: 3 }, { min: 0.1, rate: 2 }, { min: 0.3, rate: 1.5 }, { min: 0.8, rate: 1 }, { min: 3, rate: 0.95 }, { min: 8, rate: 0.9 }],
        rankedMaskRateList: [ 1 ],
    });
    await setting.save();
    settingMgr.set(setting);

    // 创建路线提成的所有分店和所有方向
    const regionProfit = new RoadmapRegionProfitRateModel({ profitRate: 0.2, level: 1 });
    await regionProfit.save();

    MediaModel._updateRef(
        { [_image]: 1 },
    );
    return { success: true, context: shop };
};

```

* PDShopServer/project/App/routers/posts/admin/test/test.js

```js
import _ from 'lodash';

export default async ({ address, oldAddress }, req) => {
    return { success: true, context: req.get('X-Real-IP') || req.get('X-Forwarded-For') || req.ip };
};

```

* PDShopServer/project/App/routers/posts/admin/version/addVersion.js

```js
import { VersionModel } from '../../../../models';

export default async ({
    platform,
    versionName,
    minVersion,
    versionCode,
    passed,
    description,
}) => {
    const version = new VersionModel({
        platform,
        versionName,
        minVersion,
        versionCode,
        passed,
        description,
    });
    const doc = await version.save();
    if (!doc) {
        return { msg: '发布失败' };
    }

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/admin/version/getVersionList.js

```js
import { VersionModel } from '../../../../models';
import _ from 'lodash';

async function getPageData (userId, platform, fromPC, pageNo, pageSize) {
    const criteria = { platform };
    const count = (fromPC && pageNo === 0) ? await VersionModel.count(criteria) : undefined;
    const query = VersionModel.find(criteria).sort({ time: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const list = await query
    .select({
        verName: 1,
        versionName: 1,
        versionCode: 1,
        minVersion: 1,
        passed: 1,
        description: 1,
        time: 1,
    });
    return {
        count,
        list,
    };
}

const PLATFORMS = ['android', 'ios'];
export default async ({ userId, platform, fromPC, pageNo, pageSize }) => {
    if (_.includes(PLATFORMS, platform)) {
        return {
            success: true,
            context: {
                [platform]: await getPageData(userId, platform, fromPC, pageNo, pageSize),
            },
        };
    } else {
        return {
            success: true,
            context: {
                android: await getPageData(userId, 'android', fromPC, pageNo, pageSize),
                ios: await getPageData(userId, 'ios', fromPC, pageNo, pageSize),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/agent/member/getMemberByPhone.js

```js
import { ClientModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _T } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    memberPhone,
}) => {
    const member = await ClientModel.findById(userId);
    const client = await ClientModel.findOne({ phone: memberPhone });
    if (!client || !client.registerTime) {
        return { msg: '没有该用户' };
    }
    if (client.shipperId) {
        return { msg: '该用户已经是物流公司的成员' };
    }
    if (client.agentId) {
        if (_T(member.agentId) === _T(client.agentId)) {
            return { msg: '该用户已经是收货点成员，无需再添加' };
        } else if (_T(client.agentId) !== _T(member.agentId)) {
            return { msg: '该用户已经是其他收货点的成员' };
        }
    }

    const clientAccount = await AccountModel.findOne({ where: { targetId: client.id, type: CONSTANTS.AT_CLIENT } });
    if (clientAccount && clientAccount.amount > 0) {
        return { msg: '请通知该用户先将其账户中的余额提现后再加入收货点' };
    }

    return { success: true, context: client };
};

```

* PDShopServer/project/App/routers/posts/agent/member/getMemberDetail.js

```js
import { ClientModel } from '../../../../models';

export default async ({
    userId,
    memberId,
}) => {
    const member = await ClientModel.findById(memberId);
    return { success: true, context: member };
};

```

* PDShopServer/project/App/routers/posts/agent/member/getMemberList.js

```js
import { ClientModel } from '../../../../models';

export default async ({
    userId,
}) => {
    const member = await ClientModel.findById(userId);
    if (!member || !member.agentId) {
        return { msg: '你不是收货点成员' };
    }
    const docs = await ClientModel.find({ agentId: member.agentId }).sort({ registerTime: 'asc' });

    return { success: true, context: { memberList: docs } };
};

```

* PDShopServer/project/App/routers/posts/agent/member/modifyMemberAuthority.js

```js
import { ClientModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _T, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    memberId,
    authority,
}) => {
    if (authority) {
        authority = _.sortBy(_.uniq(authority));
    }
    const member = await ClientModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_AGENT_MEMBER_AUTHORITY)) {
        return { msg: '你没有修改成员的权限' };
    }
    const client = await ClientModel.findById(memberId);
    if (!client || !client.registerTime) {
        return { msg: '没有该用户' };
    }
    if (client.shipperId) {
        return { msg: '该用户是物流公司的成员，你不能修改他的权限' };
    }
    if (client.agentId && _T(client.agentId) !== _T(member.agentId)) {
        return { msg: '该用户是其他收货点的成员，你不能修改他的权限' };
    }
    const clientAccount = await AccountModel.findOne({ where: { targetId: memberId, type: CONSTANTS.AT_CLIENT } });
    if (clientAccount && clientAccount.amount > 0) {
        return { msg: '请通知该用户先将其账户中的余额提现后再加入收货点' };
    }
    // 删除权限
    if (!authority || !authority.length) {
        await client.update({ $unset: { agentId: 1 }, authority: [] });
    } else {
        await client.update({ authority, agentId: member.agentId });
    }

    actionLog(userId, 'Client', memberId, 'Client', 'modifyMemberAuthority');
    const context = client.toObject();
    context.authority = authority || [];
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/agent/referShop/getReferShopList.js

```js
import { ShopModel } from '../../../../models';

export default async ({ userId }) => {
    const query = ShopModel.find({ isMasterShop: false }).sort({ createTime: 'desc' });
    const docs = await query
    .select({
        name: 1,
        addressRegion: 1,
        addressRegionLastCode: 1,
        address: 1,
        location: 1,
        profitRate: 1,
        phoneList: 1,
        chairManId: 1,
    })
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });

    return { success: true, context: {
        branchShopList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/agent/referShop/setReferShop.js

```js
import { ClientModel, AgentModel, AgentRegionProfitModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    shopId,
}) => {
    const member = await ClientModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_AGENT_INFO)) {
        return { msg: '你没有修改指定收货点信息的权限' };
    }
    const doc = await AgentModel.findByIdAndUpdate(member.agentId, { referShopId: shopId });
    if (!doc) {
        return { msg: '修改失败' };
    }

    let agentRegion = await AgentRegionProfitModel.findOne({ agentId: member.agentId, shopId, regionLastCode: 0 });
    if (!agentRegion) {
        agentRegion = new AgentRegionProfitModel({ agentId: member.agentId, shopId, profitRate: 0.2, level: 1 });
        await agentRegion.save();
    }

    actionLog(userId, 'Client', shopId, 'Shop', 'setReferShop');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/agent/order/confirmCachPayed.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId, // 收货点收货员
    orderId,
}) => {
    const member = await ClientModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_PLACE_ORDER)) {
        return { msg: '权限不足' };
    }
    const order = await OrderModel.findByIdAndUpdate(orderId, { 'stateList.0.state': CONSTANTS.OS_READY_PRINT_ORDER, modifyTime: Date.now() }, { new: true });
    if (!order) {
        return { msg: '没有该货单' };
    }
    let context = order.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'Client', orderId, 'Order', 'confirmCachPayed');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/agent/order/confirmCachPayedForOrderList.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import { hasAuthority, actionLogWithList } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderIdList,
}) => {
    const member = await ClientModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_PLACE_ORDER)) {
        return { msg: '权限不足' };
    }

    await OrderModel.update({ _id: { $in: orderIdList } }, { 'stateList.0.state': CONSTANTS.OS_READY_PRINT_ORDER, modifyTime: Date.now() }, { multi: true });

    actionLogWithList(userId, 'Client', orderIdList.map(o => ({ id: o, model: 'Order' })), 'confirmCachPayedForOrderList');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/agent/order/getLastestOrder.js

```js
import { OrderModel } from '../../../../models';
import { getAgentLatestOrder } from '../../libs/order';

export default async ({ userId }) => {
    let order = await getAgentLatestOrder(userId);
    if (!order) {
        return { msg: '暂无货单' };
    }

    order = await OrderModel.findById(order.id)
    .populate({
        path: 'agentId',
        select: { name: 1, address: 1 },
    });

    const context = order.toObject();
    context.state = context.stateList[0].state;
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/agent/order/getOrderDetail.js

```js
import { OrderModel } from '../../../../models';

export default async ({ userId, orderId }) => {
    const order = await OrderModel.findById(orderId)
    .populate({
        path: 'agentId',
        select: { name: 1, address: 1 },
    });
    if (!order) {
        return { msg: '没有该货单' };
    }

    const context = order.toObject();
    context.state = context.stateList[0].state;
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/agent/order/getOrders.js

```js
import _ from 'lodash';
import { OrderModel } from '../../../../models';
import { getKeywordCriteriaForOrder } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import moment from 'moment';

async function getPageData (userId, type, params) {
    let { keyword, startDate, endDate, fromPC, pageNo, pageSize } = params;
    let criteria = {};
    if (type === 'topay') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_PAYMENT };
    } else if (type === 'toprintbill') {
        criteria = { 'stateList.state': { $in: [ CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER, CONSTANTS.OS_READY_PRINT_ORDER ] } };
    } else if (type === 'tosend') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_SEND };
    } else if (type === 'onway') {
        criteria = { 'stateList.state': { $gte: CONSTANTS.OS_ON_TRANSFER } };
    } else if (type === 'success') {
        criteria = { 'stateList.state': { $gte: CONSTANTS.OS_RECEIVE_SUCCESS } };
    }
    if (fromPC) {
        if (startDate && endDate) {
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        } else {
            startDate = endDate = moment().format('YYYY-MM-DD');
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        }
    }
    criteria = getKeywordCriteriaForOrder(keyword, { ...criteria, receiveAgentMemberId: userId });
    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        senderName: 1,
        senderPhone: 1,
        receiverName: 1,
        receiverPhone: 1,
        name: 1,
        photo: 1,
        shopId: 1,
        agentId: 1,
        shipperId: 1,
        isCityDistribute: 1,
        endPoint: 1,
        sendDoorEndPoint: 1,
        isSendDoor: 1,
        createTime: 1,
        placeOrderTime: 1,
        modifyTime: 1,
        payMode: 1,
        receiveAgentMemberId: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        stateList: 1,
        needPayTransportFee: 1,
        needPayInsuanceFee: 1,
        totalDesignatedFee: 1,
        proxyCharge: 1,
        deductError: 1,
    }).populate({
        path: 'receiveAgentMemberId',
        select: { name: 1, phone: 1 },
    }).populate({
        path: 'shipperId',
        select: { name: 1 },
    }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    }).populate({
        path: 'agentId',
        select: { name: 1, address: 1 },
    });

    return {
        count,
        list: docs,
    };
};

// ['待支付', '待打印货单', '待发货', '运输中', '已完成']
const TYPES = ['topay', 'toprintbill', 'tosend', 'onway', 'success'];
export default async ({ userId, type, ...params }) => {
    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(userId, type, params),
            },
        };
    } else {
        return {
            success: true,
            context: {
                topay: await getPageData(userId, 'topay', params),
                toprintbill: await getPageData(userId, 'toprintbill', params),
                tosend: await getPageData(userId, 'tosend', params),
                onway: await getPageData(userId, 'onway', params),
                success: await getPageData(userId, 'success', params),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/agent/order/getPreOrderList.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import { getKeywordCriteriaForOrder } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, senderPhone, keyword, fromPC, pageNo, pageSize }) => {
    const client = await ClientModel.findOne({ phone: senderPhone });
    if (!client) {
        return { msg: '没有该用户' };
    }
    const criteria = getKeywordCriteriaForOrder(keyword, { senderId: client.id, 'stateList.state': CONSTANTS.OS_PREORDER });

    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const docs = await OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);

    return { success: true, context: {
        count,
        orderList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/agent/order/getRegionAddressWithOrder.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import { getAgentLatestOrder } from '../../libs/order';
import { getEndPointFromLastCode, getSendDoorAddressFromLastCode } from '../../libs/address';

export default async ({
    userId,
    orderId,
}) => {
    const member = await ClientModel.findById(userId).populate({ path: 'agentId', select: { addressRegionLastCode: 1, referShopId: 1 } });
    const { addressRegionLastCode, referShopId } = (member || {}).agent || {};

    let order;
    if (orderId) {
        order = await OrderModel.findById(orderId);
    } else {
        order = await getAgentLatestOrder(userId);
    }

    let addressList = [];
    let cityAddressList = [];

    if (order) {
        if (!order.isCityDistribute) {
            addressList = await getEndPointFromLastCode(referShopId, order.endPointLastCode);
            cityAddressList = await getSendDoorAddressFromLastCode(0, addressRegionLastCode);
        } else {
            addressList = await getEndPointFromLastCode(referShopId, 0);
            cityAddressList = await getSendDoorAddressFromLastCode(order.endPointLastCode);
        }
    } else {
        addressList = await getEndPointFromLastCode(referShopId, 0);
        cityAddressList = await getSendDoorAddressFromLastCode(0, addressRegionLastCode);
    }

    return { success: true, context: {
        addressList,
        cityAddressList,
    } };
};

```

* PDShopServer/project/App/routers/posts/agent/order/getRegionSendDoorAddressWithOrder.js

```js
import { OrderModel } from '../../../../models';
import { getAgentLatestOrder } from '../../libs/order';
import { getSendDoorAddressFromLastCode } from '../../libs/address';

export default async ({
    userId,
    orderId,
}) => {
    let order;
    if (orderId) {
        order = await OrderModel.findById(orderId);
    } else {
        order = await getAgentLatestOrder(userId);
    }

    const addressList = await getSendDoorAddressFromLastCode((order || {}).sendDoorEndPointLastCode);

    return { success: true, context: {
        addressList,
    } };
};

```

* PDShopServer/project/App/routers/posts/agent/order/modifyOrder.js

```js
import { OrderModel, ClientModel, MediaModel } from '../../../../models';
import { _T, hasAuthority, actionLog, omitNil, getMediaId } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { setting } from '../../../../manager';
import { setAgentOrderFee } from '../../libs/order';

export default async ({
    userId, // 收货部收货员
    orderId,
    senderPhone,
    senderName,
    receiverPhone,
    receiverName,
    name,
    photo,
    isCityDistribute, // 是否是同城配送
    endPoint, // 终点，精确到镇级别
    endPointLastCode, // 终点镇级代码
    sendDoorEndPoint, // 送货上门地址，精确到镇级别
    sendDoorEndPointLastCode, // 送货上门地址镇级代码
    totalNumbers, // 一票货总的件数
    weight, // 一票货总的重量
    size, // 一票货总的方量
    isSendDoor, // 是否送货上门
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee, // 指定向收货人应该收多少钱
    proxyCharge, // 代收货款金额
    isInsuance, //是否保价
}) => {
    const member = await ClientModel.findById(userId)
    .populate({
        path: 'agentId',
        select: { referShopId: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_PLACE_ORDER) || !member.agent.referShopId) {
        return { msg: '你没有收货的权限' };
    }

    let senderId, sender;
    if (senderPhone) {
        senderId = await ClientModel.checkExistElseCreate(senderPhone);
        if (!senderId) {
            return { msg: '系统创建账号错误，请重试' };
        }
        sender = await ClientModel.findById(senderId)
        .populate({
            path: 'shipperId',
            select: { chairManId: 1 },
        });
    }
    let receiverId;
    if (receiverPhone) {
        receiverId = await ClientModel.checkExistElseCreate(receiverPhone);
        if (!receiverId) {
            return { msg: '系统创建账号错误，请重试' };
        }
    }

    const order = await OrderModel.findById(orderId);
    if (!order) {
        return { msg: '没有该货单' };
    }
    const state = order.stateList[0].state;
    if (state >= CONSTANTS.OS_READY_PRINT_ORDER) {
        return { msg: '该货单不能被修改' };
    }
    const _photo = getMediaId(photo);
    const __photo = order.photo;
    await order.update(omitNil({
        senderId,
        senderPhone,
        senderName,
        isSenderRepresentShipper: sender && sender.shipperId && _T(sender.shipper.chairManId) === sender.id,
        receiverId,
        receiverPhone,
        receiverName,
        name,
        photo: _photo,
        isCityDistribute,
        endPoint,
        endPointLastCode,
        sendDoorEndPoint,
        sendDoorEndPointLastCode,
        totalNumbers,
        weight,
        size,
        isSendDoor,
        proxyCharge,
        proxyChargeProfit: proxyCharge !== undefined ? Math.ceil(proxyCharge * setting.proxyChargeProfitRate) : undefined,
        isReachPay,
        payMode: isReachPay !== undefined ? (isReachPay ? CONSTANTS.PM_REACH : CONSTANTS.PM_IMMEDIATE) : undefined,
        totalDesignatedFee,
        isInsuance,
        'stateList.0.count': totalNumbers || order.totalNumbers,
        modifyTime: Date.now(),
        deductError: false,
    }));

    let context = await OrderModel.findById(orderId);
    const error = await setAgentOrderFee(member.agent.referShopId, context);
    if (error) {
        context.deductError = true;
    } else if (context.stateList[0].state !== CONSTANTS.OS_READY_PRINT_BAR_CODE) {
        if (context.needPayTransportFee + context.needPayInsuanceFee) { // 需要交费的情况进入待支付状态，否则到付的情况下进入待入库状态，并且划账对接
            context.stateList = [ { state: CONSTANTS.OS_READY_PAYMENT, count: context.totalNumbers } ];
        } else {
            context.stateList = [ { state: CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER, count: context.totalNumbers } ];
        }
    }
    await context.save();

    MediaModel._updateRef(
        { [_photo]: 1 },
        { [photo && __photo]: -1 },
    );
    context = context.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'Client', orderId, 'Order', 'modifyOrder');
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/agent/order/placeOrder.js

```js
import { OrderModel, ClientModel, MediaModel } from '../../../../models';
import { _T, hasAuthority, actionLog, getMediaId } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { setting } from '../../../../manager';
import { setAgentOrderFee } from '../../libs/order';

export default async ({
    userId, // 收货点下单人员
    senderPhone,
    senderName,
    receiverPhone,
    receiverName,
    name,
    photo,
    isCityDistribute, // 是否是同城配送
    endPoint, // 终点，精确到镇级别
    endPointLastCode = 0, // 终点镇级代码
    sendDoorEndPoint, // 送货上门地址，精确到镇级别
    sendDoorEndPointLastCode = 0, // 送货上门地址镇级代码
    totalNumbers, // 一票货总的件数
    weight, // 一票货总的重量
    size, // 一票货总的方量
    isSendDoor, // 是否送货上门
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee = 0, // 指定向收货人应该收多少钱
    proxyCharge = 0, // 代收货款金额
    isInsuance, //是否保价
}) => {
    const member = await ClientModel.findById(userId)
    .populate({
        path: 'agentId',
        select: { referShopId: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_PLACE_ORDER) || !member.agent.referShopId) {
        return { msg: '你没有收货的权限' };
    }

    // 如果没有发送者或者接收者，就创建一个
    const senderId = await ClientModel.checkExistElseCreate(senderPhone);
    const receiverId = await ClientModel.checkExistElseCreate(receiverPhone);
    if (!senderId || !receiverId) {
        return { msg: '系统创建账号错误，请重试' };
    }

    const sender = await ClientModel.findById(senderId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    });

    const _photo = getMediaId(photo);
    const order = new OrderModel({
        agentId: member.agent.id,
        photo: _photo,
        senderId,
        senderName,
        senderPhone,
        isSenderRepresentShipper: sender.shipperId && _T(sender.shipper.chairManId) === _T(sender.id),
        receiverId,
        receiverPhone,
        receiverName,
        name,
        isCityDistribute,
        endPoint,
        endPointLastCode,
        sendDoorEndPoint,
        sendDoorEndPointLastCode,
        receiveAgentMemberId: userId,
        totalNumbers,
        weight,
        size,
        isSendDoor,
        proxyCharge: Math.floor(proxyCharge),
        proxyChargeProfit: Math.floor(proxyCharge * setting.proxyChargeProfitRate),
        isReachPay,
        payMode: isReachPay ? CONSTANTS.PM_REACH : CONSTANTS.PM_IMMEDIATE,
        totalDesignatedFee: Math.floor(totalDesignatedFee),
        isInsuance,
        stateList: [ { state: CONSTANTS.OS_READY_PRINT_BAR_CODE, count: totalNumbers } ],
        placeOrderTime: Date.now(),
    });

    const error = await setAgentOrderFee(member.agent.referShopId, order);
    if (error) {
        order.deductError = true;
    }
    await order.save();

    MediaModel._updateRef(
        { [_photo]: 1 },
    );
    const context = order.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'Client', order.id, 'Order', 'placeOrder');
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/agent/order/placeOrderWithPreOrder.js

```js
import { OrderModel, ClientModel, OrderGroupModel } from '../../../../models';
import { _T, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { setAgentOrderFee } from '../../libs/order';
import { setting } from '../../../../manager';

export default async ({
    userId, // 收货点下单人员
    preOrderId, // 预下单的货单号
    senderPhone,
    senderName,
    receiverPhone,
    receiverName,
    name,
    isCityDistribute, // 是否是同城配送
    endPoint, // 终点，精确到镇级别
    endPointLastCode = 0, // 终点镇级代码
    sendDoorEndPoint, // 送货上门地址，精确到镇级别
    sendDoorEndPointLastCode = 0, // 送货上门地址镇级代码
    totalNumbers, // 一票货总的件数
    weight, // 一票货总的重量
    size, // 一票货总的方量
    isSendDoor, // 是否送货上门
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee = 0, // 指定向收货人应该收多少钱
    proxyCharge = 0, // 代收货款金额
    isInsuance, //是否保价
}) => {
    const member = await ClientModel.findById(userId)
    .populate({
        path: 'agentId',
        select: { referShopId: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_PLACE_ORDER) || !member.agent.referShopId) {
        return { msg: '你没有收货的权限' };
    }

    // 如果没有发送者或者接收者，就创建一个
    const senderId = await ClientModel.checkExistElseCreate(senderPhone);
    const receiverId = await ClientModel.checkExistElseCreate(receiverPhone);
    if (!senderId || !receiverId) {
        return { msg: '系统创建账号错误，请重试' };
    }

    const sender = await ClientModel.findById(senderId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    });

    const order = await OrderModel.findByIdAndUpdate(preOrderId, {
        agentId: member.agent.id,
        senderId,
        senderName,
        senderPhone,
        isSenderRepresentShipper: sender.shipperId && _T(sender.shipper.chairManId) === _T(sender.id),
        receiverId,
        receiverPhone,
        receiverName,
        name,
        isCityDistribute,
        endPoint,
        endPointLastCode,
        sendDoorEndPoint,
        sendDoorEndPointLastCode,
        receiveAgentMemberId: userId,
        totalNumbers,
        weight,
        size,
        isSendDoor,
        proxyCharge: Math.floor(proxyCharge),
        proxyChargeProfit: Math.floor(proxyCharge * setting.proxyChargeProfitRate),
        isReachPay,
        payMode: isReachPay ? CONSTANTS.PM_REACH : CONSTANTS.PM_IMMEDIATE,
        totalDesignatedFee: Math.floor(totalDesignatedFee),
        isInsuance,
        stateList: [ { state: CONSTANTS.OS_READY_PRINT_BAR_CODE, count: totalNumbers } ],
        placeOrderTime: Date.now(),
        $unset: { groupId: 1 },
    });
    const error = await setAgentOrderFee(member.agent.referShopId, order);
    if (error) {
        order.deductError = true;
    }
    await order.save();

    const group = await OrderGroupModel.findByIdAndUpdate(order.groupId, { $inc: {
        totalNumbers: -totalNumbers,
        totalWeight: -weight,
        totalSize: -size,
        orderCount: -1,
    }, $pull: { orderIdList: preOrderId } }, { new: true });
    if (group && !group.orderIdList.length) {
        await group.remove();
    }

    actionLog(userId, 'Client', preOrderId, 'Order', 'placeOrderWithPreOrder');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/agent/order/printBarCode.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId, // 收货点下单人员
    orderId,
}) => {
    const member = await ClientModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_PLACE_ORDER)) {
        return { msg: '你没有收货的权限' };
    }
    const order = await OrderModel.findById(orderId);
    if (!order) {
        return { msg: '没有该货单' };
    }
    // if (!order.photo) {
    //     return { msg: '请先拍照上传图片' };
    // }

    if (order.needPayTransportFee + order.needPayInsuanceFee) { // 需要交费的情况进入待支付状态，否则到付的情况下进入待入库状态，并且划账对接
        order.stateList = [ { state: CONSTANTS.OS_READY_PAYMENT, count: order.totalNumbers } ];
    } else {
        order.stateList = [ { state: CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER, count: order.totalNumbers } ];
    }
    await order.save();

    let context = order.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'Client', orderId, 'Order', 'printBarCode');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/agent/order/printOrderBill.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId, // 收货点收货员
    orderId,
}) => {
    const member = await ClientModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_PLACE_ORDER)) {
        return { msg: '权限不足' };
    }
    const order = await OrderModel.findByIdAndUpdate(orderId, { 'stateList.0.state': CONSTANTS.OS_READY_SEND, modifyTime: Date.now() }, { new: true });
    if (!order) {
        return { msg: '没有该货单' };
    }
    let context = order.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'Client', orderId, 'Order', 'printOrderBill');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/agent/order/printOrderListBill.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import { hasAuthority, actionLogWithList } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderIdList,
}) => {
    const member = await ClientModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_PLACE_ORDER)) {
        return { msg: '权限不足' };
    }

    await OrderModel.update({ _id: { $in: orderIdList } }, { 'stateList.0.state': CONSTANTS.OS_READY_SEND, modifyTime: Date.now() }, { multi: true });

    actionLogWithList(userId, 'Client', orderIdList.map(o => ({ id: o, model: 'Order' })), 'printOrderListBill');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/agent/order/removeOrder.js

```js
import { OrderModel, ClientModel, MediaModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderId,
}) => {
    const client = await ClientModel.findById(userId);
    if (!hasAuthority(client, CONSTANTS.AH_PLACE_ORDER)) {
        return { msg: '你没有删除货单的权限' };
    }
    const order = await OrderModel.findByIdAndRemove(orderId);
    if (order && order.photo) {
        MediaModel._updateRef(
            { [ order.photo ]: -1 },
        );
    }

    actionLog(userId, 'Client', orderId, 'Order', 'removeOrder');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/agent/regionProfit/addRegionProfit.js

```js
import { AgentRegionProfitModel, ClientModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    region,
    regionLastCode = 0,
    profitRate,
    profitCount,
    type = 0,
}) => {
    const member = await ClientModel.findById(userId)
    .populate({
        path: 'agentId',
        select: { referShopId: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_ROADMAP_PROFIT)) {
        return { msg: '你没有修改路线的提成的权限' };
    }
    if (!member.agent.referShopId) {
        return { msg: '请先设置参考分店' };
    }
    let regionRate = await AgentRegionProfitModel.findOne({ agentId: member.agent.id, shopId: member.agent.referShopId, regionLastCode, type });
    if (regionRate) {
        return { msg: '这个方向的提成已经存在' };
    }
    regionRate = new AgentRegionProfitModel({
        agentId: member.agent.id,
        shopId: member.agent.referShopId,
        region,
        regionLastCode,
        profitRate,
        profitCount,
        type,
    });
    await regionRate.save();

    const context = await AgentRegionProfitModel.findById(regionRate.id)
    .select({
        shopId: 1,
        region: 1,
        regionLastCode: 1,
        profitRate: 1,
        profitCount: 1,
        type: 1,
        createTime: 1,
        modifyTime: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });

    actionLog(userId, 'ShopMember', regionRate.id, 'AgentRegionProfit', 'addRegionProfit');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/agent/regionProfit/getRegionAddressWithRegion.js

```js
import { AgentRegionProfitModel, ClientModel } from '../../../../models';
import { getAddressFromLastCode, getSendDoorAddressFromLastCode } from '../../libs/address';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    regionId,
}) => {
    let region;
    if (regionId) {
        region = await AgentRegionProfitModel.findById(regionId);
    }

    let addressList = [];
    let cityAddressList = [];

    const member = await ClientModel.findById(userId).populate({ path: 'agentId', select: { addressRegionLastCode: 1 } });
    const addressRegionLastCode = ((member || {}).agent || {}).addressRegionLastCode;
    if (region) {
        if (region.type === CONSTANTS.RRT_LONG) {
            addressList = await getAddressFromLastCode(region.regionLastCode);
            cityAddressList = await getSendDoorAddressFromLastCode(0, addressRegionLastCode);
        } else {
            addressList = await getAddressFromLastCode(0);
            cityAddressList = await getSendDoorAddressFromLastCode(region.regionLastCode, addressRegionLastCode);
        }
    } else {
        addressList = await getAddressFromLastCode(0);
        cityAddressList = await getSendDoorAddressFromLastCode(0, addressRegionLastCode);
    }

    return { success: true, context: {
        addressList,
        cityAddressList,
    } };
};

```

* PDShopServer/project/App/routers/posts/agent/regionProfit/getRegionProfitDetail.js

```js
import { AgentRegionProfitModel } from '../../../../models';

export default async ({ userId, regionId }) => {
    const doc = await AgentRegionProfitModel.findById(regionId)
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });
    if (!doc) {
        return { msg: '没有该设置' };
    }

    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/agent/regionProfit/getRegionProfitList.js

```js
import { AgentRegionProfitModel, ClientModel } from '../../../../models';
import { getKeywordCriteriaForRoadmapRegion } from '../../../../utils';

export default async ({ userId, keyword, fromPC, pageNo, pageSize }) => {
    const member = await ClientModel.findById(userId)
    .populate({
        path: 'agentId',
        select: { referShopId: 1 },
    });
    if (!member || !member.agent) {
        return { msg: '没有该用户' };
    }
    const criteria = getKeywordCriteriaForRoadmapRegion(keyword, { agentId: member.agent.id, shopId: member.agent.referShopId });
    const count = (fromPC && pageNo === 0) ? await AgentRegionProfitModel.count(criteria) : undefined;
    const query = AgentRegionProfitModel.find(criteria).sort({ level: 'desc', modifyTime: 'desc', createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        shopId: 1,
        region: 1,
        regionLastCode: 1,
        profitRate: 1,
        profitCount: 1,
        type: 1,
        createTime: 1,
        modifyTime: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });

    return { success: true, context: {
        count,
        regionRateList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/agent/regionProfit/modifyRegionProfit.js

```js
import { AgentRegionProfitModel, ClientModel } from '../../../../models';
import { _T, hasAuthority, actionLog, omitNil } from '../../../../utils';
import CONSTANTS from '../../../../constants';

// region 为必传参数，如果不传，则会修改为针对所有方向
// type 为必传参数
export default async ({
    userId,
    regionId,
    region,
    regionLastCode = 0,
    profitRate,
    profitCount,
    type = 0,
}) => {
    const member = await ClientModel.findById(userId)
    .populate({
        path: 'agentId',
        select: { referShopId: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_ROADMAP_PROFIT)) {
        return { msg: '你没有修改路线的提成的权限' };
    }

    let agentRegion = await AgentRegionProfitModel.findOne({ agentId: member.agent.id, shopId: member.agent.referShopId, regionLastCode, type });
    if (agentRegion && regionId !== _T(agentRegion.id)) {
        return { msg: '这个方向的提成已经存在' };
    }
    agentRegion = await AgentRegionProfitModel.findByIdAndUpdate(regionId, {
        ...omitNil({
            profitRate,
            profitCount,
        }),
        region,
        regionLastCode,
        type,
        modifyTime: Date.now(),
    }, { new: true });
    if (!agentRegion) {
        return { msg: '提成不存在' };
    }

    const context = await AgentRegionProfitModel.findById(agentRegion.id)
    .select({
        shopId: 1,
        region: 1,
        regionLastCode: 1,
        profitRate: 1,
        profitCount: 1,
        type: 1,
        createTime: 1,
        modifyTime: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });

    actionLog(userId, 'ShopMember', regionId, 'AgentRegionProfit', 'modifyRegionProfit');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/agent/regionProfit/removeRegionProfit.js

```js
import { AgentRegionProfitModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, regionId }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP_PROFIT, true)) {
        return { msg: '你没有修改路线的提成的权限' };
    }

    await AgentRegionProfitModel.findByIdAndRemove(regionId);

    actionLog(userId, 'ShopMember', regionId, 'AgentRegionProfit', 'removeRegionProfit');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/agent/regionProfit/setRegionProfitWithList.js

```js
import { AgentRegionProfitModel } from '../../../../models';
import { hasAuthority, actionLogWithList } from '../../../../utils';
import CONSTANTS from '../../../../constants';

// profitRate 为必传参数
// profitCount 为必传参数
export default async ({ userId, regionIdList, profitRate, profitCount }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP_PROFIT, true)) {
        return { msg: '你没有修改路线的提成的权限' };
    }

    await AgentRegionProfitModel.update({ _id: { $in: regionIdList } }, { profitRate, profitCount, modifyTime: Date.now() }, { multi: true });
    actionLogWithList(userId, 'ShopMember', regionIdList.map(o => ({ id: o, model: 'AgentRegionProfit' })), 'setRegionProfitWithList');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/account/getBillList.js

```js
import { ClientModel } from '../../../../models';
import { AccountModel, BillModel } from '../../../../mysql';
import { _T, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    pageNo,
    pageSize,
}) => {
    const client = await ClientModel.findById(userId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    })
    .populate({
        path: 'agentId',
        select: { chairManId: 1 },
    });
    if (!client) {
        return { msg: '没有该账户' };
    }

    let accountId;
    if (!client.shipper && !client.agent) {
        accountId = (await AccountModel.findOne({ where: { targetId: userId, type: CONSTANTS.AT_CLIENT } }) || {}).id;
    } else if (client.shipper && hasAuthority(client, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
        accountId = (await AccountModel.findOne({ where: { targetId: _T(client.shipper.chairManId), type: CONSTANTS.AT_CLIENT } }) || {}).id;
    } else if (client.agent && hasAuthority(client, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
        accountId = (await AccountModel.findOne({ where: { targetId: _T(client.agent.chairManId), type: CONSTANTS.AT_CLIENT } }) || {}).id;
    }
    if (!accountId) {
        return { msg: '你没有查看账单的权限' };
    }
    const docs = await BillModel.findAndCountAll({
        order: [['tradeTime', 'DESC']],
        where: { account: accountId },
        offset: pageNo * pageSize,
        limit: pageSize,
        attributes: [
            'tradeAmount',
            'thirdpartyAccount',
            'orderId',
            'remark',
            'tradeTime',
        ],
    });
    return { success: true, context: { count: docs.count, billList: docs.rows } };
};

```

* PDShopServer/project/App/routers/posts/client/account/getBills.js

```js
import _ from 'lodash';
import { ClientModel } from '../../../../models';
import { AccountModel, BillModel } from '../../../../mysql';
import { _T, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';
import moment from 'moment';

async function getPageData (accountId, type, params) {
    let { startDate, endDate, fromPC, pageNo, pageSize } = params;
    let options = (type === 'recharge' || type === 'withdraw') ? { thirdpartyAccount: { $ne: null } } : { thirdpartyAccount: { $eq: null } };
    let options1 = (type === 'recharge' || type === 'income') ? { tradeAmount: { $gt: 0 } } : { tradeAmount: { $lt: 0 } };

    let criteria = {};
    if (fromPC) {
        if (startDate && endDate) {
            criteria = { ...criteria, $and: [{ tradeTime: { $gte: moment(startDate).toDate() } }, { tradeTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        } else {
            startDate = endDate = moment().format('YYYY-MM-DD');
            criteria = { ...criteria, $and: [{ tradeTime: { $gte: moment(startDate).toDate() } }, { tradeTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        }
    }

    const docs = await BillModel.findAndCountAll({
        order: [['tradeTime', 'DESC']],
        where: { account: accountId, ...options, ...options1, ...criteria },
        offset: pageNo * pageSize,
        limit: pageSize,
        attributes: [
            'tradeAmount',
            'thirdpartyAccount',
            'orderId',
            'truckId',
            'remark',
            'tradeTime',
        ],
    });

    return {
        count: docs.count,
        list: docs.rows,
    };
};

// ['充值账单', '提现账单', '收入账单', '支出账单']
const TYPES = ['recharge', 'withdraw', 'income', 'pay'];
export default async ({ userId, type, ...params }) => {
    const client = await ClientModel.findById(userId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    })
    .populate({
        path: 'agentId',
        select: { chairManId: 1 },
    });
    if (!client) {
        return { msg: '没有该账户' };
    }

    let accountId;
    if (!client.shipper && !client.agent) {
        accountId = (await AccountModel.findOne({ where: { targetId: userId, type: CONSTANTS.AT_CLIENT } }) || {}).id;
    } else if (client.shipper && hasAuthority(client, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
        accountId = (await AccountModel.findOne({ where: { targetId: _T(client.shipper.chairManId), type: CONSTANTS.AT_CLIENT } }) || {}).id;
    } else if (client.agent && hasAuthority(client, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
        accountId = (await AccountModel.findOne({ where: { targetId: _T(client.agent.chairManId), type: CONSTANTS.AT_CLIENT } }) || {}).id;
    }
    if (!accountId) {
        return { msg: '你没有查看账单的权限' };
    }

    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(accountId, type, params),
            },
        };
    } else {
        return {
            success: true,
            context: {
                recharge: await getPageData(accountId, 'recharge', params),
                withdraw: await getPageData(accountId, 'withdraw', params),
                income: await getPageData(accountId, 'income', params),
                pay: await getPageData(accountId, 'pay', params),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/client/account/getRemainAmount.js

```js
import { ClientModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _T, _Y, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    let amount;
    const client = await ClientModel.findById(userId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    })
    .populate({
        path: 'agentId',
        select: { chairManId: 1 },
    });
    if (!client) {
        return { msg: '没有该账户' };
    }
    if (!client.shipper && !client.agent) {
        const clientAccount = await AccountModel.findOne({ where: { targetId: userId, type: CONSTANTS.AT_CLIENT } });
        if (!clientAccount) {
            return { msg: '没有该账户' };
        }
        amount = _Y(clientAccount.amount);
    } else if (client.shipper && hasAuthority(client, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
        const clientAccount = await AccountModel.findOne({ where: { targetId: _T(client.shipper.chairManId), type: CONSTANTS.AT_CLIENT } });
        if (!clientAccount) {
            return { msg: '没有该账户' };
        }
        amount = _Y(clientAccount.amount);
    } else if (client.agent && hasAuthority(client, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
        const clientAccount = await AccountModel.findOne({ where: { targetId: _T(client.agent.chairManId), type: CONSTANTS.AT_CLIENT } });
        if (!clientAccount) {
            return { msg: '没有该账户' };
        }
        amount = _Y(clientAccount.amount);
    }
    if (amount === undefined) {
        return { msg: '你没有查看余额的权限' };
    }

    return { success: true, context: { amount } };
};

```

* PDShopServer/project/App/routers/posts/client/account/recharge.js

```js
import { _F, actionLog } from '../../../../utils/';
import { rechargeForClient } from '../../libs/account';

export default ({
    userId,
    amount,
}) => {
    amount = _F(amount);
    const result = rechargeForClient({ userId, amount, thirdpartyAccount: 'SIMULATION_NO' });
    result.success && actionLog(userId, 'Client', amount, 'Number', 'recharge');

    return result;
};

```

* PDShopServer/project/App/routers/posts/client/account/setPaymentPassword.js

```js
import { PaymentModel, ClientModel } from '../../../../models';
import { registerUser, setPassword, checkVerifyCode, actionLog } from '../../../../utils';

export default async ({ userId, phone, password, verifyCode }) => {
    const success = await checkVerifyCode(phone, verifyCode);
    if (!success) {
        return { msg: '无效验证码' };
    }
    const client = await ClientModel.findById(userId);
    if (!client || client.phone !== phone) {
        return { msg: '获取验证码的手机号码必须为登录的手机号码' };
    }

    let payment = await PaymentModel.findOne({ userId });
    if (!payment) {
        payment = new PaymentModel({ userId });
        const error = await registerUser(PaymentModel, payment, password);
        if (error) {
            return { msg: '设置密码失败' };
        }
        client.isSetPaymentPassword = 1;
        await client.save();
        actionLog(userId, 'Client', userId, 'Client', 'setPaymentPassword');
        return { success: true };
    }

    const error = await setPassword(payment, password);
    if (error) {
        return { msg: '设置密码失败' };
    }
    await payment.save();

    actionLog(userId, 'Client', userId, 'Client', 'setPaymentPassword');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/account/withdraw.js

```js
import { _F, authenticatePaymentPassword, actionLog } from '../../../../utils/';
import { withdrawForClient } from '../../libs/account';

export default async ({
    userId,
    amount,
    payPassword,
}) => {
    const error = await authenticatePaymentPassword(userId, payPassword);
    if (error) {
        return { msg: '密码错误' };
    }
    amount = _F(amount);
    const result = withdrawForClient({ userId, amount, thirdpartyAccount: 'SIMULATION_NO' });
    result.success && actionLog(userId, 'Client', amount, 'Number', 'withdraw');

    return result;
};

```

* PDShopServer/project/App/routers/posts/client/client/getClientList.js

```js
import { ClientModel } from '../../../../models';
import { getKeywordCriteriaForUser } from '../../../../utils';

export default async ({ userId, keyword, fromPC, pageNo, pageSize }) => {
    const criteria = getKeywordCriteriaForUser(keyword);
    const query = ClientModel.find(criteria).sort({ modifyTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        phone: 1,
        name: 1,
        email: 1,
        head: 1,
        address: 1,
        location: 1,
        phoneList: 1,
    });

    return { success: true, context: {
        clientList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/client/order/finishOrder.js

```js
import _ from 'lodash';
import { ShipperModel, OrderModel, ClientModel, ShipperInBranchShopModel } from '../../../../models';
import { sequelize, AccountModel, BillModel, OrderPaymentModel } from '../../../../mysql';
import { _T, _F, _Y, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

const ERROR_HAVE_REBATED = '10000';
const ERROR_OTHER = '10004';
function payment (list) {
    return new Promise(async resolve => {
        sequelize.transaction(async (transaction) => {
            for (const item of list) {
                const { isOriginOrder, order, agentChairManId, masterProfit, proxyChargeProfit, insuanceFee, shopId, branchProfit, senderId, sendShipperId, designatedFee, proxyCharge } = item;
                const orderId = _T(order.id);
                const orderPayment = await OrderPaymentModel.findOne({ where: { orderId, isRebated: true }, transaction });
                if (orderPayment) {
                    throw ERROR_HAVE_REBATED;
                }
                if (agentChairManId) { // 收货点的货单
                    // 将手续费从收货点账号划到总部账号上
                    // 如果有指定收款和代收货款需要从收货点账号划到发货人账号（划账时不需要判断钱够不够，因为下一个货单已经会划钱到发送人即为该货单的收货点）
                    // 收货点需要扣除指定收款部分
                    const agentAmount = proxyChargeProfit + designatedFee + proxyCharge;
                    const agentAccount = await AccountModel.findOne({ where: { targetId: agentChairManId, type: CONSTANTS.AT_CLIENT, amount: { $gte: agentAmount } }, transaction });
                    await agentAccount.decrement({ amount: agentAmount }, { transaction });
                    proxyChargeProfit > 0 && await BillModel.create({ account: agentAccount.id, orderId, tradeAmount: -proxyChargeProfit, remark: '退还代收货款手续费' }, { transaction });
                    designatedFee > 0 && await BillModel.create({ account: agentAccount.id, orderId, tradeAmount: -designatedFee, remark: '退还指定收款' }, { transaction });
                    proxyCharge > 0 && await BillModel.create({ account: agentAccount.id, orderId, tradeAmount: -proxyCharge, remark: '退还代收货款' }, { transaction });

                    // 总部手续费
                    proxyChargeProfit > 0 && await AccountModel.increment({ amount: proxyChargeProfit }, { where: { id: CONSTANTS.AC_MASTER_ACCOUNT }, transaction });
                    proxyChargeProfit > 0 && await BillModel.create({ account: CONSTANTS.AC_MASTER_ACCOUNT, orderId, tradeAmount: proxyChargeProfit, remark: '代收货款手续费' }, { transaction });
                    // 发货人的指定收款部分和代收货款部分
                    const senderAccount = await AccountModel.findOne({ where: { targetId: senderId, type: CONSTANTS.AT_CLIENT }, transaction });
                    await senderAccount.increment({ amount: designatedFee + proxyCharge }, { transaction });
                    designatedFee > 0 && await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: designatedFee, remark: '指定收款' }, { transaction });
                    proxyCharge > 0 && await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: proxyCharge, remark: '代收货款' }, { transaction });
                } else {
                    // 安全账户出账
                    const securityAccount = await AccountModel.findOne({ where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
                    await securityAccount.decrement({ amount: masterProfit + proxyChargeProfit + insuanceFee + branchProfit + designatedFee + proxyCharge }, { transaction });
                    // 总部提成、手续费和保险费
                    await AccountModel.increment({ amount: masterProfit + proxyChargeProfit + insuanceFee }, { where: { id: CONSTANTS.AC_MASTER_ACCOUNT }, transaction });
                    await BillModel.create({ account: CONSTANTS.AC_MASTER_ACCOUNT, orderId, tradeAmount: masterProfit, remark: '返利总部提成' }, { transaction });
                    proxyChargeProfit > 0 && await BillModel.create({ account: CONSTANTS.AC_MASTER_ACCOUNT, orderId, tradeAmount: proxyChargeProfit, remark: '代收货款手续费' }, { transaction });
                    insuanceFee > 0 && await BillModel.create({ account: CONSTANTS.AC_MASTER_ACCOUNT, orderId, tradeAmount: insuanceFee, remark: '保险费' }, { transaction });
                    // 分部提成
                    const shopAccount = await AccountModel.findOne({ where: { targetId: shopId, type: CONSTANTS.AT_BRANCH_SHOP }, transaction });
                    await shopAccount.increment({ amount: branchProfit }, { transaction });
                    await BillModel.create({ account: shopAccount.id, orderId, tradeAmount: branchProfit, remark: '返利分店提成' }, { transaction });
                    // 发货人的指定收款部分和代收货款部分
                    const senderAccount = await AccountModel.findOne({ where: { targetId: senderId, type: CONSTANTS.AT_CLIENT }, transaction });
                    const acountAmount = _Y(senderAccount.amount + designatedFee + proxyCharge);
                    await senderAccount.increment({ amount: designatedFee + proxyCharge }, { transaction });
                    designatedFee > 0 && await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: designatedFee, remark: isOriginOrder ? '指定收款' : '退还指定收款押金' }, { transaction });
                    proxyCharge > 0 && await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: proxyCharge, remark: isOriginOrder ? '代收货款' : '退还代收货款押金' }, { transaction });

                    await OrderPaymentModel.update({ isRebated: true, rebatedTime: new Date() }, { where: { orderId }, transaction });

                    // 恢复物流公司的担保金额
                    const inShop = await ShipperInBranchShopModel.findOne({ shipperId: order.shipperId, shopId });
                    if (!inShop) {
                        throw ERROR_OTHER;
                    }
                    inShop.usedBondAmount = inShop.usedBondAmount - order.needBondAmount;
                    inShop.remainBondAmount = inShop.totalBondAmount - inShop.usedBondAmount;
                    await inShop.save();

                    // 如果发货人是物流公司，需要更新物流公司的账目
                    if (sendShipperId) {
                        const doc = await ShipperModel.findByIdAndUpdate(sendShipperId, { acountAmount }).catch(e => {
                            throw e;
                        });
                        if (!doc) {
                            throw ERROR_OTHER;
                        }
                    }
                }
            }
        }).then(() => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}
/*
* 如果需要支付，1.app支付：则调用由收货人调用 payForOrderWhenReceive，2.现金支付：由物流公司调用 finishOrder
* 如果不需要支付，由收货人调用 finishOrder
*/
export default async ({
    userId,
    orderId,
    isScan, //是否是扫描，如果是扫描，则orderId传的是原始货单的id，否则就是传的当前货单的id
}) => {
    const client = await ClientModel.findById(userId);
    if (!client) {
        return { msg: '没有该用户' };
    }
    let originOrder, currentOrder;
    if (isScan) {
        originOrder = await OrderModel.findById(orderId)
        .populate({
            path: 'senderId',
            select: { shipperId: 1 },
        })
        .populate({
            path: 'agentId',
            select: { chairManId: 1 },
        });
        if (client.shipperId) {
            currentOrder = await OrderModel.getOrderFromOriginOrder(client.shipperId, orderId, 1);
        } else {
            currentOrder = await OrderModel.getOrderFromOriginOrder(userId, orderId, 3);
        }
    } else {
        const _originOrder = await OrderModel.getOriginOrderFromNode(orderId);
        originOrder = await OrderModel.findById(_originOrder.id)
        .populate({
            path: 'senderId',
            select: { shipperId: 1 },
        })
        .populate({
            path: 'agentId',
            select: { chairManId: 1 },
        });
        currentOrder = await OrderModel.findById(orderId);
    }
    if (!originOrder || !currentOrder) {
        return { msg: '没有该货单' };
    }
    if (!(userId === _T(currentOrder.receiverId) || _T(client.shipperId) === _T(currentOrder.shipperId))) {
        return { msg: '你无权收货' };
    }
    if (_.find(currentOrder.stateList, o => o.state === CONSTANTS.OS_RECEIVE_SUCCESS)) {
        return { msg: '该货单已经确认收货' };
    }
    if (!_.find(currentOrder.stateList, o => _.includes([CONSTANTS.OS_READY_RECEIVE, CONSTANTS.OS_WAIT_CLIENT_PICK, CONSTANTS.OS_ON_THE_WAY], o.state))) {
        return { msg: '不能完成该货单' };
    }
    if (userId === currentOrder.receiverId && currentOrder.totalDesignatedFee + currentOrder.proxyCharge + currentOrder.additionalFee > 0) {
        return { msg: '该货单需要支付' };
    }

    // 收货人取货时，物流公司需要推荐收货人安装app，安装app后，扫描货单号或者进入我的货单找到该货单，需要付钱的数额 order.totalDesignatedFee + order.proxyCharge，
    // 如果数额大于0，需要显示单选框（是否线下支付，默认不选中），和按钮（立即付钱），如果勾选，按钮变成（确认收货），如果输入等于0，只显示按钮（确认收货）
    // 点确认收货，调用该接口，计算出该货单和其各个子货单的账目，这时将总部的提成从安全账户划到总部账号上, 分部的提成从安全账户划到分部账号上。将货主的钱从安全账户划到货主的账号（扣除手续费），将手续费从安全账户划到总部账号上， 将发货人的收益（指定收款部分+代收货款部分）从安全账户划到发货人的账号上
    // 如果是链式的货单，需要计算出所有的货单情况，然后使用事务统一划账。
    // 需要判断货单是否是收货点的货单，如果是，需要分别处理
    let order = originOrder;
    const list = [{
        isOriginOrder: true,
        order,
        agentChairManId: order.agentId ? _T(order.agent.chairManId) : undefined,
        masterProfit: _F(order.masterProfit),
        proxyChargeProfit: _F(order.proxyChargeProfit),
        insuanceFee: _F(order.insuanceFee),
        shopId: _T(order.shopId),
        branchProfit: _F(order.branchProfit),
        senderId: _T(order.sender.id),
        sendShipperId: order.isSenderRepresentShipper && order.sender.shipperId,
        designatedFee: _F(order.designatedFee),
        proxyCharge: _F(order.proxyCharge - order.proxyChargeProfit),
    }];
    while (order.nextOrderId) {
        order = await OrderModel.findById(order.nextOrderId)
        .populate({
            path: 'senderId',
            select: { shipperId: 1 },
        });
        list.unshift({
            order,
            masterProfit: _F(order.masterProfit),
            proxyChargeProfit: 0,
            insuanceFee: 0,
            shopId: _T(order.shopId),
            branchProfit: _F(order.branchProfit),
            senderId: _T(order.sender.id),
            sendShipperId: order.isSenderRepresentShipper && order.sender.shipperId,
            designatedFee: _F(order.designatedFee),
            proxyCharge: _F(order.proxyCharge),
        });
    }
    const err = await payment(list);
    if (err === ERROR_HAVE_REBATED) {
        for (const item of list) {
            await OrderModel.findByIdAndUpdate(item.order.id, { stateList: [ { state: CONSTANTS.OS_RECEIVE_SUCCESS, count: item.order.totalNumbers } ] });
        }
        return { msg: '该货单已经确认收货过' };
    } else if (err) {
        return { msg: '系统出错，请稍后再试' };
    }
    for (const item of list) {
        await OrderModel.findByIdAndUpdate(item.order.id, { stateList: [ { state: CONSTANTS.OS_RECEIVE_SUCCESS, count: item.order.totalNumbers } ] });
    }

    actionLog(userId, 'Client', orderId, 'Order', 'finishOrder');

    return { success: true, context: list };
};

```

* PDShopServer/project/App/routers/posts/client/order/getLogisticsList.js

```js
import { OrderModel, TruckModel } from '../../../../models';
import { getFormatTime } from '../../../../utils';

export default async ({
    userId,
    orderId,
}) => {
    let order = await OrderModel.findById(orderId)
    .populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    })
    .populate({
        path: 'agentId',
        select: { name: 1, address: 1 },
    });
    if (!order) {
        return { msg: '没有该货单' };
    }
    let logisticsList = [];
    logisticsList.push({
        shopName: (order.shop || {}).name,
        agentName: (order.agent || {}).name,
        address: (order.shop || order.agent || {}).address,
        locateTime: getFormatTime(order.placeOrderTime),
    });
    const truck = await TruckModel.findById(order.truckId);
    logisticsList = [...logisticsList, ...((truck || {}).locationList || []).map(o => ({
        address: o.address,
        locateTime: getFormatTime(o.time),
    })) ];

    while (order.nextOrderId) {
        order = await OrderModel.findById(order.nextOrderId)
        .populate({
            path: 'shopId',
            select: { name: 1, address: 1 },
        });
        logisticsList.push({
            shopName: (order.shop || {}).name,
            agentName: (order.agent || {}).name,
            address: (order.shop || order.agent || {}).address,
            locateTime: getFormatTime(order.placeOrderTime),
        });
        const truck = await TruckModel.findById(order.truckId);
        logisticsList = [...((truck || {}).locationList || []).map(o => ({
            address: o.address,
            locateTime: getFormatTime(o.time),
        })), ...logisticsList];
    }

    return { success: true, logisticsList };
};

```

* PDShopServer/project/App/routers/posts/client/order/getOrderDetail.js

```js
import { OrderModel, ClientModel } from '../../../../models';

export default async ({
    userId,
    orderId,
    isScan,
    type = 0, // type值，0：物流公司，1：发货人，2：收货人
 }) => {
    const member = await ClientModel.findById(userId);
    if (!member) {
        return { msg: '没有该用户' };
    }
    if (isScan) {
        let currentOrder;
        if (type === 0) {
            currentOrder = await OrderModel.getOrderFromOriginOrder(member.shipperId, orderId, 1);
        } else if (type === 1) {
            currentOrder = await OrderModel.getOrderFromOriginOrder(userId, orderId, 2);
        } else if (type === 2) {
            currentOrder = await OrderModel.getOrderFromOriginOrder(userId, orderId, 3);
        }
        if (!currentOrder) {
            return { msg: '没有该货单' };
        }
        orderId = currentOrder.id;
    }

    const order = await OrderModel.findById(orderId)
    .populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    })
    .populate({
        path: 'agentId',
        select: { name: 1, address: 1 },
    });
    if (!order) {
        return { msg: '没有该货单' };
    }

    const context = order.toObject();
    context.state = context.stateList[0].state;
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/client/order/getReceiveOrderList.js

```js
import { OrderModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

// type = ['全部（4-8）', '运输中（4-7）', '已完成（8）']
export default async ({ userId, type, fromPC, pageNo, pageSize }) => {
    let criteria = { receiverId: userId, isTransferOrder: false };
    if (type === 'all') {
        criteria = { ...criteria };
    } else if (type === 'unsend') {
        criteria = { ...criteria, 'stateList.state': { $gte: CONSTANTS.OS_READY_PRINT_ORDER, $lte: CONSTANTS.OS_READY_START_OFF } };
    } else if (type === 'onway') {
        criteria = { ...criteria, 'stateList.state': { $gte: CONSTANTS.OS_ON_THE_WAY, $lte: CONSTANTS.OS_ON_TRANSFER } };
    } else if (type === 'reached') {
        criteria = { ...criteria, 'stateList.state': CONSTANTS.OS_READY_RECEIVE };
    } else if (type === 'complete') {
        criteria = { ...criteria, 'stateList.state': CONSTANTS.OS_RECEIVE_SUCCESS };
    }

    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        senderPhone: 1,
        senderName: 1,
        receiverPhone: 1,
        receiverName: 1,
        photo: 1,
        realFee: 1,
        createTime: 1,
        placeOrderTime: 1,
        weight: 1,
        size: 1,
        stateList: 1,
        totalNumbers: 1,
        shopId: 1,
        agentId: 1,
        endPoint: 1,
        proxyCharge: 1,
        needPayInsuanceFee: 1,
        payMode: 1,
        isSendDoor: 1,
        isClientPick: 1,
        isReachPay: 1,
        totalDesignatedFee: 1,
        isInsuance: 1,
    }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    }).populate({
        path: 'agentId',
        select: { name: 1, address: 1 },
    });

    return { success: true, context: {
        orderList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/client/order/getReceiveOrders.js

```js
import { OrderModel } from '../../../../models';
import { getKeywordCriteriaForOrder } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { omitNil } from '../../../../utils/';
import _ from 'lodash';
import moment from 'moment';

async function getPageData (receiverId, type, params) {
    let { keyword, startDate, endDate, fromPC, pageNo, pageSize } = params;
    let criteria = {};
    if (type === 'onway') {
        criteria = { 'stateList.state': { $gte: CONSTANTS.OS_READY_STOCK, $lt: CONSTANTS.OS_READY_RECEIVE } };
    } else if (type === 'toreceive') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_RECEIVE };
    } else if (type === 'success') {
        criteria = { 'stateList.state': CONSTANTS.OS_RECEIVE_SUCCESS };
    }
    if (fromPC) {
        if (startDate && endDate) {
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        } else {
            startDate = endDate = moment().format('YYYY-MM-DD');
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        }
    }
    criteria = getKeywordCriteriaForOrder(keyword, omitNil({ ...criteria, receiverId }));

    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        shopId: 1,
        shipperId: 1,
        senderName: 1,
        senderPhone: 1,
        receiverName: 1,
        receiverPhone: 1,
        name: 1,
        endPoint: 1,
        createTime: 1,
        modifyTime: 1,
        isReachPay: 1,
        payMode: 1,
        receiveMemberId: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        stateList: 1,
        isInsuance: 1,
        insuanceMount: 1,
        insuanceFee: 1,
        fee: 1,
        profit: 1,
        masterProfit: 1,
        branchProfit: 1,
        realFee: 1,
        proxyCharge: 1,
        proxyChargeProfit: 1,
        needPayTransportFee: 1,
        needPayInsuanceFee: 1,
        totalDesignatedFee: 1,
        designatedFee: 1,
        placeOrderTime: 1,
    }).populate({
        path: 'receiveMemberId',
        select: { name: 1, phone: 1 },
    }).populate({
        path: 'shipperId',
        select: { name: 1 },
    }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });

    return {
        count,
        list: docs,
    };
};

// ['运输中', '待收货', '已完成']
const TYPES = ['onway', 'toreceive', 'success'];
export default async ({ userId, shopId, type, ...params }) => {
    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(userId, type, params),
            },
        };
    } else {
        return {
            success: true,
            context: {
                onway: await getPageData(userId, 'onway', params),
                toreceive: await getPageData(userId, 'toreceive', params),
                success: await getPageData(userId, 'success', params),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/client/order/getSendOrderList.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

// type = ['全部', '待揽货', '待发货', '运输中', '待收货']
export default async ({ userId, type, fromPC, pageNo, pageSize }) => {
    const client = await ClientModel.findById(userId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    })
    .populate({
        path: 'agentId',
        select: { chairManId: 1 },
    });
    if (!client) {
        return { msg: '没有该用户' };
    }

    let senderId = userId;
    if (client.shipper) {
        senderId = client.shipper.chairManId;
    } else if (client.agent) {
        senderId = client.agent.chairManId;
    }

    let criteria = { senderId };
    if (type === 'preorder') {
        criteria = { ...criteria, 'stateList.state': CONSTANTS.OS_PREORDER };
    } else if (type === 'waitpay') {
        criteria = { ...criteria, 'stateList.state': { $gte: CONSTANTS.OS_READY_SELECT_ROADMAP, $lte: CONSTANTS.OS_READY_PRINT_ORDER } };
    } else if (type === 'onway') {
        criteria = { ...criteria, 'stateList.state': { $gte: CONSTANTS.OS_READY_STOCK, $lte: CONSTANTS.OS_READY_RECEIVE } };
    } else if (type === 'complete') {
        criteria = { ...criteria, 'stateList.state': CONSTANTS.OS_RECEIVE_SUCCESS };
    }
    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        photo: 1,
        shopId: 1,
        agentId: 1,
        senderPhone: 1,
        senderName: 1,
        receiverPhone: 1,
        receiverName: 1,
        name: 1,
        roadmapId: 1,
        realFee: 1,
        createTime: 1,
        placeOrderTime: 1,
        weight: 1,
        size: 1,
        stateList: 1,
        totalNumbers: 1,
        endPoint: 1,
        proxyCharge: 1,
        needPayInsuanceFee: 1,
        payMode: 1,
        isSendDoor: 1,
        isClientPick: 1,
        isReachPay: 1,
        totalDesignatedFee: 1,
        isInsuance: 1,
    }).populate({
        path: 'roadmapId',
        select: { startPoint: 1, endPoint: 1 },
    }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    }).populate({
        path: 'agentId',
        select: { name: 1, address: 1 },
    });

    return { success: true, context: {
        orderList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/client/order/getSendOrders.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import { getKeywordCriteriaForOrder } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { omitNil, hasAuthority } from '../../../../utils/';
import _ from 'lodash';
import moment from 'moment';

async function getPageData (senderId, type, params) {
    let { keyword, startDate, endDate, fromPC, pageNo, pageSize } = params;
    let criteria = {};
    if (type === 'topay') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_PAYMENT };
    } else if (type === 'onway') {
        criteria = { 'stateList.state': { $gte: CONSTANTS.OS_READY_STOCK, $lt: CONSTANTS.OS_READY_RECEIVE } };
    } else if (type === 'toreceive') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_RECEIVE };
    } else if (type === 'success') {
        criteria = { 'stateList.state': CONSTANTS.OS_RECEIVE_SUCCESS };
    }
    if (fromPC) {
        if (startDate && endDate) {
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        } else {
            startDate = endDate = moment().format('YYYY-MM-DD');
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        }
    }
    criteria = getKeywordCriteriaForOrder(keyword, omitNil({ ...criteria, senderId }));

    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        shopId: 1,
        shipperId: 1,
        senderName: 1,
        senderPhone: 1,
        receiverName: 1,
        receiverPhone: 1,
        name: 1,
        endPoint: 1,
        createTime: 1,
        modifyTime: 1,
        isReachPay: 1,
        payMode: 1,
        receiveMemberId: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        stateList: 1,
        isInsuance: 1,
        insuanceMount: 1,
        insuanceFee: 1,
        fee: 1,
        profit: 1,
        masterProfit: 1,
        branchProfit: 1,
        realFee: 1,
        proxyCharge: 1,
        proxyChargeProfit: 1,
        needPayTransportFee: 1,
        needPayInsuanceFee: 1,
        totalDesignatedFee: 1,
        designatedFee: 1,
        placeOrderTime: 1,
    }).populate({
        path: 'receiveMemberId',
        select: { name: 1, phone: 1 },
    }).populate({
        path: 'shipperId',
        select: { name: 1 },
    }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });

    return {
        count,
        list: docs,
    };
};

// ['待支付', '运输中', '待收货', '已完成']
const TYPES = ['topay', 'onway', 'toreceive', 'success'];
export default async ({ userId, type, ...params }) => {
    const client = await ClientModel.findById(userId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    })
    .populate({
        path: 'agentId',
        select: { chairManId: 1 },
    });

    if (!client) {
        return { msg: '没有该用户' };
    }
    if ((client.shipper || client.agent) && !hasAuthority(client, CONSTANTS.AH_LOOK_ORDERS)) {
        return { msg: '你没有查看货单的权限' };
    }

    let senderId = userId;
    if (client.shipper) {
        senderId = client.shipper.chairManId;
    } else if (client.agent) {
        senderId = client.agent.chairManId;
    }

    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(senderId, type, params),
            },
        };
    } else {
        return {
            success: true,
            context: {
                topay: await getPageData(senderId, 'topay', params),
                onway: await getPageData(senderId, 'onway', params),
                toreceive: await getPageData(senderId, 'toreceive', params),
                success: await getPageData(senderId, 'success', params),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/client/order/payForAgentOrder.js

```js
import { sequelize, AccountModel, BillModel, OrderPaymentModel } from '../../../../mysql';
import { OrderModel } from '../../../../models';
import { _T, _F, authenticatePaymentPassword, actionLogWithList } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

const ERROR_HAVE_PAYED = '10000';
const ERROR_SENDER_MONEY_NOT_ENOUGH = '10001';
function payment (orderId, senderId, agentChairManId, amount) {
    return new Promise(async resolve => {
        sequelize.transaction(async (transaction) => {
            const orderPayment = await OrderPaymentModel.findOne({ where: { orderId, isPayed: true }, transaction });
            if (orderPayment) {
                throw ERROR_HAVE_PAYED;
            }
            // 部门的部分
            const senderAccount = await AccountModel.findOne({ where: { targetId: senderId, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } }, transaction });
            if (!senderAccount) {
                throw ERROR_SENDER_MONEY_NOT_ENOUGH;
            }
            const agentAccount = await AccountModel.findOne({ where: { targetId: agentChairManId, type: CONSTANTS.AT_CLIENT }, transaction });
            await senderAccount.decrement({ amount }, { transaction });
            await agentAccount.increment({ amount }, { transaction });
            await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: -amount, remark: '支付运费' }, { transaction });
            await BillModel.create({ account: agentAccount.id, orderId, tradeAmount: amount, remark: '收货点收到运费' }, { transaction });

            await OrderPaymentModel.create({ orderId, isPayed: true, payedTime: new Date() }, { transaction });
        }).then(() => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}

export default async ({
    userId,
    orderIdList, // 为货单列表支付
    payPassword,
}) => {
    const error = await authenticatePaymentPassword(userId, payPassword);
    if (error) {
        return { msg: '密码错误' };
    }
    const failedList = [];
    for (const orderId of orderIdList) {
        const order = await OrderModel.findById(orderId)
        .populate({
            path: 'agentId',
            select: { chairManId: 1 },
        });
        if (!order || !order.agentId) {
            failedList.push({ msg: '部分货单没找到导致部分货单未完成支付' });
            continue;
        }
        if (!_.find(order.stateList, o => o.state === CONSTANTS.OS_READY_PAYMENT)) {
            failedList.push({ msg: '部分货单不能支付' });
            continue;
        }
        if (_T(order.senderId) !== userId) {
            failedList.push({ msg: '有无权支付货单，部分货单未完成支付' });
            continue;
        }
        const err = await payment(orderId, userId, _T(order.agent.chairManId), _F(order.needPayTransportFee));
        if (err === ERROR_HAVE_PAYED) {
            order.stateList = [ { state: CONSTANTS.OS_READY_PRINT_ORDER, count: order.totalNumbers } ];
            await order.save();
            failedList.push({ msg: '有货单无需再支付，部分货单未完成支付' });
            continue;
        } else if (err === ERROR_SENDER_MONEY_NOT_ENOUGH) {
            failedList.push({ msg: '余额不足，请先充值，部分货单未完成支付' });
            continue;
        } else if (err) {
            failedList.push({ msg: '系统出错，部分货单未完成支付' });
            continue;
        }
        order.stateList = [ { state: CONSTANTS.OS_READY_PRINT_ORDER, count: order.totalNumbers } ];
        await order.save();
    }

    actionLogWithList(userId, 'Client', orderIdList.map(o => ({ id: o, model: 'Order' })), 'payForAgentOrder');

    if (failedList.length) {
        return failedList[0];
    }

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/order/payForOrderWhenReceive.js

```js
import _ from 'lodash';
import { ShipperModel, OrderModel, ShipperInBranchShopModel } from '../../../../models';
import { sequelize, AccountModel, BillModel, OrderPaymentModel } from '../../../../mysql';
import { _T, _F, _Y, actionLog, authenticatePaymentPassword } from '../../../../utils';
import CONSTANTS from '../../../../constants';

const ERROR_HAVE_REBATED = '10000';
const ERROR_RECEIVER_MONEY_NOT_ENOUGH = '10001';
const ERROR_OTHER = '10004';
function payment (list) {
    return new Promise(async resolve => {
        sequelize.transaction(async (transaction) => {
            for (const item of list) {
                const { isOriginOrder, order, agentChairManId, masterProfit, proxyChargeProfit, insuanceFee, shopId, branchProfit,
                    senderId, sendShipperId, designatedFee, proxyCharge, receiverId, shipperChairManId, shipperId, payAmount } = item;
                const orderId = _T(order.id);
                const orderPayment = await OrderPaymentModel.findOne({ where: { orderId, isRebated: true }, transaction });
                if (orderPayment) {
                    throw ERROR_HAVE_REBATED;
                }
                if (agentChairManId) { // 收货点的货单
                    // 将手续费从收货点账号划到总部账号上
                    // 如果有指定收款和代收货款需要从收货点账号划到发货人账号（划账时不需要判断钱够不够，因为下一个货单已经会划钱到发送人即为该货单的收货点）
                    // 收货点需要扣除指定收款部分
                    const agentAmount = proxyChargeProfit + designatedFee + proxyCharge;
                    const agentAccount = await AccountModel.findOne({ where: { targetId: agentChairManId, type: CONSTANTS.AT_CLIENT, amount: { $gte: agentAmount } }, transaction });
                    await agentAccount.decrement({ amount: agentAmount }, { transaction });
                    proxyChargeProfit > 0 && await BillModel.create({ account: agentAccount.id, orderId, tradeAmount: -proxyChargeProfit, remark: '退还代收货款手续费' }, { transaction });
                    designatedFee > 0 && await BillModel.create({ account: agentAccount.id, orderId, tradeAmount: -designatedFee, remark: '退还指定收款' }, { transaction });
                    proxyCharge > 0 && await BillModel.create({ account: agentAccount.id, orderId, tradeAmount: -proxyCharge, remark: '退还代收货款' }, { transaction });

                    // 总部手续费
                    proxyChargeProfit > 0 && await AccountModel.increment({ amount: proxyChargeProfit }, { where: { id: CONSTANTS.AC_MASTER_ACCOUNT }, transaction });
                    proxyChargeProfit > 0 && await BillModel.create({ account: CONSTANTS.AC_MASTER_ACCOUNT, orderId, tradeAmount: proxyChargeProfit, remark: '代收货款手续费' }, { transaction });
                    // 发货人的指定收款部分和代收货款部分
                    const senderAccount = await AccountModel.findOne({ where: { targetId: senderId, type: CONSTANTS.AT_CLIENT }, transaction });
                    await senderAccount.increment({ amount: designatedFee + proxyCharge }, { transaction });
                    designatedFee > 0 && await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: designatedFee, remark: '指定收款' }, { transaction });
                    proxyCharge > 0 && await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: proxyCharge, remark: '代收货款' }, { transaction });
                } else {
                    // 安全账户出账
                    const securityAccount = await AccountModel.findOne({ where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
                    await securityAccount.decrement({ amount: masterProfit + proxyChargeProfit + insuanceFee + branchProfit + designatedFee + proxyCharge }, { transaction });
                    // 总部提成、手续费和保险费
                    await AccountModel.increment({ amount: masterProfit + proxyChargeProfit + insuanceFee }, { where: { id: CONSTANTS.AC_MASTER_ACCOUNT }, transaction });
                    await BillModel.create({ account: CONSTANTS.AC_MASTER_ACCOUNT, orderId, tradeAmount: masterProfit, remark: '返利总部提成' }, { transaction });
                    proxyChargeProfit > 0 && await BillModel.create({ account: CONSTANTS.AC_MASTER_ACCOUNT, orderId, tradeAmount: proxyChargeProfit, remark: '代收货款手续费' }, { transaction });
                    insuanceFee > 0 && await BillModel.create({ account: CONSTANTS.AC_MASTER_ACCOUNT, orderId, tradeAmount: insuanceFee, remark: '保险费' }, { transaction });
                    // 分部提成
                    const shopAccount = await AccountModel.findOne({ where: { targetId: shopId, type: CONSTANTS.AT_BRANCH_SHOP }, transaction });
                    await shopAccount.increment({ amount: branchProfit }, { transaction });
                    await BillModel.create({ account: shopAccount.id, orderId, tradeAmount: branchProfit, remark: '返利分店提成' }, { transaction });
                    // 发货人的指定收款部分和代收货款部分
                    const senderAccount = await AccountModel.findOne({ where: { targetId: senderId, type: CONSTANTS.AT_CLIENT }, transaction });
                    const acountAmount = _Y(senderAccount.amount + designatedFee + proxyCharge);
                    await senderAccount.increment({ amount: designatedFee + proxyCharge }, { transaction });
                    designatedFee > 0 && await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: designatedFee, remark: isOriginOrder ? '指定收款' : '退还指定收款押金' }, { transaction });
                    proxyCharge > 0 && await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: proxyCharge, remark: isOriginOrder ? '代收货款' : '退还代收货款押金' }, { transaction });

                    // 将收货人要付的钱（totalDesignatedFee+proxyCharge）从收货人账号划到物流公司账号
                    let acountAmountShipper;
                    if (payAmount > 0) {
                        const receiverAccount = await AccountModel.findOne({ where: { targetId: receiverId, type: CONSTANTS.AT_CLIENT, amount: { $gte: payAmount } }, transaction });
                        if (!receiverAccount) {
                            throw ERROR_RECEIVER_MONEY_NOT_ENOUGH;
                        }
                        const shipperAccount = await AccountModel.findOne({ where: { targetId: shipperChairManId, type: CONSTANTS.AT_CLIENT }, transaction });
                        acountAmountShipper = shipperAccount.amount + payAmount;
                        await receiverAccount.decrement({ amount: payAmount }, { transaction });
                        await shipperAccount.increment({ amount: payAmount }, { transaction });
                        await BillModel.create({ account: receiverAccount.id, orderId, tradeAmount: -payAmount, remark: '支付' + (((order.totalDesignatedFee || order.additionalFee) && order.proxyCharge) ? '运费和货款' : (order.totalDesignatedFee || order.additionalFee) ? '运费' : '货款') + '给物流公司' }, { transaction });
                        await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: payAmount, remark: '收到收货人的' + (((order.totalDesignatedFee || order.additionalFee) && order.proxyCharge) ? '运费和货款' : (order.totalDesignatedFee || order.additionalFee) ? '运费' : '货款') }, { transaction });
                    }
                    await OrderPaymentModel.update({ isRebated: true, rebatedTime: new Date() }, { where: { orderId }, transaction });

                    // 恢复物流公司的担保金额
                    const inShop = await ShipperInBranchShopModel.findOne({ shipperId, shopId });
                    if (!inShop) {
                        throw ERROR_OTHER;
                    }
                    inShop.usedBondAmount = inShop.usedBondAmount - order.needBondAmount;
                    inShop.remainBondAmount = inShop.totalBondAmount - inShop.usedBondAmount;
                    await inShop.save();

                    // 如果发货人是物流公司，需要更新物流公司的账目
                    if (sendShipperId) {
                        const doc = await ShipperModel.findByIdAndUpdate(sendShipperId, { acountAmount }).catch(e => {
                            throw e;
                        });
                        if (!doc) {
                            throw ERROR_OTHER;
                        }
                    }
                    if (payAmount > 0) {
                        const doc = await ShipperModel.findByIdAndUpdate(shipperId, { acountAmount: acountAmountShipper }).catch(e => {
                            throw e;
                        });
                        if (!doc) {
                            throw ERROR_OTHER;
                        }
                    }
                }
            }
        }).then(() => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}
/*
* 如果需要支付，1.app支付：则调用由收货人调用 payForOrderWhenReceive，2.现金支付：由物流公司调用 finishOrder
* 如果不需要支付，由收货人调用 finishOrder
*/
export default async ({
    userId,
    orderId,
    payPassword,
    isScan, //是否是扫描，如果是扫描，则orderId传的是原始货单的id，否则就是传的当前货单的id
}) => {
    const error = await authenticatePaymentPassword(userId, payPassword);
    if (error) {
        return { msg: '密码错误' };
    }
    let originOrder, currentOrder;
    if (isScan) {
        originOrder = await OrderModel.findById(orderId)
        .populate({
            path: 'senderId',
            select: { shipperId: 1 },
        })
        .populate({
            path: 'agentId',
            select: { chairManId: 1 },
        });
        currentOrder = await OrderModel.getOrderFromOriginOrder(userId, orderId, 3);
    } else {
        const _originOrder = await OrderModel.getOriginOrderFromNode(orderId);
        originOrder = await OrderModel.findById(_originOrder.id)
        .populate({
            path: 'senderId',
            select: { shipperId: 1 },
        })
        .populate({
            path: 'agentId',
            select: { chairManId: 1 },
        });
        currentOrder = await OrderModel.findById(orderId);
    }
    if (!originOrder || !currentOrder) {
        return { msg: '没有该货单' };
    }
    if (userId !== _T(currentOrder.receiverId)) {
        return { msg: '你无权收货' };
    }
    if (_.find(currentOrder.stateList, o => o.state === CONSTANTS.OS_RECEIVE_SUCCESS)) {
        return { msg: '该货单已经确认收货过' };
    }
    if (!_.find(currentOrder.stateList, o => _.includes([CONSTANTS.OS_READY_RECEIVE, CONSTANTS.OS_WAIT_CLIENT_PICK, CONSTANTS.OS_ON_THE_WAY], o.state))) {
        return { msg: '该货单不能完成支付' };
    }
    // 收货人取货时，物流公司需要推荐收货人安装app，安装app后，扫描货单号或者进入我的货单找到该货单，需要付钱的数额 order.totalDesignatedFee + order.proxyCharge，
    // 如果数额大于0，需要显示单选框（是否线下支付，默认不选中），和按钮（立即付钱），如果勾选，按钮变成（确认收货），如果输入等于0，只显示按钮（确认收货）
    // 点立即支付，调用该接口，计算出该货单和其各个子货单的账目，这时将总部的提成从安全账户划到总部账号上, 分部的提成从安全账户划到分部账号上。将货主的钱从安全账户划到货主的账号（扣除手续费），将手续费从安全账户划到总部账号上，将发货人的收益从安全账户划到发货人的账号上，将货单的totalDesignatedFee+proxyCharge从收货人账号划到最后一家物流公司账号
    // 如果是链式的货单，需要计算出所有的货单情况，然后使用事务统一划账。
    // 需要判断货单是否是收货点的货单，如果是，需要分别处理
    let order = originOrder;
    const list = [{
        isOriginOrder: true,
        order,
        agentChairManId: order.agentId ? _T(order.agent.chairManId) : undefined,
        masterProfit: _F(order.masterProfit),
        proxyChargeProfit: _F(order.proxyChargeProfit),
        insuanceFee: _F(order.insuanceFee),
        shopId: _T(order.shopId),
        branchProfit: _F(order.branchProfit),
        senderId: _T(order.sender.id),
        sendShipperId: order.isSenderRepresentShipper && order.sender.shipperId,
        designatedFee: _F(order.designatedFee),
        proxyCharge: _F(order.proxyCharge - order.proxyChargeProfit),
        receiverId: _T(order.receiverId),
        shipperChairManId: _T(order.shipperChairManId),
        shipperId: _T(order.shipperId),
        payAmount: order.nextOrderId ? 0 : _F(order.totalDesignatedFee + order.proxyCharge + order.additionalFee), //末尾的个货单
    }];
    while (order.nextOrderId) {
        order = await OrderModel.findById(order.nextOrderId)
        .populate({
            path: 'senderId',
            select: { shipperId: 1 },
        });
        list.unshift({
            order,
            masterProfit: _F(order.masterProfit),
            proxyChargeProfit: 0,
            insuanceFee: 0,
            shopId: _T(order.shopId),
            branchProfit: _F(order.branchProfit),
            senderId: _T(order.sender.id),
            sendShipperId: order.isSenderRepresentShipper && order.sender.shipperId,
            designatedFee: _F(order.designatedFee),
            proxyCharge: _F(order.proxyCharge),
            receiverId: _T(order.receiverId),
            shipperChairManId: _T(order.shipperChairManId),
            shipperId: _T(order.shipperId),
            payAmount: order.nextOrderId ? 0 : _F(order.totalDesignatedFee + order.proxyCharge + order.additionalFee), //末尾的个货单
        });
    }
    const err = await payment(list);
    if (err === ERROR_HAVE_REBATED) {
        for (const item of list) {
            await OrderModel.findByIdAndUpdate(item.order.id, { stateList: [ { state: CONSTANTS.OS_RECEIVE_SUCCESS, count: item.order.totalNumbers } ] });
        }
        return { msg: '该货单已经确认收货过' };
    } else if (err === ERROR_RECEIVER_MONEY_NOT_ENOUGH) {
        return { msg: '余额不足，请先充值' };
    } else if (err) {
        return { msg: '系统出错，请稍后再试' };
    }
    for (const item of list) {
        await OrderModel.findByIdAndUpdate(item.order.id, { stateList: [ { state: CONSTANTS.OS_RECEIVE_SUCCESS, count: item.order.totalNumbers } ] });
    }

    actionLog(userId, 'Client', orderId, 'Order', 'payForOrderWhenReceive');

    return { success: true, context: list };
};

```

* PDShopServer/project/App/routers/posts/client/order/payForOrderWhenSend.js

```js
import { sequelize, AccountModel, BillModel, OrderPaymentModel } from '../../../../mysql';
import { OrderModel, ClientModel, ShipperModel, ShipperInBranchShopModel } from '../../../../models';
import { _T, _F, _Y, authenticatePaymentPassword, actionLogWithList } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

const ERROR_HAVE_PAYED = '10000';
const ERROR_SENDER_MONEY_NOT_ENOUGH = '10001';
const ERROR_SHIPPER_MONEY_NOT_ENOUGH = '10002';
const ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH = '10003';
const ERROR_OTHER = '10004';
function payment (order, sender, shipper) {
    return new Promise(async resolve => {
        const orderId = _T(order.id);
        sequelize.transaction(async (transaction) => {
            const orderPayment = await OrderPaymentModel.findOne({ where: { orderId, isPayed: true }, transaction });
            if (orderPayment) {
                throw ERROR_HAVE_PAYED;
            }
            // 发送者（物流公司/收货点）的部分
            let amount = sender.transportFee + sender.insuanceFee;
            const senderAccount = await AccountModel.findOne({ where: { targetId: sender.id, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } }, transaction });
            if (!senderAccount) {
                throw ERROR_SENDER_MONEY_NOT_ENOUGH;
            }
            const acountAmountSender = _Y(senderAccount.amount - amount);
            await senderAccount.decrement({ amount }, { transaction });
            // 保险的部分必须划到安全账户
            // 如果支付的运费大于总部和分部的提成，则只将提成部分划到安全账户，这时 deductAmount<0，这部分钱划到物流公司的账户上
            // 如果支付的运费小于总部和分部的提成，则将支付的所有的钱划到安全账户上，这时 deductAmount>0，物流公司需要将deductAmount抵押到安全账户左右总部和分部的提成
            const profit = sender.transportFee > sender.transportProfit ? sender.transportProfit : sender.transportFee;
            await AccountModel.increment({ amount: sender.insuanceFee + profit }, { where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
            sender.transportFee > 0 && await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: -sender.transportFee, remark: '支付运费' }, { transaction });
            sender.insuanceFee > 0 && await BillModel.create({ account: senderAccount.id, orderId, tradeAmount: -sender.insuanceFee, remark: '支付保险' }, { transaction });

            // 物流公司部分
            amount = shipper.proxyCharge + shipper.deductAmount;
            const shipperAccount = await AccountModel.findOne({ where: { targetId: shipper.id, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } } });
            if (!shipperAccount) {
                throw ERROR_SHIPPER_MONEY_NOT_ENOUGH;
            }
            const acountAmount = _Y(shipperAccount.amount - amount);
            amount !== 0 && await shipperAccount.decrement({ amount }, { transaction });
            // 如果shipper.deductAmount大于0，说明需要抵押运费，抵押在安全账户的钱为amount，如果小于0，说明是运费收益，这部分钱直接进物流公司账户，这是只抵押shipper.proxyCharge
            await AccountModel.increment({ amount: shipper.deductAmount > 0 ? amount : shipper.proxyCharge }, { where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
            shipper.proxyCharge > 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -shipper.proxyCharge, remark: '抵押货款' }, { transaction });
            shipper.deductAmount !== 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -shipper.deductAmount, remark: shipper.deductAmount > 0 ? '抵押指定收款' : '收取运费' }, { transaction });

            await OrderPaymentModel.create({ orderId, isPayed: true, payedTime: new Date() }, { transaction });

            // 扣除物流公司的担保金额
            const inShop = await ShipperInBranchShopModel.findOne({ shipperId: order.shipper.id, shopId: order.shopId });
            if (!inShop || inShop.remainBondAmount < order.needBondAmount) {
                throw ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH;
            }
            inShop.remainBondAmount = inShop.remainBondAmount - order.needBondAmount;
            inShop.usedBondAmount = inShop.totalBondAmount - inShop.remainBondAmount;
            await inShop.save();

            // 如果发送者是物流公司的董事长，付款成功后，需要更新该物流公司的账目
            if (sender.sendShipperId) {
                const sendShipper = await ShipperModel.findByIdAndUpdate(sender.sendShipperId, { acountAmount: acountAmountSender }).catch(e => {
                    throw e;
                });
                if (!sendShipper) {
                    throw ERROR_OTHER;
                }
            }
            // 扣款成功后，需要更新物流公司的账目
            const doc = await ShipperModel.findByIdAndUpdate(order.shipper.id, { acountAmount }).catch(e => {
                throw e;
            });
            if (!doc) {
                throw ERROR_OTHER;
            }
        }).then(() => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}

export default async ({
    userId,
    orderIdList, // 为货单列表支付
    payPassword,
}) => {
    const error = await authenticatePaymentPassword(userId, payPassword);
    if (error) {
        return { msg: '密码错误' };
    }
    const failedList = [];
    for (const orderId of orderIdList) {
        const order = await OrderModel.findById(orderId)
        .populate({
            path: 'shipperId',
            select: { chairManId: 1 },
        });
        if (!order || order.agentId) {
            failedList.push({ msg: '部分货单没找到导致部分货单未完成支付' });
            continue;
        }
        if (!_.find(order.stateList, o => o.state === CONSTANTS.OS_READY_PAYMENT)) {
            failedList.push({ msg: '有货单不能支付' });
            continue;
        }

        const client = await ClientModel.findById(userId)
        .populate({
            path: 'shipperId',
            select: { chairManId: 1 },
        })
        .populate({
            path: 'agentId',
            select: { chairManId: 1 },
        });

        let senderId = userId;
        let sendShipperId;
        if (client.shipper && _T(order.senderId) !== userId) {
            if (_T(client.shipper.chairManId) !== _T(order.sender.id)) {
                failedList.push({ msg: '有无权支付货单，部分货单未完成支付' });
                continue;
            } else {
                senderId = client.shipper.chairManId;
                sendShipperId = client.shipper.id;
            }
        } else if (client.agent && _T(order.senderId) !== userId) {
            if (_T(client.agent.chairManId) !== _T(order.sender.id)) {
                failedList.push({ msg: '有无权支付货单，部分货单未完成支付' });
                continue;
            } else {
                senderId = client.agent.chairManId;
            }
        } else if (_T(order.senderId) !== userId) {
            failedList.push({ msg: '有无权支付货单，部分货单未完成支付' });
            continue;
        }

        const err = await payment(
            order,
            { id: _T(senderId), sendShipperId, transportFee: _F(order.needPayTransportFee), insuanceFee: _F(order.needPayInsuanceFee), transportProfit: _F(order.masterProfit + order.branchProfit) },
            { id: _T(order.shipper.chairManId), proxyCharge: _F(order.proxyCharge), deductAmount: _F(order.totalDesignatedFee - order.fee) },
        );
        if (err === ERROR_HAVE_PAYED) {
            order.stateList = [ { state: CONSTANTS.OS_READY_PRINT_ORDER, count: order.totalNumbers } ];
            await order.save();
            failedList.push({ msg: '有货单无需再支付，部分货单未完成支付' });
            continue;
        } else if (err === ERROR_SENDER_MONEY_NOT_ENOUGH) {
            failedList.push({ msg: '余额不足，请先充值，部分货单未完成支付' });
            continue;
        } else if (err === ERROR_SHIPPER_MONEY_NOT_ENOUGH) {
            order.deductError = true;
            order.modifyTime = Date.now();
            await order.save();
            failedList.push({ msg: '该物流公司余额不足，请选择其他公司的路线，部分货单未完成支付' });
            continue;
        } else if (err === ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH) {
            order.deductError = true;
            order.modifyTime = Date.now();
            await order.save();
            failedList.push({ msg: '该物流公司保证金，请选择其他公司的路线，部分货单未完成支付' });
            continue;
        } else if (err) {
            failedList.push({ msg: '系统出错，部分货单未完成支付' });
            continue;
        }
        order.stateList = [ { state: CONSTANTS.OS_READY_PRINT_ORDER, count: order.totalNumbers } ];
        await order.save();
    }

    actionLogWithList(userId, 'Client', orderIdList.map(o => ({ id: o, model: 'Order' })), 'payForOrderWhenSend');

    if (failedList.length) {
        return failedList[0];
    }
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/personal/findPassword.js

```js
import { ClientModel } from '../../../../models';
import { setPassword, checkVerifyCode, actionLog } from '../../../../utils';

export default async ({ phone, password, verifyCode }) => {
    const user = await ClientModel.findOne({ phone });
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    const success = await checkVerifyCode(phone, verifyCode);
    if (!success) {
        return { msg: '无效验证码' };
    }

    const error = await setPassword(user, password);
    if (error) {
        return { msg: '服务器错误' };
    }
    await user.save();

    actionLog(user.id, 'Client', user.id, 'Client', 'findPassword');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/personal/getAgentInfo.js

```js
import { ClientModel, AgentModel } from '../../../../models';

export default async ({ userId }) => {
    const member = await ClientModel.findById(userId);
    if (!member) {
        return { msg: '没有该用户' };
    }
    const agentId = member.agentId;
    if (!agentId) {
        return { msg: '无效操作' };
    }
    const agent = await AgentModel.findById(agentId)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    })
    .populate({
        path: 'referShopId',
        select: { name: 1, address: 1 },
    });
    if (!agent) {
        return { msg: '没有该收货点' };
    }
    return { success: true, context: agent };
};

```

* PDShopServer/project/App/routers/posts/client/personal/getBranchShopInfo.js

```js
import { ShipperInBranchShopModel } from '../../../../models';
import { getOwnShipper } from '../../../../utils';

export default async ({ userId, shopId }) => {
    const shipperId = await getOwnShipper(userId);
    if (!shipperId) {
        return { msg: '你不是物流公司成员' };
    }
    const doc = await ShipperInBranchShopModel.findOne({ shipperId, shopId });
    if (!doc) {
        return { msg: '你所在的物流公司没有入住该分店' };
    }
    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/client/personal/getPersonalInfo.js

```js
import { ClientModel, ShipperInBranchShopModel, CityTruckModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _T, _Y, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default async ({ userId, fromPC }) => {
    const client = await ClientModel.findById(userId)
    .populate({
        path: 'shipperId',
        populate: [{
            path: 'chairManId',
            select: { name: 1, phone: 1 },
        }],
    }).populate({
        path: 'agentId',
        populate: [{
            path: 'chairManId',
            select: { name: 1, phone: 1 },
        }, {
            path: 'referShopId',
            select: { name: 1, address: 1 },
        }],
    }).populate({
        path: 'truckId',
        select: { plateNo: 1 },
    });
    if (!client) {
        return { msg: '没有该用户' };
    }

    const context = client.toObject();
    context.userId = context.id;
    delete context.id;

    if (context.shipper) {
        let registerShopList = await ShipperInBranchShopModel.find({ shipperId: client.shipper.id })
        .select({ shopId: 1, totalBondAmount: 1, remainBondAmount: 1 })
        .populate({
            path: 'shopId',
            select: { name: 1, address: 1, addressRegion: 1, addressRegionLastCode: 1 },
        });
        registerShopList = registerShopList.map(o => {
            o = o.toObject();
            o.shop.totalBondAmount = o.totalBondAmount;
            o.shop.remainBondAmount = o.remainBondAmount;
            return o.shop;
        });
        context.shipper.registerShopList = registerShopList;
        if (context.shipper.chairMan.id == userId) {
            context.post = '董事长';
        } else {
            context.post = '普通成员';
        }
    }

    if (!fromPC) {
        // 获取用户或者物流公司或者收货点的余额
        if (!client.shipper && !client.agent) {
            const clientAccount = await AccountModel.findOne({ where: { targetId: userId, type: CONSTANTS.AT_CLIENT } });
            if (!clientAccount) {
                return { msg: '没有该账户' };
            }
            context.remainAmount = _Y(clientAccount.amount);
        } else if (client.shipper && hasAuthority(client, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
            const clientAccount = await AccountModel.findOne({ where: { targetId: _T(client.shipper.chairMan.id), type: CONSTANTS.AT_CLIENT } });
            if (!clientAccount) {
                return { msg: '没有该账户' };
            }
            context.remainAmount = _Y(clientAccount.amount);
        } else if (client.agent && hasAuthority(client, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
            const clientAccount = await AccountModel.findOne({ where: { targetId: _T(client.agent.chairMan.id), type: CONSTANTS.AT_CLIENT } });
            if (!clientAccount) {
                return { msg: '没有该账户' };
            }
            context.remainAmount = _Y(clientAccount.amount);
        }
    }

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/client/personal/getShipperInfo.js

```js
import { ClientModel, ShipperModel, ShipperInBranchShopModel } from '../../../../models';

export default async ({ userId, shopId }) => {
    const member = await ClientModel.findById(userId);
    if (!member) {
        return { msg: '没有该用户' };
    }
    const shipperId = member.shipperId;
    if (!shipperId) {
        return { msg: '无效操作' };
    }

    let shipper = await ShipperModel.findById(shipperId)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });
    if (!shipper) {
        return { msg: '没有该物流公司' };
    }
    shipper = shipper.toObject();
    let inShop = await ShipperInBranchShopModel.findOne({ shipperId, shopId });
    if (inShop) {
        inShop = inShop.toObject();
        shipper = { ...shipper, ...inShop };
    }
    return { success: true, context: shipper };
};

```

* PDShopServer/project/App/routers/posts/client/personal/getUserNameByPhone.js

```js
import { ClientModel } from '../../../../models';

export default async ({ userId, phone }) => {
    const doc = await ClientModel.findOne({ phone });
    return { success: true, context: { name: (doc || {}).name } };
};

```

* PDShopServer/project/App/routers/posts/client/personal/login.js

```js
import { ClientModel } from '../../../../models';
import { authenticatePassword, actionLog } from '../../../../utils';

export default async ({ phone, password }) => {
    const user = await ClientModel.findOne({ phone });
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    const error = await authenticatePassword(user, password);
    if (error) {
        return { msg: '密码错误' };
    } else {
        actionLog(user.id, 'Client', user.id, 'Client', 'login');
        return { success: true, context: { userId: user.id } };
    }
};

```

* PDShopServer/project/App/routers/posts/client/personal/modifyAgentInfo.js

```js
import { AgentModel, ClientModel, MediaModel } from '../../../../models';
import { getMediaId, omitNil, hasAuthority, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    image, // 收货点背景图片
    sign, // 收货点签名
    phoneList, // 收货点联系电话
    address, // 收货点地址
    location, // 经纬度定位
}) => {
    const client = await ClientModel.findById(userId);
    if (!hasAuthority(client, CONSTANTS.AH_MODIFY_AGENT_INFO)) {
        return { msg: '你没有修改收货点信息的权限' };
    }

    let _image = getMediaId(image);

    let agent = await AgentModel.findByIdAndUpdate(client.agentId, omitNil({
        image: _image,
        sign,
        phoneList,
        address,
        location,
    }));
    if (!agent) {
        return { msg: '修改失败' };
    }

    MediaModel._updateRef(
        { [_image]: 1 },
        { [image && agent.image]: -1 },
    );

    agent = await AgentModel.findById(agent.id)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });

    actionLog(userId, 'Client', agent.id, 'Agent', 'modifyAgentInfo');

    return { success: true, context: agent };
};

```

* PDShopServer/project/App/routers/posts/client/personal/modifyPassword.js

```js
import { ClientModel } from '../../../../models';
import { setPassword, authenticatePassword, actionLog } from '../../../../utils';

export default async ({ userId, oldPassword, newPassword }) => {
    const user = await ClientModel.findById(userId);
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    let error = await authenticatePassword(user, oldPassword);
    if (error) {
        return { msg: '密码错误' };
    }
    error = await setPassword(user, newPassword);
    if (error) {
        return { msg: '设置密码失败' };
    }
    await user.save();

    actionLog(userId, 'Client', '', '', 'modifyPassword');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/personal/modifyPersonalInfo.js

```js
import { ClientModel, MediaModel } from '../../../../models';
import { getMediaId, omitNil, actionLog } from '../../../../utils';
import moment from 'moment';

const YEAR_TIME = 31536000000;

export default async ({
    userId,
    name,
    email,
    sex,
    address,
    head,
    birthday,
    phoneList,
}) => {
    let _head = getMediaId(head);
    let age = birthday ? Math.floor(moment().diff(moment(birthday)) / YEAR_TIME) : undefined;
    const doc = await ClientModel.findByIdAndUpdate(userId, omitNil({
        name,
        email,
        sex,
        address,
        head: _head,
        birthday,
        age,
        phoneList,
    }));
    if (!doc) {
        return { msg: '修改失败' };
    }
    if (_head) {
        MediaModel._updateRef(
            { [_head]: 1 },
            { [head && doc.head]: -1 },
        );
    }

    let context = await ClientModel.findById(userId)
    .populate({
        path: 'shipperId',
        populate: [{
            path: 'chairManId',
            select: { name: 1, phone: 1 },
        }],
    }).populate({
        path: 'agentId',
        populate: [{
            path: 'chairManId',
            select: { name: 1, phone: 1 },
        }],
    });

    context = context.toObject();
    context.userId = context.id;
    delete context.id;

    actionLog(userId, 'Client', userId, 'Client', 'modifyPersonalInfo');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/client/personal/modifyShipperInfo.js

```js
import { ShipperModel, ClientModel, ShipperInBranchShopModel, MediaModel } from '../../../../models';
import { getMediaId, omitNil, hasAuthority, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    shopId, // 所在分店
    image, // 物流公司背景图片
    sign, // 物流公司签名
    phoneList, // 物流公司联系电话
    address, // 物流公司地址
    clientPickPrice, // 自提价格 (只有 shipperType === 1 的物流公司需要设置)
    clientPickEnable, // 是否可用 (默认为不可用，只有物流公司设置了价格后才生效)
}) => {
    const member = await ClientModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_SHIPPER_INFO)) {
        return { msg: '你没有修改物流公司信息的权限' };
    }
    const shipperId = member.shipperId;
    if (!shipperId) {
        return { msg: '无效操作' };
    }

    let _image = getMediaId(image);

    let shipper = await ShipperModel.findByIdAndUpdate(shipperId, omitNil({
        image: _image,
        sign,
        phoneList,
        address,
    }));
    if (!shipper) {
        return { msg: '修改失败' };
    }
    if (!_.isNil(clientPickPrice) || !_.isNil(clientPickEnable)) {
        await ShipperInBranchShopModel.findOneAndUpdate({ shipperId, shopId }, omitNil({
            clientPickPrice,
            clientPickEnable,
        }));
    }

    MediaModel._updateRef(
        { [_image]: 1 },
        { [image && shipper.image]: -1 },
    );

    shipper = await ShipperModel.findById(shipper.id)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });
    shipper = shipper.toObject();
    let inShop = await ShipperInBranchShopModel.findOne({ shipperId, shopId });
    if (inShop) {
        inShop = inShop.toObject();
        shipper = { ...shipper, ...inShop };
    }

    actionLog(userId, 'Client', shipper.id, 'Shipper', 'modifyShipperInfo');

    return { success: true, context: shipper };
};

```

* PDShopServer/project/App/routers/posts/client/personal/register.js

```js
import { ClientModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _T, setPassword, registerUser, checkVerifyCode, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ phone, password, verifyCode }) => {
    const success = await checkVerifyCode(phone, verifyCode);
    if (!success) {
        return { msg: '无效验证码' };
    }

    let user = await ClientModel.findOne({ phone });
    if (!user) {
        user = new ClientModel({
            phone,
            registerTime: Date.now(),
        });
        const account = await AccountModel.create({ targetId: _T(user.id), type: CONSTANTS.AT_CLIENT });
        if (!account) {
            return { msg: '注册失败' };
        }
        let error = await registerUser(ClientModel, user, password);
        if (error) {
            return { msg: '注册失败' };
        }

        actionLog(user.id, 'Client', user.id, 'Client', 'register');

        return { success: true };
    } else if (!user.registerTime) {
        let error = await setPassword(user, password);
        if (error) {
            return { msg: '注册失败' };
        }
        user.registerTime = Date.now();
        await user.save();

        actionLog(user.id, 'Client', user.id, 'Client', 'register');

        return { success: true };
    }

    return { msg: '该账号已经被占用' };
};

```

* PDShopServer/project/App/routers/posts/client/personal/submitFeedback.js

```js
import { FeedbackModel } from '../../../../models';
import { actionLog } from '../../../../utils';

export default async ({ userId, content, email }) => {
    const feedback = new FeedbackModel({ clientId: userId, content, email });
    await feedback.save();

    actionLog(userId, 'Client', userId, 'Client', 'submitFeedback');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/preOrder/createPreOrderGroup.js

```js
import { OrderModel, OrderGroupModel } from '../../../../models';
import { actionLog } from '../../../../utils';

export default async ({
    userId,
    startPoint,
    startPointLastCode,
    shopId,
    agentId,
    groupName, // 如果是合并，groupName应该为第一个group的名字，这里由客户端判断
    orderGroupIdList = [], // 批次列表
    orderIdList = [], // 货单列表
}) => {
    let senderId, firstGroup, totalNumbers = 0, totalWeight = 0, totalSize = 0, orderCount = 0;

    if (orderIdList.length) {
        const orderList = await OrderModel.find({ _id: { $in: orderIdList } });
        for (const order of orderList) {
            !senderId && (senderId = order.senderId);
            totalNumbers += order.totalNumbers;
            totalWeight += order.weight;
            totalSize += order.size;
            orderCount++;
        }
    }
    if (orderGroupIdList.length) {
        const orderGroupList = await OrderGroupModel.find({ _id: { $in: orderGroupIdList } });
        for (const group of orderGroupList) {
            if (!firstGroup) {
                firstGroup = group;
                !senderId && (senderId = group.senderId);
            } else {
                await group.remove();
            }
            orderIdList = orderIdList.concat(group.orderIdList);
            totalNumbers += group.totalNumbers;
            totalWeight += group.totalNumbers;
            totalSize += group.totalSize;
            orderCount += group.orderCount;
        }
    }

    if (!firstGroup) {
        firstGroup = new OrderGroupModel({ name: groupName, senderId });
    } else {
        firstGroup.name = groupName;
        firstGroup.modifyTime = Date.now();
    }
    firstGroup.startPoint = startPoint;
    firstGroup.startPointLastCode = startPointLastCode;
    firstGroup.shopId = shopId;
    firstGroup.agentId = agentId;
    firstGroup.totalNumbers = totalNumbers;
    firstGroup.totalWeight = totalWeight;
    firstGroup.totalSize = totalSize;
    firstGroup.orderCount = orderCount;
    firstGroup.orderIdList = orderIdList;

    await firstGroup.save();
    await OrderModel.update({ _id: { $in: orderIdList } }, { groupId: firstGroup.id }, { multi: true });

    actionLog(userId, 'Client', firstGroup.id, 'OrderGroup', 'createPreOrderGroup');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/preOrder/getPreOrderGroupList.js

```js
import { OrderGroupModel, ClientModel } from '../../../../models';

export default async ({ userId, fromPC, pageNo, pageSize }) => {
    const sender = await ClientModel.findById(userId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
        populate: [{
            path: 'chairManId',
            select: { phone: 1 },
        }],
    })
    .populate({
        path: 'agentId',
        select: { chairManId: 1 },
        populate: [{
            path: 'chairManId',
            select: { phone: 1 },
        }],
    });
    if (!sender) {
        return { msg: '没有该用户' };
    }
    const senderId = sender.shipper ? sender.shipper.chairMan.id : sender.agent ? sender.agent.chairMan.id : userId; // 物流公司的人员预下单以物流公司的董事长为发货人
    const orderGroupList = await OrderGroupModel.find({ senderId }).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize)
    .select({
        name: 1,
        totalNumbers: 1,
        totalWeight: 1,
        totalSize: 1,
        orderCount: 1,
        shopId: 1,
        agentId: 1,
        startPoint: 1,
        startPointLastCode: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    })
    .populate({
        path: 'agentId',
        select: { name: 1 },
    });

    return { success: true, context: {
        orderGroupList,
    } };
};

```

* PDShopServer/project/App/routers/posts/client/preOrder/getPreOrderList.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({ userId, fromPC, pageNo, pageSize }) => {
    const sender = await ClientModel.findById(userId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    });
    if (!sender) {
        return { msg: '没有该用户' };
    }
    const senderId = sender.shipperId ? sender.shipper.chairManId : userId; // 物流公司的人员预下单以物流公司的董事长为发货人
    const criteria = { senderId, 'stateList.state': CONSTANTS.OS_PREORDER, groupId: { $exists: false } };
    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        senderPhone: 1,
        senderName: 1,
        receiverPhone: 1,
        receiverName: 1,
        name: 1,
        isCityDistribute: 1,
        endPoint: 1,
        endPointLastCode: 1,
        sendDoorEndPoint: 1,
        sendDoorEndPointLastCode: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        proxyCharge: 1,
        isReachPay: 1,
        totalDesignatedFee: 1,
        isSendDoor: 1,
        isInsuance: 1,
        startPoint: 1,
        startPointLastCode: 1,
        shopId: 1,
        agentId: 1,
        photo: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    })
    .populate({
        path: 'agentId',
        select: { name: 1 },
    });

    return { success: true, context: {
        count,
        orderList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/client/preOrder/getPreOrderListInGroup.js

```js
import { OrderGroupModel } from '../../../../models';

export default async ({ userId, orderGroupId }) => {
    const group = await OrderGroupModel.findById(orderGroupId)
    .populate({
        path: 'orderIdList',
        select: {
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            name: 1,
            isCityDistribute: 1,
            endPoint: 1,
            endPointLastCode: 1,
            sendDoorEndPoint: 1,
            sendDoorEndPointLastCode: 1,
            totalNumbers: 1,
            weight: 1,
            size: 1,
            proxyCharge: 1,
            isReachPay: 1,
            totalDesignatedFee: 1,
            isSendDoor: 1,
            isInsuance: 1,
            startPoint: 1,
            startPointLastCode: 1,
            shopId: 1,
            agentId: 1,
        },
        populate: [{
            path: 'shopId',
            select: { name: 1 },
        }, {
            path: 'agentId',
            select: { name: 1 },
        }],
    });
    if (!group) {
        return { msg: '没有该批次' };
    }

    return { success: true, context: {
        orderList: group.orderIdList,
    } };
};

```

* PDShopServer/project/App/routers/posts/client/preOrder/modifyPreOrder.js

```js
import _ from 'lodash';
import { OrderModel, OrderGroupModel, ClientModel, MediaModel } from '../../../../models';
import { getMediaId, omitNil, actionLog } from '../../../../utils';
import { setting } from '../../../../manager';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderId,
    orderGroupId,
    senderName,
    receiverPhone,
    receiverName,
    name,
    shopId, // 指定的某家分店 (必传参数)
    agentId, // 指定的某家收货点(必传参数)
    startPoint, // 起点
    startPointLastCode, // 起点 lastCode
    isCityDistribute, // 是否是同城配送
    endPoint, // 终点，精确到镇级别
    endPointLastCode, // 终点镇级代码
    sendDoorEndPoint,
    sendDoorEndPointLastCode,
    totalNumbers, // 一票货总的件数
    weight, // 一票货总的重量
    size, // 一票货总的方量
    isSendDoor, // 是否送货上门
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee, // 指定向收货人应该收多少钱
    proxyCharge, // 代收货款金额
    isInsuance, // 是否保价
    photo, // 图片
}) => {
    let receiverId;
    if (receiverPhone) {
        receiverId = await ClientModel.checkExistElseCreate(receiverPhone, receiverName);
        if (!receiverId) {
            return { msg: '系统创建账号错误，请重试' };
        }
    }
    const _startPoint = (shopId || agentId || startPointLastCode) ?
    { shopId: undefined, agentId: undefined, startPoint, startPointLastCode, ...(shopId ? { shopId } : agentId ? { agentId } : {}) } :
    {};

    const _photo = getMediaId(photo);
    const doc = await OrderModel.findOneAndUpdate({ _id: orderId, 'stateList.state': CONSTANTS.OS_PREORDER }, {
        ..._startPoint,
        ...omitNil({
            senderName,
            receiverId,
            receiverPhone,
            receiverName,
            name,
            isCityDistribute,
            endPoint,
            endPointLastCode,
            sendDoorEndPoint,
            sendDoorEndPointLastCode,
            totalNumbers,
            weight,
            size,
            isSendDoor,
            proxyCharge,
            proxyChargeProfit: proxyCharge !== undefined ? Math.ceil(proxyCharge * setting.proxyChargeProfitRate) : undefined,
            isReachPay,
            payMode: isReachPay !== undefined ? (isReachPay ? CONSTANTS.PM_REACH : CONSTANTS.PM_IMMEDIATE) : undefined,
            totalDesignatedFee,
            isInsuance,
            photo: _photo,
        }),
    });
    if (!doc) {
        return { msg: '修改失败' };
    }
    if (orderGroupId) {
        const options = _.omitBy({ totalNumbers: totalNumbers - doc.totalNumbers, weight: weight - doc.weight, size: size - doc.size }, o => !o);
        if (_.size(options)) {
            await OrderGroupModel.findByIdAndUpdate(orderGroupId, { $inc: options });
        }
    }

    MediaModel._updateRef(
        { [_photo]: 1 },
        { [photo && doc.photo]: -1 },
    );
    actionLog(userId, 'Client', orderId, 'Order', 'modifyPreOrder');

    const context = await OrderModel.findById(orderId)
    .select({
        senderPhone: 1,
        senderName: 1,
        receiverPhone: 1,
        receiverName: 1,
        name: 1,
        isCityDistribute: 1,
        endPoint: 1,
        endPointLastCode: 1,
        sendDoorEndPoint: 1,
        sendDoorEndPointLastCode: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        proxyCharge: 1,
        isReachPay: 1,
        totalDesignatedFee: 1,
        isSendDoor: 1,
        isInsuance: 1,
        startPoint: 1,
        startPointLastCode: 1,
        shopId: 1,
        agentId: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    })
    .populate({
        path: 'agentId',
        select: { name: 1 },
    });

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/client/preOrder/modifyPreOrderGroup.js

```js
import { OrderGroupModel } from '../../../../models';
import { omitNil, actionLog } from '../../../../utils';

export default async ({
    userId,
    orderGroupId,
    startPoint,
    startPointLastCode,
    shopId,
    agentId,
    groupName,
}) => {
    const _startPoint = (shopId || agentId || startPointLastCode) ?
    { shopId: undefined, agentId: undefined, startPoint, startPointLastCode, ...(shopId ? { shopId } : agentId ? { agentId } : {}) } :
    {};

    const group = await OrderGroupModel.findByIdAndUpdate(orderGroupId, {
        ..._startPoint,
        ...omitNil({
            name: groupName,
        }),
    });
    if (!group) {
        return { msg: '没有该货单批次' };
    }
    actionLog(userId, 'Client', orderGroupId, 'OrderGroup', 'modifyPreOrderGroup');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/preOrder/placePreOrder.js

```js
import { OrderModel, ClientModel, MediaModel } from '../../../../models';
import { actionLog, getMediaId } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { setting } from '../../../../manager';

export default async ({
    userId,
    senderName,
    receiverPhone,
    receiverName,
    name,
    shopId, // 指定的某家分店
    agentId, // 指定的某家收货点
    startPoint, // 起点
    startPointLastCode, // 起点 lastCode
    isCityDistribute, // 是否是同城配送
    endPoint, // 终点，精确到镇级别
    endPointLastCode, // 终点镇级代码
    sendDoorEndPoint, // 送货上门地址，精确到镇级别
    sendDoorEndPointLastCode, // 送货上门地址镇级代码
    totalNumbers, // 一票货总的件数
    weight, // 一票货总的重量
    size, // 一票货总的方量
    isSendDoor, // 是否送货上门
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee = 0, // 指定向收货人应该收多少钱
    proxyCharge = 0, // 代收货款金额
    isInsuance, // 是否保价
    photo, // 图片
}) => {
    const receiverId = await ClientModel.checkExistElseCreate(receiverPhone, receiverName);
    if (!receiverId) {
        return { msg: '系统创建账号错误，请重试' };
    }

    const _photo = getMediaId(photo);
    const sender = await ClientModel.findById(userId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
        populate: [{
            path: 'chairManId',
            select: { phone: 1 },
        }],
    })
    .populate({
        path: 'agentId',
        select: { chairManId: 1 },
        populate: [{
            path: 'chairManId',
            select: { phone: 1 },
        }],
    });
    const senderId = sender.shipper ? sender.shipper.chairMan.id : sender.agent ? sender.agent.chairMan.id : userId; // 物流公司的人员预下单以物流公司的董事长为发货人
    const senderPhone = sender.shipper ? sender.shipper.chairMan.phone : sender.agent ? sender.agent.chairMan.phone : sender.phone; // 物流公司的人员预下单以物流公司的董事长为发货人
    const doc = new OrderModel({
        shopId,
        agentId,
        startPoint,
        startPointLastCode,
        senderId,
        senderPhone,
        senderName,
        isSenderRepresentShipper: !!sender.shipper,
        receiverId,
        receiverPhone,
        receiverName,
        name,
        isCityDistribute,
        endPoint,
        endPointLastCode,
        sendDoorEndPoint,
        sendDoorEndPointLastCode,
        totalNumbers,
        weight,
        size,
        isSendDoor,
        proxyCharge: Math.floor(proxyCharge),
        proxyChargeProfit: Math.floor(proxyCharge * setting.proxyChargeProfitRate),
        isReachPay,
        payMode: isReachPay ? CONSTANTS.PM_REACH : CONSTANTS.PM_IMMEDIATE,
        totalDesignatedFee: Math.floor(totalDesignatedFee),
        isInsuance,
        stateList: [ { state: CONSTANTS.OS_PREORDER, count: totalNumbers } ],
        photo: _photo,
    });
    await doc.save();
    MediaModel._updateRef(
        { [_photo]: 1 },
    );
    actionLog(userId, 'Client', doc.id, 'Order', 'placePreOrder');

    const context = await OrderModel.findById(doc.id)
    .select({
        senderPhone: 1,
        senderName: 1,
        receiverPhone: 1,
        receiverName: 1,
        name: 1,
        isCityDistribute: 1,
        endPoint: 1,
        endPointLastCode: 1,
        sendDoorEndPoint: 1,
        sendDoorEndPointLastCode: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        proxyCharge: 1,
        isReachPay: 1,
        totalDesignatedFee: 1,
        isSendDoor: 1,
        isInsuance: 1,
        startPoint: 1,
        startPointLastCode: 1,
        shopId: 1,
        agentId: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    })
    .populate({
        path: 'agentId',
        select: { name: 1 },
    });

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/client/preOrder/removePreOrder.js

```js
import { OrderModel } from '../../../../models';
import { actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderId,
}) => {
    await OrderModel.findOneAndRemove({ _id: orderId, 'stateList.state': CONSTANTS.OS_PREORDER });

    actionLog(userId, 'Client', orderId, 'Order', 'removePreOrder');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/preOrder/removePreOrderFromGroup.js

```js
import { OrderModel, OrderGroupModel } from '../../../../models';
import { actionLog } from '../../../../utils';

export default async ({ userId, orderIdList, orderGroupId }) => {
    const orderList = await OrderModel.find({ _id: { $in: orderIdList } });
    let totalNumbers = 0, totalWeight = 0, totalSize = 0;
    for (const order of orderList) {
        totalNumbers += order.totalNumbers;
        totalWeight += order.weight;
        totalSize += order.size;
    }

    await OrderModel.update({ _id: { $in: orderIdList } }, { $unset: { groupId: 1 } }, { multi: true });
    const group = await OrderGroupModel.findByIdAndUpdate(orderGroupId, { $inc: {
        totalNumbers: -totalNumbers,
        totalWeight: -totalWeight,
        totalSize: -totalSize,
        orderCount: -1,
    }, $pull: { orderIdList: { $in: orderIdList } } }, { new: true });
    if (!group) {
        return { msg: '没有该货单批次' };
    }

    if (!group.orderIdList.length) {
        await group.remove();
    }

    actionLog(userId, 'Client', orderGroupId, 'OrderGroup', 'removePreOrderFromGroup');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/client/roadmap/getRoadmapDetail.js

```js
import { RoadmapModel } from '../../../../models';

export default async ({ userId, roadmapId }) => {
    const context = await RoadmapModel.findById(roadmapId).select({
        name: 1,
        shipperId: 1,
        startPoint: 1,
        endPoint: 1,
        transitPoint: 1,
        type: 1,
        price: 1,
        duration: 1,
        truckId: 1,
        remark: 1,
        modifyTime: 1,
    }).populate({
        path: 'truckId',
        select: {
            name: 1,
            plateNo: 1,
            capacity: 1,
            width: 1,
            height: 1,
            length: 1,
            remark: 1,
        },
    });
    if (!context) {
        return { msg: '没有该线路' };
    }

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/client/roadmap/getRoadmapListWithEndPoint.js

```js
import mongoose from 'mongoose';
import { RoadmapModel, ShopModel, AgentModel, AgentRegionProfitModel } from '../../../../models';
import { getRegionProfitRateByEndPoint } from '../../libs/roadmap';
import { _N } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    fromPC,
    startPointLastCode, // 始发地的lastCode，如果有 agentId 或者 shopId，该字段无效
    shopId, // 分店Id
    agentId, // 收货点Id
    endPointLastCode, // 终点code
    sendDoorEndPointLastCode, // 送货上门终点code
    isCityDistribute, // 是否送货上门
    isSendDoor, // 是否送货上门
    location, // 我的位置
    orderBy, // 排序方式，0：价格排序，1：时间排序，2：亲签率排序，3：距离排序
    pageNo,
    pageSize,
}) => {
    let sort = { price: 1, duration: 1, selfSignRate: -1 };
    if (orderBy === 1) {
        sort = { duration: 1, price: 1, selfSignRate: -1 };
    } else if (orderBy === 2) {
        sort = { selfSignRate: -1, price: 1, duration: 1 };
    }
    // 始发地为分店的情况
    if (shopId) {
        const shopList = await ShopModel.aggregate()
        .near({
            near: location,
            distanceField: 'dist.calculated',
        })
        .match({ _id: new mongoose.Types.ObjectId(shopId) })
        .project({
            distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
            name: 1,
            address: 1,
        });

        const count = (fromPC && pageNo === 0) ? await RoadmapModel.count({ shopId, endPointLastCode, enable: true }) : undefined;
        const shop = shopList[0];
        if (!shop) {
            return { msg: '没有该分店' };
        }
        const regionProfitRate = await getRegionProfitRateByEndPoint(shopId, endPointLastCode, isCityDistribute);

        const roadmaps = await RoadmapModel.aggregate()
        .match({ shopId: new mongoose.Types.ObjectId(shopId), endPointLastCode, enable: true })
        .project({
            rate: { $sum: [ 1, { $ifNull: [ '$profitRate', regionProfitRate ] } ] },
            sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
            shipperId: 1,
            price: 1,
            minFee: 1,
            selfSignRate: 1,
            duration: 1,
        })
        .project({
            sendDoorEnable: '$sendDoor.enable',
            sendDoorPrice: '$sendDoor.sendDoorPrice',
            sendDoorMinFee: '$sendDoor.sendDoorMinFee',
            shipperId: 1,
            rate: 1,
            price: 1,
            minFee: 1,
            selfSignRate: 1,
            duration: 1,
        })
        .match(isSendDoor ? { sendDoorEnable: true } : {})
        .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: '_shipper' })
        .project({
            id: '$_id',
            _id: 0,
            shipperName: { $arrayElemAt: ['$_shipper.name', 0] },
            price: { $multiply: [ '$rate', '$price' ] },
            minFee: { $multiply: [ '$rate', '$minFee' ] },
            sendDoorPrice: { $multiply: [ '$rate', '$sendDoorPrice' ] },
            sendDoorMinFee: { $multiply: [ '$rate', '$sendDoorMinFee' ] },
            shopName: { $concat: shop.name },
            shopAddress: { $concat: shop.address },
            distance: { $abs: shop.distance },
            selfSignRate: 1,
            duration: 1,
        }).sort(sort)
        .skip(pageNo * pageSize)
        .limit(pageSize);

        return { success: true, context: { shop: {
            count,
            roadmapList: roadmaps,
        } } };
    };

    // 始发地为收货点的情况
    if (agentId) {
        const agents = await AgentModel.aggregate()
        .near({
            near: location,
            distanceField: 'dist.calculated',
        })
        .match({ _id: new mongoose.Types.ObjectId(agentId) })
        .project({
            distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
            name: 1,
            address: 1,
            referShopId: 1,
            roadmapProfitRate: 1,
        });
        if (!agents.length) {
            return { msg: '没有相关收货点' };
        }
        const agent = agents[0];
        const regionProfitRate = await getRegionProfitRateByEndPoint(agent.referShopId, endPointLastCode, isCityDistribute);

        const roadmaps = await RoadmapModel.aggregate()
        .match({ shopId: agent.referShopId, endPointLastCode, enable: true })
        .project({
            sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
            rate: { $sum: [ 1, { $ifNull: [ '$profitRate', regionProfitRate ] } ] },
            shipperId: 1,
            price: 1,
            minFee: 1,
            selfSignRate: 1,
            duration: 1,
        })
        .project({
            sendDoorEnable: '$sendDoor.enable',
            sendDoorPrice: '$sendDoor.sendDoorPrice',
            sendDoorMinFee: '$sendDoor.sendDoorMinFee',
            shipperId: 1,
            rate: 1,
            price: 1,
            minFee: 1,
            selfSignRate: 1,
            duration: 1,
        })
        .match(isSendDoor ? { sendDoorEnable: true } : {})
        .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
        .project({
            id: '$_id',
            _id: 0,
            shipperName: { $arrayElemAt: ['$shipper.name', 0] },
            price: { $multiply: [ '$rate', '$price' ] },
            minFee: { $multiply: [ '$rate', '$minFee' ] },
            sendDoorPrice: { $multiply: [ '$rate', '$sendDoorPrice' ] },
            sendDoorMinFee: { $multiply: [ '$rate', '$sendDoorMinFee' ] },
            selfSignRate: 1,
            duration: 1,
        })
        .sort(sort)
        .limit(1);

        const roadmapList = [];
        const roadmap = roadmaps[0];
        if (roadmap) {
            const rateModel = await AgentRegionProfitModel.findOne({ agentId: agent._id, roadmapId: roadmap.id });
            const rate = Math.max((rateModel && rateModel.profitRate || 0), agent.roadmapProfitRate) + 1;
            roadmap.price = _N(roadmap.price * rate);
            roadmap.minFee = _N(roadmap.minFee * rate);
            roadmap.distance = agent.distance;
            roadmap.agentName = agent.name;
            roadmap.agentAddress = agent.address;
            roadmap.shipperName = roadmap.shipperName;

            roadmapList.push(roadmap);
        }

        return { success: true, context: { agent: { roadmapList } } };
    }

    // 始发地为城市范围的情况
    const shopList = await ShopModel.aggregate()
    .near({
        near: location,
        distanceField: 'dist.calculated',
    })
    .match({ addressRegionLastCode: startPointLastCode, isMasterShop: false })
    .project({
        distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
        name: 1,
        address: 1,
    });
    for (const shop of shopList) {
        shop.regionProfitRate = await getRegionProfitRateByEndPoint(shop.id, endPointLastCode, isCityDistribute);
    }

    const count = (fromPC && pageNo === 0) ? await RoadmapModel.count({ shopId: { $in: shopList.map(o => o._id) }, endPointLastCode, enable: true }) : undefined;
    const roadmapListInShop = await RoadmapModel.aggregate()
    .match({ shopId: { $in: shopList.map(o => o._id) }, endPointLastCode, enable: true })
    .project({
        shop: { $arrayElemAt: [ shopList, { $indexOfArray: [ shopList.map(o => o._id), '$shopId' ] } ] },
        sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
        rate: { $sum: [ 1, { $ifNull: [ '$profitRate', '$shop.regionProfitRate' ] } ] },
        shipperId: 1,
        price: 1,
        minFee: 1,
        selfSignRate: 1,
        duration: 1,
    })
    .project({
        rate: 1,
        shopName: '$shop.name',
        shopAddress: '$shop.address',
        distance: '$shop.distance',
        sendDoorEnable: '$sendDoor.enable',
        sendDoorPrice: '$sendDoor.sendDoorPrice',
        sendDoorMinFee: '$sendDoor.sendDoorMinFee',
        shipperId: 1,
        price: 1,
        minFee: 1,
        selfSignRate: 1,
        duration: 1,
    })
    .match(isSendDoor ? { sendDoorEnable: true } : {})
    .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: '_shipper' })
    .project({
        id: '$_id',
        _id: 0,
        shipperName: { $arrayElemAt: ['$_shipper.name', 0] },
        price: { $multiply: [ '$rate', '$price' ] },
        minFee: { $multiply: [ '$rate', '$minFee' ] },
        sendDoorPrice: { $multiply: [ '$rate', '$sendDoorPrice' ] },
        sendDoorMinFee: { $multiply: [ '$rate', '$sendDoorMinFee' ] },
        shopName: 1,
        shopAddress: 1,
        distance: 1,
        selfSignRate: 1,
        duration: 1,
    })
    .sort(sort)
    .skip(pageNo * pageSize)
    .limit(pageSize);

    // 收货点的部分
    let roadmapListInAgent = [];
    if (pageNo === 0) { // 收货点不采取分页模式
        const agents = await AgentModel.aggregate()
        .near({
            near: location,
            distanceField: 'dist.calculated',
        })
        .match({ addressRegionLastCode: startPointLastCode })
        .project({
            distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
            name: 1,
            address: 1,
            referShopId: 1,
            roadmapProfitRate: 1,
        });

        for (const agent of agents) {
            const regionProfitRate = await getRegionProfitRateByEndPoint(agent.referShopId, endPointLastCode, isCityDistribute);
            const roadmaps = await RoadmapModel.aggregate()
            .match({ shopId: agent.referShopId, endPointLastCode, enable: true })
            .project({
                sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
                rate: { $sum: [ 1, { $ifNull: [ '$profitRate', regionProfitRate ] } ] },
                shipperId: 1,
                price: 1,
                minFee: 1,
                selfSignRate: 1,
                duration: 1,
            })
            .project({
                sendDoorEnable: '$sendDoor.enable',
                sendDoorPrice: '$sendDoor.sendDoorPrice',
                sendDoorMinFee: '$sendDoor.sendDoorMinFee',
                shipperId: 1,
                rate: 1,
                price: 1,
                minFee: 1,
                selfSignRate: 1,
                duration: 1,
            })
            .match(isSendDoor ? { sendDoorEnable: true } : {})
            .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
            .project({
                id: '$_id',
                _id: 0,
                shipperName: { $arrayElemAt: ['$shipper.name', 0] },
                price: { $multiply: [ '$rate', '$price' ] },
                minFee: { $multiply: [ '$rate', '$minFee' ] },
                sendDoorPrice: { $multiply: [ '$rate', '$sendDoorPrice' ] },
                sendDoorMinFee: { $multiply: [ '$rate', '$sendDoorMinFee' ] },
                selfSignRate: 1,
                duration: 1,
            })
            .sort(sort)
            .limit(1);

            const roadmap = roadmaps[0];
            if (roadmap) {
                const rateModel = await AgentRegionProfitModel.findOne({ agentId: agent._id, roadmapId: roadmap.id });
                const rate = Math.max((rateModel && rateModel.profitRate || 0), agent.roadmapProfitRate) + 1;
                roadmap.price = _N(roadmap.price * rate);
                roadmap.minFee = _N(roadmap.minFee * rate);
                roadmap.distance = agent.distance;
                roadmap.agentName = agent.name;
                roadmap.agentAddress = agent.address;
                roadmap.shipperName = roadmap.shipperName;
                roadmap.id = agent.id;

                roadmapListInAgent.push(roadmap);
            }
        }

        // 对收货点的路线排序
        if (orderBy === 0) {
            roadmapListInAgent = _.sortBy(roadmapListInAgent, o => o.price);
        } else if (orderBy === 1) {
            roadmapListInAgent = _.sortBy(roadmapListInAgent, o => o.duration);
        } else if (orderBy === 2) {
            roadmapListInAgent = _.sortBy(roadmapListInAgent, o => o.selfSignRate);
        }
    }

    return { success: true, context: {
        shop: {
            count,
            roadmapList: roadmapListInShop,
        },
        agent: {
            roadmapList: roadmapListInAgent,
        },
    } };
};

```

* PDShopServer/project/App/routers/posts/client/roadmap/getRoadmapListWithPreOrder.js

```js
import mongoose from 'mongoose';
import { OrderModel, ShopModel, AgentModel, AgentRegionProfitModel } from '../../../../models';
import { searchRoadmapListWithShop, searchRoadmapListWithShopList } from '../../libs/roadmap';
import { _N } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderId, // 预下单单号
    location, // 我的位置
    orderBy, // 排序方式，0：价格排序，1：时间排序，2：亲签率排序，3：距离排序
    fromPC,
    pageNo,
    pageSize,
}) => {
    const order = await OrderModel.findById(orderId);
    if (!order) {
        return { msg: '没有该货单' };
    }
    const {
        startPointLastCode, // 省市（只对预下单有用）
        shopId, // 指定的分店
        agentId, // 指定的收货点
        endPointLastCode, // 终点
        sendDoorEndPointLastCode, // 送货上门终点
        isCityDistribute, // 是否同城配送
        isSendDoor, // 是否送货上门
        weight, // 重量
        size, // 方量
        proxyCharge, // 代收货款金额
        isReachPay, // 是否是到付
        totalDesignatedFee, //指定向收货人应该收多少钱
    } = order;

    // 始发地为分店的情况
    if (shopId) {
        const shops = await ShopModel.aggregate()
        .near({
            near: location,
            distanceField: 'dist.calculated',
        })
        .match({ _id: new mongoose.Types.ObjectId(shopId) })
        .project({
            distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
            name: 1,
            address: 1,
        }).limit(1);
        const shop = shops[0];

        const context = await searchRoadmapListWithShop({
            shopId,
            distance: shop.distance,
            shopName: shop.name,
            shopAddress: shop.address,
            fromPC,
            pageNo,
            pageSize,
            endPointLastCode,
            sendDoorEndPointLastCode,
            isCityDistribute,
            isSendDoor,
            weight,
            size,
            proxyCharge,
            isReachPay,
            totalDesignatedFee,
            orderBy,
        });
        return { success: true, context: { shop: context } };
    };

    // 始发地为收货点的情况
    if (agentId) {
        const agents = await AgentModel.aggregate()
        .near({
            near: location,
            distanceField: 'dist.calculated',
        })
        .match({ _id: new mongoose.Types.ObjectId(agentId) })
        .lookup({ from: 'shops', localField: 'referShopId', foreignField: '_id', as: 'shop' })
        .project({
            distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
            roadmapProfitRate: 1,
            referShopId: { $arrayElemAt: ['$shop._id', 0] },
            name: 1,
            address: 1,
        });
        const agent = agents[0];

        let { roadmapList } = await searchRoadmapListWithShop({
            shopId: agent.referShopId,
            distance: agent.distance,
            pageNo: 0,
            pageSize: 1,
            endPointLastCode,
            sendDoorEndPointLastCode,
            isCityDistribute,
            isSendDoor,
            weight,
            size,
            proxyCharge,
            isReachPay,
            totalDesignatedFee,
            orderBy,
        });

        if (roadmapList.length) {
            const roadmap = roadmapList[0];
            const rateModel = await AgentRegionProfitModel.findOne({ agentId, roadmapId: roadmap.id });
            const rate = Math.max((rateModel && rateModel.profitRate || 0), agent.roadmapProfitRate) + 1;
            roadmap.id = agent._id;
            roadmap.agentName = agent.name;
            roadmap.agentAddress = agent.address;
            roadmap.transportFee = _N(roadmap.transportFee * rate);
            roadmap.price = _N(roadmap.price * rate);
            roadmap.minFee = _N(roadmap.minFee * rate);
        }

        return { success: true, context: { agent: { roadmapList } } };
    }

    // 始发地为城市范围的情况
    const shopList = await ShopModel.aggregate()
    .near({
        near: location,
        distanceField: 'dist.calculated',
    })
    .match({ addressRegionLastCode: startPointLastCode, isMasterShop: false })
    .project({
        id: '$_id',
        _id: 0,
        distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
        name: 1,
        address: 1,
    });
    const roadmapListInShop = await searchRoadmapListWithShopList({
        shopList,
        fromPC,
        pageNo,
        pageSize,
        endPointLastCode,
        sendDoorEndPointLastCode,
        isCityDistribute,
        isSendDoor,
        weight,
        size,
        proxyCharge,
        isReachPay,
        totalDesignatedFee,
        orderBy,
    });

    let roadmapListInAgent = [];
    if (pageNo === 0) { // 收货点不采取分页模式
        const agents = await AgentModel.aggregate()
        .near({
            near: location,
            distanceField: 'dist.calculated',
        })
        .match({ addressRegionLastCode: startPointLastCode })
        .lookup({ from: 'shops', localField: 'referShopId', foreignField: '_id', as: 'shop' })
        .project({
            distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
            roadmapProfitRate: 1,
            referShopId: { $arrayElemAt: ['$shop._id', 0] },
            name: 1,
            address: 1,
        });

        for (const agent of agents) {
            if (!agent.referShopId) {
                continue;
            }
            const { roadmapList } = await searchRoadmapListWithShop({
                shopId: agent.referShopId,
                distance: agent.distance,
                pageNo: 0,
                pageSize: 1,
                endPointLastCode,
                sendDoorEndPointLastCode,
                isCityDistribute,
                isSendDoor,
                weight,
                size,
                proxyCharge,
                isReachPay,
                totalDesignatedFee,
                orderBy,
            });
            const roadmap = roadmapList[0];
            if (roadmap) {
                const rateModel = await AgentRegionProfitModel.findOne({ agentId, roadmapId: roadmap.id });
                const rate = Math.max((rateModel && rateModel.profitRate || 0), agent.roadmapProfitRate) + 1;
                roadmap.id = agent._id;
                roadmap.agentName = agent.name;
                roadmap.agentAddress = agent.address;
                roadmap.transportFee = _N(roadmap.transportFee * rate);
                roadmap.price = _N(roadmap.price * rate);
                roadmap.minFee = _N(roadmap.minFee * rate);
                roadmapListInAgent.push(roadmap);
            }
        }
    }
    return { success: true, context: {
        shop: roadmapListInShop,
        agent: { roadmapList: roadmapListInAgent },
    } };
};

```

* PDShopServer/project/App/routers/posts/client/roadmap/getRoadmapListWithPreOrderGroup.js

```js
import mongoose from 'mongoose';
import { OrderGroupModel, ShopModel, AgentModel, AgentRegionProfitModel } from '../../../../models';
import { searchRoadmapListWithShop } from '../../libs/roadmap';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderGroupId, // 预下单单号
    location, // 我的位置
    fromPC,
}) => {
    const orderGroup = await OrderGroupModel.findById(orderGroupId)
    .populate({
        path: 'orderIdList',
        select: {
            name: 1,
            endPoint: 1,
            endPointLastCode: 1,
            sendDoorEndPoint: 1,
            sendDoorEndPointLastCode: 1,
            isSendDoor: 1,
            totalNumbers: 1,
            weight: 1,
            size: 1,
            proxyCharge: 1,
            isReachPay: 1,
            totalDesignatedFee: 1,
        },
    });
    if (!orderGroup) {
        return { msg: '没有该货单群组' };
    }
    const {
        startPointLastCode, // 省市（只对预下单有用）
        shopId, // 指定的分店
        agentId, // 指定的收货点
        orderIdList, //货单列表
    } = orderGroup;

    // 始发地为分店的情况
    if (shopId) {
        const shops = await ShopModel.aggregate()
        .near({
            near: location,
            distanceField: 'dist.calculated',
        })
        .match({ _id: new mongoose.Types.ObjectId(shopId) })
        .project({
            id: '$_id',
            _id: 0,
            distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
            name: 1,
            address: 1,
        }).limit(1);
        const shop = shops[0];

        const unfindOrderList = [];
        let totalTransportFee = 0;
        for (const order of orderIdList) {
            const { roadmapList } = await searchRoadmapListWithShop({
                shopId,
                distance: shop.distance,
                shopName: shop.name,
                shopAddress: shop.address,
                pageNo: 0,
                pageSize: 1,
                endPointLastCode: order.endPointLastCode,
                sendDoorEndPointLastCode: order.sendDoorEndPointLastCode,
                isCityDistribute: order.isCityDistribute,
                isSendDoor: order.isSendDoor,
                weight: order.weight,
                size: order.size,
                proxyCharge: order.proxyCharge,
                isReachPay: order.isReachPay,
                totalDesignatedFee: order.totalDesignatedFee,
                orderBy: 0,
            });
            if (!roadmapList.length) {
                unfindOrderList.push(order);
            } else {
                totalTransportFee += roadmapList[0].transportFee;
            }
        }

        return { success: true, context: { shopList: [{
            totalTransportFee,
            name: shop.name,
            address: shop.address,
            distance: shop.distance,
            unfindOrderList,
        }] } };
    };

    // 始发地为收货点的情况
    if (agentId) {
        const agents = await AgentModel.aggregate()
        .near({
            near: location,
            distanceField: 'dist.calculated',
        })
        .match({ _id: new mongoose.Types.ObjectId(agentId) })
        .lookup({ from: 'shops', localField: 'referShopId', foreignField: '_id', as: 'shop' })
        .project({
            id: '$_id',
            _id: 0,
            distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
            referShopId: { $arrayElemAt: ['$shop._id', 0] },
            name: 1,
            address: 1,
        });
        const agent = agents[0];

        const unfindOrderList = [];
        let totalTransportFee = 0;
        for (const order of orderIdList) {
            const { roadmapList } = await searchRoadmapListWithShop({
                shopId: agent.referShopId,
                distance: agent.distance,
                pageNo: 0,
                pageSize: 1,
                endPointLastCode: order.endPointLastCode,
                sendDoorEndPointLastCode: order.sendDoorEndPointLastCode,
                isCityDistribute: order.isCityDistribute,
                isSendDoor: order.isSendDoor,
                weight: order.weight,
                size: order.size,
                proxyCharge: order.proxyCharge,
                isReachPay: order.isReachPay,
                totalDesignatedFee: order.totalDesignatedFee,
                orderBy: 0,
            });
            if (!roadmapList.length) {
                unfindOrderList.push(order);
            } else {
                const roadmap = roadmapList[0];
                const profitRate = await AgentRegionProfitModel.findOne({ agentId, roadmapId: roadmap.id });
                const rate = Math.max((profitRate && profitRate.profitRate || 0), agent.roadmapProfitRate) + 1;
                totalTransportFee += roadmapList[0].transportFee * rate;
            }
        }

        return { success: true, context: { agentList: [{
            totalTransportFee,
            id: agent.id,
            name: agent.name,
            address: agent.address,
            distance: agent.distance,
            unfindOrderList,
        }] } };
    }

    // 始发地为城市范围的情况
    const shops = await ShopModel.aggregate()
    .near({
        near: location,
        distanceField: 'dist.calculated',
    })
    .match({ addressRegionLastCode: startPointLastCode, isMasterShop: false })
    .project({
        id: '$_id',
        _id: 0,
        distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
        name: 1,
        address: 1,
    });

    const shopList = [];
    for (const shop of shops) {
        const unfindOrderList = [];
        let totalTransportFee = 0;
        for (const order of orderIdList) {
            const { roadmapList } = await searchRoadmapListWithShop({
                shopId: shop.id,
                distance: shop.distance,
                shopName: shop.name,
                shopAddress: shop.address,
                pageNo: 0,
                pageSize: 1,
                endPointLastCode: order.endPointLastCode,
                sendDoorEndPointLastCode: order.sendDoorEndPointLastCode,
                isCityDistribute: order.isCityDistribute,
                isSendDoor: order.isSendDoor,
                weight: order.weight,
                size: order.size,
                proxyCharge: order.proxyCharge,
                isReachPay: order.isReachPay,
                totalDesignatedFee: order.totalDesignatedFee,
                orderBy: 0,
            });
            if (!roadmapList.length) {
                unfindOrderList.push(order);
            } else {
                totalTransportFee += roadmapList[0].transportFee;
            }
        }
        shopList.push({
            totalTransportFee,
            id: shop.id,
            name: shop.name,
            address: shop.address,
            distance: shop.distance,
            unfindOrderList,
        });
    }

    // 收货点部分
    const agents = await AgentModel.aggregate()
    .near({
        near: location,
        distanceField: 'dist.calculated',
    })
    .match({ addressRegionLastCode: startPointLastCode })
    .lookup({ from: 'shops', localField: 'referShopId', foreignField: '_id', as: 'shop' })
    .project({
        id: '$_id',
        _id: 0,
        distance: { $multiply: [ '$dist.calculated', CONSTANTS.KILOMETRE_RADIAN_RATE ] },
        referShopId: { $arrayElemAt: ['$shop._id', 0] },
        roadmapProfitRate: 1,
        name: 1,
        address: 1,
    });

    const agentList = [];
    for (const agent of agents) {
        const unfindOrderList = [];
        let totalTransportFee = 0;
        for (const order of orderIdList) {
            const { roadmapList } = await searchRoadmapListWithShop({
                shopId: agent.referShopId,
                distance: agent.distance,
                pageNo: 0,
                pageSize: 1,
                endPointLastCode: order.endPointLastCode,
                sendDoorEndPointLastCode: order.sendDoorEndPointLastCode,
                isCityDistribute: order.isCityDistribute,
                isSendDoor: order.isSendDoor,
                weight: order.weight,
                size: order.size,
                proxyCharge: order.proxyCharge,
                isReachPay: order.isReachPay,
                totalDesignatedFee: order.totalDesignatedFee,
                orderBy: 0,
            });
            if (!roadmapList.length) {
                unfindOrderList.push(order);
            } else {
                const roadmap = roadmapList[0];
                const profitRate = await AgentRegionProfitModel.findOne({ agentId, roadmapId: roadmap.id });
                const rate = Math.max((profitRate && profitRate.profitRate || 0), agent.roadmapProfitRate) + 1;
                totalTransportFee += roadmapList[0].transportFee * rate;
            }
        }

        agentList.push({
            totalTransportFee,
            id: agent.id,
            name: agent.name,
            address: agent.address,
            distance: agent.distance,
            unfindOrderList,
        });
    }
    return { success: true, context: {
        shopList,
        agentList,
    } };
};

```

* PDShopServer/project/App/routers/posts/client/weixinPay/getUnifiedOrder.js

```js
import { ClientModel, WeixinPayModel } from '../../../../models';
import { _F, _T, hasAuthority, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';
import api from '../../libs/weixinPay/api';
import config from '../../../../../config';
import { rechargeForClient } from '../../libs/account';

export default async ({
    userId,
    amount, // 单位：元
    body, //商品描述
}, req) => {
    amount = _F(amount);
    if (config.hasNoRealPayment) {
        return rechargeForClient({ userId, amount, thirdpartyAccount: 'SIMULATION_NO' });
    }
    amount = amount / config.weixinPayRate;
    if (amount < 1) {
        return { msg: '充值金额必须大于' + config.weixinPayRate / 100 };
    }
    return new Promise(async resolve => {
        const client = await ClientModel.findById(userId);
        if (!client) {
            return resolve({ msg: '你没有充值的权限' });
        }
        if (client.shipperId && !hasAuthority(client, CONSTANTS.AH_RECHARGE)) {
            return resolve({ msg: '你没有充值的权限' });
        }

        const spbill_create_ip = req.get('X-Real-IP') || req.get('X-Forwarded-For') || req.ip;
        const order = new WeixinPayModel({
            clientId: userId,
            spbill_create_ip,
            body,
            trade_type: 'APP',
            total_fee: amount,
        });

        const params = {
            spbill_create_ip,
            body,
            out_trade_no: _T(order.out_trade_no),
            total_fee: amount,
            trade_type: 'APP',
        };
        const result = await api.unifiedorder(params);
        if (result.success) {
            order.prepay_id = result.context.prepayid;
            order.appid = result.context.appid;
            order.mch_id = result.context.partnerid;
            order.timestamp = result.context.timestamp;
            order.nonce_str = result.context.noncestr;
            order.package = result.context.package;
            await order.save();
        }

        actionLog(userId, 'Client', amount, 'Number', 'getUnifiedOrder');

        return resolve(result);
    });
};

```

* PDShopServer/project/App/routers/posts/client/weixinPay/getUnifiedOrderForPC.js

```js
import { ClientModel, WeixinPayModel } from '../../../../models';
import { _F, _T, hasAuthority, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';
import api from '../../libs/weixinPay/api';
import config from '../../../../../config';
import { rechargeForClient } from '../../libs/account';

export default async ({
    userId,
    amount, // 单位：元
    body, //商品描述
}, req) => {
    amount = _F(amount);
    if (config.hasNoRealPayment) {
        return rechargeForClient({ userId, amount, thirdpartyAccount: 'SIMULATION_NO' });
    }
    amount = amount / config.weixinPayRate;
    if (amount < 1) {
        return { msg: '充值金额必须大于' + config.weixinPayRate / 100 };
    }
    return new Promise(async resolve => {
        const client = await ClientModel.findById(userId);
        if (!client) {
            return resolve({ msg: '你没有充值的权限' });
        }
        if (client.shipperId && !hasAuthority(client, CONSTANTS.AH_RECHARGE)) {
            return resolve({ msg: '你没有充值的权限' });
        }

        const spbill_create_ip = req.get('X-Real-IP') || req.get('X-Forwarded-For') || req.ip;
        const order = new WeixinPayModel({
            clientId: userId,
            spbill_create_ip,
            body,
            trade_type: 'NATIVE',
            total_fee: amount,
        });

        const params = {
            spbill_create_ip,
            body,
            out_trade_no: _T(order.out_trade_no),
            total_fee: amount,
            trade_type: 'NATIVE',
        };
        const result = await api.unifiedorder(params);
        if (result.success) {
            order.prepay_id = result.context.prepayid;
            order.appid = result.context.appid;
            order.mch_id = result.context.partnerid;
            order.code_url = result.context.code_url;
            await order.save();

            result.context = { qrcode: result.context.code_url };
        }

        actionLog(userId, 'Client', amount, 'Number', 'getUnifiedOrderForPC');

        return resolve(result);
    });
};

```

* PDShopServer/project/App/routers/posts/common/address/getRegionAddress.js

```js
import { RegionModel } from '../../../../mysql';
import { RoadmapAddressModel, ClientModel, ShopMemberModel } from '../../../../models';
import config from '../../../../../config';
import _ from 'lodash';

export default async ({
    userId,
    parentCode = 0,
    type = 0, // 0: 长途地址，1：短途地址，2：所有地址, 3: 分店路线中存在的地址, 4: 收货点路线中存在的地址
}) => {
    (!config.useOnlyExitRoadmap && (type === 3 || type === 4)) && (type = 0);
    let addressList = [];
    if (type < 3) {
        const list = await RegionModel.findAll({ where: { parentCode } });
        addressList = type === 0 ? _.reject(list, o => o.level === 8 || o.level === 9) : list;
    } else if (type === 3) {
        const member = await ShopMemberModel.findById(userId);
        addressList = await RoadmapAddressModel.find({ shopId: member.shopId, parentCode });
    } else if (type === 4) {
        const member = await ClientModel.findById(userId).populate({ path: 'agentId', select: { referShopId: 1 } });
        addressList = await RoadmapAddressModel.find({ shopId: ((member || {}).agent || {}).referShopId, parentCode });
    }

    return { success: true, context: { addressList } };
};

```

* PDShopServer/project/App/routers/posts/common/address/getRegionAddressFromLastCode.js

```js
import { ClientModel, ShopMemberModel } from '../../../../models';
import { getAddressFromLastCode, getSendDoorAddressFromLastCode, getEndPointFromLastCode } from '../../libs/address';
import config from '../../../../../config';

export default async ({
    userId,
    addressLastCode = 0,
    type = 0, // 0: 长途地址，1：短途地址，2：所有地址, 3: 分店路线中存在的地址, 4: 收货点路线中存在的地址
}) => {
    (!config.useOnlyExitRoadmap && (type === 3 || type === 4)) && (type = 0);
    let addressList = [];
    if (type === 0) {
        addressList = await getAddressFromLastCode(addressLastCode);
    } else if (type === 1) {
        addressList = await getSendDoorAddressFromLastCode(addressLastCode);
    } else if (type === 2) {
        addressList = await getAddressFromLastCode(addressLastCode, true);
    } else if (type === 3) {
        const member = await ShopMemberModel.findById(userId);
        addressList = await getEndPointFromLastCode(member.shopId, addressLastCode);
    } else if (type === 4) {
        const member = await ClientModel.findById(userId).populate({ path: 'agentId', select: { referShopId: 1 } });
        addressList = await getEndPointFromLastCode(((member || {}).agent || {}).referShopId, addressLastCode);
    }
    return { success: true, context: {
        addressList,
    } };
};

```

* PDShopServer/project/App/routers/posts/common/address/getRegionSendDoorAddressFromLastCode.js

```js
import { getSendDoorAddressFromLastCode } from '../../libs/address';

export default async ({
    userId,
    addressLastCode,
    parentCode,
}) => {
    const addressList = await getSendDoorAddressFromLastCode(addressLastCode, parentCode);

    return { success: true, context: {
        addressList,
    } };
};

```

* PDShopServer/project/App/routers/posts/common/address/getStartPointAddress.js

```js
import { ShopModel, AgentModel, StartPointAddressModel } from '../../../../models';
import _ from 'lodash';

export default async ({
    userId,
    parentCode = 0,
}) => {
    let list = [];
    const item = await StartPointAddressModel.findOne({ code: parentCode });
    if (!item || !_.includes([ 0, 4, 5, 6, 7 ], item.level)) {
        list = await StartPointAddressModel.find({ parentCode });
    } else {
        const shopList = await ShopModel.find({ isMasterShop: false, addressRegionLastCode: parentCode }).select({ name: 1, addressRegionLastCode: 1 });
        const agentList = await AgentModel.find({ addressRegionLastCode: parentCode }).select({ name: 1, addressRegionLastCode: 1 });
        list = shopList.map(o => { o = o.toObject(); o.isShop = true; o.isLeaf = true; return o; });
        list = list.concat(agentList.map(o => { o = o.toObject(); o.isAgent = true; o.isLeaf = true; return o; }));
    }
    return { success: true, context: { addressList: list } };
};

```

* PDShopServer/project/App/routers/posts/common/address/getStartPointAddressFromLastCode.js

```js
import { ShopModel, AgentModel, StartPointAddressModel } from '../../../../models';

export default async ({
    userId,
    addressLastCode = 0,
    isLeaf, //是否是分店或者收货点（该参数根据接收到的值确定）
}) => {
    if (addressLastCode === 0) {
        const list = await StartPointAddressModel.find({ parentCode: 0 });
        return { success: true, context: { addressList: [ list ] } };
    }

    const addressList = [];
    if (isLeaf) {
        let list = [];
        const shopList = await ShopModel.find({ isMasterShop: false, addressRegionLastCode: addressLastCode }).select({ name: 1, addressRegionLastCode: 1 });
        const agentList = await AgentModel.find({ addressRegionLastCode: addressLastCode }).select({ name: 1, addressRegionLastCode: 1 });
        list = shopList.map(o => { o = o.toObject(); o.isShop = true; o.isLeaf = true; return o; });
        list = list.concat(agentList.map(o => { o = o.toObject(); o.isAgent = true; o.isLeaf = true; return o; }));
        addressList.push(list);
    }
    while (true) {
        const item = await StartPointAddressModel.findOne({ code: addressLastCode });
        if (!item) {
            break;
        }
        if (item.parentCode === 0) {
            const list = await StartPointAddressModel.find({ parentCode: 0 });
            addressList.push(list);
            break;
        } else {
            const list = await StartPointAddressModel.find({ parentCode: item.parentCode });
            addressList.push(list);
            addressLastCode = item.parentCode;
        }
    }
    return { success: true, context: { addressList } };
};

```

* PDShopServer/project/App/routers/posts/common/client/getClientNameByPhone.js

```js
import { ClientModel } from '../../../../models';

export default async ({ userId, phone }) => {
    const doc = await ClientModel.findOne({ phone });
    return { success: true, context: { name: (doc || {}).name } };
};

```

* PDShopServer/project/App/routers/posts/common/notify/getNotifies.js

```js
import { NotifyModel } from '../../../../models';
import _ from 'lodash';

async function getPageData (type, keyword, fromPC, pageNo, pageSize) {
    let criteria = { type };
    if (keyword) {
        const regex = new RegExp('.*' + (keyword || '') + '.*', 'gim');
        criteria = { ...criteria, $or: [{ title: regex }, { content: regex }] };
    }
    const count = (fromPC && pageNo === 0) ? await NotifyModel.count(criteria) : undefined;
    const query = NotifyModel.find(criteria).sort({ time: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const list = await query
    .select({
        id: 1,
        source: 1,
        title: 1,
        content: 1,
        time: 1,
    });
    return {
        count,
        list,
    };
}

const TYPES = ['news', 'publicity', 'policy', 'notice'];
export default async ({ userId, type, keyword, fromPC, pageNo, pageSize }) => {
    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(type, keyword, fromPC, pageNo, pageSize),
            },
        };
    } else {
        return {
            success: true,
            context: {
                news: await getPageData('news', keyword, fromPC, pageNo, pageSize),
                publicity: await getPageData('publicity', keyword, fromPC, pageNo, pageSize),
                policy: await getPageData('policy', keyword, fromPC, pageNo, pageSize),
                notice: await getPageData('notice', keyword, fromPC, pageNo, pageSize),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/common/notify/sendNotify.js

```js
import { NotifyModel, ShopMemberModel } from '../../../../models';
import _ from 'lodash';

export default async ({
    adminId,
    shopId,
    memberId,
    shipperId,
    type,
    source,
    title,
    content,
}) => {
    if (memberId) {
        const member = await ShopMemberModel.findById(memberId);
        shopId = (member || {}).shopId;
    }

    const notify = new NotifyModel({ adminId, shopId, memberId, shipperId, type, source, title, content });
    await notify.save();
    return { success: true, context: _.pick(notify.toObject(), ['source', 'title', 'content', 'time', 'id']) };
};

```

* PDShopServer/project/App/routers/posts/common/upload/uploadFile.js

```js
import mongoose from 'mongoose';
import { MediaModel } from '../../../../models';
import { getMediaPath } from '../../../../utils/';

export default ({
    clientId,
    shopMemberId,
    adminId,
}, req) => {
    return new Promise(async resolve => {
        const gridId = req.file.grid._id;
        const media = new MediaModel({
            gridId,
        });
        const doc = await media.save();
        if (doc) {
            return resolve({ success: true, context: { url: getMediaPath(doc._id) } });
        } else {
            mongoose.gfs.remove({ _id: gridId });
            return resolve({ msg: '上传失败' });
        }
    });
};

```

* PDShopServer/project/App/routers/posts/common/verifyCode/requestSendVerifyCode.js

```js
import { VerifyCodeModel } from '../../../../models';
import { sendSms } from '../../../../utils';
import config from '../../../../../config';
import _ from 'lodash';

function getCodeItem () {
    let code = '';
    for (let i = 0; i < 6; i++) {
        code += _.random(0, 9);
    }
    return { code, deadTime: Date.now() + 7200000 };
}

export default async ({
    phone,
}) => {
    if (config.hasNoVerifyCode) {
        return { success: true };
    }

    let codeItem;
    let doc = await VerifyCodeModel.findOne({ phone });
    if (!doc) {
        codeItem = getCodeItem();
        doc = new VerifyCodeModel({
            phone,
            codeList: [ codeItem ],
        });
    } else {
        const codeList = _.filter(doc.codeList, o => o.deadTime.getTime() > Date.now());
        if (codeList.length > 2) {
            return { msg: '操作过于频繁，请稍后再试' };
        }
        codeItem = getCodeItem();
        codeList.unshift(codeItem);
        doc.codeList = codeList;
    }
    const success = await sendSms(phone, codeItem.code);
    if (!success) {
        return { msg: '发送短信失败，请稍后再试' };
    }
    await doc.save();
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/driver/personal/findPassword.js

```js
import { DriverModel } from '../../../../models';
import { setPassword, checkVerifyCode, actionLog } from '../../../../utils';

export default async ({ phone, password, verifyCode }) => {
    const user = await DriverModel.findOne({ phone });
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    const success = await checkVerifyCode(phone, verifyCode);
    if (!success) {
        return { msg: '无效验证码' };
    }

    const error = await setPassword(user, password);
    if (error) {
        return { msg: '服务器错误' };
    }
    await user.save();

    actionLog(user.id, 'Driver', user.id, 'Driver', 'findPassword');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/driver/personal/getPersonalInfo.js

```js
import { DriverModel } from '../../../../models';

export default async ({ userId }) => {
    const user = await DriverModel.findById(userId);
    if (!user) {
        return { msg: '没有该用户' };
    }
    const context = user.toObject();
    context.userId = context.id;
    delete context.id;

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/driver/personal/getUserNameByPhone.js

```js
import { DriverModel } from '../../../../models';

export default async ({ userId, phone }) => {
    const doc = await DriverModel.findOne({ phone });
    return { success: true, context: { name: (doc || {}).name } };
};

```

* PDShopServer/project/App/routers/posts/driver/personal/login.js

```js
import { DriverModel } from '../../../../models';
import { authenticatePassword, actionLog } from '../../../../utils';

export default async ({ phone, password }) => {
    const user = await DriverModel.findOne({ phone });
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    const error = await authenticatePassword(user, password);
    if (error) {
        return { msg: '密码错误' };
    } else {
        actionLog(user.id, 'Driver', user.id, 'Driver', 'login');
        return { success: true, context: { userId: user.id, shopId: user.shopId } };
    }
};

```

* PDShopServer/project/App/routers/posts/driver/personal/modifyPassword.js

```js
import { DriverModel } from '../../../../models';
import { setPassword, authenticatePassword, actionLog } from '../../../../utils';

export default async ({ userId, oldPassword, newPassword }) => {
    const user = await DriverModel.findById(userId);
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    let error = await authenticatePassword(user, oldPassword);
    if (error) {
        return { msg: '密码错误' };
    }
    error = await setPassword(user, newPassword);
    if (error) {
        return { msg: '设置密码失败' };
    }
    await user.save();

    actionLog(userId, 'Driver', userId, 'Driver', 'modifyPassword');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/driver/personal/modifyPersonalInfo.js

```js
import { DriverModel, MediaModel } from '../../../../models';
import { getMediaId, omitNil, getAgeFromBirthday, actionLog } from '../../../../utils';

export default async ({
    userId,
    name,
    head,
    email,
    sex,
    birthday,
    address,
    phoneList,
    license,
    licenseNo,
}) => {
    let _head = getMediaId(head);
    let _license = getMediaId(license);
    const doc = await DriverModel.findByIdAndUpdate(userId, omitNil({
        name,
        head: _head,
        email,
        sex,
        birthday,
        address,
        phoneList,
        license: _license,
        licenseNo,
    }));
    if (!doc) {
        return { msg: '修改失败' };
    }
    MediaModel._updateRef(
        { [_head]: 1 },
        { [head && doc.head]: -1 },
        { [_license]: 1 },
        { [license && doc.license]: -1 },
    );

    let context = await DriverModel.findById(userId);
    context = context.toObject();
    context.userId = context.id;
    delete context.id;

    actionLog(userId, 'Driver', userId, 'Driver', 'modifyPersonalInfo');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/driver/personal/register.js

```js
import { DriverModel } from '../../../../models';
import { registerUser, checkVerifyCode, actionLog } from '../../../../utils';

export default async ({ phone, password, verifyCode }) => {
    const success = await checkVerifyCode(phone, verifyCode);
    if (!success) {
        return { msg: '无效验证码' };
    }
    const user = new DriverModel({ phone });
    const error = await registerUser(DriverModel, user, password);
    if (!error) {
        actionLog(user.id, 'Driver', user.id, 'Driver', 'register');
        return { success: true };
    } else if (error === 'UserExistsError') {
        return { msg: '该账号已经被占用' };
    } else {
        return { msg: '注册失败' };
    }
};

```

* PDShopServer/project/App/routers/posts/driver/personal/submitFeedback.js

```js
import { FeedbackModel } from '../../../../models';

export default async ({ userId, content, email }) => {
    const feedback = new FeedbackModel({ driverId: userId, content, email });
    await feedback.save();
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/driver/truck/confirmReach.js

```js
import { TruckModel } from '../../../../models';
import { actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, address, longitude, latitude }) => {
    const truck = await TruckModel.findOneAndUpdate(
        { driverId: userId, state: CONSTANTS.TS_ON_THE_WAY },
        { state: CONSTANTS.TS_RECEIVE_SUCCESS, $push: { locationList: { $each: [ { address, longitude, latitude, time: Date.now() } ], $position: 0 } } }
    );
    if (!truck) {
        return { msg: '你当前没有在运输中的车辆' };
    }
    actionLog(userId, 'Driver', truck.id, 'Truck', 'confirmReach');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/driver/truck/getLocationList.js

```js
import { TruckModel } from '../../../../models';
import CONSTANTS from '../../../../constants';
import { getFormatTime } from '../../../../utils';

export default async ({ userId }) => {
    const truck = await TruckModel.findOne({ driverId: userId, state: CONSTANTS.TS_ON_THE_WAY }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });
    let locationList = [];
    if (!truck) {
        return { success: true, locationList: [] };
    } else {
        locationList.push({
            address: (truck.shop || {}).address,
            time: getFormatTime(truck.createTime),
        });
        locationList = [...(truck || {}).locationList || [], ...locationList];
    }
    return { success: true, locationList };
};

```

* PDShopServer/project/App/routers/posts/driver/truck/getOnWayTruck.js

```js
import { TruckModel } from '../../../../models';
import CONSTANTS from '../../../../constants';
import { getFormatTime } from '../../../../utils';

export default async ({ userId }) => {
    const truck = await TruckModel.findOne({ driverId: userId, state: CONSTANTS.TS_ON_THE_WAY }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });
    let locationList = [];
    if (!truck) {
        return { msg: '你当前没有在运输中的车辆' };
    } else {
        locationList.push({
            address: (truck.shop || {}).address,
            time: getFormatTime(truck.createTime),
        });
        truck.locationList = [...(truck || {}).locationList || [], ...locationList];
    }
    return { success: true, context: truck };
};

```

* PDShopServer/project/App/routers/posts/driver/truck/uploadLocation.js

```js
import { TruckModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({ userId, address, longitude, latitude }) => {
    const truck = await TruckModel.findOneAndUpdate({ driverId: userId, state: CONSTANTS.TS_ON_THE_WAY }, { $push: { locationList: { $each: [ { address, longitude, latitude, time: Date.now() } ], $position: 0 } } });
    if (!truck) {
        return { success: false };
    }
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/libs/account/index.js

```js
export rechargeForClient from './rechargeForClient'; // 客户端充值
export rechargeForShopMember from './rechargeForShopMember'; // 店面端充值
export withdrawForClient from './withdrawForClient'; // 客户端提现
export withdrawForShopMember from './withdrawForShopMember'; // 店面端提现

```

* PDShopServer/project/App/routers/posts/libs/account/rechargeForClient.js

```js
import { ShipperModel, ClientModel } from '../../../../models';
import { sequelize, AccountModel, BillModel } from '../../../../mysql';
import { _T, _Y, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

const ERROR_OTHER = '10004';
export default ({
    userId,
    amount, // 单位：分
    thirdpartyAccount, // 第三方的货单号  0：支付宝(A)， 1：财付通(C)，2：工商银行(G)，3：建设银行(J)， 4：中国银行（Z）， 5：农业银行(N)，规则：缩写字母#第三方的货单号#第三方账号
}) => {
    return new Promise(async resolve => {
        const client = await ClientModel.findById(userId)
        .populate({
            path: 'shipperId',
            select: { chairManId: 1 },
        });
        const targetId = client.shipper ? _T(client.shipper.chairManId) : userId;

        if (client.shipper && !hasAuthority(client, CONSTANTS.AH_RECHARGE)) {
            return resolve({ msg: '你没有充值的权限' });
        }
        sequelize.transaction(async (transaction) => {
            const clientAccount = await AccountModel.findOne({ where: { targetId, type: CONSTANTS.AT_CLIENT }, transaction });
            const acountAmount = _Y(clientAccount.amount + amount);
            await AccountModel.increment({ amount }, { where: { id: CONSTANTS.AC_BANK_ACCOUNT }, transaction }); // 银行对应的账户增加amount
            await clientAccount.increment({ amount }, { transaction }); // 物流公司对应的账户增加amount
            await BillModel.create({ account: clientAccount.id, thirdpartyAccount, tradeAmount: amount, remark: client.shipper ? '物流公司充值' : '个人充值' }, { transaction });

            if (client.shipper) {
                // 充值成功后，需要更新物流公司的账目
                const doc = await ShipperModel.findByIdAndUpdate(client.shipper.id, { acountAmount }).catch(e => {
                    throw e;
                });
                if (!doc) {
                    throw ERROR_OTHER;
                }
            }
        }).then(async () => {
            return resolve({ success: true });
        }).catch(function (err) {
            console.log('[mysql transaction err]:', err);
            return resolve({ msg: '充值失败' });
        });
    });
};

```

* PDShopServer/project/App/routers/posts/libs/account/rechargeForShopMember.js

```js
import { ShopMemberModel } from '../../../../models';
import { sequelize, AccountModel, BillModel } from '../../../../mysql';
import { _T, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default ({
    userId,
    amount, // 单位：分
    thirdpartyAccount, //第三方的账号  0：支付宝(A)， 1：财付通(C)，2：工商银行(G)，3：建设银行(J)， 4：中国银行（Z）， 5：农业银行(N)，规则，给定的缩写字母+账号
}) => {
    return new Promise(async resolve => {
        const member = await ShopMemberModel.findById(userId);
        if (!hasAuthority(member, CONSTANTS.AH_RECHARGE) || !member.partmentId) {
            return resolve({ msg: '你没有充值的权限' });
        }
        sequelize.transaction(async (transaction) => {
            const partmentAccount = await AccountModel.findOne({ where: { targetId: _T(member.partmentId), type: CONSTANTS.AT_BRANCH_SHOP_PARTMENT }, transaction });
            await AccountModel.increment({ amount }, { where: { id: CONSTANTS.AC_BANK_ACCOUNT }, transaction }); // 银行对应的账户增加amount
            await partmentAccount.increment({ amount }, { transaction }); // 部门对应的账户增加amount
            await BillModel.create({ account: partmentAccount.id, thirdpartyAccount, tradeAmount: amount, remark: '部门充值' }, { transaction });
        }).then(async () => {
            return resolve({ success: true });
        }).catch(function (err) {
            console.log('[mysql transaction err]:', err);
            return resolve({ msg: '充值失败' });
        });
    });
};

```

* PDShopServer/project/App/routers/posts/libs/account/withdrawForClient.js

```js
import { ShipperModel, ClientModel } from '../../../../models';
import { sequelize, AccountModel, BillModel } from '../../../../mysql';
import { _T, _Y, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

const ERROR_OTHER = '10004';
export default ({
    userId,
    amount,
    thirdpartyAccount,  //第三方的账号  0：支付宝(A)， 1：财付通(C)，2：工商银行(G)，3：建设银行(J)， 4：中国银行（Z）， 5：农业银行(N)，规则，给定的缩写字母+账号
}) => {
    return new Promise(async resolve => {
        const client = await ClientModel.findById(userId)
        .populate({
            path: 'shipperId',
            select: { chairManId: 1 },
        })
        .populate({
            path: 'agentId',
            select: { chairManId: 1 },
        });
        if (!client) {
            return resolve({ msg: '你没有提现的权限' });
        }
        const targetId = client.shipper ? _T(client.shipper.chairManId) : client.agent ? _T(client.agent.chairManId) : userId;
        if ((client.shipper || client.agent) && !hasAuthority(client, CONSTANTS.AH_WITHDRAW)) {
            return resolve({ msg: '你没有提现的权限' });
        }
        let acountAmount = 0;
        sequelize.transaction(async (transaction) => {
            const clientAccount = await AccountModel.findOne({ where: { targetId, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } }, transaction });
            acountAmount = _Y(clientAccount.amount - amount);
            await AccountModel.increment({ amount: -amount }, { where: { id: CONSTANTS.AC_BANK_ACCOUNT }, transaction }); // 银行对应的账户减少amount
            await clientAccount.decrement({ amount }, { transaction }); // 物流公司对应的账户减少amount
            await BillModel.create({ account: clientAccount.id, thirdpartyAccount, tradeAmount: -amount, remark: client.shipper ? '物流公司提现' : '个人提现' }, { transaction });

            if (client.shipper) {
                // 提现成功后，需要更新物流公司的账目
                const doc = await ShipperModel.findByIdAndUpdate(client.shipper.id, { acountAmount }).catch(e => {
                    throw e;
                });
                if (!doc) {
                    throw ERROR_OTHER;
                }
            }
        }).then(async () => {
            return resolve({ success: true, context: { amount: acountAmount } });
        }).catch(function (err) {
            console.log('[mysql transaction err]:', err);
            return resolve({ msg: '提现失败' });
        });
    });
};

```

* PDShopServer/project/App/routers/posts/libs/account/withdrawForShopMember.js

```js
import { ShopMemberModel } from '../../../../models';
import { sequelize, AccountModel, BillModel } from '../../../../mysql';
import { _T, _Y, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default ({
    userId,
    amount,
    thirdpartyAccount, //第三方的账号  0：支付宝(A)， 1：财付通(C)，2：工商银行(G)，3：建设银行(J)， 4：中国银行（Z）， 5：农业银行(N)，规则，给定的缩写字母+账号
}) => {
    return new Promise(async resolve => {
        const member = await ShopMemberModel.findById(userId)
        .populate({
            path: 'shopId',
            select: { isMasterShop: 1 },
        });
        if (!hasAuthority(member, CONSTANTS.AH_WITHDRAW)) {
            return resolve({ msg: '你没有提现的权限' });
        }

        if (member.shop.isMasterShop) {
            let remainAmount = 0;
            sequelize.transaction(async (transaction) => {
                const masterAccount = await AccountModel.findOne({ where: { id: CONSTANTS.AC_MASTER_ACCOUNT, amount: { $gte: amount } }, transaction });
                await AccountModel.increment({ amount: -amount }, { where: { id: CONSTANTS.AC_BANK_ACCOUNT }, transaction }); // 银行对应的账户减少amount
                await masterAccount.decrement({ amount }, { transaction }); // 总部对应的账户减少amount
                await BillModel.create({ account: masterAccount.id, thirdpartyAccount, tradeAmount: -amount, remark: '总部提现' }, { transaction });
                remainAmount = _Y(masterAccount.amount - amount);
            }).then(async () => {
                return resolve({ success: true, context: { amount: remainAmount } });
            }).catch(function (err) {
                console.log('[mysql transaction err]:', err);
                return resolve({ msg: '余额不足' });
            });
        } else if (member.partmentId) {
            let remainAmount = 0;
            sequelize.transaction(async (transaction) => {
                const partmentAccount = await AccountModel.findOne({ where: { targetId: _T(member.partmentId), type: CONSTANTS.AT_BRANCH_SHOP_PARTMENT, amount: { $gte: amount } }, transaction });
                await AccountModel.increment({ amount: -amount }, { where: { id: CONSTANTS.AC_BANK_ACCOUNT }, transaction }); // 银行对应的账户减少amount
                await partmentAccount.decrement({ amount }, { transaction }); // 部门对应的账户减少amount
                await BillModel.create({ account: partmentAccount.id, thirdpartyAccount, tradeAmount: -amount, remark: '部门提现' }, { transaction });
                remainAmount = _Y(partmentAccount.amount - amount);
            }).then(async () => {
                return resolve({ success: true, context: { amount: remainAmount } });
            }).catch(function (err) {
                console.log('[mysql transaction err]:', err);
                return resolve({ msg: '余额不足' });
            });
        } else {
            let remainAmount = 0;
            sequelize.transaction(async (transaction) => {
                const branchAccount = await AccountModel.findOne({ where: { targetId: _T(member.shop.id), type: CONSTANTS.AT_BRANCH_SHOP, amount: { $gte: amount } }, transaction });
                await AccountModel.increment({ amount: -amount }, { where: { id: CONSTANTS.AC_BANK_ACCOUNT }, transaction }); // 银行对应的账户减少amount
                await branchAccount.decrement({ amount }, { transaction }); // 分店对应的账户减少amount
                await BillModel.create({ account: branchAccount.id, thirdpartyAccount, tradeAmount: -amount, remark: '分店提现' }, { transaction });
                remainAmount = _Y(branchAccount.amount - amount);
            }).then(async () => {
                return resolve({ success: true, context: { amount: remainAmount } });
            }).catch(function (err) {
                console.log('[mysql transaction err]:', err);
                return resolve({ msg: '余额不足' });
            });
        }
    });
};

```

* PDShopServer/project/App/routers/posts/libs/address/getAddressFromLastCode.js

```js
import { RegionModel } from '../../../../mysql';
import _ from 'lodash';

export default async (addressLastCode = 0, withTown) => {
    if (addressLastCode === 0) {
        const list = await RegionModel.findAll({ where: { parentCode: 0 } });
        return [ list ];
    }

    const addressList = [];
    while (true) {
        const item = await RegionModel.findOne({ where: { code: addressLastCode } });
        if (!item) {
            break;
        }
        if (item.parentCode === 0) {
            const list = await RegionModel.findAll({ where: { parentCode: 0 } });
            addressList.push(list);
            break;
        } else {
            const list = await RegionModel.findAll({ where: { parentCode: item.parentCode } });
            if (!withTown) {
                addressList.push(_.reject(list, o => o.level === 8 || o.level === 9));
            } else {
                addressList.push(list);
            }

            addressLastCode = item.parentCode;
        }
    }
    return addressList;
};

```

* PDShopServer/project/App/routers/posts/libs/address/getEndPointFromLastCode.js

```js
import { RoadmapAddressModel } from '../../../../models';
import getAddressFromLastCode from './getAddressFromLastCode';
import config from '../../../../../config';
import _ from 'lodash';

export default async (shopId, addressLastCode = 0, withTown) => {
    if (!config.useOnlyExitRoadmap) {
        return await getAddressFromLastCode(addressLastCode, withTown);
    }
    if (addressLastCode === 0) {
        const list = await RoadmapAddressModel.find({ shopId, parentCode: 0 });
        return [ list ];
    }

    const addressList = [];
    while (true) {
        const item = await RoadmapAddressModel.findOne({ shopId, code: addressLastCode });
        if (!item) {
            break;
        }
        if (item.parentCode === 0) {
            const list = await RoadmapAddressModel.find({ shopId, parentCode: 0 });
            addressList.push(list);
            break;
        } else {
            const list = await RoadmapAddressModel.find({ shopId, parentCode: item.parentCode });
            if (!withTown) {
                addressList.push(_.reject(list, o => o.level === 8 || o.level === 9));
            } else {
                addressList.push(list);
            }
            addressLastCode = item.parentCode;
        }
    }
    if (!addressList.length) {
        const list = await RoadmapAddressModel.find({ shopId, parentCode: 0 });
        return [ list ];
    }
    return addressList;
};

```

* PDShopServer/project/App/routers/posts/libs/address/getSendDoorAddressFromLastCode.js

```js
import { RegionModel } from '../../../../mysql';
import _ from 'lodash';

export default async (addressLastCode, parentCode) => {
    if (!addressLastCode) {
        return parentCode ? [ await RegionModel.findAll({ where: { parentCode } }) ] : [];
    }

    const addressList = [];
    while (true) {
        const item = await RegionModel.findOne({ where: { code: addressLastCode } });
        if (!item || item.level === 7 || item.level === 6) {
            break;
        }
        const list = await RegionModel.findAll({ where: { parentCode: item.parentCode } });
        addressList.push(_.reject(list, o => o.level === 4 || o.level === 5 || o.level === 6 || o.level === 7));
        if (item.level === 8 || item.level === 9) {
            break;
        } else {
            addressLastCode = item.parentCode;
        }
    }
    return addressList;
};

```

* PDShopServer/project/App/routers/posts/libs/address/getSendDoorLastCodeList.js

```js
import { RegionModel } from '../../../../mysql';

const getChildList = async (code, name, list) => {
    const docs = await RegionModel.findAll({ where: { parentCode: code } });
    for (const doc of docs) {
        if (doc.level === 10) {
            list.push({ code: doc.code, address: name + doc.name, sendDoorPrice: 110, sendDoorMinFee: 60 }); // todo： 需要计算该方向的平均成交价
        } else {
            await getChildList(doc.code, name + doc.name, list);
        }
    }
};

// 获取长途路线的送货上门的地址列表
export default async (code) => {
    const list = [];
    await getChildList(code, '', list);
    return list;
};

```

* PDShopServer/project/App/routers/posts/libs/address/index.js

```js
export getSendDoorLastCodeList from './getSendDoorLastCodeList'; // 获取所有的送货上门地址列表
export getAddressFromLastCode from './getAddressFromLastCode'; // 通过最后一个code获取地址列表
export getSendDoorAddressFromLastCode from './getSendDoorAddressFromLastCode'; // 通过最后一个code获取送货上门地址列表
export registeredStartPointAddress from './registeredStartPointAddress'; // 注册发货地址（包括分店和收货点）
export registeredRoadmapAddress from './registeredRoadmapAddress'; // 注册路线地址
export getEndPointFromLastCode from './getEndPointFromLastCode'; // 通过最后一个code获取地址列表(只获取分店里面已经拥有的路线)

```

* PDShopServer/project/App/routers/posts/libs/address/registeredRoadmapAddress.js

```js
import { RoadmapAddressModel, RoadmapModel } from '../../../../models';
import { RegionModel } from '../../../../mysql';

// 该接口只注册长途的地址，同城配送的不注册
export default async (shopId, endPointLastCode, oldEndPointLastCode) => {
    if (endPointLastCode === oldEndPointLastCode) {
        return;
    }
    if (oldEndPointLastCode) {
        const item = await RoadmapAddressModel.findOne({ shopId, code: oldEndPointLastCode });
        if (item) {
            const roadmap = await RoadmapModel.findOne({ shopId, endPointLastCode: oldEndPointLastCode, enable: true });
            if (!roadmap) {
                await item.remove();
                let parentCode = item.parentCode;
                while (true) {
                    if (await RoadmapAddressModel.findOne({ shopId, parentCode })) {
                        break;
                    }
                    const parent = await RoadmapAddressModel.findOne({ shopId, code: parentCode });
                    if (!parent) {
                        break;
                    }
                    parentCode = parent.parentCode;
                    await parent.remove();
                }
            }
        }
    }
    if (endPointLastCode) {
        while (true) {
            const item = await RoadmapAddressModel.findOne({ shopId, code: endPointLastCode });
            if (item) { // 如果已经存在，则无需再添加
                break;
            }
            const node = await RegionModel.findOne({ where: { code: endPointLastCode } });
            const doc = new RoadmapAddressModel({ shopId, parentCode: node.parentCode, code: node.code, name: node.name, level: node.level });
            await doc.save();
            if (node.parentCode === 0) {
                break;
            }
            endPointLastCode = node.parentCode;
        }
    }
};

```

* PDShopServer/project/App/routers/posts/libs/address/registeredStartPointAddress.js

```js
import { StartPointAddressModel, ShopModel, AgentModel } from '../../../../models';
import { RegionModel } from '../../../../mysql';

export default async (addressLastCode, oldAddressLastCode) => {
    if (!addressLastCode || addressLastCode === oldAddressLastCode) {
        return;
    }
    if (oldAddressLastCode) {
        const item = await StartPointAddressModel.findOne({ code: oldAddressLastCode });
        if (item) {
            const shopAgent = await ShopModel.findOne({ isMasterShop: false, addressRegionLastCode: oldAddressLastCode }) ||
            await AgentModel.findOne({ addressRegionLastCode: oldAddressLastCode });
            if (!shopAgent) {
                await item.remove();
                let parentCode = item.parentCode;
                while (true) {
                    if (await StartPointAddressModel.findOne({ parentCode })) {
                        break;
                    }
                    const parent = await StartPointAddressModel.findOne({ code: parentCode });
                    if (!parent) {
                        break;
                    }
                    parentCode = parent.parentCode;
                    await parent.remove();
                }
            }
        }
    }

    while (true) {
        const item = await StartPointAddressModel.findOne({ code: addressLastCode });
        if (item) { // 如果已经存在，则无需再添加
            break;
        }
        const node = await RegionModel.findOne({ where: { code: addressLastCode } });
        const doc = new StartPointAddressModel({ parentCode: node.parentCode, code: node.code, name: node.name, level: node.level });
        await doc.save();
        if (node.parentCode === 0) {
            break;
        }
        addressLastCode = node.parentCode;
    }
};

```

* PDShopServer/project/App/routers/posts/libs/order/getAgentLatestOrder.js

```js
import { OrderModel } from '../../../../models';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async (receiveAgentMemberId) => {
    const order = await OrderModel.findOne({ receiveAgentMemberId })
    .sort({ placeOrderTime: 'desc' });
    if (!order || !_.find(order.stateList, o => _.includes([
        CONSTANTS.OS_READY_PRINT_BAR_CODE,
        CONSTANTS.OS_READY_PAYMENT,
        CONSTANTS.OS_READY_PRINT_ORDER,
        CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER,
    ], o.state))) {
        return null;
    }
    return order;
};

```

* PDShopServer/project/App/routers/posts/libs/order/getAgentOrderFeeByEndPoint.js

```js
import mongoose from 'mongoose';
import { RegionModel } from '../../../../mysql';
import { AgentRegionProfitModel, OrderModel, RoadmapModel } from '../../../../models';
import { settingMgr } from '../../../../manager';
import getRegionProfitRateByEndPoint from '../roadmap/getRegionProfitRateByEndPoint';
import CONSTANTS from '../../../../constants';
import moment from 'moment';

async function getProfitRate (agentId, shopId, isCityDistribute, endPointLastCode) {
    let addressList = [0];
    while (true) {
        const item = await RegionModel.findOne({ where: { code: endPointLastCode } });
        if (!item) {
            break;
        } else {
            addressList.push(item.code);
            endPointLastCode = item.parentCode;
        }
    }
    const regionRate = await AgentRegionProfitModel.find({
        agentId,
        shopId,
        regionLastCode: { $in: addressList },
        type: { $in: [CONSTANTS.RRT_ALL, !isCityDistribute ? CONSTANTS.RRT_LONG : CONSTANTS.RRT_CITY] },
    }).sort({ regionLastCode: -1, type: 1 }).limit(1);

    return regionRate[0] || {};
}

async function getAverageRoadmapPrice (shopId, isCityDistribute, isSendDoor, endPointLastCode, sendDoorEndPointLastCode) {
    let docs = [];
    const regionProfitRate = await getRegionProfitRateByEndPoint(shopId, endPointLastCode, isCityDistribute);
    if (!isSendDoor) {
        docs = await RoadmapModel.aggregate()
        .match({ shopId, endPointLastCode, enable: true })
        .project({
            profitRate: { $ifNull: [ '$profitRate', regionProfitRate ] },
            price: 1,
            minFee: 1,
        })
        .project({
            price: { $multiply: [ '$price', { $sum: [ 1, '$profitRate' ] } ] },
            minFee: { $multiply: [ '$minFee', { $sum: [ 1, '$profitRate' ] } ] },
        })
        .group({
            _id: null,
            price: { $avg: '$price' },
            minFee: { $avg: '$minFee' },
        });
    } else {
        docs = await RoadmapModel.aggregate()
        .match({ shopId, endPointLastCode, enable: true })
        .project({
            sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
            profitRate: { $ifNull: [ '$profitRate', regionProfitRate ] },
            price: 1,
            minFee: 1,
        })
        .project({
            price: { $multiply: [ '$price', { $sum: [ 1, '$profitRate' ] } ] },
            minFee: { $multiply: [ '$minFee', { $sum: [ 1, '$profitRate' ] } ] },
            sendDoorPrice: { $multiply: [ '$sendDoor.sendDoorPrice', { $sum: [ 1, '$profitRate' ] } ] },
            sendDoorMinFee: { $multiply: [ '$sendDoor.sendDoorMinFee', { $sum: [ 1, '$profitRate' ] } ] },
        })
        .group({
            _id: null,
            price: { $avg: '$price' },
            minFee: { $avg: '$minFee' },
            sendDoorPrice: { $avg: '$sendDoorPrice' },
            sendDoorMinFee: { $avg: '$sendDoorMinFee' },
        });
    }

    return docs[0];
}

async function getAverageOrderPrice (shopId, endPointLastCode, isSendDoor, sendDoorEndPointLastCode) {
    const criteria = {
        shopId: new mongoose.Types.ObjectId(shopId),
        'stateList.0.state': { $gte: CONSTANTS.OS_READY_STOCK },
        endPointLastCode,
        placeOrderTime: { $lt: moment().subtract(10, 'd').toDate() },
    };
    const params = {
        _id: null,
        price: { $avg: '$price' },
        minFee: { $avg: '$minFee' },
    };
    if (isSendDoor) {
        Object.assign(criteria, { isSendDoor, sendDoorEndPointLastCode });
        Object.assign(params, { sendDoorPrice: { $avg: '$sendDoorPrice' }, sendDoorMinFee: { $avg: '$sendDoorMinFee' } });
    }

    const docs = await OrderModel.aggregate()
    .match(criteria)
    .group(params);

    return docs[0];
}

export default async (
    agentId, // 所在所在点
    shopId, // 参考分店
    isCityDistribute, // 是否是同城配送
    endPointLastCode, // 终点
    isSendDoor, // 是否送货上门
    sendDoorEndPointLastCode, // 送货上门终点
    weight, // 重量
    size, //方量
) => {
    weight = settingMgr.getFloatWeight(weight, size);
    const rate = await getProfitRate(agentId, shopId, isCityDistribute, endPointLastCode);
    let roadmap = await getAverageOrderPrice(shopId, endPointLastCode, isSendDoor, sendDoorEndPointLastCode);
    if (!roadmap) {
        roadmap = await getAverageRoadmapPrice(shopId, isCityDistribute, isSendDoor, endPointLastCode, sendDoorEndPointLastCode);
    }
    if (!roadmap) {
        return { error: true };
    }

    // 分店显示的价格;
    const fee = Math.floor(Math.max(roadmap.price * weight, roadmap.minFee)
    + (isSendDoor ? Math.max(roadmap.sendDoorPrice * weight, roadmap.sendDoorMinFee) : 0));

    let totalFee;
    if (rate.profitCount) {
        totalFee = fee + rate.profitCount * weight;
    } else {
        totalFee = (1 + (rate.profitRate || 0)) * fee;
    }

    return {
        fee, // 分店显示的价格
        profit: totalFee - fee, //收货点的利润
    };
};

```

* PDShopServer/project/App/routers/posts/libs/order/getRPLatestOrder.js

```js
import { OrderModel } from '../../../../models';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async (receiveMemberId) => {
    const order = await OrderModel.findOne({ receiveMemberId })
    .sort({ placeOrderTime: 'desc' });
    if (!order || !_.find(order.stateList, o => _.includes([
        CONSTANTS.OS_READY_PRINT_BAR_CODE,
        CONSTANTS.OS_READY_SELECT_ROADMAP,
        CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER,
        CONSTANTS.OS_READY_PAYMENT,
        CONSTANTS.OS_READY_PRINT_ORDER,
    ], o.state))) {
        return null;
    }
    return order;
};

```

* PDShopServer/project/App/routers/posts/libs/order/index.js

```js
export getRPLatestOrder from './getRPLatestOrder'; // 获取收货部最新的货单
export getAgentLatestOrder from './getAgentLatestOrder'; // 获取收货点最新的货单
export setAgentOrderFee from './setAgentOrderFee'; // 设置收货点的货单的价格
export setClientPickShipper from './setClientPickShipper'; // 设置用户上门自提的信息

```

* PDShopServer/project/App/routers/posts/libs/order/setAgentOrderFee.js

```js
import CONSTANTS from '../../../../constants';
import { setting } from '../../../../manager';
import getAgentOrderFeeByEndPoint from './getAgentOrderFeeByEndPoint';

export default async (referShopId, order) => {
    const options = await getAgentOrderFeeByEndPoint(
        order.agentId,
        referShopId,
        order.isCityDistribute,
        order.endPointLastCode,
        order.isSendDoor,
        order.sendDoorEndPointLastCode,
        order.weight,
        order.size,
    );
    if (options.error) {
        return true;
    }

    order.realFee = options.profit + options.fee; // 实际运费.
    if (order.payMode !== CONSTANTS.PM_IMMEDIATE && !order.totalDesignatedFee) {
        order.totalDesignatedFee = order.realFee;
    }
    order.needPayTransportFee = 0;
    if (order.payMode !== CONSTANTS.PM_IMMEDIATE) { // 到付或者混合支付
        const senderProfit = order.totalDesignatedFee - order.realFee; // 发货人收益
        order.designatedFee = Math.max(senderProfit, 0);
        if (senderProfit < 0) {
            order.needPayTransportFee = -senderProfit; // 发货人需要现付的部分
        }
    } else { // 现付
        order.needPayTransportFee = order.realFee;
    }
    // 保险费（参考分店的收钱的部分买保险）
    if (order.isInsuance) {
        order.insuanceMount = options.fee * setting.insuanceMountRate;
        order.insuanceFee = order.insuanceMount * setting.insuanceRate;
        if (order.insuanceFee < setting.insuanceBaseValue) { // 如果保险
            order.insuanceFee = setting.insuanceBaseValue;
        }
        order.insuanceFee = Math.floor(order.insuanceFee);
        order.insuanceMount = Math.floor(order.insuanceFee / setting.insuanceRate);
    } else {
        order.insuanceMount = 0;
        order.insuanceFee = 0;
    }

    order.needPayInsuanceFee = 0;
    if (order.insuanceFee > 0) { // 只有初始单才需要交保险
        order.needPayInsuanceFee = order.insuanceFee;
    }
    if (order.needPayTransportFee > 0 && order.totalDesignatedFee > 0) {
        order.payMode = CONSTANTS.PM_MIXED;
    }

    return false;
};

```

* PDShopServer/project/App/routers/posts/libs/order/setClientPickShipper.js

```js
import mongoose from 'mongoose';
import { sequelize, AccountModel, BillModel, OrderPaymentModel } from '../../../../mysql';
import { ShipperInBranchShopModel, RoadmapMaskModel, WarehouseModel, ShipperModel } from '../../../../models';
import { _T, _F, _Y } from '../../../../utils';
import { settingMgr } from '../../../../manager';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

const ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH = '10003';
const ERROR_OTHER = '10004';
function deductAmount (order, shipperId, needBondAmount, targetId, proxyCharge, totalDesignatedFee, profit) {
    return new Promise(async resolve => {
        const orderId = _T(order.id);
        const amount = proxyCharge + totalDesignatedFee + profit;
        sequelize.transaction(async (transaction) => {
            const shipperAccount = await AccountModel.findOne({ where: { targetId, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } }, transaction });
            await shipperAccount.decrement({ amount }, { transaction });
            await AccountModel.increment({ amount }, { where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
            proxyCharge > 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -proxyCharge, remark: '抵押货款' }, { transaction });
            totalDesignatedFee > 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -totalDesignatedFee, remark: '抵押指定收款' }, { transaction });
            profit > 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -profit, remark: '扣除上门自提竞价' }, { transaction });
            const acountAmount = _Y(shipperAccount.amount - amount);

            await OrderPaymentModel.create({ orderId, isPayed: true, payedTime: new Date() }, { transaction });

            // 扣除物流公司的担保金额
            const inShop = await ShipperInBranchShopModel.findOne({ shipperId, shopId: order.shopId });
            if (!inShop || inShop.remainBondAmount < needBondAmount) {
                throw ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH;
            }
            inShop.remainBondAmount = inShop.remainBondAmount - needBondAmount;
            inShop.usedBondAmount = inShop.totalBondAmount - inShop.remainBondAmount;
            await inShop.save();

            // 扣款成功后，需要更新物流公司的账目
            const doc = await ShipperModel.findByIdAndUpdate(shipperId, { acountAmount }).catch(e => {
                throw e;
            });
            if (!doc) {
                throw ERROR_OTHER;
            }
        }).then(() => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}

export default async (order, profitRate) => {
    const weight = settingMgr.getFloatWeight(order.weight, order.size);
    const needBondAmount = settingMgr.getBondAmount(order.weight, order.size);
    const docs = await ShipperInBranchShopModel.aggregate()
    .match({ shopId: new mongoose.Types.ObjectId(order.shopId) })
    .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
    .project({
        shipperId: { $arrayElemAt: ['$shipper._id', 0] },
        shipperName: { $arrayElemAt: ['$shipper.name', 0] },
        shipperChairManId: { $arrayElemAt: ['$shipper.chairManId', 0] },
        shipperType: { $arrayElemAt: ['$shipper.shipperType', 0] },
        profit: { $multiply: ['$clientPickPrice', weight] },
        hasEnoughAmount: {
            $cond: { if: {
                $gte: [ { $arrayElemAt: ['$shipper.acountAmount', 0] },
                { $sum: [ order.proxyCharge, order.totalDesignatedFee, { $multiply: ['$clientPickPrice', weight] } ] }] },
                then: true,
                else: false,
            },
        },
        remainBondAmount: 1,
        clientPickEnable: 1,
        clientPickPrice: 1,
    })
    .match({
        shipperType: CONSTANTS.ST_CITY,
        clientPickEnable: true,
        clientPickPrice: { $exists: true },
        hasEnoughAmount: true, // 需要押的钱 = 代收货款 + 指定收款 + 竞价
        remainBondAmount: { $gte: needBondAmount }, //保证金
    })
    .sort({
        clientPickPrice: -1,
    });
    const maskNumber = await RoadmapMaskModel.getMaskNumber(order.shopId, 0, docs.length);
    const item = docs[maskNumber];
    if (item) {
        const err = await deductAmount(order, item.shipperId, needBondAmount, _T(item.shipperChairManId), _F(order.proxyCharge), _F(order.totalDesignatedFee), _F(item.profit));
        if (err) {
            order.deductError = true;
        } else {
            const warehouseList = await WarehouseModel.find({ shopId: order.shopId, shipperList: item.shipperId });
            order.shipperId = item.shipperId;
            order.shipperChairManId = item.shipperChairManId;
            order.warehouse = warehouseList.length ? _.map(warehouseList, o => o.houseNo).join(';') : item.shipperName;
            order.profit = item.clientPickPrice * weight;
            order.branchProfit = order.profit * profitRate;
            order.masterProfit = order.profit - order.branchProfit;
            order.needBondAmount = needBondAmount;
            order.designatedFee = order.totalDesignatedFee;
            await RoadmapMaskModel.updateStatusList(order.shopId, 0, maskNumber, settingMgr.getRealWeight(order.weight, order.size));
        }
    } else {
        order.deductError = true;
    }
};

```

* PDShopServer/project/App/routers/posts/libs/roadmap/getRegionProfitRateByEndPoint.js

```js
import { RegionModel } from '../../../../mysql';
import { RoadmapRegionProfitRateModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async (
    shopId, // 所在分店
    endPointLastCode, // 终点
    isCityDistribute, // 是否是同城配送
) => {
    // 查找该方向的路线提成
    let addressList = [0];
    while (true) {
        const item = await RegionModel.findOne({ where: { code: endPointLastCode } });
        if (!item) {
            break;
        } else {
            addressList.push(item.code);
            endPointLastCode = item.parentCode;
        }
    }

    const regionRate = await RoadmapRegionProfitRateModel.find({
        shopId: { $in: [null, shopId] },
        regionLastCode: { $in: addressList },
        type: { $in: [CONSTANTS.RRT_ALL, !isCityDistribute ? CONSTANTS.RRT_LONG : CONSTANTS.RRT_CITY] },
    }).sort({ regionLastCode: -1, type: 1 }).limit(1);

    return regionRate.length ? regionRate[0].profitRate : 0;
};

```

* PDShopServer/project/App/routers/posts/libs/roadmap/index.js

```js
export searchRoadmapListByReceivePartment from './searchRoadmapListByReceivePartment'; // 使用收货部的身份查询路线（待屏蔽功能，限定在本店）
export searchRoadmapListWithShop from './searchRoadmapListWithShop'; // 查询分店的路线排名（不屏蔽）
export searchRoadmapListWithShopList from './searchRoadmapListWithShopList'; // 查询多家分店的路线排名（不屏蔽）
export getRegionProfitRateByEndPoint from './getRegionProfitRateByEndPoint'; // 查询多家分店的路线排名（不屏蔽）

```

* PDShopServer/project/App/routers/posts/libs/roadmap/searchRoadmapListByReceivePartment.js

```js
import mongoose from 'mongoose';
import { RoadmapModel, RoadmapMaskModel } from '../../../../models';
import { settingMgr } from '../../../../manager';
import { omitNil } from '../../../../utils';
import _ from 'lodash';

export default async ({
    shopId,
    fromPC,
    pageNo,
    pageSize,
    endPointLastCode, // 终点
    sendDoorEndPointLastCode, // 送货上门终点
    isSendDoor, // 是否送货上门
    weight, // 重量
    size, // 方量
    proxyCharge, // 代收货款金额
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee, // 指定向收货人应该收多少钱
    orderBy, // 排序方式, 0：以价格优先， 1：以时长优先
    regionProfitRate, //方向提成比例
}) => {
    weight = settingMgr.getFloatWeight(weight, size); // 通过weight和size和阶梯价格计算重量
    const orderText = orderBy === 1 ? 'fee duration' : 'duration fee';
    const needBondAmount = settingMgr.getBondAmount(weight, size);
    let count;

    if (fromPC && pageNo === 0 && pageSize !== 1) {
        const countResults = await RoadmapModel.aggregate()
        .match(omitNil({ shopId: new mongoose.Types.ObjectId(shopId), endPointLastCode, enable: true }))
        .project({
            sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
            shipperInBranchShopId: 1,
            shipperId: 1,
            price: 1,
            minFee: 1,
            profitRate: 1,
        })
        .project({
            sendDoorEnable: '$sendDoor.enable',
            sendDoorPrice: '$sendDoor.sendDoorPrice',
            sendDoorMinFee: '$sendDoor.sendDoorMinFee',
            shipperInBranchShopId: 1,
            shipperId: 1,
            price: 1,
            minFee: 1,
            profitRate: 1,
        })
        .match(isSendDoor ? { sendDoorEnable: true } : {})
        .lookup({ from: 'shipperinbranchshops', localField: 'shipperInBranchShopId', foreignField: '_id', as: 'shipperinbranchshop' })
        .project({
            remainBondAmount: { $arrayElemAt: ['$shipperinbranchshop.remainBondAmount', 0] },
            fee: {
                $floor: { $sum: [
                    { $max: [ { $multiply: ['$price', weight] }, '$minFee' ] },
                    { $cond: { if: isSendDoor, then: { $max: [ { $multiply: ['$sendDoorPrice', weight] }, '$sendDoorMinFee' ] }, else: 0 } },
                ] },
            },
            profitRate: { $ifNull: [ '$profitRate', regionProfitRate ] },
            shipperId: 1,
        })
        .match({ remainBondAmount: { $gte: needBondAmount } })
        .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
        .project({
            acountAmount: { $arrayElemAt: ['$shipper.acountAmount', 0] },
            profit: { $floor: { $multiply: [ '$fee', '$profitRate' ] } },
            fee: 1,
        })
        .project({
            hasEnoughAmount: {
                $cond: {
                    if: { $gte: [{
                        $subtract: [ '$acountAmount', {
                            $sum: [
                                proxyCharge,
                                {
                                    $cond: {
                                        if: isReachPay,
                                        then: {
                                            $cond: {
                                                if: totalDesignatedFee,
                                                // 如果有指定收款，需要扣除的钱为  totalDesignatedFee - fee
                                                then: { $subtract: [ totalDesignatedFee, '$fee' ] },
                                                // 否则，就是总部提成+分部提成，因为物流公司向收货人收取所有运费
                                                else: '$profit',
                                            },
                                        },
                                        else: 0,
                                    },
                                },
                            ],
                        } ],
                    }, 0] },
                    then: true,
                    else: false,
                },
            },
        })
        .match({ hasEnoughAmount: true })
        .group({
            _id: null,
            total: { $sum: 1 },
        });
        count = countResults.length ? countResults[0].total : 0;
    }

    const docs = await RoadmapModel.aggregate()
    .match(omitNil({ shopId: new mongoose.Types.ObjectId(shopId), endPointLastCode, enable: true }))
    .project({
        sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
        shipperInBranchShopId: 1,
        shipperId: 1,
        price: 1,
        minFee: 1,
        profitRate: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .project({
        sendDoorEnable: '$sendDoor.enable',
        sendDoorPrice: '$sendDoor.sendDoorPrice',
        sendDoorMinFee: '$sendDoor.sendDoorMinFee',
        shipperInBranchShopId: 1,
        shipperId: 1,
        price: 1,
        minFee: 1,
        profitRate: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .match(isSendDoor ? { sendDoorEnable: true } : {})
    .lookup({ from: 'shipperinbranchshops', localField: 'shipperInBranchShopId', foreignField: '_id', as: 'shipperinbranchshop' })
    .project({
        remainBondAmount: { $arrayElemAt: ['$shipperinbranchshop.remainBondAmount', 0] },
        fee: {
            $floor: { $sum: [
                { $max: [ { $multiply: ['$price', weight] }, '$minFee' ] },
                { $cond: { if: isSendDoor, then: { $max: [ { $multiply: ['$sendDoorPrice', weight] }, '$sendDoorMinFee' ] }, else: 0 } },
            ] },
        },
        profitRate: { $ifNull: [ '$profitRate', regionProfitRate ] },
        shipperId: 1,
        price: 1,
        minFee: 1,
        sendDoorPrice: 1,
        sendDoorMinFee: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .match({ remainBondAmount: { $gte: needBondAmount } })
    .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
    .project({
        shipperName: { $arrayElemAt: ['$shipper.name', 0] },
        acountAmount: { $arrayElemAt: ['$shipper.acountAmount', 0] },
        profit: { $floor: { $multiply: [ '$fee', '$profitRate' ] } },
        price: { $multiply: [ '$price', { $sum: [ 1, '$profitRate' ] } ] },
        minFee: { $multiply: [ '$minFee', { $sum: [ 1, '$profitRate' ] } ] },
        sendDoorPrice: { $multiply: [ '$sendDoorPrice', { $sum: [ 1, '$profitRate' ] } ] },
        sendDoorMinFee: { $multiply: [ '$sendDoorMinFee', { $sum: [ 1, '$profitRate' ] } ] },
        fee: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .project({
        id: '$_id',
        _id: 0,
        hasEnoughAmount: {
            $cond: {
                if: { $gte: [{
                    $subtract: [ '$acountAmount', {
                        $sum: [
                            proxyCharge,
                            {
                                $cond: {
                                    if: isReachPay,
                                    then: {
                                        $cond: {
                                            if: totalDesignatedFee,
                                            // 如果有指定收款，需要扣除的钱为  totalDesignatedFee - fee
                                            then: { $subtract: [ totalDesignatedFee, '$fee' ] },
                                            // 否则就是总部提成+分部提成，因为物流公司向收货人收取所有运费
                                            else: '$profit',
                                        },
                                    },
                                    else: 0,
                                },
                            },
                        ],
                    } ],
                }, 0] },
                then: true,
                else: false,
            },
        },
        transportFee: { $sum: ['$fee', '$profit'] },
        needBondAmount: { $sum: [ needBondAmount ] },
        shipperName: 1,
        price: 1,
        minFee: 1,
        sendDoorPrice: 1,
        sendDoorMinFee: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
        // delete below
        profit: 1,
        acountAmount: 1,
        fee: 1,
    })
    .match({ hasEnoughAmount: true })
    .sort(orderText)
    .skip(pageNo * pageSize)
    .limit(pageSize);

    let maskNumber = 0;
    if (pageNo === 0) {
        maskNumber = await RoadmapMaskModel.getMaskNumber(shopId, endPointLastCode, docs.length);
    }
    const roadmapList = _.drop(docs, maskNumber).map((o, i) => {
        o.roadmapRankIndex = i + pageNo * pageSize + maskNumber; // 该路线的排名，选择路线的时候需要上传这个参数
        return o;
    });

    return {
        count: count ? count - maskNumber : undefined,
        roadmapList,
    };
};

```

* PDShopServer/project/App/routers/posts/libs/roadmap/searchRoadmapListWithShop.js

```js
import mongoose from 'mongoose';
import { RoadmapModel } from '../../../../models';
import { settingMgr } from '../../../../manager';
import getRegionProfitRateByEndPoint from './getRegionProfitRateByEndPoint';
import { omitNil } from '../../../../utils';

export default async ({
    shopId, // 对应的分店的Id, 如果是收货点，则对应的是收货点的referShopId
    distance, // 到分店或者到收货点的距离
    shopName, // 分店的名称，如果是收货点，不需要穿该字段
    shopAddress, // 分店的地址
    fromPC,
    pageNo,
    pageSize,
    endPointLastCode, // 终点
    sendDoorEndPointLastCode, // 送货上门终点
    isCityDistribute, // 是否是同城配送
    isSendDoor, // 是否送货上门
    weight, // 重量
    size, // 方量
    proxyCharge, // 代收货款金额
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee, // 指定向收货人应该收多少钱
    orderBy, //排序方式, 0：以价格优先， 1：以时长优先
}) => {
    weight = settingMgr.getFloatWeight(weight, size); // 通过weight和size和阶梯价格计算重量
    const orderText = orderBy === 1 ? 'fee duration' : 'duration fee';
    const needBondAmount = settingMgr.getBondAmount(weight, size);
    const regionProfitRate = await getRegionProfitRateByEndPoint(shopId, endPointLastCode, isCityDistribute);
    let count;

    if (fromPC && pageNo === 0 && pageSize !== 1) {
        const countResults = await RoadmapModel.aggregate()
        .match(omitNil({ shopId: new mongoose.Types.ObjectId(shopId), endPointLastCode, enable: true }))
        .project({
            sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
            shipperInBranchShopId: 1,
            shipperId: 1,
            price: 1,
            minFee: 1,
            profitRate: 1,
        })
        .project({
            sendDoorEnable: '$sendDoor.enable',
            sendDoorPrice: '$sendDoor.sendDoorPrice',
            sendDoorMinFee: '$sendDoor.sendDoorMinFee',
            shipperInBranchShopId: 1,
            shipperId: 1,
            price: 1,
            minFee: 1,
            profitRate: 1,
        })
        .match(isSendDoor ? { sendDoorEnable: true } : {})
        .lookup({ from: 'shipperinbranchshops', localField: 'shipperInBranchShopId', foreignField: '_id', as: 'shipperinbranchshop' })
        .project({
            remainBondAmount: { $arrayElemAt: ['$shipperinbranchshop.remainBondAmount', 0] },
            fee: {
                $floor: { $sum: [
                    { $max: [ { $multiply: ['$price', weight] }, '$minFee' ] },
                    { $cond: { if: isSendDoor, then: { $max: [ { $multiply: ['$sendDoorPrice', weight] }, '$sendDoorMinFee' ] }, else: 0 } },
                ] },
            },
            profitRate: { $ifNull: [ '$profitRate', regionProfitRate ] },
            shipperId: 1,
        })
        .match({ remainBondAmount: { $gte: needBondAmount } })
        .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
        .project({
            acountAmount: { $arrayElemAt: ['$shipper.acountAmount', 0] },
            profit: { $floor: { $multiply: [ '$fee', '$profitRate' ] } },
            fee: 1,
        })
        .project({
            hasEnoughAmount: {
                $cond: {
                    if: { $gte: [{
                        $subtract: [ '$acountAmount', {
                            $sum: [
                                proxyCharge,
                                {
                                    $cond: {
                                        if: isReachPay,
                                        then: {
                                            $cond: {
                                                if: totalDesignatedFee,
                                                // 如果有指定收款，需要扣除的钱为  totalDesignatedFee - fee
                                                then: { $subtract: [ totalDesignatedFee, '$fee' ] },
                                                // 否则，就是总部提成+分部提成，因为物流公司向收货人收取所有运费
                                                else: '$profit',
                                            },
                                        },
                                        else: 0,
                                    },
                                },
                            ],
                        } ],
                    }, 0] },
                    then: true,
                    else: false,
                },
            },
        })
        .match({ hasEnoughAmount: true })
        .group({
            _id: null,
            total: { $sum: 1 },
        });
        count = countResults.length ? countResults[0].total : 0;
    }

    const docs = await RoadmapModel.aggregate()
    .match(omitNil({ shopId: new mongoose.Types.ObjectId(shopId), endPointLastCode, enable: true }))
    .project({
        sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
        shipperInBranchShopId: 1,
        shipperId: 1,
        price: 1,
        minFee: 1,
        profitRate: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .project({
        sendDoorEnable: '$sendDoor.enable',
        sendDoorPrice: '$sendDoor.sendDoorPrice',
        sendDoorMinFee: '$sendDoor.sendDoorMinFee',
        shipperInBranchShopId: 1,
        shipperId: 1,
        price: 1,
        minFee: 1,
        profitRate: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .match(isSendDoor ? { sendDoorEnable: true } : {})
    .lookup({ from: 'shipperinbranchshops', localField: 'shipperInBranchShopId', foreignField: '_id', as: 'shipperinbranchshop' })
    .project({
        remainBondAmount: { $arrayElemAt: ['$shipperinbranchshop.remainBondAmount', 0] },
        fee: {
            $floor: { $sum: [
                { $max: [ { $multiply: ['$price', weight] }, '$minFee' ] },
                { $cond: { if: isSendDoor, then: { $max: [ { $multiply: ['$sendDoorPrice', weight] }, '$sendDoorMinFee' ] }, else: 0 } },
            ] },
        },
        profitRate: { $ifNull: [ '$profitRate', regionProfitRate ] },
        shipperId: 1,
        price: 1,
        minFee: 1,
        sendDoorPrice: 1,
        sendDoorMinFee: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .match({ remainBondAmount: { $gte: needBondAmount } })
    .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
    .project({
        shipperName: { $arrayElemAt: ['$shipper.name', 0] },
        acountAmount: { $arrayElemAt: ['$shipper.acountAmount', 0] },
        profit: { $floor: { $multiply: [ '$fee', '$profitRate' ] } },
        price: { $multiply: [ '$price', { $sum: [ 1, '$profitRate' ] } ] },
        minFee: { $multiply: [ '$minFee', { $sum: [ 1, '$profitRate' ] } ] },
        sendDoorPrice: { $multiply: [ '$sendDoorPrice', { $sum: [ 1, '$profitRate' ] } ] },
        sendDoorMinFee: { $multiply: [ '$sendDoorMinFee', { $sum: [ 1, '$profitRate' ] } ] },
        fee: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .project({
        id: '$_id',
        _id: 0,
        hasEnoughAmount: {
            $cond: {
                if: { $gte: [{
                    $subtract: [ '$acountAmount', {
                        $sum: [
                            proxyCharge,
                            {
                                $cond: {
                                    if: isReachPay,
                                    then: {
                                        $cond: {
                                            if: totalDesignatedFee,
                                            // 如果有指定收款，需要扣除的钱为  totalDesignatedFee - fee
                                            then: { $subtract: [ totalDesignatedFee, '$fee' ] },
                                            // 否则就是总部提成+分部提成，因为物流公司向收货人收取所有运费
                                            else: '$profit',
                                        },
                                    },
                                    else: 0,
                                },
                            },
                        ],
                    } ],
                }, 0] },
                then: true,
                else: false,
            },
        },
        transportFee: {
            $sum: ['$fee', '$profit'],
        },
        shipperName: 1,
        shopName: { $concat: shopName },
        shopAddress: { $concat: shopAddress },
        distance: { $abs: distance },
        price: 1,
        minFee: 1,
        sendDoorPrice: 1,
        sendDoorMinFee: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
        // delete below
        profit: 1,
        acountAmount: 1,
        fee: 1,
    })
    .match({ hasEnoughAmount: true })
    .sort(orderText)
    .skip(pageNo * pageSize)
    .limit(pageSize);

    return {
        count,
        roadmapList: docs,
    };
};

```

* PDShopServer/project/App/routers/posts/libs/roadmap/searchRoadmapListWithShopList.js

```js
import mongoose from 'mongoose';
import { RoadmapModel } from '../../../../models';
import { settingMgr } from '../../../../manager';
import getRegionProfitRateByEndPoint from './getRegionProfitRateByEndPoint';
import { omitNil } from '../../../../utils';

export default async ({
    shopList,
    fromPC,
    pageNo,
    pageSize,
    endPointLastCode, // 终点
    sendDoorEndPointLastCode, // 送货上门终点
    isCityDistribute, // 是否是同城配送
    isSendDoor, // 是否送货上门
    weight, // 重量
    size, // 方量
    proxyCharge, // 代收货款金额
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee, // 指定向收货人应该收多少钱
    orderBy, //排序方式, 0：以价格优先， 1：以时长优先
}) => {
    weight = settingMgr.getFloatWeight(weight, size); // 通过weight和size和阶梯价格计算重量
    const orderText = orderBy === 1 ? 'fee duration' : 'duration fee';
    const needBondAmount = settingMgr.getBondAmount(weight, size);
    const shopIdList = shopList.map(o => new mongoose.Types.ObjectId(o.id));
    for (const shop of shopList) {
        shop.regionProfitRate = await getRegionProfitRateByEndPoint(shop.id, endPointLastCode, isCityDistribute);
    }
    let count;

    if (fromPC && pageNo === 0) {
        const countResults = await RoadmapModel.aggregate()
        .match(omitNil({ shopId: { $in: shopIdList }, endPointLastCode, enable: true }))
        .project({
            sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
            shipperInBranchShopId: 1,
            shipperId: 1,
            price: 1,
            minFee: 1,
            profitRate: 1,
        })
        .project({
            shop: { $arrayElemAt: [ shopList, { $indexOfArray: [ shopList.map(o => o._id), '$shopId' ] } ] },
            sendDoorEnable: '$sendDoor.enable',
            sendDoorPrice: '$sendDoor.sendDoorPrice',
            sendDoorMinFee: '$sendDoor.sendDoorMinFee',
            shipperInBranchShopId: 1,
            shipperId: 1,
            price: 1,
            minFee: 1,
            profitRate: 1,
        })
        .match(isSendDoor ? { sendDoorEnable: true } : {})
        .lookup({ from: 'shipperinbranchshops', localField: 'shipperInBranchShopId', foreignField: '_id', as: 'shipperinbranchshop' })
        .project({
            profitRate: { $ifNull: [ '$profitRate', '$shop.regionProfitRate' ] },
            remainBondAmount: { $arrayElemAt: ['$shipperinbranchshop.remainBondAmount', 0] },
            fee: {
                $floor: { $sum: [
                    { $max: [ { $multiply: ['$price', weight] }, '$minFee' ] },
                    { $cond: { if: isSendDoor, then: { $max: [ { $multiply: ['$sendDoorPrice', weight] }, '$sendDoorMinFee' ] }, else: 0 } },
                ] },
            },
            shipperId: 1,
        })
        .match({ remainBondAmount: { $gte: needBondAmount } })
        .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
        .project({
            acountAmount: { $arrayElemAt: ['$shipper.acountAmount', 0] },
            profit: { $floor: { $multiply: [ '$fee', '$profitRate' ] } },
            fee: 1,
        })
        .project({
            hasEnoughAmount: {
                $cond: {
                    if: { $gte: [{
                        $subtract: [ '$acountAmount', {
                            $sum: [
                                proxyCharge,
                                {
                                    $cond: {
                                        if: isReachPay,
                                        then: {
                                            $cond: {
                                                if: totalDesignatedFee,
                                                // 如果有指定收款，需要扣除的钱为  totalDesignatedFee - fee
                                                then: { $subtract: [ totalDesignatedFee, '$fee' ] },
                                                // 否则，就是总部提成+分部提成，因为物流公司向收货人收取所有运费
                                                else: '$profit',
                                            },
                                        },
                                        else: 0,
                                    },
                                },
                            ],
                        } ],
                    }, 0] },
                    then: true,
                    else: false,
                },
            },
        })
        .match({ hasEnoughAmount: true })
        .group({
            _id: null,
            total: { $sum: 1 },
        });
        count = countResults.length ? countResults[0].total : 0;
    }

    const docs = await RoadmapModel.aggregate()
    .match(omitNil({ shopId: { $in: shopIdList }, endPointLastCode, enable: true }))
    .project({
        sendDoor: { $arrayElemAt: [{ $filter: { input: '$sendDoorList', as: 'item', cond: { $eq: [ '$$item.sendDoorEndPointLastCode', sendDoorEndPointLastCode ] } } }, 0] },
        shipperInBranchShopId: 1,
        shipperId: 1,
        price: 1,
        minFee: 1,
        profitRate: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .project({
        shop: { $arrayElemAt: [ shopList, { $indexOfArray: [ shopList.map(o => o._id), '$shopId' ] } ] },
        sendDoorEnable: '$sendDoor.enable',
        sendDoorPrice: '$sendDoor.sendDoorPrice',
        sendDoorMinFee: '$sendDoor.sendDoorMinFee',
        shipperInBranchShopId: 1,
        shipperId: 1,
        price: 1,
        minFee: 1,
        profitRate: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .match(isSendDoor ? { sendDoorEnable: true } : {})
    .lookup({ from: 'shipperinbranchshops', localField: 'shipperInBranchShopId', foreignField: '_id', as: 'shipperinbranchshop' })
    .project({
        remainBondAmount: { $arrayElemAt: ['$shipperinbranchshop.remainBondAmount', 0] },
        fee: {
            $floor: { $sum: [
                { $max: [ { $multiply: ['$price', weight] }, '$minFee' ] },
                { $cond: { if: isSendDoor, then: { $max: [ { $multiply: ['$sendDoorPrice', weight] }, '$sendDoorMinFee' ] }, else: 0 } },
            ] },
        },
        profitRate: { $ifNull: [ '$profitRate', '$shop.regionProfitRate' ] },
        shopName: '$shop.name',
        shopAddress: '$shop.address',
        distance: '$shop.distance',
        shipperId: 1,
        price: 1,
        minFee: 1,
        sendDoorPrice: 1,
        sendDoorMinFee: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .match({ remainBondAmount: { $gte: needBondAmount } })
    .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
    .project({
        shipperName: { $arrayElemAt: ['$shipper.name', 0] },
        acountAmount: { $arrayElemAt: ['$shipper.acountAmount', 0] },
        profit: { $floor: { $multiply: [ '$fee', '$profitRate' ] } },
        price: { $multiply: [ '$price', { $sum: [ 1, '$profitRate' ] } ] },
        minFee: { $multiply: [ '$minFee', { $sum: [ 1, '$profitRate' ] } ] },
        sendDoorPrice: { $multiply: [ '$sendDoorPrice', { $sum: [ 1, '$profitRate' ] } ] },
        sendDoorMinFee: { $multiply: [ '$sendDoorMinFee', { $sum: [ 1, '$profitRate' ] } ] },
        shopName: 1,
        shopAddress: 1,
        distance: 1,
        fee: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
    })
    .project({
        id: '$_id',
        _id: 0,
        hasEnoughAmount: {
            $cond: {
                if: { $gte: [{
                    $subtract: [ '$acountAmount', {
                        $sum: [
                            proxyCharge,
                            {
                                $cond: {
                                    if: isReachPay,
                                    then: {
                                        $cond: {
                                            if: totalDesignatedFee,
                                            // 如果有指定收款，需要扣除的钱为  totalDesignatedFee - fee
                                            then: { $subtract: [ totalDesignatedFee, '$fee' ] },
                                            // 否则就是总部提成+分部提成，因为物流公司向收货人收取所有运费
                                            else: '$profit',
                                        },
                                    },
                                    else: 0,
                                },
                            },
                        ],
                    } ],
                }, 0] },
                then: true,
                else: false,
            },
        },
        transportFee: { $sum: ['$fee', '$profit'] },
        shipperName: 1,
        shopName: 1,
        shopAddress: 1,
        distance: 1,
        price: 1,
        minFee: 1,
        sendDoorPrice: 1,
        sendDoorMinFee: 1,
        selfSignRate: 1,
        endPoint: 1,
        duration: 1,
        // delete below
        profit: 1,
        acountAmount: 1,
        fee: 1,
    })
    .match({ hasEnoughAmount: true })
    .sort(orderText)
    .skip(pageNo * pageSize)
    .limit(pageSize);

    return {
        count,
        roadmapList: docs,
    };
};

```

* PDShopServer/project/App/routers/posts/libs/weixinPay/api.js

```js
import _ from 'lodash';
import crypto from 'crypto';
import xml2js from 'xml2js';
import superagent from 'superagent';
import randomstring from 'randomstring';
import { _T } from '../../../../utils/';
const WEIXIN_SERVER = 'https://api.mch.weixin.qq.com/';
const CONFIG = {
    mch_id: '1489315622',
    mch_key: '0guiyang0simiantong0085188622803',
    appid: 'wxcbe9347be9959618',
    appsecret: 'a4ff7d6e12a170c09e0bdd8ce016f9b8',
    notify_url: 'http://120.78.65.221/api/weixinNotifyCallBack',
};

export const generateSignature = params => {
    const crypt = crypto.createHash('MD5');
    crypt.update(new Buffer(_.keys(params).sort().map(k => k + '=' + params[k]).join('&') + '&key=' + CONFIG.mch_key));
    params.sign = crypt.digest('hex').toUpperCase();
    return params;
};

const request = (url, params, cb, ssl) => {
    params.appid = CONFIG.appid;
    params.mch_id = CONFIG.mch_id;
    params.notify_url = CONFIG.notify_url;
    params.nonce_str = randomstring.generate();
    generateSignature(params);
    const xml = '<xml>' + _.values(_.mapValues(params, (v, k) => !v ? '' : _.isNumber(v) ? `<${k}>${v}</${k}>` : `<${k}><![CDATA[${v}]]></${k}>`)).join('') + '</xml>';
    const onRequest = (error, res) => {
        if (!error && res.statusCode === 200) {
            xml2js.parseString(res.text, { explicitArray: false, ignoreAttrs: true }, (error, json) => {
                if (error) {
                    cb(true, res.text);
                } else {
                    const result = json.xml;
                    if (result.return_code === 'SUCCESS') {
                        cb(false, result);
                    } else {
                        cb(true, result.return_msg);
                    }
                }
            });
        } else {
            cb(true, res.text);
        }
    };
    superagent.post(url).set({ 'Content-Type': 'text/xml' }).send(xml).end(onRequest);
};

export default {
    unifiedorder (params) {
        return new Promise(async resolve => {
            request(WEIXIN_SERVER + 'pay/unifiedorder', params, (error, data) => {
                if (error) {
                    return resolve({ msg: data });
                } else {
                    if (data.trade_type === 'NATIVE') {
                        return resolve({
                            success: true,
                            context: {
                                appid: CONFIG.appid,
                                partnerid: CONFIG.mch_id,
                                prepayid: data.prepay_id,
                                code_url: data.code_url,
                            },
                        });
                    } else {
                        return resolve({
                            success: true,
                            context: generateSignature({
                                appid: CONFIG.appid,
                                partnerid: CONFIG.mch_id,
                                prepayid: data.prepay_id,
                                package: 'Sign=WXPay',
                                noncestr: randomstring.generate(),
                                timestamp: _T(Math.floor((new Date().getTime() / 1000))),
                            }),
                        });
                    }
                }
            });
        });
    },
    orderquery (params, cb) {
        params = _.pick(params, 'transaction_id', 'out_trade_no');
        request(WEIXIN_SERVER + 'pay/orderquery', params, cb);
    },
    closeorder (params, cb) {
        params = _.pick(WEIXIN_SERVER + 'pay/closeorder', 'out_trade_no');
        request(WEIXIN_SERVER + 'secapi/pay/closeorder', params, cb);
    },
    refund (params, cb) {
        request(WEIXIN_SERVER + 'secapi/pay/refund', params, cb, true);
    },
    refundquery (params, cb) {
        request(WEIXIN_SERVER + 'pay/refundquery', params, cb);
    },
    downloadbill (params, cb) {
        request(WEIXIN_SERVER + 'pay/downloadbill', params, cb);
    },
    report (params, cb) {
        request(WEIXIN_SERVER + 'payitil/report', params, cb);
    },
    sendredpack (params, cb) {
        request(WEIXIN_SERVER + 'mmpaymkttransfers/sendredpack', params, cb, true);
    },
    sendgroupredpack (params, cb) {
        request(WEIXIN_SERVER + 'mmpaymkttransfers/sendgroupredpack', params, cb, true);
    },
    gethbinfo (params, cb) {
        request(WEIXIN_SERVER + 'mmpaymkttransfers/gethbinfo', params, cb, true);
    },
    transfers (params, cb) {
        request(WEIXIN_SERVER + 'mmpaymkttransfers/promotion/transfers', params, cb, true);
    },
    gettransferinfo (params, cb) {
        request(WEIXIN_SERVER + 'mmpaymkttransfers/gettransferinfo', params, cb, true);
    },
};

```

* PDShopServer/project/App/routers/posts/libs/weixinPay/notifyCallBack.js

```js
import { WeixinPayModel } from '../../../../models';
import { _T, omitNil } from '../../../../utils/';
import moment from 'moment';
import CONSTANTS from '../../../../constants';
import { rechargeForClient, rechargeForShopMember } from '../account';
import { generateSignature } from './api';
import config from '../../../../../config';

export default async ({ xml }) => {
    const { sign, ...obj } = xml;
    generateSignature(obj);
    if (obj.sign !== sign) {
        return '<xml><return_code><![CDATA[FAIL]]></return_code></xml>';
    }
    const {
        return_code,
        // return_msg,
        result_code,
        out_trade_no,
        appid,
        mch_id,
        // nonce_str,
        openid,
        trade_type,
        bank_type,
        total_fee,
        cash_fee,
        transaction_id,
        time_end,
    } = xml;
    if (return_code === 'SUCCESS' && result_code === 'SUCCESS') {
        const order = await WeixinPayModel.findOne({
            _id: out_trade_no,
            state: CONSTANTS.PS_READY_PAY,
            appid,
            mch_id,
            total_fee,
            trade_type,
        });
        if (order) {
            await order.update(omitNil({
                state: CONSTANTS.PS_READY_SAVE,
                openid,
                bank_type,
                cash_fee,
                transaction_id,
                payedTime: moment(time_end, 'YYYYMMDDHHmmss').valueOf(),
            }));
            const userId = order.clientId || order.shopMemberId;
            const recharge = order.clientId ? rechargeForClient : rechargeForShopMember;
            const result = await recharge({
                userId: _T(userId),
                amount: total_fee * (config.weixinPayRate || 1), // 单位：分
                // 第三方的货单号  0：支付宝(A)， 1：财付通(C)，2：工商银行(G)，3：建设银行(J)， 4：中国银行（Z）， 5：农业银行(N)，规则：缩写字母#第三方的货单号#第三方账号
                thirdpartyAccount: `C#${transaction_id}#${openid}`,
            });
            if (result.success) {
                await order.update({ state: CONSTANTS.PS_SUCCESS });
            }
        } else {
            return '<xml><return_code><![CDATA[FAIL]]></return_code></xml>';
        }
    }

    return '<xml><return_code><![CDATA[SUCCESS]]></return_code></xml>';
};

```

* PDShopServer/project/App/routers/posts/shipper/account/getBondAmountInfo.js

```js
import { ClientModel, ShipperInBranchShopModel } from '../../../../models';

export default async ({ userId, shopId }) => {
    const client = await ClientModel.findById(userId);
    if (!client || !client.shipperId) {
        return { msg: '没有该用户' };
    }

    let inShop = await ShipperInBranchShopModel.findOne({ shipperId: client.shipperId, shopId })
    .select({ totalBondAmount: 1, remainBondAmount: 1 });
    if (!inShop) {
        return { msg: '没有入住该分店' };
    }

    return { success: true, context: { totalBondAmount: inShop.totalBondAmount, remainBondAmount: inShop.remainBondAmount } };
};

```

* PDShopServer/project/App/routers/posts/shipper/cityDistribute/getCityTruckDetail.js

```js
import { CityTruckModel, ClientModel } from '../../../../models';
import _ from 'lodash';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    const client = await ClientModel.findById(userId);
    if (!client || !client.truckId) {
        return { msg: '你没有货车' };
    }
    const truck = await CityTruckModel.findById(client.truckId)
    .populate({
        path: 'unloadAllOrderList',
        select: {
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            endPoint: 1,
            stateList: 1,
            totalNumbers: 1,
        },
    });
    if (!truck) {
        return { msg: '没有该货车' };
    }

    const context = truck.toObject();
    _.forEach(context.unloadAllOrderList, item => {
        item.unloadNumber = (_.find(item.stateList, o => o.state === CONSTANTS.OS_READY_LOAD) || {}).count || 0;
        delete item.stateList;
    });

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/cityDistribute/getClientPickShipperList.js

```js
import mongoose from 'mongoose';
import { ShipperInBranchShopModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    shopId,
}) => {
    const docs = await ShipperInBranchShopModel.aggregate()
    .match({ shopId: new mongoose.Types.ObjectId(shopId) })
    .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
    .project({
        _id: 0,
        id: { $arrayElemAt: ['$shipper._id', 0] },
        name: { $arrayElemAt: ['$shipper.name', 0] },
        shipperType: { $arrayElemAt: ['$shipper.shipperType', 0] },
        clientPickEnable: 1,
        clientPickPrice: 1,
    })
    .match({
        shipperType: CONSTANTS.ST_CITY,
        clientPickEnable: true,
        clientPickPrice: { $exists: true },
    })
    .sort({
        clientPickPrice: -1,
    });
    return { success: true, context: { shipperList: docs } };
};

```

* PDShopServer/project/App/routers/posts/shipper/cityDistribute/getOrderListInCityTruck.js

```js
import { CityTruckModel, ClientModel } from '../../../../models';

export default async ({ userId }) => {
    const client = await ClientModel.findById(userId);
    if (!client || !client.truckId) {
        return { msg: '你没有货车' };
    }
    const truck = await CityTruckModel.findById(client.truckId)
    .populate({
        path: 'orderList',
        select: {
            photo: 1,
            shopId: 1,
            agentId: 1,
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            name: 1,
            receiverId: 1,
            roadmapId: 1,
            realFee: 1,
            createTime: 1,
            placeOrderTime: 1,
            weight: 1,
            size: 1,
            stateList: 1,
            totalNumbers: 1,
            startPoint: 1,
            endPoint: 1,
            proxyCharge: 1,
            needPayInsuanceFee: 1,
            payMode: 1,
            isSendDoor: 1,
            isReachPay: 1,
            totalDesignatedFee: 1,
            isInsuance: 1,
        },
        populate: [{
            path: 'shopId',
            select: { name: 1, address: 1 },
        }, {
            path: 'agentId',
            select: { name: 1, address: 1 },
        }],
    });

    return { success: true, context: {
        orderList: (truck || {}).orderList || [],
    } };
};

```

* PDShopServer/project/App/routers/posts/shipper/cityDistribute/getOrders.js

```js
import _ from 'lodash';
import { OrderModel, ClientModel } from '../../../../models';
import { getKeywordCriteriaForOrder, omitNil, hasAuthority } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import moment from 'moment';

async function getPageData (shopId, shipperId, type, params) {
    let { keyword, startDate, endDate, fromPC, pageNo, pageSize } = params;
    let criteria = {};
    if (type === 'tostore') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_STOCK };
    } else if (type === 'stored') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_LOAD };
    } else if (type === 'onway') {
        criteria = { 'stateList.state': CONSTANTS.OS_ON_THE_WAY };
    } else if (type === 'success') {
        criteria = { 'stateList.state': CONSTANTS.OS_RECEIVE_SUCCESS };
    }
    if (fromPC) {
        if (startDate && endDate) {
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        } else {
            startDate = endDate = moment().format('YYYY-MM-DD');
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        }
    }
    criteria = getKeywordCriteriaForOrder(keyword, omitNil({ ...criteria, shopId, shipperId }));

    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        shopId: 1,
        shipperId: 1,
        senderName: 1,
        senderPhone: 1,
        receiverName: 1,
        receiverPhone: 1,
        name: 1,
        endPoint: 1,
        createTime: 1,
        modifyTime: 1,
        isReachPay: 1,
        payMode: 1,
        receiveMemberId: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        stateList: 1,
        isInsuance: 1,
        insuanceMount: 1,
        insuanceFee: 1,
        fee: 1,
        profit: 1,
        masterProfit: 1,
        branchProfit: 1,
        realFee: 1,
        proxyCharge: 1,
        proxyChargeProfit: 1,
        needPayTransportFee: 1,
        needPayInsuanceFee: 1,
        totalDesignatedFee: 1,
        designatedFee: 1,
        placeOrderTime: 1,
    }).populate({
        path: 'receiveMemberId',
        select: { name: 1, phone: 1 },
    }).populate({
        path: 'shipperId',
        select: { name: 1 },
    }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });

    return {
        count,
        list: docs,
    };
};

// ['待入库', '已入库', '运输中', '成功']
const TYPES = ['tostore', 'stored', 'onway', 'success'];
export default async ({ userId, shopId, type, ...params }) => {
    const client = await ClientModel.findById(userId);
    if (!client || !client.shipperId || !hasAuthority(client, CONSTANTS.AH_LOOK_ORDERS)) {
        return { msg: '你没有查看货单的权限' };
    }
    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(shopId, client.shipperId, type, params),
            },
        };
    } else {
        return {
            success: true,
            context: {
                tostore: await getPageData(shopId, client.shipperId, 'tostore', params),
                stored: await getPageData(shopId, client.shipperId, 'stored', params),
                onway: await getPageData(shopId, client.shipperId, 'onway', params),
                success: await getPageData(shopId, client.shipperId, 'success', params),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/shipper/cityDistribute/placeStorage.js

```js
import _ from 'lodash';
import { ClientModel, OrderModel } from '../../../../models';
import { actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderId,
    count = 1,
}) => {
    const member = await ClientModel.findById(userId);
    if (!member || !member.shipperId) {
        return { msg: '无效操作' };
    }
    const order = await OrderModel.getOrderFromOriginOrder(member.shipperId, orderId, 1);
    if (!order) {
        return { msg: '没有该货单' };
    }

    const stateList = order.stateList;
    let item = _.find(stateList, (o) => o.state === CONSTANTS.OS_READY_STOCK);
    if (!item) {
        return { msg: '该货单没有待入库的货物' };
    }
    item.count -= count;
    if (item.count < 0) {
        return { msg: '数量无效' };
    }
    if (order.isClientPick) {
        item = _.find(stateList, (o) => o.state === CONSTANTS.OS_READY_PHONE_CONFIRM);
        if (item) {
            item.count += count;
        } else {
            stateList.unshift({ state: CONSTANTS.OS_READY_PHONE_CONFIRM, count });
        }
    } else {
        item = _.find(stateList, (o) => o.state === CONSTANTS.OS_READY_LOAD);
        if (item) {
            item.count += count;
        } else {
            stateList.unshift({ state: CONSTANTS.OS_READY_LOAD, count });
        }
    }
    order.stateList = _.filter(stateList, o => o.count > 0);
    order.shipperStoreScannerId = userId;
    order.placeStoreTime = Date.now();
    await order.save();

    actionLog(userId, 'Client', orderId, 'Order', 'placeStorage');

    return { success: true, order };
};

```

* PDShopServer/project/App/routers/posts/shipper/cityDistribute/scanLoadOrderForCityDistribute.js

```js
import { CityTruckModel, OrderModel, ClientModel } from '../../../../models';
import { _T, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    orderId,
    count = 1,
}) => {
    const member = await ClientModel.findById(userId);
    if (!member || !member.shipperId) {
        return { msg: '无效操作' };
    }
    if (!hasAuthority(member, CONSTANTS.AH_CITY_DISTRIBUTE)) {
        return { msg: '你没有扫描装车的权限' };
    }
    if (!member.truckId) {
        return { msg: '请现在个人中心设置货车' };
    }
    let truck = await CityTruckModel.findById(member.truckId);
    if (!truck) {
        return { msg: '没有该货车' };
    }
    const order = await OrderModel.getOrderFromOriginOrder(member.shipperId, orderId, 1);
    if (!order) {
        return { msg: '没有该货单' };
    }
    if (_T(order.shipperId) !== _T(member.shipperId)) {
        return { msg: '你无权扫描该货单' };
    }
    const stateList = order.stateList;
    let item = _.find(stateList, (o) => o.state === CONSTANTS.OS_READY_LOAD);
    if (!item) {
        return { msg: '该货单没有待装车的货物' };
    }
    item.count -= count;
    if (item.count < 0) {
        return { msg: '数量无效' };
    } else if (item.count === 0) {
        truck.unloadAllOrderList = _.reject(truck.unloadAllOrderList, o => _T(o) === _T(order.id));
    } else {
        truck.unloadAllOrderList = _.unionWith(truck.unloadAllOrderList, [order.id], (m, n) => _T(m) === _T(n));
    }
    item = _.find(stateList, (o) => o.state === CONSTANTS.OS_ON_THE_WAY);
    truck.totalNumbers += count;
    truck.totalWeight += order.weight * count / order.totalNumbers;
    truck.totalSize += order.size * count / order.totalNumbers;
    if (item) {
        item.count += count;
    } else {
        stateList.unshift({ state: CONSTANTS.OS_ON_THE_WAY, count });
    }
    order.stateList = _.filter(stateList, o => o.count > 0);
    order.loadScannerId = userId;
    order.loadScanTime = Date.now();
    order.truckId = truck.id;
    await order.save();

    if (!_.find(truck.orderList, o => _T(o) === _T(order.id))) {
        truck.orderList.unshift(order.id);
        truck.orderCount++;
    }

    await truck.save();

    truck = await CityTruckModel.findById(truck.id)
    .populate({
        path: 'unloadAllOrderList',
        select: {
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            endPoint: 1,
            stateList: 1,
            totalNumbers: 1,
        },
    });

    const context = truck.toObject();
    _.forEach(context.unloadAllOrderList, item => {
        item.unloadNumber = (_.find(item.stateList, o => o.state === CONSTANTS.OS_READY_LOAD) || {}).count || 0;
        delete item.stateList;
    });
    context.latestOrder = await OrderModel.findById(context.orderList[0])
    .select({
        senderPhone: 1,
        senderName: 1,
        receiverPhone: 1,
        receiverName: 1,
        createTime: 1,
        photo: 1,
        endPoint: 1,
        totalNumbers: 1,
        size: 1,
        weight: 1,
    });
    delete context.orderList;

    actionLog(userId, 'Client', orderId, 'Order', 'scanLoadOrder');
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/cityDistribute/setCityTruck.js

```js
import { CityTruckModel, ClientModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    plateNo, // 车牌号码
}) => {
    const client = await ClientModel.findById(userId);
    if (!hasAuthority(client, CONSTANTS.AH_CITY_DISTRIBUTE)) {
        return { msg: '你没有创建货车的权限' };
    }
    let truck;
    if (client.truckId) {
        truck = await CityTruckModel.findById(client.truckId);
        truck.plateNo = plateNo;
    } else {
        truck = new CityTruckModel({
            shipperId: client.shipperId,
            plateNo,
            driverId: userId,
        });
        client.truckId = truck.id;
        await client.save();
    }
    await truck.save();

    actionLog(userId, 'Client', truck.id, 'CityTruck', 'setCityTruck');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shipper/cityDistribute/setClientPickOrderTransportFee.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import { _T, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderId,
    isTransport, // 是否运送，如果打电话确认后需要运送，则为true，否则为false，等待用户上门自提
    endPoint, // 送货地
    transportFee, // 运费
    isScan, //是否是扫描，如果是扫描，则orderId传的是原始货单的id，否则就是传的当前货单的id
}) => {
    const member = await ClientModel.findById(userId);
    if (!member || !member.shipperId) {
        return { msg: '无效操作' };
    }
    let order;
    if (isScan) {
        order = await OrderModel.getOrderFromOriginOrder(member.shipperId, orderId, 1);
    } else {
        order = await OrderModel.findById(orderId);
    }
    if (!order) {
        return { msg: '没有该货单' };
    }
    if (_T(member.shipperId) !== _T(order.shipperId)) {
        return { msg: '你无权操作' };
    }
    order.confirmHandOverId = userId;
    if (!isTransport) {
        order.stateList = [ { state: CONSTANTS.OS_WAIT_CLIENT_PICK, count: order.totalNumbers } ];
    } else {
        order.stateList = [ { state: CONSTANTS.OS_READY_LOAD, count: order.totalNumbers } ];
        order.additionalFee = transportFee;
        order.endPoint = endPoint;
    }
    await order.save();

    actionLog(userId, 'Client', orderId, 'Order', 'setClientPickOrderTransportFee');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shipper/loadCargo/continueScanOrder.js

```js
import { TruckModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_SCAN_LOAD_TRUCK, true)) {
        return { msg: '你没有扫描装车的权限' };
    }
    const truck = await TruckModel.findOne({
        scannerId: userId,
        state: CONSTANTS.TS_READY_PAY_FOR_CARRY_PARTMENT,
    });
    if (!truck) {
        return { msg: '你没有继续扫描的货车' };
    }
    truck.state = CONSTANTS.TS_SCAN_LOADING;
    await truck.save();

    actionLog(userId, 'Client', truck.id, 'Truck', 'continueScanOrder');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shipper/loadCargo/finishScanOrder.js

```js
import { TruckModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_SCAN_LOAD_TRUCK, true)) {
        return { msg: '你没有扫描装车的权限' };
    }
    const truck = await TruckModel.findOne({
        scannerId: userId,
        state: CONSTANTS.TS_SCAN_LOADING,
    })
    .populate({
        path: 'carryPartmentId',
        select: { name: 1 },
    });
    if (!truck) {
        return { msg: '你没有正在扫描的货车' };
    }
    if (truck.unloadAllOrderList.length) {
        return { msg: '你还有一些货单没有全部装车' };
    }
    truck.state = CONSTANTS.TS_READY_PAY_FOR_CARRY_PARTMENT;
    await truck.save();
    actionLog(userId, 'Client', truck.id, 'Truck', 'finishScanOrder');

    return { success: true, context: truck };
};

```

* PDShopServer/project/App/routers/posts/shipper/loadCargo/getCarryPartmentList.js

```js
import { TruckModel, ShopPartmentModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({ userId, truckId }) => {
    const truck = await TruckModel.findById(truckId);
    const partmenList = await ShopPartmentModel.find({
        shopId: truck.shopId,
        type: CONSTANTS.SP_CARRY_PARTMENT,
        enable: true,
        workTruckId: { $exists: false },
        price: { $exists: true },
    }).sort({ price: 'asc' });

    return { success: true, context: {
        partmenList,
    } };
};

```

* PDShopServer/project/App/routers/posts/shipper/loadCargo/payForCarryPartment.js

```js
import { TruckModel, ShopPartmentModel, ShipperModel } from '../../../../models';
import { sequelize, AccountModel, BillModel, TruckPaymentModel } from '../../../../mysql';
import { _T, _F, _Y, hasAuthority, actionLog, authenticatePaymentPassword } from '../../../../utils';
import CONSTANTS from '../../../../constants';

const ERROR_HAVE_PAYED = '10000';
const ERROR_SHIPPER_MONEY_NOT_ENOUGH = '10002';
const ERROR_OTHER = '10004';
function payment (shipperId, truckId, shipperChairManId, carryPartmentId, amount) {
    return new Promise(async resolve => {
        sequelize.transaction(async (transaction) => {
            const truckPayment = await TruckPaymentModel.findOne({ where: { truckId, isPayedForCarryPartment: true }, transaction });
            if (truckPayment) {
                throw ERROR_HAVE_PAYED;
            }
            const shipperAccount = await AccountModel.findOne({ where: { targetId: shipperChairManId, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } }, transaction });
            if (!shipperAccount) {
                throw ERROR_SHIPPER_MONEY_NOT_ENOUGH;
            }
            const acountAmount = _Y(shipperAccount.amount - amount);
            await shipperAccount.decrement({ amount }, { transaction });
            const partmentAccount = await AccountModel.findOne({ where: { targetId: carryPartmentId, type: CONSTANTS.AT_BRANCH_SHOP_PARTMENT }, transaction });
            await partmentAccount.increment({ amount }, { transaction });
            await BillModel.create({ account: shipperAccount.id, truckId, tradeAmount: -amount, remark: '付搬运款' }, { transaction });
            await BillModel.create({ account: partmentAccount.id, truckId, tradeAmount: amount, remark: '收到搬运款' }, { transaction });

            await TruckPaymentModel.create({ truckId, isPayedForCarryPartment: true, payedForCarryPartmentTime: new Date() }, { transaction });
            // 扣款成功后，需要更新物流公司的账目
            const doc = await ShipperModel.findByIdAndUpdate(shipperId, { acountAmount }).catch(e => {
                throw e;
            });
            if (!doc) {
                throw ERROR_OTHER;
            }
        }).then((result) => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}

export default async ({ userId, truckId, payPassword }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_SCAN_LOAD_TRUCK, true)) {
        return { msg: '你没有扫描装车的权限' };
    }
    const criteria = truckId ? { _id: truckId } : { scannerId: userId };
    const truck = await TruckModel.findOne({
        ...criteria,
        state: CONSTANTS.TS_READY_PAY_FOR_CARRY_PARTMENT,
    }).populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    });
    if (!truck) {
        return { msg: '你没有完成扫描的货车' };
    }
    const error = await authenticatePaymentPassword(userId, payPassword);
    if (error) {
        return { msg: '密码错误' };
    }
    const amount = _F(truck.totalWeight * truck.carryPrice);
    const err = await payment(truck.shipper.id, _T(truck.id), _T(truck.shipper.chairManId), _T(truck.carryPartmentId), amount);
    if (err === ERROR_HAVE_PAYED) {
        truck.state = CONSTANTS.TS_READY_EXIT_WAREHOUSE;
        await truck.save();
        return { msg: '该货单无需再支付' };
    } else if (err === ERROR_SHIPPER_MONEY_NOT_ENOUGH) {
        return { msg: '余额不足，请先充值' };
    } else if (err) {
        return { msg: '系统出错，请稍后再试' };
    }
    truck.state = CONSTANTS.TS_READY_EXIT_WAREHOUSE;
    await truck.save();
    await ShopPartmentModel.findByIdAndUpdate(truck.carryPartmentId, { $unset: { workTruckId: 1 } });

    actionLog(userId, 'Client', truck.id, 'Truck', 'payForCarryPartment');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shipper/loadCargo/scanLoadOrder.js

```js
import { TruckModel, OrderModel, ClientModel } from '../../../../models';
import { _T, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    orderId,
    count = 1,
}) => {
    const member = await ClientModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_SCAN_LOAD_TRUCK)) {
        return { msg: '你没有扫描装车的权限' };
    }
    let truck = await TruckModel.findOne({
        scannerId: userId,
        state: CONSTANTS.TS_SCAN_LOADING,
    });
    if (!truck) {
        return { msg: '你没有正在扫描的货车' };
    }

    const order = await OrderModel.getOrderFromOriginOrder(truck.shopId, orderId);
    if (!order) {
        return { msg: '没有找到该货单' };
    }
    if (_T(order.shipperId) !== _T(member.shipperId)) {
        return { msg: '你无权扫描该货单' };
    }
    const stateList = order.stateList;
    let item = _.find(stateList, (o) => o.state === CONSTANTS.OS_READY_LOAD);
    if (!item) {
        return { msg: '该货单没有待装车的货物' };
    }
    item.count -= count;
    if (item.count < 0) {
        return { msg: '数量无效' };
    } else if (item.count === 0) {
        truck.unloadAllOrderList = _.reject(truck.unloadAllOrderList, o => _T(o) === _T(order.id));
    } else {
        truck.unloadAllOrderList = _.unionWith(truck.unloadAllOrderList, [order.id], (m, n) => _T(m) === _T(n));
    }
    item = _.find(stateList, (o) => o.state === CONSTANTS.OS_READY_START_OFF);
    truck.totalNumbers += count;
    truck.totalWeight += order.weight * count / order.totalNumbers;
    truck.totalSize += order.size * count / order.totalNumbers;
    if (item) {
        item.count += count;
    } else {
        stateList.unshift({ state: CONSTANTS.OS_READY_START_OFF, count });
    }
    order.stateList = _.filter(stateList, o => o.count > 0);
    order.loadScannerId = userId;
    order.loadScanTime = Date.now();
    order.truckId = truck.id;
    await order.save();

    if (!_.find(truck.orderList, o => _T(o) === _T(order.id))) {
        truck.orderList.unshift(order.id);
        truck.orderCount++;
    }

    await truck.save();

    truck = await TruckModel.findById(truck.id)
    .populate({
        path: 'unloadAllOrderList',
        select: {
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            endPoint: 1,
            stateList: 1,
            totalNumbers: 1,
        },
    });

    const context = truck.toObject();
    _.forEach(context.unloadAllOrderList, item => {
        item.unloadNumber = (_.find(item.stateList, o => o.state === CONSTANTS.OS_READY_LOAD) || {}).count || 0;
        delete item.stateList;
    });
    context.latestOrder = await OrderModel.findById(context.orderList[0])
    .select({
        senderPhone: 1,
        senderName: 1,
        receiverPhone: 1,
        receiverName: 1,
        createTime: 1,
        photo: 1,
        endPoint: 1,
        totalNumbers: 1,
        size: 1,
        weight: 1,
    });
    delete context.orderList;

    actionLog(userId, 'Client', orderId, 'Order', 'scanOrder');
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/loadCargo/selectCarryPartment.js

```js
import { TruckModel, ShopPartmentModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, truckId, carryPartmentId }) => {
    if (!await hasAuthority(userId, CONSTANTS.SELECT_CARRY_PARTMENT, true)) {
        return { msg: '你没有选择搬运队的权限' };
    }
    const truck = await TruckModel.findById(truckId);
    if (!truck) {
        return { msg: '没有该货车' };
    }
    if (truck.state !== CONSTANTS.TS_READY_SEARCH_CARRY_PARTMENT) {
        return { msg: '该货车不处于待找搬运队状态' };
    }
    const carryPartment = await ShopPartmentModel.findById(carryPartmentId);
    if (!carryPartment) {
        return { msg: '没有该搬运队' };
    }
    if (!carryPartment.enable) {
        return { msg: '该搬运队已经停运，请选择其他搬运队' };
    }
    if (carryPartment.workTruckId) {
        return { msg: '该搬运队已经在搬运中，请选择其他搬运队' };
    }
    carryPartment.workTruckId = truckId;
    await carryPartment.save();
    await truck.update({ carryPartmentId, carryPrice: carryPartment.price, state: CONSTANTS.TS_READY_SCAN_LOAD });

    actionLog(userId, 'Client', truckId, 'Truck', 'selectCarryPartment');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shipper/loadCargo/startScanOrder.js

```js
import { TruckModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, truckId }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_SCAN_LOAD_TRUCK, true)) {
        return { msg: '你没有扫描装车的权限' };
    }
    const truck = await TruckModel.findById(truckId);
    if (!truck) {
        return { msg: '没有该货车' };
    }
    if (truck.state !== CONSTANTS.TS_READY_SCAN_LOAD) {
        return { msg: '该货车处于待装车状态' };
    }
    await truck.update({ scannerId: userId, state: CONSTANTS.TS_SCAN_LOADING });

    actionLog(userId, 'Client', truckId, 'Truck', 'startScanOrder');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shipper/member/getMemberByPhone.js

```js
import { ClientModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _T } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    memberPhone,
}) => {
    const member = await ClientModel.findById(userId);
    const client = await ClientModel.findOne({ phone: memberPhone });
    if (!client || !client.registerTime) {
        return { msg: '没有该用户' };
    }
    if (client.agentId) {
        return { msg: '该用户已经是收货点的成员' };
    }
    if (client.shipperId) {
        if (_T(member.shipperId) === _T(client.shipperId)) {
            return { msg: '该用户已经是物流公司成员，无需再添加' };
        } else if (_T(client.shipperId) !== _T(member.shipperId)) {
            return { msg: '该用户已经是其他物流公司的成员' };
        }
    }

    const clientAccount = await AccountModel.findOne({ where: { targetId: client.id, type: CONSTANTS.AT_CLIENT } });
    if (clientAccount && clientAccount.amount > 0) {
        return { msg: '请通知该用户先将其账户中的余额提现后再加入物流公司' };
    }

    return { success: true, context: client };
};

```

* PDShopServer/project/App/routers/posts/shipper/member/getMemberDetail.js

```js
import { ClientModel } from '../../../../models';

export default async ({
    userId,
    memberId,
}) => {
    const member = await ClientModel.findById(memberId);
    return { success: true, context: member };
};

```

* PDShopServer/project/App/routers/posts/shipper/member/getMemberList.js

```js
import { ClientModel } from '../../../../models';

export default async ({
    userId,
}) => {
    const member = await ClientModel.findById(userId);
    if (!member || !member.shipperId) {
        return { msg: '你不是物流公司成员' };
    }
    const docs = await ClientModel.find({ shipperId: member.shipperId }).sort({ registerTime: 'asc' });

    return { success: true, context: { memberList: docs } };
};

```

* PDShopServer/project/App/routers/posts/shipper/member/modifyMemberAuthority.js

```js
import { ClientModel, CityTruckModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _T, omitNil, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    memberId,
    authority,
}) => {
    if (authority) {
        authority = _.sortBy(_.uniq(authority));
    }
    const member = await ClientModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_SHIPPER_MEMBER_AUTHORITY)) {
        return { msg: '你没有修改成员的权限' };
    }
    const client = await ClientModel.findById(memberId);
    if (!client || !client.registerTime) {
        return { msg: '没有该用户' };
    }
    if (client.agentId) {
        return { msg: '该用户是收货点的成员，你不能修改他的权限' };
    }
    if (client.shipperId && _T(client.shipperId) !== _T(member.shipperId)) {
        return { msg: '该用户是其他物流公司的成员，你不能修改他的权限' };
    }
    const clientAccount = await AccountModel.findOne({ where: { targetId: memberId, type: CONSTANTS.AT_CLIENT } });
    if (clientAccount && clientAccount.amount > 0) {
        return { msg: '请通知该用户先将其账户中的余额提现后再加入物流公司' };
    }

    // 删除权限
    if (!authority || !authority.length) {
        // 如果该成员拥有同城配送的权限，则需要删除将该成员对应的货车设置为 disable
        if (_.includes(client.authority, CONSTANTS.AH_CITY_DISTRIBUTE)) {
            await CityTruckModel.findByIdAndUpdate(client.truckId, { enable: false });
        }
        await client.update({ $unset: { shipperId: 1 }, authority: [] });
    } else {
        // 如果为该成员添加同城配送的权限，需要添加相应的车辆
        let truckId;
        if (_.includes(authority, CONSTANTS.AH_CITY_DISTRIBUTE) && !_.includes(client.authority, CONSTANTS.AH_CITY_DISTRIBUTE)) {
            if (client.truckId) {
                await CityTruckModel.findByIdAndUpdate(client.truckId, { enable: true });
            } else {
                const truck = new CityTruckModel({
                    shipperId: member.shipperId,
                    driverId: memberId,
                });
                await truck.save();
                truckId = truck.id;
            }
        }
        await client.update(omitNil({ authority, shipperId: member.shipperId, truckId }));
    }

    actionLog(userId, 'Client', memberId, 'Client', 'modifyMemberAuthority');
    const context = client.toObject();
    context.authority = authority || [];
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/roadmap/getRoadmapDetail.js

```js
import { RoadmapModel } from '../../../../models';

export default async ({ userId, roadmapId }) => {
    const context = await RoadmapModel.findById(roadmapId)
    .populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });
    if (!context) {
        return { msg: '没有该线路' };
    }

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/roadmap/getRoadmapList.js

```js
import { RoadmapModel } from '../../../../models';
import { getKeywordCriteriaForRoadmap, getOwnShipper } from '../../../../utils';
import _ from 'lodash';

export default async ({ userId, shopId, isCityDistribute = false, keyword, fromPC, pageNo, pageSize }) => {
    const shipperId = await getOwnShipper(userId);
    const criteria = getKeywordCriteriaForRoadmap(keyword, { shipperId, shopId, isCityDistribute });
    const count = (fromPC && pageNo === 0) ? await RoadmapModel.count(criteria) : undefined;
    const query = RoadmapModel.find(criteria).sort({ modifyTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        endPoint: 1,
        endPointLastCode: 1,
        price: 1,
        basePrice: 1,
        updatePriceTime: 1,
        minFee: 1,
        baseMinFee: 1,
        updateMinFeeTime: 1,
        duration: 1,
        selfSignRate: 1,
        shopId: 1,
        enable: 1,
        sendDoorEnable: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });

    return { success: true, context: {
        count,
        roadmapList: docs.map(o => _.omit(o.toObject(), 'shop')),
    } };
};

```

* PDShopServer/project/App/routers/posts/shipper/roadmap/modifyRoadmap.js

```js
import { RoadmapModel } from '../../../../models';
import { omitNil, hasAuthority, actionLog } from '../../../../utils/';
import { registeredRoadmapAddress } from '../../libs/address';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    roadmapId,
    endPoint,
    endPointLastCode,
    price, // 单价
    basePrice, // 价格保底价
    minFee, // 起步价
    baseMinFee, // 起步价保底价
}) => {
    const roadmap = await RoadmapModel.findById(roadmapId);
    if (!roadmap) {
        return { msg: '没有该路线' };
    }
    if (!await hasAuthority(userId, !roadmap.isCityDistribute ? CONSTANTS.AH_MODIFY_ROADMAP : CONSTANTS.AH_MODIFY_SEND_DOOR_ROADMAP, true)) {
        return { msg: '你没有修改路线的权限' };
    }

    if (endPointLastCode) {
        if (await RoadmapModel.findOne({ _id: { $ne: roadmapId }, endPointLastCode, shopId: roadmap.shopId, shipperId: roadmap.shipperId })) {
            return { msg: '你已经在该店创建过该方向的路线' };
        }
    }
    let updatePriceTime, updateMinFeeTime;
    const now = Date.now();
    if (!_.isNil(price) && price !== roadmap.price) {
        updatePriceTime = now;
    }
    if (!_.isNil(minFee) && minFee !== roadmap.minFee) {
        updateMinFeeTime = now;
    }

    await roadmap.update(omitNil({
        endPoint,
        endPointLastCode,
        price,
        basePrice,
        updatePriceTime,
        minFee,
        baseMinFee,
        updateMinFeeTime,
        modifyTime: now,
    }));
    (!roadmap.isCityDistribute && roadmap.enable && (endPointLastCode && endPointLastCode !== roadmap.endPointLastCode)) &&
    await registeredRoadmapAddress(roadmap.shopId, endPointLastCode, roadmap.endPointLastCode); // 只有roadmap为enable的时候注册地址

    const context = await RoadmapModel.findById(roadmapId)
    .select({
        endPoint: 1,
        endPointLastCode: 1,
        price: 1,
        basePrice: 1,
        updatePriceTime: 1,
        minFee: 1,
        baseMinFee: 1,
        updateMinFeeTime: 1,
        duration: 1,
        selfSignRate: 1,
        shopId: 1,
        enable: 1,
        sendDoorEnable: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });

    actionLog(userId, 'Client', roadmapId, 'Roadmap', 'modifyRoadmap');
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/roadmap/publishRoadmap.js

```js
import { RoadmapModel, ShipperInBranchShopModel, ClientModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils/';
import { getSendDoorLastCodeList } from '../../libs/address';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    shopId,
    endPoint, // 终点，精确到镇级别
    endPointLastCode, // 终点县级代码
    price, // 单价
    basePrice, // 价格保底价
    minFee, // 起步价
    baseMinFee, // 起步价保底价
    isCityDistribute, //是否是同城配送的路线
}) => {
    const client = await ClientModel.findById(userId);
    if (!hasAuthority(client, !isCityDistribute ? CONSTANTS.AH_CREATE_ROADMAP : CONSTANTS.AH_CREATE_SEND_DOOR_ROADMAP)) {
        return { msg: '你没有创建路线的权限' };
    }
    const inShop = await ShipperInBranchShopModel.findOne({ shipperId: client.shipperId, shopId });
    if (!inShop) {
        return { msg: '你没有入驻该分店' };
    }
    const roadmap = await RoadmapModel.findOne({ shipperId: client.shipperId, shopId, endPointLastCode });
    if (roadmap) {
        return { msg: '你已经在该店创建过该方向的路线' };
    }

    // 如果是长途运送，需要为该路线添加所有的同城配送的列表，但是不设置价格，enable设置为false
    const data = {
        shipperId: client.shipperId,
        shopId,
        shipperInBranchShopId: inShop.id,
        endPoint,
        endPointLastCode,
        price,
        basePrice,
        minFee,
        baseMinFee,
        isCityDistribute,
        enable: isCityDistribute, //同城配送的路线默认为开启状态
    };
    if (!isCityDistribute) {
        const now = Date.now();
        const endPointLastCodeList = await getSendDoorLastCodeList(endPointLastCode);
        data.sendDoorList = endPointLastCodeList.map(o => ({
            sendDoorEndPoint: o.address,
            sendDoorEndPointLastCode: o.code,
            enable: false,
            sendDoorPrice: o.sendDoorPrice,
            sendDoorMinFee: o.sendDoorMinFee,
            baseSendDoorPrice: 0,
            baseSendDoorMinFee: 0,
            updateSendDoorPriceTime: now,
            updateSendDoorMinFeeTime: now,
        }));
    }

    const doc = new RoadmapModel(data);
    await doc.save();

    const context = await RoadmapModel.findById(doc.id).select({
        endPoint: 1,
        endPointLastCode: 1,
        price: 1,
        basePrice: 1,
        updatePriceTime: 1,
        minFee: 1,
        baseMinFee: 1,
        updateMinFeeTime: 1,
        duration: 1,
        selfSignRate: 1,
        shopId: 1,
        enable: 1,
        sendDoorEnable: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });

    actionLog(userId, 'Client', doc.id, 'Roadmap', 'publishRoadmap');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/roadmap/removeRoadmap.js

```js
import { RoadmapModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    roadmapId,
}) => {
    const roadmap = await RoadmapModel.findById(roadmapId);
    if (!roadmap) {
        return { msg: '没有该路线' };
    }
    if (!await hasAuthority(userId, !roadmap.isCityDistribute ? CONSTANTS.AH_REMOVE_ROADMAP : CONSTANTS.AH_REMOVE_SEND_DOOR_ROADMAP, true)) {
        return { msg: '你没有删除路线的权限' };
    }
    await roadmap.remove();

    actionLog(userId, 'Client', roadmapId, 'Roadmap', 'removeRoadmap');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shipper/roadmap/setRoadmapEnable.js

```js
import { RoadmapModel, ShipperModel, ClientModel } from '../../../../models';
import { hasAuthority, actionLogWithList } from '../../../../utils';
import { registeredRoadmapAddress } from '../../libs/address';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    roadmapIdList,
    enable,
}) => {
    const user = await ClientModel.findById(userId);
    if (!user.shipperId) {
        return { msg: '你不是物流公司' };
    }
    const shipper = await ShipperModel.findById(user.shipperId);
    if (shipper.shipperType === 1) {
        if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_SEND_DOOR_ROADMAP, true)) {
            return { msg: '你没有修改同城配送路线的权限' };
        }
    } else {
        if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP, true)) {
            return { msg: '你没有修改路线的权限' };
        }
    }

    const result = await RoadmapModel.update({ _id: { $in: roadmapIdList }, sendDoorEnable: true }, { enable }, { multi: true });
    if (result.n) {
        for (const roadmapId of roadmapIdList) {
            const roadmap = await RoadmapModel.findById(roadmapId);
            if (roadmap.enable) {
                await registeredRoadmapAddress(roadmap.shopId, roadmap.endPointLastCode);
            } else {
                await registeredRoadmapAddress(roadmap.shopId, undefined, roadmap.endPointLastCode);
            }
        }
    }

    if (roadmapIdList.length === 1) {
        if (!result.n) {
            return { msg: '必须保证每条送货上门的路线正常运营' };
        }
    } else {
        if (!result.n) {
            return { msg: '修改失败，必须保证每条路线的送货上门的路线正常运营' };
        } else if (roadmapIdList.length !== result.n) {
            return { msg: '修改失败，部分路线的送货上门的路线为非正常运营状态' };
        }
    }

    actionLogWithList(userId, 'Client', roadmapIdList.map(o => ({ id: o, model: 'Roadmap' })), 'setRoadmapEnable');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shipper/roadmap/setRoadmapPrice.js

```js
import { RoadmapModel, ClientModel, ShipperModel } from '../../../../models';
import { _T, hasAuthority, actionLogWithList } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    roadmapIdList,
    mode, // 0: 按照价格调整  1: 按照百分比调整  2: 按照名次调整
    typeList, // 数组，0：长途运费单价  1：长途运费起价
    value, // 如果mode为0和1时，正数为增加，负数为减少
}) => {
    if (!value || !typeList || !typeList.length) {
        return { msg: '参数错误' };
    }
    const user = await ClientModel.findById(userId);
    if (!user.shipperId) {
        return { msg: '你不是物流公司' };
    }
    const shipper = await ShipperModel.findById(user.shipperId);
    if (shipper.shipperType === 1) {
        if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_SEND_DOOR_ROADMAP, true)) {
            return { msg: '你没有修改同城配送路线的权限' };
        }
    } else {
        if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP, true)) {
            return { msg: '你没有修改路线的权限' };
        }
    }
    const keys = [ 'price', 'minFee' ];
    const baseKeys = [ 'basePrice', 'baseMinFee' ];
    const timeKeys = [ 'updatePriceTime', 'updateMinFeeTime' ];
    const now = Date.now();

    if (mode === 0) {
        const incKeys = Object.assign(...typeList.map(o => ({ [keys[o]]: value })));
        const setKeys = Object.assign(...typeList.map(o => ({ [timeKeys[o]]: now })));
        await RoadmapModel.update({ _id: { $in: roadmapIdList } }, { $inc: incKeys, ...setKeys }, { multi: true });
    } else if (mode === 1) {
        const incKeys = Object.assign(...typeList.map(o => ({ [keys[o]]: value / 100 + 1, [timeKeys[o]]: now })));
        const setKeys = Object.assign(...typeList.map(o => ({ [timeKeys[o]]: now })));
        await RoadmapModel.update({ _id: { $in: roadmapIdList } }, { $mul: incKeys, ...setKeys }, { multi: true });
    } else {
        let successCount = 0;
        for (const i in roadmapIdList) {
            const roadmap = await RoadmapModel.findById(roadmapIdList[i]);
            if (!roadmap) {
                continue;
            }
            const updateOptions = {};
            for (const j in typeList) {
                const key = keys[typeList[j]];
                const baseKey = baseKeys[typeList[j]];
                const timeKey = timeKeys[typeList[j]];
                const relateRoadmapList = await RoadmapModel.find({ shopId: roadmap.shopId, endPointLastCode: roadmap.endPointLastCode })
                .sort({ [key]: 'asc', [timeKey]: 'asc' }).limit(value);
                const relateRoadmap = relateRoadmapList[value - 1];
                const newPrice = relateRoadmap[key] - 0.01; // 注意： 这里每次加分减少1分钱
                if (relateRoadmap && _T(relateRoadmap.id) !== roadmapIdList[i] && !roadmap[baseKey] && newPrice > roadmap[baseKey]) {
                    updateOptions[key] = newPrice;
                    updateOptions[timeKey] = now;
                }
            }
            if (_.keys(updateOptions).length) {
                await roadmap.update(updateOptions);
                successCount++;
            }
        }
        if (successCount === 0) {
            return { msg: '设置失败，可能是没有设置底价、或者价格已经超过底价、或者已经是第' + value + '名' };
        }
        if (successCount !== roadmapIdList.length) {
            return { msg: '部分设置失败，可能是没有设置底价、或者价格已经超过底价、或者已经是第' + value + '名' };
        }
    }

    actionLogWithList(userId, 'Client', roadmapIdList.map(o => ({ id: o, model: 'Roadmap' })), 'setRoadmapPrice');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shipper/roadmap/setRoadmapSendDoorBasePrice.js

```js
import { RoadmapModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    roadmapId,
    lastCodeList,
    mode, //  0: 设置底价, 1: 按照价格调整  2: 按照百分比调整
    typeList, // 数组，0：同城配送运费单价的底价  1：同城配送运费起价的底价
    value, // 如果mode为0和1时，正数为增加，负数为减少
}) => {
    if (!value || !typeList || !typeList.length) {
        return { msg: '参数错误' };
    }
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP, true)) {
        return { msg: '你没有修改路线的权限' };
    }
    const roadmap = await RoadmapModel.findById(roadmapId);
    if (!roadmap) {
        return { msg: '没有该路线' };
    }
    const sendDoorList = roadmap.sendDoorList;
    const keys = [ 'baseSendDoorPrice', 'baseSendDoorMinFee' ];

    if (mode === 0) {
        _.forEach(lastCodeList, o => {
            const item = _.find(sendDoorList, m => m.sendDoorEndPointLastCode === o);
            item && _.forEach(typeList, k => { item[keys[k]] = value; });
        });
        await roadmap.save();
    } else if (mode === 1) {
        _.forEach(lastCodeList, o => {
            const item = _.find(sendDoorList, m => m.sendDoorEndPointLastCode === o);
            item && _.forEach(typeList, k => { item[keys[k]] += value; });
        });
        await roadmap.save();
    } else if (mode === 2) {
        _.forEach(lastCodeList, o => {
            const item = _.find(sendDoorList, m => m.sendDoorEndPointLastCode === o);
            item && _.forEach(typeList, k => { item[keys[k]] *= (value / 100 + 1); });
        });
        await roadmap.save();
    }

    actionLog(userId, 'Client', roadmapId, 'Roadmap', 'setRoadmapSendDoorBasePrice');

    return { success: true, context: { sendDoorList: _.filter(roadmap.toObject().sendDoorList, o => _.includes(lastCodeList, o.sendDoorEndPointLastCode)) } };
};

```

* PDShopServer/project/App/routers/posts/shipper/roadmap/setRoadmapSendDoorEnable.js

```js
import { RoadmapModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    roadmapId,
    lastCodeList,
    enable,
}) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP, true)) {
        return { msg: '你没有修改路线的权限' };
    }
    const roadmap = await RoadmapModel.findById(roadmapId);
    if (!roadmap) {
        return { msg: '没有该路线' };
    }
    _.forEach(lastCodeList, o => { const item = _.find(roadmap.sendDoorList, m => m.sendDoorEndPointLastCode === o); item && (item.enable = enable); });
    roadmap.sendDoorEnable = _.every(roadmap.sendDoorList, o => o.enable);
    if (roadmap.enable && !roadmap.sendDoorEnable) {
        roadmap.enable = false;
    }
    await roadmap.save();

    actionLog(userId, 'Client', roadmapId, 'Roadmap', 'setRoadmapSendDoorEnable');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shipper/roadmap/setRoadmapSendDoorPrice.js

```js
import { RoadmapModel } from '../../../../models';
import { _T, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    roadmapId,
    lastCodeList,
    mode, // 0: 按照价格调整  1: 按照百分比调整  2: 按照名次调整
    typeList, // 数组，0：送货上门单价  1：送货上门起价
    value, // 如果mode为0和1时，正数为增加，负数为减少
}) => {
    if (!value || !typeList || !typeList.length) {
        return { msg: '参数错误' };
    }
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP, true)) {
        return { msg: '你没有修改路线的权限' };
    }
    const roadmap = await RoadmapModel.findById(roadmapId);
    if (!roadmap) {
        return { msg: '没有该路线' };
    }
    const sendDoorList = roadmap.sendDoorList;
    const keys = [ 'sendDoorPrice', 'sendDoorMinFee' ];
    const baseKeys = [ 'baseSendDoorPrice', 'baseSendDoorMinFee' ];
    const timeKeys = [ 'updateSendDoorPriceTime', 'updateSendDoorMinFeeTime' ];
    const now = Date.now();

    if (mode === 0) {
        _.forEach(lastCodeList, o => {
            const item = _.find(sendDoorList, m => m.sendDoorEndPointLastCode === o);
            item && _.forEach(typeList, k => { item[keys[k]] += value; item[timeKeys[k]] = now; });
        });
        await roadmap.save();
    } else if (mode === 1) {
        _.forEach(lastCodeList, o => {
            const item = _.find(sendDoorList, m => m.sendDoorEndPointLastCode === o);
            item && _.forEach(typeList, k => { item[keys[k]] *= (value / 100 + 1); item[timeKeys[k]] = now; });
        });
        await roadmap.save();
    } else {
        let successCount = 0;
        const relateRoadmapList = await RoadmapModel.find({ shopId: roadmap.shopId, endPointLastCode: roadmap.endPointLastCode });
        for (const i in lastCodeList) {
            const item = _.find(sendDoorList, m => m.sendDoorEndPointLastCode === lastCodeList[i]);
            if (!item) {
                continue;
            }
            let updateCount = 0;
            for (const j in typeList) {
                const key = keys[typeList[j]];
                const baseKey = baseKeys[typeList[j]];
                const timeKey = timeKeys[typeList[j]];
                if (item[baseKey]) { // 只有设置了底价的才能安装名次排名
                    const _sendDoorListList = _.sortBy(
                        _.map(relateRoadmapList, o => {
                            const node = _.find(o.sendDoorList, m => lastCodeList[i] === m.sendDoorEndPointLastCode);
                            const _node = _.cloneDeep(node);
                            _node.roadmapId = o.id;
                            return _node;
                        }),
                        n => n[key]
                    );
                    const relateNode = _sendDoorListList[value - 1];
                    const newPrice = relateNode[key] - 0.01; // 注意： 这里每次加分减少1分钱
                    if (relateNode && _T(relateNode.roadmapId) !== roadmapId && newPrice > item[baseKey]) {
                        updateCount++;
                        item[key] = newPrice;
                        item[timeKey] = now;
                    }
                }
            }
            if (updateCount) {
                successCount++;
            }
        }
        await roadmap.save();
        if (successCount === 0) {
            return { msg: '设置失败，可能是没有设置底价、或者价格已经超过底价、或者已经是第' + value + '名' };
        }
        if (successCount !== lastCodeList.length) {
            return { msg: '部分设置失败，可能是没有设置底价、或者价格已经超过底价、或者已经是第' + value + '名' };
        }
    }

    actionLog(userId, 'Client', roadmapId, 'Roadmap', 'setRoadmapSendDoorPrice');

    return { success: true, context: { sendDoorList: _.filter(roadmap.toObject().sendDoorList, o => _.includes(lastCodeList, o.sendDoorEndPointLastCode)) } };
};

```

* PDShopServer/project/App/routers/posts/shipper/order/confirmHandOverOrder.js

```js
import { ClientModel, OrderModel, TruckModel, StartPointAddressModel } from '../../../../models';
import { _T, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

// 该接口只有在物流公司到达地没有分店的情况使用,否则必须由分店进行分发货物
export default async ({
    userId,
    orderId,
    isScan, //是否是扫描，如果是扫描，则orderId传的是原始货单的id，否则就是传的当前货单的id
}) => {
    const client = await ClientModel.findById(userId);
    if (!client || !client.shipperId) {
        return { msg: '无效操作' };
    }
    let order;
    if (isScan) {
        order = await OrderModel.getOrderFromOriginOrder(client.shipperId, orderId, 1);
    } else {
        order = await OrderModel.findById(orderId);
    }
    if (!order) {
        return { msg: '没有该货单' };
    }
    if (order.stateList[0].state !== CONSTANTS.OS_ON_THE_WAY) {
        return { msg: '货单状态错误' };
    }
    if (_T(client.shipperId) !== _T(order.shipperId)) {
        return { msg: '你无权确认' };
    }
    // 判断货单的终点地方是否有分店
    if (await StartPointAddressModel.findOne({ code: order.endPointLastCode })) {
        return { msg: '你无权确认，该货单必须拉到分店进行分发' };
    }
    order.stateList = [ { state: CONSTANTS.OS_READY_RECEIVE, count: order.totalNumbers } ];
    order.confirmHandOverId = userId;
    await order.save();
    // 当物流公司确认卸货后为待交货状态，需要修改它之前所有的货单的状态
    const orderIdList = await OrderModel.getOrderIdListFromNode(order);
    if (orderIdList.length) {
        await OrderModel.update({ _id: { $in: orderIdList } }, { 'stateList.0.state': CONSTANTS.OS_READY_RECEIVE }, { multi: true });
    }

    // 如果货车没有确认到达，需要强制确认到达
    await TruckModel.findByIdAndUpdate(order.truckId, { state: CONSTANTS.TS_RECEIVE_SUCCESS });

    actionLog(userId, 'Client', orderId, 'Order', 'confirmHandOver');

    return { success: true, context: { order } };
};

```

* PDShopServer/project/App/routers/posts/shipper/order/getNeedSendOrderListByEndPoint.js

```js
import { OrderModel } from '../../../../models';
import { getOwnShipper } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, shopId, endPoint, fromPC, pageNo, pageSize }) => {
    const shipperId = await getOwnShipper(userId);
    const criteria = { shipperId, shopId, endPoint, 'stateList.state': { $in: [ CONSTANTS.OS_READY_STOCK, CONSTANTS.OS_READY_LOAD ] } };
    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const query = OrderModel.find(criteria).sort({ weight: 'desc', size: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        photo: 1,
        shopId: 1,
        agentId: 1,
        senderPhone: 1,
        senderName: 1,
        receiverPhone: 1,
        receiverName: 1,
        name: 1,
        roadmapId: 1,
        realFee: 1,
        createTime: 1,
        placeOrderTime: 1,
        weight: 1,
        size: 1,
        stateList: 1,
        totalNumbers: 1,
        startPoint: 1,
        endPoint: 1,
        sendDoorEndPoint: 1,
        isSendDoor: 1,
        proxyCharge: 1,
        needPayTransportFee: 1,
        needPayInsuanceFee: 1,
        payMode: 1,
        isReachPay: 1,
        totalDesignatedFee: 1,
        isInsuance: 1,
    }).populate({
        path: 'roadmapId',
        select: { startPoint: 1, endPoint: 1 },
    });

    return { success: true, context: {
        count,
        orderList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shipper/order/getNeedSendOrderSummaryList.js

```js
import { OrderModel } from '../../../../models';
import { getKeywordCriteriaForOrder, getOwnShipper } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import mongoose from 'mongoose';

export default async ({ userId, shopId, keyword, fromPC, pageNo, pageSize }) => {
    const shipperId = await getOwnShipper(userId);
    const criteria = getKeywordCriteriaForOrder(keyword, {
        shipperId: new mongoose.Types.ObjectId(shipperId),
        shopId: new mongoose.Types.ObjectId(shopId),
        'stateList.state': { $in: [ CONSTANTS.OS_READY_STOCK, CONSTANTS.OS_READY_LOAD ] },
    });
    let count;
    if (fromPC && pageNo === 0) {
        const countResults = await OrderModel.aggregate()
        .match(criteria)
        .group({
            _id: '$endPoint',
        })
        .group({
            _id: null,
            total: { $sum: 1 },
        });
        count = countResults.length ? countResults[0].total : 0;
    }

    const docs = await OrderModel.aggregate()
    .match(criteria)
    .group({
        _id: '$endPoint',
        orderCount: { $sum: 1 },
        totalNumbers: { $sum: '$totalNumbers' },
        totalSize: { $sum: '$size' },
        totalWeight: { $sum: '$weight' },
        totalFee: { $sum: '$fee' },
    })
    .project({
        endPoint: '$_id',
        _id: 0,
        orderCount: 1,
        totalNumbers: 1,
        totalSize: 1,
        totalWeight: 1,
        totalFee: 1,
    })
    .sort('totalWeight totalSize')
    .skip(pageNo * pageSize)
    .limit(pageSize);

    return { success: true, context: {
        count,
        orderList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shipper/order/getOrders.js

```js
import _ from 'lodash';
import { OrderModel, ClientModel } from '../../../../models';
import { getKeywordCriteriaForOrder, omitNil, hasAuthority } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import moment from 'moment';

async function getPageData (shopId, shipperId, type, params) {
    let { keyword, startDate, endDate, fromPC, pageNo, pageSize } = params;
    let criteria = {};
    if (type === 'tostore') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_STOCK };
    } else if (type === 'stored') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_LOAD };
    } else if (type === 'tostartoff') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_START_OFF };
    } else if (type === 'onway') {
        criteria = { 'stateList.state': CONSTANTS.OS_ON_THE_WAY };
    } else if (type === 'success') {
        criteria = { 'stateList.state': CONSTANTS.OS_RECEIVE_SUCCESS };
    }
    if (fromPC) {
        if (startDate && endDate) {
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        } else {
            startDate = endDate = moment().format('YYYY-MM-DD');
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        }
    }
    criteria = getKeywordCriteriaForOrder(keyword, omitNil({ ...criteria, shopId, shipperId }));

    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        shopId: 1,
        shipperId: 1,
        senderName: 1,
        senderPhone: 1,
        receiverName: 1,
        receiverPhone: 1,
        name: 1,
        endPoint: 1,
        createTime: 1,
        modifyTime: 1,
        isReachPay: 1,
        payMode: 1,
        receiveMemberId: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        stateList: 1,
        isInsuance: 1,
        insuanceMount: 1,
        insuanceFee: 1,
        fee: 1,
        profit: 1,
        masterProfit: 1,
        branchProfit: 1,
        realFee: 1,
        proxyCharge: 1,
        proxyChargeProfit: 1,
        needPayTransportFee: 1,
        needPayInsuanceFee: 1,
        totalDesignatedFee: 1,
        designatedFee: 1,
        placeOrderTime: 1,
    }).populate({
        path: 'receiveMemberId',
        select: { name: 1, phone: 1 },
    }).populate({
        path: 'shipperId',
        select: { name: 1 },
    }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });

    return {
        count,
        list: docs,
    };
};

// ['待入库', '已入库', '待出发', '运输中', '成功']
const TYPES = ['tostore', 'stored', 'tostartoff', 'onway', 'success'];
export default async ({ userId, shopId, type, ...params }) => {
    const client = await ClientModel.findById(userId);
    if (!client || !client.shipperId || !hasAuthority(client, CONSTANTS.AH_LOOK_ORDERS)) {
        return { msg: '你没有查看货单的权限' };
    }
    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(shopId, client.shipperId, type, params),
            },
        };
    } else {
        return {
            success: true,
            context: {
                tostore: await getPageData(shopId, client.shipperId, 'tostore', params),
                stored: await getPageData(shopId, client.shipperId, 'stored', params),
                tostartoff: await getPageData(shopId, client.shipperId, 'tostartoff', params),
                onway: await getPageData(shopId, client.shipperId, 'onway', params),
                success: await getPageData(shopId, client.shipperId, 'success', params),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/shipper/statistics/getStatistics.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import CONSTANTS from '../../../../constants';
import { hasAuthority } from '../../../../utils';
import mongoose from 'mongoose';
import moment from 'moment';
import _ from 'lodash';
const MAX_DAYS = 30;

export default async ({ userId, days = 7 }) => {
    const client = await ClientModel.findById(userId);
    if (!client) {
        return { msg: '你没有查看统计的权限' };
    }
    if (!client.shipperId || !hasAuthority(client, CONSTANTS.AH_LOOK_STATISTICS)) {
        return { msg: '你没有查看统计的权限' };
    }

    days = (days > MAX_DAYS ? MAX_DAYS : days < 1 ? 1 : days) + 1;
    const now = moment().startOf('d').add(1, 'd');
    const timeList = _.map(_.rangeRight(-days), o => now.clone().add(o, 'd'));
    const getCond = (name, value) => _.dropRight(_.map(timeList, (o, k) => timeList[k + 1] && { [name + k]: { $sum: { $cond: { if: { $and: [{ $gte: ['$placeOrderTime', o.toDate()] }, { $lt: ['$placeOrderTime', timeList[k + 1].toDate()] }] }, then: value, else: 0 } } } }));

    const cond = [
        ...getCond('count', 1),
        ...getCond('isReachPay', '$isReachPay'),
        ...getCond('fee', '$fee'),
        ...getCond('totalDesignatedFee', '$totalDesignatedFee'),
        ...getCond('proxyCharge', '$proxyCharge'),
        ...getCond('weight', '$weight'),
        ...getCond('size', '$size'),
    ];

    const daysInfo = await OrderModel.aggregate()
    .match({ shipperId: new mongoose.Types.ObjectId(client.shipperId), placeOrderTime: { $gt: timeList[0].toDate() }, 'stateList.0.state': { $gte: CONSTANTS.OS_READY_STOCK } })
    .project({ placeOrderTime: 1, isReachPay: { $cond: { if: '$isReachPay', then: 1, else: 0 } }, fee: 1, totalDesignatedFee: 1, proxyCharge: 1, weight: 1, size: 1 })
    .group({
        _id: null,
        ..._.reduce(cond, (r, o) => Object.assign(r, o)),
    });

    daysInfo[0] && (delete daysInfo[0]._id);
    const context = _.transform(daysInfo[0], (r, v, k) => {
        const key = k.replace(/\d+/, '');
        r[key] || (r[key] = []);
        r[key].push(v);
    });

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/createTruck.js

```js
import { TruckModel, ClientModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    shopId,
    plateNo, // 车牌号码
    drivingLicense, // 行驶证编号
    truckType, // 货车类型（包括车长）
    insuanceMount, // 保险
    driverId, //司机信息
}) => {
    const client = await ClientModel.findById(userId);
    if (!hasAuthority(client, CONSTANTS.AH_CREATE_TRUCK)) {
        return { msg: '你没有创建货车的权限' };
    }
    const truck = await TruckModel.findOne({ $or: [ { driverId }, { plateNo } ], state: { $ne: CONSTANTS.TS_RECEIVE_SUCCESS }});
    if (truck) {
        return { msg: '司机当前不可用' };
    }
    const doc = new TruckModel({
        shipperId: client.shipperId,
        shopId,
        plateNo,
        drivingLicense,
        truckType,
        insuanceMount,
        driverId,
    });
    await doc.save();

    const context = await TruckModel.findById(doc.id)
    .select({
        plateNo: 1,
        drivingLicense: 1,
        truckType: 1,
        driverId: 1,
        locationList: 1,
        insuanceMount: 1,
        state: 1,
    })
    .populate({
        path: 'driverId',
        select: { name: 1, phone: 1 },
    });

    actionLog(userId, 'Client', doc.id, 'Truck', 'createTruck');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/getDriverInfoByPhone.js

```js
import { DriverModel, TruckModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({ userId, phone }) => {
    const doc = await DriverModel.findOne({ phone });
    if (!doc) {
        return { msg: '没有对应的司机' };
    }
    const truckList = await TruckModel.find({ driverId: doc.id, state: CONSTANTS.TS_RECEIVE_SUCCESS });
    if (!truckList) {
        return { msg: '司机当前不可用' };
    }
    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/getHistoryTruckList.js

```js
import { TruckModel, ClientModel } from '../../../../models';
import { getKeywordCriteriaForTruck } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, shopId, keyword, fromPC, pageNo, pageSize }) => {
    const client = await ClientModel.findById(userId);
    const criteria = getKeywordCriteriaForTruck(keyword, { shipperId: client.shipperId, shopId, state: CONSTANTS.TS_RECEIVE_SUCCESS });
    const count = (fromPC && pageNo === 0) ? await TruckModel.count(criteria) : undefined;
    const query = TruckModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        plateNo: 1,
        drivingLicense: 1,
        truckType: 1,
        driverId: 1,
        locationList: 1,
        insuanceMount: 1,
        createTime: 1,
    })
    .populate({
        path: 'driverId',
        select: { name: 1, phone: 1, licenseNo: 1 },
    });

    return { success: true, context: {
        count,
        truckList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/getOrderListInTruck.js

```js
import { TruckModel } from '../../../../models';

export default async ({ userId, truckId }) => {
    const truck = await TruckModel.findById(truckId)
    .populate({
        path: 'orderList',
        select: {
            photo: 1,
            shopId: 1,
            agentId: 1,
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            name: 1,
            receiverId: 1,
            roadmapId: 1,
            realFee: 1,
            createTime: 1,
            placeOrderTime: 1,
            weight: 1,
            size: 1,
            stateList: 1,
            totalNumbers: 1,
            startPoint: 1,
            endPoint: 1,
            proxyCharge: 1,
            needPayInsuanceFee: 1,
            payMode: 1,
            isSendDoor: 1,
            isReachPay: 1,
            totalDesignatedFee: 1,
            isInsuance: 1,
        },
        populate: [{
            path: 'shopId',
            select: { name: 1, address: 1 },
        }, {
            path: 'agentId',
            select: { name: 1, address: 1 },
        }],
    });

    return { success: true, context: {
        orderList: (truck || {}).orderList || [],
    } };
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/getTruckDetail.js

```js
import { TruckModel } from '../../../../models';
import _ from 'lodash';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    truckId,
}) => {
    const doc = await TruckModel.findById(truckId)
    .populate({
        path: 'driverId',
        select: { name: 1, phone: 1 },
    })
    .populate({
        path: 'scannerId',
        select: { name: 1, phone: 1 },
    })
    .populate({
        path: 'carryPartmentId',
        select: { name: 1, phoneList: 1 },
    })
    .populate({
        path: 'unloadAllOrderList',
        select: {
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            endPoint: 1,
            stateList: 1,
            totalNumbers: 1,
        },
    });
    if (!doc) {
        return { msg: '没有该货车' };
    }

    const context = doc.toObject();
    _.forEach(context.unloadAllOrderList, item => {
        item.unloadNumber = (_.find(item.stateList, o => o.state === CONSTANTS.OS_READY_LOAD) || {}).count || 0;
        delete item.stateList;
    });

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/getTruckType.js

```js
export default async ({ userId }) => {
    const context = [
        { name: '车长', list: [ '3.5米', '4米', '4.5米', '5米', '5.5米', '6米', '6.5米', '7米' ] },
        { name: '车型', list: [ '不限', '高栏', '平板', '箱式', '高低板', '保温' ] },
    ];

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/getTrucks.js

```js
import _ from 'lodash';
import { TruckModel, ClientModel } from '../../../../models';
import { getKeywordCriteriaForTruck } from '../../../../utils';
import CONSTANTS from '../../../../constants';

async function getPageData (shipperId, shopId, type, keyword, fromPC, pageNo, pageSize) {
    let criteria = getKeywordCriteriaForTruck(keyword, { shipperId, shopId });
    if (type === 'onway') {
        criteria = { ...criteria, state: CONSTANTS.TS_ON_THE_WAY };
    } else if (type === 'complete') {
        criteria = { ...criteria, state: CONSTANTS.TS_RECEIVE_SUCCESS };
    }
    const count = (fromPC && pageNo === 0) ? await TruckModel.count() : undefined;
    const query = TruckModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        plateNo: 1,
        drivingLicense: 1,
        truckType: 1,
        driverId: 1,
        locationList: 1,
        insuanceMount: 1,
    })
    .populate({
        path: 'driverId',
        select: { name: 1, phone: 1, licenseNo: 1 },
    });

    return {
        count,
        list: docs,
    };
};

// ['运输中', '已完成']
const TYPES = ['onway', 'complete'];
export default async ({ userId, shopId, type, keyword, fromPC, pageNo, pageSize }) => {
    const client = await ClientModel.findById(userId);
    if (!client || !client.shipperId) {
        return { msg: '权限不足' };
    }
    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(client.shipperId, shopId, type, keyword, fromPC, pageNo, pageSize),
            },
        };
    } else {
        return {
            success: true,
            context: {
                onway: await getPageData(client.shipperId, shopId, 'onway', keyword, fromPC, pageNo, pageSize),
                complete: await getPageData(client.shipperId, shopId, 'complete', keyword, fromPC, pageNo, pageSize),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/getUnfinishTruck.js

```js
import { OrderModel, ClientModel, TruckModel } from '../../../../models';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({ userId, shopId }) => {
    const client = await ClientModel.findById(userId);
    const truck = await TruckModel.findOne({
        shopId,
        shipperId: client.shipperId,
        scannerId: userId,
        state: { $in: [ CONSTANTS.TS_SCAN_LOADING, CONSTANTS.TS_READY_PAY_FOR_CARRY_PARTMENT ] },
    })
    .populate({
        path: 'unloadAllOrderList',
        select: {
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            endPoint: 1,
            stateList: 1,
            totalNumbers: 1,
        },
    });
    if (!truck) {
        return { success: false };
    }
    const context = truck.toObject();
    _.forEach(context.unloadAllOrderList, item => {
        item.unloadNumber = (_.find(item.stateList, o => o.state === CONSTANTS.OS_READY_LOAD) || {}).count || 0;
        delete item.stateList;
    });
    context.latestOrder = await OrderModel.findById(context.orderList[0])
    .select({
        createTime: 1,
        photo: 1,
        endPoint: 1,
        senderPhone: 1,
        senderName: 1,
        receiverPhone: 1,
        receiverName: 1,
        totalNumbers: 1,
        size: 1,
        weight: 1,
    });
    delete context.orderList;

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/getWorkTruckList.js

```js
import { TruckModel, ClientModel, OrderModel } from '../../../../models';
import { getKeywordCriteriaForTruck } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

// 该接口如果返回 truck, 根据truck的状态显示不同的界面，否则显示列表
export default async ({ userId, shopId, keyword, fromPC, pageNo, pageSize }) => {
    const client = await ClientModel.findById(userId);
    if (!fromPC) {
        const truck = await TruckModel.findOne({
            shopId,
            shipperId: client.shipperId,
            scannerId: userId,
            state: { $in: [ CONSTANTS.TS_SCAN_LOADING, CONSTANTS.TS_READY_PAY_FOR_CARRY_PARTMENT ] },
        })
        .populate({
            path: 'unloadAllOrderList',
            select: {
                endPoint: 1,
                senderPhone: 1,
                senderName: 1,
                receiverPhone: 1,
                receiverName: 1,
                stateList: 1,
                totalNumbers: 1,
            },
        });
        if (truck) {
            const context = truck.toObject();
            _.forEach(context.unloadAllOrderList, item => {
                item.unloadNumber = (_.find(item.stateList, o => o.state === CONSTANTS.OS_READY_LOAD) || {}).count || 0;
                delete item.stateList;
            });
            context.latestOrder = await OrderModel.findById(context.orderList[0])
            .select({
                createTime: 1,
                photo: 1,
                endPoint: 1,
                senderPhone: 1,
                senderName: 1,
                receiverPhone: 1,
                receiverName: 1,
                totalNumbers: 1,
                size: 1,
                weight: 1,
            });
            delete context.orderList;
            return { success: true, context };
        }
    }
    const criteria = getKeywordCriteriaForTruck(keyword, { shipperId: client.shipperId, shopId, state: { $lte: CONSTANTS.TS_READY_EXIT_WAREHOUSE } });
    const count = (fromPC && pageNo === 0) ? await TruckModel.count(criteria) : undefined;
    const query = TruckModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        plateNo: 1,
        drivingLicense: 1,
        truckType: 1,
        driverId: 1,
        locationList: 1,
        insuanceMount: 1,
        state: 1,
    })
    .populate({
        path: 'driverId',
        select: { name: 1, phone: 1 },
    });

    return { success: true, context: {
        count,
        truckList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/modifyTruck.js

```js
import { TruckModel } from '../../../../models';
import { omitNil, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    truckId,
    plateNo, // 车牌号码
    drivingLicense, // 行驶证编号
    truckType, // 货车类型（包括车长）
    insuanceMount, // 保险
    driverId, //司机信息
}) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_TRUCK, true)) {
        return { msg: '你没有修改货车的权限' };
    }
    const doc = await TruckModel.findByIdAndUpdate(truckId, omitNil({
        plateNo,
        drivingLicense,
        truckType,
        insuanceMount,
        driverId,
    }), { new: true })
    .select({
        plateNo: 1,
        drivingLicense: 1,
        truckType: 1,
        driverId: 1,
        locationList: 1,
        insuanceMount: 1,
        state: 1,
    })
    .populate({
        path: 'driverId',
        select: { name: 1, phone: 1 },
    });
    if (!doc) {
        return { msg: '修改失败' };
    }

    actionLog(userId, 'Client', truckId, 'Truck', 'modifyTruck');

    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shipper/truck/removeTruck.js

```js
import { TruckModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    truckId,
}) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_REMOVE_TRUCK, true)) {
        return { msg: '你没有删除货车的权限' };
    }
    await TruckModel.findByIdAndRemove(truckId);
    actionLog(userId, 'Client', truckId, 'Truck', 'removeTruck');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/account/getBillList.js

```js
import { ShopMemberModel } from '../../../../models';
import { AccountModel, BillModel } from '../../../../mysql';
import { _T, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    pageNo,
    pageSize,
}) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { isMasterShop: 1 },
    });
    if (!hasAuthority(member, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
        return { msg: '你没有查看账单的权限' };
    }

    let accountId = CONSTANTS.AC_MASTER_ACCOUNT;
    if (member.partmentId) {
        accountId = (await AccountModel.findOne({ where: { targetId: _T(member.partmentId), type: CONSTANTS.AT_BRANCH_SHOP_PARTMENT } }) || {}).id;
    } else if (!member.shop.isMasterShop) {
        accountId = (await AccountModel.findOne({ where: { targetId: _T(member.shop.id), type: CONSTANTS.AT_BRANCH_SHOP } }) || {}).id;
    }
    if (!accountId) {
        return { msg: '没有该账户' };
    }
    const docs = await BillModel.findAndCountAll({
        order: [['tradeTime', 'DESC']],
        where: { account: accountId },
        offset: pageNo * pageSize,
        limit: pageSize,
        attributes: [
            'tradeAmount',
            'thirdpartyAccount',
            'orderId',
            'remark',
            'tradeTime',
        ],
    });
    return { success: true, context: { count: docs.count, billList: docs.rows } };
};

```

* PDShopServer/project/App/routers/posts/shop/account/getBills.js

```js
import _ from 'lodash';
import { ShopMemberModel } from '../../../../models';
import { AccountModel, BillModel } from '../../../../mysql';
import { _T, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';
import moment from 'moment';

async function getPageData (accountId, type, params) {
    let { startDate, endDate, fromPC, pageNo, pageSize } = params;
    let options = (type === 'recharge' || type === 'withdraw') ? { thirdpartyAccount: { $ne: null } } : { thirdpartyAccount: { $eq: null } };
    let options1 = (type === 'recharge' || type === 'income') ? { tradeAmount: { $gt: 0 } } : { tradeAmount: { $lt: 0 } };

    let criteria = {};
    if (fromPC) {
        if (startDate && endDate) {
            criteria = { ...criteria, $and: [{ tradeTime: { $gte: moment(startDate).toDate() } }, { tradeTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        } else {
            startDate = endDate = moment().format('YYYY-MM-DD');
            criteria = { ...criteria, $and: [{ tradeTime: { $gte: moment(startDate).toDate() } }, { tradeTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        }
    }

    const docs = await BillModel.findAndCountAll({
        order: [['tradeTime', 'DESC']],
        where: { account: accountId, ...options, ...options1, ...criteria },
        offset: pageNo * pageSize,
        limit: pageSize,
        attributes: [
            'tradeAmount',
            'thirdpartyAccount',
            'orderId',
            'truckId',
            'remark',
            'tradeTime',
        ],
    });

    return {
        count: docs.count,
        list: docs.rows,
    };
};

// ['充值账单', '提现账单', '收入账单', '支出账单']
const TYPES = ['recharge', 'withdraw', 'income', 'pay'];
export default async ({ userId, type, ...params }) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { isMasterShop: 1 },
    });
    if (!hasAuthority(member, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
        return { msg: '你没有查看账单的权限' };
    }

    let accountId = CONSTANTS.AC_MASTER_ACCOUNT;
    if (member.partmentId) {
        accountId = (await AccountModel.findOne({ where: { targetId: _T(member.partmentId), type: CONSTANTS.AT_BRANCH_SHOP_PARTMENT } }) || {}).id;
    } else if (!member.shop.isMasterShop) {
        accountId = (await AccountModel.findOne({ where: { targetId: _T(member.shop.id), type: CONSTANTS.AT_BRANCH_SHOP } }) || {}).id;
    }
    if (!accountId) {
        return { msg: '没有该账户' };
    }

    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(accountId, type, params),
            },
        };
    } else {
        return {
            success: true,
            context: {
                recharge: await getPageData(accountId, 'recharge', params),
                withdraw: await getPageData(accountId, 'withdraw', params),
                income: await getPageData(accountId, 'income', params),
                pay: await getPageData(accountId, 'pay', params),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/shop/account/getRemainAmount.js

```js
import { ShopMemberModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _T, _Y, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    let amount;
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { isMasterShop: 1 },
    });
    if (!hasAuthority(member, [CONSTANTS.AH_LOOK_AMOUNT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW])) {
        return { msg: '你没有查看余额的权限' };
    }
    if (member.shop.isMasterShop) {
        const masterAccount = await AccountModel.findOne({ where: { id: CONSTANTS.AC_MASTER_ACCOUNT } });
        if (!masterAccount) {
            return { msg: '没有该账户' };
        }
        amount = _Y(masterAccount.amount);
    } else if (member.partmentId) {
        const partmentAccount = await AccountModel.findOne({ where: { targetId: _T(member.partmentId), type: CONSTANTS.AT_BRANCH_SHOP_PARTMENT } });
        if (!partmentAccount) {
            return { msg: '没有该账户' };
        }
        amount = _Y(partmentAccount.amount);
    } else {
        const branchAccount = await AccountModel.findOne({ where: { targetId: _T(member.shop.id), type: CONSTANTS.AT_BRANCH_SHOP } });
        if (!branchAccount) {
            return { msg: '没有该账户' };
        }
        amount = _Y(branchAccount.amount);
    }
    if (amount === undefined) {
        return { msg: '你没有查看余额的权限' };
    }

    return { success: true, context: { amount } };
};

```

* PDShopServer/project/App/routers/posts/shop/account/recharge.js

```js
import { _F, actionLog } from '../../../../utils/';
import { rechargeForShopMember } from '../../libs/account';

export default ({
    userId,
    amount,
}) => {
    amount = _F(amount);
    const result = rechargeForShopMember({ userId, amount, thirdpartyAccount: 'SIMULATION_NO' });
    result.success && actionLog(userId, 'ShopMember', amount, 'Number', 'recharge');

    return result;
};

```

* PDShopServer/project/App/routers/posts/shop/account/setPaymentPassword.js

```js
import { PaymentModel, ShopMemberModel } from '../../../../models';
import { registerUser, setPassword, checkVerifyCode, actionLog } from '../../../../utils';

export default async ({ userId, phone, password, verifyCode }) => {
    const success = await checkVerifyCode(phone, verifyCode);
    if (!success) {
        return { msg: '无效验证码' };
    }
    const member = await ShopMemberModel.findById(userId);
    if (!member || member.phone !== phone) {
        return { msg: '获取验证码的手机号码必须为登录的手机号码' };
    }

    let payment = await PaymentModel.findOne({ userId });
    if (!payment) {
        payment = new PaymentModel({ userId });
        const error = await registerUser(PaymentModel, payment, password);
        if (error) {
            return { msg: '设置密码失败' };
        }
        member.isSetPaymentPassword = 1;
        await member.save();
        actionLog(userId, 'ShopMember', userId, 'ShopMember', 'setPaymentPassword');
        return { success: true };
    }

    const error = await setPassword(payment, password);
    if (error) {
        return { msg: '设置密码失败' };
    }
    await payment.save();

    actionLog(userId, 'ShopMember', userId, 'ShopMember', 'setPaymentPassword');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/account/withdraw.js

```js
import { _F, authenticatePaymentPassword, actionLog } from '../../../../utils/';
import { withdrawForShopMember } from '../../libs/account';

export default async ({
    userId,
    amount,
    payPassword,
}) => {
    const error = await authenticatePaymentPassword(userId, payPassword);
    if (error) {
        return { msg: '密码错误' };
    }
    amount = _F(amount);
    const result = withdrawForShopMember({ userId, amount, thirdpartyAccount: 'SIMULATION_NO' });
    result.success && actionLog(userId, 'ShopMember', amount, 'Number', 'withdraw');

    return result;
};

```

* PDShopServer/project/App/routers/posts/shop/agent/checkAgentByChairManPhone.js

```js
import { ClientModel } from '../../../../models';
import { _T } from '../../../../utils/';

export default async ({
    userId,
    chairManPhone, // 收货点董事长 Id
}) => {
    const chairMan = await ClientModel.findOne({ phone: chairManPhone })
    .populate({
        path: 'agentId',
    });
    if (!chairMan) {
        return { msg: '必须先注册董事长账号' };
    }
    if (chairMan.agent && _T(chairMan.agent.chairManId) === _T(chairMan.id)) {
        return { msg: '该号码已经拥有收货点，请使用其他号码' };
    }
    return { success: true, context: { chairMan: { id: chairMan.id, name: chairMan.name, phone: chairManPhone } } };
};

```

* PDShopServer/project/App/routers/posts/shop/agent/createAgent.js

```js
import { AgentModel, ClientModel, MediaModel } from '../../../../models';
import { getMediaId, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    chairManId, // 收货点董事长 Id
    name, // 收货点名称
    image, // 收货点背景图片
    sign, // 收货点签名
    phoneList, // 收货点联系电话
    addressRegion, // 物流超市所在城市
    addressRegionLastCode, // 物流超市所在城市的lastCode
    address, // 商铺地址（包含经纬度）
    location, // 经纬度定位
    legalName, // 法人姓名
    legalPhone, // 法人电话
    legalIDCard, //法人身份证
}) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_CREATE_AGENT)) {
        return { msg: '你没有创建收货点的权限' };
    }

    let chairMan = await ClientModel.findById(chairManId);
    if (!chairMan) {
        return { msg: '必须先注册董事长账号' };
    }
    if (chairMan.agentId) {
        return { msg: '该董事长账号只能注册一家收货点' };
    }
    if (chairMan.shipperId) {
        return { msg: '该董事长账号已经注册了一家物流公司，不能再注册收货点' };
    }
    // 创建收货点
    let _image = getMediaId(image);
    let _legalIDCard = _.map(legalIDCard, item => getMediaId(item));

    const agent = new AgentModel({
        chairManId: chairMan.id,
        name,
        image: _image,
        sign,
        phoneList,
        addressRegion,
        addressRegionLastCode,
        address,
        location,
        legalName,
        legalPhone,
        legalIDCard: _legalIDCard,
    });
    // 赋予收货点董事长权限
    chairMan.authority = [
        CONSTANTS.AH_MODIFY_AGENT_INFO, // 修改收货点信息的权限
        CONSTANTS.AH_MODIFY_AGENT_MEMBER_AUTHORITY, // 修改收货点成员权限的权限
        CONSTANTS.AH_PLACE_ORDER, // 收货下单的权限
        // 公共权限
        CONSTANTS.AH_MODIFY_ROADMAP_PROFIT, // 修改路线的提成的权限
        CONSTANTS.AH_RECHARGE, // 充值的权限
        CONSTANTS.AH_WITHDRAW, // 提现的权限
        CONSTANTS.AH_LOOK_AMOUNT, // 查看余额的权限
        // 统计
        CONSTANTS.AH_LOOK_ORDERS, // 查看综合货单的权限
        CONSTANTS.AH_LOOK_STATISTICS, // 查看统计信息的权限
    ];
    chairMan.agentId = agent.id;
    // 保存
    await chairMan.save();
    await agent.save();

    MediaModel._updateRef(
        { [_image]: 1 },
        ..._legalIDCard.map(item => ({ [item]: 1 })),
    );

    const context = await AgentModel.findById(agent.id)
    .select({
        name: 1,
        addressRegion: 1,
        addressRegionLastCode: 1,
        address: 1,
        location: 1,
        phoneList: 1,
        chairManId: 1,
    })
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });

    actionLog(userId, 'ShopMember', agent.id, 'Agent', 'createAgent');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/agent/getAgentDetail.js

```js
import { AgentModel } from '../../../../models';
import { hasAuthority } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, agentId }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_LOOK_AGENT)) {
        return { msg: '你没有查看收货点的权限' };
    }

    const agent = await AgentModel.findById(agentId)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });
    if (!agent) {
        return { msg: '没有该收货点' };
    }
    return { success: true, context: agent };
};

```

* PDShopServer/project/App/routers/posts/shop/agent/getAgentList.js

```js
import { AgentModel } from '../../../../models';
import { getKeywordCriteriaForShipper, hasAuthority } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, keyword, fromPC, pageNo, pageSize }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_LOOK_AGENT)) {
        return { msg: '你没有查看收货点的权限' };
    }

    const criteria = getKeywordCriteriaForShipper(keyword);
    const count = (fromPC && pageNo === 0) ? await AgentModel.count(criteria) : undefined;
    const query = AgentModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        name: 1,
        addressRegion: 1,
        addressRegionLastCode: 1,
        address: 1,
        location: 1,
        phoneList: 1,
        chairManId: 1,
    })
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });

    return { success: true, context: {
        count,
        agentList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/agent/modifyAgent.js

```js
import _ from 'lodash';
import { AgentModel, ShopMemberModel, MediaModel } from '../../../../models';
import { getMediaId, hasAuthority, omitNil, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    agentId,
    name, // 物流公司名称
    image, // 物流公司背景图片
    sign, // 物流公司签名
    phoneList, // 物流公司联系电话
    addressRegion, // 物流超市所在城市
    addressRegionLastCode, // 物流超市所在城市的lastCode
    address, // 商铺地址
    location, // 经纬度定位
    legalName, // 法人姓名
    legalPhone, // 法人电话
    legalIDCard, // 法人身份证
    bondCompanyList, //担保公司列表
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_AGENT)) {
        return { msg: '你没有修改收货点信息的权限' };
    }

    let _image = getMediaId(image);
    let _legalIDCard = _.map(legalIDCard, o => getMediaId(o));

    const agent = await AgentModel.findByIdAndUpdate(agentId, omitNil({
        name,
        image: _image,
        sign,
        phoneList,
        addressRegion,
        addressRegionLastCode,
        address,
        location,
        legalName,
        legalPhone,
        legalIDCard: legalIDCard && _legalIDCard,
    }));

    MediaModel._updateRef(
        { [_image]: 1 },
        { [image && agent.image]: -1 },
        ..._legalIDCard.map(item => ({ [item]: 1 })),
        ...(legalIDCard ? agent.legalIDCard : []).map(item => ({ [item]: -1 })),
    );

    const context = await AgentModel.findById(agentId)
    .select({
        name: 1,
        addressRegion: 1,
        addressRegionLastCode: 1,
        address: 1,
        location: 1,
        phoneList: 1,
        chairManId: 1,
    })
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });

    actionLog(userId, 'ShopMember', agentId, 'Agent', 'modifyAgent');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/agent/removeAgent.js

```js
import { ShopMemberModel, AgentModel, MediaModel } from '../../../../models';
import { hasAuthority, authenticatePassword, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    agentId,
    password,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_REMOVE_AGENT)) {
        return { msg: '你没有删除收货点的权限' };
    }
    const error = await authenticatePassword(member, password);
    if (error) {
        return { msg: '密码错误' };
    }

    const agent = await AgentModel.findByIdAndRemove(agentId);
    if (agent) {
        MediaModel._updateRef(
            { [agent.image]: -1 },
            ...agent.legalIDCard.map(item => ({ [item]: -1 })),
        );
    }

    actionLog(userId, 'ShopMember', agentId, 'Agent', 'removeAgent');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/branchShop/createAndRegisterShipper.js

```js
import { ShipperModel, ClientModel, ShopMemberModel, ShipperInBranchShopModel, MediaModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _Y, _T, getMediaId, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    shipperType = 0, // 0：长途物流公司 1：同城配送物流公司（默认为0）
    chairManId, // 物流公司董事长 Id
    name, // 物流公司名称
    image, // 物流公司背景图片
    sign, // 物流公司签名
    phoneList, // 物流公司联系电话
    address, // 商铺地址
    capital, // 注册资金
    legalName, // 法人姓名
    legalPhone, // 法人电话
    legalIDCard, // 法人身份证
    bondCompanyList, //担保公司列表
}) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_CREATE_SHIPPER)) {
        return { msg: '你没有注册物流超市的权限' };
    }

    let chairMan = await ClientModel.findById(chairManId);
    if (!chairMan) {
        return { msg: '必须先注册董事长账号' };
    }
    if (chairMan.shipperId) {
        return { msg: '该董事长账号只能注册一家物流公司' };
    }
    if (chairMan.agentId) {
        return { msg: '该董事长账号已经注册了一家收货点，不能再注册物流公司' };
    }
    // 创建物流公司
    let _image = getMediaId(image);
    let _legalIDCard = _.map(legalIDCard, item => getMediaId(item));
    const _bondCompanyList = _.map(bondCompanyList, item => {
        item.certificate = _.map(item.certificate, o => getMediaId(o));
        item.legalIDCard = _.map(item.legalIDCard, o => getMediaId(o));
        return item;
    });
    const totalBondAmount = _.sumBy(_bondCompanyList, o => o.bondAmount);

    const clientAccount = await AccountModel.findOne({ where: { targetId: _T(chairMan.id), type: CONSTANTS.AT_CLIENT } });
    const acountAmount = _Y(clientAccount ? clientAccount.amount : 0);

    const shipper = new ShipperModel({
        shipperType,
        chairManId: chairMan.id,
        name,
        image: _image,
        sign,
        phoneList,
        address,
        capital,
        legalName,
        legalPhone,
        legalIDCard: _legalIDCard,
        acountAmount,
    });
    const commonAuthority = [
        CONSTANTS.AH_MODIFY_SHIPPER_INFO, // 修改物流公司信息的权限
        CONSTANTS.AH_MODIFY_SHIPPER_MEMBER_AUTHORITY, // 修改物流公司成员权限的权限

        // 充值，提现，余额
        CONSTANTS.AH_RECHARGE, // 充值的权限
        CONSTANTS.AH_WITHDRAW, // 提现的权限
        CONSTANTS.AH_LOOK_AMOUNT, // 查看余额的权限
        // 统计
        CONSTANTS.AH_LOOK_STATISTICS, // 查看统计信息的权限
        CONSTANTS.AH_LOOK_ORDERS, // 查看综合货单的权限
    ];

    // 赋予物流公司董事长权限
    if (shipperType === CONSTANTS.ST_LONG) {
        chairMan.authority = [
            ...commonAuthority,
            // 路线
            CONSTANTS.AH_CREATE_ROADMAP, // 创建路线的权限
            CONSTANTS.AH_MODIFY_ROADMAP, // 修改路线的权限
            CONSTANTS.AH_REMOVE_ROADMAP, // 删除路线的权限
            CONSTANTS.AH_LOOK_ROADMAP, // 查看路线的权限
            // 货车
            CONSTANTS.AH_CREATE_TRUCK, // 创建货车的权限
            CONSTANTS.AH_MODIFY_TRUCK, // 修改货车的权限
            CONSTANTS.AH_REMOVE_TRUCK, // 删除货车的权限
            CONSTANTS.AH_LOOK_TRUCK, // 查看货车的权限
            // 装车
            CONSTANTS.SELECT_CARRY_PARTMENT, // 选择搬运队的权限
            CONSTANTS.AH_SCAN_LOAD_TRUCK, // 扫描装车的权限
        ];
    } else {
        chairMan.authority = [
            ...commonAuthority,
            // 同城配送路线
            CONSTANTS.AH_CREATE_SEND_DOOR_ROADMAP, // 创建同城配送路线的权限
            CONSTANTS.AH_MODIFY_SEND_DOOR_ROADMAP, // 修改同城配送路线的权限
            CONSTANTS.AH_REMOVE_SEND_DOOR_ROADMAP, // 删除同城配送路线的权限
            CONSTANTS.AH_LOOK_SEND_DOOR_ROADMAP, // 查看同城配送路线的权限
            // 同城配送
            CONSTANTS.AH_CITY_DISTRIBUTE, // 同城配送的权限
        ];
    }

    chairMan.post = CONSTANTS.PO_CHAIR_MAN;
    chairMan.shipperId = shipper.id;
    // 保存
    await chairMan.save();
    await shipper.save();

    const inShop = new ShipperInBranchShopModel({
        shopId: member.shop.id,
        shipperId: shipper.id,
        totalBondAmount,
        remainBondAmount: totalBondAmount,
        bondCompanyList: _bondCompanyList,
    });
    await inShop.save();

    MediaModel._updateRef(
        { [_image]: 1 },
        ..._legalIDCard.map(item => ({ [item]: 1 })),
        ..._.flatten(_bondCompanyList.map(item => {
            const list = [...item.certificate, ...item.legalIDCard];
            return list.map(o => ({ [o]: 1 }));
        })),
    );

    actionLog(userId, 'ShopMember', shipper.id, 'Shipper', 'createAndRegisterShipper');

    return { success: true, context: {
        id: shipper.id,
        shipperType,
        shopId: member.shop.id,
        shopName: member.shop.name,
        name,
        chairMan: { name: chairMan.name, phone: chairMan.phone },
        chairManId,
        acountAmount,
        capital,
        totalBondAmount,
        remainBondAmount: totalBondAmount,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/branchShop/getShipperByChairManPhone.js

```js
import { ShopMemberModel, ShipperInBranchShopModel, ClientModel, ShipperModel } from '../../../../models';
import { _T, hasAuthority } from '../../../../utils/';
import CONSTANTS from '../../../../constants';
export default async ({
    userId,
    chairManPhone,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_CREATE_SHIPPER)) {
        return { msg: '你没有注册物流超市的权限' };
    }
    const chairMan = await ClientModel.findOne({ phone: chairManPhone });
    if (!chairMan) {
        return { msg: '必须先注册董事长账号' };
    }
    if (chairMan.shipperId) {
        const inShop = await ShipperInBranchShopModel.findOne({ shipperId: chairMan.shipperId, shopId: member.shopId });
        const shipper = await ShipperModel.findById(chairMan.shipperId);
        if (_T(shipper.chairManId) === _T(chairMan.id)) {
            if (inShop) {
                return { msg: '该物流公司已经在分店注册' };
            } else {
                const context = shipper.toObject();
                Object.assign(context, { chairMan: { id: chairMan.id, name: chairMan.name, phone: chairManPhone } });
                return { success: true, context };
            }
        } else {
            return { msg: '请使用董事长的账户进行检测' };
        }
    }
    return { success: true, context: { chairMan: { id: chairMan.id, name: chairMan.name, phone: chairManPhone } } };
};

```

* PDShopServer/project/App/routers/posts/shop/branchShop/getToExamineTruckList.js

```js
import { TruckModel, ClientModel, ShopMemberModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({ userId, phone }) => {
    const member = await ShopMemberModel.findById(userId);
    const client = await ClientModel.findOne({ phone });
    if (!client || !client.shipperId) {
        return { msg: '请确认电话号码' };
    }
    const truckList = await TruckModel.find({ shipperId: client.shipperId, shopId: member.shopId, state: CONSTANTS.TS_READY_EXAMINE })
    .sort([['modifyTime', 'desc'], ['createTime', 'desc']])
    .select({
        plateNo: 1,
        drivingLicense: 1,
        truckType: 1,
        driverId: 1,
        locationList: 1,
        insuanceMount: 1,
        state: 1,
    })
    .populate({
        path: 'driverId',
        select: { name: 1, phone: 1 },
    });

    return { success: true, context: {
        truckList,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/branchShop/registerShipper.js

```js
import { ShopMemberModel, ShipperInBranchShopModel, ShipperModel, MediaModel } from '../../../../models';
import { getMediaId, hasAuthority, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    shipperId, // 注册的物流公司
    bondCompanyList, //担保公司列表
}) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_CREATE_SHIPPER)) {
        return { msg: '你没有注册物流超市的权限' };
    }

    let inShop = await ShipperInBranchShopModel.findOne({ shipperId, shopId: member.shop.id });
    if (inShop) {
        return { msg: '该物流公司已经在分店注册' };
    }

    const shipper = await ShipperModel.findById(shipperId)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });
    if (!shipper) {
        return { msg: '没有该物流公司' };
    }

    const _bondCompanyList = _.map(bondCompanyList, item => {
        item.certificate = _.map(item.certificate, o => getMediaId(o));
        item.legalIDCard = _.map(item.legalIDCard, o => getMediaId(o));
        return item;
    });
    const totalBondAmount = _.sumBy(_bondCompanyList, o => o.bondAmount);
    inShop = new ShipperInBranchShopModel({
        shopId: member.shop.id,
        shipperId,
        totalBondAmount,
        remainBondAmount: totalBondAmount,
        bondCompanyList: _bondCompanyList,
    });
    await inShop.save();

    MediaModel._updateRef(
        ..._.flatten(_bondCompanyList.map(item => {
            const list = [...item.certificate, ...item.legalIDCard];
            return list.map(o => ({ [o]: 1 }));
        })),
    );

    actionLog(userId, 'ShopMember', shipperId, 'Shipper', 'registerShipper');

    return { success: true, context: {
        id: shipperId,
        shopId: member.shop.id,
        shopName: member.shop.name,
        name: shipper.name,
        chairMan: { name: shipper.chairMan.name, phone: shipper.chairMan.phone },
        chairManId: shipper.chairMan.id,
        acountAmount: shipper.acountAmount,
        capital: shipper.capital,
        totalBondAmount,
        remainBondAmount: totalBondAmount,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/carryPartment/getCarryList.js

```js
import { ShopMemberModel, TruckModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    pageNo,
    pageSize,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!member || !member.partmentId) {
        return { msg: '没有正在搬运的车辆' };
    }
    const criteria = { carryPartmentId: member.partmentId, state: { $gte: CONSTANTS.TS_READY_EXIT_WAREHOUSE } };
    const count = pageNo === 0 ? await TruckModel.count(criteria) : undefined;
    const query = TruckModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        plateNo: 1,
        drivingLicense: 1,
        orderCount: 1,
        carryPrice: 1,
        totalWeight: 1,
        createTime: 1,
    });
    if (!docs) {
        return { msg: '没有历史搬运信息' };
    }
    return { success: true, context: {
        count,
        truckList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/carryPartment/getCarryPartmentList.js

```js
import { ShopPartmentModel } from '../../../../models';
import { getOwnShop } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    const shopId = await getOwnShop(userId);
    const partmentList = await ShopPartmentModel.find({ shopId, type: CONSTANTS.SP_CARRY_PARTMENT });
    return { success: true, context: { partmentList } };
};

```

* PDShopServer/project/App/routers/posts/shop/carryPartment/getOrderListInTruck.js

```js
import { TruckModel } from '../../../../models';

export default async ({ userId, truckId }) => {
    const truck = await TruckModel.findById(truckId)
    .populate({
        path: 'orderList',
        select: {
            photo: 1,
            shopId: 1,
            agentId: 1,
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            name: 1,
            receiverId: 1,
            realFee: 1,
            createTime: 1,
            placeOrderTime: 1,
            weight: 1,
            size: 1,
            stateList: 1,
            totalNumbers: 1,
            startPoint: 1,
            endPoint: 1,
            proxyCharge: 1,
            needPayInsuanceFee: 1,
            payMode: 1,
            isSendDoor: 1,
            isReachPay: 1,
            totalDesignatedFee: 1,
            isInsuance: 1,
        },
        populate: [{
            path: 'shopId',
            select: { name: 1, address: 1 },
        }, {
            path: 'agentId',
            select: { name: 1, address: 1 },
        }],
    });

    return { success: true, context: {
        orderList: truck.orderList,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/carryPartment/getWaitScanTruckInfo.js

```js
import { ShopMemberModel, TruckModel } from '../../../../models';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!member || !member.partmentId) {
        return { msg: '没有待扫描的车辆' };
    }
    const query = TruckModel.find({ carryPartmentId: member.partmentId, state: CONSTANTS.TS_READY_SCAN_LOAD });
    const doc = await query
    .select({
        plateNo: 1,
        drivingLicense: 1,
        orderCount: 1,
        totalWeight: 1,
        createTime: 1,
    });
    if (!doc || doc.length == 0) {
        return { msg: '没有待扫描信息' };
    }
    return { success: true, context: doc[0] };
};

```

* PDShopServer/project/App/routers/posts/shop/carryPartment/getWorkTruckInfo.js

```js
import { ShopMemberModel, TruckModel } from '../../../../models';

export default async ({
    userId,
}) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'partmentId',
        select: { workTruckId: 1 },
    });
    if (!member || !member.partment || !member.partment.workTruckId) {
        return { msg: '没有正在搬运的车辆' };
    }
    const truck = await TruckModel.findById(member.partment.workTruckId);
    if (!truck) {
        return { msg: '没有正在搬运的车辆' };
    }
    return { success: true, context: truck };
};

```

* PDShopServer/project/App/routers/posts/shop/client/getClientDetail.js

```js
import { ClientModel } from '../../../../models';

export default async ({ userId, clientId }) => {
    const context = await ClientModel.findById(clientId);
    if (!context) {
        return { msg: '没有该线路' };
    }

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/client/getClientList.js

```js
import { ClientModel } from '../../../../models';
import { getKeywordCriteriaForUser } from '../../../../utils';

export default async ({ userId, keyword, fromPC, pageNo, pageSize }) => {
    const criteria = getKeywordCriteriaForUser(keyword);
    const count = (fromPC && pageNo === 0) ? await ClientModel.count(criteria) : undefined;
    const query = ClientModel.find(criteria).sort({ modifyTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        phone: 1,
        name: 1,
        email: 1,
        head: 1,
        address: 1,
        phoneList: 1,
    });

    return { success: true, context: {
        count,
        clientList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/masterShop/createBranchShop.js

```js
import { ShopMemberModel, ShopModel, MediaModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _T, getMediaId, registerUser, hasAuthority, getDefaultPassWord, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { registeredStartPointAddress } from '../../libs/address';

export default async ({
    userId,
    chairManPhone, // 分店董事长登录电话
    chairManName,  // 分店董事长姓名
    chairManPassword, // 分店董事长登录密码（不传入该参数，默认为手机号码后6位）
    name, // 分店名称
    image, // 分店背景图片
    sign, // 分店签名
    phoneList, // 分店联系电话
    addressRegion, // 物流超市所在城市
    addressRegionLastCode, // 物流超市所在城市的lastCode
    address, // 分店地址
    location, // 经纬度定位
    profitRate, // 分层比例
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_CREATE_BRANCH_SHOP)) {
        return { msg: '你没有创建分店的权限' };
    }
    if (await ShopModel.findOne({ name })) {
        return { msg: '该分店名已经被使用' };
    }
    // 注册董事长账号
    const chairMan = new ShopMemberModel({
        phone: chairManPhone,
        name: chairManName,
        post: CONSTANTS.PO_CHAIR_MAN,
        authority: [
            // 物流公司
            CONSTANTS.AH_CREATE_SHIPPER, // 创建物流公司
            CONSTANTS.AH_MODIFY_SHIPPER, // 修改物流公司
            CONSTANTS.AH_REMOVE_SHIPPER, // 删除物流公司
            CONSTANTS.AH_LOOK_SHIPPER, // 查看物流公司
            // 部门
            CONSTANTS.AH_CREATE_PARTMENT, // 创建部门的权限
            CONSTANTS.AH_MODIFY_PARTMENT, // 修改部门信息的权限
            CONSTANTS.AH_REMOVE_PARTMENT, // 删除部门的权限
            CONSTANTS.AH_LOOK_PARTMENT, // 查看部门的权限
            // 路线
            CONSTANTS.AH_LOOK_ROADMAP, // 查看路线的权限
            CONSTANTS.AH_TO_EXAMINE_TRUCK, // 审核货车
            // 公共权限
            CONSTANTS.AH_MODIFY_OWN_SHOP, // 修改自身分店的权限
            // 成员
            CONSTANTS.AH_CREATE_MEMBER, // 创建成员的权限
            CONSTANTS.AH_MODIFY_MEMBER, // 修改成员信息的权限
            CONSTANTS.AH_REMOVE_MEMBER, // 删除成员的权限
            CONSTANTS.AH_LOOK_MEMBER, // 查看成员的权限
            // 仓库
            CONSTANTS.AH_CREATE_WAREHOUSE, // 创建仓库的权限
            CONSTANTS.AH_MODIFY_WAREHOUSE, // 修改仓库信息的权限
            CONSTANTS.AH_REMOVE_WAREHOUSE, // 删除仓库的权限
            CONSTANTS.AH_LOOK_WAREHOUSE, // 查看仓库的权限
            // 提现，余额
            CONSTANTS.AH_WITHDRAW, // 提现的权限
            CONSTANTS.AH_LOOK_AMOUNT, // 查看余额的权限
            // 统计
            CONSTANTS.AH_LOOK_STATISTICS, // 查看统计信息的权限
            CONSTANTS.AH_LOOK_ORDERS, // 查看综合货单的权限
        ],
    });

    // 创建总部
    let _image = getMediaId(image);
    const shop = new ShopModel({
        chairManId: chairMan.id,
        name,
        image: _image,
        sign,
        phoneList,
        addressRegion,
        addressRegionLastCode,
        address,
        profitRate,
        location,
    });
    chairMan.shopId = shop.id;

    // 保存
    const error = await registerUser(ShopMemberModel, chairMan, getDefaultPassWord(chairManPhone, chairManPassword));
    if (error === 'UserExistsError') {
        return { msg: '分店总经理账号已经被占用' };
    } else if (error) {
        return { msg: '分店总经理账号注册失败' };
    }

    // 创建分店账号
    const account = await AccountModel.create({ targetId: _T(shop.id), type: CONSTANTS.AT_BRANCH_SHOP });
    if (!account) {
        return { msg: '创建分店账号失败' };
    }
    await shop.save();
    await registeredStartPointAddress(addressRegionLastCode); // 将分店的地址进行注册

    actionLog(userId, 'ShopMember', shop.id, 'Shop', 'createBranchShop');
    MediaModel._updateRef(
        { [_image]: 1 },
    );
    const context = await ShopModel.findById(shop.id)
    .select({
        name: 1,
        addressRegion: 1,
        addressRegionLastCode: 1,
        address: 1,
        location: 1,
        profitRate: 1,
        phoneList: 1,
        chairManId: 1,
    })
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/masterShop/getBranchShopDetail.js

```js
import { ShopModel } from '../../../../models';
import { hasAuthority } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, shopId }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_LOOK_BRANCH_SHOP)) {
        return { msg: '你没有查看分店的权限' };
    }

    const shop = await ShopModel.findById(shopId)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });
    if (!shop) {
        return { msg: '没有该物流超市' };
    }
    return { success: true, context: shop };
};

```

* PDShopServer/project/App/routers/posts/shop/masterShop/getBranchShopList.js

```js
import { ShopModel } from '../../../../models';
import { hasAuthority } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_LOOK_BRANCH_SHOP)) {
        return { msg: '你没有查看分店的权限' };
    }

    const query = ShopModel.find({ isMasterShop: false }).sort({ createTime: 'desc' });
    const docs = await query
    .select({
        name: 1,
        addressRegion: 1,
        addressRegionLastCode: 1,
        address: 1,
        location: 1,
        profitRate: 1,
        phoneList: 1,
        chairManId: 1,
    })
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });

    return { success: true, context: {
        branchShopList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/masterShop/getSettingInfo.js

```js
import { SettingModel } from '../../../../models';
import { hasAuthority } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
}) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_SETTING)) {
        return { msg: '权限不足' };
    }
    const doc = await SettingModel.findOne();
    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shop/masterShop/modifyBranchShop.js

```js
import { ShopModel, ShopMemberModel, MediaModel } from '../../../../models';
import { getMediaId, omitNil, hasAuthority, actionLog } from '../../../../utils/';
import { registeredStartPointAddress } from '../../libs/address';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    shopId,
    name, // 物流超市名称
    image, // 物流超市背景图片
    sign, // 物流超市签名
    phoneList, // 物流超市联系电话
    addressRegion, // 物流超市所在城市
    addressRegionLastCode, // 物流超市所在城市的lastCode
    address, // 商铺地址
    location, // 经纬度定位
    profitRate, // 分层比例
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_BRANCH_SHOP)) {
        return { msg: '你没有修改分店的权限' };
    }
    if (name && await ShopModel.findOne({ name, _id: { $ne: shopId } })) {
        return { msg: '该分店名已经被使用' };
    }

    let _image = getMediaId(image);
    const doc = await ShopModel.findByIdAndUpdate(shopId, omitNil({
        name,
        image: _image,
        sign,
        phoneList,
        addressRegion,
        addressRegionLastCode,
        address,
        location,
        profitRate,
    }));
    if (!doc) {
        return { msg: '修改失败' };
    }
    await registeredStartPointAddress(addressRegionLastCode, doc.addressRegionLastCode); // 将分店的地址进行注册

    MediaModel._updateRef(
        { [_image]: 1 },
        { [image && doc.image]: -1 },
    );

    actionLog(userId, 'ShopMember', shopId, 'Shop', 'modifyBranchShop');

    const context = await ShopModel.findById(shopId)
    .select({
        name: 1,
        address: 1,
        addressRegion: 1,
        addressRegionLastCode: 1,
        location: 1,
        profitRate: 1,
        phoneList: 1,
        chairManId: 1,
    })
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/masterShop/modifySettingInfo.js

```js
import { SettingModel } from '../../../../models';
import { settingMgr } from '../../../../manager';
import { hasAuthority, omitNil, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    sizeWeightRate,
    insuanceRate,
    insuanceMountRate,
    insuanceBaseValue,
    proxyChargeProfitRate,
    gradientPriceList,
    additionalDeliverTime,
    noticeShipperStorageWeight,
    bondAmountWeightRate,
    rankedMaskFirstRankWeight,
    rankedMaskRateList,
}) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_SETTING)) {
        return { msg: '你没有修改配置的权限' };
    }
    const doc = await SettingModel.findOneAndUpdate({}, omitNil({
        sizeWeightRate,
        insuanceRate,
        insuanceMountRate,
        insuanceBaseValue,
        proxyChargeProfitRate,
        gradientPriceList,
        additionalDeliverTime,
        noticeShipperStorageWeight,
        bondAmountWeightRate,
        rankedMaskFirstRankWeight,
        rankedMaskRateList,
    }), { new: true });
    settingMgr.set(doc);

    actionLog(userId, 'ShopMember', JSON.stringify(omitNil({
        sizeWeightRate,
        insuanceRate,
        insuanceMountRate,
        insuanceBaseValue,
        proxyChargeProfitRate,
        gradientPriceList,
        additionalDeliverTime,
        noticeShipperStorageWeight,
        bondAmountWeightRate,
        rankedMaskFirstRankWeight,
        rankedMaskRateList,
    })), 'Data', 'modifySettingInfo');

    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shop/masterShop/removeBranchShop.js

```js
import { ShopMemberModel, ShopPartmentModel, ShopModel, MediaModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

// 这个接口是硬删除，千万谨慎使用，最好不要对外提供，如果要对外提供，可以提供软删除，添加删除标记，这种软删除是可以恢复的
export default async ({
    userId,
    shopId,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_REMOVE_BRANCH_SHOP)) {
        return { msg: '权限不足' };
    }

    const doc = await ShopModel.findByIdAndRemove(shopId);
    if (doc) {
        await ShopPartmentModel.remove({ shopId }); // 删除和分店的所有部门
        await ShopMemberModel.remove({ shopId }); // 删除和分店的所有人
        // todo 删除和其他所有和分店相关的东西
        MediaModel._updateRef(
            { [doc.image]: -1 },
        );
    }

    actionLog(userId, 'ShopMember', shopId, 'Shop', 'removeBranchShop');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/member/createMember.js

```js
import { ShopMemberModel, ShopPartmentModel } from '../../../../models';
import { registerUser, hasAuthority, actionLog, getDefaultPassWord } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId,
    name,
    post,
    phone,
    password,
    authority,
}) => {
    if (authority) {
        authority = _.sortBy(_.uniq(authority));
    }
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_CREATE_MEMBER)) {
        return { msg: '你没有创建成员的权限' };
    }
    const doc = new ShopMemberModel({
        shopId: member.shopId,
        partmentId: member.partmentId,
        name,
        post,
        phone,
        authority,
    });
    const error = await registerUser(ShopMemberModel, doc, getDefaultPassWord(phone, password));
    if (error === 'UserExistsError') {
        return { msg: '该电话号码已经被人使用' };
    } else if (error) {
        return { msg: '创建失败' };
    }
    await ShopPartmentModel.findByIdAndUpdate(member.partmentId, { $inc: { memberCount: 1 } });

    let context = await ShopMemberModel.findById(doc.id)
    .select({
        phone: 1,
        email: 1,
        name: 1,
        head: 1,
        post: 1,
        authority: 1,
        phoneList: 1,
    });

    actionLog(userId, 'ShopMember', doc.id, 'ShopMember', 'createMember');
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/member/getMemberDetail.js

```js
import { ShopMemberModel } from '../../../../models';

export default async ({
    userId,
    memberId,
}) => {
    const doc = await ShopMemberModel.findById(memberId);
    if (!doc) {
        return { msg: '没有该成员' };
    }
    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shop/member/getMemberList.js

```js
import { ShopMemberModel } from '../../../../models';
import { getKeywordCriteriaForUser } from '../../../../utils';

export default async ({ userId, keyword, fromPC, pageNo, pageSize }) => {
    const member = await ShopMemberModel.findById(userId);
    const criteria = { ...getKeywordCriteriaForUser(keyword, { shopId: member.shopId }), partmentId: member.partmentId };
    const count = (fromPC && pageNo === 0) ? await ShopMemberModel.count(criteria) : undefined;
    const query = ShopMemberModel.find(criteria).sort({ registerTime: 'asc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        phone: 1,
        email: 1,
        name: 1,
        head: 1,
        post: 1,
        partmentId: 1,
        authority: 1,
        phoneList: 1,
    }).populate({
        path: 'partmentId',
        select: { name: 1 },
    });

    return { success: true, context: {
        count,
        memberList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/member/getWarehouseMemberList.js

```js
import { ShopMemberModel, ShopPartmentModel } from '../../../../models';
import { getKeywordCriteriaForUser } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, keyword, fromPC, pageNo, pageSize }) => {
    const member = await ShopMemberModel.findById(userId);
    const partmentList = await ShopPartmentModel.find({ shopId: member.shopId, type: CONSTANTS.SP_WARE_HOUSE_PARTMENT });
    const criteria = getKeywordCriteriaForUser(keyword, { shopId: member.shopId, partmentId: { $in: partmentList.map(o => o.id) } });
    const count = (fromPC && pageNo === 0) ? await ShopMemberModel.count(criteria) : undefined;
    const query = ShopMemberModel.find(criteria).sort({ registerTime: 'asc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        phone: 1,
        email: 1,
        name: 1,
        head: 1,
        post: 1,
        partmentId: 1,
        authority: 1,
        phoneList: 1,
    }).populate({
        path: 'partmentId',
        select: { name: 1 },
    });

    return { success: true, context: {
        count: (count || docs.length),
        memberList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/member/modifyMember.js

```js
import { ShopMemberModel } from '../../../../models';
import { omitNil, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default ({
    userId,
    memberId,
    name,
    post,
    phone,
    authority,
}) => {
    if (authority) {
        authority = _.sortBy(_.uniq(authority));
    }
    return new Promise(async resolve => {
        if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_MEMBER)) {
            return resolve({ msg: '你没有修改成员信息的权限' });
        }
        const doc = await ShopMemberModel.findByIdAndUpdate(memberId, omitNil({
            name,
            phone,
            post,
            authority,
        }), { new: true }).select({
            phone: 1,
            email: 1,
            name: 1,
            head: 1,
            post: 1,
            authority: 1,
            phoneList: 1,
        }).catch((err) => {
            return resolve({ msg: '登录电话号码(' + phone + ')已经被其他人使用' });
        });

        if (!doc) {
            return resolve({ msg: '修改失败' });
        }

        actionLog(userId, 'ShopMember', memberId, 'ShopMember', 'modifyMember');

        return resolve({ success: true, context: doc });
    });
};

```

* PDShopServer/project/App/routers/posts/shop/member/removeMember.js

```js
import { ShopMemberModel, ShopPartmentModel } from '../../../../models';
import { hasAuthority, authenticatePassword, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    memberId,
    password,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_REMOVE_MEMBER)) {
        return { msg: '你没有删除成员的权限' };
    }
    const error = await authenticatePassword(member, password);
    if (error) {
        return { msg: '密码错误' };
    }

    await ShopMemberModel.findByIdAndRemove(memberId);
    await ShopPartmentModel.findByIdAndUpdate(member.partmentId, { $inc: { memberCount: -1 } });

    actionLog(userId, 'ShopMember', memberId, 'ShopMember', 'removeMember');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/order/getOrderDetail.js

```js
import { OrderModel, ShopMemberModel } from '../../../../models';

export default async ({ userId, orderId, isScan }) => {
    const member = await ShopMemberModel.findById(userId);
    if (!member) {
        return { msg: '没有该用户' };
    }
    if (isScan) {
        const currentOrder = await OrderModel.getOrderFromOriginOrder(member.shopId, orderId);
        if (!currentOrder) {
            return { msg: '没有该货单' };
        }
        orderId = currentOrder.id;
    }
    const order = await OrderModel.findById(orderId)
    .populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });
    if (!order) {
        return { msg: '没有该货单' };
    }

    const context = order.toObject();
    context.state = context.stateList[0].state;
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/order/getOrders.js

```js
import _ from 'lodash';
import { OrderModel, ShopMemberModel } from '../../../../models';
import { getKeywordCriteriaForOrder, omitNil, hasAuthority } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import moment from 'moment';

async function getPageData (shopId, type, params) {
    let { keyword, startDate, endDate, fromPC, pageNo, pageSize } = params;
    let criteria = {};
    if (type === 'toprintbarcode') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_PRINT_BAR_CODE };
    } else if (type === 'toselectroadmap') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_SELECT_ROADMAP };
    } else if (type === 'topay') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_PAYMENT };
    } else if (type === 'toprintbill') {
        criteria = { 'stateList.state': { $in: [ CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER, CONSTANTS.OS_READY_PRINT_ORDER ] } };
    } else if (type === 'tostore') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_STOCK };
    } else if (type === 'stored') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_LOAD };
    } else if (type === 'tostartoff') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_START_OFF };
    } else if (type === 'onway') {
        criteria = { 'stateList.state': CONSTANTS.OS_ON_THE_WAY };
    } else if (type === 'success') {
        criteria = { 'stateList.state': CONSTANTS.OS_RECEIVE_SUCCESS };
    }
    if (fromPC) {
        if (startDate && endDate) {
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        } else {
            startDate = endDate = moment().format('YYYY-MM-DD');
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        }
    }
    criteria = getKeywordCriteriaForOrder(keyword, omitNil({ ...criteria, shopId }));

    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        shopId: 1,
        shipperId: 1,
        senderName: 1,
        senderPhone: 1,
        receiverName: 1,
        receiverPhone: 1,
        name: 1,
        endPoint: 1,
        sendDoorEndPoint: 1,
        createTime: 1,
        modifyTime: 1,
        isReachPay: 1,
        payMode: 1,
        receiveMemberId: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        stateList: 1,
        isSendDoor: 1,
        isClientPick: 1,
        isInsuance: 1,
        insuanceMount: 1,
        insuanceFee: 1,
        fee: 1,
        profit: 1,
        masterProfit: 1,
        branchProfit: 1,
        realFee: 1,
        proxyCharge: 1,
        proxyChargeProfit: 1,
        needPayTransportFee: 1,
        needPayInsuanceFee: 1,
        totalDesignatedFee: 1,
        designatedFee: 1,
        placeOrderTime: 1,
    }).populate({
        path: 'receiveMemberId',
        select: { name: 1, phone: 1 },
    }).populate({
        path: 'shipperId',
        select: { name: 1 },
    }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });

    return {
        count,
        list: docs,
    };
};

// ['待打印二维码', '待选路线', '待支付', '待打印货单', '待入库', '已入库', '待出发', '运输中', '成功']
const TYPES = ['toprintbarcode', 'toselectroadmap', 'topay', 'toprintbill', 'tostore', 'stored', 'tostartoff', 'onway', 'success'];
export default async ({ userId, shopId, type, ...params }) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { isMasterShop: 1 },
    });
    if (member.partmentId || !hasAuthority(member, CONSTANTS.AH_LOOK_ORDERS)) {
        return { msg: '你没有查看货单的权限' };
    }
    if (!member.shop.isMasterShop) {
        shopId = member.shop.id;
    }

    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(shopId, type, params),
            },
        };
    } else {
        return {
            success: true,
            context: {
                toprintbarcode: await getPageData(shopId, 'toprintbarcode', params),
                toselectroadmap: await getPageData(shopId, 'toselectroadmap', params),
                topay: await getPageData(shopId, 'topay', params),
                toprintbill: await getPageData(shopId, 'toprintbill', params),
                tostore: await getPageData(shopId, 'tostore', params),
                stored: await getPageData(shopId, 'stored', params),
                tostartoff: await getPageData(shopId, 'tostartoff', params),
                onway: await getPageData(shopId, 'onway', params),
                success: await getPageData(shopId, 'success', params),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/shop/personal/findPassword.js

```js
import { ShopMemberModel } from '../../../../models';
import { setPassword, checkVerifyCode, actionLog } from '../../../../utils';

export default async ({ phone, password, verifyCode }) => {
    const user = await ShopMemberModel.findOne({ phone });
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    const success = await checkVerifyCode(phone, verifyCode);
    if (!success) {
        return { msg: '无效验证码' };
    }

    const error = await setPassword(user, password);
    if (error) {
        return { msg: '服务器错误' };
    }
    await user.save();

    actionLog(user.id, 'ShopMember', user.id, 'ShopMember', 'findPassword');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/personal/getPersonalInfo.js

```js
import { ShopMemberModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import CONSTANTS from '../../../../constants';
import { _T, _Y } from '../../../../utils/';

export default async ({ userId, fromPC }) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'partmentId',
        populate: [{
            path: 'chargeManId',
            select: { name: 1, phone: 1 },
        }],
    })
    .populate({
        path: 'shopId',
        select: { isMasterShop: 1, chairManId: 1 },
        populate: [{
            path: 'chairManId',
            select: { name: 1, phone: 1 },
        }],
    });
    if (!member) {
        return { msg: '没有该用户' };
    }
    const context = member.toObject();
    context.userId = context.id;
    delete context.id;

    if (member.shop.isMasterShop) {
        const masterAccount = await AccountModel.findOne({ where: { id: CONSTANTS.AC_MASTER_ACCOUNT } });
        if (!masterAccount) {
            return { msg: '没有该账户' };
        }
        context.remainAmount = _Y(masterAccount.amount);
    } else if (member.partmentId) {
        const partmentAccount = await AccountModel.findOne({ where: { targetId: _T(member.partment.id), type: CONSTANTS.AT_BRANCH_SHOP_PARTMENT } });
        if (!partmentAccount) {
            return { msg: '没有该账户' };
        }
        context.remainAmount = _Y(partmentAccount.amount);
    } else {
        const branchAccount = await AccountModel.findOne({ where: { targetId: _T(member.shop.id), type: CONSTANTS.AT_BRANCH_SHOP } });
        if (!branchAccount) {
            return { msg: '没有该账户' };
        }
        context.remainAmount = _Y(branchAccount.amount);
    }

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/personal/login.js

```js
import { ShopMemberModel } from '../../../../models';
import { authenticatePassword, actionLog } from '../../../../utils';

export default async ({ phone, password }) => {
    const user = await ShopMemberModel.findOne({ phone });
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    const error = await authenticatePassword(user, password);
    if (error) {
        return { msg: '密码错误' };
    } else {
        actionLog(user.id, 'ShopMember', user.id, 'ShopMember', 'login');
        return { success: true, context: { userId: user.id, shopId: user.shopId } };
    }
};

```

* PDShopServer/project/App/routers/posts/shop/personal/modifyPassword.js

```js
import { ShopMemberModel } from '../../../../models';
import { setPassword, authenticatePassword, actionLog } from '../../../../utils';

export default async ({ userId, oldPassword, newPassword }) => {
    const user = await ShopMemberModel.findById(userId);
    if (!user) {
        return { msg: '该电话号码没有注册' };
    }
    let error = await authenticatePassword(user, oldPassword);
    if (error) {
        return { msg: '密码错误' };
    }
    error = await setPassword(user, newPassword);
    if (error) {
        return { msg: '设置密码失败' };
    }
    await user.save();

    actionLog(userId, 'ShopMember', userId, 'ShopMember', 'modifyPassword');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/personal/modifyPersonalInfo.js

```js
import { ShopMemberModel, MediaModel } from '../../../../models';
import { getMediaId, omitNil, getAgeFromBirthday, actionLog } from '../../../../utils';

export default async ({
    userId,
    name,
    head,
    email,
    sex,
    birthday,
    address,
    phoneList,
}) => {
    let _head = getMediaId(head);
    const doc = await ShopMemberModel.findByIdAndUpdate(userId, omitNil({
        name,
        head: _head,
        email,
        sex,
        birthday,
        address,
        phoneList,
    }));
    if (!doc) {
        return { msg: '修改失败' };
    }
    MediaModel._updateRef(
        { [_head]: 1 },
        { [head && doc.head]: -1 },
    );

    let context = await ShopMemberModel.findById(userId);
    context = context.toObject();
    context.userId = context.id;
    delete context.id;

    actionLog(userId, 'ShopMember', userId, 'ShopMember', 'modifyPersonalInfo');

    return { success: true, context};
};

```

* PDShopServer/project/App/routers/posts/shop/personal/submitFeedback.js

```js
import { FeedbackModel } from '../../../../models';
import { actionLog } from '../../../../utils';

export default async ({ userId, content, email }) => {
    const feedback = new FeedbackModel({ shopMemberId: userId, content, email });
    await feedback.save();

    actionLog(userId, 'ShopMember', userId, 'ShopMember', 'submitFeedback');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/getLastestOrder.js

```js
import { OrderModel } from '../../../../models';
import { getRPLatestOrder } from '../../libs/order';

export default async ({ userId }) => {
    let order = await getRPLatestOrder(userId);
    if (!order) {
        return { msg: '暂无货单' };
    }

    order = await OrderModel.findById(order.id)
    .populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });

    const context = order.toObject();
    context.state = context.stateList[0].state;
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/getOrders.js

```js
import _ from 'lodash';
import { OrderModel } from '../../../../models';
import { getKeywordCriteriaForOrder } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import moment from 'moment';

async function getPageData (userId, type, params) {
    let { keyword, startDate, endDate, fromPC, pageNo, pageSize } = params;
    let criteria = {};
    if (type === 'toselectroadmap') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_SELECT_ROADMAP };
    } else if (type === 'topay') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_PAYMENT };
    } else if (type === 'toprintbill') {
        criteria = { 'stateList.state': { $in: [ CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER, CONSTANTS.OS_READY_PRINT_ORDER ] } };
    } else if (type === 'tostore') {
        criteria = { 'stateList.state': CONSTANTS.OS_READY_STOCK };
    } else if (type === 'stored') {
        criteria = { 'stateList.state': { $gte: CONSTANTS.OS_READY_LOAD } };
    }
    if (fromPC) {
        if (startDate && endDate) {
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        } else {
            startDate = endDate = moment().format('YYYY-MM-DD');
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        }
    }
    criteria = getKeywordCriteriaForOrder(keyword, { ...criteria, receiveMemberId: userId });
    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        senderName: 1,
        senderPhone: 1,
        receiverName: 1,
        receiverPhone: 1,
        name: 1,
        photo: 1,
        shopId: 1,
        shipperId: 1,
        isCityDistribute: 1,
        endPoint: 1,
        sendDoorEndPoint: 1,
        isSendDoor: 1,
        isClientPick: 1,
        createTime: 1,
        placeOrderTime: 1,
        modifyTime: 1,
        payMode: 1,
        receiveMemberId: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        stateList: 1,
        warehouse: 1,
        needPayTransportFee: 1,
        needPayInsuanceFee: 1,
        totalDesignatedFee: 1,
        proxyCharge: 1,
    }).populate({
        path: 'receiveMemberId',
        select: { name: 1, phone: 1 },
    }).populate({
        path: 'shipperId',
        select: { name: 1 },
    }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });

    return {
        count,
        list: docs,
    };
};

// ['待选路线', '待支付', '待打印货单', '待入库', '已入库']
const TYPES = ['toselectroadmap', 'topay', 'toprintbill', 'tostore', 'stored'];
export default async ({ userId, type, ...params }) => {
    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(userId, type, params),
            },
        };
    } else {
        return {
            success: true,
            context: {
                toselectroadmap: await getPageData(userId, 'toselectroadmap', params),
                topay: await getPageData(userId, 'topay', params),
                toprintbill: await getPageData(userId, 'toprintbill', params),
                tostore: await getPageData(userId, 'tostore', params),
                stored: await getPageData(userId, 'stored', params),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/getPreOrderList.js

```js
import { OrderModel, ClientModel } from '../../../../models';
import { getKeywordCriteriaForOrder } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, senderPhone, keyword, fromPC, pageNo, pageSize }) => {
    const client = await ClientModel.findOne({ phone: senderPhone });
    if (!client) {
        return { msg: '没有该用户' };
    }
    const criteria = getKeywordCriteriaForOrder(keyword, { senderId: client.id, 'stateList.state': CONSTANTS.OS_PREORDER });

    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const docs = await OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);

    return { success: true, context: {
        count,
        orderList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/getRegionAddressWithOrder.js

```js
import { OrderModel, ShopMemberModel } from '../../../../models';
import { getRPLatestOrder } from '../../libs/order';
import { getEndPointFromLastCode, getSendDoorAddressFromLastCode } from '../../libs/address';

export default async ({
    userId,
    orderId,
}) => {
    const member = await ShopMemberModel.findById(userId).populate({ path: 'shopId', select: { addressRegionLastCode: 1 } });
    const { addressRegionLastCode, id } = (member || {}).shop || {};

    let order;
    if (orderId) {
        order = await OrderModel.findById(orderId);
    } else {
        order = await getRPLatestOrder(userId);
    }

    let addressList = [];
    let cityAddressList = [];

    if (order) {
        if (!order.isCityDistribute) {
            addressList = await getEndPointFromLastCode(id, order.endPointLastCode);
            cityAddressList = await getSendDoorAddressFromLastCode(0, addressRegionLastCode);
        } else {
            addressList = await getEndPointFromLastCode(id, 0);
            cityAddressList = await getSendDoorAddressFromLastCode(order.endPointLastCode);
        }
    } else {
        addressList = await getEndPointFromLastCode(id, 0);
        cityAddressList = await getSendDoorAddressFromLastCode(0, addressRegionLastCode);
    }

    return { success: true, context: {
        addressList,
        cityAddressList,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/getRegionSendDoorAddressWithOrder.js

```js
import { OrderModel } from '../../../../models';
import { getRPLatestOrder } from '../../libs/order';
import { getSendDoorAddressFromLastCode } from '../../libs/address';

export default async ({
    userId,
    orderId,
}) => {
    let order;
    if (orderId) {
        order = await OrderModel.findById(orderId);
    } else {
        order = await getRPLatestOrder(userId);
    }

    const addressList = await getSendDoorAddressFromLastCode((order || {}).sendDoorEndPointLastCode);

    return { success: true, context: {
        addressList,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/getRoadmapListWithOrder.js

```js
import { OrderModel } from '../../../../models';
import { searchRoadmapListByReceivePartment, getRegionProfitRateByEndPoint } from '../../libs/roadmap';

export default async ({
    userId,
    fromPC,
    pageNo,
    pageSize,
    orderId,
    orderBy, //排序方式, 0：以价格优先， 1：以时长优先
}) => {
    const order = await OrderModel.findById(orderId);
    if (!order) {
        return { msg: '没有该货单' };
    }
    const {
        endPointLastCode, // 终点
        sendDoorEndPointLastCode, // 送货上门终点
        isSendDoor, // 是否送货上门
        weight, // 重量
        size, // 方量
        proxyCharge, // 代收货款金额
        isCityDistribute, // 是否同城配送
        isReachPay, // 是否是到付
        totalDesignatedFee, //指定向收货人应该收多少钱
    } = order;
    const regionProfitRate = await getRegionProfitRateByEndPoint(order.shopId, endPointLastCode, isCityDistribute);
    const context = await searchRoadmapListByReceivePartment({
        shopId: order.shopId,
        fromPC,
        pageNo,
        pageSize,
        endPointLastCode,
        sendDoorEndPointLastCode,
        isSendDoor,
        weight,
        size,
        proxyCharge,
        isReachPay,
        totalDesignatedFee,
        orderBy,
        regionProfitRate,
    });
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/modifyOriginOrder.js

```js
import { ShopMemberModel, OrderModel, ClientModel } from '../../../../models';
import { _T, hasAuthority, actionLog, omitNil } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { setting } from '../../../../manager';

export default async ({
    userId, // 收货部收货员
    orderId,
    senderPhone,
    senderName,
    receiverPhone,
    receiverName,
    name,
    isCityDistribute, // 是否是同城配送
    endPoint, // 终点，精确到镇级别
    endPointLastCode, // 终点镇级代码
    sendDoorEndPoint, // 送货上门地址，精确到镇级别
    sendDoorEndPointLastCode, // 送货上门地址镇级代码
    totalNumbers, // 一票货总的件数
    weight, // 一票货总的重量
    size, // 一票货总的方量
    isSendDoor, // 是否送货上门
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee, // 指定向收货人应该收多少钱
    proxyCharge, // 代收货款金额
    isInsuance, //是否保价
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '你没有收货的权限' };
    }

    let senderId, sender;
    if (senderPhone) {
        senderId = await ClientModel.checkExistElseCreate(senderPhone);
        if (!senderId) {
            return { msg: '系统创建账号错误，请重试' };
        }
        sender = await ClientModel.findById(senderId)
        .populate({
            path: 'shipperId',
            select: { chairManId: 1 },
        });
    }
    let receiverId;
    if (receiverPhone) {
        receiverId = await ClientModel.checkExistElseCreate(receiverPhone);
        if (!receiverId) {
            return { msg: '系统创建账号错误，请重试' };
        }
    }

    const order = await OrderModel.findById(orderId);
    if (!order) {
        return { msg: '没有该货单' };
    }
    const state = order.stateList[0].state;
    await order.update(omitNil({
        senderId,
        senderPhone,
        senderName,
        isSenderRepresentShipper: sender && sender.shipperId && _T(sender.shipper.chairManId) === sender.id,
        receiverId,
        receiverPhone,
        receiverName,
        name,
        isCityDistribute,
        endPoint,
        endPointLastCode,
        sendDoorEndPoint,
        sendDoorEndPointLastCode,
        totalNumbers,
        weight,
        size,
        isSendDoor,
        proxyCharge,
        proxyChargeProfit: proxyCharge !== undefined ? Math.ceil(proxyCharge * setting.proxyChargeProfitRate) : undefined,
        isReachPay,
        payMode: isReachPay !== undefined ? (isReachPay ? CONSTANTS.PM_REACH : CONSTANTS.PM_IMMEDIATE) : undefined,
        totalDesignatedFee,
        isInsuance,
        'stateList.0.state': state === CONSTANTS.OS_READY_PRINT_BAR_CODE ? CONSTANTS.OS_READY_PRINT_BAR_CODE : CONSTANTS.OS_READY_SELECT_ROADMAP,
        'stateList.0.count': totalNumbers || order.totalNumbers,
        modifyTime: Date.now(),
    }));

    let context = await OrderModel.findById(orderId);
    context = context.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'ShopMember', orderId, 'Order', 'modifyOriginOrder');
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/placeOriginOrder.js

```js
import { ShopMemberModel, OrderModel, ClientModel } from '../../../../models';
import { _T, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { setting } from '../../../../manager';

export default async ({
    userId, // 收货部收货员
    senderPhone,
    senderName,
    receiverPhone,
    receiverName,
    name,
    isCityDistribute, // 是否是同城配送
    endPoint, // 终点，精确到镇级别
    endPointLastCode = 0, // 终点镇级代码
    sendDoorEndPoint, // 送货上门地址，精确到镇级别
    sendDoorEndPointLastCode = 0, // 送货上门地址镇级代码
    totalNumbers, // 一票货总的件数
    weight, // 一票货总的重量
    size, // 一票货总的方量
    isSendDoor, // 是否送货上门
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee = 0, // 指定向收货人应该收多少钱
    proxyCharge = 0, // 代收货款金额
    isInsuance, //是否保价
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '你没有收货的权限' };
    }

    if (isSendDoor && !sendDoorEndPointLastCode) {
        return { msg: '需要送货上门必须填写送货上门的地址' };
    }
    if (!isReachPay && totalDesignatedFee) {
        return { msg: '现付的情况下不能有指定收款' };
    }

    // 如果没有发送者或者接收者，就创建一个
    const senderId = await ClientModel.checkExistElseCreate(senderPhone);
    const receiverId = await ClientModel.checkExistElseCreate(receiverPhone);
    if (!senderId || !receiverId) {
        return { msg: '系统创建账号错误，请重试' };
    }

    const sender = await ClientModel.findById(senderId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    });

    const order = new OrderModel({
        shopId: member.shopId,
        senderId,
        senderName,
        senderPhone,
        isSenderRepresentShipper: sender.shipperId && _T(sender.shipper.chairManId) === _T(sender.id),
        receiverId,
        receiverPhone,
        receiverName,
        name,
        isCityDistribute,
        endPoint,
        endPointLastCode,
        sendDoorEndPoint,
        sendDoorEndPointLastCode,
        receivePartmentId: member.partmentId,
        receiveMemberId: userId,
        totalNumbers,
        weight,
        size,
        isSendDoor,
        proxyCharge: Math.floor(proxyCharge),
        proxyChargeProfit: Math.floor(proxyCharge * setting.proxyChargeProfitRate),
        isReachPay,
        payMode: isReachPay ? CONSTANTS.PM_REACH : CONSTANTS.PM_IMMEDIATE,
        totalDesignatedFee: Math.floor(totalDesignatedFee),
        isInsuance,
        stateList: [ { state: CONSTANTS.OS_READY_PRINT_BAR_CODE, count: totalNumbers } ],
        placeOrderTime: Date.now(),
    });
    await order.save();

    const context = order.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'ShopMember', order.id, 'Order', 'placeOriginOrder');
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/placeOriginOrderWithPreOrder.js

```js
import { ShopMemberModel, OrderModel, ClientModel, OrderGroupModel } from '../../../../models';
import { _T, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { setting } from '../../../../manager';

export default async ({
    userId, // 收货部收货员
    preOrderId, // 预下单的货单号
    senderPhone,
    senderName,
    receiverPhone,
    receiverName,
    name,
    isCityDistribute, // 是否是同城配送
    endPoint, // 终点，精确到镇级别
    endPointLastCode = 0, // 终点镇级代码
    sendDoorEndPoint, // 送货上门地址，精确到镇级别
    sendDoorEndPointLastCode = 0, // 送货上门地址镇级代码
    totalNumbers, // 一票货总的件数
    weight, // 一票货总的重量
    size, // 一票货总的方量
    isSendDoor, // 是否送货上门
    isReachPay, // 是否是到付 (如果是到付，并且设置了totalDesignatedFee，为指定向收货人收totalDesignatedFee的运费，否则向收货人收初始单计算出来的运费)
    totalDesignatedFee = 0, // 指定向收货人应该收多少钱
    proxyCharge = 0, // 代收货款金额
    isInsuance, //是否保价
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '你没有收货的权限' };
    }
    // 如果没有发送者或者接收者，就创建一个
    const senderId = await ClientModel.checkExistElseCreate(senderPhone);
    const receiverId = await ClientModel.checkExistElseCreate(receiverPhone);
    if (!senderId || !receiverId) {
        return { msg: '系统创建账号错误，请重试' };
    }

    const sender = await ClientModel.findById(senderId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    });

    const order = await OrderModel.findByIdAndUpdate(preOrderId, {
        shopId: member.shopId,
        senderId,
        senderName,
        senderPhone,
        isSenderRepresentShipper: sender.shipperId && _T(sender.shipper.chairManId) === _T(sender.id),
        receiverId,
        receiverPhone,
        receiverName,
        name,
        isCityDistribute,
        endPoint,
        endPointLastCode,
        sendDoorEndPoint,
        sendDoorEndPointLastCode,
        receivePartmentId: member.partmentId,
        receiveMemberId: userId,
        totalNumbers,
        weight,
        size,
        isSendDoor,
        proxyCharge: Math.floor(proxyCharge),
        proxyChargeProfit: Math.floor(proxyCharge * setting.proxyChargeProfitRate),
        isReachPay,
        payMode: isReachPay ? CONSTANTS.PM_REACH : CONSTANTS.PM_IMMEDIATE,
        totalDesignatedFee: Math.floor(totalDesignatedFee),
        isInsuance,
        stateList: [ { state: CONSTANTS.OS_READY_PRINT_BAR_CODE, count: totalNumbers } ],
        placeOrderTime: Date.now(),
        $unset: { groupId: 1 },
    });
    await order.save();

    const group = await OrderGroupModel.findByIdAndUpdate(order.groupId, { $inc: {
        totalNumbers: -totalNumbers,
        totalWeight: -weight,
        totalSize: -size,
        orderCount: -1,
    }, $pull: { orderIdList: preOrderId } }, { new: true });
    if (group && !group.orderIdList.length) {
        await group.remove();
    }

    actionLog(userId, 'ShopMember', preOrderId, 'Order', 'placeOriginOrderWithPreOrder');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/placeTransferOrder.js

```js
import { ShopMemberModel, OrderModel, TruckModel, MediaModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import { setClientPickShipper } from '../../libs/order';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({
    userId, // 收货部收货员
    orderId,
    totalNumbers, // 一票货总的件数
    weight, // 一票货总的重量
    size, //一票货总的方量
}) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { addressRegionLastCode: 1, profitRate: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '你没有收货的权限' };
    }

    const undoOrder = await OrderModel.findOne({ receiveMemberId: userId }).sort({ placeOrderTime: 'desc' });
    if (undoOrder && _.find(undoOrder.stateList, o => o.state === CONSTANTS.OS_READY_PRINT_BAR_CODE)) {
        return { msg: '请先打印上一个货单的二维码' };
    }

    let preOrder = await OrderModel.getLatestOrderFromOriginOrder(orderId);
    preOrder = await OrderModel.findById(preOrder.id)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    }).populate({
        path: 'agentId',
        select: { chairManId: 1 },
    });
    if (!preOrder) {
        return { msg: '没有该货单' };
    }
    if (!preOrder.agentId && !preOrder.shipperId) {
        return { msg: '错误的货单' };
    }
    if (_.find(preOrder.stateList, o => o.state === CONSTANTS.OS_ON_TRANSFER)) {
        return { msg: '该货单已经存在中转单' };
    }

    let isCityDistribute = false;
    // 如果上一单的终点是该店所在的城市，则为同城配送
    if (preOrder.endPointLastCode === member.shop.addressRegionLastCode) {
        isCityDistribute = true;
    }
    const hasReadyReceive = !!_.find(preOrder.stateList, o => o.state === CONSTANTS.OS_READY_RECEIVE);

    const isClientPick = isCityDistribute && !preOrder.isSendDoor;
    // 如果不是送货上门，则为自提，如果是同城配送，但是又不是送货上门，则需要物流公司自提竞价
    const order = new OrderModel({
        isTransferOrder: true,
        preOrderId: orderId,
        shopId: member.shop.id,
        senderId: preOrder.shipperId ? preOrder.shipper.chairManId : preOrder.agent.chairManId,
        senderPhone: preOrder.senderPhone,
        senderName: preOrder.senderName,
        isSenderRepresentShipper: !!preOrder.shipperId,
        receiverId: preOrder.receiverId,
        receiverPhone: preOrder.receiverPhone,
        receiverName: preOrder.receiverName,
        name: preOrder.name,
        isCityDistribute,
        endPoint: isCityDistribute ? preOrder.sendDoorEndPoint : preOrder.endPoint,
        endPointLastCode: isCityDistribute ? preOrder.sendDoorEndPointLastCode : preOrder.endPointLastCode,
        sendDoorEndPoint: isCityDistribute ? undefined : preOrder.sendDoorEndPoint,
        sendDoorEndPointLastCode: isCityDistribute ? 0 : preOrder.sendDoorEndPointLastCode,
        receivePartmentId: member.partmentId,
        receiveMemberId: userId,
        totalNumbers: totalNumbers || preOrder.totalNumbers,
        weight: weight || preOrder.weight,
        size: size || preOrder.size,
        isSendDoor: isCityDistribute ? false : preOrder.isSendDoor,
        isClientPick,
        proxyCharge: preOrder.proxyCharge,
        proxyChargeProfit: preOrder.proxyChargeProfit,
        isReachPay: preOrder.isReachPay,
        payMode: preOrder.isReachPay ? CONSTANTS.PM_REACH : CONSTANTS.PM_IMMEDIATE,
        totalDesignatedFee: preOrder.totalDesignatedFee,
        isInsuance: preOrder.isInsuance,
        insuanceFee: preOrder.insuanceFee,
        insuanceMount: preOrder.insuanceMount,
        stateList: [ { state: isClientPick ? CONSTANTS.OS_READY_STOCK : CONSTANTS.OS_READY_SELECT_ROADMAP, count: totalNumbers } ],
        photo: preOrder.photo,
        placeOrderTime: Date.now(),
    });
    !isClientPick && await order.save();
    preOrder.nextOrderId = order.id;
    preOrder.stateList = [ { state: CONSTANTS.OS_ON_TRANSFER, count: preOrder.totalNumbers } ];
    await preOrder.save();

    // 当该货单已经被物流公司确认卸货过，再拿来中转，需要将之前的货单全部变为在中转中
    if (hasReadyReceive) {
        const orderIdList = await OrderModel.getOrderIdListFromNode(preOrder);
        if (orderIdList.length) {
            await OrderModel.update({ _id: { $in: orderIdList } }, { 'stateList.0.state': CONSTANTS.OS_ON_TRANSFER }, { multi: true });
        }
    }

    // 如果有下了中转单，则前一个货单的货车需要强制确认到达
    preOrder.truckId && await TruckModel.findByIdAndUpdate(preOrder.truckId, { state: CONSTANTS.TS_RECEIVE_SUCCESS });

    MediaModel._updateRef(
        { [ preOrder.photo ]: 1 },
    );

    // 如果是用户自提的情况
    if (isClientPick) {
        await setClientPickShipper(order, member.shop.profitRate);
        await order.save();
    }

    const context = order.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'ShopMember', order.id, 'Order', 'placeTransferOrder');
    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/printBarCode.js

```js
import { OrderModel, ShopMemberModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId, // 收货部收货员
    orderId,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '权限不足' };
    }
    const order = await OrderModel.findById(orderId);
    if (!order) {
        return { msg: '没有该货单' };
    }
    if (order.stateList[0].state !== CONSTANTS.OS_READY_PRINT_BAR_CODE) {
        return { msg: '该货单状态不正确' };
    }

    // if (!order.photo) {
    //     return { msg: '请先拍照上传图片' };
    // }

    order.stateList = [ { state: CONSTANTS.OS_READY_SELECT_ROADMAP, count: order.totalNumbers } ];
    order.modifyTime = Date.now();
    await order.save();

    let context = order.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'ShopMember', orderId, 'Order', 'printBarCode');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/printOrderBill.js

```js
import { OrderModel, ShopMemberModel, RoadmapMaskModel, ShipperModel, ShipperInBranchShopModel } from '../../../../models';
import { sequelize, AccountModel, BillModel, OrderPaymentModel } from '../../../../mysql';
import { _T, _F, _Y, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { settingMgr } from '../../../../manager';

const ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH = '10003';
const ERROR_OTHER = '10004';
function deductAmount (order, targetId, shipperId, proxyCharge, deductAmount) {
    return new Promise(async resolve => {
        const orderId = _T(order.id);
        const amount = proxyCharge + deductAmount;
        sequelize.transaction(async (transaction) => {
            const shipperAccount = await AccountModel.findOne({ where: { targetId, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } }, transaction });
            await shipperAccount.decrement({ amount }, { transaction });
            await AccountModel.increment({ amount }, { where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
            proxyCharge > 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -proxyCharge, remark: '抵押货款' }, { transaction });
            deductAmount !== 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -deductAmount, remark: deductAmount < 0 ? '收取运费' : '抵押指定收款' }, { transaction });
            const acountAmount = _Y(shipperAccount.amount - amount);

            await OrderPaymentModel.create({ orderId, isPayed: true, payedTime: new Date() }, { transaction });

            // 扣除物流公司的担保金额
            const inShop = await ShipperInBranchShopModel.findOne({ shipperId: order.shipper.id, shopId: order.shopId });
            if (!inShop || inShop.remainBondAmount < order.needBondAmount) {
                throw ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH;
            }
            inShop.remainBondAmount = inShop.remainBondAmount - order.needBondAmount;
            inShop.usedBondAmount = inShop.totalBondAmount - inShop.remainBondAmount;
            await inShop.save();

            // 扣款成功后，需要更新物流公司的账目
            const doc = await ShipperModel.findByIdAndUpdate(shipperId, { acountAmount }).catch(e => {
                throw e;
            });
            if (!doc) {
                throw ERROR_OTHER;
            }
        }).then(() => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}

export default async ({
    userId, // 收货部收货员
    orderId,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '权限不足' };
    }
    const order = await OrderModel.findById(orderId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    });
    if (!order) {
        return { msg: '没有该货单' };
    }
    const state = order.stateList[0].state;
    if (state !== CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER && state !== CONSTANTS.OS_READY_PRINT_ORDER) {
        return { msg: '该货单状态不正确' };
    }
    // 物流公司需要抵押总部和分店的提成部分划到总部账号
    if (state === CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER) {
        const err = await deductAmount(order, _T(order.shipper.chairManId), order.shipper.id, _F(order.proxyCharge), _F(order.totalDesignatedFee - order.fee));
        if (err === ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH) {
            order.deductError = true;
            order.modifyTime = Date.now();
            await order.save();
            return { msg: '该物流公司保证金，请选择其他公司的路线' };
        } else if (err) {
            order.deductError = true;
            order.modifyTime = Date.now();
            await order.save();
            return { msg: '物流公司扣款失败, 请选择其他路线' };
        }
    }
    order.stateList = [ { state: CONSTANTS.OS_READY_STOCK, count: order.totalNumbers } ];
    order.modifyTime = Date.now();
    await order.save();
    await RoadmapMaskModel.updateStatusList(order.shopId, order.endPointLastCode, order.roadmapRankIndex, settingMgr.getRealWeight(order.weight, order.size));

    let context = await OrderModel.findById(orderId);
    context = context.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'ShopMember', orderId, 'Order', 'printOrderBill');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/printOrderListBill.js

```js
import { OrderModel, ShopMemberModel, RoadmapMaskModel, ShipperModel, ShipperInBranchShopModel } from '../../../../models';
import { sequelize, AccountModel, BillModel, OrderPaymentModel } from '../../../../mysql';
import { _T, _F, _Y, hasAuthority, actionLogWithList } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import { settingMgr } from '../../../../manager';

const ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH = '10003';
const ERROR_OTHER = '10004';
function deductAmount (order, targetId, shipperId, proxyCharge, deductAmount) {
    return new Promise(async resolve => {
        const orderId = _T(order.id);
        const amount = proxyCharge + deductAmount;
        sequelize.transaction(async (transaction) => {
            const shipperAccount = await AccountModel.findOne({ where: { targetId, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } }, transaction });
            await shipperAccount.decrement({ amount }, { transaction });
            await AccountModel.increment({ amount }, { where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
            proxyCharge > 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -proxyCharge, remark: '抵押货款' }, { transaction });
            deductAmount !== 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -deductAmount, remark: deductAmount < 0 ? '收取运费' : '抵押指定收款' }, { transaction });
            const acountAmount = _Y(shipperAccount.amount - amount);

            await OrderPaymentModel.create({ orderId, isPayed: true, payedTime: new Date() }, { transaction });

            // 扣除物流公司的担保金额
            const inShop = await ShipperInBranchShopModel.findOne({ shipperId: order.shipper.id, shopId: order.shopId });
            if (!inShop || inShop.remainBondAmount < order.needBondAmount) {
                throw ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH;
            }
            inShop.remainBondAmount = inShop.remainBondAmount - order.needBondAmount;
            inShop.usedBondAmount = inShop.totalBondAmount - inShop.remainBondAmount;
            await inShop.save();

            // 扣款成功后，需要更新物流公司的账目
            const doc = await ShipperModel.findByIdAndUpdate(shipperId, { acountAmount }).catch(e => {
                throw e;
            });
            if (!doc) {
                throw ERROR_OTHER;
            }
        }).then(() => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}

export default async ({
    userId, // 收货部收货员
    orderIdList,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '权限不足' };
    }
    const failedList = [];
    for (const orderId of orderIdList) {
        const order = await OrderModel.findById(orderId)
        .populate({
            path: 'shipperId',
            select: { chairManId: 1 },
        });
        if (!order) {
            failedList.push({ msg: '某些货单不存在' });
            continue;
        }
        const state = order.stateList[0].state;
        if (state !== CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER && state !== CONSTANTS.OS_READY_PRINT_ORDER) {
            failedList.push({ msg: '某些货单状态不正确' });
            continue;
        }
        // 物流公司需要抵押总部和分店的提成部分划到总部账号
        if (order.stateList[0].state === CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER) {
            const err = await deductAmount(order, _T(order.shipper.chairManId), order.shipper.id, _F(order.proxyCharge), _F(order.totalDesignatedFee - order.fee));
            if (err === ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH) {
                order.deductError = true;
                order.modifyTime = Date.now();
                await order.save();
                return { msg: '该物流公司保证金，请选择其他公司的路线' };
            } else if (err) {
                failedList.push({ msg: '物流公司扣款失败, 请选择其他路线' });
                order.deductError = true;
                order.modifyTime = Date.now();
                await order.save();
                continue;
            }
        }
        order.stateList = [ { state: CONSTANTS.OS_READY_PRINT_ORDER, count: order.totalNumbers } ];
        order.modifyTime = Date.now();
        await order.save();
    }

    if (failedList.length) {
        return failedList[0];
    }

    for (const orderId of orderIdList) {
        const order = await OrderModel.findById(orderId);
        order.stateList = [ { state: CONSTANTS.OS_READY_STOCK, count: order.totalNumbers } ];
        order.modifyTime = Date.now();
        await order.save();
        await RoadmapMaskModel.updateStatusList(order.shopId, order.endPointLastCode, order.roadmapRankIndex, settingMgr.getRealWeight(order.weight, order.size));
    }
    actionLogWithList(userId, 'ShopMember', orderIdList.map(o => ({ id: o, model: 'Order' })), 'printOrderListBill');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/proxyPay.js

```js
import { sequelize, AccountModel, BillModel, OrderPaymentModel } from '../../../../mysql';
import { ShopMemberModel, OrderModel, ShipperModel, ShipperInBranchShopModel } from '../../../../models';
import { _T, _F, _Y, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

const ERROR_HAVE_PAYED = '10000';
const ERROR_PARTMENT_MONEY_NOT_ENOUGH = '10001';
const ERROR_SHIPPER_MONEY_NOT_ENOUGH = '10002';
const ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH = '10003';
const ERROR_OTHER = '10004';

function payment (order, partment, shipper) {
    return new Promise(async resolve => {
        const orderId = _T(order.id);
        sequelize.transaction(async (transaction) => {
            const orderPayment = await OrderPaymentModel.findOne({ where: { orderId, isPayed: true }, transaction });
            if (orderPayment) {
                throw ERROR_HAVE_PAYED;
            }
            // 部门的部分
            let amount = partment.transportFee + partment.insuanceFee;
            const partmentAccount = await AccountModel.findOne({ where: { targetId: partment.id, type: CONSTANTS.AT_BRANCH_SHOP_PARTMENT, amount: { $gte: amount } }, transaction });
            if (!partmentAccount) {
                throw ERROR_PARTMENT_MONEY_NOT_ENOUGH;
            }
            await partmentAccount.decrement({ amount }, { transaction });
            // 部门需要将运费部分中总部的提成和分店的提出优先划到安全账户
            const profit = partment.transportFee > partment.transportProfit ? partment.transportProfit : partment.transportFee;
            await AccountModel.increment({ amount: partment.insuanceFee + profit }, { where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
            partment.transportFee > 0 && await BillModel.create({ account: partmentAccount.id, orderId, tradeAmount: -partment.transportFee, remark: '代支付运费' }, { transaction });
            partment.insuanceFee > 0 && await BillModel.create({ account: partmentAccount.id, orderId, tradeAmount: -partment.insuanceFee, remark: '代支付保险' }, { transaction });
            // 物流公司部分
            amount = shipper.proxyCharge + shipper.deductAmount;
            const shipperAccount = await AccountModel.findOne({ where: { targetId: shipper.id, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } } });
            if (!shipperAccount) {
                throw ERROR_SHIPPER_MONEY_NOT_ENOUGH;
            }
            const acountAmount = _Y(shipperAccount.amount - amount);
            amount !== 0 && await shipperAccount.decrement({ amount }, { transaction });
            // 如果shipper.deductAmount大于0，说明需要抵押运费，抵押在安全账户的钱为amount，如果小于0，说明是运费收益，这部分钱直接进物流公司账户，这是只抵押shipper.proxyCharge
            await AccountModel.increment({ amount: shipper.deductAmount > 0 ? amount : shipper.proxyCharge }, { where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
            shipper.proxyCharge > 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -shipper.proxyCharge, remark: '抵押货款' }, { transaction });
            shipper.deductAmount !== 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -shipper.deductAmount, remark: shipper.deductAmount > 0 ? '抵押指定收款' : '收取运费' }, { transaction });

            await OrderPaymentModel.create({ orderId, isPayed: true, payedTime: new Date() }, { transaction });

            // 扣除物流公司的担保金额
            const inShop = await ShipperInBranchShopModel.findOne({ shipperId: order.shipper.id, shopId: order.shopId });
            if (!inShop || inShop.remainBondAmount < order.needBondAmount) {
                throw ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH;
            }
            inShop.remainBondAmount = inShop.remainBondAmount - order.needBondAmount;
            inShop.usedBondAmount = inShop.totalBondAmount - inShop.remainBondAmount;
            await inShop.save();

            // 扣款成功后，需要更新物流公司的账目
            const doc = await ShipperModel.findByIdAndUpdate(order.shipper.id, { acountAmount }).catch(e => {
                throw e;
            });
            if (!doc) {
                throw ERROR_OTHER;
            }
        }).then((result) => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}

export default async ({
    userId,
    orderId, //为某个货单支付
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '权限不足' };
    }
    const order = await OrderModel.findById(orderId)
    .populate({
        path: 'shipperId',
        select: { chairManId: 1 },
    });
    const state = order.stateList[0].state;
    if (state !== CONSTANTS.OS_READY_PAYMENT) {
        if (state === CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER || state > CONSTANTS.OS_READY_PAYMENT) {
            return { msg: '该货单无需再支付' };
        }
        return { msg: '该货单状态不正确' };
    }
    const err = await payment(
        order,
        { id: _T(member.partmentId), transportFee: _F(order.needPayTransportFee), insuanceFee: _F(order.needPayInsuanceFee), transportProfit: _F(order.masterProfit + order.branchProfit) },
        { id: _T(order.shipper.chairManId), proxyCharge: _F(order.proxyCharge), deductAmount: _F(order.totalDesignatedFee - order.fee) },
    );
    if (err === ERROR_HAVE_PAYED) {
        order.stateList = [ { state: CONSTANTS.OS_READY_PRINT_ORDER, count: order.totalNumbers } ];
        order.modifyTime = Date.now();
        await order.save();
        return { msg: '该货单无需再支付' };
    } else if (err === ERROR_PARTMENT_MONEY_NOT_ENOUGH) {
        return { msg: '余额不足，请先充值' };
    } else if (err === ERROR_SHIPPER_MONEY_NOT_ENOUGH) {
        order.deductError = true;
        order.modifyTime = Date.now();
        await order.save();
        return { msg: '该物流公司余额不足，请选择其他公司的路线' };
    } else if (err === ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH) {
        order.deductError = true;
        order.modifyTime = Date.now();
        await order.save();
        return { msg: '该物流公司保证金，请选择其他公司的路线' };
    } else if (err) {
        return { msg: '系统出错，请稍后再试' };
    }

    order.stateList = [ { state: CONSTANTS.OS_READY_PRINT_ORDER, count: order.totalNumbers } ];
    order.modifyTime = Date.now();
    await order.save();

    let context = await OrderModel.findById(orderId);
    context = context.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'ShopMember', orderId, 'Order', 'proxyOrder');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/proxyPayForOrderList.js

```js
import { sequelize, AccountModel, BillModel, OrderPaymentModel } from '../../../../mysql';
import { ShopMemberModel, OrderModel, ShipperModel, ShipperInBranchShopModel } from '../../../../models';
import { _T, _F, _Y, hasAuthority, actionLogWithList } from '../../../../utils';
import CONSTANTS from '../../../../constants';

const ERROR_HAVE_PAYED = '10000';
const ERROR_PARTMENT_MONEY_NOT_ENOUGH = '10001';
const ERROR_SHIPPER_MONEY_NOT_ENOUGH = '10002';
const ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH = '10003';
const ERROR_OTHER = '10004';

function payment (order, partment, shipper) {
    return new Promise(async resolve => {
        const orderId = _T(order.id);
        sequelize.transaction(async (transaction) => {
            const orderPayment = await OrderPaymentModel.findOne({ where: { orderId, isPayed: true }, transaction });
            if (orderPayment) {
                throw ERROR_HAVE_PAYED;
            }
            // 部门的部分
            let amount = partment.transportFee + partment.insuanceFee;
            const partmentAccount = await AccountModel.findOne({ where: { targetId: partment.id, type: CONSTANTS.AT_BRANCH_SHOP_PARTMENT, amount: { $gte: amount } }, transaction });
            if (!partmentAccount) {
                throw ERROR_PARTMENT_MONEY_NOT_ENOUGH;
            }
            await partmentAccount.decrement({ amount }, { transaction });
            // 部门需要将运费部分中总部的提成和分店的提出优先划到安全账户
            const profit = partment.transportFee > partment.transportProfit ? partment.transportProfit : partment.transportFee;
            await AccountModel.increment({ amount: partment.insuanceFee + profit }, { where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
            partment.transportFee > 0 && await BillModel.create({ account: partmentAccount.id, orderId, tradeAmount: -partment.transportFee, remark: '代支付运费' }, { transaction });
            partment.insuanceFee > 0 && await BillModel.create({ account: partmentAccount.id, orderId, tradeAmount: -partment.insuanceFee, remark: '代支付保险' }, { transaction });
            // 物流公司部分
            amount = shipper.proxyCharge + shipper.deductAmount;
            const shipperAccount = await AccountModel.findOne({ where: { targetId: shipper.id, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } } });
            if (!shipperAccount) {
                throw ERROR_SHIPPER_MONEY_NOT_ENOUGH;
            }
            const acountAmount = _Y(shipperAccount.amount - amount);
            amount !== 0 && await shipperAccount.decrement({ amount }, { transaction });
            // 如果shipper.deductAmount大于0，说明需要抵押运费，抵押在安全账户的钱为amount，如果小于0，说明是运费收益，这部分钱直接进物流公司账户，这是只抵押shipper.proxyCharge
            await AccountModel.increment({ amount: shipper.deductAmount > 0 ? amount : shipper.proxyCharge }, { where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
            shipper.proxyCharge > 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -shipper.proxyCharge, remark: '抵押货款' }, { transaction });
            shipper.deductAmount !== 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -shipper.deductAmount, remark: shipper.deductAmount > 0 ? '抵押指定收款' : '收取运费' }, { transaction });

            await OrderPaymentModel.create({ orderId, isPayed: true, payedTime: new Date() }, { transaction });

            // 扣除物流公司的担保金额
            const inShop = await ShipperInBranchShopModel.findOne({ shipperId: order.shipper.id, shopId: order.shopId });
            if (!inShop || inShop.remainBondAmount < order.needBondAmount) {
                throw ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH;
            }
            inShop.remainBondAmount = inShop.remainBondAmount - order.needBondAmount;
            inShop.usedBondAmount = inShop.totalBondAmount - inShop.remainBondAmount;
            await inShop.save();

            // 扣款成功后，需要更新物流公司的账目
            const doc = await ShipperModel.findByIdAndUpdate(order.shipper.id, { acountAmount }).catch(e => {
                throw e;
            });
            if (!doc) {
                throw ERROR_OTHER;
            }
        }).then((result) => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}

function deductAmount (order, targetId, shipperId, proxyCharge, deductAmount) {
    return new Promise(async resolve => {
        const orderId = _T(order.id);
        const amount = proxyCharge + deductAmount;
        sequelize.transaction(async (transaction) => {
            const shipperAccount = await AccountModel.findOne({ where: { targetId, type: CONSTANTS.AT_CLIENT, amount: { $gte: amount } }, transaction });
            await shipperAccount.decrement({ amount }, { transaction });
            await AccountModel.increment({ amount }, { where: { id: CONSTANTS.AC_SECURITY_ACCOUNT }, transaction });
            proxyCharge > 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -proxyCharge, remark: '抵押货款' }, { transaction });
            deductAmount !== 0 && await BillModel.create({ account: shipperAccount.id, orderId, tradeAmount: -deductAmount, remark: deductAmount < 0 ? '收取运费' : '抵押指定收款' }, { transaction });
            const acountAmount = _Y(shipperAccount.amount - amount);

            await OrderPaymentModel.create({ orderId, isPayed: true, payedTime: new Date() }, { transaction });

            // 扣除物流公司的担保金额
            const inShop = await ShipperInBranchShopModel.findOne({ shipperId: order.shipper.id, shopId: order.shopId });
            if (!inShop || inShop.remainBondAmount < order.needBondAmount) {
                throw ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH;
            }
            inShop.remainBondAmount = inShop.remainBondAmount - order.needBondAmount;
            inShop.usedBondAmount = inShop.totalBondAmount - inShop.remainBondAmount;
            await inShop.save();

            // 扣款成功后，需要更新物流公司的账目
            const doc = await ShipperModel.findByIdAndUpdate(shipperId, { acountAmount }).catch(e => {
                throw e;
            });
            if (!doc) {
                throw ERROR_OTHER;
            }
        }).then(() => {
            return resolve();
        }).catch(err => {
            console.log('[transaction]:', err);
            return resolve(err);
        });
    });
}

export default async ({
    userId,
    orderIdList,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '权限不足' };
    }
    const failedList = [];
    for (const orderId of orderIdList) {
        const order = await OrderModel.findById(orderId)
        .populate({
            path: 'shipperId',
            select: { chairManId: 1 },
        });
        if (!order) {
            failedList.push({ msg: '某些货单不存在' });
            continue;
        }
        if (order.stateList[0].state === CONSTANTS.OS_READY_PAYMENT) {
            const err = await payment(
                order,
                { id: _T(member.partmentId), transportFee: _F(order.needPayTransportFee), insuanceFee: _F(order.needPayInsuanceFee), transportProfit: _F(order.masterProfit + order.branchProfit) },
                { id: _T(order.shipper.chairManId), proxyCharge: _F(order.proxyCharge), deductAmount: _F(order.totalDesignatedFee - order.fee) },
            );
            if (err === ERROR_HAVE_PAYED) {
                order.stateList = [ { state: CONSTANTS.OS_READY_PRINT_ORDER, count: order.totalNumbers } ];
                order.modifyTime = Date.now();
                await order.save();
                continue;
            } else if (err === ERROR_PARTMENT_MONEY_NOT_ENOUGH) {
                return { msg: '余额不足，部分货单未完成支付，请先充值' };
            } else if (err === ERROR_SHIPPER_MONEY_NOT_ENOUGH) {
                order.deductError = true;
                order.modifyTime = Date.now();
                await order.save();
                failedList.push({ msg: '物流公司余额不足, 部分货单未完成支付' });
                continue;
            } else if (err === ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH) {
                order.deductError = true;
                order.modifyTime = Date.now();
                await order.save();
                failedList.push({ msg: '该物流公司保证金, 部分货单未完成支付' });
            } else if (err) {
                failedList.push({ msg: '系统出错, 部分货单未完成支付' });
                continue;
            }
        } else if (order.stateList[0].state === CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER) {
            const err = await deductAmount(order, _T(order.shipper.chairManId), order.shipper.id, _F(order.proxyCharge), _F(order.totalDesignatedFee - order.fee));
            if (err === ERROR_SHIPPER_BOND_AMOUNT_NOT_ENOUGH) {
                order.deductError = true;
                order.modifyTime = Date.now();
                await order.save();
                failedList.push({ msg: '该物流公司保证金, 部分货单未完成支付' });
            } else if (err) {
                order.deductError = true;
                order.modifyTime = Date.now();
                await order.save();
                failedList.push({ msg: '部分货单未扣款失败' });
                continue;
            }
        }
        order.stateList = [ { state: CONSTANTS.OS_READY_PRINT_ORDER, count: order.totalNumbers } ];
        order.modifyTime = Date.now();
        await order.save();
    }

    actionLogWithList(userId, 'ShopMember', orderIdList.map(o => ({ id: o, model: 'Order' })), 'proxyOrderForOrderList');

    if (failedList.length) {
        return failedList[0];
    }

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/removeOrder.js

```js
import { OrderModel, ShopMemberModel, MediaModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderId,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '你没有删除货单的权限' };
    }
    const order = await OrderModel.findByIdAndRemove(orderId);
    if (order && order.photo) {
        MediaModel._updateRef(
            { [ order.photo ]: -1 },
        );
    }

    actionLog(userId, 'ShopMember', orderId, 'Order', 'removeOrder');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/selectRoadmap.js

```js
import { OrderModel, ShopMemberModel, RoadmapModel, ShipperInBranchShopModel, WarehouseModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import { getRegionProfitRateByEndPoint } from '../../libs/roadmap';
import CONSTANTS from '../../../../constants';
import { setting, settingMgr } from '../../../../manager';
import _ from 'lodash';

export default async ({
    userId,
    orderId,
    roadmapId,
    roadmapRankIndex, //路线的排名
}) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { profitRate: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '权限不足' };
    }
    const order = await OrderModel.findById(orderId);
    if (!order) {
        return { msg: '没有该货单' };
    }
    if (order.stateList[0].state !== CONSTANTS.OS_READY_SELECT_ROADMAP) {
        return { msg: '该货单状态不正确' };
    }
    const roadmap = await RoadmapModel.findById(roadmapId)
    .populate({
        path: 'shipperId',
        select: { acountAmount: 1, chairManId: 1, name: 1 },
    });
    if (!roadmap) {
        return { msg: '没有该路线' };
    }
    if (!roadmap.enable) {
        return { msg: '该路线停运中' };
    }
    const inShop = await ShipperInBranchShopModel.findById(roadmap.shipperInBranchShopId);
    if (!inShop) {
        return { msg: '该路线无效' };
    }
    const needBondAmount = settingMgr.getBondAmount(order.weight, order.size);
    if (inShop.remainBondAmount < needBondAmount) {
        return { msg: '该路线的保证金不足' };
    }
    const weight = settingMgr.getFloatWeight(order.weight, order.size);
    const regionProfitRate = !_.isNil(roadmap.profitRate) ? roadmap.profitRate : await getRegionProfitRateByEndPoint(roadmap.shopId, roadmap.endPointLastCode, roadmap.isCityDistribute);
    const sendDoor = order.isSendDoor && _.find(roadmap.sendDoorList, o => o.sendDoorEndPointLastCode === order.sendDoorEndPointLastCode);
    const warehouseList = await WarehouseModel.find({ shopId: roadmap.shopId, shipperList: roadmap.shipper.id });

    order.warehouse = warehouseList.length ? _.map(warehouseList, o => o.houseNo).join(';') : roadmap.shipper.name;
    order.price = roadmap.price * (regionProfitRate + 1);
    order.minFee = roadmap.minFee * (regionProfitRate + 1);
    order.sendDoorPrice = order.isSendDoor ? sendDoor.sendDoorPrice * (regionProfitRate + 1) : 0;
    order.sendDoorMinFee = order.isSendDoor ? sendDoor.sendDoorMinFee * (regionProfitRate + 1) : 0;
    order.needBondAmount = needBondAmount;
    order.roadmapId = roadmapId;
    order.roadmapRankIndex = roadmapRankIndex;
    order.shipperId = roadmap.shipper.id;
    order.shipperChairManId = roadmap.shipper.chairManId;
    order.fee = Math.floor(Math.max(roadmap.price * weight, roadmap.minFee) + (order.isSendDoor ? Math.max(sendDoor.sendDoorPrice * weight, sendDoor.sendDoorMinFee) : 0)); // 起价
    order.profit = Math.floor(order.fee * regionProfitRate);
    order.branchProfit = Math.floor(order.profit * member.shop.profitRate);
    order.masterProfit = order.profit - order.branchProfit;
    order.realFee = order.profit + order.fee; // 实际运费.
    if (order.payMode !== CONSTANTS.PM_IMMEDIATE && !order.totalDesignatedFee) {
        order.totalDesignatedFee = order.realFee;
    }
    const needDeductAmount = order.proxyCharge + (order.payMode !== CONSTANTS.PM_IMMEDIATE ? order.totalDesignatedFee - order.fee : 0);
    if (roadmap.shipper.acountAmount < needDeductAmount) {
        return { msg: '该物流公司资金不足' };
    }
    order.needPayTransportFee = 0;
    if (order.payMode !== CONSTANTS.PM_IMMEDIATE) { // 到付或者混合支付
        const senderProfit = order.totalDesignatedFee - order.realFee; // 发货人收益
        order.designatedFee = Math.max(senderProfit, 0);
        if (senderProfit < 0) {
            order.needPayTransportFee = -senderProfit; // 发货人需要现付的部分
        }
    } else { // 现付
        order.needPayTransportFee = order.realFee;
    }
    // 保险费
    if (order.isInsuance) {
        order.insuanceMount = order.realFee * setting.insuanceMountRate;
        order.insuanceFee = order.insuanceMount * setting.insuanceRate;
        if (order.insuanceFee < setting.insuanceBaseValue) { // 如果保险
            order.insuanceFee = setting.insuanceBaseValue;
        }
        order.insuanceFee = Math.floor(order.insuanceFee);
        order.insuanceMount = Math.floor(order.insuanceFee / setting.insuanceRate);
    } else {
        order.insuanceMount = 0;
        order.insuanceFee = 0;
    }

    order.needPayInsuanceFee = 0;
    if (order.insuanceFee > 0) { // 只有初始单才需要交保险(但是如果是收货点的单，需要交纳保险费，因为他收取了用户的保险费)
        if (!order.isTransferOrder) {
            order.needPayInsuanceFee = order.insuanceFee;
        } else {
            const preOrder = await OrderModel.findById(order.preOrderId);
            if (preOrder && preOrder.agentId) {
                order.needPayInsuanceFee = order.insuanceFee;
            }
        }
    }
    if (order.needPayTransportFee > 0 && order.totalDesignatedFee > 0) {
        order.payMode = CONSTANTS.PM_MIXED;
    }
    if (order.needPayTransportFee + order.needPayInsuanceFee) { // 需要交费的情况进入待支付状态，否则到付的情况下进入待入库状态，并且划账对接
        order.stateList = [ { state: CONSTANTS.OS_READY_PAYMENT, count: order.totalNumbers } ];
    } else {
        order.stateList = [ { state: CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER, count: order.totalNumbers } ];
    }
    order.deductError = false;
    order.modifyTime = Date.now();
    await order.save();

    let context = await OrderModel.findById(orderId);
    context = context.toObject();
    context.state = context.stateList[0].state;

    actionLog(userId, 'ShopMember', orderId, 'Order', 'selectRoadmap');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/selectRoadmapForOrderList.js

```js
import { OrderModel, ShopMemberModel, RoadmapModel, WarehouseModel } from '../../../../models';
import { hasAuthority, actionLogWithList } from '../../../../utils';
import { searchRoadmapListByReceivePartment, getRegionProfitRateByEndPoint } from '../../libs/roadmap';
import CONSTANTS from '../../../../constants';
import { setting, settingMgr } from '../../../../manager';
import _ from 'lodash';

export default async ({
    userId,
    orderIdList,
}) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { profitRate: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '权限不足' };
    }

    const failedList = [];
    for (const orderId of orderIdList) {
        const order = await OrderModel.findById(orderId);
        if (!order) {
            failedList.push({ msg: '某些货单不存在' });
            continue;
        }
        if (order.stateList[0].state !== CONSTANTS.OS_READY_SELECT_ROADMAP) {
            return { msg: '该货单状态不正确' };
        }
        let {
            endPointLastCode, // 终点
            sendDoorEndPointLastCode, // 送货上门终点
            isSendDoor, // 是否送货上门
            weight, // 重量
            size, // 方量
            proxyCharge, // 代收货款金额
            isCityDistribute, // 是否是同城配送
            isReachPay, // 是否是到付
            totalDesignatedFee, //指定向收货人应该收多少钱
        } = order;
        const regionProfitRate = await getRegionProfitRateByEndPoint(member.shop.id, endPointLastCode, isCityDistribute);
        const { roadmapList } = await searchRoadmapListByReceivePartment({
            shopId: member.shop.id,
            pageNo: 0,
            pageSize: 1,
            endPointLastCode,
            sendDoorEndPointLastCode,
            isSendDoor,
            weight,
            size,
            proxyCharge,
            isReachPay,
            totalDesignatedFee,
            orderBy: 0,
            regionProfitRate,
        });

        if (!roadmapList || !roadmapList.length) {
            failedList.push({ msg: '某些货单没有找到适合的路线' });
            continue;
        }

        const roadmap = await RoadmapModel.findById(roadmapList[0].id)
        .populate({
            path: 'shipperId',
            select: { acountAmount: 1, chairManId: 1, name: 1 },
        });
        weight = settingMgr.getFloatWeight(order.weight, order.size);
        const sendDoor = order.isSendDoor && _.find(roadmap.sendDoorList, o => o.sendDoorEndPointLastCode === order.sendDoorEndPointLastCode);
        const profitRate = !_.isNil(roadmap.profitRate) ? roadmap.profitRate : regionProfitRate;
        const warehouseList = await WarehouseModel.find({ shopId: roadmap.shopId, shipperList: roadmap.shipper.id });
        order.warehouse = warehouseList.length ? _.map(warehouseList, o => o.houseNo).join(';') : roadmap.shipper.name;
        order.price = roadmap.price * (profitRate + 1);
        order.minFee = roadmap.minFee * (profitRate + 1);
        order.sendDoorPrice = order.isSendDoor ? sendDoor.sendDoorPrice * (profitRate + 1) : 0;
        order.sendDoorMinFee = order.isSendDoor ? sendDoor.sendDoorMinFee * (profitRate + 1) : 0;
        order.needBondAmount = roadmapList[0].needBondAmount;
        order.roadmapId = roadmap.id;
        order.roadmapRankIndex = roadmapList[0].roadmapRankIndex;
        order.shipperId = roadmap.shipper.id;
        order.shipperChairManId = roadmap.shipper.chairManId;
        order.fee = Math.floor(Math.max(roadmap.price * weight, roadmap.minFee) + (order.isSendDoor ? Math.max(sendDoor.sendDoorPrice * weight, sendDoor.sendDoorMinFee) : 0)); // 起价
        order.profit = Math.floor(order.fee * profitRate);
        order.branchProfit = Math.floor(order.profit * member.shop.profitRate);
        order.masterProfit = order.profit - order.branchProfit;
        order.realFee = order.profit + order.fee; // 实际运费.
        if (order.payMode !== CONSTANTS.PM_IMMEDIATE && !order.totalDesignatedFee) {
            order.totalDesignatedFee = order.realFee;
        }
        order.needPayTransportFee = 0;
        if (order.payMode !== CONSTANTS.PM_IMMEDIATE) { // 到付或者混合支付
            const senderProfit = order.totalDesignatedFee - order.realFee; // 发货人收益
            order.designatedFee = Math.max(senderProfit, 0);
            if (senderProfit < 0) {
                order.needPayTransportFee = -senderProfit; // 发货人需要现付的部分
            }
        } else { // 现付
            order.needPayTransportFee = order.realFee;
        }
        // 保险费
        if (order.isInsuance) {
            order.insuanceMount = order.realFee * setting.insuanceMountRate;
            order.insuanceFee = order.insuanceMount * setting.insuanceRate;
            if (order.insuanceFee < setting.insuanceBaseValue) { // 如果保险
                order.insuanceFee = setting.insuanceBaseValue;
            }
            order.insuanceFee = Math.floor(order.insuanceFee);
            order.insuanceMount = Math.floor(order.insuanceFee / setting.insuanceRate);
        } else {
            order.insuanceMount = 0;
            order.insuanceFee = 0;
        }

        order.needPayInsuanceFee = 0;
        if (order.insuanceFee > 0) { // 只有初始单才需要交保险(但是如果是收货点的单，需要交纳保险费，因为他收取了用户的保险费)
            if (!order.isTransferOrder) {
                order.needPayInsuanceFee = order.insuanceFee;
            } else {
                const preOrder = await OrderModel.findById(order.preOrderId);
                if (preOrder && preOrder.agentId) {
                    order.needPayInsuanceFee = order.insuanceFee;
                }
            }
        }
        if (order.needPayTransportFee > 0 && order.totalDesignatedFee > 0) {
            order.payMode = CONSTANTS.PM_MIXED;
        }
        if (order.needPayTransportFee + order.needPayInsuanceFee) { // 需要交费的情况进入待支付状态，否则到付的情况下进入待入库状态，并且划账对接
            order.stateList = [ { state: CONSTANTS.OS_READY_PAYMENT, count: order.totalNumbers } ];
        } else {
            order.stateList = [ { state: CONSTANTS.OS_READY_DEDUCT_AND_PRINT_ORDER, count: order.totalNumbers } ];
        }
        order.deductError = false;
        order.modifyTime = Date.now();
        await order.save();
    }

    actionLogWithList(userId, 'ShopMember', orderIdList.map(o => ({ id: o, model: 'Order' })), 'selectRoadmapForOrderList');

    if (failedList.length) {
        return failedList[0];
    }

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/receivePartment/setPhotoForOriginOrder.js

```js
import { ShopMemberModel, OrderModel, MediaModel } from '../../../../models';
import { getMediaId, hasAuthority, actionLog } from '../../../../utils';
import { getRPLatestOrder } from '../../libs/order';
import CONSTANTS from '../../../../constants';

export default async ({
    userId, // 收货部收货员
    orderId,
    photo,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_RECEIVE_PARTMENT)) {
        return { msg: '你没有收货的权限' };
    }
    let order;
    const _photo = getMediaId(photo);
    if (orderId) {
        order = await OrderModel.findById(orderId);
    } else {
        order = await getRPLatestOrder(userId);
    }
    if (!order) {
        return { msg: '没有该货单' };
    }
    const __photo = order.photo;
    order.photo = _photo;
    order.modifyTime = Date.now();
    await order.save();

    MediaModel._updateRef(
        { [_photo]: 1 },
        { [photo && __photo]: -1 },
    );
    actionLog(userId, 'ShopMember', orderId, 'Order', 'setPhotoForOriginOrder');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/partment/createPartment.js

```js
import { ShopPartmentModel, ShopMemberModel } from '../../../../models';
import { AccountModel } from '../../../../mysql';
import { _T, registerUser, hasAuthority, getDefaultPassWord, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    chargeManPhone, // 部门负责人登录电话
    chargeManName,  // 部门负责人姓名
    chargeManPassword, // 部门负责人登录密码（不传入该参数，默认为手机号码后6位）
    name,
    type,
    descript,
    phoneList,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_CREATE_PARTMENT)) {
        return { msg: '你没有创建部门的权限' };
    }

    const authorities = {
        [CONSTANTS.SP_RECEIVE_PARTMENT]: [CONSTANTS.AH_RECEIVE_PARTMENT, CONSTANTS.AH_RECHARGE, CONSTANTS.AH_WITHDRAW, CONSTANTS.AH_LOOK_AMOUNT],
        [CONSTANTS.SP_WARE_HOUSE_PARTMENT]: [CONSTANTS.AH_WARE_HOUSE_PARTMENT],
        [CONSTANTS.SP_CARRY_PARTMENT]: [CONSTANTS.AH_CARRY_PARTMENT, CONSTANTS.AH_WITHDRAW, CONSTANTS.AH_LOOK_AMOUNT],
        [CONSTANTS.SP_SECURITY_CHECK_PARTMENT]: [CONSTANTS.AH_SECURITY_CHECK_PARTMENT],
        [CONSTANTS.SP_DISTRIBUTION_PARTMENT]: [CONSTANTS.AH_DISTRIBUTION_PARTMENT],
    };

    // 注册部门负责人号
    const chargeMan = new ShopMemberModel({
        shopId: member.shopId,
        phone: chargeManPhone,
        name: chargeManName,
        post: CONSTANTS.PO_PARTMENT_CHARGE_MAN,
        authority: [
            ...(authorities[type] || []), // 部门的权限
            // 部门
            CONSTANTS.AH_MODIFY_OWN_PARTMENT, // 修改所在部门信息的权限
            // 成员
            CONSTANTS.AH_CREATE_MEMBER, // 创建成员的权限
            CONSTANTS.AH_MODIFY_MEMBER, // 修改成员信息的权限
            CONSTANTS.AH_REMOVE_MEMBER, // 删除成员的权限
            CONSTANTS.AH_LOOK_MEMBER, // 查看成员的权限
            // 统计
            CONSTANTS.AH_LOOK_STATISTICS, // 查看统计信息的权限
        ],
    });

    // 创建部门
    const partment = new ShopPartmentModel({
        shopId: member.shopId,
        chargeManId: chargeMan.id,
        name,
        type,
        descript,
        phoneList,
    });
    chargeMan.partmentId = partment.id;

    // 保存
    const error = await registerUser(ShopMemberModel, chargeMan, getDefaultPassWord(chargeManPhone, chargeManPassword));
    if (error === 'UserExistsError') {
        return { msg: '部门负责人账号已经被占用' };
    } else if (error) {
        return { msg: '部门负责人账号注册失败' };
    }

    // 创建部门
    const account = await AccountModel.create({ targetId: _T(partment.id), type: CONSTANTS.AT_BRANCH_SHOP_PARTMENT });
    if (!account) {
        return { msg: '创建部门账号失败' };
    }
    await partment.save();

    actionLog(userId, 'ShopMember', partment.id, 'ShopPartment', 'createPartment');

    const context = await ShopPartmentModel.findById(partment.id)
    .select({
        name: 1,
        type: 1,
        descript: 1,
        phoneList: 1,
        chargeManId: 1,
    }).populate({
        path: 'chargeManId',
        select: { name: 1, phone: 1 },
    });

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/partment/getOwnPartmentInfo.js

```js
import { ShopPartmentModel } from '../../../../models';
import { getPartmentIdByMember } from '../../../../utils';

export default async ({
    userId,
}) => {
    const partmentId = await getPartmentIdByMember(userId);
    if (!partmentId) {
        return { msg: '你没有加入任何部门' };
    }
    const doc = await ShopPartmentModel.findById(partmentId)
    .populate({
        path: 'chargeManId',
        select: { name: 1, phone: 1 },
    });
    if (!doc) {
        return { msg: '你没有加入任何部门' };
    }
    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shop/partment/getPartmentDetail.js

```js
import { ShopPartmentModel } from '../../../../models';

export default async ({
    userId,
    partmentId,
}) => {
    const doc = await ShopPartmentModel.findById(partmentId)
    .populate({
        path: 'chargeManId',
        select: { name: 1, phone: 1, head: 1, email: 1, phoneList: 1 },
    });
    if (!doc) {
        return { msg: '没有该部门' };
    }
    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shop/partment/getPartmentList.js

```js
import { ShopPartmentModel } from '../../../../models';
import { getKeywordCriteriaForPartment, getOwnShop } from '../../../../utils';

export default async ({ userId, keyword, fromPC, pageNo, pageSize }) => {
    const shopId = await getOwnShop(userId);
    const criteria = getKeywordCriteriaForPartment(keyword, { shopId });
    const count = (fromPC && pageNo === 0) ? await ShopPartmentModel.count(criteria) : undefined;
    const query = ShopPartmentModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        name: 1,
        type: 1,
        descript: 1,
        phoneList: 1,
        chargeManId: 1,
    }).populate({
        path: 'chargeManId',
        select: { name: 1, phone: 1 },
    });

    return { success: true, context: {
        count,
        partmentList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/partment/modifyOwnPartment.js

```js
import { ShopPartmentModel, ShopMemberModel } from '../../../../models';
import { omitNil, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    descript,
    phoneList,
    price,
    enable,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_OWN_PARTMENT)) {
        return { msg: '你没有修改部门的权限' };
    }

    const doc = await ShopPartmentModel.findByIdAndUpdate(member.partmentId, omitNil({
        descript,
        phoneList,
        price,
        enable,
        modifyTime: Date.now(),
    }), { new: true }).select({
        chargeManId: 1,
        name: 1,
        type: 1,
        descript: 1,
        phoneList: 1,
        price: 1,
        enable: 1,
    }).populate({
        path: 'chargeManId',
        select: { name: 1, phone: 1 },
    });
    if (!doc) {
        return { msg: '修改失败' };
    }

    actionLog(userId, 'ShopMember', member.partmentId, 'ShopPartment', 'modifyOwnPartment');

    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shop/partment/modifyPartment.js

```js
import { ShopPartmentModel } from '../../../../models';
import { omitNil, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    partmentId,
    name,
    type,
    descript,
    phoneList,
}) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_PARTMENT)) {
        return { msg: '你没有修改部门的权限' };
    }

    const doc = await ShopPartmentModel.findByIdAndUpdate(partmentId, omitNil({
        name,
        type,
        descript,
        phoneList,
        modifyTime: Date.now(),
    }), { new: true }).select({
        name: 1,
        descript: 1,
        phoneList: 1,
        chargeManId: 1,
    }).populate({
        path: 'chargeManId',
        select: { name: 1, phone: 1 },
    });
    if (!doc) {
        return { msg: '修改失败' };
    }

    actionLog(userId, 'ShopMember', partmentId, 'ShopPartment', 'modifyPartment');

    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shop/partment/removePartment.js

```js
import { ShopPartmentModel, ShopMemberModel } from '../../../../models';
import { authenticatePassword, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    partmentId,
    password,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_REMOVE_PARTMENT)) {
        return { msg: '你没有删除部门的权限' };
    }
    const error = await authenticatePassword(member, password);
    if (error) {
        return { msg: '密码错误' };
    }
    await ShopPartmentModel.findByIdAndRemove(partmentId);

    actionLog(userId, 'ShopMember', partmentId, 'ShopPartment', 'removePartment');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/roadmap/addRoadmapRegionProfitRate.js

```js
import { RoadmapRegionProfitRateModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    shopId,
    region = '',
    regionLastCode = 0,
    profitRate,
    type = 0,
}) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP_PROFIT)) {
        return { msg: '你没有修改路线的提成的权限' };
    }
    let regionRate = await RoadmapRegionProfitRateModel.findOne({ shopId, regionLastCode, type });
    if (regionRate) {
        return { msg: '这个方向的提成已经存在' };
    }

    regionRate = new RoadmapRegionProfitRateModel({
        shopId,
        region,
        regionLastCode,
        profitRate,
        type,
    });
    await regionRate.save();

    const context = await RoadmapRegionProfitRateModel.findById(regionRate.id)
    .select({
        shopId: 1,
        region: 1,
        regionLastCode: 1,
        profitRate: 1,
        type: 1,
        createTime: 1,
        modifyTime: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });

    actionLog(userId, 'ShopMember', regionRate.id, 'RoadmapRegionProfitRate', 'addRoadmapRegionProfitRate');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/roadmap/getRoadmapDetail.js

```js
import { RoadmapModel } from '../../../../models';

export default async ({ userId, roadmapId }) => {
    const context = await RoadmapModel.findById(roadmapId)
    .populate({
        path: 'truckId',
        select: {
            name: 1,
            plateNo: 1,
            capacity: 1,
            size: 1,
            remark: 1,
        },
    });
    if (!context) {
        return { msg: '没有该线路' };
    }

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/roadmap/getRoadmapList.js

```js
import { RoadmapModel, ShopMemberModel } from '../../../../models';
import { getKeywordCriteriaForRoadmap } from '../../../../utils';
import { getRegionProfitRateByEndPoint } from '../../libs/roadmap';
import _ from 'lodash';

export default async ({ userId, keyword, shopId, fromPC, pageNo, pageSize }) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { isMasterShop: 1 },
    });
    if (!member.shop.isMasterShop) {
        shopId = member.shop.id;
    }
    const criteria = getKeywordCriteriaForRoadmap(keyword, shopId ? { shopId } : {});
    const count = (fromPC && pageNo === 0) ? await RoadmapModel.count(criteria) : undefined;
    const query = RoadmapModel.find(criteria).sort({ modifyTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        shipperId: 1,
        shopId: 1,
        endPoint: 1,
        endPointLastCode: 1,
        transitPoint: 1,
        price: 1,
        minFee: 1,
        duration: 1,
        profitRate: 1,
    })
    .populate({
        path: 'shipperId',
        select: { name: 1 },
    })
    .populate({
        path: 'shopId',
        select: { name: 1, profitRate: 1, address: 1 },
    });

    const roadmapList = [];
    for (const doc of docs) {
        const item = doc.toObject();
        _.isNil(item.profitRate) && (item.defaultProfitRate = await getRegionProfitRateByEndPoint(item.shop.id, item.endPointLastCode, item.isCityDistribute));
        roadmapList.push(item);
    }

    return { success: true, context: {
        count,
        roadmapList,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/roadmap/getRoadmapRegionProfitRateDetail.js

```js
import { RoadmapRegionProfitRateModel } from '../../../../models';

export default async ({ userId, regionRateId }) => {
    const doc = await RoadmapRegionProfitRateModel.findById(regionRateId)
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });
    if (!doc) {
        return { msg: '没有该设置' };
    }

    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shop/roadmap/getRoadmapRegionProfitRateList.js

```js
import { RoadmapRegionProfitRateModel } from '../../../../models';
import { getKeywordCriteriaForRoadmapRegion } from '../../../../utils';

export default async ({ userId, shopId, keyword, fromPC, pageNo, pageSize }) => {
    const criteria = getKeywordCriteriaForRoadmapRegion(keyword, shopId ? { shopId: { $in: [ shopId, null ] } } : undefined);
    const count = (fromPC && pageNo === 0) ? await RoadmapRegionProfitRateModel.count(criteria) : undefined;
    const query = RoadmapRegionProfitRateModel.find(criteria).sort({ level: 'desc', modifyTime: 'desc', createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        shopId: 1,
        region: 1,
        regionLastCode: 1,
        profitRate: 1,
        type: 1,
        createTime: 1,
        modifyTime: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });

    return { success: true, context: {
        count,
        regionRateList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/roadmap/modifyRoadmapRegionProfitRate.js

```js
import { RoadmapRegionProfitRateModel } from '../../../../models';
import { _T, hasAuthority, actionLog, omitNil } from '../../../../utils';
import CONSTANTS from '../../../../constants';

// shopId 为必传参数，如果不传，则会修改为针对所有分店
// region 为必传参数，如果不传，则会修改为针对所有方向
// type 为必传参数，如果不传，则会修改为针对所有类型
export default async ({
    userId,
    regionRateId,
    shopId,
    region = '',
    regionLastCode = 0,
    profitRate,
    type = 0,
}) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP_PROFIT)) {
        return { msg: '你没有修改路线的提成的权限' };
    }

    let regionRate = await RoadmapRegionProfitRateModel.findOne({ shopId, regionLastCode, type });
    if (regionRate && regionRateId !== _T(regionRate.id)) {
        return { msg: '这个方向的提成已经存在' };
    }

    regionRate = await RoadmapRegionProfitRateModel.findByIdAndUpdate(regionRateId, {
        ...omitNil({
            profitRate,
        }),
        region,
        regionLastCode,
        type,
        shopId,
        modifyTime: Date.now(),
    }, { new: true });

    if (!regionRate) {
        return { msg: '提成不存在' };
    }

    const context = await RoadmapRegionProfitRateModel.findById(regionRate.id)
    .select({
        shopId: 1,
        region: 1,
        regionLastCode: 1,
        profitRate: 1,
        type: 1,
        createTime: 1,
        modifyTime: 1,
    })
    .populate({
        path: 'shopId',
        select: { name: 1 },
    });

    actionLog(userId, 'ShopMember', regionRateId, 'RoadmapRegionProfitRate', 'modifyRoadmapRegionProfitRate');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/roadmap/removeRoadmapRegionProfitRate.js

```js
import { RoadmapRegionProfitRateModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, regionRateId }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP_PROFIT)) {
        return { msg: '你没有修改路线的提成的权限' };
    }

    await RoadmapRegionProfitRateModel.findByIdAndRemove(regionRateId);

    actionLog(userId, 'ShopMember', regionRateId, 'RoadmapRegionProfitRate', 'removeRoadmapRegionProfitRate');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/roadmap/setRegionRateProfit.js

```js
import { RoadmapRegionProfitRateModel } from '../../../../models';
import { hasAuthority, actionLogWithList } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, regionRateIdList, profitRate = 0 }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP_PROFIT)) {
        return { msg: '你没有修改路线的提成的权限' };
    }

    await RoadmapRegionProfitRateModel.update({ _id: { $in: regionRateIdList } }, { profitRate, modifyTime: Date.now() }, { multi: true, upsert: true });
    actionLogWithList(userId, 'ShopMember', regionRateIdList.map(o => ({ id: o, model: 'RoadmapRegionProfitRate' })), 'setRegionRateProfit');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/roadmap/setRoadmapProfit.js

```js
import { RoadmapModel } from '../../../../models';
import { hasAuthority, actionLogWithList } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({ userId, roadmapIdList, profitRate }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_ROADMAP_PROFIT)) {
        return { msg: '你没有修改路线的提成的权限' };
    }

    let options = { profitRate };
    if (_.isNil(profitRate)) {
        options = { $unset: { profitRate: true } };
    }
    await RoadmapModel.update({ _id: { $in: roadmapIdList } }, { ...options, setProfitRateTime: Date.now() }, { multi: true, upsert: true });
    actionLogWithList(userId, 'ShopMember', roadmapIdList.map(o => ({ id: o, model: 'Roadmap' })), 'setRoadmapProfit');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/securityCheckPartment/checkTruckPass.js

```js
import { ShopMemberModel, TruckModel, OrderModel, DriverModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    driverId,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_SECURITY_CHECK_PARTMENT)) {
        return { msg: '权限不足' };
    }
    const truck = await TruckModel.findOne({ driverId, shopId: member.shopId, state: { $in: [ CONSTANTS.TS_READY_ENTER_WAREHOUSE, CONSTANTS.TS_READY_EXIT_WAREHOUSE ] } });
    if (!truck) {
        return { msg: '该货车未完成审核或者未付搬运款' };
    }
    if (truck.state === CONSTANTS.TS_READY_ENTER_WAREHOUSE) {
        await truck.update({ state: CONSTANTS.TS_READY_SEARCH_CARRY_PARTMENT });
    } else {
        await OrderModel.update({ _id: { $in: truck.orderList } }, { 'stateList.0.state': CONSTANTS.OS_ON_THE_WAY }, { multi: true });
        await truck.update({ state: CONSTANTS.TS_ON_THE_WAY });
    }
    const doc = await DriverModel.findById(driverId);

    actionLog(userId, 'ShopMember', driverId, 'Driver', 'checkTruckPass');

    return { success: true, context: {
        isEnter: truck.state === CONSTANTS.TS_READY_ENTER_WAREHOUSE,
        plateNo: truck.plateNo,
        driver: doc,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/shipper/getNeedExamineTruckList.js

```js
import { TruckModel, ShopMemberModel } from '../../../../models';
import { getKeywordCriteriaForTruck } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, keyword, fromPC, pageNo, pageSize }) => {
    const member = await ShopMemberModel.findById(userId);
    const criteria = getKeywordCriteriaForTruck(keyword, { shopId: member.shopId, state: CONSTANTS.TS_READY_EXAMINE });
    const count = (fromPC && pageNo === 0) ? await TruckModel.count(criteria) : undefined;
    const query = TruckModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        plateNo: 1,
        drivingLicense: 1,
        truckType: 1,
        driverId: 1,
        locationList: 1,
        insuanceMount: 1,
        createTime: 1,
    })
    .populate({
        path: 'driverId',
        select: { name: 1, phone: 1, licenseNo: 1 },
    });

    return { success: true, context: {
        count,
        truckList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/shipper/getShipperDetail.js

```js
import { ShipperModel, ShipperInBranchShopModel } from '../../../../models';

export default async ({ userId, shipperId, shopId }) => {
    let shipper = await ShipperModel.findById(shipperId)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });
    if (!shipper) {
        return { msg: '没有该物流公司' };
    }
    shipper = shipper.toObject();
    let inShop = await ShipperInBranchShopModel.findOne({ shipperId, shopId });
    if (!inShop) {
        return { msg: '该物流公司没有入住该分店' };
    }
    inShop = inShop.toObject();
    shipper = { ...shipper, ...inShop };

    return { success: true, context: shipper };
};

```

* PDShopServer/project/App/routers/posts/shop/shipper/getShipperList.js

```js
import mongoose from 'mongoose';
import { ShipperInBranchShopModel, ShopMemberModel } from '../../../../models';
import { getKeywordCriteriaForShipper } from '../../../../utils';

export default async ({ userId, shopId, keyword, fromPC, pageNo, pageSize }) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { isMasterShop: 1 },
    });
    if (!member.shop.isMasterShop) {
        shopId = member.shop.id;
    }
    const criteria = shopId ? { shopId: new mongoose.Types.ObjectId(shopId) } : {};
    let count;
    if (fromPC && pageNo === 0) {
        const countResults = await ShipperInBranchShopModel.aggregate()
        .match(criteria)
        .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
        .project({
            name: { $arrayElemAt: ['$shipper.name', 0] },
        })
        .match(getKeywordCriteriaForShipper(keyword))
        .group({
            _id: null,
            total: { $sum: 1 },
        });
        count = countResults.length ? countResults[0].total : 0;
    }

    const docs = await ShipperInBranchShopModel.aggregate()
    .match(criteria)
    .lookup({ from: 'shippers', localField: 'shipperId', foreignField: '_id', as: 'shipper' })
    .project({
        id: { $arrayElemAt: ['$shipper._id', 0] },
        name: { $arrayElemAt: ['$shipper.name', 0] },
        chairManId: { $arrayElemAt: ['$shipper.chairManId', 0] },
        acountAmount: { $arrayElemAt: ['$shipper.acountAmount', 0] },
        capital: { $arrayElemAt: ['$shipper.capital', 0] },
        shipperType: { $arrayElemAt: ['$shipper.shipperType', 0] },
        shopId: 1,
        totalBondAmount: 1,
        remainBondAmount: 1,
    })
    .match(getKeywordCriteriaForShipper(keyword))
    .lookup({ from: 'shops', localField: 'shopId', foreignField: '_id', as: 'shop' })
    .lookup({ from: 'clients', localField: 'chairManId', foreignField: '_id', as: '_chairMan' })
    .project({
        _id: 0,
        shopId: { $arrayElemAt: ['$shop._id', 0] },
        shopName: { $arrayElemAt: ['$shop.name', 0] },
        chairMan: { name: { $arrayElemAt: ['$_chairMan.name', 0] }, phone: { $arrayElemAt: ['$_chairMan.phone', 0] } },
        id: 1,
        name: 1,
        shipperType: 1,
        chairManId: 1,
        acountAmount: 1,
        capital: 1,
        totalBondAmount: 1,
        remainBondAmount: 1,
    });

    return { success: true, context: {
        count,
        shipperList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/shipper/modifyShipper.js

```js
import _ from 'lodash';
import { ShipperModel, ShipperInBranchShopModel, ShopMemberModel, MediaModel } from '../../../../models';
import { getMediaId, hasAuthority, omitNil, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    shipperId,
    name, // 物流公司名称
    image, // 物流公司背景图片
    sign, // 物流公司签名
    phoneList, // 物流公司联系电话
    address, // 商铺地址
    capital, // 注册资金
    legalName, // 法人姓名
    legalPhone, // 法人电话
    legalIDCard, // 法人身份证
    bondCompanyList, //担保公司列表
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_SHIPPER)) {
        return { msg: '你没有修改物流公司信息的权限' };
    }

    let _image = getMediaId(image);
    let _legalIDCard = _.map(legalIDCard, o => getMediaId(o));
    let _bondCompanyList = _.map(bondCompanyList, item => {
        item.certificate = _.map(item.certificate, o => getMediaId(o));
        item.legalIDCard = _.map(item.legalIDCard, o => getMediaId(o));
        return item;
    });

    const totalBondAmount = _.sumBy(_bondCompanyList, o => o.bondAmount) || 0;
    let inShop = await ShipperInBranchShopModel.findOne({ shipperId, shopId: member.shopId });
    if (!inShop) {
        return { msg: '该物流公司没有入住分店' };
    }
    await inShop.update(omitNil({
        bondCompanyList: bondCompanyList && _bondCompanyList,
        totalBondAmount: bondCompanyList && totalBondAmount,
        remainBondAmount: bondCompanyList && (totalBondAmount - inShop.usedBondAmount),
    }));
    let shipper = await ShipperModel.findByIdAndUpdate(shipperId, omitNil({
        name,
        image: _image,
        sign,
        phoneList,
        address,
        capital,
        legalName,
        legalPhone,
        legalIDCard: legalIDCard && _legalIDCard,
    }));

    MediaModel._updateRef(
        { [_image]: 1 },
        { [image && shipper.image]: -1 },
        ..._legalIDCard.map(item => ({ [item]: 1 })),
        ...(legalIDCard ? shipper.legalIDCard : []).map(item => ({ [item]: -1 })),
        ..._.flatten(_bondCompanyList.map(item => {
            const list = [...item.certificate, ...item.legalIDCard];
            return list.map(o => ({ [o]: 1 }));
        })),
        ..._.flatten((bondCompanyList ? inShop.bondCompanyList : []).map(item => {
            const list = [...item.certificate, ...item.legalIDCard];
            return list.map(o => ({ [o]: -1 }));
        })),
    );

    shipper = await ShipperModel.findById(shipperId)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });
    shipper = shipper.toObject();
    inShop = await ShipperInBranchShopModel.findOne({ shipperId, shopId: member.shopId });
    inShop = inShop.toObject();
    shipper = { ...shipper, ...inShop, shopId: member.shopId, id: shipper.id };

    actionLog(userId, 'ShopMember', shipperId, 'Shipper', 'modifyShipper');

    return { success: true, context: shipper };
};

```

* PDShopServer/project/App/routers/posts/shop/shipper/passExamineTruck.js

```js
import { TruckModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({ userId, truckId }) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_TO_EXAMINE_TRUCK)) {
        return { msg: '你没有审核货车的权限' };
    }

    const truck = await TruckModel.findById(truckId);
    if (!truck) {
        return { msg: '没有该货车' };
    }
    if (truck.state !== CONSTANTS.TS_READY_EXAMINE) {
        return { msg: '该货车无需再审核' };
    }
    await truck.update({ state: CONSTANTS.TS_READY_ENTER_WAREHOUSE });

    actionLog(userId, 'ShopMember', truckId, 'Truck', 'passExamineTruck');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/shipper/removeBondCompany.js

```js
import _ from 'lodash';
import { ShipperModel, MediaModel } from '../../../../models';
import { actionLog } from '../../../../utils';

export default async ({
    userId,
    shipperId,
    name, //担保公司名称
}) => {
    const doc = await ShipperModel.findById(shipperId);
    if (!doc) {
        return { msg: '没有找到该物流公司' };
    }
    const company = _.find(doc.bondCompanyList, item => item.name === name);
    if (!company) {
        return { msg: '没有找到该担保公司' };
    }
    let certificate_ = company.certificate;
    let legalIDCard_ = company.legalIDCard;
    doc.bondCompanyList = _.reject(doc.bondCompanyList, item => item.name === name);
    doc.payAmount -= company.amount;
    await doc.save();

    MediaModel._updateRef(
        ...certificate_.map(item => ({ [item]: -1 })),
        ...legalIDCard_.map(item => ({ [item]: -1 })),
    );

    actionLog(userId, 'ShopMember', shipperId, 'Shipper', 'removeBondCompany');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/shipper/setBondCompany.js

```js
import _ from 'lodash';
import { ShipperModel, MediaModel } from '../../../../models';
import { getMediaId, actionLog } from '../../../../utils';

export default async ({
    userId,
    shipperId,
    name, // 担保公司名称
    phone, // 担保公司电话
    address, // 担保公司背景图片
    certificate, // 担保公司签名
    amount, // 担保公司承担的保额
    legalName, // 担保公司法人姓名
    legalIDCard, //担保公司法人身份证书
}) => {
    let _certificate = certificate.map(o => getMediaId(o));
    let _legalIDCard = legalIDCard.map(o => getMediaId(o));

    const doc = await ShipperModel.findById(shipperId);
    if (!doc) {
        return { msg: '没有找到该物流公司' };
    }
    const bondCompany = {
        name,
        phone,
        address,
        certificate: _certificate,
        amount,
        legalName,
        legalIDCard: _legalIDCard,
    };
    let certificate_ = [], legalIDCard_ = [];
    const company = _.find(doc.bondCompanyList, item => item.name === name);
    if (!company) {
        doc.bondCompanyList.push(bondCompany);
        doc.payAmount += amount;
    } else {
        certificate_ = company.certificate;
        legalIDCard_ = company.legalIDCard;
        doc.payAmount += amount - company.amount;
        Object.assign(company, bondCompany);
    }
    await doc.save();

    MediaModel._updateRef(
        ..._certificate.map(item => ({ [item]: 1 })),
        ..._legalIDCard.map(item => ({ [item]: 1 })),
        ...certificate_.map(item => ({ [item]: -1 })),
        ...legalIDCard_.map(item => ({ [item]: -1 })),
    );

    actionLog(userId, 'ShopMember', shipperId, 'Shipper', 'setBondCompany');

    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/statistics/getStatistics.js

```js
import { OrderModel, ShopMemberModel } from '../../../../models';
import CONSTANTS from '../../../../constants';
import { hasAuthority } from '../../../../utils';
import mongoose from 'mongoose';
import moment from 'moment';
import _ from 'lodash';
const MAX_DAYS = 30;

export default async ({ userId, shopId, days = 7 }) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { isMasterShop: 1 },
    });
    if (member.partmentId || !hasAuthority(member, CONSTANTS.AH_LOOK_STATISTICS)) {
        return { msg: '你没有查看统计的权限' };
    }
    if (!member.shop.isMasterShop) {
        shopId = member.shop.id;
    }

    days = (days > MAX_DAYS ? MAX_DAYS : days < 1 ? 1 : days) + 1;
    const now = moment().startOf('d').add(1, 'd');
    const timeList = _.map(_.rangeRight(-days), o => now.clone().add(o, 'd'));
    const getCond = (name, value) => _.dropRight(_.map(timeList, (o, k) => timeList[k + 1] && { [name + k]: { $sum: { $cond: { if: { $and: [{ $gte: ['$placeOrderTime', o.toDate()] }, { $lt: ['$placeOrderTime', timeList[k + 1].toDate()] }] }, then: value, else: 0 } } } }));

    const cond = [
        ...getCond('count', 1),
        ...getCond('isReachPay', '$isReachPay'),
        ...getCond('isInsuance', '$isInsuance'),
        ...getCond('insuanceMount', '$insuanceMount'),
        ...getCond('insuanceFee', '$insuanceFee'),
        ...getCond('realFee', '$realFee'),
        ...getCond('totalDesignatedFee', '$totalDesignatedFee'),
        ...getCond('proxyCharge', '$proxyCharge'),
        ...getCond('masterProfit', '$masterProfit'),
        ...getCond('branchProfit', '$branchProfit'),
        ...getCond('weight', '$weight'),
        ...getCond('size', '$size'),
    ];

    const daysInfo = await OrderModel.aggregate()
    .match({
        ...(shopId ? { shopId: new mongoose.Types.ObjectId(shopId) } : {}),
        placeOrderTime: { $gt: timeList[0].toDate() },
        'stateList.0.state': { $gte: CONSTANTS.OS_READY_STOCK },
    })
    .project({
        placeOrderTime: 1,
        isReachPay: { $cond: { if: '$isReachPay', then: 1, else: 0 } },
        isInsuance: { $cond: { if: '$isInsuance', then: 1, else: 0 } },
        insuanceMount: 1,
        insuanceFee: 1,
        realFee: 1,
        totalDesignatedFee: 1,
        proxyCharge: 1,
        masterProfit: 1,
        branchProfit: 1,
        weight: 1,
        size: 1,
    })
    .group({
        _id: null,
        ..._.reduce(cond, (r, o) => Object.assign(r, o)),
    });

    daysInfo[0] && (delete daysInfo[0]._id);
    const context = _.transform(daysInfo[0], (r, v, k) => {
        const key = k.replace(/\d+/, '');
        r[key] || (r[key] = []);
        r[key].push(v);
    });

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/shop/getOwnShopDetail.js

```js
import { ShopModel, ShopMemberModel } from '../../../../models';

export default async ({ userId }) => {
    const user = await ShopMemberModel.findById(userId);
    if (!user) {
        return { msg: '没有该用户' };
    }
    const shop = await ShopModel.findById(user.shopId)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });
    if (!shop) {
        return { msg: '没有该物流超市' };
    }
    return { success: true, context: shop };
};

```

* PDShopServer/project/App/routers/posts/shop/shop/modifyOwnShop.js

```js
import { ShopModel, ShopMemberModel, MediaModel } from '../../../../models';
import { getMediaId, omitNil, hasAuthority, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    name, // 物流超市名称
    image, // 物流超市背景图片
    sign, // 物流超市签名
    phoneList, // 物流超市联系电话
    address, // 商铺地址
    location, // // 经纬度定位
}) => {
    const member = await ShopMemberModel.findById(userId)
    .populate({
        path: 'shopId',
        select: { isMasterShop: 1 },
    });
    if (!hasAuthority(member, CONSTANTS.AH_MODIFY_OWN_SHOP)) {
        return { msg: `你没有权限修改${member.shop.isMasterShop ? '四面通总部' : '物流超市'}信息` };
    }

    let _image = getMediaId(image);
    const doc = await ShopModel.findByIdAndUpdate(member.shop.id, omitNil({
        name,
        image: _image,
        sign,
        phoneList,
        address,
        location,
    }));
    if (!doc) {
        return { msg: '修改失败' };
    }
    MediaModel._updateRef(
        { [_image]: 1 },
        { [image && doc.image]: -1 },
    );

    const context = await ShopModel.findById(member.shop.id)
    .populate({
        path: 'chairManId',
        select: { name: 1, phone: 1 },
    });

    actionLog(userId, 'ShopMember', member.shop.id, 'Shop', 'modifyOwnShop');

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/warehousePartment/getLoadingOrderList.js

```js
import { ShopMemberModel, TruckModel } from '../../../../models';
import CONSTANTS from '../../../../constants';
import _ from 'lodash';

export default async ({ userId }) => {
    const member = await ShopMemberModel.findById(userId);
    const truck = await TruckModel.findOne({
        carryPartmentId: member.partmentId,
        state: { $in: [ CONSTANTS.TS_SCAN_LOADING, CONSTANTS.TS_READY_PAY_FOR_CARRY_PARTMENT ] },
    })
    .populate({
        path: 'unloadAllOrderList',
        select: {
            endPoint: 1,
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            stateList: 1,
            totalNumbers: 1,
        },
    })
    .populate({
        path: 'orderList',
        select: {
            createTime: 1,
            photo: 1,
            endPoint: 1,
            senderPhone: 1,
            senderName: 1,
            receiverPhone: 1,
            receiverName: 1,
            totalNumbers: 1,
            size: 1,
            weight: 1,
        },
    });
    if (!truck) {
        return { msg: '目前没有正在装车的货车' };
    }
    const context = truck.toObject();
    _.forEach(context.unloadAllOrderList, item => {
        item.unloadNumber = (_.find(item.stateList, o => o.state === CONSTANTS.OS_READY_LOAD) || {}).count || 0;
        delete item.stateList;
    });

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/warehousePartment/getOrders.js

```js
import _ from 'lodash';
import { OrderModel } from '../../../../models';
import { getKeywordCriteriaForOrder } from '../../../../utils';
import CONSTANTS from '../../../../constants';
import moment from 'moment';

async function getPageData (userId, type, params) {
    let { keyword, startDate, endDate, fromPC, pageNo, pageSize } = params;
    let criteria = getKeywordCriteriaForOrder(keyword, {
        storeScannerId: userId,
    });
    if (type === 'stored') {
        criteria = { ...criteria, 'stateList.state': CONSTANTS.OS_READY_LOAD };
    } else if (type === 'loaded') {
        criteria = { ...criteria, 'stateList.state': { $gte: CONSTANTS.OS_READY_START_OFF } };
    }
    if (fromPC) {
        if (startDate && endDate) {
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        } else {
            startDate = endDate = moment().format('YYYY-MM-DD');
            criteria = { ...criteria, $and: [{ placeOrderTime: { $gte: moment(startDate).toDate() } }, { placeOrderTime: { $lt: moment(endDate).add(1, 'd').toDate() } }] };
        }
    }
    const count = (fromPC && pageNo === 0) ? await OrderModel.count(criteria) : undefined;
    const query = OrderModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        shopId: 1,
        shipperId: 1,
        senderPhone: 1,
        senderName: 1,
        receiverPhone: 1,
        receiverName: 1,
        name: 1,
        endPoint: 1,
        sendDoorEndPoint: 1,
        isSendDoor: 1,
        createTime: 1,
        placeOrderTime: 1,
        modifyTime: 1,
        payMode: 1,
        receiveMemberId: 1,
        totalNumbers: 1,
        weight: 1,
        size: 1,
        photo: 1,
        warehouse: 1,
        stateList: 1,
        needPayTransportFee: 1,
        needPayInsuanceFee: 1,
    }).populate({
        path: 'receiveMemberId',
        select: { name: 1, phone: 1 },
    }).populate({
        path: 'shipperId',
        select: { name: 1 },
    }).populate({
        path: 'shopId',
        select: { name: 1, address: 1 },
    });

    return {
        count,
        list: docs,
    };
};

// ['待入库', '已入库']
const TYPES = ['stored', 'loaded'];
export default async ({ userId, type, ...params }) => {
    if (_.includes(TYPES, type)) {
        return {
            success: true,
            context: {
                [type]: await getPageData(userId, type, params),
            },
        };
    } else {
        return {
            success: true,
            context: {
                stored: await getPageData(userId, 'stored', params),
                loaded: await getPageData(userId, 'loaded', params),
            },
        };
    }
};

```

* PDShopServer/project/App/routers/posts/shop/warehousePartment/placeStorage.js

```js
import _ from 'lodash';
import { ShopMemberModel, OrderModel, WarehouseModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    orderId,
    count = 1,
    force, //强制入库
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_WARE_HOUSE_PARTMENT)) {
        return { msg: '权限不足' };
    }
    const order = await OrderModel.getOrderFromOriginOrder(member.shopId, orderId);
    if (!order) {
        return { msg: '没有该货单' };
    }
    if (!force) {
        if (!order.warehouse) {
            return { msg: '该货单没有分配仓库，请谨慎入库', retry: true, context: order };
        }
        const warehouseNoList = order.warehouse.split(';');
        const warehouse = await WarehouseModel.findOne({ shopId: member.shopId, houseNo: { $in: warehouseNoList }, houseManId: userId });
        if (!warehouse) {
            return { msg: '你不是该货单分配库管员，请谨慎入库', retry: true, context: order };
        }
    }

    const stateList = order.stateList;
    let item = _.find(stateList, (o) => o.state === CONSTANTS.OS_READY_STOCK);
    if (!item) {
        return { msg: '该货单没有待入库的货物' };
    }
    item.count -= count;
    if (item.count < 0) {
        return { msg: '数量无效' };
    }
    item = _.find(stateList, (o) => o.state === CONSTANTS.OS_READY_LOAD);
    if (item) {
        item.count += count;
    } else {
        stateList.unshift({ state: CONSTANTS.OS_READY_LOAD, count });
    }
    order.stateList = _.filter(stateList, o => o.count > 0);
    order.storeScannerId = userId;
    order.placeStoreTime = Date.now();
    await order.save();

    actionLog(userId, 'ShopMember', orderId, 'Order', 'placeStorage');

    return { success: true, context: order };
};

```

* PDShopServer/project/App/routers/posts/shop/warehouse/createWarehouse.js

```js
import { WarehouseModel, ShopMemberModel } from '../../../../models';
import { hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    shipperList,
    houseManId,
    houseNo,
    maxStoreWeight,
    maxStoreSize,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_CREATE_WAREHOUSE)) {
        return { msg: '你没有创建仓库的权限' };
    }

    let warehouse = await WarehouseModel.findOne({ shopId: member.shopId, houseNo });
    if (warehouse) {
        return { msg: '该仓库号已经存在' };
    }
    warehouse = new WarehouseModel({
        shopId: member.shopId,
        shipperList,
        houseManId,
        houseNo,
        maxStoreWeight,
        maxStoreSize,
    });
    await warehouse.save();

    actionLog(userId, 'ShopMember', warehouse.id, 'Warehouse', 'createWarehouse');

    const context = await WarehouseModel.findById(warehouse.id)
    .select({
        houseManId: 1,
        houseNo: 1,
        maxStoreWeight: 1,
        maxStoreSize: 1,
        orderCount: 1,
        totalNumbers: 1,
        totalWeight: 1,
        totalSize: 1,
        shipperList: 1,
    }).populate({
        path: 'houseManId',
        select: { name: 1, phone: 1 },
    }).populate({
        path: 'shipperList',
        select: { name: 1 },
    });

    return { success: true, context };
};

```

* PDShopServer/project/App/routers/posts/shop/warehouse/getWarehouseDetail.js

```js
import { WarehouseModel } from '../../../../models';

export default async ({
    userId,
    warehouseId,
}) => {
    const doc = await WarehouseModel.findById(warehouseId)
    .populate({
        path: 'houseManId',
        select: { name: 1, phone: 1 },
    })
    .populate({
        path: 'shipperList',
        select: { name: 1 },
    });
    if (!doc) {
        return { msg: '没有该仓库' };
    }

    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shop/warehouse/getWarehouseList.js

```js
import { WarehouseModel } from '../../../../models';
import { getKeywordCriteriaForWarehouse, getOwnShop } from '../../../../utils';

export default async ({ userId, keyword, fromPC, pageNo, pageSize }) => {
    const shopId = await getOwnShop(userId);
    const criteria = getKeywordCriteriaForWarehouse(keyword, { shopId });
    const count = (fromPC && pageNo === 0) ? await WarehouseModel.count(criteria) : undefined;
    const query = WarehouseModel.find(criteria).sort({ createTime: 'desc' }).skip(pageNo * pageSize).limit(pageSize);
    const docs = await query
    .select({
        houseManId: 1,
        houseNo: 1,
        maxStoreWeight: 1,
        maxStoreSize: 1,
        orderCount: 1,
        totalNumbers: 1,
        totalWeight: 1,
        totalSize: 1,
        shipperList: 1,
    })
    .populate({
        path: 'houseManId',
        select: { name: 1, phone: 1 },
    })
    .populate({
        path: 'shipperList',
        select: { name: 1 },
    });

    return { success: true, context: {
        count,
        warehouseList: docs,
    } };
};

```

* PDShopServer/project/App/routers/posts/shop/warehouse/modifyWarehouse.js

```js
import { WarehouseModel } from '../../../../models';
import { omitNil, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    warehouseId,
    shipperList,
    houseManId,
    houseNo,
    maxStoreWeight,
    maxStoreSize,
}) => {
    if (!await hasAuthority(userId, CONSTANTS.AH_MODIFY_WAREHOUSE)) {
        return { msg: '你没有修改仓库的权限' };
    }

    const doc = await WarehouseModel.findByIdAndUpdate(warehouseId, omitNil({
        shipperList,
        houseManId,
        houseNo,
        maxStoreWeight,
        maxStoreSize,
        modifyTime: Date.now(),
    }), { new: true }).select({
        houseManId: 1,
        houseNo: 1,
        maxStoreWeight: 1,
        maxStoreSize: 1,
        orderCount: 1,
        totalNumbers: 1,
        totalWeight: 1,
        totalSize: 1,
        shipperList: 1,
    }).populate({
        path: 'houseManId',
        select: { name: 1, phone: 1 },
    })
    .populate({
        path: 'shipperList',
        select: { name: 1 },
    });
    if (!doc) {
        return { msg: '修改失败' };
    }

    actionLog(userId, 'ShopMember', warehouseId, 'Warehouse', 'modifyWarehouse');

    return { success: true, context: doc };
};

```

* PDShopServer/project/App/routers/posts/shop/warehouse/removeWarehouse.js

```js
import { WarehouseModel, ShopMemberModel } from '../../../../models';
import { authenticatePassword, hasAuthority, actionLog } from '../../../../utils';
import CONSTANTS from '../../../../constants';

export default async ({
    userId,
    warehouseId,
    password,
}) => {
    const member = await ShopMemberModel.findById(userId);
    if (!hasAuthority(member, CONSTANTS.AH_REMOVE_WAREHOUSE)) {
        return { msg: '你没有删除仓库的权限' };
    }
    const error = await authenticatePassword(member, password);
    if (error) {
        return { msg: '密码错误' };
    }
    await WarehouseModel.findByIdAndRemove(warehouseId);

    actionLog(userId, 'ShopMember', warehouseId, 'Warehouse', 'removeWarehouse');
    return { success: true };
};

```

* PDShopServer/project/App/routers/posts/shop/weixinPay/getUnifiedOrder.js

```js
import { ShopMemberModel, WeixinPayModel } from '../../../../models';
import { _F, _T, hasAuthority, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';
import api from '../../libs/weixinPay/api';
import config from '../../../../../config';
import { rechargeForShopMember } from '../../libs/account';

export default async ({
    userId,
    amount, // 单位：元
    body, //商品描述
}, req) => {
    amount = _F(amount);
    if (config.hasNoRealPayment) {
        return rechargeForShopMember({ userId, amount, thirdpartyAccount: 'SIMULATION_NO' });
    }
    amount = amount / config.weixinPayRate;
    if (amount < 1) {
        return { msg: '充值金额必须大于' + config.weixinPayRate / 100 };
    }
    return new Promise(async resolve => {
        const member = await ShopMemberModel.findById(userId);
        if (!hasAuthority(member, CONSTANTS.AH_RECHARGE) || !member.partmentId) {
            return resolve({ msg: '你没有充值的权限' });
        }

        const spbill_create_ip = req.get('X-Real-IP') || req.get('X-Forwarded-For') || req.ip;
        const order = new WeixinPayModel({
            shopMemberId: userId,
            spbill_create_ip,
            body,
            trade_type: 'APP',
            total_fee: amount,
        });

        const params = {
            spbill_create_ip,
            body,
            out_trade_no: _T(order.out_trade_no),
            total_fee: amount,
            trade_type: 'APP',
        };
        const result = await api.unifiedorder(params);
        if (result.success) {
            order.prepay_id = result.context.prepayid;
            order.appid = result.context.appid;
            order.mch_id = result.context.partnerid;
            order.timestamp = result.context.timestamp;
            order.nonce_str = result.context.noncestr;
            order.package = result.context.package;
            await order.save();
        }
        actionLog(userId, 'ShopMember', amount, 'Number', 'getUnifiedOrder');

        return resolve(result);
    });
};

```

* PDShopServer/project/App/routers/posts/shop/weixinPay/getUnifiedOrderForPC.js

```js
import { ShopMemberModel, WeixinPayModel } from '../../../../models';
import { _F, _T, hasAuthority, actionLog } from '../../../../utils/';
import CONSTANTS from '../../../../constants';
import api from '../../libs/weixinPay/api';
import config from '../../../../../config';
import { rechargeForShopMember } from '../../libs/account';

export default async ({
    userId,
    amount, // 单位：元
    body, //商品描述
}, req) => {
    amount = _F(amount);
    if (config.hasNoRealPayment) {
        return rechargeForShopMember({ userId, amount, thirdpartyAccount: 'SIMULATION_NO' });
    }
    amount = amount / config.weixinPayRate;
    if (amount < 1) {
        return { msg: '充值金额必须大于' + config.weixinPayRate / 100 };
    }
    return new Promise(async resolve => {
        const member = await ShopMemberModel.findById(userId);
        if (!hasAuthority(member, CONSTANTS.AH_RECHARGE) || !member.partmentId) {
            return resolve({ msg: '你没有充值的权限' });
        }

        const spbill_create_ip = req.get('X-Real-IP') || req.get('X-Forwarded-For') || req.ip;
        const order = new WeixinPayModel({
            shopMemberId: userId,
            spbill_create_ip,
            body,
            trade_type: 'NATIVE',
            total_fee: amount,
        });

        const params = {
            spbill_create_ip,
            body,
            out_trade_no: _T(order.out_trade_no),
            total_fee: amount,
            trade_type: 'NATIVE',
        };
        const result = await api.unifiedorder(params);
        if (result.success) {
            order.prepay_id = result.context.prepayid;
            order.appid = result.context.appid;
            order.mch_id = result.context.partnerid;
            order.code_url = result.context.code_url;
            await order.save();

            result.context = { qrcode: result.context.code_url };
        }

        actionLog(userId, 'ShopMember', amount, 'Number', 'getUnifiedOrderForPC');

        return resolve(result);
    });
};

```
