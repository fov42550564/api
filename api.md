* PDClient/project/App/modules/baidumap/index.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

import { MapView, MapTypes, Geolocation } from '@remobile/react-native-baidu-map';

const { Button } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '货物所在地',
    },
    getInitialState () {
        return {
            mayType: MapTypes.NORMAL,
            zoom: 15,
            center: {
                longitude: 106.6886,
                latitude: 26.566621,
            },
            trafficEnabled: false,
            baiduHeatMapEnabled: false,
            marker: {
                longitude: 106.6886,
                latitude: 26.566621,
                title: '花果园',
                city:'贵阳市',
                province:'贵州省',
            },
        };
    },
    componentWillMount () {
        Geolocation.getCurrentPosition()
            .then(data => {
                console.warn(JSON.stringify(data));
                this.setState({
                    zoom: 15,
                    marker: {
                        latitude: data.latitude,
                        longitude: data.longitude,
                        title: data.address,
                        city:data.city,
                        province:data.province,
                    },
                    center: {
                        latitude: data.latitude,
                        longitude: data.longitude,
                        rand: Math.random(),
                    },
                });
            })
            .catch(e => {
                console.warn(e, 'error');
            });
    },
    render () {
        const { marker } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.mapTitle}>
                    <Text style={styles.selectPositionText}>{marker.title}</Text>
                    <Button onPress={() => { this.props.confirm(marker); app.pop(); }} style={styles.btnConfirm} >
                        确认货物所在地
                    </Button>
                </View>
                <MapView
                    trafficEnabled={this.state.trafficEnabled}
                    baiduHeatMapEnabled={this.state.baiduHeatMapEnabled}
                    zoom={this.state.zoom}
                    mapType={this.state.mapType}
                    center={this.state.center}
                    marker={this.state.marker}
                    markers={this.state.markers}
                    style={styles.map}
                    onMarkerClick={(data) => {
                        console.warn('onMarkerClick', JSON.stringify(data));
                        Geolocation.reverseGeoCodeGPS(data.position.latitude, data.position.longitude)
                            .then(e => {
                                console.warn(JSON.stringify(e));
                                this.setState({
                                    zoom: 15,
                                    marker: {
                                        latitude: data.position.latitude,
                                        longitude: data.position.longitude,
                                        title: e.address,
                                        city:e.city,
                                        province:e.province,
                                    },
                                });
                            })
                            .catch(e => {
                                console.warn(e, 'error');
                            });
                    }}
                    onMapPoiClick={(data) => {
                        console.warn('onMapPoiClick', JSON.stringify(data));
                        Geolocation.reverseGeoCodeGPS(data.latitude, data.longitude)
                            .then(e => {
                                console.warn(JSON.stringify(e));
                                this.setState({
                                    zoom: 15,
                                    marker: {
                                        latitude: data.latitude,
                                        longitude: data.longitude,
                                        title: e.address,
                                        city:e.city,
                                        province:e.province,
                                    },
                                });
                            })
                            .catch(e => {
                                console.warn(e, 'error');
                            });
                    }}
                    onMapClick={(data) => {
                        console.warn('onMapClick', JSON.stringify(data));
                        Geolocation.reverseGeoCodeGPS(data.latitude, data.longitude)
                            .then(e => {
                                console.warn(JSON.stringify(e));
                                this.setState({
                                    zoom: 15,
                                    marker: {
                                        latitude: data.latitude,
                                        longitude: data.longitude,
                                        title: e.address,
                                        city:e.city,
                                        province:e.province,
                                    },
                                });
                            })
                            .catch(e => {
                                console.warn(e, 'error');
                            });
                    }}

                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'flex-start',
        alignItems: 'center',
        backgroundColor: '#F5FCFF',
    },
    map: {
        width: sr.w,
        height: sr.th * 0.78,
        marginBottom: 50,
        marginTop:10,
        backgroundColor: 'red',
    },
    selectPositionText :{
        paddingVertical:12,
        alignSelf:'center',
        fontSize:16,
    },
    btnConfirm:{
        height:35,
        width:200,
        borderRadius:2,
    },
    mapTitle:{
        alignItems:'center',
    },
});

```

* PDClient/project/App/modules/baidumap/test.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Button,
} = ReactNative;

import { MapView, MapTypes, Geolocation } from '@remobile/react-native-baidu-map';

const SplashScreen = require('@remobile/react-native-splashscreen');

module.exports = React.createClass({
    componentWillMount () {
        SplashScreen.hide();
    },
    getInitialState () {
        return {
            mayType: MapTypes.NORMAL,
            zoom: 15,
            center: {
                longitude: 113.981718,
                latitude: 22.542449,
            },
            trafficEnabled: false,
            baiduHeatMapEnabled: false,
            markers: [{
                longitude: 113.981718,
                latitude: 22.542449,
                title: 'Window of the world',
            }, {
                longitude: 113.995516,
                latitude: 22.537642,
                title: '',
            }],
        };
    },
    render () {
        return (
            <View style={styles.container}>
                <MapView
                    trafficEnabled={this.state.trafficEnabled}
                    baiduHeatMapEnabled={this.state.baiduHeatMapEnabled}
                    zoom={this.state.zoom}
                    mapType={this.state.mapType}
                    center={this.state.center}
                    marker={this.state.marker}
                    markers={this.state.markers}
                    style={styles.map}
                    onMarkerClick={(e) => {
                        console.warn(JSON.stringify(e));
                    }}
                    onMapPoiClick={(e) => {
                        console.warn('onMapPoiClick', JSON.stringify(e));
                        this.setState({
                            zoom: 15,
                            marker: {
                                latitude: e.latitude,
                                longitude: e.longitude,
                                title: e.name,
                            },
                        });
                    }}
                    onMapClick={(data) => {
                        console.warn(JSON.stringify(data));
                        Geolocation.reverseGeoCodeGPS(data.latitude, data.longitude)
                            .then(e => {
                                console.warn(JSON.stringify(e));
                                this.setState({
                                    zoom: 15,
                                    marker: {
                                        latitude: data.latitude,
                                        longitude: data.longitude,
                                        title: e.address,
                                    },
                                });
                            })
                            .catch(e => {
                                console.warn(e, 'error');
                            });
                    }}

                    />
                <View style={styles.row}>
                    <Button title='Normal' onPress={() => {
                        this.setState({
                            mapType: MapTypes.NORMAL,
                        });
                    }}
                        />
                    <Button style={styles.btn} title='Satellite' onPress={() => {
                        this.setState({
                            mapType: MapTypes.SATELLITE,
                        });
                    }}
                        />

                    <Button style={styles.btn} title='Locate' onPress={() => {
                        console.warn('center', this.state.center);
                        Geolocation.getCurrentPosition()
                            .then(data => {
                                console.warn(JSON.stringify(data));
                                this.setState({
                                    zoom: 15,
                                    marker: {
                                        latitude: data.latitude,
                                        longitude: data.longitude,
                                        title: 'Your location',
                                    },
                                    center: {
                                        latitude: data.latitude,
                                        longitude: data.longitude,
                                        rand: Math.random(),
                                    },
                                });
                            })
                            .catch(e => {
                                console.warn(e, 'error');
                            });
                    }}
                        />
                </View>

                <View style={styles.row}>
                    <Button title='Zoom+' onPress={() => {
                        this.setState({
                            zoom: this.state.zoom + 1,
                        });
                    }}
                        />
                    <Button title='Zoom-' onPress={() => {
                        if (this.state.zoom > 0) {
                            this.setState({
                                zoom: this.state.zoom - 1,
                            });
                        }
                    }}
                        />
                </View>

                <View style={styles.row}>
                    <Button title='Traffic' onPress={() => {
                        this.setState({
                            trafficEnabled: !this.state.trafficEnabled,
                        });
                    }}
                        />

                    <Button title='Baidu HeatMap' onPress={() => {
                        this.setState({
                            baiduHeatMapEnabled: !this.state.baiduHeatMapEnabled,
                        });
                    }}
                        />
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'flex-start',
        alignItems: 'center',
        backgroundColor: '#F5FCFF',
    },
    row: {
        flexDirection: 'row',
        height: 40,
    },
    map: {
        width: sr.w,
        height: sr.h / 2,
        marginBottom: 50,
        backgroundColor: 'red',
    },
    closeTouchableHighlight: {
        position:'absolute',
        top:0,
        left:sr.w * 6 / 7 + 3,
        width: 38,
        height: 38,
        marginTop: (sr.h - 200 - 20) / 2,
    },
    closeIcon: {
        width: 38,
        height: 38,
    },
});

```

* PDClient/project/App/modules/login/ForgetPassword.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    TextInput,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button } = COMPONENTS;
const PasswordConfirm = require('./PasswordConfirm');
const TimerMixin = require('react-timer-mixin');

module.exports = React.createClass({
    mixins: [TimerMixin],
    statics: {
        title: '忘记密码',
    },
    getInitialState () {
        return {
            phone:'',
            readSecond:false,
            time:59,
            verifyCode:'',
        };
    },
    doPress () {
        const { phone } = this.state;
        if (!app.utils.checkPhone(phone)) {
            Toast('请填写正确的手机号码');
            return;
        };
        const param = {
            phone,
        };
        POST(app.route.ROUTE_REQUEST_SEND_VERIFY_CODE, param, this.requestSendVerifyCodeSuccess, false);
    },
    timer () {
        this.setState({ readSecond:true });
        this.setTimeout(
            () => {
                this.setState({ time:(this.state.time - 1) });
                if (this.state.time <= 0) {
                    return this.setState({ readSecond:false, time:59 });
                }
                this.timer();
            },
            1000
        );
    },
    requestSendVerifyCodeSuccess (data) {
        if (data.success) {
            this.timer();
        } else {
            Toast(data.msg);
        }
    },
    doNext () {
        const { phone, verifyCode } = this.state;
        if (!app.utils.checkPhone(phone)) {
            Toast('请填写正确的手机号码');
            return;
        };
        if (!verifyCode) {
            Toast('请填写验证码');
            return;
        }
        if (!app.utils.checkVerificationCode(verifyCode)) {
            Toast('请填写正确的格式的验证码');
            return;
        }
        app.push({
            component:PasswordConfirm,
            passProps:{
                phone,
                verifyCode,
            },
        });
    },
    render () {
        const { readSecond, time } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.ItemBgTop}>
                    <View style={styles.infoStyle}>
                        <Image resizeMode='stretch'
                            source={app.img.login_phone}
                            style={styles.icon_add} />
                        <TextInput placeholderTextColor='#aaaaaa'
                            placeholder='请填写手机号码'
                            underlineColorAndroid='transparent'
                            keyboardType='numeric'
                            maxLength={11}
                            onChangeText={(text) => this.setState({ phone: text })}
                            style={styles.itemNameText} />
                    </View>
                </View>
                <View style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <TextInput placeholderTextColor='#aaaaaa'
                            placeholder='请填写验证码'
                            maxLength={6}
                            keyboardType='phone-pad'
                            underlineColorAndroid='transparent'
                            onChangeText={(text) => this.setState({ verifyCode: text })}
                            style={styles.itemNameText} />
                        <TouchableOpacity
                            disabled={readSecond}
                            style={styles.btnBind}
                            onPress={this.doPress}>
                            {
                                    !readSecond ?
                                        <Text style={readSecond ? styles.btnBindTextGray : styles.btnBindText}>获取验证码</Text>
                                    :
                                        <Text style={readSecond ? styles.btnBindTextGray : styles.btnBindText}>{time}s后重新发送</Text>
                                }
                        </TouchableOpacity>
                    </View>
                </View>
                <Button onPress={this.doNext}
                    style={styles.btnNext}
                    textStyle={styles.btnNextText} >
                    下一步
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    ItemBgTop: {
        height: 45,
        marginTop:10,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
    },
    ItemBg: {
        height: 45,
        marginTop:1,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
    },
    infoStyle: {
        width:sr.w,
        flexDirection: 'row',
        alignItems:'center',
        justifyContent: 'space-between',
    },
    icon_add:{
        width:25,
        height:25,
        marginLeft: 8,
    },
    itemNameText: {
        flex:1,
        fontSize: 14,
        color: '#444444',
        marginLeft: 10,
    },

    btnNext:{
        marginTop:50,
        height:45,
        width:300,
        marginLeft:(sr.w - 300) / 2,
        borderRadius:4,
        backgroundColor:'#c81622',
    },
    btnNextText:{
        color:'#ffffff',
        fontSize:16,
        fontWeight:'500',
    },
    btnBind:{
        justifyContent:'center',
        alignItems:'center',
        borderLeftWidth:1,
        borderColor:'#f4f4f4',
        height:45,
    },
    btnBindText:{
        color:'#FF981A',
        fontSize:16,
        fontWeight:'500',
        width:sr.w - 254,
        textAlign:'center',
    },
    btnBindTextGray:{
        color:'#999999',
        backgroundColor:'#fafafa',
        borderLeftWidth:1,
        borderColor:'#f4f4f4',
        fontSize:16,
        fontWeight:'500',
        textAlign:'center',
        width:sr.w - 254,
        flex:1,
        textAlignVertical:'center',
    },
});

```

* PDClient/project/App/modules/login/Login.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    Image,
    View,
    TextInput,
    ListView,
    Text,
    TouchableOpacity,
} = ReactNative;

const ForgetPassword = require('./ForgetPassword');
const Register = require('./Register');
const ReceivePoint = require('../receivePoint');
const NormalUser = require('../normalUser');
const LogisticsCompany = require('../logisticsCompany');
const ShortLogisticsCompany = require('../shortLogisticsCompany');

const { Button } = COMPONENTS;

const WeixinQQPanel = React.createClass({
    render () {
        return (
            <View style={styles.thirdpartyContainer}>
                <View style={styles.sepratorContainer}>
                    <View style={styles.sepratorLine} />
                    <Text style={styles.sepratorText} >{app.isandroid ? '    ' : ''}或者您也可以</Text>
                </View>
                <View style={styles.thirdpartyButtonContainer}>
                    {
                        !!this.props.weixininstalled &&
                        <View style={styles.thirdpartyLeftButtonContainer}>
                            <Image
                                resizeMode='stretch'
                                source={app.img.login_weixin_button}
                                style={styles.image_button}
                                />
                            <Text style={styles.image_button_text}>微信登录</Text>
                        </View>
                    }
                    {
                        !!this.props.qqinstalled &&
                        <View style={styles.thirdpartyRightButtonContainer}>
                            <Image
                                resizeMode='stretch'
                                source={app.img.login_qq_button}
                                style={styles.image_button}
                                />
                            <Text style={styles.image_button_text}>QQ登录</Text>
                        </View>
                    }
                </View>
            </View>
        );
    },
});

const NoWeixinQQPanel = React.createClass({
    render () {
        return (
            <View style={styles.thirdpartyContainer2}>
                <Text style={[styles.thirdpartyContainer2_text, { color: app.setting.THEME_COLOR }]} />
            </View>
        );
    },
});

module.exports = React.createClass({
    mixins: [SceneMixin],
    statics: {
        title: '',
    },
    componentDidMount () {
        app.toggleNavigationBar(true);
    },
    doLogin () {
        const { phone, password } = this.state;
        if (!app.utils.checkPhone(phone)) {
            Toast('手机号码不是有效的手机号码');
            return;
        }
        if (!app.utils.checkPassword(password)) {
            Toast('密码必须有6-20位的数字，字母，下划线组成');
            return;
        }
        const param = {
            phone,
            password,
        };
        app.showLoading();
        POST(app.route.ROUTE_LOGIN, param, this.doLoginSuccess, this.doLoginError);
    },
    doLoginSuccess (data) {
        if (data.success) {
            app.personal.info.userId = data.context.userId;
            app.login.savePhone(this.state.phone);
            this.getPersonalInfo();
            this.getRemainAmount();
        } else {
            Toast(data.msg);
            app.hideLoading();
        }
    },
    getRemainAmount () {
        const param = {
            userId: app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_REMAIN_AMOUNT, param, this.getRemainAmountSuccess, true);
    },
    getRemainAmountSuccess (data) {
        if (data.success) {
            const context = data.context;
            app.personal.setRemainAmount(context.amount);
        } else {
            Toast(data.msg);
            app.personal.setRemainAmount(0);
        }
    },
    doLoginError (error) {
        app.hideLoading();
    },
    doShowForgetPassword () {
        app.push({
            component: ForgetPassword,
            passProps: {
                phone: this.state.phone,
            },
        });
    },
    doShowRegister () {
        app.push({
            component: Register,
            passProps: {
                phone: this.state.phone,
                changeToLoginPanel: this.changeToLoginPanel,
            },
        });
    },
    getPersonalInfo () {
        const param = {
            userId: app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_PERSONAL_INFO, param, this.getPersonalInfoSuccess, this.getPersonalInfoError);
    },
    getPersonalInfoSuccess (data) {
        if (data.success) {
            const context = data.context;
            app.personal.set(context);
            app.navigator.replace({
                component: app.authority.info.currentRole == '收货点' ? ReceivePoint : app.authority.info.currentRole == '物流公司' ? app.personal.info.shipper.shipperType == 0 ? LogisticsCompany : ShortLogisticsCompany : NormalUser,
            });
            app.personal.setNeedLogin(false);
        } else {
            Toast(data.msg);
        }
        app.hideLoading();
    },
    getPersonalInfoError (error) {
        app.hideLoading();
    },
    getInitialState () {
        const ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            phone: app.login.list[0] || '',
            password: '',
            dataSource: ds.cloneWithRows(app.login.list),
            showList: false,
            weixininstalled: false,
            qqinstalled: false,
        };
    },
    changeToLoginPanel (phone) {
        this.setState({ phone });
    },
    onPhoneTextInputLayout (e) {
        const frame = e.nativeEvent.layout;
        this.listTop = frame.y + frame.height;
    },
    renderRow (text) {
        return (
            <TouchableOpacity onPress={() => this.setState({ phone: text, showList:false })}>
                <View style={styles.itemTextContainer}>
                    <Text style={styles.itemText}>
                        {text}
                    </Text>
                </View>
            </TouchableOpacity>
        );
    },
    renderSeparator (sectionID, rowID) {
        return (
            <View style={styles.separator} key={sectionID + rowID} />
        );
    },
    onFocus () {
        this.setState({ showList: this.state.dataSource.getRowCount() > 0 && this.state.dataSource.getRowData(0, 0).length < 11 });
    },
    onBlur () {
        this.setState({ showList: false });
    },
    onPhoneTextChange (text) {
        const dataSource = this.state.dataSource;
        if (!(/[!~@#$*+%]+/.test(text))) {
            const newData = _.filter(app.login.list, (item) => { const reg = new RegExp('^' + text + '.*'); return reg.test(item); });
            this.setState({
                phone: text,
                dataSource: dataSource.cloneWithRows(newData),
                showList: newData.length > 0 && text.length < 11,
            });
        } else {
            this.setState({
                phone: text
            });
        }
    },
    render () {
        const row = this.state.dataSource.getRowCount();
        const listHeight = row > 4 ? styles.listHeightMax : row < 2 ? styles.listHeightMin : null;
        return (
            <View style={styles.container}>
                <View style={styles.logoContainer}>
                    <Image
                        resizeMode='stretch'
                        source={app.img.login_logo}
                        style={styles.logo}
                        />
                </View>
                <View style={styles.loginBox}>
                    <View
                        style={styles.phoneContainer}
                        onLayout={this.onPhoneTextInputLayout}>
                        <Image
                            resizeMode='stretch'
                            source={app.img.login_phone}
                            style={styles.input_icon}
                            />
                        <TextInput
                            underlineColorAndroid='transparent'
                            placeholder='您的手机号码'
                            maxLength={11}
                            onChangeText={this.onPhoneTextChange}
                            defaultValue={this.state.phone}
                            style={styles.text_input}
                            keyboardType='phone-pad'
                            onFocus={this.onFocus}
                            onBlur={this.onBlur}
                            />
                    </View>
                    <View style={styles.listView} />
                    <View style={styles.passwordContainer}
                        >
                        <Image
                            resizeMode='stretch'
                            source={app.img.login_password}
                            style={styles.input_icon}
                            />
                        <TextInput
                            underlineColorAndroid='transparent'
                            placeholder='您的密码'
                            secureTextEntry
                            onChangeText={(text) => this.setState({ password: text })}
                            defaultValue={this.state.password}
                            style={styles.text_input}
                            />
                    </View>
                </View>

                <View style={styles.un_loading} >
                    <TouchableOpacity onPress={this.doShowRegister}>
                        <Text style={styles.register}>新用户注册</Text>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={this.doShowForgetPassword}>
                        <Text style={styles.forgetPassword}>忘记密码</Text>
                    </TouchableOpacity>
                </View>

                <Button onPress={this.doLogin} style={styles.btnLogin} textStyle={styles.btnLoginText}>登录</Button>

                {this.state.qqinstalled || this.state.weixininstalled ? <WeixinQQPanel qqinstalled={this.state.qqinstalled} weixininstalled={this.state.weixininstalled} /> : <NoWeixinQQPanel />}
                {
                    this.state.showList &&
                    <ListView
                        initialListSize={1}
                        enableEmptySections
                        dataSource={this.state.dataSource}
                        keyboardShouldPersistTaps='always'
                        renderRow={this.renderRow}
                        renderSeparator={this.renderSeparator}
                        style={[styles.list, { top: this.listTop + 135 }, listHeight]}
                        />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor:'#ffffff',
    },
    logoContainer: {
        height: 120,
        justifyContent: 'center',
        alignItems: 'center',
        marginTop:18,
    },
    logo: {
        width: 85.5,
        height: 96,
    },
    listView:{
        height:1,
        width: 300,
        backgroundColor:'#dddddd',
    },
    loginBox:{
        marginLeft: (sr.w - 300) / 2,
        width: 300,
        height: 90,
        borderRadius:4,
        borderColor:'#dddddd',
        borderWidth:1,
        alignItems:'center',
        justifyContent:'center',
    },
    phoneContainer: {
        width: 300,
        height: 45,
        flexDirection: 'row',
        overflow: 'hidden',
        alignItems:'center',
    },
    passwordContainer:{
        width: 300,
        height: 45,
        flexDirection: 'row',
        alignItems:'center',
    },
    input_icon: {
        width: 28,
        height: 28,
        marginLeft: 10,
        marginRight: 10,
    },
    text_input: {
        height:40,
        width: 200,
        padding:0,
        fontSize:14,
        alignSelf: 'center',
    },
    un_loading:{
        flexDirection: 'row',
        justifyContent: 'space-between',
        marginTop:12,
    },
    register:{
        color:'#fd7b1b',
        marginLeft: (sr.w - 300) / 2,
    },
    forgetPassword:{
        color:'#f41728',
        marginRight: (sr.w - 300) / 2,
    },

    btnLogin: {
        height: 46,
        width: 300,
        marginLeft: (sr.w - 300) / 2,
        marginTop: 60,
        backgroundColor: '#c81622',
        borderRadius:4,
    },
    btnLoginText: {
        fontSize: 18,
        fontWeight: '600',
        color: '#FFFFFF',
    },
    thirdpartyContainer: {
        flex:1,
    },
    sepratorContainer: {
        height: 30,
        alignItems:'center',
        justifyContent: 'center',
        marginTop: 50,
    },
    sepratorLine: {
        top: 10,
        height: 2,
        width: sr.w - 20,
        backgroundColor: '#858687',
    },
    sepratorText: {
        backgroundColor:'#EEEEEE',
        color: '#A3A3A4',
        paddingHorizontal: 10,
    },
    thirdpartyButtonContainer: {
        marginTop: 30,
        height: 120,
        flexDirection: 'row',
    },
    thirdpartyLeftButtonContainer: {
        flex:1,
        alignItems:'center',
    },
    thirdpartyRightButtonContainer: {
        flex:1,
        alignItems:'center',
    },
    image_button: {
        width: 80,
        height: 80,
        margin: 10,
    },
    image_button_text: {
        color: '#4C4D4E',
        fontSize: 16,
    },
    thirdpartyContainer2: {
        marginTop: 30,
        height: 200,
        alignItems:'center',
        justifyContent: 'flex-end',
    },
    thirdpartyContainer2_text: {
        fontSize: 18,
        marginBottom:60,
    },
    list: {
        position: 'absolute',
        backgroundColor: 'rgba(255, 255, 255, 0.6)',
        borderWidth: 1,
        borderRadius: 5,
        borderColor: '#D7D7D7',
        width: 150,
        left: 80,
        paddingLeft: 10,
    },
    listHeightMin: {
        height: 30,
    },
    listHeightMax: {
        height: 184,
    },
    itemTextContainer: {
        height: 30,
        justifyContent: 'center',
    },
    itemText: {
        fontSize: 16,
    },
    separator: {
        backgroundColor: '#DDDDDD',
        height: 1,
    },
});

```

* PDClient/project/App/modules/login/PasswordConfirm.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    TextInput,
    Image,
} = ReactNative;

const { Button } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '忘记密码',
    },
    getInitialState () {
        return {
            newPassword:'',
            rePassword:'',
        };
    },
    doPress () {
        const { newPassword, rePassword } = this.state;
        const { phone, verifyCode } = this.props;
        if (newPassword != rePassword) {
            Toast('两次输入密码不一致');
            return;
        }
        const param = {
            phone,
            verifyCode,
            password:newPassword,
        };
        POST(app.route.ROUTE_FIND_PASSWORD, param, this.findPasswordSuccess, false);
    },
    findPasswordSuccess (data) {
        if (data.success) {
            Toast('密码设置成功');
            app.navigator.popToTop();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        return (
            <View style={styles.container}>
                <View style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <Image resizeMode='stretch'
                            source={app.img.login_password}
                            style={styles.icon_add} />
                        <TextInput placeholderTextColor='#aaaaaa'
                            placeholder='请输入新密码'
                            underlineColorAndroid='transparent'
                            onChangeText={(text) => this.setState({ newPassword: text })}
                            style={styles.itemNameText} />
                    </View>
                </View>
                <View style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <Image resizeMode='stretch'
                            source={app.img.login_password_check}
                            style={styles.icon_add} />
                        <TextInput placeholderTextColor='#aaaaaa'
                            placeholder='确认新密码'
                            underlineColorAndroid='transparent'
                            onChangeText={(text) => this.setState({ rePassword: text })}
                            style={styles.itemNameText} />
                    </View>
                </View>
                <Button onPress={this.doPress}
                    style={styles.btnSubmit}
                    textStyle={styles.btnSubmitText} >
                    确认修改
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    ItemBg: {
        height: 45,
        flexDirection: 'row',
        backgroundColor:'white',
    },
    infoStyle: {
        width:sr.w,
        flexDirection: 'row',
        alignItems:'center',
        height: 45,
    },
    icon_add:{
        width:25,
        height:25,
        marginLeft: 8,
    },
    itemNameText: {
        width: 300,
        fontSize:14,
        marginLeft:10,
        marginTop:5,
        padding:0,
    },
    btnSubmit:{
        marginTop:50,
        height:45,
        width:300,
        marginLeft:(sr.w - 300) / 2,
        borderRadius:4,
        backgroundColor:'#c81622',
    },
    btnSubmitText:{
        color:'#ffffff',
        fontSize:16,
        fontWeight:'500',
    },
});

```

* PDClient/project/App/modules/login/Register.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    Image,
    Text,
    View,
    TextInput,
    TouchableOpacity,
} = ReactNative;

const TimerMixin = require('react-timer-mixin');
const { Button, WebviewMessageBox } = COMPONENTS;

module.exports = React.createClass({
    mixins: [TimerMixin],
    statics: {
        title: '手机注册',
    },
    doRegister () {
        const { protocalRead, phone, password, rePassword, verifyCode } = this.state;
        if (!protocalRead) {
            Toast('注册前请先阅读用户协议');
            return;
        }
        if (!app.utils.checkPhone(phone)) {
            Toast('请填写正确的手机号码');
            return;
        }
        if (!app.utils.checkPassword(password)) {
            Toast('密码必须由 6-20 位的数字或，字母，下划线组成');
            return;
        }
        if (password !== rePassword) {
            Toast('两次输入的密码不一致');
            return;
        }
        if (!verifyCode) {
            Toast('请填写验证码');
            return;
        }
        if (!app.utils.checkVerificationCode(verifyCode)) {
            Toast('请填写正确的格式的验证码');
            return;
        }
        // if (!app.utils.checkMailAddress(email)) {
        //     Toast('邮箱地址不规范，请重新输入');
        //     return;
        // }
        const param = {
            phone,
            verifyCode,
            password,
        };
        POST(app.route.ROUTE_REGISTER, param, this.doRegisterSuccess, true);
    },
    doRegisterSuccess (data) {
        if (data.success) {
            Toast('注册成功');
            this.props.changeToLoginPanel(this.state.phone);
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    doShowProtocal () {
        app.showModal(
            <WebviewMessageBox webAddress={app.route.ROUTE_USER_LICENSE} title={CONSTANTS.APP_NAME + '用户协议'} />
        );
    },
    getInitialState () {
        return {
            phone: '',
            password: '',
            rePassword: '',
            verifyCode: '',
            protocalRead: true,
            overlayShow:false,
            readSecond:false,
            time:parseInt(59),
        };
    },
    changeProtocalState () {
        this.setState({ protocalRead: !this.state.protocalRead });
    },
    doPress () {
        const { phone } = this.state;
        if (!app.utils.checkPhone(phone)) {
            Toast('请填写正确的手机号码');
            return;
        };
        const param = {
            phone,
        };
        POST(app.route.ROUTE_REQUEST_SEND_VERIFY_CODE, param, this.requestSendVerifyCodeSuccess, false);
    },
    timer () {
        this.setState({ readSecond:true });
        this.setTimeout(
            () => {
                this.setState({ time:(this.state.time - 1) });
                if (this.state.time <= 0) {
                    return this.setState({ readSecond:false, time:59 });
                }
                this.timer();
            },
            1000
        );
    },
    requestSendVerifyCodeSuccess (data) {
        if (data.success) {
            this.timer();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { readSecond, time } = this.state;
        return (
            <View style={styles.container}>
                <View style={[styles.phoneContainer, { marginTop:10 }]}>
                    <Image
                        source={app.img.login_phone}
                        style={styles.text_phone_header} />
                    <TextInput
                        underlineColorAndroid='transparent'
                        placeholder='手机号'
                        maxLength={11}
                        onChangeText={(text) => this.setState({ phone: text })}
                        style={styles.text_input}
                        keyboardType='phone-pad'
                        />
                </View>
                <View style={styles.verification}>
                    <TextInput
                        underlineColorAndroid='transparent'
                        placeholder='请填写验证码'
                        keyboardType='phone-pad'
                        maxLength={CONSTANTS.VERIFICATION_MAX_LENGTH}
                        onChangeText={(text) => this.setState({ verifyCode: text })}
                        style={styles.text_input2}
                        />
                    <TouchableOpacity
                        disabled={readSecond}
                        style={styles.btnBind}
                        onPress={this.doPress}>
                        {
                            !readSecond ?
                                <Text style={readSecond ? styles.btnBindTextGray : styles.btnBindText}>获取验证码</Text>
                            :
                                <Text style={readSecond ? styles.btnBindTextGray : styles.btnBindText}>{time}s后重新发送</Text>
                        }
                    </TouchableOpacity>
                </View>
                <View style={styles.phoneContainer}>
                    <Image
                        source={app.img.login_password}
                        style={styles.text_phone_header} />
                    <TextInput
                        underlineColorAndroid='transparent'
                        placeholder='请输入密码'
                        maxLength={11}
                        onChangeText={(text) => this.setState({ password: text })}
                        style={styles.text_input}
                        />
                </View>
                <View style={styles.phoneContainer}>
                    <Image
                        source={app.img.login_password_check}
                        style={styles.text_phone_header} />
                    <TextInput
                        underlineColorAndroid='transparent'
                        placeholder='请再次输入密码'
                        maxLength={11}
                        onChangeText={(text) => this.setState({ rePassword: text })}
                        style={styles.text_input}
                        />
                </View>
                <View style={styles.agreement}>
                    <TouchableOpacity onPress={this.changeProtocalState}>
                        <Image
                            resizeMode='cover'
                            source={this.state.protocalRead ? app.img.order_choose_r : app.img.order_no_choose_r}
                            style={styles.protocal_icon}
                            />
                    </TouchableOpacity>
                    <Text style={styles.protocal_text}>  我已阅读并同意 </Text>
                    <Button onPress={this.doShowProtocal}
                        style={styles.protocal_button}
                        textStyle={styles.protocal_button_text}>
                        {'《四面通物流超市协议》'}
                    </Button>
                </View>
                <Button onPress={this.doRegister} style={styles.btnRegister} textStyle={styles.btnRegisterText}>注  册</Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    phoneContainer:{
        height:45,
        flexDirection: 'row',
        alignItems:'center',
        borderBottomWidth:1,
        borderColor:'#f9f9f9',
        backgroundColor:'#FFFFFF',
    },
    text_phone_header: {
        height:20,
        width:16,
        marginLeft:12,
    },
    verification:{
        height:45,
        flexDirection: 'row',
        justifyContent: 'space-between',
        backgroundColor:'#FFFFFF',
        borderBottomWidth:1,
        borderColor:'#f9f9f9',
        alignItems:'center',
    },
    text_input: {
        marginLeft:14,
        fontSize:15,
        height:45,
        lineHeight:45,
        width: 180,
        alignSelf: 'center',
        justifyContent:'center',
    },
    text_input2: {
        marginLeft: 12,
        fontSize:15,
        height: 45,
        lineHeight:45,
        width: 220,
        alignSelf: 'center',
    },
    btnRegister: {
        height: 46,
        width: sr.w - 100,
        marginLeft: 50,
        marginTop: 35,
        borderRadius: 4,
        backgroundColor: '#c81622',
    },
    btnRegisterText: {
        fontSize: 20,
        fontWeight: '600',
        color: '#FFFFFF',
    },
    protocal_icon: {
        height: 15,
        width: 15,
        marginRight: 10,
    },
    protocal_text: {
        fontSize: 13,
    },
    agreement:{
        flexDirection: 'row',
        height:20,
        alignItems:'center',
        marginLeft:13,
        marginTop:10,
    },
    protocal_button: {
        height: 18,
        backgroundColor:'#f4f4f4',
    },
    protocal_button_text: {
        fontSize: 13,
        color: '#c81622',
        fontWeight:'400',
    },
    btnBind:{
        justifyContent:'center',
        alignItems:'center',
        borderLeftWidth:1,
        borderColor:'#f4f4f4',
        height:45,
    },
    btnBindText:{
        color:'#FF981A',
        fontSize:16,
        fontWeight:'500',
        width:sr.w - 254,
        textAlign:'center',
    },
    btnBindTextGray:{
        color:'#999999',
        backgroundColor:'#fafafa',
        borderLeftWidth:1,
        borderColor:'#f4f4f4',
        fontSize:16,
        fontWeight:'500',
        textAlign:'center',
        width:sr.w - 254,
        height:45,
        lineHeight:45,
        textAlignVertical:'center',
    },
});

```

* PDClient/project/App/modules/common/CellPhoneBox.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    Text,
    View,
    Linking,
    TouchableOpacity,
} = ReactNative;

module.exports = React.createClass({
    openPhone () {
        app.closeModal();
        Linking.canOpenURL('tel:' + this.props.phoneNum).then(supported => {
            if (!supported) {
                console.log('Can\'t handle url: ' + this.props.phoneNum);
            } else {
                return Linking.openURL('tel:' + this.props.phoneNum);
            }
        }).catch(err => console.error('An error occurred', err));
    },
    render () {
        return (
            <View style={styles.overlayContainer}>
                <TouchableOpacity
                    activeOpacity={1}
                    style={styles.container}>
                    <View style={styles.topView}>
                        <Text style={styles.titleText}>
                            {'拨打电话'}
                        </Text>
                    </View>
                    <TouchableOpacity
                        onPress={this.openPhone}
                        style={styles.btnView}>
                        <Text style={styles.phoneText}>
                            {this.props.phoneNum}
                        </Text>
                    </TouchableOpacity>
                </TouchableOpacity>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        height: 114,
        width: sr.w - 50,
        borderRadius: 4,
        backgroundColor: '#FFFFFF',
        alignItems:'center',
        justifyContent:'center',
    },
    overlayContainer: {
        position:'absolute',
        top: 0,
        alignItems:'center',
        justifyContent: 'center',
        width:sr.w,
        height:sr.h,
    },
    titleText: {
        fontSize: 24,
        color: '#444444',
        backgroundColor: 'transparent',
    },
    topView: {
        height: 47,
        width: sr.w - 50,
        alignItems:'center',
        justifyContent:'center',
    },
    btnView: {
        height: 57,
        width: sr.w - 50,
        alignItems:'center',
        justifyContent:'center',
    },
    phoneText: {
        fontSize: 18,
        color: '#666666',
        backgroundColor: 'transparent',
    },
});

```

* PDClient/project/App/modules/common/ConfirmMerge.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TextInput,
    TouchableOpacity,
} = ReactNative;

const { Button, SelectStartPointAddress } = COMPONENTS;

module.exports = React.createClass({
    getInitialState () {
        return {
            groupName: '',
            startInfo: {},
            isShowAddress:false,
        };
    },
    doConfirm () {
        const { groupName, startInfo } = this.state;
        if (groupName == '') {
            Toast('请输入群组名');
            return;
        }
        if (!startInfo.startPoint) {
            Toast('请确定选择始发地');
            return;
        }
        app.closeModal();
        this.props.doConfirm({ groupName, ...startInfo });
    },
    render () {
        const { isShowAddress, startInfo } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.containerTop}>
                    <Text style={styles.textTop}>合并信息</Text>
                    <TouchableOpacity onPress={app.closeModal}>
                        <Image
                            resizeMode='stretch'
                            source={app.img.common_close}
                            style={styles.closeStyle} />
                    </TouchableOpacity>
                </View>
                <View style={styles.containerItem}>
                    <Text style={styles.textLeft}>群组名称</Text>
                    <TextInput
                        underlineColorAndroid='transparent'
                        placeholder='请输入群组名'
                        maxLength={10}
                        onChangeText={(text) => this.setState({ groupName: text })}
                        defaultValue={this.state.groupName}
                        style={styles.text_input}
                        />
                </View>
                <View style={styles.containerItem}>
                    <Text style={styles.textLeft}>始发地</Text>
                    <TouchableOpacity
                        activeOpacity={0.6}
                        onPress={() => { this.setState({ isShowAddress:true }); }}
                        style={styles.rightView}>
                        <Text style={styles.textRight}>{startInfo && (startInfo.startPoint || '') || '选择地址'}</Text>
                        <Image source={app.img.common_go} style={styles.goStyle} />
                    </TouchableOpacity>
                </View>
                {isShowAddress && <SelectStartPointAddress style={styles.SelectAddress} confirmAddress={(obj) => { this.setState({ startInfo:obj, isShowAddress:false }); }} />}
                <Button onPress={this.doConfirm} style={styles.btnCheck} textStyle={styles.btnCheckText} >
                    确认合并
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        backgroundColor:'#ffffff',
    },
    SelectAddress:{
        height:200,
    },
    containerTop:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textTop:{
        fontSize:17,
        marginLeft:(sr.w - 17 * 4) / 2,
        color:'#333333',
    },
    closeStyle:{
        height:30,
        width:30,
    },
    containerItem:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textLeft:{
        fontSize:14,
        color:'#333333',
        marginLeft:10,
    },
    textRight:{
        fontSize:14,
        color:'#888888',
        marginRight:10,
    },
    text_input: {
        height:40,
        width: 200,
        padding:0,
        fontSize:14,
        marginRight:10,
        alignSelf: 'center',
        textAlign:'right',
    },
    rightView:{
        flexDirection: 'row',
        alignItems: 'center',
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 10,
    },
    btnCheck:{
        marginTop:23,
        height:50,
        width:sr.w,
        borderRadius:0,
        backgroundColor:'#c81622',
    },
    btnCheckText:{
        color:'#ffffff',
        fontSize:16,
        fontWeight:'600',
    },
});

```

* PDClient/project/App/modules/common/ConfirmSendDoorAddress.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TextInput,
    Keyboard,
    TouchableOpacity,
} = ReactNative;

const { Button, SelectShortAddress } = COMPONENTS;

module.exports = React.createClass({
    getInitialState () {
        return {
            transportFee: '',
            sendDoorEndPoint: '',
            sendDoorLastCode: '',
            sendDoorAddress:'',
            isShowAddress:false,
        };
    },
    doCancel(){
        app.closeModal();
    },
    doConfirm () {
        const { sendDoorEndPoint, sendDoorAddress, transportFee } = this.state;
        if (sendDoorEndPoint == '') {
            Toast('请选择送货上门地址');
            return;
        }
        if (sendDoorAddress == '' && app.utils.checkStr(sendDoorAddress)) {
            Toast('请填写送货上门详细地址');
            return;
        }
        if (transportFee == '') {
            Toast('请填写送货上门金额');
            return;
        }
        if (!app.utils.checkInt2PointNum(transportFee)) {
            Toast('金额为数字且最多保留2位小数');
            return;
        }
        const param = {
            userId:app.personal.info.userId,
            orderId:this.props.orderId,
            isTransport:true,
            endPoint:sendDoorEndPoint + sendDoorAddress,
            transportFee,
            isScan:false,
        };
        POST(app.route.ROUTE_SET_CLIENT_PICK_ORDER_TRANSPORT_FEE,param,this.doConfirmSuccess,false);
    },
    doConfirmSuccess(data){
        if (data.success) {
            Toast('设置成功');
            app.closeModal();
            this.props.refresh();
            app.navigator.popToTop();
        }else{
            Toast(data.msg);
            app.closeModal();
        }
    },
    render () {
        const { isShowAddress, sendDoorEndPoint, sendDoorEndPointLastCode } = this.state;
        return (
            <View style={styles.container}>
                    <View style={styles.containerItem}>
                        <Text style={styles.textLeft}>送货上门金额</Text>
                        <TextInput
                            underlineColorAndroid='transparent'
                            placeholder='请输入商议金额'
                            maxLength={10}
                            onBlur={()=> Keyboard.dismiss()}
                            keyboardType='numeric'
                            onChangeText={(text) => this.setState({ transportFee: text })}
                            defaultValue={this.state.sendDoorPrice}
                            style={styles.text_input}
                            />
                        <Text style={styles.textRight}>元</Text>
                    </View>
                <View style={styles.containerItem}>
                    <Text style={styles.textLeft}>送货上门</Text>
                    <TouchableOpacity
                        activeOpacity={0.6}
                        onPress={() => { this.setState({ isShowAddress:true }); }}
                        style={styles.rightView}>
                        <Text style={styles.textRight}>{sendDoorEndPoint && (sendDoorEndPoint || '') || '选择地址'}</Text>
                        <Image source={app.img.common_go} style={styles.goStyle} />
                    </TouchableOpacity>
                </View>
                {isShowAddress && <SelectShortAddress style={styles.SelectAddress}
                parentCode = {this.props.endPointLastCode}
                endPointLastCode = {sendDoorEndPointLastCode}
                confirmAddress={(sendDoorEndPoint, sendDoorEndPointLastCode) => { this.setState({ sendDoorEndPoint, sendDoorEndPointLastCode, isShowAddress:false }); }} />}
                {
                    !sendDoorEndPoint == '' && <View style={styles.containerItem}>
                        <Text style={styles.textLeft}>详细地址</Text>
                        <TextInput
                            underlineColorAndroid='transparent'
                            placeholder='请输入详细地址'
                            onChangeText={(text) => this.setState({ sendDoorAddress: text })}
                            defaultValue={this.state.sendDoorAddress}
                            style={styles.text_input}
                            />
                    </View>
                }
                <View style={{flexDirection:'row'}}>
                    <Button onPress={this.doConfirm} style={styles.btnCheck} textStyle={styles.btnCheckText} >
                        确认
                    </Button>
                    <Button onPress={this.doCancel} style={styles.btnCancel} textStyle={styles.btnCheckText} >
                        取消
                    </Button>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        backgroundColor:'#ffffff',
    },
    SelectAddress:{
        height:150,
    },
    containerTop:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textTop:{
        fontSize:17,
        marginLeft:(sr.w - 17 * 4) / 2,
        color:'#333333',
    },
    closeStyle:{
        height:30,
        width:30,
    },
    containerItem:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textLeft:{
        fontSize:14,
        color:'#333333',
        marginLeft:10,
        flex:1,
    },
    textRight:{
        fontSize:14,
        color:'#888888',
        marginRight:10,
    },
    text_input: {
        flex:1,
        height:40,
        padding:0,
        fontSize:14,
        alignSelf: 'center',
        textAlign:'right',
        marginRight:10,
    },
    rightView:{
        flexDirection: 'row',
        alignItems: 'center',
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 10,
    },
    btnCheck:{
        marginTop:23,
        height:50,
        width:sr.w/2,
        borderRadius:0,
        backgroundColor:'#c81622',
    },
    btnCancel:{
        marginTop:23,
        height:50,
        width:sr.w/2,
        borderRadius:0,
        backgroundColor:'#F79835',
    },
    btnCheckText:{
        color:'#ffffff',
        fontSize:16,
        fontWeight:'600',
    },
});

```

* PDClient/project/App/modules/common/DigitalKeyboard.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

module.exports = React.createClass({
    statics: {
    },
    getInitialState () {
        return {
        };
    },
    render () {
        const { list } = this.props;
        return (
            <View style={styles.container}>
                {
                    list.map((item, i) => {
                        return (
                            <TouchableOpacity
                                onPress={this.props.passwordItem.bind(null, item)}
                                key={i} style={styles.itemLine}>
                                <Text style={styles.item}>{ item }</Text>
                            </TouchableOpacity>
                        );
                    })
                }
                <TouchableOpacity onPress={this.props.confirm} style={styles.itemImage}>
                    <Text style={styles.item}>确认</Text>
                </TouchableOpacity>
                <TouchableOpacity onPress={this.props.zero} style={styles.itemLine}>
                    <Text style={styles.item} >0</Text>
                </TouchableOpacity>
                <TouchableOpacity onPress={this.props.delete} style={styles.itemImage}>
                    <Image style={styles.delete}
                        source={app.img.personal_delete} />
                </TouchableOpacity>
            </View>
        );
    },
});
const styles = StyleSheet.create({
    container:{
        width:sr.w,
        backgroundColor:'#f4f4f4',
        flexDirection:'row',
        flexWrap:'wrap', // 换行
    },
    item:{
        fontSize:18,
        fontWeight:'600',
    },
    itemLine:{
        width:sr.w / 3,
        height:44,
        borderLeftWidth:1,
        borderBottomWidth:1,
        borderColor:'#f1f1f1',
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
        alignItems:'center',
    },
    itemGray:{
        height:45,
        width:sr.w / 3,
        backgroundColor:'#dddddd',
    },
    itemImage:{
        height:45,
        width:sr.w / 3,
        backgroundColor:'#dddddd',
        justifyContent:'center',
        alignItems:'center',
    },
    delete:{
        height:15,
        width:25,
    },
});

```

* PDClient/project/App/modules/common/ImageCrop.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    ImageStore,
    ImageEditor,
} = ReactNative;

const ImageCrop = require('@remobile/react-native-image-crop');

module.exports = React.createClass({
    mixins: [SceneMixin],
    statics: {
        title: '编辑图片',
        rightButton: { title: '完成', handler: () => {
            app.scene.cropImage&&app.scene.cropImage();
        } },
    },
    goBack () {
        app.pop();
    },
    cropImage () {
        const { image } = this.props;
        const cropData = this.imageCrop.getCropData();
        ImageEditor.cropImage(
            image.uri,
            cropData,
            (croppedImageURI) => {
                this.uploadImage(croppedImageURI);
            },
            (error) => {
                Toast('文件出错');
                this.goBack();
            }
        );
    },
    uploadImage (filePath) {
        if (app.isandroid) {
            const options = {};
            options.fileKey = 'file';
            options.fileName = filePath.substr(filePath.lastIndexOf('/') + 1);
            options.mimeType = 'image/png';
            options.params = {
                userId:app.personal.info.userId,
            };
            UPLOAD(filePath, app.route.ROUTE_UPDATE_FILE, options, (progress) => console.log(progress), this.uploadSuccessCallback, this.uploadErrorCallback, true);
        } else {
            ImageStore.getBase64ForTag(filePath, (data) => {
                data = 'data:image/png;base64,' + data;
                const options = {};
                options.fileKey = 'file';
                options.fileName = 'image.png';
                options.mimeType = 'image/png';
                options.params = {
                    userId:app.personal.info.userId,
                };
                ImageStore.removeImageForTag(filePath);
                UPLOAD(data, app.route.ROUTE_UPDATE_FILE, options, (progress) => console.log(progress), this.uploadSuccessCallback, this.uploadErrorCallback, true);
            }, () => {
                Toast('文件出错');
                this.goBack();
            });
        }
    },
    uploadSuccessCallback (data) {
        if (data.success) {
            this.props.onCropImage(data.context.url);
            this.goBack();
        } else {
            Toast('上传头像失败');
            this.goBack();
        }
    },
    uploadErrorCallback () {
        Toast('上传头像失败');
        this.goBack();
    },
    render () {
        const { image } = this.props;
        return (
            <View style={styles.container}>
                <ImageCrop
                    imageWidth={image.width}
                    imageHeight={image.height}
                    editRectRadius={0}
                    ref={(ref) => { this.imageCrop = ref; }}
                    source={{ uri: image.uri }} />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
});

```

* PDClient/project/App/modules/common/ImagePicker.js

```js
const React = require('react');
const ReactNative = require('react-native');
const {
    Image,
} = ReactNative;

const CameraRollPicker = require('@remobile/react-native-camera-roll-picker');
const Camera = require('@remobile/react-native-camera');
const ImageCrop = require('./ImageCrop');

module.exports = React.createClass({
    statics: {
        title: '选择头像',
    },
    cropImage (image) {
        app.navigator.replace({
            component: ImageCrop,
            passProps: {
                image,
                onCropImage: this.props.onCropImage,
            },
        });
    },
    onSelectedImages (images, image) {
        this.cropImage(image);
    },
    openCamera () {
        Camera.getPicture((filePath) => {
            const uri = 'file://' + filePath;
            Image.getSize(uri, (width, height) => {
                this.cropImage({
                    uri,
                    width,
                    height,
                });
            });
        }, null, {
            quality: 100,
            allowEdit: false,
            cameraDirection: Camera.Direction.FRONT,
            destinationType: Camera.DestinationType.FILE_URI,
        });
    },
    render () {
        return (
            <CameraRollPicker selected={[]} onSelectedImages={this.onSelectedImages} openCamera={this.openCamera} maximum={1} />
        );
    },
});

```

* PDClient/project/App/modules/common/LogisticsDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    ListView,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '物流详情',
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            list: [],
        };
    },
    componentWillMount () {
        const param = {
            userId: app.personal.info.userId,
            orderId: this.props.obj.id,
        };
        POST(app.route.ROUTE_GET_LOGISTICS_LIST, param, this.doReceiveSuccess, true);
    },
    doReceiveSuccess (data) {
        if (data.success) {
            this.setState({
                list : data.logisticsList,
            });
        } else {
            Toast(data.msg);
        }
    },
    toPayMode (payMode) {
        if (payMode == 0) {
            return '现付';
        } else if (payMode == 1) {
            return '到付';
        } else {
            return '混合付款';
        }
    },
    renderRow (obj, sectionID, rowID) {
        return (
            <View style={styles.containerItem} >
                <Image style={styles.imagePoint}
                    resizeMode='stretch'
                    source={rowID == 0 ? app.img.order_point_green : app.img.order_point}
                    />
                <View style={styles.containerRight}>
                    <View style={styles.address}>
                        <Text style={rowID == 0 ? styles.addressRightGreen : styles.addressRight}>{obj.address}</Text>
                    </View>
                    <Text style={rowID == 0 ? styles.textItemGreen : styles.textItem}>{obj.locateTime}</Text>
                </View>
            </View>
        );
    },
    render () {
        const { obj, type } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.logisticsState}>
                    <Image style={styles.imageGoods}
                        defaultSource={app.img.common_default}
                        source={{ uri:obj.photo }}
                        resizeMode='stretch'
                        />
                    <View style={styles.containerInfo}>
                        <View style={styles.infoTop}>
                            <Text style={styles.topLeft}>物流状态</Text>
                            <Text style={styles.topRight}>{type == 'agent' ? obj.stateList ? OS.MAP[obj.stateList[0].state] : '' : obj.stateList ? OS.MAP[obj.stateList[0].state] : ''}</Text>
                        </View>
                        <View style={styles.roadmapInfo}>
                            <Text style={styles.textGetState} numberOfLines={1}>路线：{obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : ''}</Text>
                            <Text style={styles.textGetState} numberOfLines={1}>->{obj.endPoint}</Text>
                        </View>
                        <Text style={styles.textGetState}>付款方式:{this.toPayMode(obj.payMode)}</Text>
                        <Text style={styles.textGetState}>是否送货上门:{obj.isSendDoor ? '是' : '否'}</Text>
                    </View>
                </View>
                <View style={styles.addressInfo}>
                    <ListView
                        dataSource={this.ds.cloneWithRows(this.state.list)}
                        enableEmptySections
                        renderRow={this.renderRow} />
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    logisticsState:{
        height:85,
        backgroundColor:'#ffffff',
        marginTop:10,
        flexDirection: 'row',
    },
    imageGoods:{
        height:65,
        width:65,
        marginTop:10,
        marginLeft:10,
    },
    containerInfo:{
        marginLeft:15,
        flex:1,
        marginBottom:5,
        marginTop:5,
    },
    infoTop:{
        flexDirection: 'row',
        flex:1,
    },
    topLeft:{
        fontSize:14,
        color:'#333333',
    },
    topRight:{
        marginLeft:15,
        fontSize:14,
        color:'#33a645',
    },
    textGetState:{
        fontSize:12,
        color:'#888888',
        flex:1,
    },
    addressInfo:{
        height:sr.h - 167,
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingTop:32,
    },
    containerItem:{
        height:75,
        flexDirection: 'row',
    },
    imagePoint:{
        height:75,
        width:8,
        marginLeft:30,
    },
    containerRight:{
        height:75,
        marginLeft:21,
    },
    address:{
        flexDirection: 'row',
        marginBottom:2,
    },
    addressLeft:{
        color:'#333333',
        fontSize:14,
    },
    addressLeftGreen:{
        fontSize:14,
        color:'#82c186',
    },
    addressRight:{
        color:'#888888',
        fontSize:12,
    },
    addressRightGreen:{
        color:'#82c186',
        fontSize:12,
    },
    textItem:{
        color:'#888888',
        fontSize:12,
        marginTop:2,
    },
    textItemGreen:{
        color:'#82c186',
        fontSize:12,
        marginTop:2,
    },
    roadmapInfo:{
        flexDirection:'row',
        flex:1,
    },
});

```

* PDClient/project/App/modules/common/PayInfo.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '支付信息',
        rightButton: { title: '关闭', handler: () => { app.scene.toClose&&app.scene.toClose(); } },
    },
    toClose () {
        app.pop();
    },
    render () {
        const { data, type, money, orderCount } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.containerTop}>
                    <Text style={styles.textTop}>支付信息</Text>
                    <TouchableOpacity onPress={this.props.doClose}>
                        <Image
                            resizeMode='stretch'
                            source={app.img.common_close}
                            style={styles.closeStyle} />
                    </TouchableOpacity>
                </View>
                <View style={styles.containerItem}>
                    <Text style={styles.textLeft}>订单信息</Text>
                    <Text style={styles.textRight}>{type == 'all' ? orderCount + '单' : data.id}</Text>
                </View>
                <TouchableOpacity>
                    <View style={styles.containerItem}>
                        <Text style={styles.textLeft}>支付选择</Text>
                        <View style={styles.rightView} >
                            <Text style={styles.textRightCash}>余额</Text>
                            <Image
                                source={app.img.common_go}
                                style={styles.goStyle} />
                        </View>
                    </View>
                </TouchableOpacity>
                <View style={styles.containerItem}>
                    <Text style={styles.textLeft}>运费</Text>
                    <Text style={styles.textRightFee}>
                        ¥
                        {
                            type == 'receive' ?
                            app.utils.N(data.totalDesignatedFee + data.proxyCharge + data.additionalFee)
                            :
                            type == 'send' ?
                            app.utils.N(data.needPayTransportFee + data.needPayInsuanceFee)
                            :
                            type == 'all' ?
                            money
                            :
                            app.utils.N(data.carryPrice * data.totalWeight)
                        }
                    </Text>
                </View>
                <Button onPress={this.props.doConfirmPay} style={styles.btnCheck} textStyle={styles.btnCheckText} >
                    确认支付
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#ffffff',
    },
    containerTop:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textTop:{
        fontSize:17,
        marginLeft:(sr.w - 17 * 4) / 2,
        color:'#333333',
    },
    closeStyle:{
        height:30,
        width:30,
    },
    containerItem:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textLeft:{
        fontSize:14,
        color:'#333333',
        marginLeft:10,
    },
    textRight:{
        fontSize:14,
        color:'#888888',
        marginRight:10,
    },
    textRightFee:{
        color:'red',
        fontWeight:'500',
        fontSize:14,
        marginRight:10,
    },
    textRightCash:{
        fontSize:14,
        marginRight:6,
        color:'#3d3e40',
    },
    rightView:{
        flexDirection: 'row',
        alignItems: 'center',
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 10,
    },
    btnCheck:{
        marginTop:23,
        height:50,
        width:sr.w,
        borderRadius:0,
        backgroundColor:'#c81622',
    },
    btnCheckText:{
        color:'#ffffff',
        fontSize:16,
        fontWeight:'600',
    },
});

```

* PDClient/project/App/modules/common/PayInfoModel.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button } = COMPONENTS;

module.exports = React.createClass({
    doConfirmPay () {
        app.closeModal();
        this.props.doConfirmPay();
    },
    render () {
        const { data } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.containerTop}>
                    <Text style={styles.textTop}>支付信息</Text>
                    <TouchableOpacity onPress={app.closeModal}>
                        <Image
                            resizeMode='stretch'
                            source={app.img.common_close}
                            style={styles.closeStyle} />
                    </TouchableOpacity>
                </View>
                <View style={styles.containerItem}>
                    <Text style={styles.textLeft}>订单信息</Text>
                    <Text style={styles.textRight}>{data.id}</Text>
                </View>
                <TouchableOpacity>
                    <View style={styles.containerItem}>
                        <Text style={styles.textLeft}>支付选择</Text>
                        <View style={styles.rightView} >
                            <Text style={styles.textRightCash}>余额</Text>
                            <Image
                                source={app.img.common_go}
                                style={styles.goStyle} />
                        </View>
                    </View>
                </TouchableOpacity>
                <View style={styles.containerItem}>
                    <Text style={styles.textLeft}>运费</Text>
                    <Text style={styles.textRightFee}>¥{app.utils.N(data.carryPrice * data.totalWeight) }</Text>
                </View>
                <Button onPress={this.doConfirmPay} style={styles.btnCheck} textStyle={styles.btnCheckText} >
                        确认支付
                    </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        height:230,
        top:sr.h - 448,
        backgroundColor:'#ffffff',
    },
    containerTop:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textTop:{
        fontSize:17,
        marginLeft:(sr.w - 17 * 4) / 2,
        color:'#333333',
    },
    closeStyle:{
        height:30,
        width:30,
    },
    containerItem:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textLeft:{
        fontSize:14,
        color:'#333333',
        marginLeft:10,
        flex:1,
    },
    textRight:{
        fontSize:14,
        color:'#888888',
        marginRight:10,
    },
    textRightFee:{
        color:'red',
        fontWeight:'500',
        fontSize:14,
        marginRight:10,
        textAlign:'right',
        flex:1,
    },
    textRightCash:{
        fontSize:14,
        marginRight:6,
        color:'#3d3e40',
    },
    rightView:{
        flexDirection: 'row',
        alignItems: 'center',
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 10,
    },
    btnCheck:{
        marginTop:23,
        height:50,
        width:sr.w,
        borderRadius:0,
        backgroundColor:'#c81622',
    },
    btnCheckText:{
        color:'#ffffff',
        fontSize:16,
        fontWeight:'600',
    },
});

```

* PDClient/project/App/modules/common/PayPassword.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const { MessageBox } = COMPONENTS;
const DigitalKeyboard = require('./DigitalKeyboard');
const VerificationCode = require('../person/VerificationCode');

const list = [ 1, 2, 3, 4, 5, 6, 7, 8, 9 ];

module.exports = React.createClass({
    statics: {
        title: '支付密码',

    },
    getInitialState () {
        return {
            password: '',
            type:false,
        };
    },
    passwordItem (item) {
        const { password } = this.state;
        this.setState({ password: password + item });
    },
    zero () {
        const { password } = this.state;
        this.setState({ password: password + 0 });
    },
    delete () {
        const { password } = this.state;
        this.setState({
            password: _.dropRight(password).join(''),
        });
    },
    confirm () {
        const { password, type } = this.state;
        const tmp = password.substring(0, 6);
        if (tmp.length == 6) {
            this.setState({
                type : !type,
            });
            this.toPay(tmp);
        } else {
            Toast('支付密码有误');
            this.toClear(tmp);
        }
    },
    toPay (tmp) {
        let { param, routeName } = this.props;
        param['payPassword'] = tmp;
        POST(routeName, param, this.toPaySuccess, true);
    },
    toPaySuccess (data) {
        const { prompt, amount, doCloseActionSheet, money, alreadyPaid, refresh, setRemainAmount } = this.props;
        if (data.success) {
            app.showModal(
                <MessageBox
                    onConfirm={this.doConfirmDelete}
                    content={prompt}
                    width={sr.s(250)} />
            );
            !!doCloseActionSheet && doCloseActionSheet();
            !!amount && app.personal.setRemainAmount((app.personal.remainAmount * 1) + (amount * 1));
            !!money && app.personal.setRemainAmount((app.personal.remainAmount * 1) - (money * 1));
            !!setRemainAmount && setRemainAmount((app.personal.remainAmount * 1) - (money * 1));
            !!alreadyPaid && alreadyPaid();
            !!refresh && refresh();
            app.navigator.popToTop();
            return;
        } else {
            Toast(data.msg);
            !!doCloseActionSheet && doCloseActionSheet();
            app.pop();
        }
    },
    toClear (tmp) {
        this.setState({
            password: '',
            tmp: '',
        });
    },
    componentWillMount () {
        this.toSetPaypassword();
    },
    toSetPaypassword () {
        if (app.personal.info.isSetPaymentPassword == 0) {
            Toast('请先设置支付密码');
            app.push({
                component:VerificationCode,
            });
        }
    },
    render () {
        const max = [1, 2, 3, 4, 5, 6];
        const { password, type } = this.state;
        return (
            <View style={styles.container}>
                <Text style={styles.text}>请输入支付密码</Text>
                <View style={styles.password}>
                    {
                        max.map((item, i) => (
                            <View style={i == 0 ? styles.itemStart : styles.item} key={i}>
                                { password.length >= (i + 1) ?
                                    <View style={styles.point} />
                                    :
                                    <View style={styles.noPoint} />
                                }
                            </View>
                        ))
                    }
                </View>
                <View style={styles.digitalKeyboard} />
                {
                    !type &&
                    <View>
                        <DigitalKeyboard
                            list={list}
                            passwordItem={this.passwordItem}
                            zero={this.zero} delete={this.delete}
                            confirm={this.confirm}
                            />
                    </View>
                }
            </View>
        );
    },
});
const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#f4f4f4',
    },

    text:{
        fontSize:14,
        textAlign:'center',
        color:'#333333',
        marginTop:30,
    },
    password:{
        height:40,
        width:330,
        backgroundColor:'#ffffff',
        flexDirection:'row',
        marginTop:15,
        marginLeft:(sr.w - 330) / 2,
        borderWidth:1,
        borderColor:'#EDEDED',
        borderRadius:4,
    },
    itemStart:{
        height:40,
        width:55,
        borderColor:'#EDEDED',
        justifyContent:'center',
        alignItems:'center',
    },
    item:{
        height:40,
        width:55,
        borderLeftWidth:1,
        borderColor:'#EDEDED',
        justifyContent:'center',
        alignItems:'center',
    },
    point:{
        height:10,
        width:10,
        borderRadius:5,
        backgroundColor:'#000000',
    },
    noPoint:{
        height:10,
        width:10,
        borderRadius:5,
        backgroundColor:'#FFFFFF',
    },
    digitalKeyboard:{
        flex:1,
    },
});

```

* PDClient/project/App/modules/common/Roadmap.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TextInput,
    TouchableOpacity,
} = ReactNative;

const { SelectRegionAddress, SelectShortAddress } = COMPONENTS;

const { Button } = COMPONENTS;

const ChangeType = {
    PRICE:0,
    PERCENT:1,
};

module.exports = React.createClass({
    statics: {
        title: '路线设置',
    },
    getInitialState () {
        const { obj } = this.props;
        var profit = '';
        if (obj) {
            if ((obj.profitCount == 0) && (obj.profitRate != 0)) {
                profit = obj.profitRate;
            } else if ((obj.profitCount != 0) && (obj.profitRate == 0)) {
                profit = obj.profitCount;
            } else {
                profit = '';
            }
        } else {
            profit = '';
        }
        return {
            price: profit,
            type: obj.type ? obj.type : '',
            change: (obj.profitCount == 0) && (obj.profitRate != 0) ? ChangeType.PERCENT : ChangeType.PRICE,
            endPointLastCode: obj.regionLastCode ? obj.regionLastCode : 0,
            endPoint: obj.region ? obj.region : 0,
        };
    },
    toChangePrice () {
        this.setState({
            change: ChangeType.PRICE,
            price: '',
        });
    },
    toChangePercent () {
        this.setState({
            change: ChangeType.PERCENT,
            price: '',
        });
    },
    selectAspect () {
        const { endPointLastCode } = this.state;
        const { type } = this.props;
        if (type == 2) {
            app.push({
                component: SelectShortAddress,
                title:'到货地',
                passProps:{
                    parentCode : app.personal.info.agent.addressRegionLastCode,
                    confirmAddress:(endPoint, endPointLastCode) => {
                        this.setState({ endPoint, endPointLastCode });
                    },
                },
            });
        }
        if (type == 1) {
            app.push({
                component: SelectRegionAddress,
                title:'到货地',
                passProps:{
                    type : 4,
                    confirmAddress:(endPoint, endPointLastCode) => {
                        this.setState({ endPoint, endPointLastCode });
                    },
                },
            });
        }
    },
    doConfirmChange () {
        const { change, price, endPoint, endPointLastCode } = this.state;
        const { type, isModify, obj } = this.props;
        var profit = 0;
        if (isModify) {
            const param = {
                userId: app.personal.info.userId,
                regionId: obj.id,
                region: endPoint,
                regionLastCode: endPointLastCode,
                profitRate: (change == 1 ? price : 0),
                profitCount: (change == 0 ? price : 0),
                type,
            };
            POST(app.route.ROUTE_MODIFY_REGION_PROFIT, param, this.addRegionProfitSuccess, false);
        } else {
            const param = {
                userId: app.personal.info.userId,
                region: endPoint,
                regionLastCode: endPointLastCode,
                profitRate: (change == 1 ? price : 0),
                profitCount: (change == 0 ? price : 0),
                type,
            };
            POST(app.route.ROUTE_ADD_REGION_PROFIT, param, this.addRegionProfitSuccess, false);
        }
    },
    addRegionProfitSuccess (data) {
        if (data.success) {
            Toast('设置成功');
            this.props.refreshList();
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { change, price, endPoint, endPointLastCode } = this.state;
        const { index, tabIndex, type } = this.props;
        return (
            <View style={index === tabIndex ? styles.container : { left:-sr.tw, top:0, position:'absolute' }}>
                <TouchableOpacity onPress={this.selectAspect}
                    style={styles.aspect} >
                    <Text style={styles.aspectText}>方向</Text>
                    <View style={styles.rightView} >
                        <Text style={styles.itemValueText}>{ type != 0 ? (endPoint == 0 ? '' : endPoint) : '全部' }</Text>
                        {
                            type != 0 &&
                            <Image
                                source={app.img.common_go}
                                style={styles.goStyle} />
                        }
                    </View>
                </TouchableOpacity>
                <View style={styles.priceChange}>
                    <TouchableOpacity onPress={this.toChangePrice}
                        style={[styles.itemChange, change == 0 ? { backgroundColor:'#c91521' } : {}]}>
                        <Text style={[styles.changeText, change == 0 ? { color:'#FFFFFF' } : {}]}>按价格调整</Text>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={this.toChangePercent}
                        style={[styles.itemChange, change == 1 ? { backgroundColor:'#c91521' } : {}]}>
                        <Text style={[styles.changeText, change == 1 ? { color:'#FFFFFF' } : {}]}>按百分比调整</Text>
                    </TouchableOpacity>
                </View>
                <View style={styles.priceNumber}>
                    {
                        (change == 1 || change == 0) ?
                            <View onPress={this.toAdd} style={styles.priceAdd} >
                                <Image style={styles.chooseImage}
                                    source={app.img.order_choose} />
                                <Text style={styles.priceAddText} >上调</Text>
                            </View>
                        :
                            <View />
                    }
                    {
                        change == 0 ?
                            <View style={styles.priceInput}>
                                <TextInput
                                    onChangeText={(text) => this.setState({ price: text })}
                                    multiline={false}
                                    style={styles.bottomRight}
                                    placeholder={'请输入需要提高的价格'}
                                    autoCapitalize={'none'}
                                    underlineColorAndroid={'transparent'}
                                    defaultValue={price + ''}
                                />
                                <Text style={styles.bottomRightText}>元</Text>
                            </View>
                        :
                        change == 1 ?
                            <View style={styles.priceInput}>
                                <TextInput
                                    onChangeText={(text) => this.setState({ price: text })}
                                    multiline={false}
                                    style={styles.bottomRight}
                                    placeholder={'请输入需要提高的百分比'}
                                    autoCapitalize={'none'}
                                    underlineColorAndroid={'transparent'}
                                    defaultValue={price + ''}
                                />
                                <Text style={styles.bottomRightText}>%</Text>
                            </View>
                        :
                            <View />
                    }
                </View>
                <Button onPress={this.doConfirmChange}
                    style={styles.btnConfirmChange}
                    textStyle={styles.btnConfirmChangeText}>
                    确认
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor:'#F4F4F4',
    },
    aspect:{
        marginTop:5,
        backgroundColor: 'white',
        flexDirection: 'row',
        height: 42,
        width:sr.w,
        justifyContent: 'space-between',
        alignItems:'center',
    },
    aspectText:{
        color: '#888888',
        fontSize: 14,
        marginLeft: 10,
    },
    rightView: {
        flexDirection: 'row',
        height: 45,
        alignItems:'center',
    },
    itemValueText: {
        color: '#888888',
        fontSize: 14,
        marginRight: 10,
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 10,
    },
    priceChange:{
        height:45,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        alignItems:'center',
        justifyContent:'space-around',
        marginTop:2,
    },
    itemChange:{
        height:20,
        width:100,
        backgroundColor:'#f3f3f3',
    },
    changeText:{
        fontSize:12,
        color:'#333333',
        textAlign:'center',
        height:20,
        lineHeight:18,
    },
    priceNumber:{
        height:40,
        backgroundColor:'#FFFFFF',
        marginTop:2,
        flexDirection: 'row',
        justifyContent:'space-between',
    },
    bottomLeft:{
        flexDirection: 'row',
        alignItems:'center',
        height:40,
    },
    priceAdd:{
        flexDirection: 'row',
        alignItems:'center',
        marginLeft:10,
    },
    chooseImage:{
        height:9,
        width:9,
    },
    priceAddText:{
        fontSize:14,
        color:'#333333',
        marginLeft:5,
    },
    priceSub:{
        flexDirection: 'row',
        alignItems:'center',
        marginLeft:20,
    },
    priceInput:{
        flexDirection: 'row',
        alignItems:'center',
    },
    bottomRight:{
        height:40,
        width:sr.w / 2,
        fontSize:12,
        color: '#444444',
        textAlign:'right',
    },
    bottomRightText:{
        fontSize:13,
        color:'#333333',
        marginRight:15,
        marginLeft:3,
    },
    btnConfirmChange:{
        height:40,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        marginTop:250,
        borderRadius:4,
    },
    btnConfirmChangeText:{
        fontSize:16,
        fontWeight:'500',
    },
});

```

* PDClient/project/App/modules/common/ScanBarCode.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

import BarcodeScanner from 'react-native-camera';
const { Button } = COMPONENTS;

module.exports = React.createClass({
    onBarCodeRead (code) {
        if (!this.hasReadCode) {
            app.closeModal();
            this.hasReadCode = true;
            this.props.onBarCodeRead(app.utils.getQRResult(code.data));
        }
    },
    render () {
        return (
            <View style={styles.container}>
                <BarcodeScanner
                    onBarCodeRead={this.onBarCodeRead}
                    style={styles.camera}>
                    <View style={styles.rectangleContainer}>
                        <View style={styles.rectangleTop} />
                        <View style={styles.rectangleMiddle}>
                            <View style={styles.rectangleLeft} />
                            <View style={styles.rectangleMiddleMiddle}>
                                <View style={[styles.makeup, styles.makeupTL]} />
                                <View style={[styles.makeup, styles.makeupTR]} />
                                <View style={[styles.makeup, styles.makeupDL]} />
                                <View style={[styles.makeup, styles.makeupDR]} />
                            </View>
                            <View style={styles.rectangleRight} />
                        </View>
                        <View style={styles.rectangleBottom}>
                            <Text style={styles.info}>
                                将二维码放入框内，即可自动扫描
                            </Text>
                        </View>
                    </View>
                </BarcodeScanner>
                <Button
                    onPress={app.closeModal}
                    style={styles.contentButton}
                    textStyle={styles.contentButtonText}>
                        关闭</Button>
            </View>
        );
    },
});
const OVERLAY = 'rgba(0, 0, 0, 0.2)';
const BORDER = 2;
const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    camera: {
        flex: 1,
        width: sr.w,
    },
    rectangleContainer: {
        flex: 1,
    },
    rectangleTop: {
        flex: 1,
        backgroundColor: OVERLAY,
    },
    rectangleMiddle: {
        height: 250,
        flexDirection: 'row',
    },
    rectangleBottom: {
        flex: 1,
        backgroundColor: OVERLAY,
        alignItems: 'center',
    },
    rectangleLeft: {
        flex:1,
        backgroundColor: OVERLAY,
    },
    rectangleMiddleMiddle: {
        width:250,
    },
    rectangleRight: {
        flex:1,
        backgroundColor: OVERLAY,
    },
    makeup: {
        width: 16,
        height: 16,
        position: 'absolute',
        borderColor: '#00FF00',
    },
    makeupTL: {
        top: 0,
        left: 0,
        borderTopWidth: BORDER,
        borderLeftWidth: BORDER,
    },
    makeupTR: {
        top: 0,
        right: 0,
        borderTopWidth: BORDER,
        borderRightWidth: BORDER,
    },
    makeupDL: {
        bottom: 0,
        left: 0,
        borderBottomWidth: BORDER,
        borderLeftWidth: BORDER,
    },
    makeupDR: {
        bottom: 0,
        right: 0,
        borderBottomWidth: BORDER,
        borderRightWidth: BORDER,
    },
    info: {
        marginTop: 30,
        fontSize: 15,
        color: '#646566',
    },
    contentButton: {
        width:sr.w,
        height:50,
        borderRadius:0,
    },
    contentButtonText: {
        color: 'white',
        fontWeight: '800',
    },
});

```

* PDClient/project/App/modules/common/SelectClient.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    Image,
    TouchableOpacity,
} = ReactNative;

const { PageList } = COMPONENTS;

let searchText = '';
const Title = React.createClass({
    render () {
        return (
            <View style={styles.searchContainer}>
                <Image
                    resizeMode='cover'
                    source={app.img.common_search_button}
                    style={styles.iconSearch} />
                <TextInput
                    placeholder='输入关键字搜索用户'
                    defaultValue={searchText}
                    onChangeText={(text) => { searchText = text; }}
                    onSubmitEditing={this.props.doStartSearch}
                    style={styles.searchTextInput}
                    />
            </View>
        );
    },
});

module.exports = React.createClass({
    statics: {
        title: (<Title doStartSearch={() => { app.scene.doStartSearch&&app.scene.doStartSearch(); }} />),
        rightButton: { title: '搜索', handler: () => { app.scene.refresh&&app.scene.refresh(); } },
    },
    getInitialState () {
        return {
            keyword: '',
        };
    },
    refresh () {
        this.setState({ keyword: searchText }, () => {
            this.listView.refresh();
        });
    },
    onRowSelect (obj) {
        this.props.onSelect(obj);
        app.pop();
    },
    renderRow (obj, sectionID, rowID) {
        const { userId } = this.props;
        const { name, head, email, address, phone, reservePhone } = obj;
        return (
            <TouchableOpacity style={styles.row} onPress={this.onRowSelect.bind(null, obj)}>
                <View style={styles.leftContainer}>
                    <View style={styles.nameContainer}>
                        <Image
                            resizeMode='stretch'
                            source={head ? { uri: head } : app.img.personal_default_head}
                            style={styles.head} />
                        <Text style={styles.name}>{name ? `${name} ( ${phone} )` : phone }</Text>
                    </View>
                    <View style={styles.nameContainer}>
                        <Text style={styles.label}>邮箱：</Text>
                        <Text style={styles.name}>{ email || '无' }</Text>
                    </View>
                    <View style={styles.nameContainer}>
                        <Text style={styles.label}>住址：</Text>
                        <Text style={styles.name}>{ address || '无' }</Text>
                    </View>
                    <View style={styles.nameContainer}>
                        <Text style={styles.label}>预留电话：</Text>
                        <Text style={styles.name}>{ reservePhone ? reservePhone.join('; ') : '无' }</Text>
                    </View>
                </View>
                {
                    obj.userId === userId &&
                    <Image resizeMode='stretch' source={app.img.common_radio} style={styles.icon} />
                }
            </TouchableOpacity>
        );
    },
    render () {
        const { keyword } = this.state;
        return (
            <View style={styles.container}>
                <PageList
                    ref={(ref) => { this.listView = ref; }}
                    renderRow={this.renderRow}
                    listParam={{ userId: app.personal.info.userId, keyword }}
                    listName={'clientList'}
                    listUrl={app.route.ROUTE_GET_CLIENT_LIST}
                    refreshEnable
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    searchContainer: {
        height: 30,
        width: 250,
        paddingVertical: 2,
        borderRadius: 105,
        backgroundColor: '#E5E5E5',
        flexDirection: 'row',
        overflow: 'hidden',
        alignItems:'center',
    },
    searchTextInput: {
        height: 25,
        width: 220,
        fontSize: 14,
        marginLeft: 5,
        overflow: 'hidden',
        alignItems:'center',
    },
    iconSearch: {
        height: 20,
        width: 20,
    },
    row: {
        paddingHorizontal: 8,
        paddingVertical: 10,
        flexDirection: 'row',
        alignItems: 'center',
    },
    leftContainer: {
        flex: 1,
    },
    nameContainer: {
        marginTop: 10,
        flexDirection: 'row',
        paddingHorizontal: 10,
        alignItems: 'center',
    },
    head: {
        width: 30,
        height: 30,
        borderRadius: 15,
        marginRight: 5,
    },
    name: {
        fontSize: 14,
        color: 'gray',
    },
    label: {
        fontSize: 14,
        color: '#8B8B7A',
        marginLeft: 34,
    },
    icon: {
        width: 20,
        height: 16,
    },
});

```

* PDClient/project/App/modules/common/SelectRoadmap.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    Image,
    TouchableOpacity,
} = ReactNative;

const { PageList } = COMPONENTS;

let searchText = '';
const Title = React.createClass({
    render () {
        return (
            <View style={styles.searchContainer}>
                <Image
                    resizeMode='cover'
                    source={app.img.common_search_button}
                    style={styles.iconSearch} />
                <TextInput
                    placeholder='输入关键字搜索路线'
                    defaultValue={searchText}
                    onChangeText={(text) => { searchText = text; }}
                    onSubmitEditing={this.props.doStartSearch}
                    style={styles.searchTextInput}
                    />
            </View>
        );
    },
});

module.exports = React.createClass({
    statics: {
        title: (<Title doStartSearch={() => { app.scene.doStartSearch&&app.scene.doStartSearch(); }} />),
        rightButton: { title: '搜索', handler: () => { app.scene.refresh&&app.scene.refresh(); } },
    },
    getInitialState () {
        return {
            keyword: '',
        };
    },
    refresh () {
        this.setState({ keyword: searchText }, () => {
            this.listView.refresh();
        });
    },
    onRowSelect (obj) {
        this.props.onSelect(obj);
        app.pop();
    },
    renderRow (obj, sectionID, rowID) {
        const { roadmapId } = this.props;
        const { name, type, shipperName, shipperLogo, startPoint, endPoint, transitPoint, price, duration } = obj;
        return (
            <TouchableOpacity style={styles.row} onPress={this.onRowSelect.bind(null, obj)}>
                <View style={styles.leftContainer}>
                    <View style={styles.shipperNameContainer}>
                        <Image
                            resizeMode='stretch'
                            source={{ uri: shipperLogo }}
                            style={styles.shipperLogo} />
                        <Text numberOfLines={1} style={styles.shipperName}>{shipperName}</Text>
                    </View>
                    <View style={styles.nameContainer}>
                        <Text numberOfLines={1} style={styles.name}>{name}</Text>
                        <Text numberOfLines={1} style={[styles.type, { color: type === 0 ? '#228B22' : '#FF4500' }]}>{type === 0 ? '整车' : '零担'}</Text>
                    </View>
                    <View style={styles.pointContainer}>
                        <Text style={styles.name}>{startPoint.name}</Text>
                        {!!transitPoint[0] && <Text style={styles.name}>→</Text>}
                        {!!transitPoint[0] && <Text style={styles.name}>{transitPoint[0].name}</Text>}
                        <Text style={styles.name}>→</Text>
                        <Text style={styles.name}>{endPoint.name}</Text>
                    </View>
                    <View style={styles.priceContainer}>
                        <Text style={styles.name}><Text style={{ color: '#DAA520' }}>{price}</Text>元/{type === 0 ? '车' : '吨'}</Text>
                        <Text style={styles.name}>历时<Text style={{ color: '#FF4500' }}>{duration}</Text>小时</Text>
                    </View>
                </View>
                {
                    obj.id === roadmapId &&
                    <Image resizeMode='stretch' source={app.img.common_radio} style={styles.icon} />
                }
            </TouchableOpacity>
        );
    },
    render () {
        const { keyword } = this.state;
        return (
            <View style={styles.container}>
                <PageList
                    ref={(ref) => { this.listView = ref; }}
                    renderRow={this.renderRow}
                    listParam={{ userId: app.personal.info.userId, keyword }}
                    listName={'roadmapList'}
                    listUrl={app.route.ROUTE_GET_ROADMAP_LIST}
                    refreshEnable
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    searchContainer: {
        height: 30,
        width: 250,
        paddingVertical: 2,
        borderRadius: 105,
        backgroundColor: '#E5E5E5',
        flexDirection: 'row',
        overflow: 'hidden',
        alignItems:'center',
    },
    searchTextInput: {
        height: 25,
        width: 220,
        fontSize: 14,
        marginLeft: 5,
        overflow: 'hidden',
        alignItems:'center',
    },
    iconSearch: {
        height: 20,
        width: 20,
    },
    row: {
        paddingHorizontal: 8,
        paddingVertical: 10,
        flexDirection: 'row',
        alignItems: 'center',
    },
    leftContainer: {
        flex: 1,
    },
    shipperNameContainer: {
        flexDirection: 'row',
        alignItems: 'center',
    },
    shipperContainer: {
        flexDirection: 'row',
        alignItems: 'center',
        marginTop: 20,
        marginLeft: 10,
    },
    shipperLogo: {
        width: 20,
        height: 20,
        borderRadius: 10,
        marginRight: 5,
    },
    shipperName: {
        fontSize: 14,
        color: 'gray',
    },
    nameContainer: {
        marginTop: 10,
        flexDirection: 'row',
        justifyContent: 'space-between',
        paddingHorizontal: 20,
        alignItems: 'center',
    },
    name: {
        fontSize: 14,
        color: 'gray',
    },
    type: {
        fontSize: 14,
    },
    pointContainer: {
        marginTop: 10,
        flexDirection: 'row',
        justifyContent: 'space-between',
        paddingHorizontal: 20,
    },
    priceContainer: {
        marginTop: 10,
        flexDirection: 'row',
        justifyContent: 'space-between',
        paddingHorizontal: 20,
    },
    icon: {
        width: 20,
        height: 16,
    },
});

```

* PDClient/project/App/modules/common/SendOrderDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button, CustomSheet } = COMPONENTS;
const LogisticsDetail = require('./LogisticsDetail');
const PayInfo = require('./PayInfo');
const PayPassword = require('./PayPassword');
const moment = require('moment');

module.exports = React.createClass({
    statics: {
        title: '货单详情',
    },
    getInitialState () {
        return {
            choose:false,
            canPay:false || this.props.canPay,
            actionSheetVisible: false,
        };
    },
    componentWillMount () {
        const param = {
            list: [],
            userId: app.personal.info.userId,
            orderId: this.props.data.id,
        };
        POST(app.route.ROUTE_GET_LOGISTICS_LIST, param, this.doReceiveSuccess, true);
    },
    doReceiveSuccess (data) {
        if (data.success) {
            this.setState({
                list : data.logisticsList,
            });
        } else {
            Toast(data.msg);
        }
    },
    doChoose () {
        this.setState({
            choose:!this.state.choose,
        });
    },
    doCloseActionSheet () {
        this.setState({ actionSheetVisible:false });
    },
    doShowActionSheet () {
        this.setState({ actionSheetVisible:true });
    },
    doConfirmPay () {
        const { data } = this.props;
        const param = {
            userId: app.personal.info.userId,
            orderIdList: [this.props.data.id],
        };
        app.push({
            component:PayPassword,
            passProps:{
                param,
                routeName : data.agent ? app.route.ROUTE_PAY_AGENT_FOR_ORDER : app.route.ROUTE_PAY_FOR_ORDER_WHEN_SEND,
                prompt : '支付成功',
                doCloseActionSheet : this.doCloseActionSheet,
                alreadyPaid: this.props.alreadyPaid,
                money:app.utils.N(data.needPayInsuanceFee + data.needPayTransportFee),
            },
        });
    },
    doLookTransport () {
        app.push({
            component: LogisticsDetail,
            passProps:{
                obj: this.props.data,
            },
        });
    },
    toPayMode (payMode) {
        if (payMode == 0) {
            return '现付';
        } else if (payMode == 1) {
            return '到付';
        } else {
            return '混合付款';
        }
    },
    render () {
        const { data } = this.props;
        const { list } = this.state;
        const { stateList } = data;
        console.log('---',list);
        return (
            <View style={styles.container}>
                <View style={styles.containerReceive}>
                    <Text style={styles.textName}> 收货人：{!data.receiverName ? '暂无' : data.receiverName}</Text>
                    <Text style={styles.textName}> 收货人电话:{!data.receiverPhone ? '暂无' : data.receiverPhone}</Text>
                    <Text style={styles.textAddress}> 到货地：{data && data.endPoint} </Text>
                </View>
                <View style={styles.containerMarket}>
                    <Image style={styles.marketLogo}
                        source={app.img.order_logo}
                        resizeMode='stretch'
                        />
                    <Text style={styles.textMarket}>{ data.shop ? data.shop.name : data.agent ? data.agent.name : ''}</Text>
                    <Text style={styles.orderState}>{ stateList ? OS.MAP[stateList[0].state] : ''}</Text>
                </View>
                <View style={styles.containerDetails}>
                    <Image style={styles.imageGoods}
                        resizeMode='stretch'
                        defaultSource={app.img.common_default}
                        source={{ uri:data.photo }}
                        />
                    <View style={styles.goodsDetails}>
                        <View style={styles.goodsDetailsTop}>
                            <View style={styles.detailsTopLeft}>
                                <Text style={styles.detailsAddress}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{data.shop ? data.shop.address : data.agent ? data.agent.address : ''}</Text>
                                <Text >-</Text>
                                <Text style={styles.detailsAddress}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{data.endPoint}</Text>
                            </View>
                            <View style={styles.detailsTopRight}>
                                <Text style={styles.moneyLeft}>运费：</Text>
                                <Text style={styles.moneyRight}>¥{data.realFee}</Text>
                            </View>
                        </View>
                        <Text style={styles.textDetailsItem}>重量：{data.size}m³</Text>
                        <Text style={styles.textDetailsItem}>方量：{data.weight}吨</Text>
                        <Text style={styles.textDetailsItem}>件数：{data.totalNumbers}件</Text>
                        <Text style={styles.textDetailsItem}>下单时间：{moment(data.createTime).format('YYYY-MM-DD')}</Text>
                    </View>
                </View>
                <View style={styles.containerWhite} />
                <View style={styles.containerOrderNumber}>
                    <Text style={styles.textDate}>创建时间：{data.createTime}</Text>
                </View>
                <View style={styles.containerInfo}>
                    <Text style={styles.textInfoTop}>运费：¥{ data.realFee }</Text>
                    <Text style={styles.textInfo}>保险费：¥{app.utils.N(data.needPayInsuanceFee)}</Text>
                    <Text style={styles.textInfo}>指定收款：¥{app.utils.N(data.totalDesignatedFee)}</Text>
                    <Text style={styles.textInfo}>代收货款：¥{app.utils.N(data.proxyCharge)}</Text>
                    <Text style={styles.textInfo}>支付方式：{this.toPayMode(data.payMode)}</Text>
                    {
                        stateList[0].state > OS.OS_READY_PAYMENT ?
                        <Text style={styles.textTotal}>已支付：¥{app.utils.N(data.needPayInsuanceFee + data.needPayTransportFee)}</Text>
                        :
                        <Text style={styles.textTotal}>需支付：¥{app.utils.N(data.needPayInsuanceFee + data.needPayTransportFee)}</Text>
                    }
                </View>
                {
                    stateList[0].state >= OS.OS_ON_THE_WAY &&
                    <TouchableOpacity
                        onPress={this.doLookTransport}
                        style={styles.transportInfo}>
                        <Text style={styles.textTransportInfo}>物流信息</Text>
                        <View style={styles.transportInfoRight}>
                            <Text style={styles.textTransportInfoRight}
                                numberOfLines={1}
                                ellipsizeMode='tail'>
                                {list && list[0].address}
                            </Text>
                            <Image
                                style={styles.imageCommon_go}
                                resizeMode='stretch'
                                source={app.img.common_go}
                                />
                        </View>
                    </TouchableOpacity>
                }
                {
                    this.state.canPay &&
                    <View style={styles.bottomView}>
                        <View style={styles.bottomInfo}>
                            <View style={styles.infoLeft}>
                                <Text style={styles.totalLeft}>总计:</Text>
                                <Text style={styles.totalRight}>¥{app.utils.N(data.needPayTransportFee + data.needPayInsuanceFee)}</Text>
                            </View>
                            <Button onPress={this.doShowActionSheet} style={styles.btnPayNow} textStyle={styles.btnPayNowText}>
                                立即支付
                            </Button>
                        </View>
                    </View>
                }
                <CustomSheet
                    visible={this.state.actionSheetVisible} >
                    <PayInfo doClose={this.doCloseActionSheet} data={this.props.data} doConfirmPay={this.doConfirmPay} type={'send'} />
                </CustomSheet>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    containerTop:{
        height:60,
        backgroundColor:'#c81622',
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'center',
    },
    imageTop:{
        height:32,
        width:27,
    },
    textTop:{
        fontSize:12,
        fontWeight:'900',
        marginLeft:20,
        color:'#ffffff',
    },
    containerReceive:{
        marginTop:8,
        paddingLeft:10,
        height:55,
        backgroundColor:'#ffffff',
        justifyContent: 'center',
    },
    textAddress:{
        fontSize:12,
        marginBottom:3,
        color:'#333333',
        flex:1,
    },
    textName:{
        fontSize:12,
        color:'#333333',
        flex:1,
    },
    containerMarket:{
        height:25,
        marginTop:8,
        alignItems: 'center',
        justifyContent: 'space-between',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    containerMarketLeft:{
        flexDirection: 'row',
    },
    marketLogo:{
        height:13,
        width:16.5,
        marginLeft:10,
    },
    textMarket:{
        flex:1,
        marginLeft:10,
        color:'#333333',
        fontSize:12,
    },
    orderState:{
        color:'#00a117',
        marginRight:10,
        fontSize:12,
    },
    containerDetails:{
        height:105,
        flexDirection: 'row',
    },
    containerWhite:{
        height:8,
        backgroundColor:'#ffffff',
    },
    imageGoods:{
        height:75,
        width:75,
        marginLeft:12,
        marginTop:11,
    },
    goodsDetails:{
        marginLeft:8,
        height:95,
        paddingBottom:8,
        justifyContent: 'center',
    },
    goodsDetailsTop:{
        width:sr.w - 99,
        flexDirection: 'row',
        justifyContent: 'space-between',
    },
    detailsTopLeft:{
        flexDirection: 'row',
        alignItems: 'center',
        marginTop:15,
    },
    detailsTopRight:{
        flexDirection: 'row',
        alignItems: 'center',
        marginTop:15,
    },
    detailsAddress:{
        fontSize:14,
        width:80,
        color:'#333333',
        flexDirection: 'row',

    },
    moneyLeft:{
        fontSize:12,
        color:'#333333',
    },
    moneyRight:{
        fontSize:13,
        fontWeight:'500',
        color:'#f01114',
        marginRight:10,
    },
    textDetailsItem:{
        marginTop:1,
        fontSize:12,
        color:'#888888',
    },
    containerOrderNumber:{
        height:50,
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingLeft:10,
        justifyContent: 'center',
    },
    textOrderNumber:{
        color:'#333333',
        marginBottom:3,
        fontSize:12,
    },
    textDate:{
        color:'#888888',
        marginTop:3,
        fontSize:14,
    },
    containerInfo:{
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingLeft:10,
        paddingBottom:5,
        paddingTop:5,
    },
    textInfoTop:{
        color:'#888888',
        fontSize:12,
        height:15,
    },
    textInfo:{
        color:'#888888',
        fontSize:12,
        marginTop:3,
        height:15,
    },
    textTotal:{
        marginTop:3,
        fontSize:12,
        color:'#333333',
        fontWeight:'400',
        height:15,
    },
    transportInfo:{
        height:40,
        backgroundColor:'#ffffff',
        marginTop:8,
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textTransportInfo:{
        color:'#888888',
        fontSize:14,
        marginLeft:10,
    },
    transportInfoRight:{
        marginRight:10,
        flexDirection: 'row',
        alignItems: 'center',
    },
    textTransportInfoRight:{
        fontSize:12,
        color:'#82c186',
        width: sr.w / 3 * 2,
        textAlign: 'right',
        marginRight:5,
    },
    imageCommon_go:{
        width: 8,
        height: 15,
        marginRight: 7,
    },
    choose:{
        marginLeft:10,
        flexDirection: 'row',
        alignItems: 'center',
    },
    imageChoose:{
        height:15,
        width:15,
    },
    bottomView:{
        position:'absolute',
        bottom:0,
        width:sr.w,
    },
    bottomInfo:{
        height:45,
        flexDirection: 'row',
        backgroundColor:'white',
        marginBottom:0,
    },
    infoLeft:{
        height:45,
        flexDirection: 'row',
        flex:1,
        alignItems: 'center',
    },
    btnPayNow:{
        height:45,
        flex:1,
        borderRadius:0,
    },
    btnPayNowText:{
        fontSize:16,
        fontWeight:'500',
    },
    totalLeft:{
        fontSize:12,
        color:'#333333',
        marginLeft:10,
    },
    totalRight:{
        fontSize:13,
        fontWeight:'500',
        color:'#f01114',
        marginLeft:5,
    },
});

```

* PDClient/project/App/modules/common/SendOrderList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

import _ from 'lodash';

const { PageList, Button, CustomSheet, DImage } = COMPONENTS;
const SendOrderDetail = require('./SendOrderDetail');
const LogisticsDetail = require('./LogisticsDetail');
const PayPassword = require('./PayPassword');
const PayInfo = require('./PayInfo');

let sumWeight = 0;
let sumCount = 0;
let sumSize = 0;
let money = 0;

module.exports = React.createClass({
    getInitialState () {
        return {
            paid: false,
            list:[],
            data: {},
            orderIdList: [],
        };
    },
    getOrderDetail (obj, index) {
        const param = {
            userId: app.personal.info.userId,
            orderId: obj.id,
        };
        POST(app.route.ROUTE_GET_ORDER_DETAIL, param, this.getOrderDetailSuccess.bind(null, index), true);
    },
    getOrderDetailSuccess (index, data) {
        const { success, msg } = data;
        if (success) {
            this.setState({
                orderIdList: [],
                list: [],
            });
            sumWeight = 0;
            sumCount = 0;
            sumSize = 0;
            money = 0;
            app.push({
                component: SendOrderDetail,
                passProps: {
                    data: data.context,
                    type: 1,
                    updateList: (data) => {
                        this.listView.updateList((list) => {
                            list[index] = data;
                            return list;
                        });
                    },
                },
            });
        } else {
            Toast(msg);
        }
    },
    doLookTransport (obj) {
        app.push({
            component : LogisticsDetail,
            passProps:{
                obj,
            },
        });
    },
    doPay (obj, rowId) {
        const param = {
            userId: app.personal.info.userId,
            orderId: obj.id,
        };
        POST(app.route.ROUTE_GET_ORDER_DETAIL, param, this.doPaySuccess.bind(null, rowId), false);
    },
    doPaySuccess (index, data) {
        const { success, msg } = data;
        if (success) {
            this.setState({
                data: data.context,
                orderIdList: [],
                list: [],
            });
            sumWeight = 0;
            sumCount = 0;
            sumSize = 0;
            money = 0;
            app.push({
                component: SendOrderDetail,
                passProps: {
                    canPay:true,
                    data: data.context,
                    type: 1,
                    alreadyPaid: this.alreadyPaid,
                    actionSheetVisible: false,
                },
            });
        } else {
            Toast(msg);
        }
    },
    alreadyPaid () {
        this.setState({ paid: true });
    },
    doLookDetail (obj, index) {
        const param = {
            userId: app.personal.info.userId,
            orderId: obj.id,
        };
        POST(app.route.ROUTE_GET_ORDER_DETAIL, param, this.getOrderDetailSuccess.bind(null, index), true);
    },
    renderSeparator (sectionID, rowId) {
        return (
            <View style={styles.separator} key={sectionID + rowId} />
        );
    },
    doChoose (obj, rowId) {
        const { list, orderIdList } = this.state;
        list[rowId].selected = !list[rowId].selected;
        if (_.findIndex(orderIdList, o => o == obj.id) == -1) {
            orderIdList.push(obj.id);
        } else {
            this.setState({ orderIdList: _.reject(orderIdList, o => o == obj.id) });
        }
        this.setState({
            list,
        });
        this.doTotalCalculation();
    },
    doSelectedAll () {
        let { list, orderIdList } = this.state;
        if (_.every(list, o => o.selected == true)) {
            _.forEach(list, o => { o.selected = false; });
            _.forEach(list, o => {
                this.setState({ orderIdList: [] });
            });
        } else {
            _.forEach(list, o => { o.selected = true; });
            _.forEach(list, o => { orderIdList.push(o.item.id); });
            this.setState({
                orderIdList: _.uniq(orderIdList),
            });
        }
        this.setState({ list });
        this.doTotalCalculation();
    },
    doTotalCalculation () {
        const { list } = this.state;
        sumWeight = 0;
        sumCount = 0;
        sumSize = 0;
        money = 0;
        _.filter(list, (o) => o.selected == true).forEach((item, i) => {
            sumSize = sumSize + item.item.size;
            sumCount = sumCount + 1;
            sumWeight = sumWeight + item.item.weight;
            money = money + item.item.needPayTransportFee + item.item.needPayInsuanceFee;
        });
    },
    doCloseActionSheet () {
        this.setState({ actionSheetVisible:false });
    },
    doShowActionSheet () {
        const { orderIdList, list } = this.state;
        if (orderIdList.length == 0) {
            Toast('请选择您要支付的货单');
            return;
        }
        if (!(_.find(list, o => o.item.stateList[0].state == OS.OS_READY_PAYMENT) || _.find(list, o => o.item.stateList[0].state == OS.OS_READY_DEDUCT_AND_PRINT_ORDER))) {
            Toast('请选择待支付的订单');
            return;
        }
        this.setState({ actionSheetVisible:true });
    },
    doConfirmPay () {
        const { orderIdList, list } = this.state;
        const param = {
            userId: app.personal.info.userId,
            orderIdList: orderIdList,
        };
        app.push({
            component:PayPassword,
            passProps:{
                param,
                routeName : !_.find(_.filter(list, (o) => o.selected == true), o => o.item.agent) ? app.route.ROUTE_PAY_FOR_ORDER_WHEN_SEND : app.route.ROUTE_PAY_AGENT_FOR_ORDER,
                prompt : '支付成功',
                doCloseActionSheet : this.doCloseActionSheet,
                alreadyPaid: this.alreadyPaid,
                money,
            },
        });
    },
    renderRow (obj, rowId) {
        const { stateList } = obj;
        const { type } = this.props;
        const { paid, list } = this.state;
        if (type == 'topay') {
            if (!_.find(list, (o) => o.index == rowId)) {
                list.push({ selected:false, index:rowId, item:obj });
            }
        }
        return (
            <View style={styles.row}>
                <View style={styles.rowTopContainer}>
                    <Image
                        resizeMode='stretch'
                        source={app.img.order_logo}
                        style={styles.topImage} />
                    <Text style={styles.topLeftText}>{obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : ''}</Text>
                    <Text style={styles.topRightText}>{stateList ? OS.MAP[stateList[0].state] : ''}</Text>
                </View>
                <View style={styles.middle}>
                    {
                        type == 'topay' &&
                        <TouchableOpacity onPress={this.doChoose.bind(null, obj, rowId)}
                            style={styles.btnChoose}>
                            <Image
                                resizeMode='stretch'
                                source={!list[rowId].selected ? app.img.order_no_choose : app.img.order_choose}
                                style={styles.choose} />
                        </TouchableOpacity>
                    }
                    <TouchableOpacity style={styles.rowMiddleContainer} onPress={this.getOrderDetail.bind(null, obj, rowId)}>
                        <DImage
                            resizeMode='stretch'
                            defaultSource={app.img.common_default}
                            source={{ uri: obj.photo }}
                            style={styles.middleImage} />
                        <View style={styles.middleRight}>
                            <View style={styles.pointContainer}>
                                <View style={styles.placeContainer}>
                                    <Text style={styles.pointName}
                                        numberOfLines={1}
                                        ellipsizeMode='tail'>{obj.shop ? obj.shop.address : obj.agent ? obj.agent.address : ''}</Text>
                                    <Text style={styles.pointText}>→</Text>
                                    <Text style={styles.pointName}
                                        numberOfLines={1}
                                        ellipsizeMode='tail'>{obj.endPoint}</Text>
                                </View>
                                {
                                    stateList[0].state > OS.OS_READY_PAYMENT ?
                                    <Text style={styles.transformFee}>{'已付款：'}<Text style={styles.money}>￥{parseInt(obj.needPayTransportFee + obj.needPayInsuanceFee)}</Text></Text>
                                    :
                                    <Text style={styles.transformFee}>{'需付款：'}<Text style={styles.money}>￥{parseInt(obj.needPayTransportFee + obj.needPayInsuanceFee)}</Text></Text>
                                }
                            </View>
                            <Text style={styles.commonInfo}>重量：{obj.weight}吨</Text>
                            <Text style={styles.commonInfo}>方量：{obj.size}m³</Text>
                            <Text style={styles.commonInfo}>件数：{obj.totalNumbers}件</Text>
                            <Text style={styles.commonInfo}>下单时间：{obj.createTime}</Text>
                        </View>
                    </TouchableOpacity>
                </View>
                <View style={styles.rowButtomContainer}>
                    {
                        stateList[0].state == OS.OS_READY_PAYMENT ?
                        <Button
                            style={styles.lookTransport}
                            disable={paid}
                            onPress={this.doPay.bind(null, obj, rowId)}
                            textStyle={styles.btnTransportText}>
                            {!paid ? '支付' : '已支付'}
                        </Button>
                        :
                        stateList[0].state >= OS.OS_ON_THE_WAY ?
                        <Button
                            onPress={this.doLookTransport.bind(null, obj)}
                            style={styles.lookTransport}
                            textStyle={styles.btnTransportText}>
                            {'查看物流'}
                        </Button>
                        :
                        <View />
                    }
                    <Button onPress={this.doLookDetail.bind(null, obj, rowId)} style={styles.lookDetail} textStyle={styles.btnDetallText}>{'查看详情'}</Button>
                </View>
            </View>
        );
    },
    render () {
        const { index, tabIndex, type, data,listUrl  } = this.props;
        const { list, actionSheetVisible, orderIdList } = this.state;
        const status = list.length != 0 && _.every(list, (o) => o.selected == true);
        return (
            <View style={index === tabIndex ? styles.container : { left:-sr.tw, top:0, position:'absolute' }}>
                <View style={styles.separator} />
                    {
                        !!data &&
                        <PageList
                            ref={(ref) => { this.listView = ref; }}
                            renderRow={this.renderRow}
                            renderSeparator={this.renderSeparator}
                            listParam={{ userId: app.personal.info.userId, type: type }}
                            listName={type + '.list'}
                            list={data.list}
                            autoLoad={false}
                            listUrl={listUrl}
                            refreshEnable
                            />
                    }
                {
                    type == 'topay' &&
                    <View style={styles.selectedAllContainer}>
                        <View style={styles.totalLeft}>
                            <TouchableOpacity onPress={this.doSelectedAll}
                                style={styles.selectedAllBtn}>
                                <Image style={styles.chooseAll}
                                    source={status ? app.img.order_choose : app.img.order_no_choose} />
                                <Text style={styles.selectedAllText}>全选</Text>
                            </TouchableOpacity>
                            <Text style={styles.totalText}>总计</Text>
                            <Text style={styles.totalItem}>{sumWeight}吨</Text>
                            <Text style={styles.totalItem}>{sumSize}m³</Text>
                            <Text style={styles.totalItemRight}>{sumCount}单</Text>
                        </View>
                        <TouchableOpacity onPress={this.doShowActionSheet}
                            style={styles.pay}>
                            <Text style={styles.payText}>立即支付</Text>
                        </TouchableOpacity>
                    </View>
                }
                <CustomSheet
                    visible={actionSheetVisible} >
                    <PayInfo
                        orderCount={orderIdList.length}
                        doClose={this.doCloseActionSheet}
                        money={money}
                        doConfirmPay={this.doConfirmPay}
                        type={'all'} />
                </CustomSheet>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f4f4f4',
    },
    row: {
        paddingVertical: 2,
        paddingHorizontal: 2,
    },
    separator: {
        backgroundColor: '#f4f4f4',
        height: 8,
    },
    rowTopContainer: {
        paddingHorizontal:10,
        paddingVertical:5,
        backgroundColor:'white',
        flexDirection: 'row',
    },
    topImage: {
        width: 17,
        height:17,
    },
    topLeftText: {
        marginLeft:10,
        flex:1,
        fontSize: 14,
    },
    topRightText: {
        flex:1,
        fontSize: 14,
        textAlign:'right',
        color:'#f51b1a',
    },
    middle:{
        flexDirection:'row',
        height:100,
        alignItems:'center',
        width:sr.w,
        backgroundColor:'#f8f8f8',
    },
    rowMiddleContainer: {
        flexDirection: 'row',
        alignItems: 'center',
        paddingVertical:11,
        height:100,
    },
    btnChoose:{
        height:100,
        width:20,
        marginRight:3,
        justifyContent:'center',
        alignItems:'center',
    },
    choose:{
        height:12,
        width:12,
    },
    middleImage: {
        width: 80,
        height:80,
        borderRadius: 10,
    },
    middleRight: {
        paddingLeft: 10,
        width:sr.w - 105,
    },
    money: {
        fontSize: 12,
        color:'#f51b1a',
    },
    commonInfo: {
        fontSize: 13,
        color:'#888888',
    },
    pointName: {
        fontSize: 13,
        maxWidth: 60,
    },
    pointText:{
        marginLeft:5,
        marginRight:10,
    },
    transformFee: {
        fontSize: 12,
        color: 'gray',
        marginRight:10,
        textAlign:'right',
    },
    pointContainer: {
        flexDirection: 'row',
        marginBottom:4,
    },
    placeContainer: {
        flex:2,
        flexDirection: 'row',
    },
    rowButtomContainer: {
        backgroundColor:'white',
        flexDirection: 'row',
        justifyContent: 'flex-end',
        paddingVertical:5,
    },
    lookTransport: {
        height:20,
        width:73,
        borderRadius:5,
        borderColor:'#ff6200',
        borderWidth:1,
        backgroundColor:'white',
    },
    lookDetail: {
        height:20,
        width:73,
        borderRadius:5,
        borderColor:'#007ef7',
        borderWidth:1,
        marginLeft:20,
        marginRight:10,
        backgroundColor:'white',
    },
    btnTransportText:{
        color:'#ff6200',
        fontSize:12,
        fontWeight:'300',
    },
    btnDetallText:{
        color:'#007ef7',
        fontSize:12,
        fontWeight:'300',
    },
    totalLeft:{
        flex:7,
        flexDirection:'row',
        alignItems:'center',
    },
    selectedAllContainer:{
        height:50,
        width:sr.w,
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
        alignItems:'center',
    },
    selectedAllBtn:{
        flexDirection:'row',
        alignItems:'center',
        height:40,
    },
    chooseAll:{
        height:12,
        width:12,
        marginLeft:5,
    },
    selectedAllText:{
        color:'#333333',
        fontSize:12,
        marginLeft:10,
    },
    totalText:{
        color:'#333333',
        fontSize:15,
        fontWeight:'500',
        marginLeft:20,
    },
    totalItem:{
        width:45,
        fontSize:12,
        color:'#f01823',
        textAlign:'right',
        marginLeft:5,
    },
    totalItemRight:{
        width:42,
        fontSize:12,
        color:'#f01823',
        textAlign:'right',
        marginRight:5,
    },
    pay:{
        flex:3,
        marginLeft:10,
        height:50,
        width:150,
        justifyContent:'center',
        alignItems:'center',
        backgroundColor:'#c81622',
    },
    payText:{
        fontSize:14,
        fontWeight:'500',
        color:'#FFFFFF',
        letterSpacing:2,
    },
});

```

* PDClient/project/App/modules/common/ShowBigImage.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
    TouchableOpacity,
} = ReactNative;

import Swiper from 'react-native-swiper';

module.exports = React.createClass({
    getInitialState () {
        const list = [];
        const list1 = [];
        this.props.defaultImageArray.map((item, i) => {
            if (i >= this.props.defaultIndex) {
                list1.push(item);
            } else {
                list.push(item);
            }
        });
        return {
            imageList: list1.concat(list),
        };
    },
    render () {
        return (
            <View style={styles.overlayContainer}>
                <Swiper
                    paginationStyle={styles.paginationStyle}
                    height={sr.th}>
                    {
                           this.state.imageList.map((item, i) => {
                               return (
                                   <TouchableOpacity
                                       key={i}
                                       activeOpacity={1}
                                       onPress={app.closeModal}
                                       style={styles.imageContainer}>
                                       <Image
                                           resizeMode='contain'
                                           defaultSource={app.img.common_default}
                                           source={{ uri: item }}
                                           style={styles.bannerImage}
                                           />
                                   </TouchableOpacity>
                               );
                           })
                       }
                </Swiper>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    overlayContainer: {
        flex: 1,
        alignItems:'center',
        justifyContent: 'center',
    },
    paginationStyle: {
        bottom: 30,
    },
    imageContainer: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
    },
    bannerImage: {
        alignSelf: 'center',
        width: sr.w,
        flex: 1,
    },
});

```

* PDClient/project/App/modules/common/VerificationCode.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    TextInput,
    Text,
    TouchableOpacity,
} = ReactNative;

const { Button } = COMPONENTS;
const TimerMixin = require('react-timer-mixin');

module.exports = React.createClass({
    mixins: [TimerMixin],
    statics: {
        title: '代确认收货',
    },
    getInitialState () {
        return {
            readSecond:false,
            time:59,
            verifyCode:'',
        };
    },
    doPress () {
        const { phone } = this.props;
        const param = {
            phone,
        };
        POST(app.route.ROUTE_REQUEST_SEND_VERIFY_CODE, param, this.requestSendVerifyCodeSuccess, false);
    },
    requestSendVerifyCodeSuccess (data) {
        if (data.success) {
            this.timer();
        } else {
            Toast(data.msg);
        }
    },
    doNext () {
        const { verifyCode } = this.state;
        const { orderId } = this.props;
        const param = {
            userId: app.personal.info.userId,
            orderId,
            verifyCode,
        };
        POST(app.route.ROUTE_FINISH_ORDER, param, this.finishOrderSuccess, false);
    },
    finishOrderSuccess (data) {
        if (data.success) {
            Toast('代确认收货成功');
            app.pop();
        } else {
            Toast('代确认收货失败');
        }
    },
    timer () {
        this.setState({ readSecond:true });
        this.setTimeout(
            () => {
                this.setState({ time:(this.state.time - 1) });
                if (this.state.time <= 0) {
                    return this.setState({ readSecond:false, time:59 });
                }
                this.timer();
            },
            1000
        );
    },
    render () {
        const { readSecond, time } = this.state;
        const { phone } = this.props;
        return (
            <View style={styles.container}>
                <Text style={styles.phone}>收货人手机号:{phone}</Text>
                <View style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <TextInput placeholderTextColor='#aaaaaa'
                            placeholder='请填写验证码'
                            maxLength={6}
                            keyboardType='phone-pad'
                            underlineColorAndroid='transparent'
                            onChangeText={(text) => this.setState({ verifyCode: text })}
                            style={styles.itemNameText} />
                        <TouchableOpacity
                            disabled={readSecond}
                            style={styles.btnBind}
                            onPress={this.doPress}>
                            {
                                !readSecond ?
                                <Text style={readSecond ? styles.btnBindTextGray : styles.btnBindText}>获取验证码</Text>
                                :
                                <Text style={readSecond ? styles.btnBindTextGray : styles.btnBindText}>{time}s后重新发送</Text>
                            }
                        </TouchableOpacity>
                    </View>
                </View>
                <Button onPress={this.doNext}
                    style={styles.btnNext}
                    textStyle={styles.btnNextText} >
                    代确认收货
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    phone:{
        marginTop:10,
        marginLeft:8,
        marginBottom:5,
        fontSize:14,
        color:'#333333',
    },
    ItemBgTop: {
        height: 45,
        marginTop:10,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
    },
    ItemBg: {
        height: 45,
        marginTop:1,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
    },
    infoStyle: {
        width:sr.w,
        flexDirection: 'row',
        alignItems:'center',
        justifyContent: 'space-between',
    },
    icon_add:{
        width:25,
        height:25,
        marginLeft: 8,
    },
    itemNameText: {
        flex:1,
        fontSize: 14,
        color: '#444444',
        marginLeft: 10,
    },

    btnNext:{
        marginTop:50,
        height:45,
        width:300,
        marginLeft:(sr.w - 300) / 2,
        borderRadius:4,
        backgroundColor:'#c81622',
    },
    btnNextText:{
        color:'#ffffff',
        fontSize:16,
        fontWeight:'500',
    },
    btnBind:{
        justifyContent:'center',
        alignItems:'center',
        borderLeftWidth:1,
        borderColor:'#f4f4f4',
        height:45,
    },
    btnBindText:{
        color:'#FF981A',
        fontSize:16,
        fontWeight:'500',
        width:sr.w - 254,
        textAlign:'center',
    },
    btnBindTextGray:{
        color:'#999999',
        backgroundColor:'#fafafa',
        borderLeftWidth:1,
        borderColor:'#f4f4f4',
        fontSize:16,
        fontWeight:'500',
        textAlign:'center',
        width:sr.w - 254,
        flex:1,
        textAlignVertical:'center',
    },
});

```

* PDClient/project/App/modules/person/About.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    WebView,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '关于软件',
    },
    render () {
        return (
            <WebView
                style={styles.container}
                source={{ uri: app.route.ROUTE_ABOUT_PAGE }}
                scalesPageToFit
                />
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
});

```

* PDClient/project/App/modules/person/AmountRule.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    WebView,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '担保金规则',
    },
    toAmountRule () {

    },
    render () {
        return (
            <WebView
                style={styles.container}
                source={{ uri: app.route.ROUTE_ABOUT_PAGE }}
                scalesPageToFit
            />
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
});

```

* PDClient/project/App/modules/person/BalanceDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '详情',
    },
    render () {
        const obj = this.props.obj;
        return (
            <View style={styles.container}>
                <View style={[styles.item, { paddingTop:2 }]}>
                    <Text style={styles.itemLeft}> 流水号 </Text>
                    <Text style={styles.itemRight}> {obj.thirdpartyAccount == null ? '内部支付' : obj.thirdpartyAccount} </Text>
                </View>
                <View style={styles.item}>
                    <Text style={styles.itemLeft}> 类型 </Text>
                    <Text style={styles.itemRight}> {obj.remark} </Text>
                </View>
                <View style={styles.item}>
                    <Text style={styles.itemLeft}> 金额 </Text>
                    <Text style={obj.tradeAmountYuan > 0 ? styles.itemMoneyGreen : styles.itemMoneyRed}>
                        {obj.tradeAmountYuan}
                    </Text>
                </View>
                <View style={styles.item}>
                    <Text style={styles.itemLeft}> 时间 </Text>
                    <Text style={styles.itemRight}> {obj.tradeTime} </Text>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#F4F4F4',
    },
    item:{
        marginTop:8,
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems:'center',
    },
    itemLeft:{
        fontSize:14,
        color:'#333333',
        paddingLeft:8,
    },
    itemRight:{
        fontSize:14,
        color:'#333333',
        paddingRight:10,
    },
    itemMoneyRed:{
        fontSize:14,
        color:'#C81622',
        paddingRight:10,
    },
    itemMoneyGreen:{
        fontSize:14,
        color:'#2F971C',
        paddingRight:10,
    },
});

```

* PDClient/project/App/modules/person/BalanceDetailList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableOpacity,
} = ReactNative;

const { PageList } = COMPONENTS;
const BalanceDetail = require('./BalanceDetail');

module.exports = React.createClass({
    statics: {
        title: '余额明细',
    },
    renderRow (obj) {
        return (
            <TouchableOpacity onPress={this.toAccountDetails.bind(null, obj)}>
                <View style={styles.recharge}>
                    <View>
                        <Text style={styles.rechargeText} numberOfLine={1}>
                            {obj.remark}
                        </Text>
                        <Text style={styles.rechargeDate}>
                            {obj.tradeTime}
                        </Text>
                    </View>
                    <Text style={obj.tradeAmountYuan > 0 ? styles.rechargeMoney : styles.expenditureMoney}>
                        {obj.tradeAmountYuan}
                    </Text>
                </View>
            </TouchableOpacity>
        );
    },
    toAccountDetails (obj) {
        app.push({
            component:BalanceDetail,
            passProps: {
                obj:obj,
            },
        });
    },
    render () {
        return (
            <View style={styles.container}>
                <PageList
                    ref={(ref) => { this.listView = ref; }}
                    renderRow={this.renderRow}
                    listParam={{ userId: app.personal.info.userId }}
                    listName={'billList'}
                    listUrl={app.route.ROUTE_GET_BILL_LIST}
                    refreshEnable
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#F4F4F4',
    },
    recharge:{
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
    },
    rechargeText:{
        paddingTop:5,
        paddingLeft:8,
        color:'#333333',
        fontSize:14,
        height:22,
        width:sr.w * 4 / 5,
    },
    rechargeDate:{
        color:'#aaaaaa',
        paddingVertical:5,
        paddingLeft:8,
        height:28,
    },
    rechargeMoney:{
        color:'#2F971C',
        fontSize:14,
        paddingRight:10,
        flex:1,
        textAlign:'right',
    },
    expenditureMoney:{
        color:'#C81622',
        fontSize:14,
        paddingRight:10,
        flex:1,
        textAlign:'right',
    },
});

```

* PDClient/project/App/modules/person/BindBankcard.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
} = ReactNative;

const { Button } = COMPONENTS;
module.exports = React.createClass({
    statics: {
        title: '绑定银行卡',
    },
    doBind () {

    },
    doScanBankcard () {

    },
    render () {
        return (
            <View style={styles.container}>
                <View style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <View style={styles.infoStyleText}>
                            <Text style={styles.itemNameText}>{'账号'}</Text>
                        </View>
                        <TextInput style={styles.itemNameLeftText}
                            underlineColorAndroid='transparent'
                            placeholderTextColor='#aaaaaa'
                            placeholder='请填入银行卡号'
                            onChangeText={this.onContentTextChange}
                            />
                    </View>
                </View>
                <View style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <View style={styles.infoStyleText}>
                            <Text style={styles.itemNameText}>{'姓名'}</Text>
                        </View>
                        <TextInput style={styles.itemNameLeftText}
                            underlineColorAndroid='transparent'
                            placeholderTextColor='#aaaaaa'
                            placeholder='请填入姓名'
                            onChangeText={this.onContentTextChange}
                            />
                    </View>
                </View>
                <View style={styles.lineView} />
                <Button onPress={this.doBind} style={styles.btnBind} textStyle={styles.btnBindText} >
                    绑定
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        paddingVertical: 10,
    },
    ItemBg: {
        padding: 10,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
        marginTop:1,
    },
    infoStyle: {
        flexDirection: 'row',
    },
    infoStyleText:{
        justifyContent:'center',
    },
    icon_add:{
        width:22,
        height:20,
        marginRight:5,
    },
    itemNameText: {
        fontSize: 15,
        color: '#444444',
        marginLeft: 8,
    },
    itemNameLeftText: {
        flex:1,
        marginTop:2,
        fontSize: 15,
        color: '#444444',
        marginLeft: 8,
    },
    lineView: {
        position:'absolute',
        top: 0,
        width: sr.w,
        height: 1,
        backgroundColor: '#f4f4f4',
    },
    btnBind:{
        marginTop:50,
        height:45,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        borderRadius:2,
        backgroundColor:'#c81622',
    },
    btnBindText:{
        color:'#ffffff',
        fontSize:16,
        fontWeight:'300',
    },
});

```

* PDClient/project/App/modules/person/BindBankcardList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
    ListView,
} = ReactNative;

const BindBankcard = require('./BindBankcard');
const { MessageBox } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '银行卡',
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            list: [1, 2],
        };
    },
    doAddBankcard () {
        app.push({
            component: BindBankcard,
        });
    },
    unBindBankcard (rowID) {
        app.showModal(
            <MessageBox
                onConfirm={this.doUnBindBankcard.bind(rowID)}
                content={'是否确认解绑该银行卡'}
                onCancel title={false}
                width={sr.s(250)} />
        );
    },
    doUnBindBankcard (rowID) {
        const { list } = this.state;
        list.splice(rowID, 1);
    },
    renderRow (obj, sectionID, rowID) {
        return (
            <View style={styles.ItemBg}>
                <View style={styles.infoStyle}>
                    <View >
                        <Text style={styles.itemNameText}>{'农业银行'}</Text>
                        <Text style={styles.itemNumberText}>{'**0851储蓄卡'}</Text>
                    </View>
                    <TouchableOpacity
                        activeOpacity={0.6}
                        onPress={() => this.unBindBankcard(rowID)}>
                        <Text style={styles.itemNameRightText}>{'解绑'}</Text>
                    </TouchableOpacity>
                </View>
            </View>
        );
    },
    renderFooter (obj, sectionID, rowId) {
        return (
            <TouchableOpacity
                activeOpacity={0.6}
                onPress={this.doAddBankcard}
                style={styles.touch_add}>
                <Image
                    resizeMode='stretch'
                    source={app.img.personal_add}
                    style={styles.icon_add} />
                <Text style={styles.itemAddText}>{'添加银行卡'}</Text>
            </TouchableOpacity>
        );
    },
    render () {
        return (
            <View style={styles.container}>
                <ListView
                    style={styles.ListView}
                    dataSource={this.ds.cloneWithRows(this.state.list)}
                    renderRow={this.renderRow}
                    renderFooter={this.renderFooter}
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    ItemBg: {
        height: 42,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
        marginTop:2,
    },
    ListView:{
        marginTop:8,
    },
    infoStyle: {
        width:sr.w,
        flexDirection: 'row',
        justifyContent: 'space-between',
    },
    touch_add:{
        alignItems:'center',
        marginTop:29,
    },
    icon_add:{
        width:30,
        height:30,
    },
    itemAddText: {
        fontSize: 14,
        color: '#888888',
        marginTop:10,
    },
    itemNameText: {
        fontSize: 14,
        color: '#333333',
        marginLeft: 8,
        marginBottom:2,
    },
    itemNumberText:{
        fontSize: 12,
        color: '#888888',
        marginLeft: 8,
        marginTop:2,
    },
    itemNameRightText: {
        textAlign:'right',
        fontSize: 13,
        color: '#3fa0ff',
        marginRight:10,
        marginTop:10,
    },
    lineView: {
        position:'absolute',
        top: 0,
        width: sr.w,
        height: 1,
        backgroundColor: '#f4f4f4',
    },
});

```

* PDClient/project/App/modules/person/BranchDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    ListView,
    TouchableOpacity,
} = ReactNative;

import moment from 'moment';
const GuaranteeDetail = require('./GuaranteeDetail');

module.exports = React.createClass({
    statics: {
        title: '入驻详情',
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            list:[],
            guaranteeList:[],
        };
    },
    componentWillMount () {
        this.getBranchShopInfo();
    },
    getBranchShopInfo () {
        const param = {
            userId:app.personal.info.userId,
            shopId:this.props.id,
        };
        POST(app.route.ROUTE_GET_BRANCH_SHOP_INFO, param, this.getBranchShopInfoSuccess, false);
    },
    getBranchShopInfoSuccess (data) {
        if (data.success) {
            this.setState({
                list:data.context,
                guaranteeList:data.context.bondCompanyList,
            });
        } else {
            Toast(data.msg);
        }
    },
    showGuaranteeDetail (obj) {
        app.push({
            component:GuaranteeDetail,
            passProps:{
                obj,
            },
        });
    },
    renderRow (obj, sectionID, rowID) {
        return (
            <TouchableOpacity style={styles.item} onPress={this.showGuaranteeDetail.bind(null, obj)}>
                <Text style={styles.itemFont}>{obj.name}</Text>
                <View style={styles.priceRow}>
                    <Text style={styles.itemPrice}>{obj.bondAmount > 100000 ?
                            (obj.bondAmount / 10000) + '万' : obj.bondAmount}</Text>
                    <Image resizeMode='contain'
                        style={styles.imgGo}
                        source={app.img.common_go} />
                </View>
            </TouchableOpacity>
        );
    },
    render () {
        const { list, guaranteeList } = this.state;
        const name = this.props.name;
        return (
            <View style={styles.container}>
                <View style={styles.top}>
                    <Text style={styles.topTitle}>{name}</Text>
                    <Text style={styles.topItem}>入驻时间：{moment(list.registerTime).format('YYYY-MM-DD')}</Text>
                    <Text style={styles.topItem}>担保金额：{list.totalBondAmount > 100000 ?
                            (list.totalBondAmount / 10000) + '万' : list.totalBondAmount}</Text>
                </View>
                <Text style={styles.title}>担保公司</Text>
                <ListView
                    dataSource={this.ds.cloneWithRows(guaranteeList)}
                    renderRow={this.renderRow}
                    enableEmptySections
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    top:{
        backgroundColor:'#FFFFFF',
        height:90,
        justifyContent:'center',
    },
    topTitle:{
        fontSize:14,
        fontWeight:'700',
        marginTop:13,
        marginLeft:10,
        flex:1,
    },
    topItem:{
        fontSize:12,
        marginLeft:10,
        flex:1,
        fontWeight:'300',
    },
    title:{
        height:20,
        color:'#666666',
        fontSize:14,
        marginLeft:10,
        marginTop:15,
    },
    item:{
        flexDirection:'row',
        backgroundColor:'#FFFFFF',
        marginTop:1,
        height:42,
        alignItems:'center',
    },
    priceRow:{
        flexDirection:'row',
        flex:1,
        justifyContent:'flex-end',
    },
    itemFont:{
        fontSize:14,
        marginLeft:10,
        fontWeight:'300',
    },
    itemPrice:{
        fontSize:14,
        fontWeight:'300',
    },
    imgGo:{
        width:15,
        height:15,
        marginLeft:5,
        marginRight:15,
    },
});

```

* PDClient/project/App/modules/person/BranchStore.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableOpacity,
    ListView,
} = ReactNative;

const BranchDetail = require('./BranchDetail');

module.exports = React.createClass({
    statics: {
        title: '入驻分店',
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            list:app.personal.info.shipper.registerShopList,
        };
    },
    showBranchDetail (id, name) {
        app.push({
            component:BranchDetail,
            passProps:{
                id,
                name,
            },
        });
    },
    renderRow (obj, sectionID, rowId) {
        return (
            <TouchableOpacity onPress={this.showBranchDetail.bind(null, obj.id, obj.name)}>
                <View style={styles.itemContainer}>
                    <Text style={styles.itemText}>{obj.name}</Text>
                </View>
                <View style={styles.listView} />
            </TouchableOpacity>
        );
    },
    render () {
        return (
            <View style={styles.container}>
                <ListView
                    style={styles.ListView}
                    dataSource={this.ds.cloneWithRows(this.state.list)}
                    renderRow={this.renderRow}
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        paddingTop:10,
    },
    itemContainer:{
        height:45,
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
    },
    itemText:{
        fontSize:14,
        color:'#333333',
        marginLeft:15,
    },
    itemTime:{
        fontSize:12,
        color: '#888888',
        marginLeft:15,
    },
    listView:{
        height:1,
    },
});

```

* PDClient/project/App/modules/person/CashBalance.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const WithdrawCash = require('./WithdrawCash');
const Recharge = require('./Recharge');
const BalanceDetailList = require('./BalanceDetailList');

module.exports = React.createClass({
    statics: {
        title: '余额',
        rightButton: { title: '明细', handler: () => { app.scene.detail&&app.scene.detail(); } },
    },
    getInitialState () {
        return {
            type:'1',
            remainAmount:app.personal.remainAmount,
        };
    },
    componentWillMount () {
        this.getRemainAmount();
    },
    getRemainAmount () {
        const param = {
            userId: app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_REMAIN_AMOUNT, param, this.getRemainAmountSuccess, false);
    },
    getRemainAmountSuccess (data) {
        if (data.success) {
            const context = data.context;
            app.personal.setRemainAmount(context.amount);
            this.setState({ remainAmount:app.personal.remainAmount });
        }
    },
    detail () {
        app.push({
            component:BalanceDetailList,
        });
    },
    showWithdrawCash () {
        app.push({
            component: WithdrawCash,
            passProps:{
                setRemainAmount : this.setRemainAmount,
            },
        });
    },
    showRecharge () {
        app.push({
            component: Recharge,
            passProps:{
                setRemainAmount : this.setRemainAmount,
            },
        });
    },
    setRemainAmount (amount) {
        this.setState({
            remainAmount : amount,
        });
    },
    render () {
        const { remainAmount } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.TopView}>
                    <Text style={styles.CashBalanceTitle}>账户余额</Text>
                    <View style={styles.CashBalance}>
                        <Text style={styles.CashBalanceTitleLeft}>￥</Text><Text style={styles.CashBalanceContent}>{app.utils.N(remainAmount)}</Text>
                    </View>
                </View>
                {
                    (_.intersection(app.personal.info.authority, [AH.AH_WITHDRAW]).length != 0 || app.authority.info.currentRole == '发货人') &&
                    <TouchableOpacity
                        activeOpacity={0.6}
                        onPress={this.showWithdrawCash}
                        style={styles.ItemBg}>
                        <View style={styles.infoStyle}>
                            <Text style={styles.itemNameText}>{'提现'}</Text>
                        </View>
                        <Image
                            resizeMode='stretch'
                            source={app.img.common_go}
                            style={styles.icon_go} />
                        <View style={styles.lineView} />
                    </TouchableOpacity>
                }
                {
                    (_.intersection(app.personal.info.authority, [AH.AH_RECHARGE]).length != 0 || app.authority.info.currentRole == '发货人') &&
                    <TouchableOpacity
                        activeOpacity={0.6}
                        onPress={this.showRecharge}
                        style={styles.ItemBg}>
                        <View style={styles.infoStyle}>
                            <Text style={styles.itemNameText}>{'充值'}</Text>
                        </View>
                        <Image
                            resizeMode='stretch'
                            source={app.img.common_go}
                            style={styles.icon_go} />
                        <View style={styles.lineView} />
                    </TouchableOpacity>
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    TopView: {
        height:100,
        backgroundColor:'#c81622',
        marginBottom:8,
        paddingLeft:20,
        paddingTop:15,
        paddingBottom:20,
    },
    CashBalanceTitle:{
        fontSize:14,
        color:'white',
    },
    CashBalance: {
        flexDirection: 'row',
        marginTop:22,
    },
    CashBalanceTitleLeft:{
        fontSize:14,
        color:'white',
        alignSelf:'flex-end',
        paddingBottom:5,
    },
    CashBalanceContent:{
        fontSize:24,
        color:'white',
        alignSelf:'flex-end',
    },
    ItemBg: {
        padding: 10,
        height: 44,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
    },
    infoStyle: {
        flexDirection: 'row',
    },
    itemNameText: {
        fontSize: 15,
        color: '#444444',
        marginLeft: 8,
    },
    icon_go: {
        width: 8,
        height: 15,
        marginRight: 7,
    },
    lineView: {
        position:'absolute',
        top: 0,
        width: sr.w,
        height: 1,
        backgroundColor: '#f4f4f4',
    },
});

```

* PDClient/project/App/modules/person/CommonSetting.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    Switch,
    View,
    Text,
    TouchableOpacity,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '基础设置',
    },
    getInitialState () {
        return {
            onlyWifiUpload: !!app.setting.data.onlyWifiUpload,
        };
    },
    onValueChange (value) {
        this.setState({ onlyWifiUpload: value });
        app.setting.setOnlyWifiUpload(value);
    },
    selectThemeColor (color) {
        app.setting.setThemeColor(color);
    },
    render () {
        return (
            <View style={styles.container}>
                <View style={styles.row}>
                    <Text style={styles.title}>只在wifi下上传</Text>
                    <Switch
                        style={styles.switch}
                        onValueChange={this.onValueChange}
                        value={this.state.onlyWifiUpload} />
                </View>
                <View style={styles.row}>
                    <Text style={styles.title}>主题颜色</Text>
                    <View style={styles.colorContainer}>
                        {
                            CONSTANTS.THEME_COLORS.map((color, i) => {
                                return (
                                    <TouchableOpacity
                                        key={i}
                                        onPress={this.selectThemeColor.bind(null, color)}
                                        style={[styles.color, { backgroundColor: color }]}
                                        />
                                );
                            })
                        }
                    </View>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        paddingVertical: 20,
    },
    row: {
        paddingHorizontal: 14,
        height: 80,
        flexDirection: 'row',
        backgroundColor: '#FFFFFF',
        alignItems: 'center',
    },
    title: {
        flex: 7,
        fontSize: 17,
        color: 'gray',
        marginLeft: 2,
    },
    switch: {
        flex: 2,
    },
    colorContainer: {
        flex: 20,
        flexDirection: 'row',
        justifyContent: 'space-around',
    },
    color: {
        flex: 1,
        height:80,
        marginRight: 10,
    },
});

```

* PDClient/project/App/modules/person/ConfirmPayPassword.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const DigitalKeyboard = require('../common/DigitalKeyboard');

const list = [1, 2, 3, 4, 5, 6, 7, 8, 9];

module.exports = React.createClass({
    statics: {
        title: '确认支付密码',
        leftButton: { handler: () => { app.scene.goBack&&app.scene.goBack(); } },
    },
    goBack () {
        this.props.toClear();
        app.pop();
    },
    getInitialState () {
        return {
            password: '',
        };
    },
    passwordItem (item) {
        const { password } = this.state;
        this.setState({ password: password + item });
    },
    zero () {
        const { password } = this.state;
        this.setState({ password: password + 0 });
    },
    delete () {
        const { password } = this.state;
        this.setState({
            password: _.dropRight(password).join(''),
        });
    },
    confirm () {
        const { password, type } = this.state;
        const tmp = password.substring(0, 6);
        if (tmp.length == 6) {
            this.toNext(tmp);
        } else {
            Toast('支付密码有误');
            this.toClear();
        }
    },
    toClear () {
        this.setState({
            password: '',
        });
    },
    toNext (tmp) {
        const { number, verifyCode } = this.props;
        if (number == tmp) {
            const param = {
                userId:app.personal.info.userId,
                verifyCode,
                phone:app.personal.info.phone,
                password : tmp,
            };
            POST(app.route.ROUTE_SET_PAYMENT_PASSWORD, param, this.modifySuccess, true);
        } else {
            Toast('两次输入密码不一致，请重试');
            this.goBack();
        }
    },
    modifySuccess (data) {
        if (data.success) {
            Toast('修改成功');
            const info = app.personal.info;
            info.isSetPaymentPassword = 1;
            app.personal.set(info);
            app.pop(3);
        } else {
            Toast(data.msg);
            app.pop(2);
        }
    },
    render () {
        const max = [1, 2, 3, 4, 5, 6];
        const { password, type } = this.state;
        return (
            <View style={styles.container}>
                <Text style={styles.text}>再次输入密码以作确认</Text>
                <View style={styles.password}>
                    {
                        max.map((item, i) => (
                            <View style={i == 0 ? styles.itemStart : styles.item} key={i}>
                                { password.length >= (i + 1) ?
                                    <View style={styles.point} />
                                    :
                                    <View style={styles.noPoint} />
                                }
                            </View>
                        ))
                    }
                </View>
                <View style={styles.digitalKeyboard} />
                <View>
                    <DigitalKeyboard list={list}
                        passwordItem={this.passwordItem}
                        zero={this.zero}
                        delete={this.delete}
                        confirm={this.confirm} />
                </View>
            </View>
        );
    },
});
const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#f4f4f4',
    },

    text:{
        fontSize:14,
        textAlign:'center',
        color:'#333333',
        marginTop:30,
    },
    password:{
        height:40,
        width:330,
        backgroundColor:'#ffffff',
        flexDirection:'row',
        marginTop:15,
        marginLeft:(sr.w - 330) / 2,
        borderRightWidth:1,
        borderBottomWidth:1,
        borderTopWidth:1,
        borderColor:'#EDEDED',
        borderRadius:4,
    },
    itemStart:{
        height:40,
        width:55,
        borderColor:'#EDEDED',
        justifyContent:'center',
        alignItems:'center',
    },
    item:{
        height:40,
        width:55,
        borderLeftWidth:1,
        borderColor:'#EDEDED',
        justifyContent:'center',
        alignItems:'center',
    },
    point:{
        height:10,
        width:10,
        borderRadius:5,
        backgroundColor:'#000000',
    },
    noPoint:{
        height:10,
        width:10,
        borderRadius:5,
        backgroundColor:'#FFFFFF',
    },
    digitalKeyboard:{
        flex:1,
    },
});

```

* PDClient/project/App/modules/person/EditPersonInfo.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    Image,
    Text,
    StyleSheet,
    View,
    TouchableOpacity,
} = ReactNative;

const { DImage, Picker } = COMPONENTS;

const ImagePicker = require('./ImagePicker');
const EditSingleInfo = require('./EditSingleInfo');

module.exports = React.createClass({
    mixins: [SceneMixin],
    statics: {
        title: '个人信息',
        leftButton: { handler: () => { app.scene.goBack&&app.scene.goBack(); } },
    },
    goBack () {
        this.props.updateInfo(app.personal.info);
        Picker.hide();
        app.pop();
    },
    getInitialState () {
        const info = app.personal.info;
        return {
            name: info.name,
            address: info.address,
            head: info.head,
            sex: info.sex === 0 ? '男' : '女',
            phone:info.phone,
            pickerData: ['男', '女'],
        };
    },
    toEditHead () {
        app.push({
            component: ImagePicker,
            passProps: {
                onCropImage: this.setUserHead,
            },
        });
    },
    setUserHead (uri) {
        this.setState({ head: uri });
    },
    toEditSingleInfo (name, tip, maxLength, checkFun) {
        Picker.hide();
        app.push({
            title: '修改' + tip,
            component: EditSingleInfo,
            passProps: {
                keyName:name,
                placeholder:tip,
                maxLength,
                updateSingleState:this.updateSingleState,
                checkFun,
            },
        });
    },
    updateSingleState (keyName, value) {
        let state = this.state;
        state[keyName] = value;
        this.setState({ state });
    },
    toEditSex () {
        const { sex, pickerData } = this.state;
        Picker(pickerData, [sex], '选择性别').then((value) => {
            const sex = value[0];
            this.setState({ sex }, () => {
                this.modifyPersonalInfo();
            });
        });
    },
    modifyPersonalInfo () {
        const { sex } = this.state;
        const param = {
            userId : app.personal.info.userId,
            sex : sex == '男' ? 0 : 1,
        };
        POST(app.route.ROUTE_MODIFY_PERSONAL_INFO, param, this.modifyUserInfoSuccess, true);
    },
    modifyUserInfoSuccess (data) {
        if (data.success) {
            let info = app.personal.info;
            app.personal.set(Object.assign(info, data.context));
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { name, address, sex, head, phone } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.headStyle} >
                    <TouchableOpacity
                        onPress={this.toEditHead}
                        style={styles.head_itemView}>
                        <DImage
                            defaultSource={app.img.personal_default_head}
                            source={{ uri: head }}
                            style={styles.headImgStyle} />
                        <View style={styles.rightView} >
                            <Image
                                source={app.img.common_go}
                                style={styles.goStyle} />
                        </View>
                    </TouchableOpacity>
                </View>
                <TouchableOpacity
                    onPress={this.toEditSingleInfo.bind(null, 'name', '昵称', CONSTANTS.NAME_MAX_LENGTH, app.utils.checkStr)}
                    style={styles.itemView} >
                    <Text style={styles.itemKeyText}>昵称</Text>
                    <View style={styles.rightView} >
                        <Text style={styles.itemValueText}>{name || app.personal.info.phone}</Text>
                        <Image
                            source={app.img.common_go}
                            style={styles.goStyle} />
                    </View>
                </TouchableOpacity>
                <TouchableOpacity
                    onPress={this.toEditSex}
                    style={styles.itemView}>
                    <Text style={styles.itemKeyText}>性别</Text>
                    <View style={styles.rightView} >
                        <Text style={styles.itemValueText}>{sex}</Text>
                        <Image
                            source={app.img.common_go}
                            style={styles.goStyle} />
                    </View>
                </TouchableOpacity>
                <TouchableOpacity
                    onPress={this.toEditSingleInfo.bind(null, 'address', '地址', CONSTANTS.ADDRESS_MAX_LENGTH, app.utils.checkStr)}
                    style={styles.itemView}>
                    <Text style={styles.itemKeyText} numberOfLines={1}>地址</Text>
                    <View style={styles.rightView} >
                        <Text style={styles.itemValueText}>{address}</Text>
                        <Image
                            source={app.img.common_go}
                            style={styles.goStyle} />
                    </View>
                </TouchableOpacity>
                <TouchableOpacity
                    style={styles.itemView}>
                    <Text style={styles.itemKeyText}>手机号码</Text>
                    <View style={styles.rightView} >
                        <Text style={styles.itemValueText}>{phone}</Text>
                    </View>
                </TouchableOpacity>
                {
                    (!app.personal.info.shipper && !app.personal.info.agent) &&
                    <Text />
                    ||
                    <TouchableOpacity
                        style={styles.itemView}>
                        <Text style={styles.itemKeyText}>所属{app.personal.info.shipper ? '物流公司' : '收货点'}</Text>
                        <View style={styles.rightView} >
                            <Text style={styles.itemValueText}>
                                {
                                    app.personal.info.shipper ?
                                    app.personal.info.shipper.name
                                    :
                                    app.personal.info.agent.name
                                }
                            </Text>
                        </View>
                    </TouchableOpacity>
                }
                {
                    (!app.personal.info.shipper && !app.personal.info.agent) &&
                    <Text />
                    ||
                    <TouchableOpacity
                        style={styles.itemView}>
                        <Text style={styles.itemKeyText}>职务</Text>
                        <View style={styles.rightView} >
                            <Text style={styles.itemValueText}>
                                {
                                    app.personal.info.shipper ?
                                    (app.personal.info.phone == app.personal.info.shipper.chairMan.phone ? '董事长' : '普通员工')
                                    :
                                    (app.personal.info.phone == app.personal.info.agent.chairMan.phone ? '董事长' : '普通员工')
                                }
                            </Text>
                        </View>
                    </TouchableOpacity>
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex:1,
        backgroundColor: '#F4F4F4',
        alignItems:'center',
    },
    headStyle: {
        width: sr.w,
        height:60,
        marginTop: 10,
    },
    head_itemView:{
        backgroundColor: 'white',
        flexDirection: 'row',
        height: 60,
        marginBottom: 8,
        justifyContent: 'space-between',
        alignItems:'center',
    },
    itemView: {
        marginTop:1,
        backgroundColor: 'white',
        flexDirection: 'row',
        height: 42,
        width:sr.w,
        justifyContent: 'space-between',
        alignItems:'center',
    },
    rightView: {
        flexDirection: 'row',
        height: 45,
        alignItems:'center',
    },
    headImgStyle: {
        height: 36,
        width: 36,
        borderRadius: 18,
        marginLeft: 20,
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 20,
    },
    separatorView: {
        height: 1,
        width: sr.w,
        backgroundColor: '#aab9ba',
    },
    itemKeyText: {
        color: '#444444',
        fontSize: 15,
        marginLeft: 20,
    },
    itemValueText: {
        color: '#888888',
        fontSize: 14,
        marginRight: 10,
    },
    btnChangeRole:{
        position:'absolute',
        top:sr.h - 210,
        marginTop:15,
        height:45,
        width:sr.w - 25,
        borderRadius:2,
    },
});

```

* PDClient/project/App/modules/person/EditSex.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    Text,
    StyleSheet,
    View,
    TouchableOpacity,
} = ReactNative;

const { Picker } = COMPONENTS;

module.exports = React.createClass({
    mixins: [SceneMixin],
    statics: {
        leftButton: { handler: () => { app.scene.goBack&&app.scene.goBack(); } },
    },
    getInitialState () {
        const info = app.personal.info;
        return {
            sex: info.sex === 0 ? '男' : '女',
            pickerData: ['男', '女'],
        };
    },
    goBack () {
        Picker.hide();
        app.pop();
    },
    toEditSex () {
        const { sex, pickerData } = this.state;
        Picker(pickerData, [sex], '选择性别').then((value) => {
            const sex = value[0];
            this.setState({ sex }, () => {
                this.modifyPersonalInfo();
            });
        });
    },
    modifyPersonalInfo () {
        const info = app.personal.info;
        const param = {
            userId: info.userId,
            sex: this.state.sex === '男' ? 0 : 1,
        };
        POST(app.route.ROUTE_MODIFY_PERSONAL_INFO, param, this.modifyUserInfoSuccess, true);
    },
    modifyUserInfoSuccess (data) {
        if (data.success) {
            let info = app.personal.info;
            info.sex = this.state.sex === '男' ? 0 : 1;
            app.personal.set(info);
            this.props.setSex(info.sex);
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { sex } = this.state;
        return (
            <View style={styles.container}>
                <View style={[styles.separatorView, { marginTop: 30 }]} />
                <TouchableOpacity
                    onPress={this.toEditSex}
                    style={styles.itemView}>
                    <Text style={styles.itemKeyText}>{sex || '请选择您的性别'}</Text>
                </TouchableOpacity>
                <View style={styles.separatorView} />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex:1,
        backgroundColor: '#EEEEEE',
    },
    itemView: {
        backgroundColor: 'white',
        height: 45,
        justifyContent: 'center',
        alignItems:'center',
    },
    separatorView: {
        height: 1,
        width: sr.w,
        backgroundColor: '#aab9ba',
    },
    itemKeyText: {
        color: '#444444',
        fontSize: 15,
        marginLeft: 20,
    },
});

```

* PDClient/project/App/modules/person/EditSingleInfo.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    Text,
    StyleSheet,
    View,
    TextInput,
} = ReactNative;

module.exports = React.createClass({
    mixins: [SceneMixin],
    statics: {
        rightButton: { title: '保存', handler: () => { app.scene.doSave&&app.scene.doSave(); } },
    },
    doSave () {
        const { placeholder, checkFun } = this.props;
        const info = app.personal.info;
        const { value } = this.state;
        if (!!checkFun && !checkFun(value)) {
            Toast('请填写正确的' + placeholder);
            return;
        }
        if (value.length == 0) {
            Toast(placeholder + '不能为空');
            return;
        }
        let param = {
            userId: info.userId,
        };
        param[this.keyName] = value;
        POST(app.route.ROUTE_MODIFY_PERSONAL_INFO, param, this.modifyUserInfoSuccess, true);
    },
    getInitialState () {
        const info = app.personal.info;
        this.keyName = this.props.keyName;
        return {
            value: info[this.keyName] || '',
        };
    },
    modifyUserInfoSuccess (data) {
        if (data.success) {
            let info = app.personal.info;
            info[this.keyName] = this.state.value;
            app.personal.set(info);
            this.props.updateSingleState(this.keyName, info[this.keyName]);
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { value } = this.state;
        const { placeholder, maxLength } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.itemView}>
                    <Text style={styles.itemKeyText} />
                    <TextInput
                        ref={(ref) => { this.contentInput = ref; }}
                        style={styles.rightView}
                        onChangeText={(text) => this.setState({ value: text })}
                        multiline={false}
                        maxLength={maxLength}
                        placeholder={'请输入你的' + placeholder}
                        autoCapitalize={'none'}
                        underlineColorAndroid={'transparent'}
                        defaultValue={value}
                        />
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex:1,
        backgroundColor: '#F4F4F4',
    },
    itemView: {
        marginTop:10,
        backgroundColor: 'white',
        flexDirection: 'row',
        height: 42,
        justifyContent: 'space-between',
        alignItems:'center',
    },
    rightView: {
        flex: 1,
        fontSize: 15,
        paddingVertical: 2,
        color: '#444444',
    },
    separatorView: {
        height: 1,
        width: sr.w,
        backgroundColor: '#aab9ba',
    },
    itemKeyText: {
        color: '#444444',
        fontSize: 15,
        marginLeft: 20,
    },
});

```

* PDClient/project/App/modules/person/Feedback.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    TextInput,
    Text,
} = ReactNative;

const { Button } = COMPONENTS;

module.exports = React.createClass({
    mixins: [SceneMixin],
    statics: {
        title: '意见反馈',
    },
    getInitialState () {
        return {
            content:'',
            length:0,
        };
    },
    onContentTextChange (text) {
        this.setState({
            content:text.trim(),
            length:text.length,
        });
    },
    doSubmit () {
        if (!this.state.content) {
            Toast('请填写您需要反馈的内容');
            return;
        }
        const param = {
            userId: app.personal.info.userId,
            content: this.state.content,
        };
        POST(app.route.ROUTE_SUBMIT_FEEDBACK, param, this.doSubmitSuccess);
    },
    doSubmitSuccess (data) {
        if (data.success) {
            Toast('提交成功');
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const length = this.state.length;
        return (
            <View style={styles.container}>
                <Text style={styles.content_text}>
                    具体内容
                </Text>
                <TextInput style={styles.content_input}
                    underlineColorAndroid='transparent'
                    placeholderTextColor='#aaaaaa'
                    placeholder='请填入意见内容'
                    onChangeText={this.onContentTextChange}
                    multiline
                    maxLength={CONSTANTS.FEEDBACK_MAX_LENGTH}
                    />
                <Text style={styles.wordnumber_text}>
                    {length}/500
                </Text>
                <Button onPress={this.doSubmit}
                    style={styles.button}
                    textStyle={styles.btnLoginText}>
                    提交反馈
                    </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    content_text:{
        fontSize:14,
        height:40,
        width:351,
        color:'#666666',
        marginLeft:(sr.w - 351) / 2,
        paddingTop:18,
    },
    content_input:{
        fontSize:14,
        height:200,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        backgroundColor:'#FFFFFF',
        padding:3,
        textAlignVertical: 'top',
    },
    wordnumber_text:{
        fontSize:14,
        height:30,
        width:351,
        color:'#666666',
        marginLeft:(sr.w - 351) / 2,
        paddingTop:5,
        textAlign:'right',
    },
    button:{
        height:50,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        backgroundColor:'#f69934',
        borderRadius:4,
        marginTop:60,
    },
    btnLoginText:{
        fontSize:18,
        color:'#FFFFFF',
        fontWeight:'600',
    },
});

```

* PDClient/project/App/modules/person/GuaranteeAmount.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const AmountRule = require('./AmountRule');

module.exports = React.createClass({
    statics: {
        title: '担保金额',
    },
    getInitialState () {
        return {
            data:[],
        };
    },
    componentWillMount () {
        this.getBondAmountInfo();
    },
    getBondAmountInfo () {
        const param = {
            userId:app.personal.info.userId,
            shopId:app.personal.currentShopId,
        };
        POST(app.route.ROUTE_GET_BRANCH_SHOP_INFO, param, this.getBondAmountInfoSuccess, false);
    },
    getBondAmountInfoSuccess (data) {
        if (data.success) {
            this.setState({
                data:data.context,
            });
        } else {
            Toast(data.msg);
        }
    },
    toAmountRule () {
        app.push({
            component:AmountRule,
        });
    },
    render () {
        const { data } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.titleRow}>
                    <Text style={styles.titleAmount}>{data.remainBondAmount > 100000 ?
                            (app.utils.N(data.remainBondAmount / 10000)) + '万' : app.utils.N(data.remainBondAmount)}</Text>
                    <Text style={styles.titleFont}>可用担保金</Text>
                </View>
                <View style={styles.titleBottom}>
                    <Text style={styles.totalAmount}>总担保金：</Text>
                    <Text style={styles.totalAmount}>{data.totalBondAmount > 100000 ?
                            (app.utils.N(data.totalBondAmount / 10000)) + '万' : app.utils.N(data.totalBondAmount)}</Text>
                </View>
                <TouchableOpacity onPress={this.toAmountRule}>
                    <View style={styles.item}>
                        <Text style={styles.itemFont}>担保金规则</Text>
                        <Image source={app.img.common_go}
                            style={styles.imgGo}
                            resizeMode='cover' />
                    </View>
                </TouchableOpacity>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    titleRow:{
        backgroundColor:'#c81622',
        height:77,
        alignItems:'center',
    },
    titleAmount:{
        color:'#ffffff',
        fontSize:30,
        fontWeight:'600',
        flex:1,
    },
    titleFont:{
        color:'#ffffff',
        fontSize:14,
        fontWeight:'500',
        flex:1,
        marginTop:10,
    },
    titleBottom:{
        backgroundColor:'#b21715',
        height:35,
        width:sr.w,
        flexDirection:'row',
        alignItems:'center',
    },
    totalAmount:{
        color:'#ffffff',
        fontSize:14,
        fontWeight:'500',
        marginLeft:10,
    },
    item:{
        height:42,
        backgroundColor:'#ffffff',
        justifyContent:'center',
    },
    imgGo:{
        width:15,
        height:15,
        position:'absolute',
        left:(sr.w - 30),
    },
    itemFont:{
        marginLeft:10,
        fontSize:14,
        fontWeight:'300',
    },
});

```

* PDClient/project/App/modules/person/GuaranteeDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '担保公司',
    },
    render () {
        const { obj } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.item}>
                    <Text style={styles.itemLeft}>担保公司</Text>
                    <Text style={styles.itemRight}>{obj.name}</Text>
                </View>
                <View style={styles.item}>
                    <Text style={styles.itemLeft}>担保金额</Text>
                    <Text style={styles.itemRight}>{obj.bondAmount > 100000 ?
                            (obj.bondAmount / 10000) + '万' : obj.bondAmount}</Text>
                </View>
                <View style={styles.item}>
                    <Text style={styles.itemLeft}>担保公司注册资金</Text>
                    <Text style={styles.itemRight}>{obj.capital > 100000 ?
                            (obj.capital / 10000) + '万' : obj.capital}</Text>
                </View>
                <View style={styles.item}>
                    <Text style={styles.itemLeft}>法人姓名</Text>
                    <Text style={styles.itemRight}>{obj.legalName}</Text>
                </View>
                <View style={styles.item}>
                    <Text style={styles.itemLeft}>法人电话</Text>
                    <Text style={styles.itemRight}>{obj.legalPhone}</Text>
                </View>
                <View style={styles.item}>
                    <Text style={styles.itemLeft}>担保公司电话</Text>
                    <Text style={styles.itemRight}>{obj.phoneList}</Text>
                </View>
                <View style={styles.item}>
                    <Text style={styles.itemLeft}>担保公司地址</Text>
                    <Text style={styles.itemRight}>{obj.address}</Text>
                </View>
                <View style={styles.itemImage}>
                    <Text style={styles.itemTitle}>担保公司图片资料</Text>
                    <View style={styles.images}>
                        {
                            obj.certificate.map((item, i) => (
                                <Image key={i}
                                    resizeMode='contain'
                                    style={styles.img}
                                    source={{ uri:item }} />
                            ))
                        }
                    </View>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    item:{
        height:40,
        flexDirection:'row',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
        marginBottom:1,
    },
    itemLeft:{
        marginLeft:10,
        color:'#666666',
        flex:1,
    },
    itemRight:{
        width:200,
        textAlign:'right',
        marginRight:10,
    },
    itemImage:{
        height:155,
        backgroundColor:'#FFFFFF',
    },
    itemTitle:{
        marginLeft:10,
        marginTop:12,
        marginBottom:15.5,
        color:'#666666',
        fontSize:14,
    },
    images:{
        marginLeft:5,
        flexDirection:'row',
        width:sr.w - 5,
    },
    img:{
        flex:1,
        height:115,
        marginRight:5,
    },
});

```

* PDClient/project/App/modules/person/Help.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    WebView,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '软件许可协议',
    },
    render () {
        return (
            <WebView
                style={styles.container}
                source={{ uri:app.route.ROUTE_SOFTWARE_LICENSE }}
                scalesPageToFit
                />
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
});

```

* PDClient/project/App/modules/person/ImageCrop.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    ImageStore,
    ImageEditor,
} = ReactNative;

const ImageCrop = require('@remobile/react-native-image-crop');

module.exports = React.createClass({
    mixins: [SceneMixin],
    statics: {
        title: '编辑头像',
        rightButton: { title: '完成', handler: () => {
            app.scene.cropImage&&app.scene.cropImage();
        } },
    },
    goBack () {
        app.pop();
    },
    cropImage () {
        const { image } = this.props;
        const cropData = this.imageCrop.getCropData();
        ImageEditor.cropImage(
            image.uri,
            cropData,
            (croppedImageURI) => {
                this.uploadUserHead(croppedImageURI);
            },
            (error) => {
                Toast('文件出错');
                this.goBack();
            }
        );
    },
    uploadUserHead (filePath) {
        if (app.isandroid) {
            const options = {};
            options.fileKey = 'file';
            options.fileName = filePath.substr(filePath.lastIndexOf('/') + 1);
            options.mimeType = 'image/png';
            options.params = {
                userId:app.personal.info.userId,
            };
            UPLOAD(filePath, app.route.ROUTE_UPDATE_FILE, options, (progress) => console.log(progress), this.uploadSuccessCallback, this.uploadErrorCallback, true);
        } else {
            ImageStore.getBase64ForTag(filePath, (data) => {
                data = 'data:image/png;base64,' + data;
                const options = {};
                options.fileKey = 'file';
                options.fileName = 'user_head.png';
                options.mimeType = 'image/png';
                options.params = {
                    userId:app.personal.info.userId,
                };
                ImageStore.removeImageForTag(filePath);
                UPLOAD(data, app.route.ROUTE_UPDATE_FILE, options, (progress) => console.log(progress), this.uploadSuccessCallback, this.uploadErrorCallback, true);
            }, () => {
                Toast('文件出错');
                this.goBack();
            });
        }
    },
    uploadSuccessCallback (data) {
        if (data.success) {
            this.modifyPersonalInfo(data.context.url);
        } else {
            Toast('上传头像失败');
            this.goBack();
        }
    },
    uploadErrorCallback () {
        Toast('上传头像失败');
        this.goBack();
    },
    modifyPersonalInfo (url) {
        const info = app.personal.info;
        const param = {
            userId: info.userId,
            head: url,
        };
        POST(app.route.ROUTE_MODIFY_PERSONAL_INFO, param, this.modifyUserInfoSuccess.bind(null, url), this.modifyUserInfoError, true);
    },
    modifyUserInfoSuccess (url, data) {
        if (data.success) {
            let info = app.personal.info;
            info.head = url;
            app.personal.set(info);
            this.props.onCropImage(url);
            this.goBack();
        } else {
            Toast(data.msg);
            this.goBack();
        }
    },
    modifyUserInfoError () {
        Toast('修改头像失败');
        this.goBack();
    },
    render () {
        const { image } = this.props;
        return (
            <View style={styles.container}>
                <ImageCrop
                    imageWidth={image.width}
                    imageHeight={image.height}
                    editRectRadius={0}
                    ref={(ref) => { this.imageCrop = ref; }}
                    source={{ uri: image.uri }} />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
});

```

* PDClient/project/App/modules/person/ImagePicker.js

```js
const React = require('react');
const ReactNative = require('react-native');
const {
    Image,
} = ReactNative;

const CameraRollPicker = require('@remobile/react-native-camera-roll-picker');
const Camera = require('@remobile/react-native-camera');
const ImageCrop = require('./ImageCrop');

module.exports = React.createClass({
    statics: {
        title: '选择头像',
    },
    cropImage (image) {
        app.navigator.replace({
            component: ImageCrop,
            passProps: {
                image,
                onCropImage: this.props.onCropImage,
            },
        });
    },
    onSelectedImages (images, image) {
        this.cropImage(image);
    },
    openCamera () {
        Camera.getPicture((filePath) => {
            const uri = 'file://' + filePath;
            Image.getSize(uri, (width, height) => {
                this.cropImage({
                    uri,
                    width,
                    height,
                });
            });
        }, null, {
            quality: 100,
            allowEdit: false,
            cameraDirection: Camera.Direction.FRONT,
            destinationType: Camera.DestinationType.FILE_URI,
        });
    },
    render () {
        return (
            <CameraRollPicker selected={[]} onSelectedImages={this.onSelectedImages} openCamera={this.openCamera} maximum={1} />
        );
    },
});

```

* PDClient/project/App/modules/person/ModifyPassword.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    TextInput,
} = ReactNative;

const { Button, Label } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '修改密码',
    },
    getInitialState () {
        return {
            oldPassword: '',
            newPassword1: '',
            newPassword2: '',
        };
    },
    goBack () {
        app.pop();
    },
    doSubmit () {
        const { oldPassword, newPassword1, newPassword2 } = this.state;
        const { checkPassword } = app.utils;
        if (!checkPassword(oldPassword) || !checkPassword(newPassword1)) {
            Toast('密码必须有6-20位的数字，字母，下划线组成');
            return;
        }
        if (!oldPassword) {
            Toast('请填写旧密码');
            return;
        }
        if (!newPassword1) {
            Toast('请填写新密码');
            return;
        }
        if (!newPassword2) {
            Toast('请再次填写新密码');
            return;
        }
        if (newPassword2 !== newPassword1) {
            Toast('两次填写新密码不一致');
            return;
        }
        if (oldPassword === newPassword1) {
            Toast('新密码不能和原来密码一样');
            return;
        }
        const param = {
            userId: app.personal.info.userId,
            oldPassword,
            newPassword: newPassword1,
        };
        POST(app.route.ROUTE_MODIFY_PASSWORD, param, this.doSubmitSuccess, true);
    },
    doSubmitSuccess (data) {
        if (data.success) {
            Toast('修改成功');
            this.goBack();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        return (
            <View style={styles.container}>
                <Label img={app.img.login_user}>请输入旧密码</Label>
                <TextInput
                    underlineColorAndroid='transparent'
                    placeholder='请输入旧密码'
                    style={styles.text_input}
                    secureTextEntry
                    onChangeText={(text) => this.setState({ oldPassword: text })}
                    />
                <Label img={app.img.login_user}>请输入新密码</Label>
                <TextInput
                    underlineColorAndroid='transparent'
                    placeholder='请输入新密码'
                    style={styles.text_input}
                    secureTextEntry
                    onChangeText={(text) => this.setState({ newPassword1: text })}
                    />
                <Label img={app.img.login_user}>请再次输入新密码</Label>
                <TextInput
                    underlineColorAndroid='transparent'
                    placeholder='请再次输入新密码'
                    style={styles.text_input}
                    secureTextEntry
                    onChangeText={(text) => this.setState({ newPassword2: text })}
                    />
                <Button onPress={this.doSubmit} style={[styles.btnSubmit, { backgroundColor:app.setting.THEME_COLOR }]}>提交</Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex:1,
        backgroundColor: 'white',
        paddingVertical: 20,
        paddingHorizontal: 25,
    },
    text_input: {
        height: 40,
        fontSize: 14,
        paddingLeft: 10,
        backgroundColor: 'white',
        textAlignVertical: 'top',
        borderWidth: 1,
        borderRadius: 5,
        marginBottom: 20,
        borderColor: '#DBDBDB',
    },
    btnSubmit: {
        width: sr.w - 50,
        marginTop: 80,
        height: 40,
        borderRadius: 10,
    },
});

```

* PDClient/project/App/modules/person/ModifyProfit.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TextInput,
    TouchableOpacity,
} = ReactNative;

const { SelectRegionAddress } = COMPONENTS;

const { Button } = COMPONENTS;

const ChangeType = {
    PRICE:0,
    PERCENT:1,
};

module.exports = React.createClass({
    statics: {
        title: '批量修改',
    },
    getInitialState () {
        return {
            price: '',
            change: ChangeType.PRICE,
        };
    },
    toChangePrice () {
        this.setState({
            change: ChangeType.PRICE,
            price: '',
        });
    },
    toChangePercent () {
        this.setState({
            change: ChangeType.PERCENT,
            price: '',
        });
    },
    doConfirmChange () {
        const { change, price } = this.state;
        const { roadmapId } = this.props;
        var profitRate = 0;
        var profitCount = 0;
        if (change == ChangeType.PERCENT) {
            profitRate = price;
        } else if (change == ChangeType.PRICE) {
            profitCount = price;
        }
        const param = {
            userId: app.personal.info.userId,
            profitRate,
            profitCount,
            regionIdList: roadmapId,
        };
        POST(app.route.ROUTE_SET_REGION_PROFIT_WITH_LIST, param, this.setRegionProfitWithListSuccess, false);
    },
    setRegionProfitWithListSuccess (data) {
        if (data.success) {
            Toast('修改成功');
            app.pop();
            this.props.refreshList();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { change, price } = this.state;
        const { roadmapId } = this.props;
        console.log('432143124', roadmapId);
        return (
            <View style={styles.container}>
                <View style={styles.priceChange}>
                    <TouchableOpacity onPress={this.toChangePrice}
                        style={[styles.itemChange, change == 0 ? { backgroundColor:'#c91521' } : {}]}>
                        <Text style={[styles.changeText, change == 0 ? { color:'#FFFFFF' } : {}]}>按价格调整</Text>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={this.toChangePercent}
                        style={[styles.itemChange, change == 1 ? { backgroundColor:'#c91521' } : {}]}>
                        <Text style={[styles.changeText, change == 1 ? { color:'#FFFFFF' } : {}]}>按百分比调整</Text>
                    </TouchableOpacity>
                </View>
                <View style={styles.priceNumber}>
                    {
                        (change == 1 || change == 0) ?
                            <View onPress={this.toAdd} style={styles.priceAdd} >
                                <Image style={styles.chooseImage}
                                    source={app.img.order_choose} />
                                <Text style={styles.priceAddText} >上调</Text>
                            </View>
                        :
                            <View />
                    }
                    {
                        change == 0 ?
                            <View style={styles.priceInput}>
                                <TextInput
                                    onChangeText={(text) => this.setState({ price: text })}
                                    multiline={false}
                                    style={styles.bottomRight}
                                    placeholder={'请输入需要提高的价格'}
                                    autoCapitalize={'none'}
                                    underlineColorAndroid={'transparent'}
                                    defaultValue={price + ''}
                                />
                                <Text style={styles.bottomRightText}>元</Text>
                            </View>
                        :
                        change == 1 ?
                            <View style={styles.priceInput}>
                                <TextInput
                                    onChangeText={(text) => this.setState({ price: text })}
                                    multiline={false}
                                    style={styles.bottomRight}
                                    placeholder={'请输入需要提高的百分比'}
                                    autoCapitalize={'none'}
                                    underlineColorAndroid={'transparent'}
                                    defaultValue={price + ''}
                                />
                                <Text style={styles.bottomRightText}>%</Text>
                            </View>
                        :
                            <View />
                    }
                </View>
                <Button onPress={this.doConfirmChange}
                    style={styles.btnConfirmChange}
                    textStyle={styles.btnConfirmChangeText}>
                    确认
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor:'#F4F4F4',
        paddingTop:8,
    },
    aspect:{
        marginTop:5,
        backgroundColor: 'white',
        flexDirection: 'row',
        height: 42,
        width:sr.w,
        justifyContent: 'space-between',
        alignItems:'center',
    },
    aspectText:{
        color: '#888888',
        fontSize: 14,
        marginLeft: 10,
    },
    rightView: {
        flexDirection: 'row',
        height: 45,
        alignItems:'center',
    },
    itemValueText: {
        color: '#888888',
        fontSize: 14,
        marginRight: 10,
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 10,
    },
    priceChange:{
        height:45,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        alignItems:'center',
        justifyContent:'space-around',
        marginTop:2,
    },
    itemChange:{
        height:20,
        width:100,
        backgroundColor:'#f3f3f3',
    },
    changeText:{
        fontSize:12,
        color:'#333333',
        textAlign:'center',
        height:20,
        lineHeight:18,
    },
    priceNumber:{
        height:40,
        backgroundColor:'#FFFFFF',
        marginTop:2,
        flexDirection: 'row',
        justifyContent:'space-between',
    },
    bottomLeft:{
        flexDirection: 'row',
        alignItems:'center',
        height:40,
    },
    priceAdd:{
        flexDirection: 'row',
        alignItems:'center',
        marginLeft:10,
    },
    chooseImage:{
        height:9,
        width:9,
    },
    priceAddText:{
        fontSize:14,
        color:'#333333',
        marginLeft:5,
    },
    priceSub:{
        flexDirection: 'row',
        alignItems:'center',
        marginLeft:20,
    },
    priceInput:{
        flexDirection: 'row',
        alignItems:'center',
    },
    bottomRight:{
        height:40,
        width:sr.w / 2,
        fontSize:12,
        color: '#444444',
        textAlign:'right',
    },
    bottomRightText:{
        fontSize:13,
        color:'#333333',
        marginRight:15,
        marginLeft:3,
    },
    btnConfirmChange:{
        height:40,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        marginTop:250,
        borderRadius:4,
    },
    btnConfirmChangeText:{
        fontSize:16,
        fontWeight:'500',
    },
});

```

* PDClient/project/App/modules/person/ModifyShop.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    ListView,
    RefreshControl,
    TouchableOpacity,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '修改参考分店',
        rightButton: { title:'保存', handler: () => { app.scene.doSave&&app.scene.doSave(); } },
    },
    doSave () {
        const { shop } = this.state;
        const param = {
            userId: app.personal.info.userId,
            shopId: shop.id,
        };
        POST(app.route.ROUTE_SET_REFER_SHOP, param, this.setReferShopSuccess, false);
    },
    setReferShopSuccess (data) {
        const { shop } = this.state;
        if (data.success) {
            app.personal.info.agent.referShop = shop;
            Toast('修改成功');
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    componentWillMount () {
        this.getShopList();
    },
    getShopList () {
        const param = { userId: app.personal.info.userId };
        POST(app.route.ROUTE_GET_REFER_SHOP_LIST, param, this.getReferShopListSuccess, false);
    },
    getReferShopListSuccess (data) {
        if (data.success) {
            this.setState({
                list: data.context.branchShopList || {},
            });
        } else {
            Toast(data.msg);
        }
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            list:{},
            shop: app.personal.info.agent.referShop ? app.personal.info.agent.referShop : '',
        };
    },
    doChoose (obj) {
        this.setState({
            shop: obj,
        });
    },
    renderRow (obj, sectionID, rowId) {
        const { shop } = this.state;
        return (
            <TouchableOpacity style={styles.item}
                onPress={this.doChoose.bind(null, obj)}>
                <View style={styles.itemContainer}>
                    <Text style={styles.itemText}>{obj.name}</Text>
                    <Image style={styles.choose}
                        source={shop.id == obj.id ? app.img.common_radio : {}} />
                </View>
            </TouchableOpacity>
        );
    },
    onScroll () {
        this.getShopList();
    },
    render () {
        const { list, shop } = this.state;
        return (
            <View style={styles.container}>
                <ListView
                    dataSource={this.ds.cloneWithRows(list)}
                    renderRow={this.renderRow}
                    enableEmptySections
                    refreshControl={
                        <RefreshControl
                            refreshing={false}
                            onRefresh={this.onScroll}
                            title='下拉刷新...' />
                    }
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    item:{
        marginTop:1,
    },
    itemContainer:{
        height:45,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        justifyContent:'space-between',
        alignItems:'center',
    },
    itemText:{
        fontSize:14,
        color:'#333333',
        marginLeft:15,
    },
    choose:{
        height:10,
        width:10,
        marginRight:15,
    },
});

```

* PDClient/project/App/modules/person/NormalQuestions.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    WebView,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '常见问题',
    },
    render () {
        return (
            <WebView
                style={styles.container}
                source={{ uri: app.route.ROUTE_ABOUT_PAGE }}
                scalesPageToFit
                />
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
});

```

* PDClient/project/App/modules/person/PayPasswordSetting.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const DigitalKeyboard = require('../common/DigitalKeyboard');
const ConfirmPayPassword = require('./ConfirmPayPassword');

const list = [1, 2, 3, 4, 5, 6, 7, 8, 9];

module.exports = React.createClass({
    statics: {
        title: '支付密码设置',

    },
    getInitialState () {
        return {
            password: '',
        };
    },
    passwordItem (item) {
        const { password } = this.state;
        this.setState({ password: password + item });
    },
    zero () {
        const { password } = this.state;
        this.setState({ password: password + 0 });
    },
    delete () {
        const { password } = this.state;
        this.setState({
            password: _.dropRight(password).join(''),
        });
    },
    confirm () {
        const { password, type } = this.state;
        const tmp = password.substring(0, 6);
        if (tmp.length == 6) {
            this.toNext(tmp);
        } else {
            Toast('请输入6位的支付密码');
            this.toClear();
        }
    },
    toNext (tmp) {
        app.push({
            component: ConfirmPayPassword,
            passProps:{
                number: tmp,
                toClear: this.toClear,
                verifyCode : this.props.verifyCode,
            },
        });
    },
    toClear () {
        this.setState({
            password: '',
        });
    },
    render () {
        const max = [1, 2, 3, 4, 5, 6];
        const { password, type } = this.state;
        return (
            <View style={styles.container}>
                <Text style={styles.text}>该六位号码将作为手机支付密码</Text>
                <View style={styles.password}>
                    {
                        max.map((item, i) => (
                            <View style={i == 0 ? styles.itemStart : styles.item} key={i}>
                                { password.length >= (i + 1) ?
                                    <View style={styles.point} />
                                    :
                                    <View style={styles.noPoint} />
                                }
                            </View>
                        ))
                    }
                </View>
                <View style={styles.digitalKeyboard} />
                <View>
                    <DigitalKeyboard list={list}
                        passwordItem={this.passwordItem}
                        zero={this.zero}
                        delete={this.delete}
                        confirm={this.confirm} />
                </View>
            </View>
        );
    },
});
const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#f4f4f4',
    },

    text:{
        fontSize:14,
        textAlign:'center',
        color:'#333333',
        marginTop:30,
    },
    password:{
        height:40,
        width:330,
        backgroundColor:'#ffffff',
        flexDirection:'row',
        marginTop:15,
        marginLeft:(sr.w - 330) / 2,
        borderWidth:1,
        borderColor:'#EDEDED',
        borderRadius:4,
    },
    itemStart:{
        height:40,
        width:55,
        borderColor:'#EDEDED',
        justifyContent:'center',
        alignItems:'center',
    },
    item:{
        height:40,
        width:55,
        borderLeftWidth:1,
        borderColor:'#EDEDED',
        justifyContent:'center',
        alignItems:'center',
    },
    point:{
        height:10,
        width:10,
        borderRadius:5,
        backgroundColor:'#000000',
    },
    noPoint:{
        height:10,
        width:10,
        borderRadius:5,
        backgroundColor:'#FFFFFF',
    },
    digitalKeyboard:{
        flex:1,
    },
});

```

* PDClient/project/App/modules/person/Permission.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
    Text,
    ListView,
    TouchableOpacity,
} = ReactNative;

import _ from 'lodash';

module.exports = React.createClass({
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {};
    },
    doSelect (obj) {
        this.props.setSelect(obj);
    },
    renderRow (obj, sectionID, rowID) {
        const { employeeAuthority } = this.props;
        return (
            obj.length > 11 ?
            <View style={styles.longItem}>
                <TouchableOpacity onPress={this.doSelect.bind(null, obj)}
                    style={styles.press}>
                    <Image source={_.find(employeeAuthority, o => { return o == obj.o; }) != undefined ?
                        app.img.order_choose_r : app.img.order_no_choose_r}
                        style={styles.imgChoose} />
                    <Text style={styles.itemFont}>{AH.MAP[obj.o]}</Text>
                </TouchableOpacity>
            </View>
            :
            <View style={styles.item}>
                <TouchableOpacity onPress={this.doSelect.bind(null, obj)}
                    style={styles.press}>
                    <Image source={_.find(employeeAuthority, o => { return o == obj.o; }) != undefined ?
                        app.img.order_choose_r : app.img.order_no_choose_r}
                        style={styles.imgChoose} />
                    <Text style={styles.itemFont}>{AH.MAP[obj.o]}</Text>
                </TouchableOpacity>
            </View>
        );
    },
    render () {
        const { list, title } = this.props;
        return (
            <View style={styles.container}>
                {
                    list.length > 0 &&
                    <Text style={styles.head}>{title}</Text>
                }
                <ListView
                    dataSource={this.ds.cloneWithRows(list)}
                    contentContainerStyle={styles.listStyle}
                    renderRow={this.renderRow} />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        marginTop: 5,
    },
    listStyle:{
        flexDirection:'row', // 改变ListView的主轴方向
        flexWrap:'wrap', // 换行
        width:sr.w,
    },
    item:{
        width:sr.w / 2,
        height:30,
        paddingLeft:10,
    },
    longItem:{
        width:sr.w,
        paddingLeft:10,
    },
    press:{
        flexDirection:'row',
        alignItems:'center',
        height:30,
    },
    imgChoose:{
        width:14,
        height:14,
    },
    itemFont:{
        width:(sr.w / 2 - 35),
        marginLeft:10,
        fontSize:14,
        color:'#333333',
        fontWeight:'300',
    },
    head:{
        color:'#333333',
        fontSize:14,
        fontWeight:'500',
        marginLeft:9,
        marginTop:10,
        marginBottom:12,
    },
});

```

* PDClient/project/App/modules/person/PermissionSetting.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    ScrollView,
} = ReactNative;

import _ from 'lodash';

const Permission = require('./Permission');

module.exports = React.createClass({
    statics: {
        title: '权限设置',
        leftButton : { handler: () => { app.scene.gaBack&&app.scene.gaBack(); } },
        rightButton : { title: '确认', handler: () => { app.scene.confirm&&app.scene.confirm(); } },
    },
    getInitialState () {
        return {
            list:this.props.employeeInfo && this.props.employeeInfo.authority || this.props.authority || [],
        };
    },
    gaBack(){
        app.pop();
        this.props.refresh();
    },
    confirm () {
        const { employeeInfo, isCreate } = this.props;
        const { list } = this.state;
        if (isCreate) {
            app.pop();
            this.props.backAuthority(list);
        } else {
            const param = {
                userId: app.personal.info.userId,
                memberId : employeeInfo.id,
                authority: list,
            };
            if (app.personal.info.shipper) {
                POST(app.route.ROUTE_MODIFY_MEMBER_AUTHORITY, param, this.modifyMemberAuthoritySuccess, true);
            } else if (app.personal.info.agent) {
                POST(app.route.ROUTE_MODIFY_MEMBER_AUTHORITY_AGENT, param, this.modifyMemberAuthoritySuccess, true);
            }
        }
    },
    modifyMemberAuthoritySuccess (data) {
        if (data.success) {
            Toast('修改成功');
            app.pop();
            this.props.refresh();
        } else {
            Toast(data.msg);
        }
    },
    setSelect (obj) {
        const { list } = this.state;
        _.find(list, o => { return obj.o == o; }) != undefined ? _.remove(list, o => { return obj.o == o; }) : list.push(parseInt(obj.o));
        this.setState({ list });
    },
    render () {
        const { selfAuthority, employeeInfo, isCreate } = this.props;
        const { list } = this.state;
        return (
            <View style={styles.container}>
                {
                    !isCreate &&
                    <View style={styles.top}>
                        <Text style={styles.name}>姓名：{employeeInfo.name || '无'}</Text>
                        <Text style={styles.topFont}>职务：{employeeInfo.post || '普通员工'}</Text>
                        <Text style={styles.topFont}>已有权限：{list.length}项</Text>
                    </View>
                }
                <Text style={styles.title}>选择权限</Text>
                <ScrollView style={styles.jurisdiction}>
                    <View>
                        <Permission list={_.filter(_.map(selfAuthority, (o) => { if (o >= 0 && o < 10000) return { o }; }), undefined)}
                            employeeAuthority={list} setSelect={this.setSelect} title={'总店权限'} />
                    </View>
                    <View>
                        <Permission list={_.filter(_.map(selfAuthority, (o) => { if (o >= 10000 && o < 20000) return { o }; }), undefined)}
                            employeeAuthority={list} setSelect={this.setSelect} title={'公共权限'} />
                    </View>
                    <View>
                        <Permission list={_.filter(_.map(selfAuthority, (o) => { if (o >= 20000 && o < 30000) return { o }; }), undefined)}
                            employeeAuthority={list} setSelect={this.setSelect} title={'分店权限'} />
                    </View>
                    <View>
                        <Permission list={_.filter(_.map(selfAuthority, (o) => { if (o >= 30000 && o < 40000) return { o }; }), undefined)}
                            employeeAuthority={list} setSelect={this.setSelect} title={'物流公司'} />
                    </View>
                    <View>
                        <Permission list={_.filter(_.map(selfAuthority, (o) => { if (o >= 40000 && o < 50000) return { o }; }), undefined)}
                            employeeAuthority={list} setSelect={this.setSelect} title={'收货点'} />
                    </View>
                </ScrollView>

            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    top:{
        backgroundColor:'#FFFFFF',
        height:80,
    },
    name:{
        fontSize:14,
        marginLeft:10,
        marginTop:5,
        flex:1,
    },
    topFont:{
        fontSize:12,
        marginLeft:10,
        color:'#333333',
        flex:1,
    },
    title:{
        color:'#666666',
        height:35,
        lineHeight:35,
        paddingLeft:10,
        textAlignVertical:'center',
    },
    jurisdiction:{
        flex:1,
        backgroundColor:'#FFFFFF',
    },
});

```

* PDClient/project/App/modules/person/PickPrice.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    TextInput,
    Text,
    TouchableOpacity,
} = ReactNative;

const { Button } = COMPONENTS;
const PickPriceMarket = require('./PickPriceMarket');

module.exports = React.createClass({
    statics: {
        title: '自提价格',
        rightButton : { title: '行情', handler: () => { app.scene.market&&app.scene.market(); } },
    },
    market(){
        app.push({
            component:PickPriceMarket,
        });
    },
    getInitialState(){
        return{
            clientPickPrice:'',
            clientPickEnable:true,
        };
    },
    toSave(){
        const { clientPickPrice, clientPickEnable } = this.state;
        const param = {
            userId:app.personal.info.userId,
            shopId:app.personal.currentShopId,
            clientPickPrice,
            clientPickEnable,
        };
        POST(app.route.ROUTE_MODIFY_SHIPPER_INFO,param,this.toSaceSuccess,true);
    },
    toSaceSuccess(data){
        if (data.success) {
            Toast('修改成功');
            app.pop();
        }else {
            Toast(data.msg);
        }
    },
    changePickEnable(){
        this.setState({ clientPickEnable : !this.state.clientPickEnable});
    },
    render () {
        const { clientPickEnable } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.item}>
                    <Text style={styles.title}>自提价格</Text>
                    <TextInput style={styles.content}
                        underlineColorAndroid='transparent'
                        placeholder='请输入自提货物价格'
                        onChangeText = {(text)=> this.setState({ clientPickPrice : text })}/>
                    <Text style={styles.company}>元/吨</Text>
                </View>
                <TouchableOpacity style={styles.item} onPress={this.changePickEnable}>
                    <Text style={styles.title}>是否可用</Text>
                    <Text style={styles.company}>{ clientPickEnable && '是' || '否'}</Text>
                </TouchableOpacity>
                <Button style={styles.btnCommit} textStyle={styles.btnCommitText} onPress={this.toSave.bind(null)}>保存</Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    item:{
        height:45,
        backgroundColor:'#FFFFFF',
        flexDirection:'row',
        marginTop:10,
        alignItems:'center',
        paddingHorizontal:5,
        justifyContent:'space-between',
    },
    title:{
        color:'#333333',
        fontSize:14,
    },
    content:{
        flex:1,
        textAlign:'right',
        marginRight:5,
        fontSize:14,
    },
    company:{
        color:'#666666',
    },
    btnCommit:{
        marginTop:130,
        marginHorizontal:10,
        borderRadius:4,
        height:45,
    },
    btnCommitText:{
        fontWeight:'500',
    },
});

```

* PDClient/project/App/modules/person/PickPriceMarket.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const { PageList } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '自提行情',
    },
    renderRow(obj,rowID){
        {
            obj.clientPickEnable &&
            <View style={styles.item}>
                <Text style={styles.title}>{obj.name}</Text>
                <View style={styles.info}>
                    <Text style={styles.content}>{obj.clientPickPrice}</Text>
                    <Text style={styles.company}>元/吨</Text>
                </View>
            </View>
        }
    },
    render () {
        return (
            <View style={styles.container}>
                <PageList
                ref={(ref) => { this.listView = ref; }}
                renderRow={this.renderRow}
                listParam={{ userId: app.personal.info.userId, shopId:app.personal.currentShopId  }}
                listName={'shipperList'}
                listUrl={app.route.ROUTE_GET_CLIENT_PICK_SHIPPER_LIST}
                refreshEnable
                />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    item:{
        height:45,
        backgroundColor:'#FFFFFF',
        flexDirection:'row',
        marginTop:10,
        alignItems:'center',
        paddingHorizontal:5,
        justifyContent:'space-between',
    },
    info:{
        flexDirection:'row',
    },
    title:{
        color:'#333333',
        fontSize:14,
    },
    content:{
        textAlign:'right',
        marginRight:5,
        fontSize:14,
        color:'#f01725'
    },
    company:{
        color:'#666666',
    },
});

```

* PDClient/project/App/modules/person/Recharge.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
    TextInput,
} = ReactNative;

const { Button, MessageBox } = COMPONENTS;
const BankCardList = require('./SelectBankCardList');

module.exports = React.createClass({
    statics: {
        title: '充值',
    },
    getInitialState () {
        return {
            bankCard:'请选择银行卡',
            amount:'',
            type:'',
            selectedRowID:'',
        };
    },
    showBankCardList () {
        const { selectedRowID } = this.state;
        app.push({
            component:BankCardList,
            passProps: {
                setBankCard: this.setCardInfo,
                selectedRowID,
            },
        });
    },
    doRecharge () {
        const { amount } = this.state;
        if (!app.utils.checkInt2PointNum(amount)) {
            Toast('请输入数字且小数点后最多2位');
            return;
        }
        if (amount > 9999999) {
            Toast('单笔金额不能超过9999999');
            return;
        }
        app.wxpay.doPay(amount, '充值', (data) => {
            if (data.success) {
                app.personal.setRemainAmount(app.personal.remainAmount + amount * 1);
                this.props.setRemainAmount(app.personal.remainAmount);
                app.pop();
                app.showModal(
                    <MessageBox
                        onConfirm={this.doConfirmDelete}
                        content={'成功充值' + amount + '元'}
                        width={sr.s(250)} />
                );
            }
        }, (data) => {
            Toast(data.errStr);
        });
        // app.push({
        //     component:PayPassword,
        //     passProps:{
        //         param,
        //         routeName:app.route.ROUTE_RECHARGE,
        //         amount,
        //         prompt : '充值成功',
        //     },
        // });
    },
    setCardInfo (cardName, selectedRowID) {
        this.setState({
            bankCard:cardName,
            selectedRowID,
        });
    },
    render () {
        const { amount } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.row}>
                    <Text style={styles.titleText}>金额</Text>
                    <TextInput
                        underlineColorAndroid='transparent'
                        keyboardType='numeric'
                        defaultValue={amount}
                        onChangeText={(text) => {
                            this.setState({ amount: text });
                            if (text > 9999999) {
                                Toast('单笔金额不能超过9999999');
                            }
                        }
                        }
                        style={styles.ItemBgText} />
                </View>
                <TouchableOpacity
                    activeOpacity={0.6}
                    onPress={this.showBankCardList}
                    style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <Text style={styles.itemNameText}>{this.state.bankCard}</Text>
                    </View>
                    <Image
                        resizeMode='stretch'
                        source={app.img.common_go}
                        style={styles.icon_go} />
                    <View style={styles.lineView} />
                </TouchableOpacity>
                <Button onPress={this.doRecharge} style={styles.btnRecharge} textStyle={styles.btnRechargeText} >
                    充值
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        paddingVertical: 10,
    },
    ItemBg: {
        paddingLeft:10,
        height: 44,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
    },
    row:{
        flexDirection:'row',
        alignItems:'center',
        height:44,
        backgroundColor:'#ffffff',
    },
    titleText:{
        marginLeft:8,
        fontSize:15,
        color:'#888888',
    },
    ItemBgText: {
        padding:0,
        flexDirection: 'row',
        alignItems: 'center',
        textAlign:'right',
        fontSize: 15,
        flex:1,
        marginRight:10,
    },
    infoStyle: {
        flexDirection: 'row',
    },
    itemNameText: {
        fontSize: 15,
        color: '#444444',

    },
    icon_go: {
        width: 8,
        height: 15,
        marginRight: 7,
    },
    lineView: {
        position:'absolute',
        top: 0,
        width: sr.w,
        height: 1,
        backgroundColor: '#f4f4f4',
    },
    btnRecharge:{
        marginTop:215,
        height:45,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        borderRadius:2,
        backgroundColor:'#c81622',
    },
    btnRechargeText:{
        color:'#ffffff',
        fontSize:16,
        fontWeight:'300',
    },
});

```

* PDClient/project/App/modules/person/RegionProfitDetail.js

```js

```

* PDClient/project/App/modules/person/RegionProfitList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button, MessageBox, PageList } = COMPONENTS;
const ModifyProfit = require('./ModifyProfit');
const RoadmapSetting = require('./RoadmapSetting');

let searchText = '';
const Title = React.createClass({
    render () {
        return (
            <View style={styles.searchContainer}>
                <Image
                    resizeMode='cover'
                    source={app.img.common_search_button}
                    style={styles.iconSearch} />
                <TextInput
                    placeholder='输入路线终点查询路线'
                    defaultValue={searchText}
                    underlineColorAndroid='transparent'
                    onChangeText={(text) => { searchText = text; }}
                    onSubmitEditing={this.props.doStartSearch}
                    style={styles.searchTextInput}
                    />
            </View>
        );
    },
});

module.exports = React.createClass({
    statics: {
        title: (<Title doStartSearch={() => { app.scene.doStartSearch&&app.scene.doStartSearch(); }} />),
    },
    onWillFocus () {
        app.getCurrentRoute().title = (<Title doStartSearch={() => { app.scene.doStartSearch&&app.scene.doStartSearch(); }} />);
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    getInitialState () {
        const agent = app.personal.info.agent;
        return {
            userId:app.personal.info.userId,
            keyword: '',
            dataList: [],
            list:[],
            roadmapId:[],
            shopId: agent.referShop ? agent.referShop.id : '',
        };
    },
    doStartSearch () {
        this.setState({
            keyword:searchText,
        });
    },
    // showRoadmapMarket (endPoint) {
    //     app.push({
    //         component: RoadmapMarketResult,
    //         passProps:{
    //             endPoint,
    //         },
    //     });
    // },
    showRoadmapDetail (obj) {
        app.push({
            component:RoadmapSetting,
            passProps:{
                obj:obj,
                isModify: true,
                refreshList: this.refreshList,
            },
        });
    },
    refresh () {
        this.setState({ keyword: searchText }, () => {
            this.listView.refresh();
        });
    },
    doChoose (obj, rowId) {
        const { list } = this.state;
        list[rowId].selected = !list[rowId].selected;
        this.setState({
            list,
        });
    },
    doSelectedAll () {
        const { list } = this.state;
        if (_.every(list, (o) => o.selected == true)) {
            _.forEach(list, o => { o.selected = false; });
        } else {
            _.forEach(list, o => { o.selected = true; });
        }
        this.setState({
            list,
        });
    },
    toModifyProfit () {
        const { list, userId, roadmapId } = this.state;
        list.forEach((item, i) => {
            item.selected == true && roadmapId.push(item.item.id);
        });
        if (roadmapId.length == 0) {
            Toast('请选择需要调价的路线');
            return;
        }
        app.push({
            component: ModifyProfit,
            passProps: {
                userId,
                roadmapId: _.uniq(roadmapId),
                refreshList: this.refreshList,
            },
        });
    },
    toDeleteRoadmap () {
        const { list } = this.state;
        if (_.every(list, (o) => o.selected == false)) {
            Toast('请选择需要删除的路线');
        } else {
            app.showModal(
                <MessageBox
                    onConfirm={this.doConfirmDelete}
                    content={'是否确定删除所选路线'}
                    onCancel title={false}
                    width={sr.s(250)} />
            );
        }
    },
    delete (id) {
        const param = {
            userId: app.personal.info.userId,
            regionId: id,
        };
        let roadmapId = id;
        POST(app.route.ROUTE_REMOVE_REGION_PROFIT, param, this.doConfirmDeleteSuccess.bind(null, roadmapId), false);
    },
    doConfirmDelete () {
        const { list } = this.state;
        list.forEach((item, i) => {
            item.selected == true && this.delete(item.item.id);
        });
    },
    doConfirmDeleteSuccess (roadmapId, data) {
        if (data.success) {
            var list = this.listView.list;
            _.remove(list, function (o) {
                return o.id == roadmapId;
            });
            Toast('路线删除成功');
            this.setState({
                list:[],
            });
        } else {
            Toast('路线删除失败');
        }
    },
    toAddRoadmap () {
        if (!app.personal.info.agent.referShop) {
            Toast('请先设置参考分店');
            return;
        }
        app.push({
            component:RoadmapSetting,
            passProps:{
                refreshList:this.refreshList,
                isModify: false,
            },
        });
    },
    refreshList () {
        this.listView.refresh();
        this.setState({
            list:[],
        });
    },
    renderRow (obj, rowId) {
        const { list } = this.state;
        if (!_.find(list, (o) => o.index == rowId)) {
            list.push({ selected: false, index: rowId, item: obj });
        }
        return (
            <View style={styles.row}>
                <TouchableOpacity onPress={this.doChoose.bind(null, obj, rowId)}>
                    <Image
                        resizeMode='cover'
                        style={styles.imgChoose}
                        source={!list[rowId].selected ? app.img.order_no_choose : app.img.order_choose} />
                </TouchableOpacity>
                <TouchableOpacity style={styles.point} onPress={this.showRoadmapDetail.bind(null, obj)}>
                    <View style={styles.piontContainer}>
                        <Text style={styles.pointFont} numberOfLines={1}>{obj.shop ? obj.shop.name : ''}</Text>
                    </View>
                    <Image resizeMode='contain'
                        source={app.img.cargo_point}
                        style={styles.imgLine} />
                    <View style={styles.endPiontContainer}>
                        <Text style={styles.pointFont} numberOfLines={1}>{obj.regionLastCode == 0 ? '全部' : obj.region}</Text>
                    </View>
                    <View style={styles.profitContainer}>
                        <Text style={styles.pointFont} numberOfLines={1}>{obj.profitRate ? obj.profitRate : obj.profitCount }{ obj.profitRate ? '%' : '元'}</Text>
                    </View>
                </TouchableOpacity>
            </View>
        );
    },
    render () {
        const { list, keyword, shopId, userId } = this.state;
        const status = list.length != 0 && _.every(list, (o) => o.selected == true);
        return (
            <View style={styles.container}>
                <View style={styles.top}>
                    <Button style={styles.btnAddRoadmap}
                        textStyle={styles.btnReleaseFont}
                        onPress={this.toAddRoadmap}>
                        设置路线利润
                    </Button>
                </View>
                <View style={styles.listView} />
                <PageList
                    ref={(ref) => { this.listView = ref; }}
                    renderRow={this.renderRow}
                    listName={'regionRateList'}
                    renderSeparator={null}
                    listParam={{ userId, shopId, keyword }}
                    listUrl={app.route.ROUTE_GET_REGION_PROFIT_LIST}
                    refreshEnable
                    />
                <View style={styles.bottom}>
                    <TouchableOpacity style={styles.chooseAll} onPress={this.doSelectedAll}>
                        <Image
                            resizeMode='cover'
                            style={styles.imgChoose}
                            source={status ? app.img.order_choose : app.img.order_no_choose} />
                        <Text style={styles.chooseAllFont}>全选</Text>
                    </TouchableOpacity>
                    <Button onPress={this.toDeleteRoadmap}
                        style={styles.btnDelete}
                        textStyle={styles.btnDeleteFont}>删除</Button>
                    <Button onPress={this.toModifyProfit}
                        style={styles.btnModifyPrice}
                        textStyle={styles.btnModifyPriceFont}>调价</Button>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f4f4f4',
    },
    searchContainer: {
        height: 30,
        width: sr.w - 125,
        marginLeft:sr.w - 300,
        paddingVertical: 2,
        borderRadius: 4,
        backgroundColor: '#FFFFFF',
        flexDirection: 'row',
        overflow: 'hidden',
        alignItems:'center',
    },
    searchTextInput: {
        padding:0,
        height: 25,
        width: sr.w - 155,
        fontSize: 14,
        marginLeft: 5,
    },
    iconSearch: {
        height: 15,
        width: 15,
        marginLeft:5,
    },
    top:{
        height:80,
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent:'center',
    },
    btnAddRoadmap:{
        width:150,
        height:35,
        borderRadius:4,
        backgroundColor:'#fe9917',
    },
    btnReleaseFont:{
        color:'#FFFFFF',
        fontWeight:'400',
    },
    listView:{
        marginTop:8,
    },
    row:{
        height:42,
        flexDirection:'row',
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        marginBottom:1,
    },
    imgChoose:{
        marginLeft:10,
        width:13,
        height:13,
    },
    imgLine:{
        marginLeft:10,
        width:35,
    },
    piontContainer:{
        marginLeft:10,
        width:90,
        justifyContent:'center',
    },
    endPiontContainer:{
        marginLeft:10,
        width:120,
        justifyContent:'center',
    },
    profitContainer:{
        marginLeft:10,
        width:60,
        justifyContent:'center',
    },
    pointFont:{
        fontSize:14,
        alignSelf:'center',
    },
    btnRoadmapShipper:{
        width:65,
        height:20,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#f01725',
        position:'absolute',
        right:(sr.w - 350) / 2,
    },
    btnShipperFont:{
        color:'#f01725',
        fontSize:12,
    },
    bottom:{
        height:50,
        width:sr.w,
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
        alignItems:'center',
    },
    btnDelete:{
        width:60,
        height:25,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#aaaaaa',
        marginLeft:(sr.w - 200),
    },
    btnDeleteFont:{
        color:'#343434',
        fontSize:13,
        fontWeight:'400',
    },
    btnModifyPrice:{
        width:60,
        height:25,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#f21620',
        marginLeft:15,
    },
    btnModifyPriceFont:{
        color:'#f21620',
        fontSize:13,
        fontWeight:'400',
    },
    chooseAllFont:{
        fontSize:14,
        marginLeft:10,
    },
    chooseAll:{
        alignItems:'center',
        flexDirection:'row',
    },
    point:{
        flex:1,
        flexDirection:'row',
    },
});

```

* PDClient/project/App/modules/person/RoadmapSetting.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

const Roadmap = require('../common/Roadmap');

module.exports = React.createClass({
    statics: {
        title: '路线设置',
    },
    getInitialState () {
        const { obj } = this.props;
        return {
            tabIndex: obj ? obj.type : 0,
        };
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    render () {
        const { obj, isModify } = this.props;
        const { tabIndex } = this.state;
        const menus = [{ name:'所有路线', type: 0 }, { name:'长途路线', type: 1 }, { name:'短途路线', type: 2 }];
        return (
            <View style={styles.container}>
                <View style={styles.tabContainer}>
                    {
                        menus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={this.changeTab.bind(null, i)}
                                style={styles.touchTab}>
                                <View style={styles.tabButton}>
                                    <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                        {item.name}
                                    </Text>
                                    <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                </View>
                            </TouchableHighlight>
                        ))
                    }
                </View>
                {
                    menus.map((item, i) => (
                        <Roadmap
                            refreshList={this.props.refreshList}
                            key={i}
                            index={i}
                            type={item.type}
                            obj={obj || {}}
                            isModify={isModify}
                            tabIndex={tabIndex}
                            />
                    ))
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#FFFFFF',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    }, tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```

* PDClient/project/App/modules/person/SelectBankCardList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const { PageList } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '银行卡',
        leftButton: { handler: () => { app.scene.goBack&&app.scene.goBack(); } },
    },
    goBack () {
        const { bankCard, selectedRowID } = this.state;
        this.props.setBankCard(bankCard, selectedRowID);
        app.pop();
    },
    getInitialState () {
        const { selectedRowID } = this.props;
        return {
            selectedRowID,
            bankCard:'农业银行',
        };
    },
    toChoose (obj, rowID) {
        this.setState({
            selectedRowID:rowID,
        });
    },
    renderRow (obj, rowID) {
        const { selectedRowID } = this.state;
        return (
            <View style={styles.container} >
                <TouchableOpacity onPress={this.toChoose.bind(null, obj, rowID)}>
                    <View style={styles.item}>
                        <Text style={styles.nameText}>
                            {this.state.bankCard}
                        </Text>
                        {
                            (rowID == selectedRowID) &&
                            <Image
                                resizeMode='stretch'
                                source={app.img.common_radio}
                                style={styles.icon_go} />
                        }

                    </View>
                </TouchableOpacity>
            </View>
        );
    },
    render () {
        return (
            <View style={styles.container}>
                <PageList
                    ref={(ref) => { this.listView = ref; }}
                    renderRow={this.renderRow}
                    listName={'clientList'}
                    listUrl={app.route.ROUTE_GET_CLIENT_LIST}
                    refreshEnable
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#F4F4F4',
    },
    item:{
        height:42,
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
    },
    nameText:{
        paddingLeft:8,
        color:'#333333',
        fontSize:14,
    },
    icon_go: {
        width: 10,
        height: 10,
        marginRight: 12,
    },
});

```

* PDClient/project/App/modules/person/SelectEmployees.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
    Text,
    TouchableOpacity,
} = ReactNative;

const { PageList } = COMPONENTS;
const PermissionSetting = require('./PermissionSetting');
const addEmployee = require('./addEmployee');

module.exports = React.createClass({
    statics: {
        title: '选择员工',
        rightButton : { title: '添加员工', handler: () => { app.scene.addEmployee&&app.scene.addEmployee(); } },
    },
    addEmployee () {
        app.push({
            component:addEmployee,
            passProps:{
                refresh:this.refresh,
            },
        });
    },
    showPermissionSetting (obj) {
        app.push({
            component:PermissionSetting,
            passProps:{
                employeeInfo:obj,
                selfAuthority:app.personal.info.authority,
                refresh: this.refresh,
            },
        });
    },
    refresh () {
        this.listView.refresh();
    },
    renderRow (obj, rowId) {
        var info = {};
        if (app.personal.info.shipper) {
            info = app.personal.info.shipper;
        } else {
            info = app.personal.info.agent;
        }
        if (obj.id == app.personal.info.userId) {
            return null;
        } else {
            return (
                <TouchableOpacity onPress={
                        info.chairMan.id == obj.id ? null :
                        this.showPermissionSetting.bind(null, obj)}>
                    <View style={styles.item}>
                        <Text style={styles.itemFont}>{obj.name ? obj.name : obj.phone}</Text>
                        <Text style={styles.itemNum}>{obj.authority.length}项</Text>
                        <Image resizeMode='contain' source={app.img.common_go} style={styles.itemImg} />
                    </View>
                </TouchableOpacity>
            );
        }
    },
    render () {
        return (
            <View style={styles.container}>
                <PageList
                    ref={(ref) => { this.listView = ref; }}
                    renderRow={this.renderRow}
                    listParam={{ userId: app.personal.info.userId }}
                    listName={'memberList'}
                    renderSeparator={null}
                    listUrl={app.personal.info.agent ? app.route.ROUTE_GET_MEMBER_LIST_AGENT : app.route.ROUTE_GET_MEMBER_LIST}
                    refreshEnable
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    item:{
        height:42,
        marginTop:1,
        flexDirection:'row',
        backgroundColor:'#FFFFFF',
        alignItems:'center',
    },
    itemFont:{
        marginLeft:10,
        fontWeight:'300',
        flex:1,
    },
    itemNum:{
        fontWeight:'300',
        marginRight:10,
    },
    itemImg:{
        width:20,
        height:20,
    },
});

```

* PDClient/project/App/modules/person/Software.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    WebView,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '软件许可协议',
    },
    render () {
        return (
            <WebView
                style={styles.container}
                source={{ uri: app.route.ROUTE_SOFTWARE_LICENSE }}
                scalesPageToFit
                />
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
});

```

* PDClient/project/App/modules/person/VerificationCode.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    TextInput,
    Text,
    Keyboard,
    TouchableOpacity,
} = ReactNative;

const { Button } = COMPONENTS;
const PayPasswordSetting = require('./PayPasswordSetting');
const TimerMixin = require('react-timer-mixin');

module.exports = React.createClass({
    mixins: [TimerMixin],
    statics: {
        title: '获取验证码',
    },
    getInitialState () {
        return {
            readSecond:false,
            time:59,
            verifyCode:'',
            button: false,
        };
    },
    doPress () {
        const param = {
            phone: app.personal.info.phone,
        };
        POST(app.route.ROUTE_REQUEST_SEND_VERIFY_CODE, param, this.requestSendVerifyCodeSuccess, false);
    },
    timer () {
        this.setState({ readSecond:true });
        this.setTimeout(
            () => {
                this.setState({ time:(this.state.time - 1) });
                if (this.state.time <= 0) {
                    return this.setState({ readSecond:false, time:59 });
                }
                this.timer();
            },
            1000
        );
    },
    requestSendVerifyCodeSuccess (data) {
        if (data.success) {
            this.timer();
        } else {
            Toast(data.msg);
        }
    },
    doNext () {
        const { verifyCode } = this.state;
        if (!verifyCode) {
            Toast('请填写验证码');
            return;
        }
        if (!app.utils.checkVerificationCode(verifyCode)) {
            Toast('请填写正确的验证码');
            return;
        }
        app.push({
            component:PayPasswordSetting,
            passProps:{
                verifyCode,
            },
        });
        Keyboard.dismiss();
    },
    render () {
        const { readSecond } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <TextInput placeholderTextColor='#aaaaaa'
                            placeholder='请填写验证码'
                            keyboardType='numeric'
                            maxLength={6}
                            underlineColorAndroid='transparent'
                            onChangeText={(text) => this.setState({ verifyCode: text })}
                            style={styles.itemNameText} />
                        <TouchableOpacity
                            disabled={readSecond}
                            onPress={this.doPress}
                            style={styles.btnBind}>
                            { !readSecond ?
                                <View style={styles.btnContainer}>
                                    <Text style={readSecond ? styles.btnBindTextGray : styles.btnBindText}>获取验证码</Text>
                                </View>
                                :
                                <View style={styles.btnContainerGray}>
                                    <Text style={readSecond ? styles.btnBindTextGray : styles.btnBindText}>{this.state.time}s后重新发送</Text>
                                </View>
                            }
                        </TouchableOpacity>
                    </View>
                </View>
                <Button onPress={this.doNext}
                    style={styles.btnNext}
                    textStyle={styles.btnNextText} >
                    下一步
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    ItemBgTop: {
        height: 45,
        marginTop:10,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
    },
    ItemBg: {
        height: 45,
        marginTop:1,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
    },
    infoStyle: {
        width:sr.w,
        flexDirection: 'row',
        alignItems:'center',
        justifyContent: 'space-between',
    },
    icon_add:{
        width:25,
        height:25,
        marginLeft: 8,
    },
    itemNameText: {
        flex:1,
        fontSize: 14,
        color: '#444444',
        marginLeft: 10,
    },

    btnNext:{
        marginTop:50,
        height:45,
        width:300,
        marginLeft:(sr.w - 300) / 2,
        borderRadius:4,
        backgroundColor:'#c81622',
    },
    btnNextText:{
        color:'#ffffff',
        fontSize:16,
        fontWeight:'500',
    },
    btnBind:{
        height:45,
        width:137,
        borderLeftWidth:1,
        borderColor:'#f4f4f4',
        alignItems:'center',
    },
    btnContainer:{
        height:45,
        alignItems:'center',
        justifyContent:'center',
        width:137,
    },
    btnBindText:{
        color:'#FF981A',
        fontSize:14,
        fontWeight:'500',
        textAlign:'center',
        width:sr.w - 254,
    },
    btnContainerGray:{
        height:45,
        width:137,
        alignItems:'center',
        justifyContent:'center',
        backgroundColor:'#fafafa',
    },
    btnBindTextGray:{
        color:'#999999',
        backgroundColor:'#fafafa',
        fontSize:14,
        fontWeight:'500',
        textAlign:'center',
        width:sr.w - 254,
    },
});

```

* PDClient/project/App/modules/person/WithdrawCash.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
    TextInput,
} = ReactNative;

const { Button } = COMPONENTS;
const BankCardList = require('./SelectBankCardList');
const PayPassword = require('../common/PayPassword');

module.exports = React.createClass({
    statics: {
        title: '提现',
    },
    showBankCardList () {
        const { selectedRowID } = this.state;
        app.push({
            component:BankCardList,
            passProps: {
                setBankCard: this.setCardInfo,
                selectedRowID,
            },
        });
    },
    getInitialState () {
        return {
            bankName: '请选择银行卡',
            money:'',
            selectedRowID:'',
        };
    },
    doWithdraw () {
        const { money } = this.state;
        if (!app.utils.checkInt2PointNum(money)) {
            Toast('请输入数字且小数点后最多2位');
            return;
        }
        if (money > 9999999) {
            Toast('单笔金额不能超过9999999');
            return;
        }
        if (money == 0) {
            Toast('提现金额必须大于0元');
            return;
        }
        const param = {
            userId: app.personal.info.userId,
            amount:money,
            thirdpartyAccount:'N1231242311',
        };
        app.push({
            component:PayPassword,
            passProps:{
                param,
                routeName:app.route.ROUTE_WITHDRAW,
                setRemainAmount : this.props.setRemainAmount,
                money,
                prompt : '提现成功',
            },
        });
    },
    setCardInfo (cardName, selectedRowID) {
        this.setState({
            bankName:cardName,
            selectedRowID,
        });
    },
    WithdrawalsAllMoney () {
        this.setState({
            money:app.personal.remainAmount + '',
        });
    },
    render () {
        const { money } = this.state;
        return (
            <View style={styles.container}>
                <Text style={styles.drawCashTitle}>
                    提现金额
                </Text>
                <View style={styles.drawCash}>
                    <Text style={styles.drawCashLeft}>
                        ￥
                    </Text>
                    <TextInput
                        keyboardType='numeric'
                        underlineColorAndroid='transparent'
                        placeholderTextColor='#888888'
                        placeholder='请输入提现金额'
                        defaultValue={money}
                        onChangeText={(text) => {
                            this.setState({ money: text });
                            if (text > 9999999) {
                                Toast('单笔金额不能超过9999999');
                            }
                        }
                        }
                        style={styles.drawCashContent} />
                </View>
                <View style={styles.balance}>
                    <View style={styles.balanceLeft}>
                        <Text style={styles.balanceTextLeft}>
                            账户余额：
                        </Text>
                        <Text style={styles.balanceTextRight}>
                            {app.personal.remainAmount}元
                        </Text>
                    </View>
                    <TouchableOpacity onPress={this.WithdrawalsAllMoney}>
                        <Text style={styles.allDrawcash}>
                            全部提现
                        </Text>
                    </TouchableOpacity>
                </View>
                <TouchableOpacity
                    activeOpacity={0.6}
                    onPress={this.showBankCardList}
                    style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <Text style={styles.itemNameText}>{this.state.bankName}</Text>
                    </View>
                    <Image
                        resizeMode='stretch'
                        source={app.img.common_go}
                        style={styles.icon_go} />
                    <View style={styles.lineView} />
                </TouchableOpacity>
                <Button onPress={this.doWithdraw} style={styles.btnDrawCash} textStyle={styles.btnDrawCashText}>
                    确认提现
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#F4F4F4',
    },
    drawCashTitle:{
        marginTop:10,
        paddingLeft:12,
        paddingTop:10,
        fontSize:12,
        backgroundColor:'#FFFFFF',
    },
    drawCash:{
        height:48,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        alignItems:'center',
    },
    drawCashLeft:{
        fontSize:12,
        paddingLeft:12,
    },
    drawCashContent:{
        flex:1,
        fontSize:20,
    },
    balance:{
        height:34,
        marginTop:1,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        justifyContent:'space-between',
    },
    balanceLeft:{
        height:34,
        marginTop:1,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
    },
    balanceTextLeft:{
        fontSize:14,
        color:'#888888',
        paddingLeft:12,
        paddingTop:10,
    },
    balanceTextRight:{
        fontSize:14,
        color:'#888888',
        paddingTop:10,
    },
    allDrawcash:{
        color:'#fd9a14',
        paddingRight:10,
        paddingTop:10,
    },
    ItemBg: {
        marginTop:17,
        padding: 10,
        height: 44,
        width:sr.w,
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
        backgroundColor:'white',
    },
    infoStyle: {
        flexDirection: 'row',
    },
    itemNameText: {
        fontSize: 15,
        width:sr.w - 45,
        color: '#444444',
        marginLeft: 8,
    },
    icon_go: {
        width: 8,
        height: 15,
        marginRight: 7,
    },
    btnDrawCash:{
        marginTop:285,
        backgroundColor:'#FE9917',
        height:46,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        borderRadius:2,
    },
});

```

* PDClient/project/App/modules/person/addEmployee.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
    Text,
    TextInput,
    Keyboard,
    TouchableOpacity,
} = ReactNative;

const { Button } = COMPONENTS;
const PermissionSetting = require('./PermissionSetting');

module.exports = React.createClass({
    statics: {
        title: '添加员工',
        leftButton:{ handler: () => { app.scene.goBack&&app.scene.goBack(); } },
    },
    componentWillMount () {
        this.keyboardDidHideListener = Keyboard.addListener('keyboardDidHide', this._keyboardDidHide);
    },
    componentWillUnmount () {
        this.keyboardDidHideListener.remove();
    },
    _keyboardDidHide () {
        this.getMemberByPhone();
    },
    goBack () {
        this.props.refresh();
        app.pop();
    },
    getInitialState () {
        return {
            authority:[],
            phone:'',
            name:'',
            tip:'',
            memberId:'',
        };
    },
    selectPermission () {
        const { authority } = this.state;
        Keyboard.dismiss();
        app.push({
            component:PermissionSetting,
            passProps:{
                isCreate:true,
                authority,
                selfAuthority:app.personal.info.authority,
                backAuthority:(authority) => { this.setState({ authority }); },
            },
        });
    },
    getMemberByPhone () {
        const { phone } = this.state;
        const param = {
            userId: app.personal.info.userId,
            memberPhone:phone,
        };
        if (app.personal.info.agent) {
            POST(app.route.ROUTE_GET_MEMBER_BY_PHONE_AGENT, param, this.getMemberByPhoneSuccess, false);
        } else {
            POST(app.route.ROUTE_GET_MEMBER_BY_PHONE, param, this.getMemberByPhoneSuccess, false);
        }
    },
    getMemberByPhoneSuccess (data) {
        if (data.success) {
            this.setState({ name: data.context.name, memberId: data.context.id, tip: null });
        } else {
            this.setState({ tip:data.msg });
        }
    },
    modifyMemberAuthority () {
        const { memberId, authority } = this.state;
        if (!memberId) {
            Keyboard.dismiss();
            Toast('没有该用户,请请重新输入');
            return;
        }
        if (authority.length == 0) {
            Keyboard.dismiss();
            Toast('请给该用户配置最少一个权限');
            return;
        }
        const param = {
            userId: app.personal.info.userId,
            memberId,
            authority,
        };
        if (app.personal.info.shipper) {
            POST(app.route.ROUTE_MODIFY_MEMBER_AUTHORITY, param, this.modifyMemberAuthoritySuccess, true);
        } else if (app.personal.info.agent) {
            POST(app.route.ROUTE_MODIFY_MEMBER_AUTHORITY_AGENT, param, this.modifyMemberAuthoritySuccess, true);
        }
    },
    modifyMemberAuthoritySuccess (data) {
        if (data.success) {
            Toast('添加成功');
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { tip, name, authority } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.top}>
                    <View style={styles.row}>
                        <Text style={styles.rowFont}>手机号码</Text>
                        <TextInput style={styles.rowRightFont}
                            underlineColorAndroid='transparent'
                            placeholderColor='#aaaaaa'
                            maxLength={11}
                            onChangeText={(text) => this.setState({ phone:text })}
                            onSubmitEditing={Keyboard.dismiss}
                            onBlur={this.getMemberByPhone}
                            placeholder='请输入手机号码' />
                    </View>
                    {
                        tip != '' ? tip != null ?
                            <Text style={styles.rowTip}>{tip}</Text>
                        :
                            <View style={styles.row}>
                                <Text style={styles.rowFont}>姓名</Text>
                                <Text style={styles.rowRightFont}>{name || '无'}</Text>
                            </View>
                        : <View />
                }

                </View>
                <TouchableOpacity style={styles.row} onPress={this.selectPermission}>
                    <Text style={styles.rowFont}>权限设置</Text>
                    <Text style={styles.rowRightFont}>{authority.length}项</Text>
                    <Image resizeMode='contain' style={styles.imgGo} source={app.img.common_go} />
                </TouchableOpacity>
                <Button onPress={this.modifyMemberAuthority} style={styles.btnConfirm} textStyle={styles.btnFont}>确认添加</Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    top:{
        marginTop:10,
    },
    row:{
        backgroundColor:'#FFFFFF',
        height:42,
        flexDirection:'row',
        alignItems:'center',
    },
    rowFont:{
        marginLeft:10,
        fontWeight:'300',
        flex:1,
    },
    rowTip:{
        height:35,
        fontWeight:'300',
        color:'#ee1824',
        textAlignVertical:'center',
        paddingLeft:10,
        lineHeight:35,
    },
    rowRightFont:{
        flex:1,
        fontSize:14,
        textAlign:'right',
        marginRight:10,
    },
    imgGo:{
        width:15,
        height:15,
        marginRight:10,
    },
    btnConfirm:{
        width:150,
        height:45,
        alignSelf:'center',
        borderRadius:4,
        marginTop:65,
    },
    btnFont:{
        fontSize:14,
        fontWeight:'500',
    },
});

```

* PDClient/project/App/modules/person/index.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    Image,
    StyleSheet,
    Text,
    View,
    TouchableOpacity,
    ScrollView,
} = ReactNative;

import _ from 'lodash';

const CashBalance = require('./CashBalance');
const Bankcards = require('./BindBankcardList');

const EditPersonInfo = require('./EditPersonInfo');
const About = require('./About');
const Feedback = require('./Feedback');
const NormalQuestions = require('./NormalQuestions');
const UpdatePage = require('../update/UpdatePage');
const Software = require('./Software');
const ModifyPassword = require('./ModifyPassword');
const Order = require('../normalUser/order');
const GuaranteeAmount = require('./GuaranteeAmount');
const AddCar = require('../shortLogisticsCompany/cargo/AddCar');
const BranchStore = require('./BranchStore');
const PickPrice = require('./PickPrice');
const SelectEmployees = require('./SelectEmployees');
const VerificationCode = require('./VerificationCode');
const RegionProfitList = require('./RegionProfitList');
const ModifyShop = require('./ModifyShop');

const { Button, DImage } = COMPONENTS;

const CHILD_PAGES_TOP = [
    { strict:true, title:'余额', module: CashBalance, img:'' },
    { strict:true, title:'银行卡', module: Bankcards, img:'' },
    { strict:true, title:'支付密码设置', module: VerificationCode, img:'' },
    { strict:true, title:'修改登录密码', module: ModifyPassword, img:'' },
];
const CHILD_PAGES_MIDDLE = [
    { strict:true, title:'负责人', module: null, img:'' },
    { strict:true, title:'员工设置', module: SelectEmployees, img:'' },
    { strict:true, title:'路线设置', module: RegionProfitList, img:'' },
    { strict:true, title:'修改参考分店', module: ModifyShop, img:'' },

];

const MenuItem = React.createClass({
    showChildPage () {
        if (this.props.page.module) {
            app.push({
                component: this.props.page.module,
                passProps:{
                    type: this.props.type ? this.props.type : '',
                },
            });
        } else {
            return;
        }
    },
    render () {
        const { title } = this.props.page;
        return (
            <TouchableOpacity
                activeOpacity={0.6}
                onPress={this.showChildPage}
                style={styles.ItemBg}>
                <View style={styles.infoStyle}>
                    <Text style={styles.itemNameText}>{title}</Text>
                </View>
                {
                    this.props.page.module &&
                    <Image
                        resizeMode='stretch'
                        source={app.img.common_go}
                        style={styles.icon_go} />
                }
                <View style={styles.lineView} />
            </TouchableOpacity>
        );
    },
});

module.exports = React.createClass({
    statics: {
        title: '个人中心',
    },
    onWillFocus () {
        this.setState({
            CHILD_PAGES_BOTTOM : [
                { strict:true, title:'添加货车', module: AddCar, img:'', currentRole:'物流公司' },
                { strict:true, title:'自提价格', module: PickPrice, img:'', currentRole:'物流公司' },
                { strict:true, title:'担保金额', module: GuaranteeAmount, img:'', currentRole:'物流公司' },
                { strict:true, title:'入驻分店', module: BranchStore, img:'', currentRole:'物流公司' },
                { strict:true, title:'权限设置', module: SelectEmployees, img:'', currentRole:'物流公司' },
                { strict:true, title:'软件许可协议', module: Software, img:'', currentRole:'发货人' },
                { strict:true, title:'常见问题', module: NormalQuestions, img:'', currentRole: app.authority.info.currentRole },
                { strict:true, title:'在线更新', module: UpdatePage, img:'', currentRole: app.authority.info.currentRole },
                { strict:true, title:'关于软件', module: About, img:'', currentRole: app.authority.info.currentRole },
                { strict:true, title:'意见反馈', module: Feedback, img:'', currentRole: app.authority.info.currentRole },
            ],
        });
        app.getCurrentRoute().title = '个人中心';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    doLogout () {
        app.navigator.resetTo({
            component: require('../login/Login'),
        }, 0);
        app.personal.setNeedLogin(true);
    },
    getInitialState () {
        const info = app.personal.info;
        return {
            info,
            CHILD_PAGES_BOTTOM : [],
        };
    },
    toExitDatum () {
        app.push({
            component: EditPersonInfo,
            passProps: { updateInfo: this.updateInfo },
        });
    },
    updateInfo (info) {
        this.setState({ info });
    },
    doChangeRole (name) {
        app.authority.changeRole(name);
        if (name == '物流公司') {
            app.navigator.resetTo({
                component: require('../logisticsCompany'),
            }, 0);
        } else if (name == '发货人') {
            app.navigator.resetTo({
                component: require('../normalUser'),
            }, 0);
        } else if (name == '收货点') {
            app.navigator.resetTo({
                component: require('../receivePoint'),
            }, 0);
        }
    },
    render () {
        const { info, CHILD_PAGES_BOTTOM } = this.state;
        const canSelectList = app.authority.getCanSelectRoles();
        return (
            <ScrollView style={styles.container}>
                <View style={styles.headImgBg}>
                    <TouchableOpacity
                        onPress={this.toExitDatum}>
                        <DImage
                            resizeMode='cover'
                            defaultSource={app.img.personal_default_head}
                            source={{ uri: info.head }}
                            style={styles.headStyle} />
                        <Image
                            resizeMode='stretch'
                            source={app.personal.info.sex == 1 ? app.img.personal_male : app.img.personal_female}
                            style={styles.careStyle} />
                    </TouchableOpacity>
                    <Text style={styles.name}>{info.name ? info.name : info.phone}</Text>
                    <View style={styles.btnRow}>
                        {
                            // canSelectList.map((item, i) => {
                            //     return (
                            //         <Button onPress={this.doChangeRole.bind(null, item)} style={styles.btnChangeRole} key={i}
                            //             textStyle={styles.btnChangeRoleText}>
                            //             切换到{item}
                            //         </Button>
                            //     );
                            // })
                        }
                    </View>
                </View>
                <View>
                    <View>
                        <View style={styles.lineViewTop} />
                        {
                             (app.authority.info.currentRole == '发货人' || app.authority.info.currentRole == '物流公司') &&
                             <MenuItem page={{ strict:true, title:'我的货单', module: Order }} />
                        }

                        <View style={styles.lineViewTop} />
                    </View>
                    {
                        CHILD_PAGES_TOP.map((item, i) => {
                            if (!app.personal.info.phone && item.strict) {
                                return null;
                            }
                            if (item.title == '余额') {
                                if ((_.intersection(app.personal.info.authority, [AH.AH_LOOK_AMOUNT, AH.AH_RECHARGE, AH.AH_WITHDRAW]).length != 0) || app.authority.info.currentRole == '发货人') {
                                    return (<MenuItem page={item} key={i} />);
                                } else {
                                    return null;
                                }
                            } else if (item.title == '支付密码设置') {
                                if ((_.intersection(app.personal.info.authority, [AH.AH_WITHDRAW]).length != 0) || app.authority.info.currentRole == '发货人') {
                                    return (<MenuItem page={item} key={i} />);
                                } else {
                                    return null;
                                }
                            } else {
                                return (<MenuItem page={item} key={i} />);
                            }
                        })
                    }

                    {
                        (app.personal.info.agent && app.authority.info.currentRole != '发货人' && app.authority.info.currentRole != '物流公司') ?
                        <View>
                            <View style={styles.lineViewTop} />
                            {
                                CHILD_PAGES_MIDDLE.map((item, i) => {
                                    if (item.title == '员工设置') {
                                        return (<MenuItem page={item} key={i} type='agent' />);
                                    } else {
                                        return (<MenuItem page={item} key={i} />);
                                    }
                                })
                            }
                        </View>
                        : null
                    }
                    <View style={styles.lineViewTop} />
                    {
                        _.filter(CHILD_PAGES_BOTTOM, o => app.authority.info.currentRole == o.currentRole).map((item, i) => {
                            if (!app.personal.info.phone && item.strict) {
                                return null;
                            }
                            if (item.title == '担保金额') {
                                if (_.intersection(app.personal.info.authority, [AH.AH_LOOK_AMOUNT, AH.AH_RECHARGE, AH.AH_WITHDRAW]).length != 0) {
                                    return (<MenuItem page={item} key={i} />);
                                } else {
                                    return null;
                                }
                            } else if (item.title == '权限设置') {
                                if (_.intersection(app.personal.info.authority, [AH.AH_MODIFY_SHIPPER_MEMBER_AUTHORITY]).length != 0) {
                                    return (<MenuItem page={item} key={i} />);
                                } else {
                                    return null;
                                }
                            } else if (item.title == '添加货车') {
                                if (_.intersection(app.personal.info.authority, [AH.AH_CITY_DISTRIBUTE]).length != 0) {
                                    return (<MenuItem page={item} key={i} />);
                                } else {
                                    return null;
                                }
                            } else if (item.title == '自提价格') {
                                if (app.personal.info.shipper.shipperType == 1) {
                                    return (<MenuItem page={item} key={i} />);
                                } else {
                                    return null;
                                }
                            } else {
                                return (<MenuItem page={item} key={i} />);
                            }
                        })
                    }
                    <View style={styles.line} />

                </View>
                <Button onPress={this.doLogout} style={styles.btnLogout} textStyle={styles.btnLogoutText} >
                    退出登录
                </Button>
            </ScrollView>

        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex:1,
        backgroundColor:'white',
    },
    headImgBg:{
        height: 170,
        width: sr.w,
        backgroundColor:'#c81622',
        alignItems:'center',
    },
    ItemBg: {
        padding: 10,
        height: 44,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
    },
    btnRow:{
        marginTop:3,
        flexDirection:'row',
    },
    headStyle: {
        width: 87,
        height: 87,
        marginTop: 18,
        borderRadius: 43.5,
    },
    icon_go: {
        width: 8,
        height: 15,
        marginRight: 7,
    },
    lineView: {
        position:'absolute',
        top: 0,
        width: sr.w,
        height: 1,
        backgroundColor: '#f4f4f4',
    },
    lineViewTop:{
        backgroundColor: '#f4f4f4',
        height:8,
    },
    infoStyle: {
        flexDirection: 'row',
    },
    careStyle: {
        width: 30,
        height: 30,
        marginLeft:60,
        marginTop:-25,
    },
    itemNameText: {
        fontSize: 15,
        color: '#444444',
        marginLeft: 8,
    },
    name: {
        marginTop: 10,
        fontSize: 19,
        color: 'white',
        backgroundColor: 'transparent',
    },
    midView: {
        width: sr.w,
        height: 105,
        alignItems: 'center',
    },
    careShop: {
        marginVertical: 10,
        width:sr.w / 2.0,
        height: 50,
        alignItems: 'center',
        borderRadius: 8,
        justifyContent: 'center',
    },
    line: {
        width:sr.w,
        height:1,
        backgroundColor: '#f4f4f4',
    },
    btnChangeRole:{
        height:20,
        width:90,
        backgroundColor:'#fb9816',
    },
    btnChangeRoleText:{
        fontSize:12,
    },
    btnLogout:{
        marginTop:30,
        height:45,
        width:sr.w - 25,
        borderRadius:2,
        backgroundColor:'#e0e0e0',
        alignSelf:'center',
    },
    btnLogoutText:{
        color:'#000000',
        fontSize:16,
        fontWeight:'300',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/index.js

```js
'use strict';
const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
} = ReactNative;

import TabNavigator from 'react-native-tab-navigator';

const Cargo = require('./cargo');
const ShortRoadmap = require('./shortroadmap');
const Roadmap = require('./shortroadmap');
const Order = require('./order/Orders');
const Person = require('../person');

const INIT_ROUTE_INDEX = 0;
const ROUTE_STACK = [
    { index: 0, component: Cargo },
    { index: 1, component: Order },
    { index: 2, component: Roadmap },
    { index: 3, component: Person },
];

const HomeTabBar = React.createClass({
    componentWillMount () {
        app.showMainScene = (i) => {
            const { title, leftButton, rightButton } = _.find(ROUTE_STACK, (o) => o.index === i).component;
            Object.assign(app.getCurrentRoute().component, {
                title: title,
                leftButton: leftButton,
                rightButton: rightButton,
            });
            this.props.onTabIndex(i);
            app.forceUpdateNavbar();
        };
    },
    componentDidMount () {
        app.hasLoadMainPage = true;
        app.toggleNavigationBar(true);
    },
    componentWillUnmount () {
        app.hasLoadMainPage = false;
    },
    getInitialState () {
        return {
            tabIndex: this.props.initTabIndex,
        };
    },
    handleWillFocus (route) {
        const tabIndex = route.index;
        this.setState({ tabIndex });
    },
    render () {
        const menus = [
            { index: 0, title: '竞得货物', icon: app.img.home_cargo, selected: app.img.home_cargo_press },
            { index: 1, title: '我的货单', icon: app.img.home_truck_unselect, selected: app.img.home_truck },
            { index: 2, title: '路线设置', icon: app.img.home_roadmap, selected: app.img.home_roadmap_press },
            { index: 3, title: '我的', icon: app.img.home_mine, selected: app.img.home_mine_press },
        ];
        const TabNavigatorItems = menus.map((item) => {
            return (
                <TabNavigator.Item
                    key={item.index}
                    selected={this.state.tabIndex === item.index}
                    title={item.title}
                    titleStyle={styles.titleStyle}
                    selectedTitleStyle={styles.titleSelectedStyle}
                    renderIcon={() =>
                        <Image
                            resizeMode='stretch'
                            source={item.icon}
                            style={styles.icon} />
                    }
                    renderSelectedIcon={() =>
                        <Image
                            resizeMode='stretch'
                            source={item.selected}
                            style={styles.icon} />
                    }
                    onPress={() => {
                        app.showMainScene(item.index);
                    }}>
                    <View />
                </TabNavigator.Item>
            );
        });
        return (
            <View style={styles.tabs}>
                <TabNavigator
                    tabBarStyle={styles.tabBarStyle}
                    tabBarShadowStyle={styles.tabBarShadowStyle}
                    hidesTabTouch >
                    {TabNavigatorItems}
                </TabNavigator>
            </View>
        );
    },
});

module.exports = React.createClass({
    statics: {
        title: ROUTE_STACK[INIT_ROUTE_INDEX].component.title,
        leftButton: ROUTE_STACK[INIT_ROUTE_INDEX].component.leftButton,
        rightButton: ROUTE_STACK[INIT_ROUTE_INDEX].component.rightButton,
    },
    getChildScene () {
        return this.scene;
    },
    renderScene (route, navigator) {
        return <route.component ref={(ref) => { if (ref)route.ref = ref; }} />;
    },
    render () {
        return (
            <Navigator
                debugOverlay={false}
                style={styles.container}
                ref={(navigator) => {
                    this._navigator = navigator;
                }}
                initialRoute={ROUTE_STACK[INIT_ROUTE_INDEX]}
                initialRouteStack={ROUTE_STACK}
                renderScene={this.renderScene}
                onDidFocus={(route) => {
                    const ref = this.scene = app.scene = route.ref;
                    ref && ref.onDidFocus && ref.onDidFocus();
                }}
                onWillFocus={(route) => {
                    if (this._navigator) {
                        const { routeStack, presentedIndex } = this._navigator.state;
                        const preRoute = routeStack[presentedIndex];
                        if (preRoute) {
                            const preRef = preRoute.ref;
                            preRef && preRef.onWillHide && preRef.onWillHide();
                        }
                    }
                    const ref = route.ref;
                    ref && ref.onWillFocus && ref.onWillFocus(true); // 注意：因为有initialRouteStack，在mounted的时候所有的页面都会加载，因此只有第一个页面首次不会调用，需要在componentDidMount中调用，其他页面可以调用
                }}
                configureScene={(route) => ({
                    ...app.configureScene(route),
                })}
                navigationBar={
                    <HomeTabBar
                        initTabIndex={INIT_ROUTE_INDEX}
                        onTabIndex={(index) => {
                            this._navigator.jumpTo(_.find(ROUTE_STACK, (o) => o.index === index));
                        }}
                        />
                }
                />
        );
    },
});

const styles = StyleSheet.create({
    container: {
        overflow: 'hidden',
        flex: 1,
    },
    tabs: {
        height: 50,
        width: sr.w,
        left: 0,
        bottom:0,
    },
    titleStyle: {
        fontSize:10,
        color: '#929292',
    },
    titleSelectedStyle: {
        fontSize:10,
        color: '#DF3932',
    },
    tabBarStyle: {
        borderColor: '#EEEEEE',
        borderTopWidth: 1,
        height:50,
        backgroundColor: '#FEFCFD',
        alignItems: 'center',
    },
    tabBarShadowStyle: {
        height: 1,
        backgroundColor: '#EEEEEE',
    },
    icon: {
        width:22,
        height:22,
    },
});

```

* PDClient/project/App/modules/receivePoint/index.js

```js
'use strict';
const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
} = ReactNative;

import TabNavigator from 'react-native-tab-navigator';

const placeOrder = require('./placeOrder');
const order = require('./order');
const Person = require('../person');
const PreOrder = require('./preOrder');

const INIT_ROUTE_INDEX = 0;
const ROUTE_STACK = [
    { index: 0, component: placeOrder },
    { index: 1, component: PreOrder },
    { index: 2, component: order },
    { index: 3, component: Person },
];

const HomeTabBar = React.createClass({
    componentWillMount () {
        app.showMainScene = (i) => {
            const { title, leftButton, rightButton } = _.find(ROUTE_STACK, (o) => o.index === i).component;
            Object.assign(app.getCurrentRoute().component, {
                title: title,
                leftButton: leftButton,
                rightButton: rightButton,
            });
            this.props.onTabIndex(i);
            app.forceUpdateNavbar();
        };
    },
    componentDidMount () {
        app.hasLoadMainPage = true;
        app.toggleNavigationBar(true);
    },
    componentWillUnmount () {
        app.hasLoadMainPage = false;
    },
    getInitialState () {
        return {
            tabIndex: this.props.initTabIndex,
        };
    },
    handleWillFocus (route) {
        const tabIndex = route.index;
        this.setState({ tabIndex });
    },
    render () {
        const menus = [
            { index: 0, title: '填单', icon:app.img.home_pre_order, selected: app.img.home_pre_order_press },
            { index: 1, title: '发货', icon:app.img.home_receivePoint, selected: app.img.home_receivePoint_press },
            { index: 2, title: '货单', icon: app.img.home_order, selected: app.img.home_order_press },
            { index: 3, title: '我的', icon: app.img.home_mine, selected: app.img.home_mine_press },
        ];
        const TabNavigatorItems = menus.map((item) => {
            return (
                <TabNavigator.Item
                    key={item.index}
                    selected={this.state.tabIndex === item.index}
                    title={item.title}
                    titleStyle={styles.titleStyle}
                    selectedTitleStyle={styles.titleSelectedStyle}
                    renderIcon={() =>
                        <Image
                            resizeMode='stretch'
                            source={item.icon}
                            style={styles.icon} />
                    }
                    renderSelectedIcon={() =>
                        <Image
                            resizeMode='stretch'
                            source={item.selected}
                            style={styles.icon} />
                    }
                    onPress={() => {
                        app.showMainScene(item.index);
                    }}>
                    <View />
                </TabNavigator.Item>
            );
        });
        return (
            <View style={styles.tabs}>
                <TabNavigator
                    tabBarStyle={styles.tabBarStyle}
                    tabBarShadowStyle={styles.tabBarShadowStyle}
                    hidesTabTouch >
                    {TabNavigatorItems}
                </TabNavigator>
            </View>
        );
    },
});

module.exports = React.createClass({
    statics: {
        title: ROUTE_STACK[INIT_ROUTE_INDEX].component.title,
        leftButton: ROUTE_STACK[INIT_ROUTE_INDEX].component.leftButton,
        rightButton: ROUTE_STACK[INIT_ROUTE_INDEX].component.rightButton,
    },
    getChildScene () {
        return this.scene;
    },
    renderScene (route, navigator) {
        return <route.component ref={(ref) => { if (ref)route.ref = ref; }} />;
    },
    render () {
        return (
            <Navigator
                debugOverlay={false}
                style={styles.container}
                ref={(navigator) => {
                    this._navigator = navigator;
                }}
                initialRoute={ROUTE_STACK[INIT_ROUTE_INDEX]}
                initialRouteStack={ROUTE_STACK}
                renderScene={this.renderScene}
                onDidFocus={(route) => {
                    const ref = this.scene = app.scene = route.ref;
                    ref && ref.onDidFocus && ref.onDidFocus();
                }}
                onWillFocus={(route) => {
                    if (this._navigator) {
                        const { routeStack, presentedIndex } = this._navigator.state;
                        const preRoute = routeStack[presentedIndex];
                        if (preRoute) {
                            const preRef = preRoute.ref;
                            preRef && preRef.onWillHide && preRef.onWillHide();
                        }
                    }
                    const ref = route.ref;
                    ref && ref.onWillFocus && ref.onWillFocus(true); // 注意：因为有initialRouteStack，在mounted的时候所有的页面都会加载，因此只有第一个页面首次不会调用，需要在componentDidMount中调用，其他页面可以调用
                }}
                configureScene={(route) => ({
                    ...app.configureScene(route),
                })}
                navigationBar={
                    <HomeTabBar
                        initTabIndex={INIT_ROUTE_INDEX}
                        onTabIndex={(index) => {
                            this._navigator.jumpTo(_.find(ROUTE_STACK, (o) => o.index === index));
                        }}
                        />
                }
                />
        );
    },
});

const styles = StyleSheet.create({
    container: {
        overflow: 'hidden',
        flex: 1,
    },
    tabs: {
        height: 50,
        width: sr.w,
        left: 0,
        bottom:0,
    },
    titleStyle: {
        fontSize:10,
        color: '#929292',
    },
    titleSelectedStyle: {
        fontSize:10,
        color: '#DF3932',
    },
    tabBarStyle: {
        borderColor: '#EEEEEE',
        borderTopWidth: 1,
        height:50,
        backgroundColor: '#FEFCFD',
        alignItems: 'center',
    },
    tabBarShadowStyle: {
        height: 1,
        backgroundColor: '#EEEEEE',
    },
    icon: {
        width:22,
        height:22,
    },
});

```

* PDClient/project/App/modules/splash/index.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
    TouchableOpacity,
} = ReactNative;

import Swiper from 'react-native-swiper';
const TimerMixin = require('react-timer-mixin');
const SplashScreen = require('@remobile/react-native-splashscreen');
const Login = require('../login/Login');
const NormalUser = require('../normalUser');
const LogisticsCompany = require('../logisticsCompany');
const ShortLogisticsCompany = require('../shortLogisticsCompany');
const ReceivePoint = require('../receivePoint');

module.exports = React.createClass({
    mixins: [TimerMixin],
    getInitialState () {
        return {
            renderSplashType: 0,
        };
    },
    doGetPersonalInfo () {
        const param = {
            userId: app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_PERSONAL_INFO, param, this.getPersonalInfoSuccess, this.getInfoError);
    },
    getPersonalInfoSuccess (data) {
        if (data.success) {
            const context = data.context;
            app.personal.set(context);
            this.changeToHomePage();
            app.personal.setNeedLogin(false);
        } else {
            this.getInfoError();
        }
    },
    getInfoError () {
        app.personal.setNeedLogin(true);
        this.changeToLoginPage();
    },
    enterLoginPage (needHideSplashScreen) {
        app.navigator.replace({
            component: Login,
        });
        needHideSplashScreen && SplashScreen.hide();
    },
    changeToLoginPage () {
        if (app.updateMgr.needShowSplash) {
            this.setState({ renderSplashType: 0 }, () => {
                SplashScreen.hide();
            });
        } else {
            this.setState({ renderSplashType: 1 }, () => {
                this.enterLoginPage(true);
            });
        }
    },
    enterHomePage (needHideSplashScreen) {
        app.navigator.replace({
            component: app.authority.info.currentRole == '收货点' ? ReceivePoint : app.authority.info.currentRole == '物流公司' ? app.personal.info.shipper.shipperType == 0 ? LogisticsCompany : ShortLogisticsCompany : NormalUser,
        });
        needHideSplashScreen && SplashScreen.hide();
    },
    changeToHomePage () {
        if (app.updateMgr.needShowSplash) {
            this.setState({ renderSplashType: 0 }, () => {
                SplashScreen.hide();
            });
        } else {
            this.setState({ renderSplashType: 2 }, () => {
                this.enterHomePage(true);
            });
        }
    },
    enterNextPage () {
        app.updateMgr.setNeedShowSplash(false);
        if (this.state.renderSplashType === 0) {
            app.personal.setNeedLogin(true);
            this.setState({ renderSplashType: 1 }, () => {
                this.enterLoginPage();
            });
        } else {
            this.setState({ renderSplashType: 2 }, () => {
                this.enterHomePage();
            });
        }
    },
    changeToNextPage () {
        if (app.personal.needLogin) {
            app.personal.setNeedLogin(true);
            this.changeToLoginPage();
        } else {
            this.doGetPersonalInfo();
        }
    },
    componentDidMount () {
        app.utils.until(
            () => app.personal.initialized && app.updateMgr.initialized && app.navigator,
            (cb) => setTimeout(cb, 100),
            () => this.changeToNextPage()
        );
    },
    componentWillUnmount () {
        app.updateMgr.checkUpdate();
    },
    onLayout (e) {
        const { height } = e.nativeEvent.layout;
        if (this.state.height !== height) {
            this.heightHasChange = !!this.state.height;
            this.setState({ height });
        }
    },
    renderSwiperSplash () {
        const { height } = this.state;
        const marginBottom = (!this.heightHasChange || Math.floor(height) === Math.floor(sr.th)) ? 0 : 30;
        return (
            <View style={{ flex: 1 }} onLayout={this.onLayout}>
                {
                    height &&
                    <Swiper
                        paginationStyle={styles.paginationStyle}
                        dot={<View style={{ backgroundColor:'#87E5D6', width: 8, height: 8, borderRadius: 4, marginLeft: 12, marginRight: 12, marginBottom }} />}
                        activeDot={<View style={{ backgroundColor:'#1A7AE9', width: 18, height: 9, borderRadius: 6, marginLeft: 7, marginRight: 7, marginBottom }} />}
                        height={height}
                        loop={false}>
                        {
                            [1, 2, 3].map((i) => {
                                return (
                                    <Image
                                        key={i}
                                        resizeMode='stretch'
                                        source={app.img['splash_splash' + i]}
                                        style={[styles.bannerImage, { height }]}>
                                        {
                                            i === 3 &&
                                            <TouchableOpacity
                                                style={styles.enterButtonContainer}
                                                onPress={this.enterNextPage}>
                                                <Image resizeMode='stretch' style={styles.enterButton} source={app.img.splash_start} />
                                            </TouchableOpacity>
                                        }
                                    </Image>
                                );
                            })
                        }
                    </Swiper>
                }
            </View>
        );
    },
    render () {
        return this.state.renderSplashType != 0 ? <Image style={{ width:sr.w, height:sr.h }} source={app.img.splash_splash1} /> : this.renderSwiperSplash();
    },
});

const styles = StyleSheet.create({
    paginationStyle: {
        bottom: 30,
    },
    bannerImage: {
        width: sr.w,
    },
    enterButtonContainer: {
        position: 'absolute',
        width: 165,
        height: 40,
        left: (sr.w - 165) / 2,
        bottom: 80,
        alignItems:'center',
        justifyContent: 'center',
    },
    enterButton: {
        width: 140,
        height: 36,
    },
});

```

* PDClient/project/App/modules/test/index.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    FlatList,
    Button,
} = ReactNative;

const SplashScreen = require('@remobile/react-native-splashscreen');

module.exports = React.createClass({
    getInitialState () {
        return {
            list: [1, 2, 3, 4, 5, 6, 7, 8],
        };
    },
    componentWillMount () {
        SplashScreen.hide();
    },
    renderItem ({ item, separators }) {
        console.log('========', separators);
        return (
            <View style={styles.row}>
                <Text>
                    {item}
                </Text>
            </View>
        );
    },
    onPress () {
        this.setState({ list: [2, 3, 4, 5] });
    },
    render () {
        return (
            <View style={styles.container}>
                <Button title='click' onPress={this.onPress} />
                <FlatList
                    data={this.state.list}
                    extraData={this.state}
                    renderItem={this.renderItem}
                    refreshing={false}
                    initialNumToRender={4}
                    ListHeaderComponent={<Text>'这是头'</Text>}
                    ListFooterComponent={<Text>'这是尾'</Text>}
                    ListEmptyComponent={<Text>'没有数据'</Text>}
                    ItemSeparatorComponent={() => <View style={styles.separator} />}
                    onRefresh={() => { console.log('========'); }}
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        paddingTop: 60,
    },
    row: {
        height: 80,
        justifyContent: 'center',
        alignItems: 'center',
    },
    separator: {
        backgroundColor: '#DDDDDD',
        height: 1,
    },
});

```

* PDClient/project/App/modules/test/pageList.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const { PageList } = COMPONENTS;
const SplashScreen = require('@remobile/react-native-splashscreen');

module.exports = React.createClass({
    getInitialState () {
        return {
            list: [1, 2, 3, 4, 5, 6, 7, 8],
        };
    },
    componentWillMount () {
        SplashScreen.hide();
    },
    renderRow (obj) {
        return (
            <View style={styles.row}>
                <Text>
                    {obj}
                </Text>
            </View>
        );
    },
    onPress () {
        this.setState({ list: [2, 3, 4, 5] });
    },
    render () {
        return (
            <View style={styles.container}>
                <PageList
                    ref={(ref) => { this.listView = ref; }}
                    renderRow={this.renderRow}
                    listName={'testList'}
                    listUrl='http://localhost:4000/getList'
                    refreshEnable
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        paddingTop: 60,
    },
    row: {
        height: 80,
        justifyContent: 'center',
        alignItems: 'center',
    },
    separator: {
        backgroundColor: '#DDDDDD',
        height: 1,
    },
});

```

* PDClient/project/App/modules/test/testStartApp.js

```js
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

import React, { Component } from 'react';
import {
  Platform,
  StyleSheet,
  Text,
  View,
} from 'react-native';
const SplashScreen = require('@remobile/react-native-splashscreen');
const instructions = Platform.select({
    ios: 'Press Cmd+R to reload,\n' +
    'Cmd+D or shake for dev menu',
    android: 'Double tap R on your keyboard to reload,\n' +
    'Shake or press menu button for dev menu',
});
import TabNavigator from 'react-native-tab-navigator';
// const CameraRollPicker = require('@remobile/react-native-camera-roll-picker');
// import Swipeout from 'react-native-swipeout';
const { Slider } = require('../components/index');
// const { Button } = require('../components/index');
module.exports = React.createClass({
    componentWillMount () {
        SplashScreen.hide();
    },
    render () {
        return (
            <View style={styles.container}>
                <Text style={styles.welcome}>
          Welcome to React Native!
        </Text>
                <Text style={styles.instructions}>
          To get started, edit App.js
        </Text>
                <Text style={styles.instructions}>
                    {instructions}
                </Text>
                <TabNavigator tabBarStyle={styles.tabBarStyle}
                    tabBarShadowStyle={styles.tabBarShadowStyle}>
                    <TabNavigator.Item
                        key={1}
                        selected
                        title={'item.title'}
                        titleStyle={styles.titleStyle}
                        selectedTitleStyle={styles.titleSelectedStyle}>
                        <Slider />
                    </TabNavigator.Item>
                </TabNavigator>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: 'red',
    },
    welcome: {
        fontSize: 20,
        textAlign: 'center',
        margin: 10,
    },
    instructions: {
        textAlign: 'center',
        color: '#333333',
        marginBottom: 5,
    },
    tabBarStyle: {
        borderColor: '#EEEEEE',
        borderTopWidth: 1,
        height:50,
        backgroundColor: '#FEFCFD',
        alignItems: 'center',
    },
    tabBarShadowStyle: {
        height: 1,
        backgroundColor: '#EEEEEE',
    },
    titleStyle: {
        fontSize:10,
        color: '#929292',
    },
    titleSelectedStyle: {
        fontSize:10,
        color: '#DF3932',
    },
});

```

* PDClient/project/App/modules/test/uppay.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
} = ReactNative;
const { Button } = COMPONENTS;

module.exports = React.createClass({
    doPay () {
        app.uppay.doPay(10000, '充值', (data) => {
            console.log('-----------------1',data);
        }, (data) => {
            console.log('-----------------2',data);
        });
    },
    render () {
        return (
            <View style={styles.container}>
                <Button onPress={this.doPay}>银联支付</Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: 'transparent',
        justifyContent: 'space-around',
        paddingVertical: 150,
    },
});

```

* PDClient/project/App/modules/test/wxpay.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
} = ReactNative;
const { Button } = COMPONENTS;

const Wxpay = require('../../native/index.js').WeixinPay;

module.exports = React.createClass({
    componentWillMount () {
        const param = {
            userId: '59c1c5a2c2e39a4a3f34bf0b', // app.personal.info.userId,
            total_fee:1, // 总金额 (分)
            body:'充值',
        };
        POST(app.route.ROUTE_GET_UNIFIED_ORDER, param, this.getUnifiedOrderSuccess, true);
    },
    getUnifiedOrderSuccess (data) {
        console.log('---------------', data);
        if (data.success) {
            Wxpay.pay(data.context, function (results) { console.log('success:', results); }, function (results) { Toast('error:' + results); });
        }
    },
    /*
    * 注：订单总金额，只能为整数，单位为【分】，参数值不能带小数。
    * appid: 公众账号ID
    * noncestr: 随机字符串
    * package: 扩展字段
    * partnerid: 商户号
    * prepayid: 预支付交易会话ID
    * timestamp: 时间戳
    * sign: 签名
    */
    doPay () {
        Wxpay.pay({
            appid: 'wxcbe9347be9959618',
            noncestr: '6E19AACCBF1947C6B4174002AA4A0880',
            package: 'Sign=WXPay',
            partnerid: '1319502301',
            prepayid: 'wx201604261017572078a543080024071363',
            timestamp:'1461637077',
            sign: 'D33D5A6FAE941D5253DF998247B1C75A',
        }, function (results) { console.log('success:', results); }, function (results) { Toast('error:' + results); });
    },
    render () {
        return (
            <View style={styles.container}>
                <Button onPress={this.doPay}>微信支付</Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: 'transparent',
        justifyContent: 'space-around',
        paddingVertical: 150,
    },
});

```

* PDClient/project/App/modules/update/UpdateInfoBox.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableOpacity,
} = ReactNative;

const Update = require('@remobile/react-native-update');

const
    STATUS_HAS_VEW_VERSION = 0,
    STATUS_DOWNLOAD_APK_PROGESS = 1,
    STATUS_DOWNLOAD_JS_PROGESS = 2,
    STATUS_UNZIP_JS_PROGESS = 3,
    STATUS_DOWNKOAD_APK_ERROR = 4,
    STATUS_DOWNKOAD_JS_ERROR = 5,
    STATUS_UNZIP_JS_ERROR = 6,
    STATUS_FAILED_INSTALL_ERROR = 7,
    STATUS_UPDATE_END = 8;

const
    ERROR_DOWNKOAD_APK = 1,
    ERROR_DOWNKOAD_JS = 2,
    ERROR_FAILED_INSTALL = 3,
    ERROR_UNZIP_JS = 4;

const PROGRESS_WIDTH = sr.tw * 0.7;
const { ProgressBar } = COMPONENTS;

const ProgressInfo = React.createClass({
    render () {
        const { progress } = this.props;
        if (progress < 1000) {
            return (
                <View style={[styles.functionContainer, { alignItems: 'center', paddingVertical: 30 }]}>
                    <Text>{this.props.title} [{progress}%]</Text>
                    <ProgressBar
                        fillStyle={{}}
                        backgroundStyle={{ backgroundColor: '#cccccc', borderRadius: 2 }}
                        style={{ marginTop: 10, width:PROGRESS_WIDTH }}
                        progress={progress / 100.0}
                        />
                    <View style={styles.progressText}>
                        <Text>0</Text>
                        <Text>100</Text>
                    </View>
                </View>
            );
        } else {
            const size = progress / 1000 / 1024 / 1024;
            return (
                <View style={[styles.functionContainer, { alignItems: 'center', paddingVertical: 30 }]}>
                    <Text>{this.props.title} [ {size.toFixed(2)} M ]</Text>
                </View>
            );
        }
    },
});

module.exports = React.createClass({
    getInitialState () {
        return {
            status:STATUS_HAS_VEW_VERSION,
            progress: 0,
        };
    },
    onError (errCode) {
        if (errCode == ERROR_DOWNKOAD_APK) {
            this.setState({ status: STATUS_DOWNKOAD_APK_ERROR });
        } else if (errCode == ERROR_DOWNKOAD_JS) {
            this.setState({ status: STATUS_DOWNKOAD_JS_ERROR });
        } else if (errCode == ERROR_FAILED_INSTALL) {
            this.setState({ status: STATUS_FAILED_INSTALL_ERROR });
        } else if (errCode == ERROR_UNZIP_JS) {
            this.setState({ status: STATUS_UNZIP_JS_ERROR });
        }
    },
    doUpdate () {
        const { jsVersionCode, trackViewUrl } = this.props.options;
        if (jsVersionCode !== undefined) {
            Update.updateJS({
                jsVersionCode,
                jsbundleUrl: app.isandroid ? app.route.ROUTE_JS_ANDROID_URL : app.route.ROUTE_JS_IOS_URL,
                onDownloadJSProgress:(progress) => { this.setState({ status: STATUS_DOWNLOAD_JS_PROGESS, progress }); },
                onUnzipJSProgress:(progress) => { this.setState({ status: STATUS_UNZIP_JS_PROGESS, progress }); },
                onUnzipJSEnd:() => { this.setState({ status: STATUS_UPDATE_END }); },
                onError:(errCode) => { this.onError(errCode); },
            });
        } else {
            Update.updateApp({
                trackViewUrl,
                androidApkUrl:app.route.ROUTE_APK_URL,
                androidApkDownloadDestPath:'/sdcard/pdclient.apk',
                onDownloadAPKProgress:(progress) => { this.setState({ status: STATUS_DOWNLOAD_APK_PROGESS, progress }); },
                onError:(errCode) => { this.onError(errCode); },
            });
        }
    },
    render () {
        const components = {};
        const { newVersion, description } = this.props.options;
        components[STATUS_HAS_VEW_VERSION] = (
            <View style={styles.functionContainer}>
                <Text style={styles.title}>{`发现新版本(${newVersion})`}</Text>
                <Text style={styles.redLine} />
                <Text style={styles.content}>
                    {'更新内容：'}
                </Text>
                {
                    description.map((item, i) => {
                        return (
                            <Text style={styles.contentItem} key={i}>{'- ' + item}</Text>
                        );
                    })
                }
                <View style={styles.buttonViewStyle}>
                    <TouchableOpacity
                        onPress={app.closeModal}
                        style={styles.buttonStyleContainCannel}>
                        <Text style={styles.buttonStyleCannel}>以后再说</Text>
                    </TouchableOpacity>
                    <TouchableOpacity
                        onPress={this.doUpdate}
                        style={styles.buttonStyleContain}>
                        <Text style={styles.buttonStyle} >立即更新</Text>
                    </TouchableOpacity>
                </View>
            </View>
        );
        components[STATUS_DOWNKOAD_APK_ERROR] = (
            <View style={[styles.functionContainer, { alignItems: 'center', paddingVertical: 30 }]}>
                <Text style={styles.textInfo}>下载apk文件失败，请在设置里重新更新</Text>
                <TouchableOpacity
                    onPress={app.closeModal}
                    style={styles.buttonStyleContainCannel}>
                    <Text style={styles.buttonStyleCannel}>我知道了</Text>
                </TouchableOpacity>
            </View>
        );
        components[STATUS_DOWNKOAD_JS_ERROR] = (
            <View style={[styles.functionContainer, { alignItems: 'center', paddingVertical: 30 }]}>
                <Text style={styles.textInfo}>下载js bundle失败，请在设置里重新更新</Text>
                <TouchableOpacity
                    onPress={app.closeModal}
                    style={styles.buttonStyleContainCannel}>
                    <Text style={styles.buttonStyleCannel}>我知道了</Text>
                </TouchableOpacity>
            </View>
        );
        components[STATUS_UNZIP_JS_ERROR] = (
            <View style={[styles.functionContainer, { alignItems: 'center', paddingVertical: 30 }]}>
                <Text style={styles.textInfo}>解压js bundle失败，请在设置里重新更新</Text>
                <TouchableOpacity
                    onPress={app.closeModal}
                    style={styles.buttonStyleContainCannel}>
                    <Text style={styles.buttonStyleCannel}>我知道了</Text>
                </TouchableOpacity>
            </View>
        );
        components[STATUS_FAILED_INSTALL_ERROR] = (
            <View style={[styles.functionContainer, { alignItems: 'center', paddingVertical: 30 }]}>
                <Text style={styles.textInfo}>你放弃了安装</Text>
                <TouchableOpacity
                    onPress={app.closeModal}
                    style={styles.buttonStyleContainCannel}>
                    <Text style={styles.buttonStyleCannel}>我知道了</Text>
                </TouchableOpacity>
            </View>
        );
        components[STATUS_DOWNLOAD_APK_PROGESS] = (
            <ProgressInfo
                title='正在下载APK'
                progress={this.state.progress} />
        );
        components[STATUS_DOWNLOAD_JS_PROGESS] = (
            <ProgressInfo
                title='正在下载Bundle文件'
                progress={this.state.progress} />
        );
        components[STATUS_UNZIP_JS_PROGESS] = (
            <ProgressInfo
                title='正在解压Bundle文件'
                progress={this.state.progress} />
        );
        components[STATUS_UPDATE_END] = (
            <Text>即将重启...</Text>
        );
        return (
            <View style={styles.container}>
                {components[this.state.status]}
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
    },
    functionContainer: {
        width:sr.w - 60,
        backgroundColor:'#FFFFFF',
    },
    progressText: {
        flexDirection:'row',
        justifyContent:'space-between',
        width: sr.w * 0.7,
    },
    textInfo: {
        color: '#000000',
        fontSize: 14,
        marginBottom: 20,
        textAlign: 'center',
    },
    buttonViewStyle: {
        flexDirection: 'row',
        width: sr.w - 40,
        height: 60,
        justifyContent: 'center',
    },
    redLine: {
        marginTop: 15,
        width: sr.w - 60,
        height: 1,
        backgroundColor: '#2F4F4F',
    },
    buttonStyleContain: {
        width: 120,
        height: 35,
        marginLeft: 30,
        justifyContent:'center',
        alignItems:'center',
        backgroundColor: '#2F4F4F',
    },
    buttonStyleContainCannel: {
        width: 120,
        height: 35,
        justifyContent:'center',
        alignItems:'center',
        backgroundColor: 'white',
        borderWidth: 1,
        borderColor: '#2F4F4F',
    },
    buttonStyle: {
        fontSize: 16,
        color: 'white',
        fontFamily: 'STHeitiSC-Medium',
    },
    buttonStyleCannel: {
        fontSize: 16,
        color: '#2F4F4F',
        fontFamily: 'STHeitiSC-Medium',
    },
    title: {
        color: '#2F4F4F',
        fontSize: 18,
        textAlign: 'center',
        overflow: 'hidden',
        marginTop: 15,
        fontFamily: 'STHeitiSC-Medium',
    },
    content: {
        color:'#000000',
        marginTop: 10,
        marginBottom: 10,
        marginHorizontal: 20,
        fontSize:16,
        fontFamily: 'STHeitiSC-Medium',
    },
    contentItem: {
        color:'#000000',
        marginBottom: 10,
        marginHorizontal: 20,
        fontSize:12,
    },
});

```

* PDClient/project/App/modules/update/UpdatePage.js

```js
'use strict';
const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
} = ReactNative;

const Update = require('@remobile/react-native-update');

const
    STATUS_GET_VERSION = 0,
    STATUS_HAS_VEW_VERSION = 1,
    STATUS_HAS_NOT_VEW_VERSION = 2,
    STATUS_DOWNLOAD_APK_PROGESS = 3,
    STATUS_DOWNLOAD_JS_PROGESS = 4,
    STATUS_UNZIP_JS_PROGESS = 5,
    STATUS_GET_VERSION_ERROR = 6,
    STATUS_DOWNKOAD_APK_ERROR = 7,
    STATUS_DOWNKOAD_JS_ERROR = 8,
    STATUS_UNZIP_JS_ERROR = 9,
    STATUS_FAILED_INSTALL_ERROR = 10,
    STATUS_UPDATE_END = 11;

const
    ERROR_DOWNKOAD_APK = 1,
    ERROR_DOWNKOAD_JS = 2,
    ERROR_FAILED_INSTALL = 3,
    ERROR_UNZIP_JS = 4;

const PROGRESS_WIDTH = sr.tw * 0.7;
const { Button, ProgressBar } = COMPONENTS;

const ProgressInfo = React.createClass({
    render () {
        const { progress } = this.props;
        if (progress < 1000) {
            return (
                <View>
                    <Text>{this.props.title} [{progress}%]</Text>
                    <ProgressBar
                        fillStyle={{}}
                        backgroundStyle={{ backgroundColor: '#cccccc', borderRadius: 2 }}
                        style={{ marginTop: 10, width:PROGRESS_WIDTH }}
                        progress={progress / 100.0}
                        />
                    <View style={styles.progressText}>
                        <Text>0</Text>
                        <Text>100</Text>
                    </View>
                </View>
            );
        } else {
            const size = progress / 1000 / 1024 / 1024;
            return (
                <View style={{ flex: 1, alignItems: 'center' }}>
                    <Text>{this.props.title} [{size.toFixed(2)} M]</Text>
                </View>
            );
        }
    },
});

module.exports = React.createClass({
    statics: {
        title: '在线更新',
    },
    pageName: 'UpdatePage',
    getInitialState () {
        const { options } = this.props;
        return {
            options,
            status: !options ? STATUS_GET_VERSION : options.newVersion ? STATUS_HAS_VEW_VERSION : STATUS_HAS_NOT_VEW_VERSION,
            progress: 0,
        };
    },
    componentWillMount () {
        if (!this.state.options) {
            Update.checkVersion({
                versionUrl: app.route.ROUTE_VERSION_INFO_URL,
                iosAppId: CONSTANTS.IOS_APPID,
            }).then((options) => {
                this.setState({ options, status: !options ? STATUS_GET_VERSION_ERROR : options.newVersion ? STATUS_HAS_VEW_VERSION : STATUS_HAS_NOT_VEW_VERSION });
            });
        }
    },
    onError (errCode) {
        if (errCode == ERROR_DOWNKOAD_APK) {
            this.setState({ status: STATUS_DOWNKOAD_APK_ERROR });
        } else if (errCode == ERROR_DOWNKOAD_JS) {
            this.setState({ status: STATUS_DOWNKOAD_JS_ERROR });
        } else if (errCode == ERROR_FAILED_INSTALL) {
            this.setState({ status: STATUS_FAILED_INSTALL_ERROR });
        } else if (errCode == ERROR_UNZIP_JS) {
            this.setState({ status: STATUS_UNZIP_JS_ERROR });
        }
    },
    doUpdate () {
        const { jsVersionCode, trackViewUrl } = this.state.options;
        if (jsVersionCode !== undefined) {
            Update.updateJS({
                jsVersionCode,
                jsbundleUrl: app.isandroid ? app.route.ROUTE_JS_ANDROID_URL : app.route.ROUTE_JS_IOS_URL,
                onDownloadJSProgress:(progress) => { this.setState({ status: STATUS_DOWNLOAD_JS_PROGESS, progress }); },
                onUnzipJSProgress:(progress) => { this.setState({ status: STATUS_UNZIP_JS_PROGESS, progress }); },
                onUnzipJSEnd:() => { this.setState({ status: STATUS_UPDATE_END }); },
                onError:(errCode) => { this.onError(errCode); },
            });
        } else {
            Update.updateApp({
                trackViewUrl,
                androidApkUrl:app.route.ROUTE_APK_URL,
                androidApkDownloadDestPath:'/sdcard/pdclient.apk',
                onDownloadAPKProgress:(progress) => { this.setState({ status: STATUS_DOWNLOAD_APK_PROGESS, progress }); },
                onError:(errCode) => { this.onError(errCode); },
            });
        }
    },
    render () {
        const components = {};
        const { APP_NAME } = CONSTANTS;
        const { currentVersion, newVersion, description } = this.state.options || { currentVersion:Update.getVersion() };
        components[STATUS_GET_VERSION] = (
            <Text style={styles.textInfo}>正在获取版本号</Text>
        );
        components[STATUS_HAS_NOT_VEW_VERSION] = (
            <Text style={styles.textInfo}>当前版本已经是最新版本</Text>
        );
        components[STATUS_GET_VERSION_ERROR] = (
            <Text style={styles.textInfo}>获取版本信息失败，请稍后再试</Text>
        );
        components[STATUS_DOWNKOAD_APK_ERROR] = (
            <Text style={styles.textInfo}>下载apk文件失败，请稍后再试</Text>
        );
        components[STATUS_DOWNKOAD_JS_ERROR] = (
            <Text style={styles.textInfo}>下载js bundle失败，请稍后再试</Text>
        );
        components[STATUS_UNZIP_JS_ERROR] = (
            <Text style={styles.textInfo}>解压js bundle失败，请稍后再试</Text>
        );
        components[STATUS_FAILED_INSTALL_ERROR] = (
            <Text style={styles.textInfo}>你放弃了安装</Text>
        );
        components[STATUS_HAS_VEW_VERSION] = (
            <View style={styles.textInfoContainer}>
                <Text style={styles.textInfo}>发现新版本{newVersion}</Text>
                <View style={styles.descriptionContainer}>
                    {
                        description && description.map((item, i) => {
                            return (
                                <Text style={styles.textInfo} key={i}>{(i + 1) + '. ' + item}</Text>
                            );
                        })
                    }
                </View>
                <Button onPress={this.doUpdate} style={styles.button_layer} textStyle={styles.button_text}>立即更新</Button>
            </View>
        );
        components[STATUS_DOWNLOAD_APK_PROGESS] = (
            <ProgressInfo
                title='正在下载APK'
                progress={this.state.progress} />
        );
        components[STATUS_DOWNLOAD_JS_PROGESS] = (
            <ProgressInfo
                title='正在下载Bundle文件'
                progress={this.state.progress} />
        );
        components[STATUS_UNZIP_JS_PROGESS] = (
            <ProgressInfo
                title='正在解压Bundle文件'
                progress={this.state.progress} />
        );
        components[STATUS_UPDATE_END] = (
            <Text>正在重启...</Text>
        );
        return (
            <View style={styles.container}>
                <View style={styles.logoContainer}>
                    <Image
                        resizeMode='stretch'
                        source={app.img.personal_logo}
                        style={styles.logo}
                        />
                    <Text style={styles.app_name}>{APP_NAME + 'V' + currentVersion}</Text>
                </View>
                <View style={styles.functionContainer}>
                    {components[this.state.status]}
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex:1,
    },
    logoContainer: {
        alignItems:'center',
        marginTop: 52,
    },
    logo: {
        width: 83,
        height: 83,
    },
    app_name: {
        marginTop: 13,
        fontSize: 16,
        color: '#727272',
    },
    functionContainer: {
        flex: 1,
        width: sr.w,
        marginTop: 51,
        alignItems:'center',
    },
    button_layer: {
        width:178,
        height:49,
        borderRadius: 4,
        left: (sr.w - 178) / 2,
        position: 'absolute',
        bottom: 120,
        backgroundColor: '#DE3031',
    },
    button_text: {
        fontSize: 18,
        fontWeight: '400',
        color: '#FFFFFF',
        alignSelf: 'center',
        fontFamily: 'STHeitiSC-Medium',
    },
    progressText: {
        flexDirection:'row',
        justifyContent:'space-between',
        width: sr.w * 0.7,
    },
    textInfoContainer: {
        flex: 1,
        width: sr.w,
    },
    textInfo: {
        color: '#000000',
        fontSize: 18,
        marginBottom: 6,
        textAlign: 'center',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/cargo/AddCar.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    TouchableOpacity,
} = ReactNative;
const { Button, PageList } = COMPONENTS;
import moment from 'moment';
const CarInfo = require('./SelectCarType');

module.exports = React.createClass({
    statics: {
        title: '添加货车',
    },
    getInitialState () {
        return {
            plateNo:app.personal.info.truck && app.personal.info.truck.plateNo,
        };
    },
    submit () {
        const { plateNo } = this.state;
        if (!app.utils.checkPlate(plateNo)) {
            Toast('请输入合法车牌号');
        }  else {
            this.setCityTruck();
        }
    },
    setCityTruck () {
        const { plateNo } = this.state;
        const param = {
            userId: app.personal.info.userId,
            plateNo,
        };
        POST(app.route.ROUTE_SET_CITY_TRUCK, param, this.doSetCityTruckSuccess, true);
    },
    doSetCityTruckSuccess (data) {
        if (data.success) {
            Toast('提交成功');
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { carInfo, plateNo, drivingLicense } = this.state;
        return (
            <View style={styles.container}>
                <View>
                    <View style={styles.title}>
                        <Text style={styles.titleFont}>货车信息</Text>
                    </View>
                    <View>
                        <View style={styles.row}>
                            <Text style={styles.carInfoTitle}>车牌</Text>
                            <TextInput style={styles.carInfo}
                                underlineColorAndroid='transparent'
                                maxLength={7}
                                selectTextOnFocus
                                onChangeText={(text) => this.setState({ plateNo: text })}
                                defaultValue={!app.personal.info.truck ? '' : app.personal.info.truck.plateNo }
                                placeholderColor='#666666'
                                placeholder='请输入车牌号' />
                        </View>
                    </View>
                    <Button onPress={this.submit} style={styles.btn} textStyle={styles.btnFont}>确认添加</Button>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    title:{
        height:30,
        justifyContent:'center',
        marginLeft:10,
    },
    titleFont:{
        fontSize:13,
    },
    row:{
        height:40,
        flexDirection:'row',
        backgroundColor:'#FFFFFF',
        marginBottom:1,
        alignItems:'center',
    },
    carInfo:{
        flex:1.6,
        fontSize:14,
        padding:0,
        color:'#333333',
        marginLeft:(sr.w - 300) / 2,
        textAlign:'right',
        marginRight:10,
    },
    carInfoTitle:{
        flex:0.4,
        fontSize:14,
        color:'#666666',
        marginLeft:10,
    },
    btn:{
        width:351,
        height:45,
        marginTop:18,
        borderRadius:4,
        alignSelf:'center',
        backgroundColor:'#FE9917',
    },
    btnFont:{
        fontWeight:'400',
    },
    history:{
        fontSize:13,
        marginLeft:10,
        marginTop:50,
        marginBottom:8,
    },
    listRow:{
        backgroundColor:'#FFFFFF',
        marginBottom:1,
        paddingLeft:10,
        paddingTop:5,
        height:50,
    },
    rowTitle:{
        flex:1,
    },
    rowInfo:{
        flex:1,
        fontSize:12,
        color:'#888888',
    },
    rowTime:{
        position:'absolute',
        left:(sr.w + 180) / 2,
        top:5,
        color:'#888888',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/cargo/BranchList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    ListView,
    TouchableOpacity,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '物流超市',
        leftButton:{ handler: () => { app.scene.goBack&&app.scene.goBack(); } },
    },
    goBack () {
        const { onWillFocus } = this.props;
        app.pop();
        if (onWillFocus) {
            onWillFocus();
        }
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            choose:app.personal.currentShopIndex,
        };
    },
    doChoose (rowId) {
        const { setCurrentShopId } = this.props;
        this.setState({
            choose:rowId,
        });
        app.personal.setCurrentShopIndex(rowId);
        app.personal.currentShopName = app.personal.info.shipper && app.personal.info.shipper.registerShopList[app.personal.currentShopIndex].name || '';
        app.personal.currentShopId = app.personal.info.shipper && app.personal.info.shipper.registerShopList[app.personal.currentShopIndex].id || null;
        if (setCurrentShopId) {
            setCurrentShopId(app.personal.currentShopId);
        }
    },
    renderRow (obj, sectionID, rowId) {
        const { choose } = this.state;
        return (
            <TouchableOpacity onPress={this.doChoose.bind(null, rowId)}>
                <View style={styles.itemContainer}>
                    <Text style={styles.itemText}>{obj.name}</Text>
                    <Image style={styles.choose}
                        source={choose == rowId ? app.img.common_radio : {}} />
                </View>
            </TouchableOpacity>
        );
    },
    render () {
        return (
            <View style={styles.container}>
                <ListView
                    dataSource={this.ds.cloneWithRows(app.personal.info.shipper.registerShopList)}
                    renderRow={this.renderRow}
                    enableEmptySections
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        paddingTop:10,
    },
    itemContainer:{
        height:45,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        justifyContent:'space-between',
        alignItems:'center',
    },
    itemText:{
        fontSize:14,
        color:'#333333',
        marginLeft:15,
    },
    choose:{
        height:10,
        width:10,
        marginRight:15,
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/cargo/OrderDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
    TextInput,
} = ReactNative;

const { Button, CustomSheet, TouchAbleLink,DImage } = COMPONENTS;
const PayInfo = require('../../common/PayInfo');
const PayPassword = require('../../common/PayPassword');
const LogisticsDetail = require('../../common/LogisticsDetail');
const ConfirmSendDoorAddress = require('../../common/ConfirmSendDoorAddress');
const moment = require('moment');

module.exports = React.createClass({
    statics: {
        title: '货单详情',
    },
    getInitialState () {
        return {
            data:{},
        };
    },
    componentWillMount () {
        this.getOrderDetail();
    },
    getOrderDetail () {
        const param = {
            userId: app.personal.info.userId,
            orderId: this.props.orderId,
        };
        POST(app.route.ROUTE_GET_ORDER_DETAIL, param, this.getOrderDetailSuccess, true);
    },
    getOrderDetailSuccess (data) {
        if (data.success) {
            this.setState({ data:data.context });
        } else {
            Toast(data.msg);
        }
    },
    toLogisticsDetail () {
        const { data } = this.state;
        app.push({
            component:LogisticsDetail,
            passProps:{
                obj:data,
            },
        });
    },
    toPayMode (payMode) {
        if (payMode == 0) {
            return '现付';
        } else if (payMode == 1) {
            return '到付';
        } else {
            return '混合付款';
        }
    },
    sendDoorPrice(endPointLastCode){
        app.showModal(
            <ConfirmSendDoorAddress orderId={this.props.orderId} refresh={this.props.refresh} endPointLastCode={endPointLastCode}/>
        );
    },
    sendDoorBySelf(){
        const param = {
            userId:app.personal.info.userId,
            orderId:this.props.orderId,
            isTransport:false,
            isScan:false,
        };
        POST(app.route.ROUTE_SET_CLIENT_PICK_ORDER_TRANSPORT_FEE,param,this.setSendDoorSuccess,false);
    },
    setSendDoorSuccess(data){
        if (data.success) {
            Toast('状态改变成功');
            this.props.refresh();
            app.pop();
        }else{
            Toast(data.msg);
        }
    },
    render () {
        const { data } = this.state;
        const { stateList } = data;
        return (
            <View style={styles.container}>
                <View style={styles.containerMarket}>
                    <DImage style={styles.marketLogo}
                        source={app.img.order_logo}
                        resizeMode='stretch'
                        />
                    <Text style={styles.textMarket}>{ data.shop ? data.shop.name : data.agent ? data.agent.name : ''}</Text>
                    <Text style={styles.orderState}>{ stateList ? OS.MAP[stateList[0].state] : ''}</Text>
                </View>
                    <View style={styles.containerDetails}>
                        <DImage style={styles.imageGoods}
                            resizeMode='stretch'
                            defaultSource={app.img.common_default}
                            source={{ uri:data.photo }}
                            />
                        <View style={styles.goodsDetails}>
                            <Text style={styles.textDetailsItem}>收货人：{!data.receiverName ? '暂无' : data.receiverName}</Text>
                            <Text style={styles.textDetailsItem}>收货人电话:{!data.receiverPhone ? '暂无' : data.receiverPhone}</Text>
                            <Text style={styles.textDetailsItem}>到货地：{data && data.endPoint}</Text>
                            <Text style={styles.textDetailsItem}>方量：{data.size}m³ 重量：{data.weight}吨 件数：{data.totalNumbers}件 </Text>
                            <Text style={styles.textDetailsItem}>下单时间：{data.createTime} </Text>
                        </View>
                    </View>
                    <View style={styles.containerWhite} />
                    <View style={styles.containerInfo}>
                        <Text style={styles.textInfoTop}>支付方式：{this.toPayMode(data.payMode)}</Text>
                            <Text style={styles.textInfo}>运费总额：¥{app.utils.N(data.payMode == 0 ? data.needPayTransportFee :
                                data.payMode == 1 ?  data.totalDesignatedFee :  data.totalDesignatedFee + data.needPayTransportFee ) + app.utils.N(data.additionalFee)}(实际所得运费：¥{ data.additionalFee + data.fee })</Text>
                        <Text style={styles.textInfo}>代收货款：¥{app.utils.N(data.proxyCharge)}</Text>
                        {
                            data.payMode != 0 &&
                            <View style={styles.payInfo}>
                                <Text style={styles.textTotal}>需收款:</Text>
                                <Text style={styles.payInfoText}>¥{app.utils.N(data.totalDesignatedFee + data.additionalFee +data.proxyCharge)}</Text>
                            </View>

                        }
                    </View>
                <TouchableOpacity onPress={this.toLogisticsDetail}>
                    <View style={styles.logInfo}>
                        <Text style={styles.textLogInfo}>仓储</Text>
                        <View style={styles.logInfoRight}>
                            <Text style={styles.textLogInfoRight}>{data.warehouse}</Text>
                        </View>
                    </View>
                </TouchableOpacity>
                <View style={styles.bottomView}>
                    <View style={styles.btnPhone} >
                        <Image source={app.img.shipper_tell_phone} style={styles.phone} resizeMode='contain'/>
                        <TouchAbleLink url={'tel:' + data.receiverPhone }
                        children={ <Text style={styles.btnCallText}>联系客户</Text>}/>
                    </View>
                    <Button  style={styles.btnBySelf} textStyle={styles.btnReceiveText} onPress={this.sendDoorBySelf}>
                        客户自提
                    </Button>
                    <Button  style={styles.btnReceive} textStyle={styles.btnReceiveText} onPress={this.sendDoorPrice.bind( null, data.endPointLastCode)}>
                        送货上门
                    </Button>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    containerTop:{
        height:60,
        backgroundColor:'#c81622',
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'center',
    },
    imageTop:{
        height:32,
        width:27,
    },
    textTop:{
        fontSize:12,
        fontWeight:'900',
        marginLeft:20,
        color:'#ffffff',
    },
    containerReceive:{
        marginTop:8,
        paddingLeft:10,
        height:55,
        backgroundColor:'#ffffff',
        justifyContent: 'center',
    },
    textAddress:{
        fontSize:12,
        marginBottom:3,
        color:'#333333',
        flex:1,
    },
    textName:{
        fontSize:12,
        color:'#333333',
        flex:1,
    },
    containerMarket:{
        height:25,
        marginTop:8,
        alignItems: 'center',
        justifyContent: 'space-between',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    containerMarketLeft:{
        flexDirection: 'row',
    },
    marketLogo:{
        height:13,
        width:16.5,
        marginLeft:10,
    },
    textMarket:{
        flex:1,
        marginLeft:10,
        color:'#333333',
        fontSize:12,
    },
    orderState:{
        color:'#00a117',
        marginRight:10,
        fontSize:12,
    },
    containerDetails:{
        height:105,
        flexDirection: 'row',
    },
    containerWhite:{
        height:8,
        backgroundColor:'#ffffff',
    },
    imageGoods:{
        height:75,
        width:75,
        marginLeft:12,
        marginTop:11,
    },
    payInfo:{
        flexDirection:'row',
    },
    payInfoText:{
        marginTop:3,
        fontSize:12,
        color:'#f01114',
        fontWeight:'400',
        height:15,
    },
    goodsDetails:{
        marginLeft:8,
        height:95,
        paddingVertical:8,
        justifyContent: 'center',
    },
    goodsDetailsTop:{
        width:sr.w - 99,
        flexDirection: 'row',
        justifyContent: 'space-between',
    },
    detailsTopLeft:{
        flexDirection: 'row',
        alignItems: 'center',
        marginTop:15,
    },
    detailsTopRight:{
        flexDirection: 'row',
        alignItems: 'center',
        marginTop:15,
    },
    detailsAddress:{
        fontSize:14,
        width:80,
        color:'#333333',
        flexDirection: 'row',

    },
    moneyLeft:{
        fontSize:12,
        color:'#333333',
    },
    moneyRight:{
        fontSize:13,
        fontWeight:'500',
        color:'#f01114',
        marginRight:10,
    },
    textDetailsItem:{
        marginTop:1,
        fontSize:12,
        color:'#888888',
    },
    containerOrderNumber:{
        height:50,
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingLeft:10,
        justifyContent: 'center',
    },
    textOrderNumber:{
        color:'#333333',
        marginBottom:3,
        fontSize:12,
    },
    textDate:{
        color:'#888888',
        marginTop:3,
        fontSize:14,
    },
    containerInfo:{
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingLeft:10,
        paddingBottom:5,
        paddingTop:5,
    },
    textInfoTop:{
        color:'#888888',
        fontSize:12,
        height:15,
    },
    textInfo:{
        color:'#888888',
        fontSize:12,
        marginTop:3,
        height:15,
    },
    textTotal:{
        marginTop:3,
        fontSize:12,
        color:'#333333',
        fontWeight:'400',
        height:15,
    },
    logInfo:{
        height:40,
        backgroundColor:'#ffffff',
        marginTop:8,
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textLogInfo:{
        color:'#888888',
        fontSize:14,
        marginLeft:10,
    },
    logInfoRight:{
        marginRight:10,
        flexDirection: 'row',
        alignItems: 'center',
    },
    textLogInfoRight:{
        fontSize:14,
        color:'#333333',
        marginRight:5,
    },
    imageCommon_go:{
        width: 8,
        height: 15,
        marginRight: 7,
    },
    btnReceive:{
        height:50,
        borderRadius:0,
        flex:0.8,
    },
    btnReceiveText:{
        fontSize:16,
        fontWeight:'500',
    },
    isReachPay:{
        backgroundColor:'#f4f4f4',
        width:sr.w,
        height:45,
    },
    btnPhone:{
        flexDirection:'row',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
        flex:1,
        height:50,
    },
    bottomView:{
        flex:1,
        alignItems:'flex-end',
        flexDirection:'row',
    },
    btnCall:{
        backgroundColor:'#FFFFFF',
        height:50,
        borderRadius:0,
    },
    btnCallText:{
        fontSize:16,
        color:'#333333',
    },
    phone:{
        height:20,
    },
    btnBySelf:{
        height:50,
        borderRadius:0,
        backgroundColor:'#F79835',
        flex:0.8,
    },
    sendDoorAddress:{
        marginTop:10,
    },
    listView:{
        height:1,
        backgroundColor:'#333333',
    },
    containerItem:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textLeft:{
        fontSize:14,
        color:'#333333',
        marginLeft:10,
    },
    textRight:{
        fontSize:14,
        color:'#888888',
        marginRight:10,
    },
    text_input: {
        height:40,
        width: 200,
        padding:0,
        fontSize:14,
        alignSelf: 'center',
        textAlign:'right',
    },
    rightView:{
        flexDirection: 'row',
        alignItems: 'center',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/cargo/ScanOrderDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button,DImage } = COMPONENTS;
import moment from 'moment';

module.exports = React.createClass({
    statics: {
        title: '扫描订单详情',
    },
    getInitialState () {
        return {
            data:[],
            count:1,
            unStoageNumbers:0,
        };
    },
    componentWillMount () {
        this.getOrderDetail();
    },
    getOrderDetail () {
        const { orderId } = this.props;
        const param = {
            userId:app.personal.info.userId,
            orderId,
            isScan:true,
            type:0,
        };
        POST(app.route.ROUTE_GET_ORDER_DETAIL, param, this.postScanOrderSuccess, true);
    },
    postScanOrderSuccess (data) {
        if (data.success) {
            if (_.find(data.context.stateList, o => o.state == 5 && o.count)) {
                this.setState({
                    data:data.context,
                    unStoageNumbers: _.find(data.context.stateList, o => o.state == 5 && o.count).count,
                });
            }else {
                Toast('该货单已全部收货完成');
                app.pop();
            }
        } else {
            Toast(data.msg);
            app.pop();
        }
    },
    minusCount () {
        const { count } = this.state;
        if (count > 0) {
            this.setState({ count:count - 1 });
        } else {
            Toast('入库数量不能小于0');
        }
    },
    addCount () {
        const { count, unStoageNumbers } = this.state;
        if (count < unStoageNumbers) {
            this.setState({ count:count + 1 });
        } else {
            Toast('入库数量不能大于未装载件数');
        }
    },
    showScanResult () {
        const { orderId } = this.props;
        const { count, unStoageNumbers } = this.state;
        if (count >= 0 && count <= unStoageNumbers) {
            app.navigator.replace({
                component: require('./ScanResult'),
                passProps:{
                    orderId,
                    count:count * 1,
                },
            });
        } else {
            Toast('入库数量有误');
        }
    },
    doCancel () {
        app.navigator.popToTop();
    },
    render () {
        const { data, count, unStoageNumbers } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.containerDetails}>
                    <DImage style={styles.imageGoods}
                        resizeMode='stretch'
                        defaultSource={app.img.common_default}
                        source={{ uri:data.photo ? data.photo : '' }}
                        />
                    <View style={styles.goodsDetails}>
                        <View style={styles.goodsDetailsTop}>
                            <View style={styles.detailsTopLeft}>
                                <Text style={styles.detailsAddress}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{app.personal.currentShopName}</Text>
                                <Text >-</Text>
                                <Text style={styles.detailsAddress}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{data.endPoint}</Text>
                            </View>
                        </View>
                        <Text style={styles.textDetailsItem}>总重量：{app.utils.N(data.weight)}m³</Text>
                        <Text style={styles.textDetailsItem}>总方量：{app.utils.N(data.size)}吨</Text>
                        <Text style={styles.textDetailsItem}>未入库件数：{app.utils.N(unStoageNumbers)}件</Text>
                        <Text style={styles.textDetailsItem}>下单时间：{moment(data.createTime).format('YYYY-MM-DD')}</Text>
                    </View>
                </View>
                <View style={styles.modifyCount}>
                    <Text style={styles.modifyCountTitle}>入库数量</Text>
                    <TouchableOpacity onPress={this.minusCount}>
                        <Text style={styles.imgSymbol}>－</Text>
                    </TouchableOpacity>
                    <TextInput
                        style={styles.countFont}
                        defaultValue={count + ''}
                        underlineColorAndroid='transparent'
                        onChangeText={(text) => this.setState({ count:text })}
                        keyboardType='numeric' />
                    <TouchableOpacity onPress={this.addCount}>
                        <Text style={styles.imgSymbol}>＋</Text>
                    </TouchableOpacity>
                </View>
                <View style={styles.bottom}>
                    <Button style={styles.btnConfirm} textStyle={styles.btnFont} onPress={this.showScanResult}>确认</Button>
                    <Button style={styles.btnCancel} textStyle={styles.btnFont} onPress={this.doCancel}>取消</Button>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    containerDetails:{
        height:150,
        backgroundColor:'#f8f8f8',
        flexDirection: 'row',
    },
    imageGoods:{
        height:125,
        width:125,
        marginLeft:12,
        marginTop:11,
    },
    goodsDetails:{
        marginLeft:8,
        height:125,
        paddingBottom:8,
        marginTop:10,
        justifyContent: 'center',
    },
    goodsDetailsTop:{
        width:sr.w - 99,
        flexDirection: 'row',
        justifyContent: 'space-between',
    },
    detailsTopLeft:{
        flexDirection: 'row',
        alignItems: 'center',
        flex:1,
    },
    detailsTopRight:{
        flexDirection: 'row',
        alignItems: 'center',
    },
    detailsAddress:{
        fontSize:16,
        width:100,
        color:'#333333',
        flexDirection: 'row',
    },
    moneyLeft:{
        fontSize:14,
        color:'#333333',
    },
    moneyRight:{
        fontSize:15,
        fontWeight:'500',
        color:'#f01114',
        marginRight:10,
    },
    textDetailsItem:{
        marginTop:2,
        fontSize:14,
        color:'#888888',
        flex:1,
    },
    modifyCount:{
        flexDirection:'row',
        backgroundColor:'#f4f4f4',
        height:50,
        marginTop:10,
        alignItems:'center',
    },
    modifyCountTitle:{
        marginLeft:20,
        color:'#888888',
        marginRight:130,
    },
    imgSymbol:{
        marginRight:10,
        fontSize:20,
    },
    countFont:{
        padding:0,
        fontSize:14,
        width:30,
        height:20,
        color:'#888888',
        backgroundColor:'#FFFFFF',
        marginRight:10,
        alignSelf:'center',
        textAlign:'right',
    },
    btnConfirm:{
        flex:1,
        height:45,
        borderRadius:0,
        backgroundColor:'#FE9917',
    },
    btnCancel:{
        flex:1,
        height:45,
        borderRadius:0,
        backgroundColor:'#c81622',
    },
    btnFont:{
        color:'#FFFFFF',
        fontWeight:'400',
    },
    bottom:{
        height:45,
        width:sr.w,
        flexDirection:'row',
        position:'absolute',
        bottom:0,
        right:0,
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/cargo/ScanResult.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
} = ReactNative;

const { Button,DImage } = COMPONENTS;
import moment from 'moment';
const ScanBarCode = require('../../common/ScanBarCode');

module.exports = React.createClass({
    statics: {
        title: '扫描结果',
    },
    componentWillMount () {
        const { orderId, count } = this.props;
        this.postScanOrder( orderId, count );
    },
    postScanOrder ( orderId, count) {
        const param = {
            userId:app.personal.info.userId,
            orderId,
            count,
        };
        POST(app.route.ROUTE_PLACE_STORAGE, param, this.postScanOrderSuccess, true);
    },
    postScanOrderSuccess (data) {
        if (data.success) {
            this.setState({
                data:data.order,
            });
        } else {
            Toast(data.msg);
        }
    },
    getInitialState () {
        return {
            data:{},
        };
    },
    finishScanOrder () {
        Toast('扫描结束');
        app.navigator.popToTop();
    },
    onBarCodeRead (data) {
        if (!data || data.type != CONSTANTS.QRCODE_TYPE_ORDER) { // type==1代表货单二维码
            Toast('无效的二维码');
        } else {
            app.navigator.replace({
                component: require('./ScanOrderDetail'),
                passProps: {
                    orderId:data.id,
                },
            });
        }
    },
    continueScanOrder () {
        app.showModal(
            <ScanBarCode onBarCodeRead={this.onBarCodeRead} />
        );
    },
    render () {
        const { data } = this.state;
        return (
            <View style={styles.container}>
            <View style={styles.order}>
                <DImage
                    resizeMode='cover'
                    defaultSource={app.img.common_default}
                    source={{ uri:data.photo }}
                    style={styles.icon} />
                <View style={styles.rowInfo}>
                    <Text style={styles.id}
                        numberOfLines={1}
                        ellipsizeMode='tail'>货单号:{data.id}</Text>
                    <Text style={styles.info}>重量:{app.utils.N(data.weight)}吨</Text>
                    <Text style={styles.info}>方量:{app.utils.N(data.size)}m³</Text>
                    <Text style={styles.info}>收货地址:{data.endPoint}</Text>
                </View>
                <Text style={styles.timeInfo}>{moment(data.createTime).format('YYYY-MM-DD')}</Text>
            </View>
                <View style={styles.bottom}>
                    <Button style={styles.btnEnd} textStyle={styles.btnFont} onPress={this.finishScanOrder}>结束扫描</Button>
                    <Button style={styles.btnContinue} textStyle={styles.btnFont} onPress={this.continueScanOrder}>继续扫描</Button>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    top:{
        flexDirection:'row',
        height:60,
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent:'center',
    },
    order:{
        flexDirection:'row',
        height:90,
        backgroundColor:'#FFFFFF',
        marginTop:10,
        alignItems:'center',
    },
    row:{
        flexDirection:'row',
        height:90,
        backgroundColor:'#FFFFFF',
        marginTop:10,
        alignItems:'center',
    },
    icon:{
        width:70,
        height:70,
        borderRadius:2,
        marginLeft:8,
    },
    rowInfo:{
        marginTop:10,
        marginBottom:10,
        marginLeft:5,
    },
    id:{
        fontSize:14,
        color:'#333333',
        width:150,
        flex:1,
    },
    info:{
        fontSize:12,
        color:'#888888',
        marginTop:1,
        flex:1,
    },
    timeInfo:{
        fontSize:12,
        width:80,
        color:'#888888',
        alignSelf:'flex-start',
        marginTop:12,
        marginLeft:30,
    },
    btnContinue:{
        flex:1,
        height:45,
        borderRadius:0,
        backgroundColor:'#FE9917',
    },
    btnEnd:{
        flex:1,
        height:45,
        borderRadius:0,
        backgroundColor:'#c81622',
    },
    btnFont:{
        color:'#FFFFFF',
        fontWeight:'400',
    },
    bottom:{
        marginTop:sr.h/3,
        height:45,
        flexDirection:'row',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/cargo/SelectCarType.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button } = COMPONENTS;
const carInfo = [
    {
        name:'车长',
        array:['4.5米', '5米', '5.5米', '6.5米', '7.5米', '8.5米', '9.6米', '13.5米', '17.5米'],
    },
    {
        name:'车型',
        array:['其他', '高栏', '平板', '箱式', '高低板', '保温'],
    },
];
module.exports = React.createClass({
    getInitialState () {
        return {
            lengthName:'4.5米',
            typeName:'不限',
            selectArray : [0, 0],
        };
    },
    toClose () {
        app.pop();
    },
    toChoose (i, index, obj) {
        const { selectArray } = this.state;
        if (i == 0) {
            selectArray[i] = index;
            this.setState({
                lengthName:obj,
            });
        } else {
            selectArray[i] = index;
            this.setState({
                typeName:obj,
            });
        }
    },
    toSubmit (lengthName, typeName) {
        const { setCarInfo, doClose } = this.props;
        setCarInfo(lengthName, typeName);
        doClose();
    },
    render () {
        const { selectArray, lengthName, typeName } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.containerTop}>
                    <Text style={styles.textTop}>车长车型</Text>
                    <TouchableOpacity onPress={this.props.doClose}>
                        <Image
                            resizeMode='stretch'
                            source={app.img.cargo_cancel}
                            style={styles.closeStyle} />
                    </TouchableOpacity>
                </View>
                <View style={styles.listView} />
                {
                    carInfo.map((item, i) => (
                        <View style={styles.containerMiddle} key={i}>
                            <Text style={styles.rowTitle}>{item.name}</Text>
                            <View style={styles.row}>
                                {
                                    item.array.map((obj, index) => (
                                        <TouchableOpacity
                                            key={index}
                                            onPress={this.toChoose.bind(null, i, index, obj)}
                                            style={selectArray[i] == index ? styles.chooseItem : styles.item}
                                            >
                                            <Text style={styles.itemFont}>{obj}</Text>
                                        </TouchableOpacity>
                                    ))
                                }
                            </View>
                        </View>
                        ))
                    }
                <Button style={styles.btn} textStyle={styles.btnFont} onPress={this.toSubmit.bind(null, lengthName, typeName)}>确认</Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        height:350,
        backgroundColor:'#FFFFFF',
    },
    containerTop:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',

    },
    textTop:{
        fontSize:17,
        marginLeft:(sr.w - 17 * 4) / 2,
        color:'#333333',
    },
    closeStyle:{
        height:15,
        width:15,
        marginRight:10,
    },
    containerMiddle:{
        marginTop:1,
        backgroundColor:'#ffffff',
    },
    rowTitle:{
        fontSize:13,
        marginLeft:10,
    },
    row:{
        flexDirection:'row',
        marginLeft:5,
        flexWrap:'wrap',
        width:sr.w,
    },
    item:{
        width:50,
        height:20,
        backgroundColor:'#f4f4f4',
        margin:5,
        alignItems:'center',
        justifyContent:'center',
    },
    chooseItem:{
        width:50,
        height:20,
        backgroundColor:'#FE9917',
        margin:5,
        alignItems:'center',
        justifyContent:'center',
    },
    itemFont:{
        fontSize:12,
    },
    rowTitle1:{
        fontSize:13,
        marginLeft:10,
        marginTop:19,
    },
    btn:{
        height:45,
        width:351,
        backgroundColor:'#c81622',
        borderRadius:4,
        alignSelf:'center',
        marginTop:50,
    },
    btnFont:{
        color:'#FFFFFF',
    },
    listView:{
        height:1,
        backgroundColor:'#f4f4f4',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/cargo/index.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

import _ from 'lodash';

const { PageList, Button } = COMPONENTS;

const BranchList = require('./BranchList');
const OrderDetail = require('../../common/SendOrderDetail');
const ScanOrderDetail = require('./ScanOrderDetail');
const ScanBarCode = require('../../common/ScanBarCode');
const PayInfo = require('../../common/PayInfoModel');
const PayPassword = require('../../common/PayPassword');

let sumWeight = 0;
let sumCount = 0;
let sumSize = 0;

module.exports = React.createClass({
    getInitialState () {
        return {
            list:[],
            remainBondAmount:0,
            remainAmount:app.personal.remainAmount,
        };
    },
    componentWillMount () {
        this.updateNavBar();
        this.getBranchShopInfo();
        this.getRemainAmount();
    },
    onWillFocus () {
        this.updateNavBar();
        this.getBranchShopInfo();
        this.getRemainAmount();
    },
    updateNavBar () {
        app.getCurrentRoute().title = '竞得货物';
        app.getCurrentRoute().rightButton = { title: '扫描收货', handler: () => { this.toScanStorage(); } };
        app.getCurrentRoute().leftButton = { title: app.personal.currentShopName, handler: () => { app.scene.toShop&&app.scene.toShop(); }, isLimitLength:true };
        app.forceUpdateNavbar();
    },
    getRemainAmount () {
        const param = {
            userId: app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_REMAIN_AMOUNT, param, this.getRemainAmountSuccess, false);
    },
    getRemainAmountSuccess (data) {
        if (data.success) {
            const context = data.context;
            app.personal.setRemainAmount(context.amount);
            this.setState({ remainAmount:app.personal.remainAmount });
        }
    },
    getBranchShopInfo () {
        const param = {
            userId : app.personal.info.userId,
            shopId : app.personal.currentShopId,
        };
        POST(app.route.ROUTE_GET_BRANCH_SHOP_INFO, param, this.getBranchShopInfoSuccess, false);
    },
    getBranchShopInfoSuccess (data) {
        if (data.success) {
            this.setState({ remainBondAmount : data.context.remainBondAmount });
        }
    },
    onBarCodeRead (data) {
        if (!data || data.type != CONSTANTS.QRCODE_TYPE_ORDER) { // type==1代表货单二维码
            Toast('无效的二维码');
        } else {
            this.showScanOrderDetail(data.id);
        }
    },
    showScanOrderDetail (orderId) {
        app.push({
            component:ScanOrderDetail,
            passProps:{
                orderId,
            },
        });
    },
    toShop () {
        app.push({
            component:BranchList,
        });
    },
    toScanStorage () {
        app.showModal(<ScanBarCode onBarCodeRead={this.onBarCodeRead.bind( null )} />);
    },
    doChoose (obj, rowId) {
        const { list } = this.state;
        list[rowId].selected = !list[rowId].selected;
        this.setState({
            list,
        });
        this.doTotalCalculation();
    },
    toDetail (obj) {
        app.push({
            component:OrderDetail,
            passProps:{
                data : obj,
            }
        });
    },
    doSelectedAll () {
        let { list } = this.state;
        if (_.every(list, o => o.selected == true)) {
            _.forEach(list, o => { o.selected = false; });
        } else {
            _.forEach(list, o => { o.selected = true; });
        }
        this.setState({ list });
        this.doTotalCalculation();
    },
    doTotalCalculation () {
        const { list } = this.state;
        sumWeight = 0;
        sumCount = 0;
        sumSize = 0;
        _.filter(list, (o) => o.selected == true).forEach((item, i) => {
            sumSize = sumSize + item.item.size;
            sumCount = sumCount + item.item.totalNumbers;
            sumWeight = sumWeight + item.item.weight;
        });
    },
    refreshList () {
        this.listView.refresh();
    },
    renderRow (obj, rowId) {
        const { list } = this.state;
        if (!_.find(list, (o) => o.index == rowId)) {
            list.push({ selected:false, index:rowId, item:obj });
        }
        return (
            <View style={styles.itemContainer}>
                <TouchableOpacity style={styles.btnChoose}
                    onPress={this.doChoose.bind(null, obj, rowId)}>
                    <Image style={styles.choose}
                        source={!list[rowId].selected ? app.img.order_no_choose : app.img.order_choose} />
                </TouchableOpacity>
                <TouchableOpacity onPress={this.toDetail.bind( null, obj )}
                    style={styles.detail}>
                    <View style={styles.rightContainer}>
                        <View style={styles.pointContainer}>
                            <Text style={styles.itemAddress}>{app.personal.currentShopName}</Text>
                            <Image style={styles.point} source={app.img.cargo_point} />
                            <Text style={styles.itemAddress}>{obj.endPoint}</Text>
                        </View>
                        <View style={styles.pointContainerTop}>
                            <Text style={styles.textLeft}>总重量：</Text>
                            <Text style={styles.textRight}>{app.utils.N(obj.weight)}吨</Text>
                        </View>
                        <View style={styles.itemNumber}>
                            <Text style={styles.textLeft}>总方量：</Text>
                            <Text style={styles.textRight}>{app.utils.N(obj.size)}m³</Text>
                        </View>
                        <View style={styles.itemNumberBottom}>
                            <Text style={styles.textLeft}>总件数：</Text>
                            <Text style={styles.textRight}>{app.utils.N(obj.totalNumbers)}件</Text>
                        </View>
                    </View>
                </TouchableOpacity>
            </View>
        );
    },
    render () {
        const { list, remainAmount, remainBondAmount } = this.state;
        const status = list.length != 0 && _.every(list, (o) => o.selected == true);
        return (
            <View style={styles.container}>
                {
                    (_.intersection(app.personal.info.authority, [AH.AH_LOOK_AMOUNT, AH.AH_RECHARGE, AH.AH_WITHDRAW]).length != 0) &&
                    <Image style={styles.balance}
                        source={app.img.cargo_balance_background}>
                        <Text style={styles.balanceText}>余额：
                            {remainAmount}</Text>
                        <Text style={styles.balanceTextRight}>担保余额：{app.utils.N(remainBondAmount)}</Text>
                    </Image>

                }
                <View style={styles.listView}>
                        <PageList
                            ref={(ref) => { this.listView = ref; }}
                            renderRow={this.renderRow}
                            listName={'tostore.list'}
                            listParam={{ userId: app.personal.info.userId, type : 'tostore' }}
                            listUrl={app.route.ROUTE_GET_CITY_DISTRIBUTE_ORDERS}
                            refreshEnable
                            />
                        <View style={styles.selectedAllContainer}>
                            <TouchableOpacity onPress={this.doSelectedAll}
                                style={styles.selectedAllBtn}>
                                <Image style={styles.chooseAll}
                                    source={status ? app.img.order_choose : app.img.order_no_choose} />
                                <Text style={styles.selectedAllText}>全选</Text>
                            </TouchableOpacity>
                            <Text style={styles.totalText}>总计</Text>
                            <Text style={styles.totalItem}>{app.utils.N(sumWeight)}吨</Text>
                            <Text style={styles.totalItem}>{app.utils.N(sumSize)}m³</Text>
                            <Text style={styles.totalItem}>{app.utils.N(sumCount)}件</Text>
                        </View>
                    </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    listView:{
        flex:1,
        marginTop:10,
    },
    balance:{
        width:300,
        height:30,
        marginLeft:(sr.w - 300) / 2,
        flexDirection: 'row',
        justifyContent:'center',
        alignItems:'center',
    },
    balanceText:{
        fontSize:14,
        color:'#f69838',
        marginRight:5,
    },
    balanceTextRight:{
        fontSize:12,
        color:'#f69838',
        marginLeft:5,
    },
    itemContainer:{
        height:85,
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    btnChoose:{
        height:85,
        justifyContent:'center',
        alignItems:'center',
        width:35,
    },
    choose:{
        height:12,
        width:12,
    },
    detail:{
        justifyContent:'center',
        width:sr.w - 35,
    },
    rightContainer:{
        justifyContent:'center',
    },
    pointContainer:{
        flexDirection: 'row',
        alignItems:'center',
    },
    pointContainerTop:{
        flexDirection: 'row',
        alignItems:'center',
        marginTop:2,
    },
    itemNumber:{
        flexDirection: 'row',
        alignItems:'center',
        marginTop:1,
    },
    itemNumberBottom:{
        flexDirection: 'row',
        alignItems:'center',
        marginTop:1,
    },
    point:{
        width:35,
        height:5,
        marginLeft:5,
        marginRight:5,
    },
    itemAddress:{
        marginTop:2,
        fontSize:14,
        color:'#333333',
    },
    textLeft:{
        fontSize:12,
        color:'#888888',
    },
    textRight:{
        fontSize:12,
        color:'#333333',
    },
    selectedAllContainer:{
        height:50,
        width:sr.w,
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
        alignItems:'center',
    },
    selectedAllBtn:{
        flexDirection:'row',
        alignItems:'center',
    },
    chooseAll:{
        height:9,
        width:9,
        marginLeft:10,
    },
    selectedAllText:{
        color:'#333333',
        fontSize:12,
        marginLeft:10,
    },
    totalText:{
        color:'#333333',
        fontSize:15,
        fontWeight:'500',
        marginLeft:30,
    },
    totalItem:{
        fontSize:12,
        color:'#f01823',
        marginLeft:20,
    },
    item: {
        padding:10,
        backgroundColor:'#FFFFFF',
    },
    row:{
        flexDirection:'row',
        marginTop:10,
        alignItems:'center',
    },
    icon:{
        width:70,
        height:70,
        borderRadius:2,
        marginLeft:8,
    },
    rowInfo:{
        marginTop:10,
        marginBottom:10,
        marginLeft:5,
    },
    id:{
        fontSize:14,
        color:'#333333',
        flex:1,
    },
    info:{
        fontSize:12,
        color:'#888888',
        flex:1,
    },
    bottomView:{
        position:'absolute',
        top:200,
        width:sr.w,
    },
    bottomInfo:{
        flexDirection: 'row',
        backgroundColor:'white',
    },
    infoLeft:{
        height:45,
        flexDirection: 'row',
        flex:1,
        justifyContent:'center',
        alignItems: 'center',
    },
    moneyLeft:{
        fontSize:12,
        color:'#333333',
    },
    moneyRight:{
        fontSize:13,
        fontWeight:'500',
        color:'#f01114',
        marginRight:10,
    },
    btnPayNow:{
        height:45,
        flex:1,
        borderRadius:0,
    },
    btnPayNowText:{
        fontSize:16,
        fontWeight:'500',
    },
    orderInfo:{
        height:480,
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/order/OrderDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button, CustomSheet, DImage } = COMPONENTS;
const LogisticsDetail = require('../../common/LogisticsDetail');
const PayPassword = require('../../common/PayPassword');

module.exports = React.createClass({
    statics: {
        title: '货单详情',
    },
    getInitialState () {
        return {
            choose:false,
        };
    },
    doChoose () {
        this.setState({
            choose:!this.state.choose,
        });
    },
    doLookTransport () {
        app.push({
            component: LogisticsDetail,
            passProps:{
                obj: this.props.data,
            },
        });
    },
    toPayMode (payMode) {
        if (payMode == 0) {
            return '现付';
        } else if (payMode == 1) {
            return '到付';
        } else {
            return '混合付款';
        }
    },
    render () {
        const { data } = this.props;
        const { stateList } = data;
        return (
            <View style={styles.container}>
            <View style={styles.containerMarket}>
                <DImage style={styles.marketLogo}
                    source={app.img.order_logo}
                    resizeMode='stretch'
                    />
                <Text style={styles.textMarket}>{ data.shop ? data.shop.name : data.agent ? data.agent.name : ''}</Text>
                <Text style={styles.orderState}>{ stateList ? OS.MAP[stateList[0].state] : ''}</Text>
            </View>
                <View style={styles.containerDetails}>
                    <DImage style={styles.imageGoods}
                        resizeMode='stretch'
                        defaultSource={app.img.common_default}
                        source={{ uri:data.photo }}
                        />
                    <View style={styles.goodsDetails}>
                        <Text style={styles.textDetailsItem}>收货人：{!data.receiverName ? '暂无' : data.receiverName}</Text>
                        <Text style={styles.textDetailsItem}>收货人电话:{!data.receiverPhone ? '暂无' : data.receiverPhone}</Text>
                        <Text style={styles.textDetailsItem}>到货地：{data && data.endPoint}</Text>
                        <Text style={styles.textDetailsItem}>方量：{data.size}m³ 重量：{data.weight}吨 件数：{data.totalNumbers}件 </Text>
                        <Text style={styles.textDetailsItem}>下单时间：{data.createTime} </Text>
                    </View>
                </View>
                <View style={styles.containerWhite} />
                <View style={styles.containerInfo}>
                    <Text style={styles.textInfoTop}>支付方式：{this.toPayMode(data.payMode)}</Text>
                        <Text style={styles.textInfo}>运费总额：¥{app.utils.N(data.payMode == 0 ? data.needPayTransportFee :
                            data.payMode == 1 ?  data.totalDesignatedFee :  data.totalDesignatedFee + data.needPayTransportFee ) + app.utils.N(data.additionalFee)}(实际所得运费：¥{ data.additionalFee + data.fee })</Text>
                    <Text style={styles.textInfo}>代收货款：¥{app.utils.N(data.proxyCharge)}</Text>
                    {
                        data.payMode != 0 &&
                        <View style={styles.payInfo}>
                            <Text style={styles.textTotal}>需收款:</Text>
                            <Text style={styles.payInfoText}>¥{app.utils.N(data.totalDesignatedFee + data.additionalFee +data.proxyCharge)}</Text>
                        </View>

                    }
                </View>
                {
                    stateList[0].state >= OS.OS_ON_THE_WAY &&
                    <TouchableOpacity
                        onPress={this.doLookTransport}
                        style={styles.transportInfo}>
                        <Text style={styles.textTransportInfo}>物流信息</Text>
                        <View style={styles.transportInfoRight}>
                            <Text style={styles.textTransportInfoRight}>{stateList ? OS.MAP[stateList[0].state] : ''}</Text>
                            <Image
                                style={styles.imageCommon_go}
                                resizeMode='stretch'
                                source={app.img.common_go}
                                />
                        </View>
                    </TouchableOpacity>
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    containerTop:{
        height:60,
        backgroundColor:'#c81622',
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'center',
    },
    imageTop:{
        height:32,
        width:27,
    },
    textTop:{
        fontSize:12,
        fontWeight:'900',
        marginLeft:20,
        color:'#ffffff',
    },
    containerReceive:{
        marginTop:8,
        paddingLeft:10,
        height:55,
        backgroundColor:'#ffffff',
        justifyContent: 'center',
    },
    textAddress:{
        fontSize:12,
        marginBottom:3,
        color:'#333333',
        flex:1,
    },
    textName:{
        fontSize:12,
        color:'#333333',
        flex:1,
    },
    containerMarket:{
        height:25,
        marginTop:8,
        alignItems: 'center',
        justifyContent: 'space-between',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    containerMarketLeft:{
        flexDirection: 'row',
    },
    marketLogo:{
        height:13,
        width:16.5,
        marginLeft:10,
    },
    textMarket:{
        flex:1,
        marginLeft:10,
        color:'#333333',
        fontSize:12,
    },
    orderState:{
        color:'#00a117',
        marginRight:10,
        fontSize:12,
    },
    containerDetails:{
        height:105,
        flexDirection: 'row',
    },
    containerWhite:{
        height:8,
        backgroundColor:'#ffffff',
    },
    imageGoods:{
        height:75,
        width:75,
        marginLeft:12,
        marginTop:11,
    },
    payInfo:{
        flexDirection:'row',
    },
    payInfoText:{
        marginTop:3,
        fontSize:12,
        color:'#f01114',
        fontWeight:'400',
        height:15,
    },
    goodsDetails:{
        marginLeft:8,
        height:95,
        paddingVertical:8,
        justifyContent: 'center',
    },
    goodsDetailsTop:{
        width:sr.w - 99,
        flexDirection: 'row',
        justifyContent: 'space-between',
    },
    detailsTopLeft:{
        flexDirection: 'row',
        alignItems: 'center',
        marginTop:15,
    },
    detailsTopRight:{
        flexDirection: 'row',
        alignItems: 'center',
        marginTop:15,
    },
    detailsAddress:{
        fontSize:14,
        width:80,
        color:'#333333',
        flexDirection: 'row',

    },
    moneyLeft:{
        fontSize:12,
        color:'#333333',
    },
    moneyRight:{
        fontSize:13,
        fontWeight:'500',
        color:'#f01114',
        marginRight:10,
    },
    textDetailsItem:{
        marginTop:1,
        fontSize:12,
        color:'#888888',
    },
    containerOrderNumber:{
        height:50,
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingLeft:10,
        justifyContent: 'center',
    },
    textOrderNumber:{
        color:'#333333',
        marginBottom:3,
        fontSize:12,
    },
    textDate:{
        color:'#888888',
        marginTop:3,
        fontSize:14,
    },
    containerInfo:{
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingLeft:10,
        paddingBottom:5,
        paddingTop:5,
    },
    textInfoTop:{
        color:'#888888',
        fontSize:12,
        height:15,
    },
    textInfo:{
        color:'#888888',
        fontSize:12,
        marginTop:3,
        height:15,
    },
    textTotal:{
        marginTop:3,
        fontSize:12,
        color:'#333333',
        fontWeight:'400',
        height:15,
    },
    transportInfo:{
        height:40,
        backgroundColor:'#ffffff',
        marginTop:8,
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textTransportInfo:{
        color:'#888888',
        fontSize:14,
        marginLeft:10,
    },
    transportInfoRight:{
        marginRight:10,
        flexDirection: 'row',
        alignItems: 'center',
    },
    textTransportInfoRight:{
        fontSize:14,
        color:'#333333',
        marginRight:5,
    },
    imageCommon_go:{
        width: 8,
        height: 15,
        marginRight: 7,
    },
    choose:{
        marginLeft:10,
        flexDirection: 'row',
        alignItems: 'center',
    },
    imageChoose:{
        height:15,
        width:15,
    },
    bottomView:{
        position:'absolute',
        bottom:0,
        width:sr.w,
    },
    bottomInfo:{
        height:45,
        flexDirection: 'row',
        backgroundColor:'white',
        marginBottom:0,
    },
    infoLeft:{
        height:45,
        flexDirection: 'row',
        flex:1,
        alignItems: 'center',
    },
    btnPayNow:{
        height:45,
        flex:1,
        borderRadius:0,
    },
    btnPayNowText:{
        fontSize:16,
        fontWeight:'500',
    },
    totalLeft:{
        fontSize:12,
        color:'#333333',
        marginLeft:10,
    },
    totalRight:{
        fontSize:13,
        fontWeight:'500',
        color:'#f01114',
        marginLeft:5,
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/order/OrderList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableOpacity,
} = ReactNative;

const { PageList, DImage, Button, MessageBox } = COMPONENTS;
const OrderDetail = require('../cargo/OrderDetail');
const LogisticsDetail = require('../../common/LogisticsDetail');
const VerificationCode = require('../../common/VerificationCode');

module.exports = React.createClass({
    renderSeparator (sectionID, rowID) {
        return (
            <View style={styles.separator} key={sectionID + rowID} />
        );
    },
    getOrderDetail(obj,rowID){
        const { type } = this.props;
        app.push({
            component: type == 'toconfirm' ? OrderDetail : require('./OrderDetail'),
            passProps:{
                orderId: obj.id,
                data: obj,
                refresh : this.listView.refresh,
            },
        });
    },
    toVerificationCode(obj,rowID){
        app.push({
            component: VerificationCode,
            passProps: {
                phone: obj.receiverPhone,
                orderId: obj.id,
            },
        });
    },
    doLookTransport(obj,rowID){
        app.push({
            component:LogisticsDetail,
            passProps:{
                obj,
            },
        });
    },
    doConfirmReceiveMoney (obj, index) {
        app.showModal(
            <MessageBox onConfirm={this.clickConfirmReceiveMoney.bind(null, obj, index)} content={'是否确定收款'} onCancel title={false} width={sr.s(250)} />
        );
    },
    clickConfirmReceiveMoney (obj, index) {
        const param = {
            userId: app.personal.info.userId,
            orderId: obj.id,
        };
        POST(app.route.ROUTE_FINISH_ORDER, param, this.doConfirmSuccess.bind(null, index), true);
    },
    doConfirmSuccess (index, data) {
        const { success, context, msg } = data;
        if (success) {
            this.listView.updateList((list) => {
                list[index] = context.order || {};
                return list;
            });
            this.listView.refresh();
            Toast('确认成功');
        } else {
            Toast(msg);
        }
    },
    renderRow (obj, rowID) {
        const { stateList } = obj;
        const { type,data } = this.props;
        return (
            <View style={styles.row}>
                <View style={styles.rowTopContainer}>
                    <DImage
                        resizeMode='stretch'
                        source={app.img.order_logo}
                        style={styles.topImage} />
                    <Text style={styles.topLeftText}>{obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : ''}</Text>
                    <Text style={styles.topRightText}>{stateList ? OS.MAP[stateList[0].state] : ''}</Text>
                </View>
                <View style={styles.middle}>
                    <TouchableOpacity style={styles.rowMiddleContainer} onPress={this.getOrderDetail.bind(null, obj, rowID)}>
                        <DImage
                            resizeMode='stretch'
                            defaultSource={app.img.common_default}
                            source={{ uri: obj.photo }}
                            style={styles.middleImage} />
                        <View style={styles.middleRight}>
                            <View style={styles.pointContainer}>
                                <View style={styles.placeContainer}>
                                    <Text style={styles.pointName}
                                        numberOfLines={1}
                                        ellipsizeMode='tail'>{obj.shop ? obj.shop.address : obj.agent ? obj.agent.address : ''}</Text>
                                    <Text style={styles.pointText}>→</Text>
                                    <Text style={styles.pointName}
                                        numberOfLines={1}
                                        ellipsizeMode='tail'>{obj.endPoint}</Text>
                                </View>
                            </View>
                            <View style={styles.common}>
                                <Text style={styles.commonInfo}>重量：{app.utils.N(obj.weight)}吨</Text>
                                <Text style={styles.commonInfo}>方量：{app.utils.N(obj.size)}m³</Text>
                                <Text style={styles.commonInfo}>件数：{app.utils.N(obj.totalNumbers)}件</Text>
                            </View>
                            <View style={styles.common}>
                                <Text style={styles.commonInfo}>运费总额：¥{app.utils.N(obj.realFee + obj.additionalFee)}</Text>
                                <Text style={styles.commonInfo}>实际所得：￥{app.utils.N(obj.additionalFee + obj.fee)}</Text>
                            </View>
                            <View style={styles.common}>
                                <Text style={styles.commonInfo}>代收货款：￥{app.utils.N(obj.proxyCharge)}</Text>
                                {
                                    (type == 'waitpick' || type =='onway') &&
                                    <Text style={styles.commonInfo}>需收款：￥{app.utils.N(obj.totalDesignatedFee + obj.additionalFee + obj.proxyCharge)}</Text>
                                }
                            </View>
                            <Text style={styles.commonInfo}>下单时间：{obj.createTime}</Text>
                        </View>
                    </TouchableOpacity>
                </View>
                <View style={styles.rowButtomContainer}>
                    {
                        stateList && stateList[0].state == OS.OS_READY_RECEIVE && (obj.totalDesignatedFee + obj.proxyCharge + obj.additionalFee) > 0 &&
                        <Button onPress={this.doConfirmReceiveMoney.bind(null, obj, rowID)} style={styles.ConfirmReach} textStyle={styles.btnConfirmReachText}>{'确认收款'}</Button>
                    }
                    {
                        (obj.totalDesignatedFee + obj.proxyCharge + obj.additionalFee) == 0 && stateList[0].state == OS.OS_READY_RECEIVE &&
                        <Button
                            onPress={this.toVerificationCode.bind(null, obj, rowID)}
                            style={styles.lookDetail}
                            textStyle={styles.btnDetallText}>
                            {'代确认收货'}
                        </Button>
                    }
                    <Button
                        onPress={this.doLookTransport.bind(null, obj)}
                        style={styles.lookTransport}
                        textStyle={styles.btnTransportText}>
                        {'查看物流'}
                    </Button>
                </View>
            </View>
        );
    },
    render () {
        const { index, tabIndex, type, data, listUrl } = this.props;
        return (
            <View style={index === tabIndex ? styles.container : { left:-sr.tw, top:0, position:'absolute' }}>
                <View style={styles.separator} />
                {
                    !!data &&
                    <PageList
                        ref={(ref) => { this.listView = ref; }}
                        renderRow={this.renderRow}
                        renderSeparator={this.renderSeparator}
                        listParam={{ userId: app.personal.info.userId, type: type }}
                        listName={type + '.list'}
                        list={data.list}
                        autoLoad={false}
                        listUrl={listUrl}
                        refreshEnable
                        />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f4f4f4',
    },
    row: {
        paddingVertical: 2,
        paddingHorizontal: 2,
    },
    separator: {
        backgroundColor: '#f4f4f4',
        height: 8,
    },
    rowTopContainer: {
        paddingHorizontal:10,
        paddingVertical:5,
        backgroundColor:'white',
        flexDirection: 'row',
    },
    topImage: {
        width: 17,
        height:17,
    },
    topLeftText: {
        marginLeft:10,
        flex:1,
        fontSize: 14,
    },
    topRightText: {
        flex:1,
        fontSize: 14,
        textAlign:'right',
        color:'#f51b1a',
    },
    middle:{
        flexDirection:'row',
        height:100,
        alignItems:'center',
        width:sr.w,
        backgroundColor:'#f8f8f8',
    },
    rowMiddleContainer: {
        flexDirection: 'row',
        alignItems: 'center',
        paddingVertical:11,
        height:100,
    },
    btnChoose:{
        height:100,
        width:20,
        marginRight:3,
        justifyContent:'center',
        alignItems:'center',
    },
    choose:{
        height:12,
        width:12,
    },
    middleImage: {
        width: 80,
        height:80,
        borderRadius: 10,
    },
    middleRight: {
        paddingLeft: 10,
        width:sr.w - 105,
    },
    money: {
        fontSize: 12,
        color:'#f51b1a',
    },
    common:{
        flexDirection:'row',
    },
    commonInfo: {
        fontSize: 13,
        color:'#888888',
        marginHorizontal:5,
    },
    pointName: {
        fontSize: 13,
        maxWidth: 130,
    },
    pointText:{
        marginLeft:5,
        marginRight:10,
    },
    transformFee: {
        fontSize: 12,
        color: 'gray',
        marginRight:10,
        textAlign:'right',
    },
    pointContainer: {
        flexDirection: 'row',
        marginBottom:4,
    },
    placeContainer: {
        flex:2,
        flexDirection: 'row',
    },
    rowButtomContainer: {
        backgroundColor:'white',
        flexDirection: 'row',
        justifyContent: 'flex-end',
        paddingVertical:5,
    },
    lookTransport: {
        height:20,
        width:73,
        borderRadius:5,
        borderColor:'#ff6200',
        borderWidth:1,
        backgroundColor:'white',
        marginRight:5,
    },
    lookDetail: {
        height:20,
        width:80,
        borderRadius:5,
        borderColor:'#007ef7',
        borderWidth:1,
        marginLeft:20,
        marginRight:10,
        backgroundColor:'white',
    },
    btnTransportText:{
        color:'#ff6200',
        fontSize:12,
        fontWeight:'300',
    },
    btnDetallText:{
        color:'#007ef7',
        fontSize:12,
        fontWeight:'300',
    },
    totalLeft:{
        flex:7,
        flexDirection:'row',
        alignItems:'center',
    },
    selectedAllContainer:{
        height:50,
        width:sr.w,
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
        alignItems:'center',
    },
    selectedAllBtn:{
        flexDirection:'row',
        alignItems:'center',
        height:40,
    },
    chooseAll:{
        height:12,
        width:12,
        marginLeft:5,
    },
    selectedAllText:{
        color:'#333333',
        fontSize:12,
        marginLeft:10,
    },
    totalText:{
        color:'#333333',
        fontSize:15,
        fontWeight:'500',
        marginLeft:20,
    },
    totalItem:{
        width:45,
        fontSize:12,
        color:'#f01823',
        textAlign:'right',
        marginLeft:5,
    },
    totalItemRight:{
        width:50,
        fontSize:12,
        color:'#f01823',
        textAlign:'right',
        marginRight:5,
    },
    pay:{
        flex:3,
        marginLeft:10,
        height:50,
        width:150,
        justifyContent:'center',
        alignItems:'center',
        backgroundColor:'#c81622',
    },
    payText:{
        fontSize:14,
        fontWeight:'500',
        color:'#FFFFFF',
        letterSpacing:2,
    },
    btnConfirmReachText:{
        color:'#ff6200',
        fontSize:12,
        fontWeight:'300',
    },
    ConfirmReach: {
        height:20,
        width:73,
        borderRadius:5,
        borderColor:'#ff6200',
        borderWidth:1,
        marginRight:5,
        backgroundColor:'white',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/order/Orders.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

const OrderList = require('./OrderList');
const BranchList = require('../cargo/BranchList');
const ScanBarCode = require('../../common/ScanBarCode');
const ScanOrderDetail = require('./ScanOrderDetail');
module.exports = React.createClass({
    getInitialState () {
        return {
            tabIndex: 0,
            context:[],
        };
    },
    onWillFocus () {
        app.getCurrentRoute().title = '我的货单';
        app.getCurrentRoute().rightButton = { title: '扫描装车', handler: () => { app.scene.toScan&&app.scene.toScan(); } };
        app.getCurrentRoute().leftButton = { title: app.personal.currentShopName, handler: () => { app.scene.toShop&&app.scene.toShop(); }, isLimitLength:true };
        app.forceUpdateNavbar();
    },
    componentWillMount(){
        this.getCityDistributeOrders();
    },
    getCityDistributeOrders(){
        const param = {
            userId:app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_CITY_DISTRIBUTE_ORDERS,param,this.getCityDistributeOrdersSuccess,false);
    },
    getCityDistributeOrdersSuccess(data){
        if (data.success) {
            this.setState({ context : data.context });
        }
    },
    onBarCodeRead (data) {
        if (!data || data.type != CONSTANTS.QRCODE_TYPE_ORDER) { // type==1代表货单二维码
            Toast('无效的二维码');
        } else {
            this.carLoading(data.id);
        }
    },
    toScan(){
        app.showModal(<ScanBarCode onBarCodeRead={this.onBarCodeRead.bind( null )} />);
    },
    carLoading(orderId){
        app.push({
            component:ScanOrderDetail,
            passProps:{
                orderId,
            },
        });
    },
    toShop () {
        app.push({
            component:BranchList,
            passProps:{
                onWillFocus:this.onWillFocus,
            },
        });
    },
    selectShop () {
        app.push({
            component:BranchList,
        });
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    render () {
        const menus = [{ name:'电话确认', type:'toconfirm' }, { name:'待装车', type:'toload' }, { name:'上门自提', type:'waitpick' }, { name:'运输中', type:'onway' }, { name:'完成', type:'success' }];
        const { tabIndex, context } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.tabContainer}>
                    {
                        menus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={this.changeTab.bind(null, i)}
                                style={styles.touchTab}>
                                <View style={styles.tabButton}>
                                    <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                        {item.name}
                                    </Text>
                                    <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                </View>
                            </TouchableHighlight>
                        ))
                    }
                </View>
                {
                    menus.map((item, i) => (
                        <OrderList key={i} index={i} type={item.type} tabIndex={tabIndex} data={context && context[item.type]} listUrl={app.route.ROUTE_GET_CITY_DISTRIBUTE_ORDERS} />
                    ))
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonCenter: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonRight: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        borderRadius: 9,
    },
    tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/order/ScanOrderDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button, DImage } = COMPONENTS;
import moment from 'moment';

module.exports = React.createClass({
    statics: {
        title: '扫描装车详情',
    },
    getInitialState () {
        return {
            data: [],
            unStoageNumbers: '',
            count : 0,
        };
    },
    componentWillMount(){
        this.getScanDetail();
    },
    getScanDetail(){
        const { count } = this.state;
        const param = {
            userId: app.personal.info.userId,
            orderId: this.props.orderId,
            count,
        };
        POST(app.route.ROUTE_SCAN_LOAD_ORDER_FOR_CITY_DISTRIBUTE, param, this.getScanDetailSuccess, false);
    },
    getScanDetailSuccess(data){
        if (data.success) {
            this.setState({
                data : data.context.latestOrder,
                unStoageNumbers : _.reduce(data.context.unloadAllOrderList, (result, item) => result+item.unloadNumber,0),
            });
        }else{
            Toast(data.msg);
        }
    },
    minusCount () {
        const { count } = this.state;
        if (count > 0) {
            this.setState({ count:count - 1 });
        } else {
            Toast('装车数量不能小于0');
        }
    },
    addCount () {
        const { count, data,unStoageNumbers } = this.state;
        if (count < unStoageNumbers) {
            this.setState({ count:count + 1 });
        } else {
            Toast('装车数量不能超过未装车数量');
        }
    },
    showScanResult () {
        const { count, data, unStoageNumbers } = this.state;
        if (count > 0 && count <= unStoageNumbers) {
            const param = {
                userId: app.personal.info.userId,
                orderId: data.id,
                count : count * 1,
            };
            POST(app.route.ROUTE_SCAN_LOAD_ORDER_FOR_CITY_DISTRIBUTE, param, this.doConfirmSuccess, true);
        } else {
            Toast('数量清点有误,请重新核实');
        }
    },
    doConfirmSuccess (data) {
        if (data.success) {
            Toast('装车成功');
            app.navigator.popToTop();
        } else {
            Toast(data.msg);
        }
    },
    doCancel () {
        app.navigator.popToTop();
    },
    render () {
        const { data, count, unStoageNumbers } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.containerDetails}>
                    <DImage style={styles.imageGoods}
                        resizeMode='stretch'
                        defaultSource={app.img.common_default}
                        source={{ uri:data.photo }}
                        />
                    <View style={styles.goodsDetails}>
                        <View style={styles.goodsDetailsTop}>
                            <View style={styles.detailsTopLeft}>
                                <Text style={styles.detailsAddress}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{app.personal.currentShopName}</Text>
                                <Text >-</Text>
                                <Text style={styles.detailsAddress}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{data.endPoint}</Text>
                            </View>
                        </View>
                        <Text style={styles.textDetailsItem}>装车重量：{app.utils.N(data.weight)}m³</Text>
                        <Text style={styles.textDetailsItem}>装车方量：{app.utils.N(data.size)}吨</Text>
                        <Text style={styles.textDetailsItem}>未装车货物数量：{app.utils.N(unStoageNumbers)}件</Text>
                        <Text style={styles.textDetailsItem}>下单时间：{moment(data.createTime).format('YYYY-MM-DD')}</Text>
                    </View>
                </View>
                <View style={styles.modifyCount}>
                    <Text style={styles.modifyCountTitle}>装车件数</Text>
                    <TouchableOpacity onPress={this.minusCount}>
                        <Text style={styles.imgSymbol}>－</Text>
                    </TouchableOpacity>
                    <TextInput
                        style={styles.countFont}
                        defaultValue={count + ''}
                        underlineColorAndroid='transparent'
                        onChangeText={(text) => this.setState({ count:text })}
                        keyboardType='numeric' />
                    <TouchableOpacity onPress={this.addCount}>
                        <Text style={styles.imgSymbol}>＋</Text>
                    </TouchableOpacity>
                </View>
                <View style={styles.bottom}>
                    <Button style={styles.btnConfirm} textStyle={styles.btnFont} onPress={this.showScanResult}>确认</Button>
                    <Button style={styles.btnCancel} textStyle={styles.btnFont} onPress={this.doCancel}>取消</Button>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    containerDetails:{
        height:150,
        backgroundColor:'#f8f8f8',
        flexDirection: 'row',
    },
    imageGoods:{
        height:125,
        width:125,
        marginLeft:12,
        marginTop:11,
    },
    goodsDetails:{
        marginLeft:8,
        height:125,
        paddingBottom:8,
        marginTop:10,
        justifyContent: 'center',
    },
    goodsDetailsTop:{
        width:sr.w - 99,
        flexDirection: 'row',
        justifyContent: 'space-between',
    },
    detailsTopLeft:{
        flexDirection: 'row',
        alignItems: 'center',
        flex:1,
    },
    detailsTopRight:{
        flexDirection: 'row',
        alignItems: 'center',
    },
    detailsAddress:{
        fontSize:16,
        width:100,
        color:'#333333',
        flexDirection: 'row',
    },
    moneyLeft:{
        fontSize:14,
        color:'#333333',
    },
    moneyRight:{
        fontSize:15,
        fontWeight:'500',
        color:'#f01114',
        marginRight:10,
    },
    textDetailsItem:{
        marginTop:2,
        fontSize:14,
        color:'#888888',
        flex:1,
    },
    modifyCount:{
        flexDirection:'row',
        backgroundColor:'#f4f4f4',
        height:50,
        marginTop:10,
        alignItems:'center',
    },
    modifyCountTitle:{
        marginLeft:20,
        color:'#888888',
        marginRight:130,
    },
    imgSymbol:{
        marginRight:10,
        fontSize:20,
    },
    countFont:{
        padding:0,
        fontSize:14,
        width:30,
        height:20,
        color:'#888888',
        backgroundColor:'#FFFFFF',
        marginRight:10,
        alignSelf:'center',
        textAlign:'right',
    },
    btnConfirm:{
        flex:1,
        height:45,
        borderRadius:0,
        backgroundColor:'#FE9917',
    },
    btnCancel:{
        flex:1,
        height:45,
        borderRadius:0,
        backgroundColor:'#c81622',
    },
    btnFont:{
        color:'#FFFFFF',
        fontWeight:'400',
    },
    bottom:{
        height:45,
        width:sr.w,
        flexDirection:'row',
        position:'absolute',
        bottom:0,
        right:0,
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/shortroadmap/AddRoadmap.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    Image,
    TouchableOpacity,
    ScrollView,
} = ReactNative;
const { Button, SelectShortAddress } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '发布路线',
    },
    getInitialState () {
        return {
            endPoint: '',
            endPointLastCode:'',
            status:false,
            price:0,
            minFee:0,
            basePrice:0,
            baseMinFee:0,
        };
    },
    showEndPlace () {
        const { endPointLastCode } = this.state;
        app.push({
            component: SelectShortAddress,
            title:'到货地',
            passProps:{
                parentCode : 520101,
                confirmAddress:(endPoint, endPointLastCode) => {
                    this.setState({ endPoint, endPointLastCode });
                },
            },
        });
    },
    publishRoadmap () {
        const { endPoint, endPointLastCode, price, minFee, basePrice, baseMinFee, status } = this.state;
        if (!endPoint || endPoint == '') {
            Toast('到货地不能为空');
            return;
        } else if (!price || price == '') {
            Toast('短途起价不能为空');
            return;
        } else if (!minFee || minFee == '') {
            Toast('短途单价不能为空');
            return;
        } else if (status) {
            if (!basePrice || basePrice == '') {
                Toast('短途起价保底不能为空');
                return;
            } else if (!baseMinFee || baseMinFee == '') {
                Toast('短途单价保底不能为空');
                return;
            }
        }
        const param = {
            userId:app.personal.info.userId,
            shopId:app.personal.currentShopId,
            endPoint,
            endPointLastCode,
            price,
            minFee,
            basePrice,
            baseMinFee,
            isCityDistribute:true,
        };
        POST(app.route.ROUTE_PUBLISH_ROADMAP, param, this.publishRoadmapSuccess, true);
    },
    publishRoadmapSuccess (data) {
        if (data.success) {
            Toast('发布路线成功');
            this.props.refreshList();
        } else {
            Toast(data.msg);
        }
    },
    joinRank () {
        this.setState({
            status:!this.state.status,
        });
    },
    render () {
        const { endPoint, status } = this.state;
        return (
            <ScrollView>
                <View style={styles.container}>
                    <View>
                        <View style={styles.itemStart}>
                            <Text style={styles.itemLeft}>始发地</Text>
                            <View style={styles.itemPoint}>
                                <Text style={styles.itemRightStart}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{(app.utils.getVisibleText(app.personal.currentShopAddress + app.personal.currentShopName, 50))}</Text>
                            </View>
                        </View>
                        <TouchableOpacity
                            activeOpacity={0.6}
                            onPress={this.showEndPlace}>
                            <View style={styles.item}>
                                <Text style={styles.itemLeft}>到货地</Text>
                                <View style={styles.itemPoint}>
                                    <Text style={styles.itemRight}
                                        numberOfLines={1}
                                        ellipsizeMode='tail'>{endPoint}</Text>
                                    <Image
                                        source={app.img.common_go}
                                        style={styles.goStyle} />
                                </View>
                            </View>
                        </TouchableOpacity>
                    </View>
                    <View style={styles.list}>
                        <View style={styles.item}>
                            <Text style={styles.itemLeft}>短途单价</Text>
                            <TextInput style={styles.roadInfo}
                                underlineColorAndroid='transparent'
                                keyboardType='numeric'
                                placeholder='请输入短途单价'
                                onChangeText={(text) => this.setState({ price: text })} />
                            <Text style={styles.itemEnd}>元/吨</Text>
                        </View>
                        <View style={styles.item}>
                            <Text style={styles.itemLeft}>短途起价</Text>
                            <TextInput style={styles.roadInfo}
                                underlineColorAndroid='transparent'
                                keyboardType='numeric'
                                placeholder='请输入短途起价'
                                onChangeText={(text) => this.setState({ minFee: text })} />
                            <Text style={styles.itemEnd}>元/票</Text>
                        </View>
                        <View style={styles.list}>

                            <TouchableOpacity onPress={this.joinRank}>
                                <View style={styles.item}>
                                    {
                                        status ? <Image style={styles.imgChoose} source={app.img.order_choose_r} /> : <Image style={styles.imgChoose} source={app.img.order_no_choose_r} />
                                }
                                    <Text style={styles.itemLeft}>智能排名保底价设置</Text>
                                </View>
                            </TouchableOpacity>

                            {
                            status ? <View>
                                <View style={styles.item}>
                                    <Text style={styles.itemLeft}>短途单价保底</Text>
                                    <TextInput style={styles.roadInfo}
                                        underlineColorAndroid='transparent'
                                        placeholder='请输入短途单价保底'
                                        keyboardType='numeric'
                                        onChangeText={(text) => this.setState({ basePrice: text })} />
                                    <Text style={styles.itemEnd}>元/吨</Text>
                                </View>
                                <View style={styles.item}>
                                    <Text style={styles.itemLeft}>短途起价保底</Text>
                                    <TextInput style={styles.roadInfo}
                                        underlineColorAndroid='transparent'
                                        placeholder='请输入短途起价保底'
                                        keyboardType='numeric'
                                        onChangeText={(text) => this.setState({ baseMinFee: text })} />
                                    <Text style={styles.itemEnd}>元/票</Text>
                                </View>
                            </View> : <Text />
                    }
                            <Button style={styles.btnpublishRoadmap}
                                textStyle={styles.btnpublishRoadmapFont}
                                onPress={this.publishRoadmap}>发布路线</Button>
                        </View>
                    </View>
                </View>
            </ScrollView>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    itemStart:{
        height:40,
        justifyContent: 'space-between',
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    itemLeft:{
        flex:1,
        paddingLeft:12,
        fontSize:14,
        color:'#333333',
    },
    itemPoint:{
        flexDirection: 'row',
    },
    itemRightStart:{
        fontSize:14,
        color:'#888888',
        width:sr.w / 3 * 2,
        textAlign:'right',
        paddingRight:5,
    },
    itemRight:{
        fontSize:14,
        color:'#333333',
        width:sr.w / 3 * 2,
        textAlign:'right',
        paddingRight:5,
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 12,
    },
    item:{
        height:45,
        marginTop:1,
        justifyContent: 'space-between',
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    list:{
        marginTop:7,
    },
    roadInfo:{
        flex:1,
        textAlign:'right',
        fontSize:14,
        padding: 0,
    },
    itemEnd:{
        fontSize:14,
        marginRight:10,
        marginLeft:5,
    },
    imgChoose:{
        width:14,
        height:14,
        marginLeft:12,
    },
    btnpublishRoadmap:{
        width:351,
        backgroundColor:'#c61a29',
        height:45,
        borderRadius:4,
        alignSelf:'center',
        marginTop:20,
    },
    btnpublishRoadmapFont:{
        fontWeight:'400',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/shortroadmap/ModifyPrice.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    Image,
    TouchableOpacity,
} = ReactNative;

import _ from 'lodash';

const { Button, CustomSheet } = COMPONENTS;
const Ranking = require('./Ranking');

const PriceType = {
    LONG:0,
    LONG_START:1,
};
const ChangeType = {
    PRICE:0,
    PERCENT:1,
    RANK:2,
};
module.exports = React.createClass({
    statics: {
        title: '调价',
    },
    getInitialState () {
        return {
            price: '',
            type: [],
            change: ChangeType.PRICE,
            add: false,
            sub: false,
            ranking:0,
            changePrice:0,
            actionSheetVisible:false,
        };
    },
    toLongStart () {
        const { type } = this.state;
        if (_.findIndex(type, (o) => o == PriceType.LONG_START) == -1) {
            type.push(PriceType.LONG_START);
            this.setState({ type });
        } else {
            this.setState({ type: _.reject(type, o => o == PriceType.LONG_START) });
        }
    },
    toLongPrice () {
        const { type } = this.state;
        if (_.findIndex(type, (o) => o == PriceType.LONG) == -1) {
            type.push(PriceType.LONG);
            this.setState({ type });
        } else {
            this.setState({ type: _.reject(type, o => o == PriceType.LONG) });
        }
    },
    toAdd () {
        this.setState({
            add: true,
            sub: false,
        });
    },
    toSub () {
        this.setState({
            add: false,
            sub: true,
        });
    },
    toChangePrice () {
        this.setState({
            change: ChangeType.PRICE,
            add: false,
            sub: false,
            price: '',
        });
    },
    toChangePercent () {
        this.setState({
            change: ChangeType.PERCENT,
            add: false,
            sub: false,
            price: '',
        });
    },
    toChangeRanking () {
        this.setState({
            change: ChangeType.RANK,
            add: false,
            sub: false,
        });
    },
    toRanking () {
        this.setState({ actionSheetVisible: true });
    },
    doClose () {
        this.setState({ actionSheetVisible: false });
    },
    toFirst () {
        this.setState({
            ranking: 1,
            actionSheetVisible: false,
        });
    },
    toSecond () {
        this.setState({
            ranking: 2,
            actionSheetVisible: false,
        });
    },
    toThird () {
        this.setState({
            ranking: 3,
            actionSheetVisible: false,
        });
    },
    toFourth () {
        this.setState({
            ranking: 4,
            actionSheetVisible: false,
        });
    },
    doConfirmChange () {
        const { type, change, price, ranking, add, sub } = this.state;
        const { userId, roadmapId } = this.props;
        if (type.length == 0) {
            Toast('请选择需要调价的类型');
            return;
        }
        if (change == ChangeType.UNKNOW) {
            Toast('请选择需要调价的模式');
            return;
        }
        if (change == ChangeType.RANK && ranking == 0) {
            Toast('请选择需要调整的名次');
            return;
        }
        if (add == false && sub == false && ranking == 0) {
            Toast('请选择上调还是下调或者排名');
            return;
        }
        if ((price == null || price == '') && change == ChangeType.PRICE) {
            Toast('请输入需要调整的价格');
            return;
        }
        if ((price == null || price == '') && change == ChangeType.PERCENT) {
            Toast('请输入需要调整的百分比');
            return;
        }
        var value = 0;
        if (change != ChangeType.RANK && change != ChangeType.UNKNOW) {
            if (sub) {
                value = -parseInt(price);
            } else {
                value = parseInt(price);
            }
        } else {
            value = ranking;
        }
        const param = {
            userId: userId,                           // 用户Id
            roadmapIdList: roadmapId,                 // 路线Id列表
            typeList: type,                         // 修改类型列表（0：短途运费单价 1：短途运费起价 ）
            mode: change,                             // 修改模式 （0: 按照价格调整 1: 按照百分比调整 2: 按照名次调整）
            value,                                    // 修改量（如果mode为0和1时，正数为增加，负数为减少）
        };
        POST(app.route.ROUTE_SET_ROADMAP_PRICE, param, this.doSetRoadmapPriceSuccess, true);
    },
    doSetRoadmapPriceSuccess (data) {
        if (data.success) {
            Toast('修改成功');
            this.props.refresh();
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { change, type, add, sub, ranking, actionSheetVisible, price } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.typeContainer}>
                    <View style={styles.typeTop}>
                        <TouchableOpacity onPress={this.toLongPrice}
                            style={styles.toChooseFirst}>
                            <Image style={styles.btnImage}
                                source={_.find(type, o => o == PriceType.LONG) == PriceType.LONG ?
                                    app.img.order_choose_r : app.img.order_no_choose_r} />
                            <Text style={styles.itemText}>短途单价</Text>
                        </TouchableOpacity>
                        <TouchableOpacity onPress={this.toLongStart}
                            style={styles.toChoose}>
                            <Image style={styles.btnImage}
                                source={_.find(type, o => o == PriceType.LONG_START) == PriceType.LONG_START
                                    ? app.img.order_choose_r : app.img.order_no_choose_r} />
                            <Text style={styles.itemText}>短途起价</Text>
                        </TouchableOpacity>
                    </View>
                </View>
                <View style={styles.priceChange}>
                    <TouchableOpacity onPress={this.toChangePrice}
                        style={[styles.itemChange, change == 0 ? { backgroundColor:'#c91521' } : {}]}>
                        <Text style={[styles.changeText, change == 0 ? { color:'#FFFFFF' } : {}]}>按价格调整</Text>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={this.toChangePercent}
                        style={[styles.itemChange, change == 1 ? { backgroundColor:'#c91521' } : {}]}>
                        <Text style={[styles.changeText, change == 1 ? { color:'#FFFFFF' } : {}]}>按百分比调整</Text>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={this.toChangeRanking}
                        style={[styles.itemChange, change == 2 ? { backgroundColor:'#c91521' } : {}]}>
                        <Text style={[styles.changeText, change == 2 ? { color:'#FFFFFF' } : {}]}>按排名调整</Text>
                    </TouchableOpacity>
                </View>
                <View style={styles.priceNumber}>
                    {
                        (change == 1 || change == 0) ?
                            <View style={styles.bottomLeft}>
                                <TouchableOpacity onPress={this.toAdd}
                                    style={styles.priceAdd}>
                                    <Image style={styles.chooseImage}
                                        source={add ? app.img.order_choose : app.img.order_no_choose} />
                                    <Text style={styles.priceAddText}>上调</Text>
                                </TouchableOpacity>
                                <TouchableOpacity onPress={this.toSub}
                                    style={styles.priceSub}>
                                    <Image style={styles.chooseImage}
                                        source={sub ? app.img.order_choose : app.img.order_no_choose} />
                                    <Text style={styles.priceAddText}>下调</Text>
                                </TouchableOpacity>
                            </View>
                        :
                            <View />
                    }
                    {
                        change == 0 ?
                            <View style={styles.priceInput}>
                                <TextInput
                                    onChangeText={(text) => this.setState({ price: text })}
                                    multiline={false}
                                    style={styles.bottomRight}
                                    placeholder={'请输入需要修改的价格'}
                                    autoCapitalize={'none'}
                                    underlineColorAndroid={'transparent'}
                                    defaultValue={price}
                                />
                                <Text style={styles.bottomRightText}>元</Text>
                            </View>
                        :
                        change == 1 ?
                            <View style={styles.priceInput}>
                                <TextInput
                                    onChangeText={(text) => this.setState({ price: text })}
                                    multiline={false}
                                    style={styles.bottomRight}
                                    placeholder={'请输入需要修改的百分比'}
                                    defaultValue={price}
                                    autoCapitalize={'none'}
                                    underlineColorAndroid={'transparent'}
                                />
                                <Text style={styles.bottomRightText}>%</Text>
                            </View>
                        :
                        change == 2 ?
                            <View style={styles.priceInput}>
                                <TouchableOpacity onPress={this.toRanking}>
                                    <Text style={styles.bottomRightText}>
                                        {ranking == 1 ?
                                        '第一名' : ranking == 2 ?
                                        '第二名' : ranking == 3 ?
                                        '第三名' : ranking == 4 ?
                                        '第四名' : '请选择排名'}
                                    </Text>
                                </TouchableOpacity>
                            </View>
                            :
                            <View />
                        }
                </View>
                <Button onPress={this.doConfirmChange}
                    style={styles.btnConfirmChange}
                    textStyle={styles.btnConfirmChangeText}>
                        确认调价
                    </Button>
                <CustomSheet
                    visible={actionSheetVisible}>
                    <Ranking doClose={this.doClose}
                        toFirst={this.toFirst}
                        toSecond={this.toSecond}
                        toThird={this.toThird}
                        toFourth={this.toFourth} />
                </CustomSheet>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f4f4f4',
    },
    typeContainer:{
        height:40,
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
    },
    typeTop:{
        flexDirection: 'row',
        alignItems:'center',
    },
    toChooseFirst:{
        flexDirection: 'row',
        alignItems:'center',
    },
    toChoose:{
        flexDirection: 'row',
        alignItems:'center',
        marginLeft:30,
    },
    btnImage:{
        height:16,
        width:16,
        marginLeft:10,
    },
    itemText:{
        fontSize:14,
        color:'#333333',
        marginLeft:5,
    },
    typeBottom:{
        flexDirection: 'row',
        alignItems:'center',
        marginTop:7,
    },
    priceChange:{
        height:40,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        alignItems:'center',
        justifyContent:'space-around',
        marginTop:10,
    },
    itemChange:{
        height:20,
        width:100,
        backgroundColor:'#f3f3f3',
    },
    changeText:{
        fontSize:12,
        color:'#333333',
        textAlign:'center',
        height:20,
        lineHeight:18,
    },
    priceNumber:{
        height:40,
        backgroundColor:'#FFFFFF',
        marginTop:1,
        flexDirection: 'row',
        justifyContent:'space-between',
    },
    bottomLeft:{
        flexDirection: 'row',
        alignItems:'center',
        height:40,
    },
    priceAdd:{
        flexDirection: 'row',
        alignItems:'center',
        marginLeft:10,
    },
    chooseImage:{
        height:9,
        width:9,
    },
    priceAddText:{
        fontSize:14,
        color:'#333333',
        marginLeft:5,
    },
    priceSub:{
        flexDirection: 'row',
        alignItems:'center',
        marginLeft:20,
    },
    priceInput:{
        flexDirection: 'row',
        alignItems:'center',
    },
    bottomRight:{
        height:40,
        width:sr.w / 2,
        fontSize:12,
        color: '#444444',
        textAlign:'right',
    },
    bottomRightText:{
        fontSize:13,
        color:'#333333',
        marginRight:15,
        marginLeft:3,
    },
    btnConfirmChange:{
        height:40,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        marginTop:250,
        borderRadius:4,
    },
    btnConfirmChangeText:{
        fontSize:16,
        fontWeight:'500',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/shortroadmap/Ranking.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '',
    },
    render () {
        return (
            <View style={styles.container}>
                <View style={styles.containerTop}>
                    <Text style={styles.textTop}>名次</Text>
                    <TouchableOpacity onPress={this.props.doClose}>
                        <Image
                            resizeMode='stretch'
                            source={app.img.common_close}
                            style={styles.closeStyle} />
                    </TouchableOpacity>
                </View>
                <View style={styles.line} />
                <TouchableOpacity onPress={this.props.toFirst}
                    style={styles.rankingStyleTop}>
                    <Text>第一名</Text>
                </TouchableOpacity>
                <TouchableOpacity onPress={this.props.toSecond}
                    style={styles.rankingStyle}>
                    <Text>第二名</Text>
                </TouchableOpacity>
                <TouchableOpacity onPress={this.props.toThird}
                    style={styles.rankingStyle}>
                    <Text>第三名</Text>
                </TouchableOpacity>
                <TouchableOpacity onPress={this.props.toFourth}
                    style={styles.rankingStyle}>
                    <Text>第四名</Text>
                </TouchableOpacity>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#fafafa',
    },
    containerTop:{
        height:40,
        backgroundColor:'#ffffff',
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textTop:{
        fontSize:17,
        marginLeft:(sr.w - 17 * 2) / 2,
        color:'#333333',
    },
    closeStyle:{
        height:30,
        width:30,
    },
    line:{
        height:1,
        backgroundColor:'#ededed',
    },
    rankingStyleTop:{
        height:40,
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
        marginLeft:10,
    },
    rankingStyle:{
        height:40,
        backgroundColor:'#FFFFFF',
        marginTop:1,
        justifyContent:'center',
        marginLeft:10,
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/shortroadmap/ReceivePointMarketResult.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

const RoadmapMarketList = require('./RoadmapMarketList');
module.exports = React.createClass({
    statics: {
        title: '查询结果',
    },
    getInitialState () {
        return {
            tabIndex: 0,
        };
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    render () {
        const menus = ['同城配送单价', '同城配送起价', '时间最短'];
        const { tabIndex } = this.state;
        const { info, data } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.tabContainer}>
                    {
                        menus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={this.changeTab.bind(null, i)}
                                style={styles.touchTab}>
                                <View style={styles.tabButton}>
                                    <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                        {item}
                                    </Text>
                                    <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                </View>
                            </TouchableHighlight>
                        ))
                    }
                </View>
                {
                    menus.map((item, i) => (
                        <RoadmapMarketList key={i} index={i} tabIndex={tabIndex} info={info} data={data.agent || []} listName={'agent'} />
                    ))
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonCenter: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonRight: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        borderRadius: 9,
    },
    tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/shortroadmap/RoadmapDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    ScrollView,
    Image,
    TouchableOpacity,
} = ReactNative;
const { Button, SelectStartPointAddress, SelectShortAddress } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '路线详情',
    },
    componentWillMount () {
        this.getRoadmapDetail();
    },
    getInitialState () {
        return {
            startInfo:'',
            endPoint: '',
            endPointLastCode:'',
            price:'',
            minFee:'',
            basePrice:0,
            baseMinFee:0,
        };
    },
    showStartPlace () {
        const { startInfo } = this.state;
        app.push({
            component: SelectStartPointAddress,
            passProps:{
                endPointLastCode:startInfo.startPointLastCode,
                id:startInfo.shopId || startInfo.agentId,
                confirmAddress:(obj) => {
                    this.setState({ startInfo:obj });
                },
            },
        });
    },
    showEndPlace () {
        const { endPointLastCode, startInfo } = this.state;
        app.push({
            component: SelectShortAddress,
            title:'到货地',
            passProps:{
                endPointLastCode,
                parentCode : startInfo.startPointLastCode,
                confirmAddress:(endPoint, endPointLastCode) => {
                    this.setState({ endPoint, endPointLastCode });
                },
            },
        });
    },
    getRoadmapDetail () {
        const id = this.props.roadmapId;
        const param = {
            roadmapId:id,
        };
        POST(app.route.ROUTE_GET_SHIPPER_ROADMAP_DETAIL, param, this.getRoadmapDetailSuccess, true);
    },
    getRoadmapDetailSuccess (data) {
        if (data.success) {
            const obj = data.context;
            this.setState({
                startInfo:{ startPoint:obj.startPoint, id:obj.shop ? obj.shop.id : obj.agent ? obj.agent.id : '' },
                endPoint: obj.endPoint,
                endPointLastCode:obj.endPointLastCode,
                price:obj.price,
                minFee:obj.minFee + '',
                basePrice:obj.basePrice == null ? 0 : obj.basePrice,
                baseMinFee:obj.baseMinFee == null ? 0 : obj.baseMinFee,
            });
        } else {
            Toast(data.msg);
        }
    },
    doMadifyRoadmap () {
        const id = this.props.roadmapId;
        const { endPoint, endPointLastCode, price, minFee, basePrice, baseMinFee } = this.state;
        const param = {
            userId:app.personal.info.userId,
            roadmapId:id,
            endPoint,
            endPointLastCode,
            price,
            basePrice,
            minFee,
            baseMinFee,
        };
        POST(app.route.ROUTE_MODIFY_ROADMAP, param, this.doMadifyRoadmapSuccess, true);
    },
    doMadifyRoadmapSuccess (data) {
        if (data.success) {
            Toast('修改成功');
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { startInfo, endPoint, price, minFee, basePrice, baseMinFee } = this.state;
        return (
            <ScrollView>
                <View style={styles.container}>
                    <View>
                        <TouchableOpacity
                            activeOpacity={0.6}
                            onPress={this.showStartPlace}>
                            <View style={styles.itemStart}>
                                <Text style={styles.itemLeft}>始发地</Text>
                                <View style={styles.itemPoint}>
                                    <Text style={styles.itemRightStart}
                                        numberOfLines={1}
                                        ellipsizeMode='tail'>{(app.utils.getVisibleText(app.personal.currentShopAddress + app.personal.currentShopName, 50))}</Text>
                                </View>
                            </View>
                        </TouchableOpacity>
                        <TouchableOpacity
                            activeOpacity={0.6}
                            onPress={this.showEndPlace}>
                            <View style={styles.item}>
                                <Text style={styles.itemLeft}>到货地</Text>
                                <View style={styles.itemPoint}>
                                    <Text style={styles.itemRight}
                                        numberOfLines={1}
                                        ellipsizeMode='tail'>{endPoint}</Text>
                                    <Image
                                        source={app.img.common_go}
                                        style={styles.goStyle} />
                                </View>
                            </View>
                        </TouchableOpacity>
                    </View>
                    <View style={styles.list}>
                        <View style={styles.item}>
                            <Text style={styles.itemLeft}>短途起价</Text>
                            <TextInput style={styles.roadInfo}
                                underlineColorAndroid='transparent'
                                placeholder='请输入短途起价'
                                onChangeText={(text) => this.setState({ minFee: text })}
                                defaultValue={app.utils.N(minFee * 1) + ''} />
                            <Text style={styles.itemEnd}>元</Text>
                        </View>
                        <View style={styles.item}>
                            <Text style={styles.itemLeft}>短途单价/吨</Text>
                            <TextInput style={styles.roadInfo}
                                underlineColorAndroid='transparent'
                                placeholder='请输入短途单价'
                                onChangeText={(text) => this.setState({ price: text })}
                                defaultValue={app.utils.N(price * 1) + ''} />
                            <Text style={styles.itemEnd}>元</Text>
                        </View>
                        <View style={styles.list}>
                            <View>
                                <View style={styles.item}>
                                    <Text style={styles.itemLeft}>短途起价保底</Text>
                                    <TextInput style={styles.roadInfo}
                                        underlineColorAndroid='transparent'
                                        placeholder='请输入短途起价保底'
                                        onChangeText={(text) => this.setState({ baseMinFee: text })}
                                        defaultValue={app.utils.N(baseMinFee * 1) + ''} />
                                    <Text style={styles.itemEnd}>元</Text>
                                </View>
                                <View style={styles.item}>
                                    <Text style={styles.itemLeft}>短途单价保底/吨</Text>
                                    <TextInput style={styles.roadInfo}
                                        underlineColorAndroid='transparent'
                                        placeholder='请输入短途单价保底'
                                        onChangeText={(text) => this.setState({ basePrice: text })}
                                        defaultValue={app.utils.N(basePrice * 1) + ''} />
                                    <Text style={styles.itemEnd}>元</Text>
                                </View>
                            </View>
                            <Button onPress={this.doMadifyRoadmap} style={styles.btn} textStyle={styles.btnFont}>确认修改</Button>
                        </View>
                    </View>
                </View>
            </ScrollView>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    itemStart:{
        height:40,
        justifyContent: 'space-between',
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    itemLeft:{
        flex:1,
        paddingLeft:12,
        fontSize:14,
        color:'#333333',
    },
    itemPoint:{
        flexDirection: 'row',
    },
    itemRightStart:{
        fontSize:14,
        color:'#888888',
        width:sr.w / 3 * 2,
        textAlign:'right',
        paddingRight:10,
    },
    itemRight:{
        fontSize:14,
        color:'#333333',
        width:sr.w / 3 * 2,
        textAlign:'right',
        paddingRight:5,
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 12,
    },
    item:{
        height:40,
        marginTop:1,
        justifyContent: 'space-between',
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    list:{
        marginTop:7,
    },
    roadInfo:{
        flex:1,
        textAlign:'right',
        fontSize:14,
        padding:0,
    },
    itemEnd:{
        fontSize:14,
        marginRight:10,
        marginLeft:5,
    },
    imgChoose:{
        width:14,
        height:14,
        marginLeft:12,
    },
    btnpublishRoadmap:{
        width:351,
        backgroundColor:'#c61a29',
        height:45,
        borderRadius:4,
        alignSelf:'center',
        position:'absolute',
        top:250,
    },
    btnpublishRoadmapFont:{
        fontWeight:'400',
    },
    btn:{
        width:351,
        height:45,
        marginTop:18,
        borderRadius:4,
        alignSelf:'center',
        backgroundColor:'#FE9917',
    },
    btnFont:{
        fontWeight:'400',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/shortroadmap/RoadmapMarketList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const { PageList } = COMPONENTS;
let km = '';

module.exports = React.createClass({
    renderRow (obj, rowID) {
        if (obj.distance) {
            if (obj.distance > 1) {
                km = parseInt(!obj.distance ? 0 : obj.distance) + 'km';
            } else {
                km = (parseInt(!obj.distance ? 0 : obj.distance) * 1000) + 'm';
            }
        }
        return (
            <View style={styles.row}>
                <View style={styles.rowChild}>
                    <View style={styles.address}>
                        <Text style={styles.rowChildFont} numberOfLines={1}>{obj.shipperName}</Text>
                        <Text style={{ flex:0.3 }}>{'-'}</Text>
                        <Text style={styles.rowChildFont} numberOfLines={1}>{this.props.info.endPoint}</Text>
                    </View>
                    <Text style={styles.rowChildDistance}>距我:{km}</Text>
                </View>
                <View style={styles.itemInfo}>
                    <View style={styles.item}>
                        <Text style={styles.rowChildNumber}>￥{app.utils.N(obj.price) + '/吨'}</Text>
                        <Text style={styles.rowChildPrice}>运费单价</Text>
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.rowChildNumber}>￥{app.utils.N(obj.minFee) + '/吨'}</Text>
                        <Text style={styles.rowChildTitle}>运费起价</Text>
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.rowChildTime}>{obj.duration}天</Text>
                        <Text style={styles.rowChildTitle}>预计用时</Text>
                    </View>
                </View>
            </View>
        );
    },
    render () {
        const { index, tabIndex, listName, info, data } = this.props;
        return (
            <View style={index === tabIndex ? styles.container : { left:-sr.tw, top:0, position:'absolute' }}>
                {
                    !!data &&
                    <PageList
                        ref={(ref) => { this.listView = ref; }}
                        renderRow={this.renderRow}
                        listParam={{ userId: app.personal.info.userId, orderBy: index,isCityDistribute:true,shopId:app.personal.currentShopId, ...info }}
                        listName={listName}
                        list={data}
                        autoLoad={false}
                        listUrl={app.route.ROUTE_GET_ROADMAP_LIST_WITH_END_POINT}
                        refreshEnable
                        />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    row: {
        flex:1,
        paddingVertical: 20,
        paddingHorizontal: 2,
        backgroundColor:'#FFFFFF',
        marginTop:1,
    },
    rowChild:{
        width:sr.w,
        flexDirection:'row',
        marginTop:3,
        marginBottom:3,
    },
    rowChildFont:{
        flex:2,
        fontSize:14,
        marginLeft:5,
        fontWeight:'300',
    },
    rowChildDistance:{
        fontSize:12,
        color:'#FF9D24',
        fontWeight:'300',
        marginRight:5,
        flex:0.5,
    },
    rowChildNumber:{
        color:'#f61525',
        fontSize:15,
        fontWeight:'500',
    },
    rowChildTime:{
        fontSize:16,
        color:'#333333',
    },
    rowChildTitle:{
        flex:1,
        fontSize:12,
        color:'#333333',
    },
    rowChildPrice:{
        flex:1,
        fontSize:12,
        color:'#333333',
    },
    address:{
        flexDirection:'row',
        flex:1.5,
    },
    itemInfo:{
        flexDirection:'row',
        flex:1,
        alignItems:'center',
        justifyContent:'center',
        marginTop:3,
    },
    item:{
        flex:1,
        alignItems:'center',
        justifyContent:'center',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/shortroadmap/RoadmapMarketResult.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

import { Geolocation } from '@remobile/react-native-baidu-map';

const ShopMarketResult = require('./ShopMarketResult');
const ReceivePointMarketResult = require('./ReceivePointMarketResult');
const BaidumapSelectPosition = require('../../baidumap');
let selectIndex = 0;
const TabTitle = React.createClass({
    getInitialState () {
        return {
            tab: 0,
        };
    },
    render () {
        const { tab } = this.state;
        const tabMenus = this.props.tabMenus ? this.props.tabMenus : [];

        return (
            <View style={styles.tabContainer}>
                {
                tabMenus.map((item, i) => (
                    <TouchableHighlight
                        key={i}
                        underlayColor='rgba(0, 0, 0, 0)'
                        onPress={() => {
                            selectIndex = i;
                            this.setState({ tab:i });
                            this.props.changeTab();
                        }}
                        style={[styles.touchTab, tab === i ? { backgroundColor:'#E5E5E5' } : { backgroundColor:'#c81622' }]}>
                        <Text style={tab === i ? { color:'#c81622' } : { color:'#E5E5E5' }}>
                            {item}
                        </Text>
                    </TouchableHighlight>
                ))
            }
            </View>
        );
    },
});
module.exports = React.createClass({
    statics: {
        title: (<TabTitle changeTab={() => { app.scene.changeTab&&app.scene.changeTab(); }} />),
        rightButton: { title: '位置', handler: () => { app.scene.setPosition&&app.scene.setPosition(); } },
    },
    getInitialState () {
        return {
            tabIndex: 0,
            context: {},
            tabMenus:[],
            mapInfo:{ location:[0, 0], startPoint:'' },
        };
    },
    setPosition () {
        app.push({
            component: BaidumapSelectPosition,
            passProps:{
                confirm:(marker) => {
                    this.setState({ mapInfo:{ startPoint:marker.province + marker.city, location:[marker.longitude, marker.latitude] } }, () => { this.updateRightButton(marker.city); }); ;
                },
            },
        });
    },
    componentDidMount () {
        Geolocation.getCurrentPosition()
            .then(data => {
                console.warn(JSON.stringify(data));
                this.setState({ mapInfo:{ startPoint:data.province + data.city, location:[data.longitude, data.latitude] } }, () => { this.getRoadMaps(data.city); });
            })
            .catch(e => {
                console.warn(e, 'error');
            });
    },
    getRoadMaps (city) {
        this.updateRightButton(city);
        const { mapInfo } = this.state;
        const { endPointLastCode, endPoint } = this.props;
        const param = {
            userId: app.personal.info.userId,
            orderBy: 0,
            pageNo:0,
            pageSize:10,
            endPoint,
            endPointLastCode,
            ...mapInfo,
        };
        POST(app.route.ROUTE_GET_ROADMAP_LIST_WITH_END_POINT, param, this.getRoadMapsSuccess, false);
    },
    getRoadMapsSuccess (data) {
        if (data.success) {
            this.setState({ context:data.context, tabMenus:(!data.context.agent || data.context.agent.length == 0) ? ['物流超市'] : ['物流超市', '揽收点'] }, () => { this.updateTitle(); });
        } else {
            Toast(data.msg);
        }
    },
    updateRightButton (city) {
        app.getCurrentRoute().rightButton = { title:city, handler: () => { this.setPosition(); } };
        app.forceUpdateNavbar();
    },
    updateTitle () {
        app.getCurrentRoute().title = (<TabTitle changeTab={this.changeTab} tabMenus={this.state.tabMenus} />);
        app.forceUpdateNavbar();
    },
    changeTab () {
        this.setState({ tabIndex:selectIndex });
    },
    render () {
        const { tabIndex, mapInfo, context } = this.state;
        const { endPointLastCode, endPoint } = this.props;
        const info = {
            endPoint,
            endPointLastCode,
            ...mapInfo,
        };
        return (
            <View style={styles.container}>
                {
                    tabIndex === 0 ? <ShopMarketResult info={info} data={context} /> : <ReceivePointMarketResult info={info} data={context} />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        height: 30,
        width: 140,
        borderRadius:5,
        borderColor:'white',
        borderWidth:1,
        flexDirection: 'row',
        overflow: 'hidden',
    },
    touchTab: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/shortroadmap/ShopMarketResult.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

const RoadmapMarketList = require('./RoadmapMarketList');
module.exports = React.createClass({
    statics: {
        title: '查询结果',
    },
    getInitialState () {
        return {
            tabIndex: 0,
        };
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    render () {
        const menus = ['同城配送单价', '同城配送起价', '时间最短'];
        const { tabIndex } = this.state;
        const { info, data } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.tabContainer}>
                    {
                        menus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={this.changeTab.bind(null, i)}
                                style={styles.touchTab}>
                                <View style={styles.tabButton}>
                                    <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                        {item}
                                    </Text>
                                    <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                </View>
                            </TouchableHighlight>
                        ))
                    }
                </View>
                {
                    menus.map((item, i) => (
                        <RoadmapMarketList key={i} index={i} info={info} data={data.shop ? data.shop.roadmapList : []} tabIndex={tabIndex} listName={'shop.roadmapList'} />
                    ))
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonCenter: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonRight: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        borderRadius: 9,
    },
    tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```

* PDClient/project/App/modules/shortLogisticsCompany/shortroadmap/index.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button, MessageBox, PageList } = COMPONENTS;
const RoadmapDetail = require('./RoadmapDetail');
const BranchList = require('../cargo/BranchList');
const ModifyPrice = require('./ModifyPrice');
const AddRoadmap = require('./AddRoadmap');
const RoadmapMarketResult = require('./RoadmapMarketResult');

let searchText = '';
const Title = React.createClass({
    render () {
        return (
            <View style={styles.searchContainer}>
                <Image
                    resizeMode='cover'
                    source={app.img.common_search_button}
                    style={styles.iconSearch} />
                <TextInput
                    placeholder='输入路线终点查询路线'
                    defaultValue={searchText}
                    underlineColorAndroid='transparent'
                    onChangeText={(text) => { searchText = text; }}
                    onSubmitEditing={this.props.doStartSearch}
                    style={styles.searchTextInput}
                    />
            </View>
        );
    },
});

module.exports = React.createClass({
    statics: {
        title: (<Title doStartSearch={() => { app.scene.doStartSearch&&app.scene.doStartSearch(); }} />),
    },
    onWillFocus () {
        app.getCurrentRoute().title = (<Title doStartSearch={() => { app.scene.doStartSearch&&app.scene.doStartSearch(); }} />);
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = { title: app.personal.currentShopName, handler: () => { app.scene.selectShop&&app.scene.selectShop(); }, isLimitLength:true };
        app.forceUpdateNavbar();
    },
    selectShop () {
        app.push({
            component:BranchList,
            passProps:{
                onWillFocus:this.onWillFocus,
                setCurrentShopId:this.setCurrentShopId,
            },
        });
    },
    setCurrentShopId (id) {
        this.setState({
            shopId:id,
        });
    },
    getInitialState () {
        return {
            userId:app.personal.info.userId,
            keyword: '',
            dataList: [],
            list:[],
            roadmapId:[],
            shopId:app.personal.currentShopId,
        };
    },
    doStartSearch () {
        this.setState({
            keyword:searchText,
        });
    },
    showRoadmapMarket (endPointLastCode, endPoint) {
        app.push({
            component:RoadmapMarketResult,
            passProps:{
                endPointLastCode,
                endPoint,
            },
        });
    },
    showRoadmapDetail (id) {
        app.push({
            component:RoadmapDetail,
            passProps:{
                roadmapId: id,
                refreshList: this.refreshList,
            },
        });
    },
    refresh () {
        this.setState({ keyword: searchText }, () => {
            this.listView.refresh();
        });
    },
    doChoose (obj, rowId) {
        const { list } = this.state;
        list[rowId].selected = !list[rowId].selected;
        this.setState({
            list,
        });
    },
    doSelectedAll () {
        const { list } = this.state;
        if (_.every(list, (o) => o.selected == true)) {
            _.forEach(list, o => { o.selected = false; });
        } else {
            _.forEach(list, o => { o.selected = true; });
        }
        this.setState({
            list,
        });
    },
    toModifyPrice () {
        const { list, userId, roadmapId } = this.state;
        list.forEach((item, i) => {
            item.selected == true && roadmapId.push(item.item.id);
        });
        if (roadmapId.length == 0) {
            Toast('请选择需要调价的路线');
            return;
        }
        app.push({
            component: ModifyPrice,
            passProps: {
                userId,
                roadmapId: _.uniq(roadmapId),
                refresh:this.refreshList,
            },
        });
        this.setState({
            roadmapId:[],
        });
    },
    toDeleteRoadmap () {
        const { list } = this.state;
        if (_.every(list, (o) => o.selected == false)) {
            Toast('请选择需要删除的路线');
        } else {
            app.showModal(
                <MessageBox
                    onConfirm={this.doConfirmDelete}
                    content={'是否确定删除所选路线'}
                    onCancel title={false}
                    width={sr.s(250)} />
            );
        }
    },
    delete (id) {
        const param = {
            userId:app.personal.info.userId,
            roadmapId:id,
        };
        let roadmapId = id;
        POST(app.route.ROUTE_REMOVE_ROADMAP, param, this.doConfirmDeleteSuccess.bind(null, roadmapId), false);
    },
    doConfirmDelete () {
        const { list } = this.state;
        list.forEach((item, i) => {
            item.selected == true && this.delete(item.item.id);
        });
    },
    doConfirmDeleteSuccess (roadmapId, data) {
        if (data.success) {
            var list = this.listView.list;
            _.remove(list, function (o) {
                return o.id == roadmapId;
            });
            Toast('路线删除成功');
            this.setState({
                list:[],
            });
        } else {
            Toast('路线删除失败');
        }
    },
    toAddRoadmap () {
            app.push({
                component:AddRoadmap,
                passProps:{
                    refreshList:this.refreshList,
                },
            });
    },
    setEnable (type) {
        const { list } = this.state;
        var roadmapIdList = [];
        list.forEach((item, i) => {
            item.selected == true && roadmapIdList.push(item.item.id);
        });
        if (roadmapIdList.length == 0) {
            Toast('请选择您需要修改的路线');
            return;
        }
        const param = {
            userId: app.personal.info.userId,
            roadmapIdList,
            enable: type,
        };
        POST(app.route.ROUTE_SET_ROADMAP_ENABLE, param, this.setRoadmapEnableSuccess.bind(null,type), false);
    },
    setRoadmapEnableSuccess (type, data) {
        const { list } = this.state;
        if (data.success) {
            Toast('修改成功');
            number = 0;
            list.forEach((item, i) => {
                if (item.selected == true) {
                    item.selected = false;
                    item.item.enable = type;
                }
            });
        } else {
            Toast(data.msg);
        }
        this.setState({
            list
        });
        this.refreshList();
    },
    refreshList () {
        this.listView.refresh();
    },
    renderRow (obj, rowId) {
        const { list } = this.state;
        if (!_.find(list, (o) => o.index == rowId)) {
            list.push({ selected:false, index:rowId, item:obj });
        }
        return (
            <View style={styles.item}>
                <View style={styles.rowTop}>
                    <TouchableOpacity
                        onPress={this.doChoose.bind(null, obj, rowId)}>
                        <Image
                            resizeMode='cover'
                            style={styles.imgChoose}
                            source={!list[rowId].selected ? app.img.order_no_choose : app.img.order_choose} />
                    </TouchableOpacity>
                    <TouchableOpacity style={styles.point} onPress={this.showRoadmapDetail.bind(null, obj.id)}>
                        <View style={styles.pointStart}>
                            <Text style={styles.address} numberOfLines={1}>{obj.startPoint}</Text>
                        </View>
                        <Image resizeMode='contain'
                            source={app.img.cargo_point}
                            style={styles.imgLine} />
                        <View style={styles.pointEnd}>
                            <Text style={styles.address} numberOfLines={1}>{obj.endPoint}</Text>
                        </View>
                    </TouchableOpacity>
                    {
                        obj.enable ?
                        <Text style={styles.roadmapStart}>已开通</Text>
                        :
                        <Text style={styles.roadmapStop}>已停运</Text>
                    }
                </View>
                <View style={styles.rowBottom}>
                    <View style={styles.price}>
                        <Text style={styles.priceText}>起价:</Text>
                        <Text style={styles.priceNumber}>{obj.minFee}</Text>
                        <Text style={styles.priceText}>单价:</Text>
                        <Text style={styles.priceNumber}>{obj.price}</Text>
                    </View>
                    <Button style={styles.btnRoadmapShipper}
                        textStyle={styles.btnShipperFont}
                        onPress={this.showRoadmapMarket.bind(null, obj.endPointLastCode,obj.endPoint)}>
                        路线行情
                    </Button>
                </View>
            </View>
        );
    },
    render () {
        const { list, keyword, shopId } = this.state;
        const status = list.length != 0 && _.every(list, (o) => o.selected == true);
        var setType = {
            open: true,
            close: false,
         };
        var number = 0;
        var roadmapList = [];
        list.forEach((item, i) => {
            item.selected == true && number++;
        });
        list.forEach((item, i) => {
            item.selected == true && roadmapList.push({type:item.item.enable,id:item.item.id});
        });
        return (
            <View style={styles.container}>
                <View style={styles.top}>
                        <Button style={styles.btnAddRoadmap}
                            textStyle={styles.btnReleaseFont}
                            onPress={this.toAddRoadmap}>
                            发布短途路线
                        </Button>
                </View>
                <View style={styles.listView} />
                <PageList
                    ref={(ref) => { this.listView = ref; }}
                    renderRow={this.renderRow}
                    onGetList={this.onGetList}
                    listName={'roadmapList'}
                    renderSeparator={null}
                    listParam={{ userId: app.personal.info.userId, shopId, keyword, isCityDistribute : true }}
                    listUrl={app.route.ROUTE_GET_ROADMAP_LIST_SHIPPER}
                    refreshEnable
                    />
                <View style={styles.bottom}>
                    <View style={{flexDirection:'row'}}>
                        <TouchableOpacity style={styles.chooseAll} onPress={this.doSelectedAll}>
                            <Image
                                resizeMode='cover'
                                style={styles.imgChoose}
                                source={status ? app.img.order_choose : app.img.order_no_choose} />
                            <Text style={styles.chooseAllFont}>全选</Text>
                        </TouchableOpacity>
                        <Text style={styles.alreadyChoose}>已选{number}条</Text>
                    </View>
                    <View style={{flexDirection:'row',marginRight:8}}>
                        {
                            _.every(roadmapList, (o) => o.type == false) ?
                            <Button onPress={this.setEnable.bind(null,setType.open)}
                                style={styles.btnAllOpen}
                                textStyle={styles.btnAllOpenText}>
                                开通
                            </Button>
                            :
                            _.every(roadmapList, (o) => o.type == true) ?
                            <Button onPress={this.setEnable.bind(null,setType.close)}
                                style={styles.btnAllClose}
                                textStyle={styles.btnAllCloseText}>
                                停运
                            </Button>
                            :
                            <View style={{flexDirection:'row'}}>
                                <Button onPress={this.setEnable.bind(null,setType.open)}
                                    style={styles.btnAllOpen}
                                    textStyle={styles.btnAllOpenText}>
                                    开通
                                </Button>
                                <Button onPress={this.setEnable.bind(null,setType.close)}
                                    style={styles.btnAllClose}
                                    textStyle={styles.btnAllCloseText}>
                                    停运
                                </Button>
                            </View>
                        }
                        <Button onPress={this.toDeleteRoadmap}
                            style={styles.btnDelete}
                            textStyle={styles.btnDeleteFont}>删除</Button>
                        <Button onPress={this.toModifyPrice}
                            style={styles.btnModifyPrice}
                            textStyle={styles.btnModifyPriceFont}>调价</Button>
                    </View>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f4f4f4',
    },
    searchContainer: {
        height: 30,
        width: sr.w - 125,
        marginLeft:sr.w - 300,
        paddingVertical: 2,
        borderRadius: 4,
        backgroundColor: '#FFFFFF',
        flexDirection: 'row',
        overflow: 'hidden',
        alignItems:'center',
    },
    searchTextInput: {
        padding:0,
        height: 25,
        width: sr.w - 155,
        fontSize: 14,
        marginLeft: 5,
    },
    iconSearch: {
        height: 15,
        width: 15,
        marginLeft:5,
    },
    top:{
        height:80,
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent:'center',
    },
    btnAddRoadmap:{
        width:150,
        height:35,
        borderRadius:4,
        backgroundColor:'#fe9917',
    },
    btnReleaseFont:{
        color:'#FFFFFF',
        fontWeight:'400',
    },
    listView:{
        marginTop:8,
    },
    rowTop:{
        height:42,
        flexDirection:'row',
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        marginTop:3,
    },
    rowBottom:{
        marginTop:1,
        height:30,
        backgroundColor:'#FFFFFF',
    },
    price:{
        flexDirection:'row',
        height:30,
        alignItems:'center',
    },
    priceText:{
        marginLeft:8,
        fontSize:12,
        color:'#333333',
    },
    priceNumber:{
        fontSize:12,
        color:'#f01725',
    },
    imgChoose:{
        marginLeft:5,
        width:13,
        height:13,
    },
    imgLine:{
        marginLeft:10,
        width:35,
    },
    pointStart:{
        marginLeft:8,
        width:105,
        justifyContent:'center',
        alignItems:'center',
    },
    pointEnd:{
        marginLeft:8,
        width:115,
        justifyContent:'center',
        alignItems:'center',
    },
    address:{
        fontSize:14,
    },
    roadmapStart:{
        fontSize:14,
        color:'#50c248',
        right:(sr.w - 355) / 2,
    },
    roadmapStop:{
        fontSize:14,
        color:'#f01725',
        right:(sr.w - 355) / 2,
    },
    btnOpen:{
        marginTop:5,
        width:50,
        height:20,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#50c248',
        position:'absolute',
        right:(sr.w - 70) / 2,
    },
    btnOpenText:{
        color:'#50c248',
        fontSize:12,
    },
    btnClose:{
        marginTop:5,
        width:50,
        height:20,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#f01725',
        position:'absolute',
        right:(sr.w - 195) / 2,
    },
    btnRoadmapShipper:{
        marginTop:5,
        width:65,
        height:20,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#f01725',
        position:'absolute',
        right:(sr.w - 350) / 2,
    },
    btnShipperFont:{
        color:'#f01725',
        fontSize:12,
    },
    bottom:{
        height:50,
        width:sr.w,
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent:'space-between',
    },
    btnAllOpen:{
        marginLeft:9,
        width:50,
        height:25,
        backgroundColor:'#FFFFFF',
        borderColor:'#50c248',
        borderWidth:0.5,
        borderRadius:2,
    },
    btnAllOpenText:{
        color:'#50c248',
        fontSize:14,
    },
    btnAllCloseText:{
        color:'#f01725',
        fontSize:14,
    },
    btnAllClose:{
        marginLeft:9,
        width:50,
        height:25,
        backgroundColor:'#FFFFFF',
        borderColor:'#f01725',
        borderWidth:0.5,
        borderRadius:2,
    },
    btnDelete:{
        marginLeft:9,
        width:50,
        height:25,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
    },
    btnDeleteFont:{
        color:'#343434',
        fontSize:13,
        fontWeight:'400',
    },
    btnModifyPrice:{
        marginLeft:9,
        width:50,
        height:25,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#f21620',
    },
    btnModifyPriceFont:{
        color:'#f21620',
        fontSize:13,
        fontWeight:'400',
    },
    chooseAllFont:{
        fontSize:13,
        marginLeft:8,
    },
    alreadyChoose:{
        fontSize:12,
        marginLeft:5,
        marginTop:1,
    },
    chooseAll:{
        alignItems:'center',
        flexDirection:'row',
    },
    point:{
        flex:1,
        flexDirection:'row',
    },
});

```

* PDClient/project/App/modules/receivePoint/order/ReceiveOrderDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button, CustomSheet } = COMPONENTS;
const LogisticsDetail = require('../../common/LogisticsDetail');
const moment = require('moment');

module.exports = React.createClass({
    statics: {
        title: '货单详情',
    },
    getInitialState () {
        return {
            choose:false,
            receiving:false || this.props.receiving,
            data:{},
        };
    },
    componentWillMount () {
        this.getOrderDetail();
    },
    getOrderDetail () {
        const param = {
            userId: app.personal.info.userId,
            orderId: this.props.data.id,
        };
        POST(app.route.ROUTE_GET_ORDER_DETAIL, param, this.getOrderDetailSuccess, true);
    },
    getOrderDetailSuccess (data) {
        if (data.success) {
            this.setState({ data:data.context });
        } else {
            Toast(data.msg);
        }
    },
    toLogisticsDetail () {
        const { data } = this.state;
        app.push({
            component:LogisticsDetail,
            passProps:{
                obj:data,
                type:'agent',
            },
        });
    },
    toPayMode (payMode) {
        if (payMode == 0) {
            return '现付';
        } else if (payMode == 1) {
            return '到付';
        } else {
            return '混合付款';
        }
    },
    doChoose () {
        this.setState({
            choose:!this.state.choose,
        });
    },
    render () {
        const { data } = this.state;
        const { stateList } = data;
        return (
            <View style={styles.container}>
                <View style={styles.containerReceive}>
                    <Text style={styles.textName}> 收货人：{!data.receiverName ? '暂无' : data.receiverName}</Text>
                    <Text style={styles.textName}> 收货人电话:{!data.receiverPhone ? '暂无' : data.receiverPhone}</Text>
                    <Text style={styles.textAddress}> 到货地：{data && data.endPoint} </Text>
                </View>
                <View style={styles.containerMarket}>
                    <View style={styles.containerMarketLeft}>
                        <Image style={styles.imageLogo}
                            source={app.img.order_logo}
                            resizeMode='stretch'
                            />
                        <Text style={styles.textMarket}>{data.shop ? data.shop.name : data.agent ? data.agent.name : ''}</Text>
                    </View>
                    <Text style={styles.textGetState}>{stateList ? OS.MAP[stateList[0].state] : ''}</Text>
                </View>
                <View style={styles.containerDetails}>
                    <Image style={styles.imageGoods}
                        resizeMode='stretch'
                        defaultSource={app.img.home_order}
                        source={{ uri:data.photo }}
                        />
                    <View style={styles.goodsDetails}>
                        <View style={styles.goodsDetailsTop}>
                            <View style={styles.detailsTopLeft}>
                                <Text style={styles.detailsAddress}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{data.shop ? data.shop.address : data.agent ? data.agent.address : ''}</Text>
                                <Text>-></Text>
                                <Text style={styles.detailsAddress}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{data.endPoint}</Text>
                            </View>
                            <View style={styles.detailsTopRight}>
                                <Text style={styles.moneyLeft}>运费：</Text>
                                <Text style={styles.moneyRight}>¥{data.realFee}</Text>
                            </View>
                        </View>
                        <Text style={styles.textDetailsItem}>重量：{data.size}m³</Text>
                        <Text style={styles.textDetailsItem}>方量：{data.weight}吨</Text>
                        <Text style={styles.textDetailsItem}>件数：{data.totalNumbers}件</Text>
                        <Text style={styles.textDetailsItem}>下单时间：{moment(data.createTime).format('YYYY-MM-DD')}</Text>
                    </View>
                </View>
                <View style={styles.containerWhite} />
                <View style={styles.containerNumber}>
                    <Text style={styles.textDate}>创建时间：{data.createTime}</Text>
                </View>
                <View style={styles.containerInfo}>
                    <Text style={styles.textInfo}>运费：¥{data.realFee}</Text>
                    <Text style={styles.textInfo}>指定收款：¥{data.totalDesignatedFee}</Text>
                    <Text style={styles.textInfo}>代收货款：¥{data.proxyCharge}</Text>
                    <Text style={styles.textInfo}>
                        支付方式：{this.toPayMode(data.payMode)}
                    </Text>
                    <Text style={styles.textTotal}>需支付：¥{app.utils.N(data.totalDesignatedFee + data.proxyCharge)}</Text>
                </View>
                <TouchableOpacity onPress={this.toLogisticsDetail}>
                    <View style={styles.logInfo}>
                        <Text style={styles.textLogInfo}>物流信息</Text>
                        <View style={styles.logInfoRight}>
                            <Text style={styles.textLogInfoRight}>{stateList ? OS.MAP[stateList[0].state] : ''}</Text>
                            <Image
                                style={styles.imageCommon_go}
                                resizeMode='stretch'
                                source={app.img.common_go}
                                />
                        </View>
                    </View>
                </TouchableOpacity>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    containerTop:{
        height:60,
        backgroundColor:'#c81622',
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'center',
    },
    imageTop:{
        height:32,
        width:27,
    },
    textTop:{
        fontSize:12,
        fontWeight:'900',
        marginLeft:20,
        color:'#ffffff',
    },
    containerReceive:{
        marginTop:8,
        paddingLeft:10,
        height:55,
        backgroundColor:'#ffffff',
        justifyContent: 'center',
    },
    textAddress:{
        fontSize:12,
        marginBottom:3,
        color:'#333333',
        flex:1,
    },
    textName:{
        fontSize:12,
        color:'#333333',
        flex:1,
    },
    containerMarket:{
        height:25,
        marginTop:8,
        backgroundColor:'#ffffff',
        alignItems: 'center',
        justifyContent: 'space-between',
        flexDirection: 'row',
    },
    containerMarketLeft:{
        flexDirection: 'row',
    },
    imageLogo:{
        height:13,
        width:16.5,
        marginLeft:10,
    },
    textMarket:{
        marginLeft:10,
        color:'#333333',
        fontSize:12,
    },
    textGetState:{
        color:'#00a117',
        marginRight:10,
        fontSize:12,
    },
    containerDetails:{
        height:105,
        backgroundColor:'#f8f8f8',
        flexDirection: 'row',
    },
    containerWhite:{
        height:8,
        backgroundColor:'#ffffff',
    },
    imageGoods:{
        height:75,
        width:75,
        marginLeft:12,
        marginTop:15,
    },
    goodsDetails:{
        marginLeft:8,
        height:95,
        paddingBottom:8,
        justifyContent: 'center',
    },
    goodsDetailsTop:{
        width:sr.w - 99,
        flexDirection: 'row',
        justifyContent: 'space-between',
    },
    detailsTopLeft:{
        flexDirection: 'row',
        alignItems: 'center',
        marginTop:15,
    },
    detailsTopRight:{
        flexDirection: 'row',
        alignItems: 'center',
        marginTop:15,
    },
    detailsAddress:{
        fontSize:14,
        width:80,
        color:'#333333',
        flexDirection: 'row',

    },
    moneyLeft:{
        fontSize:12,
        color:'#333333',
    },
    moneyRight:{
        fontSize:13,
        fontWeight:'500',
        color:'#f01114',
        marginRight:10,
    },
    textDetailsItem:{
        marginTop:2,
        fontSize:12,
        color:'#888888',
    },
    containerNumber:{
        height:50,
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingLeft:10,
        justifyContent: 'center',
    },
    textNumber:{
        color:'#333333',
        marginBottom:3,
        fontSize:12,
    },
    textDate:{
        color:'#888888',
        marginTop:3,
        fontSize:14,
    },
    containerInfo:{
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingLeft:10,
        paddingBottom:5,
        paddingTop:5,
    },
    textInfo:{
        color:'#888888',
        fontSize:12,
        marginTop:5,
        height:15,
    },
    textTotal:{
        marginTop:5,
        fontSize:12,
        color:'#333333',
        fontWeight:'400',
        height:15,
    },
    logInfo:{
        height:40,
        backgroundColor:'#ffffff',
        marginTop:8,
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textLogInfo:{
        color:'#888888',
        fontSize:14,
        marginLeft:10,
    },
    logInfoRight:{
        marginRight:10,
        flexDirection: 'row',
        alignItems: 'center',
    },
    textLogInfoRight:{
        fontSize:14,
        color:'#333333',
        marginRight:5,
    },
    imageCommon_go:{
        width: 8,
        height: 15,
        marginRight: 7,
    },
    choose:{
        marginLeft:10,
        flexDirection: 'row',
        alignItems: 'center',
        flex:1,
    },
    imageChoose:{
        height:15,
        width:15,
    },
    btnReceive:{
        height:45,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        borderRadius:4,
        marginTop:30,
    },
    btnReceiveText:{
        fontSize:16,
        fontWeight:'500',
    },
    isReachPay:{
        backgroundColor:'#f4f4f4',
        width:sr.w,
        height:45,
    },
});

```

* PDClient/project/App/modules/receivePoint/order/ReceviePointOrderList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

import _ from 'lodash';

const { PageList, Button, MessageBox, DImage } = COMPONENTS;
const ReceiveOrderDetail = require('./ReceiveOrderDetail');
const LogisticsDetail = require('../../common/LogisticsDetail');
const PayPassword = require('../../common/PayPassword');
const PayInfo = require('../../common/PayInfo');

let money = 0;
let totalCount = 0;

module.exports = React.createClass({
    getInitialState () {
        return {
            paid: false,
            list:[],
            data: {},
            orderIdList: [],
            billList: [],
            orderBillList: [],
        };
    },
    getOrderDetail (obj, index) {
        app.push({
            component: ReceiveOrderDetail,
            passProps: {
                data: obj,
            },
        });
    },
    doLookTransport (obj) {
        app.push({
            component : LogisticsDetail,
            passProps:{
                obj,
            },
        });
    },
    doPrint (obj, rowId) {
        app.showModal(
            <MessageBox
                onConfirm={() => {
                    const param = {
                        userId: app.personal.info.userId,
                        orderIdList: [obj.id],
                    };
                    POST(app.route.ROUTE_PRINT_ORDER_LIST_BILL, param, this.doPrintSuccess, false);
                }}
                content={'确认打印收据'}
                width={sr.s(250)} />
        );
    },
    doPrintSuccess (data) {
        const { success, msg } = data;
        if (success) {
            this.listView.refresh();
            this.setState({
                orderBillList: [],
                billList: [],
            });
        } else {
            Toast(msg);
        }
    },
    doPay (obj, rowId) {
        app.showModal(
            <MessageBox
                onConfirm={() => {
                    const param = {
                        userId: app.personal.info.userId,
                        orderIdList: [obj.id],
                    };
                    POST(app.route.ROUTE_CONFIRM_CACH_PAYED_FOR_ORDER_LIST, param, this.doPaySuccess, false);
                }}
                content={'确认已现金收款'}
                width={sr.s(250)} />
        );
    },
    doPaySuccess (data) {
        const { success, msg } = data;
        if (success) {
            this.setState({
                orderIdList: [],
                list: [],
            });
            this.listView.refresh();
        } else {
            Toast(msg);
        }
    },
    renderSeparator (sectionID, rowId) {
        return (
            <View style={styles.separator} key={sectionID + rowId} />
        );
    },
    doChoose (obj, rowId) {
        const { list, orderIdList, billList, orderBillList } = this.state;
        const { type } = this.props;
        if (type == 'topay') {
            list[rowId].selected = !list[rowId].selected;
            if (_.findIndex(orderIdList, o => o == obj.id) == -1) {
                orderIdList.push(obj.id);
            } else {
                this.setState({ orderIdList: _.reject(orderIdList, o => o == obj.id) });
            }
            this.setState({
                list,
            });
            this.doTotalCalculation();
        } else {
            billList[rowId].selected = !billList[rowId].selected;
            if (_.findIndex(orderBillList, o => o == obj.id) == -1) {
                orderBillList.push(obj.id);
            } else {
                this.setState({ orderBillList: _.reject(orderBillList, o => o == obj.id) });
            }
            this.setState({
                billList,
            });
            this.doTotalCalculationBill();
        }
    },
    doSelectedBillOrderAll () {
        let { billList, orderBillList } = this.state;
        if (_.every(billList, o => o.selected == true)) {
            _.forEach(billList, o => { o.selected = false; });
            _.forEach(billList, o => {
                this.setState({ orderBillList: [] });
            });
        } else {
            _.forEach(billList, o => { o.selected = true; });
            _.forEach(billList, o => { orderBillList.push(o.item.id); });
            this.setState({
                orderBillList: _.uniq(orderBillList),
            });
        }
        this.setState({ billList });
        this.doTotalCalculationBill();
    },
    doPrintBillList () {
        const { orderBillList, billList } = this.state;
        if (orderBillList.length == 0) {
            Toast('请选择您要打印的货单');
            return;
        }
        if (!(_.find(billList, o => o.item.stateList[0].state == OS.OS_READY_PRINT_ORDER) || _.find(billList, o => o.item.stateList[0].state == OS.OS_READY_DEDUCT_AND_PRINT_ORDER))) {
            Toast('请选择待打印收据的订单');
            return;
        }
        app.showModal(
            <MessageBox
                onConfirm={() => { this.doPrintBill(); }}
                content={'确认打印收据' + totalCount + '单'}
                width={sr.s(250)} />
        );
    },
    doPrintBill () {
        const { orderBillList } = this.state;
        const param = {
            userId : app.personal.info.userId,
            orderIdList : orderBillList,
        };
        POST(app.route.ROUTE_PRINT_ORDER_LIST_BILL, param, this.doPrintBillSuccess, false);
    },
    doPrintBillSuccess (data) {
        if (data.success) {
            Toast('打印成功');
            this.listView.refresh();
            this.setState({ orderBillList : [], billList : [] });
            totalCount = 0;
        } else {
            Toast(data.msg);
        }
    },
    doTotalCalculationBill () {
        const { billList } = this.state;
        totalCount = _.filter(billList, (o) => o.selected == true).length;
    },
    doSelectedAll () {
        let { list, orderIdList } = this.state;
        if (_.every(list, o => o.selected == true)) {
            _.forEach(list, o => { o.selected = false; });
            _.forEach(list, o => {
                this.setState({ orderIdList: [] });
            });
        } else {
            _.forEach(list, o => { o.selected = true; });
            _.forEach(list, o => { orderIdList.push(o.item.id); });
            this.setState({
                orderIdList: _.uniq(orderIdList),
            });
        }
        this.setState({ list });
        this.doTotalCalculation();
    },
    doTotalCalculation () {
        const { list } = this.state;
        money = 0;
        _.filter(list, (o) => o.selected == true).forEach((item, i) => {
            money = money + item.item.needPayInsuanceFee + item.item.needPayTransportFee;
        });
    },
    doPayOrderList () {
        const { orderIdList, list } = this.state;
        if (orderIdList.length == 0) {
            Toast('请选择您要支付的货单');
            return;
        }
        if (!(_.find(list, o => o.item.stateList[0].state == OS.OS_READY_PAYMENT))) {
            Toast('请选择待支付的订单');
            return;
        }
        app.showModal(
            <MessageBox
                onConfirm={() => { this.doConfirmPay(); }}
                content={'确认已现金收款'}
                width={sr.s(250)} />
        );
    },
    doConfirmPay () {
        const { orderIdList } = this.state;
        const param = {
            userId : app.personal.info.userId,
            orderIdList,
        };
        POST(app.route.ROUTE_CONFIRM_CACH_PAYED_FOR_ORDER_LIST, param, this.doConfirmPaySuccess, false);
    },
    doConfirmPaySuccess (data) {
        if (data.success) {
            Toast('支付成功');
            this.listView.refresh();
            this.setState({ orderIdList : [], list : [] });
            money = 0;
        } else {
            Toast(data.msg);
        }
    },
    renderRow (obj, rowId) {
        const { stateList } = obj;
        const { type } = this.props;
        const { paid, list, billList } = this.state;
        if (type == 'topay') {
            if (!_.find(list, (o) => o.index == rowId)) {
                list.push({ selected:false, index:rowId, item:obj });
            }
        }
        if (type == 'toprintbill') {
            if (!_.find(billList, (o) => o.index == rowId)) {
                billList.push({ selected:false, index:rowId, item:obj });
            }
        }
        return (
            <View style={styles.row}>
                <View style={styles.rowTopContainer}>
                    <DImage
                        resizeMode='stretch'
                        source={app.img.order_logo}
                        style={styles.topImage} />
                    <Text style={styles.topLeftText}>{obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : ''}</Text>
                    <Text style={styles.topRightText}>{stateList ? OS.MAP[stateList[0].state] : ''}</Text>
                </View>
                <View style={styles.middle}>
                    {
                        type == 'topay' &&
                        <TouchableOpacity onPress={this.doChoose.bind(null, obj, rowId)}
                            style={styles.btnChoose}>
                            <Image
                                resizeMode='stretch'
                                source={!list[rowId].selected ? app.img.order_no_choose : app.img.order_choose}
                                style={styles.choose} />
                        </TouchableOpacity>
                    }
                    {
                        type == 'toprintbill' &&
                        <TouchableOpacity onPress={this.doChoose.bind(null, obj, rowId)}
                            style={styles.btnChoose}>
                            <Image
                                resizeMode='stretch'
                                source={!billList[rowId].selected ? app.img.order_no_choose : app.img.order_choose}
                                style={styles.choose} />
                        </TouchableOpacity>
                    }
                    <TouchableOpacity style={styles.rowMiddleContainer} onPress={this.getOrderDetail.bind(null, obj, rowId)}>
                        <DImage
                            resizeMode='stretch'
                            defaultSource={app.img.common_default}
                            source={{ uri: obj.photo }}
                            style={styles.middleImage} />
                        <View style={styles.middleRight}>
                            <View style={styles.pointContainer}>
                                <View style={styles.placeContainer}>
                                    <Text style={styles.pointName}
                                        numberOfLines={1}
                                        ellipsizeMode='tail'>{obj.shop ? obj.shop.address : obj.agent ? obj.agent.address : ''}</Text>
                                    <Text style={styles.pointText}>→</Text>
                                    <Text style={styles.pointName}
                                        numberOfLines={1}
                                        ellipsizeMode='tail'>{obj.endPoint}</Text>
                                </View>
                                <Text style={styles.transformFee}>{'运费：'}<Text style={styles.money}>￥{obj.needPayInsuanceFee + obj.needPayTransportFee}</Text></Text>
                            </View>
                            <Text style={styles.commonInfo}>重量：{obj.weight}吨</Text>
                            <Text style={styles.commonInfo}>方量：{obj.size}m³</Text>
                            <Text style={styles.commonInfo}>件数：{obj.totalNumbers}件</Text>
                            <Text style={styles.commonInfo}>下单时间：{obj.createTime}</Text>
                        </View>
                    </TouchableOpacity>
                </View>
                <View style={styles.rowButtomContainer}>
                    {
                        stateList[0].state == OS.OS_READY_PAYMENT ?
                            <Button
                                style={styles.lookTransport}
                                disable={paid}
                                onPress={this.doPay.bind(null, obj, rowId)}
                                textStyle={styles.btnTransportText}>
                                {!paid ? '支付' : '已支付'}
                            </Button>
                        :
                        (stateList[0].state == OS.OS_READY_PRINT_ORDER || stateList[0].state == OS.OS_READY_DEDUCT_AND_PRINT_ORDER) ?
                            <Button
                                onPress={this.doPrint.bind(null, obj)}
                                style={styles.lookTransport}
                                textStyle={styles.btnTransportText}>
                                {'打印收据'}
                            </Button>
                        :
                        stateList[0].state >= OS.OS_ON_THE_WAY ?
                            <Button
                                onPress={this.doLookTransport.bind(null, obj)}
                                style={styles.lookTransport}
                                textStyle={styles.btnTransportText}>
                                {'查看物流'}
                            </Button>
                        :
                            <View />
                    }
                    <Button onPress={this.getOrderDetail.bind(null, obj, rowId)} style={styles.lookDetail} textStyle={styles.btnDetallText}>{'查看详情'}</Button>
                </View>
            </View>
        );
    },
    render () {
        const { index, tabIndex, type, data } = this.props;
        const { list, orderIdList, billList, orderBillList } = this.state;
        const status = list.length != 0 && _.every(list, (o) => o.selected == true);
        const billStatus = billList.length != 0 && _.every(billList, (o) => o.selected == true);
        return (
            <View style={index === tabIndex ? styles.container : { left:-sr.tw, top:0, position:'absolute' }}>
                <View style={styles.separator} />
                {
                    !!data &&
                    <PageList
                        ref={(ref) => { this.listView = ref; }}
                        renderRow={this.renderRow}
                        renderSeparator={this.renderSeparator}
                        listParam={{ userId: app.personal.info.userId, type: type }}
                        listName={type + '.list'}
                        list={data.list}
                        listUrl={app.route.ROUTE_GET_ORDERS}
                        autoLoad={false}
                        refreshEnable
                        />
                }
                {
                    type == 'topay' &&
                    <View style={styles.selectedAllContainer}>
                        <View style={styles.totalLeft}>
                            <TouchableOpacity onPress={this.doSelectedAll}
                                style={styles.selectedAllBtn}>
                                <Image style={styles.chooseAll}
                                    source={status ? app.img.order_choose : app.img.order_no_choose} />
                                <Text style={styles.selectedAllText}>全选</Text>
                            </TouchableOpacity>
                            <Text style={styles.totalText}>总计</Text>
                            <Text style={styles.totalItemRight}>{money}元</Text>
                        </View>
                        <TouchableOpacity onPress={this.doPayOrderList}
                            style={styles.pay}>
                            <Text style={styles.payText}>现金支付</Text>
                        </TouchableOpacity>
                    </View>
                }
                {
                    type == 'toprintbill' &&
                    <View style={styles.selectedAllContainer}>
                        <View style={styles.totalLeft}>
                            <TouchableOpacity onPress={this.doSelectedBillOrderAll}
                                style={styles.selectedAllBtn}>
                                <Image style={styles.chooseAll}
                                    source={billStatus ? app.img.order_choose : app.img.order_no_choose} />
                                <Text style={styles.selectedAllText}>全选</Text>
                            </TouchableOpacity>
                            <Text style={styles.totalText}>总计</Text>
                            <Text style={styles.totalItemRight}>{totalCount}单</Text>
                        </View>
                        <TouchableOpacity onPress={this.doPrintBillList}
                            style={styles.pay}>
                            <Text style={styles.payText}>批量打印收据</Text>
                        </TouchableOpacity>
                    </View>
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f4f4f4',
    },
    row: {
        paddingVertical: 2,
        paddingHorizontal: 2,
    },
    separator: {
        backgroundColor: '#f4f4f4',
        height: 8,
    },
    rowTopContainer: {
        paddingHorizontal:10,
        paddingVertical:5,
        backgroundColor:'white',
        flexDirection: 'row',
    },
    topImage: {
        width: 17,
        height:17,
    },
    topLeftText: {
        marginLeft:10,
        flex:1,
        fontSize: 14,
    },
    topRightText: {
        flex:1,
        fontSize: 14,
        textAlign:'right',
        color:'#f51b1a',
    },
    middle:{
        flexDirection:'row',
        height:100,
        alignItems:'center',
        width:sr.w,
        backgroundColor:'#f8f8f8',
    },
    rowMiddleContainer: {
        flexDirection: 'row',
        alignItems: 'center',
        paddingVertical:11,
        height:100,
    },
    btnChoose:{
        height:100,
        width:20,
        marginRight:3,
        justifyContent:'center',
        alignItems:'center',
    },
    choose:{
        height:12,
        width:12,
    },
    middleImage: {
        width: 80,
        height:80,
        borderRadius: 10,
    },
    middleRight: {
        paddingLeft: 10,
        width:sr.w - 105,
    },
    money: {
        fontSize: 12,
        color:'#f51b1a',
    },
    commonInfo: {
        fontSize: 13,
        color:'#888888',
    },
    pointName: {
        fontSize: 13,
        maxWidth: 60,
    },
    pointText:{
        marginLeft:5,
        marginRight:10,
    },
    transformFee: {
        fontSize: 12,
        color: 'gray',
        marginRight:10,
        textAlign:'right',
    },
    pointContainer: {
        flexDirection: 'row',
        marginBottom:4,
    },
    placeContainer: {
        flex:2,
        flexDirection: 'row',
    },
    rowButtomContainer: {
        backgroundColor:'white',
        flexDirection: 'row',
        justifyContent: 'flex-end',
        paddingVertical:5,
    },
    lookTransport: {
        height:20,
        width:73,
        borderRadius:5,
        borderColor:'#ff6200',
        borderWidth:1,
        backgroundColor:'white',
    },
    lookDetail: {
        height:20,
        width:73,
        borderRadius:5,
        borderColor:'#007ef7',
        borderWidth:1,
        marginLeft:20,
        marginRight:10,
        backgroundColor:'white',
    },
    btnTransportText:{
        color:'#ff6200',
        fontSize:12,
        fontWeight:'300',
    },
    btnDetallText:{
        color:'#007ef7',
        fontSize:12,
        fontWeight:'300',
    },
    totalLeft:{
        flex:7,
        flexDirection:'row',
        alignItems:'center',
    },
    selectedAllContainer:{
        height:50,
        width:sr.w,
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
        alignItems:'center',
    },
    selectedAllBtn:{
        flexDirection:'row',
        alignItems:'center',
        height:40,
    },
    chooseAll:{
        height:12,
        width:12,
        marginLeft:5,
    },
    selectedAllText:{
        color:'#333333',
        fontSize:12,
        marginLeft:10,
    },
    totalText:{
        color:'#333333',
        fontSize:15,
        fontWeight:'500',
        marginLeft:20,
    },
    totalItem:{
        width:45,
        fontSize:12,
        color:'#f01823',
        textAlign:'right',
        marginLeft:5,
    },
    totalItemRight:{
        width:50,
        fontSize:12,
        color:'#f01823',
        textAlign:'right',
        marginRight:5,
    },
    pay:{
        flex:3,
        marginLeft:10,
        height:50,
        width:150,
        justifyContent:'center',
        alignItems:'center',
        backgroundColor:'#c81622',
    },
    payText:{
        fontSize:14,
        fontWeight:'500',
        color:'#FFFFFF',
        letterSpacing:2,
    },
});

```

* PDClient/project/App/modules/receivePoint/order/index.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

const ReceviePointOrderList = require('./ReceviePointOrderList');
const TimerMixin = require('react-timer-mixin');

module.exports = React.createClass({
    mixins: [TimerMixin],
    statics: {
        title: '货单',
    },
    getInitialState () {
        return {
            tabIndex: 0,
            data : [],
        };
    },
    componentWillMount () {
        this.getOrders();
    },
    onWillFocus () {
        this.setTimeout(() => {
            app.getCurrentRoute().title = '货单';
            app.getCurrentRoute().rightButton = null;
            app.getCurrentRoute().leftButton = null;
            app.forceUpdateNavbar();
        }, 200);
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    getOrders () {
        const param = {
            userId : app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_ORDERS, param, this.getOrdersSuccess, false);
    },
    getOrdersSuccess (data) {
        if (data.success) {
            this.setState({ data : data.context });
        }
    },
    render () {
        const menus = [{ name:'待付款', type:'topay' }, { name:'待打印收据', type:'toprintbill' }, { name:'待发货', type:'tosend' }, { name:'运输中', type:'onway' }, { name:'已完成', type:'success' }];
        const { tabIndex, data } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.tabContainer}>
                    {
                        menus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={this.changeTab.bind(null, i)}
                                style={styles.touchTab}>
                                <View style={styles.tabButton}>
                                    <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                        {item.name}
                                    </Text>
                                    <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                </View>
                            </TouchableHighlight>
                        ))
                    }
                </View>
                {
                    menus.map((item, i) => (
                        <ReceviePointOrderList key={i} index={i} type={item.type} tabIndex={tabIndex} data={data && data[item.type]} />
                    ))
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor:'#FFFFFF',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonCenter: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonRight: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        borderRadius: 9,
    },
    tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```

* PDClient/project/App/modules/receivePoint/placeOrder/LogisticsRanking.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableOpacity,
} = ReactNative;

const BaidumapSelectPosition = require('../../baidumap');

module.exports = React.createClass({
    statics: {
        title: '填单',
        rightButton: { title: '保存', handler: () => { app.scene.prePlaceOrder&&app.scene.prePlaceOrder(); } },
    },
    prePlaceOrder () {

    },
    getInitialState () {
        return {
            set:0,
            position:'位置',
        };
    },
    toPrice () {
        this.setState({ set:1 });
    },
    toTime () {
        this.setState({ set:2 });
    },
    todistance () {
        this.setState({ set:3 });
    },
    toMap () {
        app.push({
            component: BaidumapSelectPosition,
            passProps:{
                confirm:(marker) => {
                    this.setState({ position:marker.title });
                },
            },
        });
    },
    render () {
        return (
            <View style={styles.container}>
                <View style={styles.containerTop}>
                    <Text style={styles.topText}>物流排名</Text>
                    <TouchableOpacity onPress={this.toMap}>
                        <Text style={styles.topTextRight}>{this.state.position}</Text>
                    </TouchableOpacity>
                </View>
                <View style={styles.top}>
                    <TouchableOpacity onPress={this.toPrice}>
                        <Text style={this.state.set == 1 ? styles.redText : styles.itemText}>价格最低</Text>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={this.toTime}>
                        <Text style={this.state.set == 2 ? styles.redText : styles.itemText}>时间最短</Text>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={this.todistance}>
                        <Text style={this.state.set == 3 ? styles.redText : styles.itemText}>距离最短</Text>
                    </TouchableOpacity>
                </View>
                <View style={styles.info}>
                    <View style={styles.shop}>
                        <Text style={styles.shopInfo}>六面通物流公司 - 四面通物流大超市石板店</Text>
                        <Text style={styles.distance}>据我2km</Text>
                    </View>
                    <View style={styles.infoDetails}>
                        <View style={styles.unit}>
                            <Text style={styles.unitPriceNum}>¥20.00/吨</Text>
                            <Text style={styles.unitPrice}>运费单价</Text>
                        </View>
                        <View style={styles.unit}>
                            <Text style={styles.unitPriceNum}>¥150.00/吨</Text>
                            <Text style={styles.unitPrice}>送货上门</Text>
                        </View>
                        <View style={styles.unit}>
                            <Text style={styles.time}>2天</Text>
                            <Text style={styles.EstimateTime}>预计用时</Text>
                        </View>
                    </View>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    containerTop:{
        flexDirection: 'row',
        justifyContent:'space-between',
    },
    topText:{
        fontSize:16,
        marginTop:16,
        marginLeft:12,
        marginBottom:10,
        color:'#333333',
    },
    topTextRight:{
        marginRight:10,
        fontSize:16,
        marginTop:16,
        marginBottom:10,
        color:'#333333',
    },
    top:{
        height:35,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        justifyContent:'space-around',
        alignItems:'center',
    },
    itemText:{
        fontSize:13,
        color:'#333333',
    },
    redText:{
        fontSize:13,
        color:'#f11821',
    },
    info:{
        marginTop:1,
        height:95,
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
    },
    shop:{
        flexDirection: 'row',
    },
    shopInfo:{
        fontSize:14,
        color:'#333333',
        marginLeft:10,
    },
    distance:{
        fontSize:12,
        color:'#ff9517',
        marginTop:2,
        marginLeft:8,
    },
    infoDetails:{
        marginTop:10,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        justifyContent:'space-around',
    },
    unit:{
        justifyContent:'center',
        alignItems:'center',
    },
    unitPriceNum:{
        fontSize:15,
        color:'#f11821',
        fontWeight:'500',
        marginBottom:3,
    },
    unitPrice:{
        fontSize:12,
        color:'#333333',
        marginTop:3,
    },
    time:{
        paddingTop:2,
        fontSize:15,
        color:'#333333',
        marginBottom:3,
    },
    EstimateTime:{
        fontSize:12,
        color:'#333333',
        marginTop:3,
    },
});

```

* PDClient/project/App/modules/receivePoint/placeOrder/OrderDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

const { Button } = COMPONENTS;
const moment = require('moment');

module.exports = React.createClass({
    mixins: [SceneMixin],
    statics: {
        title: '货单信息',
    },
    getInitialState () {
        const { data } = this.props;
        return {
            data,
        };
    },
    componentWillReceiveProps (nextProps) {
        this.setState({ data: nextProps.data });
    },
    render () {
        const { data } = this.state;
        const { showConfirmPayBox, doPrintReceipt, doPrintQrcode } = this.props;
        return (
            <View style={styles.container}>

                <View style={styles.containerDetails}>
                    <Image style={styles.imageGoods}
                        resizeMode='stretch'
                        defaultSource={app.img.home_order}
                        source={{ uri:data.photo }}
                        />
                    <View style={styles.goodsDetails}>
                        <Text style={styles.textDetailsItem}>重量：{data.size}m³</Text>
                        <Text style={styles.textDetailsItem}>方量：{data.weight}吨</Text>
                        <Text style={styles.textDetailsItem}>件数：{data.totalNumbers}件</Text>
                        <Text style={styles.textDetailsItem}>下单时间：{moment(data.createTime).format('YYYY-MM-DD')}</Text>
                    </View>
                </View>
                <View style={styles.containerWhite} />
                <View>
                    <Text style={styles.textTotal}>{'发货人 : ' + data.senderName}</Text>
                    <Text style={styles.textTotal}>{'发货人电话 : ' + data.senderPhone}</Text>
                    <Text style={styles.textTotal}>{'收货人 : ' + data.receiverName}</Text>
                    <Text style={styles.textTotal}>{'收货人电话 : ' + data.receiverPhone}</Text>
                    {
                        data.isCityDistribute?
                        <Text style={styles.textTotal}>{'同城配送地址 : ' + data.endPoint}</Text>
                        :
                        <View>
                            <Text style={styles.textTotal}>{'到货地 : ' + data.endPoint}</Text>
                            {
                                data.isSendDoor&& <Text style={styles.textTotal}>{'送货上门地址 : ' + data.sendDoorEndPoint}</Text>
                            }
                        </View>
                    }
                </View>
                <View style={styles.containerWhite} />
                <View style={styles.containerInfo}>
                    <Text style={styles.textInfo}>运费：¥{app.utils.N(data.needPayTransportFee) || 0}</Text>
                    <Text style={styles.textInfo}>指定收款：¥{app.utils.N(data.totalDesignatedFee) || 0}</Text>
                    <Text style={styles.textInfo}>代收货款：¥{app.utils.N(data.proxyCharge) || 0}</Text>
                    <Text style={styles.textInfo}>保险费：¥{app.utils.N(data.needPayInsuanceFee) || 0}</Text>
                    <Text style={styles.textTotal}>需支付：¥{app.utils.N(data.needPayTransportFee + data.needPayInsuanceFee) || 0}</Text>
                </View>
                {
                    data.state > OS.OS_READY_PRINT_BAR_CODE ? data.state == OS.OS_READY_PAYMENT ?
                        <Button onPress={showConfirmPayBox} style={styles.btnReceive} textStyle={styles.btnReceiveText}>
                        确认支付
                    </Button>
                    : <Button onPress={doPrintReceipt} style={styles.btnReceive} textStyle={styles.btnReceiveText}>
                        打印收据
                    </Button>
                    : <Button onPress={doPrintQrcode} style={styles.btnReceive} textStyle={styles.btnReceiveText}>
                        打印二维码
                    </Button>
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    containerTop:{
        height:60,
        backgroundColor:'#c81622',
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'center',
    },
    imageTop:{
        height:32,
        width:27,
    },
    textTop:{
        fontSize:12,
        fontWeight:'900',
        marginLeft:20,
        color:'#ffffff',
    },
    containerDetails:{
        height:105,
        backgroundColor:'#f8f8f8',
        flexDirection: 'row',
    },
    containerWhite:{
        height:8,
        backgroundColor:'#ffffff',
    },
    imageGoods:{
        height:75,
        width:75,
        marginLeft:12,
        marginTop:15,
    },
    goodsDetails:{
        marginLeft:8,
        height:95,
        marginTop:3,
        justifyContent: 'center',
    },
    textDetailsItem:{
        marginTop:4,
        fontSize:12,
        color:'#888888',
    },
    containerInfo:{
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingBottom:5,
        paddingTop:5,
    },
    textInfo:{
        color:'#888888',
        fontSize:12,
        marginTop:5,
        height:15,
        marginLeft:10,
    },
    textTotal:{
        marginTop:5,
        fontSize:12,
        color:'#333333',
        fontWeight:'400',
        height:15,
        marginLeft:10,
    },
    logInfo:{
        height:40,
        backgroundColor:'#ffffff',
        marginTop:8,
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
    },
    textLogInfo:{
        color:'#888888',
        fontSize:14,
        marginLeft:10,
    },
    logInfoRight:{
        marginRight:10,
        flexDirection: 'row',
        alignItems: 'center',
    },
    textLogInfoRight:{
        fontSize:14,
        color:'#333333',
        marginRight:5,
    },
    imageCommon_go:{
        width: 8,
        height: 15,
        marginRight: 7,
    },
    btnReceive:{
        height:45,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        borderRadius:4,
        marginTop:30,
    },
    btnReceiveText:{
        fontSize:16,
        fontWeight:'500',
    },
    isReachPay:{
        backgroundColor:'#f4f4f4',
        width:sr.w,
        height:45,
    },
});

```

* PDClient/project/App/modules/receivePoint/placeOrder/PlaceOrder.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    Image,
    TouchableOpacity,
    ScrollView,
} = ReactNative;

const { DImage, SelectRegionAddress, SelectShortAddress, ActionView, Button } = COMPONENTS;
const Camera = require('@remobile/react-native-camera');

module.exports = React.createClass({
    statics: {
        title: '填单',
    },
    doConfim () {
        const { sendDoorEndPointLastCode, sendDoorEndPoint, name, senderName, senderPhone, isInsuance, endPoint, receiverName, receiverPhone, weight, size, totalNumbers, photo, endPointLastCode, isSendDoor, isCityDistribute, proxyCharge, totalDesignatedFee } = this.state;
        if (photo == '') {
            Toast('请选择货物图片');
            return;
        }
        if (senderName == '') {
            Toast('发货人不能为空');
            return;
        }
        if (!app.utils.checkPhone(senderPhone)) {
            Toast('请填写正确的发货人手机号码');
            return;
        }
        if (name == '') {
            Toast('货物名称不能为空');
            return;
        }
        if (receiverName == '') {
            Toast('收货人不能为空');
            return;
        }
        if (!app.utils.checkPhone(receiverPhone)) {
            Toast('请填写正确的收货人手机号码');
            return;
        }
        if (!isCityDistribute && endPoint == '') {
            Toast('请选择收货地');
            return;
        }
        if (!app.utils.checkNum(weight)) {
            Toast('请填写货物重量');
            return;
        }
        if (!app.utils.checkNum(size)) {
            Toast('请填写货物体积');
            return;
        }
        if (!app.utils.checkIntNum(totalNumbers)) {
            Toast('请填写货物件数');
            return;
        }
        if (isCityDistribute && endPoint == '') {
            Toast('请选择同城配送地址');
            return;
        }
        if (isSendDoor && sendDoorEndPoint == '') {
            Toast('请选择送货上门地址');
            return;
        }
        if (proxyCharge !=0 && !app.utils.check2PointNum(proxyCharge)) {
            Toast('请填写正确代收货款');
            return;
        }
        if (totalDesignatedFee !=0 && !app.utils.check2PointNum(totalDesignatedFee)) {
            Toast('请填写正确指定收款');
            return;
        }
        this.props.confirm(this.state);
    },
    getInitialState () {
        const { info } = this.props;
        if (info) {
            return {
                senderName:info.senderName || '',
                senderPhone:info.senderPhone || '',
                photo:info.photo || '',
                receiverName:info.receiverName || '',
                receiverPhone:info.receiverPhone || '',
                endPoint: info.endPoint || '',
                endPointLastCode:info.endPointLastCode || '',
                sendDoorEndPoint:info.sendDoorEndPoint || '',
                sendDoorEndPointLastCode:info.sendDoorEndPointLastCode || '',
                weight:info.weight || '',
                name:info.name || '',
                size:info.size || '',
                totalNumbers:info.totalNumbers || '',
                proxyCharge:info.proxyCharge || '',
                totalDesignatedFee:info.totalDesignatedFee || '',
                isInsuance:info.isInsuance || false,
                isSendDoor: info.isSendDoor || false,
                isCityDistribute: info.isCityDistribute || false,
                isReachPay:info.isReachPay || false,
            };
        } else {
            return {
                senderName:'',
                senderPhone:'',
                photo:'',
                receiverName:'',
                receiverPhone:'',
                endPoint: '',
                endPointLastCode:'',
                sendDoorEndPoint:'',
                sendDoorEndPointLastCode:'',
                weight:'',
                name:'',
                size:'',
                totalNumbers:'',
                proxyCharge:0,
                totalDesignatedFee:0,
                isInsuance:false,
                isSendDoor:false,
                isCityDistribute:false,
                isReachPay:false,
            };
        }
    },
    setIsInsuance () {
        this.setState({
            isInsuance: !this.state.isInsuance,
        });
    },
    setIsCityDistribute () {
        this.setState({
            isCityDistribute: !this.state.isCityDistribute,
            endPoint:'',
            endPointLastCode:0,
            isSendDoor:false,
            sendDoorEndPoint:'',
            sendDoorEndPointLastCode:0
        });
    },
    setIsSendDoor () {
        this.setState({
            isSendDoor: !this.state.isSendDoor,
        });
    },
    toReachPay () {
        this.setState({
            isReachPay: !this.state.isReachPay,
        });
    },
    showEndPlace () {
        const { endPointLastCode } = this.state;
        app.push({
            component: SelectRegionAddress,
            title:'到货地',
            passProps:{
                endPointLastCode,
                type:4,
                confirmAddress:(endPoint, endPointLastCode) => {
                    if (endPointLastCode != this.state.endPointLastCode) {
                        this.setState({ sendDoorEndPointLastCode:0, sendDoorEndPoint:'', isSendDoor : false });
                    }
                    this.setState({ endPoint, endPointLastCode });
                },
            },
        });
    },
    showCityDistributeEndPlace () {
        const { endPointLastCode } = this.state;
        app.push({
            component: SelectShortAddress,
            title:'同城配送地址',
            passProps:{
                endPointLastCode:endPointLastCode,
                parentCode : app.personal.info.agent.addressRegionLastCode,
                confirmAddress:(endPoint, endPointLastCode) => {
                    this.setState({ endPoint, endPointLastCode });
                },
            },
        });
    },
    showSendDoorEndPlace () {
        const { endPointLastCode, sendDoorEndPointLastCode } = this.state;
        if (!endPointLastCode) {
            Toast('请先选择到货地');
            return;
        }
        app.push({
            component: SelectShortAddress,
            title:'送货上门地址',
            passProps:{
                endPointLastCode:sendDoorEndPointLastCode,
                parentCode : endPointLastCode,
                confirmAddress:(sendDoorEndPoint, sendDoorEndPointLastCode) => {
                    this.setState({ sendDoorEndPoint, sendDoorEndPointLastCode });
                },
            },
        });
    },
    doSelectPhoto () {
        Camera.getPicture((filePath) => {
            const uri = 'file://' + filePath;
            Image.getSize(uri, (width, height) => {
                this.setState({ photo: uri });
            });
        }, null, {
            quality: 100,
            allowEdit: false,
            sourceType:Camera.PictureSourceType.PHOTOLIBRARY,
            cameraDirection: Camera.Direction.FRONT,
            destinationType: Camera.DestinationType.FILE_URI,
        });
    },
    selectPicture () {
        app.closeModal();
        const options = {
            quality: 30,
            targetWidth: 240,
            targetHeight: 240,
            allowEdit: true,
            destinationType: Camera.DestinationType.FILE_URI,
            sourceType: Camera.PictureSourceType.PHOTOLIBRARY,
        };
        Camera.getPicture((filePath) => {
            this.uploadPhoto(filePath);
        }, () => {
            Toast('操作失败');
        }, options);
    },
    takePicture () {
        app.closeModal();
        const options = {
            quality: 30,
            allowEdit: true,
            targetWidth: 240,
            targetHeight: 240,
            cameraDirection: Camera.Direction.FRONT,
            destinationType: Camera.DestinationType.FILE_URI,
        };
        Camera.getPicture((filePath) => {
            this.uploadPhoto(filePath);
        }, () => {
            Toast('操作失败');
        }, options);
    },
    uploadPhoto (filePath) {
        this.setState({ headImgSource: { uri: filePath } });
        const options = {};
        options.fileKey = 'file';
        options.fileName = filePath.substr(filePath.lastIndexOf('/') + 1);
        options.mimeType = 'image/jpeg';
        options.params = {
            clientId:app.personal.info.userID,
        };
        this.uploadOn = true;
        UPLOAD(filePath, app.route.ROUTE_UPDATE_FILE, options, (progress) => console.log(progress),
        this.uploadSuccessCallback, this.uploadErrorCallback, true);
    },
    uploadSuccessCallback (data) {
        if (data.success) {
            const context = data.context;
            this.setState({ photo:context.url });
        } else {
            Toast('上传失败');
        }
        this.uploadOn = false;
    },
    uploadErrorCallback () {
        this.uploadOn = false;
        this.setState({ photo: app.personal.info.head });
    },
    doShowActionSheet () {
        app.showModal(<ActionView
            cancelText='取  消'
            onCancel={app.closeModal} >
            <ActionView.Button onPress={this.takePicture}>拍    照</ActionView.Button>
            <ActionView.Button onPress={this.selectPicture}>从相册选择</ActionView.Button>
        </ActionView>);
    },
    render () {
        const {
            sendDoorEndPointLastCode,
            sendDoorEndPoint,
            name,
            senderName,
            senderPhone,
            isInsuance,
            endPoint,
            receiverName,
            receiverPhone,
            weight, size,
            totalNumbers,
            photo,
            endPointLastCode,
            isSendDoor,
            isCityDistribute,
            isReachPay,
            proxyCharge,
            totalDesignatedFee,
        } = this.state;
        return (
            <View style={styles.container}>
                <ScrollView>
                    { photo == '' ?
                        <TouchableOpacity
                            onPress={this.doShowActionSheet}
                            style={styles.topImage}>
                            <DImage
                                defaultSource={app.img.order_upload_picture}
                                source={{ uri: photo }}
                                style={styles.photo} />
                            <Text style={styles.topImageText}>上传货物图片</Text>
                        </TouchableOpacity>
                        :
                        <TouchableOpacity
                            onPress={this.doShowActionSheet}
                            style={styles.topBigImage}>
                            <Text style={{ color:'#666666' }}>货物图片</Text>
                            <DImage
                                source={{ uri: photo }}
                                style={styles.bigPicture} />
                        </TouchableOpacity>
                    }
                    <View style={styles.itemStart}>
                        <Text style={styles.itemLeft}>发货人</Text>
                        <TextInput style={styles.inputText}
                            underlineColorAndroid='transparent'
                            placeholder='请输入发货人姓名'
                            defaultValue={senderName}
                            onChangeText={(text) => this.setState({ senderName: text })}
                            />
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.itemLeft}>发货人电话</Text>
                        <TextInput style={styles.inputText}
                            underlineColorAndroid='transparent'
                            keyboardType='numeric'
                            placeholder='请输入发货人电话'
                            defaultValue={senderPhone}
                            maxLength={11}
                            onChangeText={(text) => this.setState({ senderPhone: text })}
                            />
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.itemLeft}>货物名称</Text>
                        <TextInput style={styles.inputText}
                            underlineColorAndroid='transparent'
                            placeholder='请输入货物名称'
                            defaultValue={name}
                            onChangeText={(text) => this.setState({ name: text })}
                            />
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.itemLeft}>收货人</Text>
                        <TextInput style={styles.inputText}
                            underlineColorAndroid='transparent'
                            placeholder='请输入收货人姓名'
                            defaultValue={receiverName}
                            onChangeText={(text) => this.setState({ receiverName: text })}
                            />
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.itemLeft}>收货人电话</Text>
                        <TextInput style={styles.inputText}
                            underlineColorAndroid='transparent'
                            keyboardType='numeric'
                            placeholder='请输入收货人手机号码'
                            defaultValue={receiverPhone}
                            maxLength={11}
                            onChangeText={(text) => this.setState({ receiverPhone: text })}
                            />
                    </View>
                    <TouchableOpacity onPress={this.setIsCityDistribute}>
                        <View style={styles.itemStart}>
                            <Text style={styles.itemLeft}>是否同城配送</Text>
                            <View style={styles.itemPoint}>
                                <Text style={styles.itemRight}>{isCityDistribute ? '是' : '否'}</Text>
                                <Image
                                    source={app.img.common_go}
                                    style={styles.goStyle} />
                            </View>
                        </View>
                    </TouchableOpacity>
                    {
                        isCityDistribute ?
                            <TouchableOpacity
                                activeOpacity={0.6}
                                onPress={this.showCityDistributeEndPlace}>
                                <View style={styles.item}>
                                    <Text style={styles.itemLeft}>同城配送地址</Text>
                                    <View style={styles.itemPoint}>
                                        <Text style={[styles.itemRight, { paddingRight:5 }]}
                                            numberOfLines={1}
                                            ellipsizeMode='tail'>{endPoint}</Text>
                                        <Image
                                            source={app.img.common_go}
                                            style={styles.goStyle} />
                                    </View>
                                </View>
                            </TouchableOpacity>
                        :
                            <View>
                                <TouchableOpacity
                                    activeOpacity={0.6}
                                    onPress={this.showEndPlace}>
                                    <View style={styles.item}>
                                        <Text style={styles.itemLeft}>到货地</Text>
                                        <View style={styles.itemPoint}>
                                            <Text style={[styles.itemRight, { paddingRight:5 }]}
                                                numberOfLines={1}
                                                ellipsizeMode='tail'>{endPoint}</Text>
                                            <Image
                                                source={app.img.common_go}
                                                style={styles.goStyle} />
                                        </View>
                                    </View>
                                </TouchableOpacity>
                                <TouchableOpacity onPress={this.setIsSendDoor}>
                                    <View style={styles.item}>
                                        <Text style={styles.itemLeft}>送货上门</Text>
                                        <View style={styles.itemPoint}>
                                            <Text style={styles.itemRight}>{isSendDoor ? '是' : '否'}</Text>
                                            <Image
                                                source={app.img.common_go}
                                                style={styles.goStyle} />
                                        </View>
                                    </View>
                                </TouchableOpacity>
                                {
                                isSendDoor &&
                                <TouchableOpacity
                                    activeOpacity={0.6}
                                    onPress={this.showSendDoorEndPlace}>
                                    <View style={styles.item}>
                                        <Text style={styles.itemLeft}>送货上门地址</Text>
                                        <View style={styles.itemPoint}>
                                            <Text style={[styles.itemRight, { paddingRight:5 }]}
                                                numberOfLines={1}
                                                ellipsizeMode='tail'>{sendDoorEndPoint}</Text>
                                            <Image
                                                source={app.img.common_go}
                                                style={styles.goStyle} />
                                        </View>
                                    </View>
                                </TouchableOpacity>
                            }
                            </View>
                    }
                    <View style={styles.itemUnitTop}>
                        <Text style={styles.itemLeft}>重量</Text>
                        <TextInput style={styles.inputText}
                            underlineColorAndroid='transparent'
                            keyboardType='numeric'
                            placeholder='请输入货物重量'
                            defaultValue={weight + ''}
                            onChangeText={(text) => this.setState({ weight: text })}
                            />
                        <Text style={styles.unit}>吨</Text>
                    </View>
                    <View style={styles.itemUnit}>
                        <Text style={styles.itemLeft}>方量</Text>
                        <TextInput style={styles.inputTextSymbol}
                            underlineColorAndroid='transparent'
                            keyboardType='numeric'
                            placeholder='请输入货物方量'
                            defaultValue={size + ''}
                            onChangeText={(text) => this.setState({ size: text })}
                            />
                        <Text style={styles.unit}>m³</Text>
                    </View>
                    <View style={styles.itemUnit}>
                        <Text style={styles.itemLeft}>件数</Text>
                        <TextInput style={styles.inputText}
                            underlineColorAndroid='transparent'
                            keyboardType='numeric'
                            placeholder='请输入货物件数'
                            defaultValue={totalNumbers + ''}
                            onChangeText={(text) => this.setState({ totalNumbers: text })}
                            />
                        <Text style={styles.unit}>件</Text>
                    </View>

                    <View style={styles.itemUnitTop}>
                        <Text style={styles.itemLeft}>代收货款</Text>
                        <TextInput style={styles.inputText}
                            underlineColorAndroid='transparent'
                            keyboardType='numeric'
                            placeholder='0'
                            defaultValue={proxyCharge + ''}
                            onChangeText={(text) => this.setState({ proxyCharge : text })}
                            />
                        <Text style={styles.unit}>元</Text>
                    </View>
                    <View style={styles.itemUnit}>
                        <Text style={styles.itemLeft}>指定收款</Text>
                        <TextInput style={styles.inputText}
                            underlineColorAndroid='transparent'
                            keyboardType='numeric'
                            placeholder='0'
                            defaultValue={totalDesignatedFee + ''}
                            onChangeText={(text) => this.setState({ totalDesignatedFee : text })}
                            />
                        <Text style={styles.unit}>元</Text>
                    </View>
                    <TouchableOpacity onPress={this.toReachPay}>
                        <View style={styles.item}>
                            <Text style={styles.itemLeft}>是否到付</Text>
                            <View style={styles.itemPoint}>
                                <Text style={styles.itemRight}>{isReachPay ? '是' : '否'}</Text>
                                <Image
                                    source={app.img.common_go}
                                    style={styles.goStyle} />
                            </View>
                        </View>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={this.setIsInsuance}>
                        <View style={styles.item}>
                            <Text style={styles.itemLeft}>是否保价</Text>
                            <View style={styles.itemPoint}>
                                <Text style={styles.itemRight}>{isInsuance ? '是' : '否'}</Text>
                                <Image
                                    source={app.img.common_go}
                                    style={styles.goStyle} />
                            </View>
                        </View>
                    </TouchableOpacity>
                </ScrollView>
                <View style={styles.itemAddCar}>
                    <Button style={styles.btnAddPreOrder} textStyle={styles.btnAddPreOrderFont} onPress={this.doConfim}>确定</Button>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    topImage:{
        width:sr.w,
        height:65,
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent: 'center',
    },
    topBigImage:{
        height:100,
        width:sr.w,
        padding:12,
        flexDirection:'row',
        alignItems:'center',
        justifyContent:'space-between',
        backgroundColor:'#FFFFFF',
    },
    photo:{
        height:23,
        width:25,
    },
    bigPicture:{
        height:80,
        width:120,
    },
    topImageText:{
        marginTop:10,
        fontSize:12,
        color:'#888888',
    },
    itemStart:{
        height:40,
        marginTop:8,
        justifyContent: 'space-between',
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    item:{
        height:40,
        marginTop:1,
        justifyContent: 'space-between',
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    itemLeft:{
        flex:1,
        paddingLeft:12,
        fontSize:14,
        color:'#666666',
    },
    itemRight:{
        paddingRight:12,
        fontSize:14,
        color:'#333333',
        width:sr.w / 3 * 2,
        textAlign:'right',
    },
    itemPoint:{
        flexDirection: 'row',
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 12,
    },
    inputText:{
        flex:1,
        height:40,
        fontSize:14,
        width:sr.w / 2,
        marginRight:10,
        textAlign:'right',
    },
    itemUnitTop:{
        marginTop:8,
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    itemUnit:{
        marginTop:1,
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    unit:{
        paddingRight:12,
        fontSize:14,
        color:'#888888',
    },
    inputTextSymbol:{
        flex:1,
        height:40,
        fontSize:14,
        width:sr.w / 2,
        marginRight:7,
        textAlign:'right',
    },
    protocal_icon: {
        height: 18,
        width: 18,
        backgroundColor:'#87d84f',
    },
    btnAddPreOrder:{
        width:150,
        height:35,
        backgroundColor:'#FE9917',
        borderRadius: 4,
        alignSelf:'center',
        marginVertical:10,
    },
    btnAddPreOrderFont:{
        fontWeight:'500',
    },
});

```

* PDClient/project/App/modules/receivePoint/placeOrder/PreOrderList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TextInput,
    Keyboard,
    TouchableOpacity,
} = ReactNative;

import Swipeout from 'react-native-swipeout';

const { PageList, Button, MessageBox, DImage } = COMPONENTS;

module.exports = React.createClass({
    getInitialState () {
        return {
            text:'',
            senderPhone:'',
            status:false,
        };
    },
    doStartSearch () {
        const { text } = this.state;
        this.setState({
            senderPhone: text,
            status : true,
        }, () => { this.listView.refresh(); Keyboard.dismiss(); });
    },
    doDetail (obj, sectionID) {
        this.props.onClickRow(obj);
    },
    renderRow (obj, rowID) {
        return (
            <View style={styles.item}>
                <TouchableOpacity style={styles.content} onPress={this.doDetail.bind(null, obj, rowID)}>
                    <DImage
                        resizeMode='stretch'
                        defaultSource={app.img.common_default}
                        source={{ uri:obj.photo }}
                        style={styles.image}
                    />
                    <View style={styles.row}>
                        <View style={styles.itemTop}>
                            <Text style={styles.itemText}>始发地：{(obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : obj.startPoint)}</Text>
                        </View>
                        <Text style={styles.itemText}>
                        收货人：{obj.receiverName ? obj.receiverName : obj.receiverName == '' ? obj.receiverPhone : obj.receiverName}
                        </Text>
                        <Text style={styles.itemText}
                            numberOfLines={1}>到货地：{obj.endPoint}</Text>
                    </View>
                </TouchableOpacity>
            </View>
        );
    },
    render () {
        const { text, senderPhone, status } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.searchContainer}>
                    <Image
                        resizeMode='cover'
                        source={app.img.common_search_button}
                        style={styles.iconSearch} />
                    <TextInput
                        placeholder='输入发货人手机号'
                        defaultValue={text}
                        underlineColorAndroid='transparent'
                        onSubmitEditing={Keyboard.dismiss}
                        onChangeText={(text) => { this.setState({ text }); }}
                        style={styles.searchTextInput}
                        />
                    <Button onPress={this.doStartSearch} style={styles.searchBtnStyle} textStyle={styles.searchBtnTextStyle}>搜索</Button>
                </View>
                <View style={styles.list}>
                    {
                        status ? <PageList
                            ref={(ref) => { this.listView = ref; }}
                            renderRow={this.renderRow}
                            listParam={{ userId: app.personal.info.userId, senderPhone }}
                            listName={'orderList'}
                            autoLoad={false}
                            pageNo={-1}
                            listUrl={app.route.ROUTE_GET_PRE_ORDER_LIST_AGENT}
                            refreshEnable
                            /> : null
                    }

                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
        paddingTop:10,
    },
    searchContainer: {
        height: 30,
        marginHorizontal:50,
        marginBottom:20,
        paddingVertical: 2,
        borderRadius: 14,
        borderWidth:1,
        borderColor:'#dbdbdb',
        backgroundColor: '#FFFFFF',
        flexDirection: 'row',
        overflow: 'hidden',
        alignItems:'center',
    },
    searchTextInput: {
        padding:0,
        height: 25,
        width: sr.w - 200,
        fontSize: 14,
        marginLeft: 5,
    },
    iconSearch: {
        height: 15,
        width: 15,
        marginLeft:5,
    },
    searchBtnStyle: {
        width:75,
        height:30,
        borderRadius:0,
        borderColor:'#dbdbdb',
        backgroundColor:'#F4F4F4',
        borderLeftWidth:1,
    },
    searchBtnTextStyle: {
        color:'black',
        fontSize:15,
    },
    item:{
        height:70,
        marginLeft:10,
        flexDirection: 'row',
        alignItems:'center',
    },
    content:{
        flexDirection: 'row',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
    },
    image:{
        height:50,
        width:50,
        marginRight:5,
    },
    itemTop:{
        flex:1,
        flexDirection: 'row',
    },
    itemText:{
        fontSize:12,
        color:'#888888',
        flex:1,
    },
    list:{
        width:sr.w,
        flex:1,
    },
    row:{
        flex:1,
        height:60,
        marginTop:10,
    },
});

```

* PDClient/project/App/modules/receivePoint/placeOrder/index.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    ScrollView,
    RefreshControl,
    Text,
    TouchableHighlight,
} = ReactNative;

const { MessageBox } = COMPONENTS;
const PreOrderList = require('./PreOrderList');
const PlaceOrder = require('./PlaceOrder');
const OrderDetail = require('./OrderDetail');
const TimerMixin = require('react-timer-mixin');

const SHOW_PLACE_VIEW = 0;
const SHOW_ORDER_DETAIL = 1;
module.exports = React.createClass({
    mixins: [TimerMixin],
    statics: {
        title: '填单',
    },
    getInitialState () {
        return {
            tabIndex: 0,
            showType:SHOW_PLACE_VIEW, // 0显示PlaceOrder和PreOrderList,1显示OrderDetail
            data:{},
            isModify:false, // 是修改货单还是下单
            isUpdateView:true, // 是否在onWillFocus中调用getLastestOrder
        };
    },
    setPlaceTitle () {
        app.getCurrentRoute().title = '填单';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    setModifyTitle () {
        app.getCurrentRoute().title = '货单信息';
        app.getCurrentRoute().rightButton = { title: '修改', handler: () => { this.modify(); } };
        app.forceUpdateNavbar();
    },
    setContinueTitle () {
        app.getCurrentRoute().title = '货单信息';
        app.getCurrentRoute().rightButton = { title: '继续下单', handler: () => { this.continuePlaceOrder(); } };
        app.forceUpdateNavbar();
    },
    getLastestOrder () {
        const param = {
            userId:app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_LASTEST_ORDER, param, this.getLastestOrderSuccess, false);
    },
    getLastestOrderSuccess (data) {
        this.setState({ isUpdateView:true });
        if (data.success) {
            if (data.context.state == OS.OS_READY_PAYMENT || data.context.state == OS.OS_READY_PRINT_BAR_CODE) {
                this.setState({ data:data.context, showType:SHOW_ORDER_DETAIL }, this.setModifyTitle);
            } else {
                this.setState({ data:data.context, showType:SHOW_ORDER_DETAIL }, this.setContinueTitle);
            }
        } else {
            this.setState({ showType:SHOW_PLACE_VIEW, tabIndex:0, data:{} }, this.setPlaceTitle);
        }
    },
    confirm (data) {
        const { isModify } = this.state;
        this.setState({ isUpdateView:true });
        isModify ? this.modifyOrder(data) : this.placeOrder(data);
    },
    placeOrder (data) {
        const { sendDoorEndPointLastCode, isReachPay, sendDoorEndPoint, name, senderName, senderPhone, isInsuance, endPoint, receiverName, receiverPhone, weight, size, totalNumbers, photo, isCityDistribute, endPointLastCode, isSendDoor, proxyCharge, totalDesignatedFee } = data;
        const param = {
            userId: app.personal.info.userId,
            photo,
            senderName,
            receiverPhone,
            receiverName,
            isCityDistribute,
            endPoint,
            endPointLastCode,
            sendDoorEndPoint,
            sendDoorEndPointLastCode,
            weight,
            size,
            totalNumbers,
            isSendDoor,
            proxyCharge,
            isInsuance,
            totalDesignatedFee,
            isReachPay,
            senderPhone,
            name,
        };
        POST(app.route.ROUTE_PLACE_ORDER_AGENT, param, this.placeOrderSuccess, true);
    },
    placeOrderSuccess (data) {
        if (data.success) {
            Toast('填单成功');
            this.setState({ showType:SHOW_ORDER_DETAIL, data:data.context }, this.setModifyTitle);
        } else {
            Toast(data.msg);
            this.setState({ showType:SHOW_PLACE_VIEW });
        }
    },
    modifyOrder (data) {
        const { sendDoorEndPointLastCode, isReachPay, sendDoorEndPoint, name, senderName, senderPhone, isInsuance, isCityDistribute, endPoint, receiverName, receiverPhone, weight, size, totalNumbers, photo, endPointLastCode, isSendDoor, proxyCharge, totalDesignatedFee } = data;
        const param = {
            userId: app.personal.info.userId,
            orderId:this.state.data.id,
            isCityDistribute,
            photo,
            senderName,
            receiverPhone,
            receiverName,
            endPoint,
            endPointLastCode,
            sendDoorEndPoint,
            sendDoorEndPointLastCode,
            weight,
            size,
            totalNumbers,
            isSendDoor,
            proxyCharge,
            isInsuance,
            totalDesignatedFee,
            isReachPay,
            senderPhone,
            name,
        };
        _.forIn(param, (v, k) => {
            if (_.isEqual(this.state.data[k], v) && k != 'userId' && k != 'orderId') {
                delete param[k];
            }
        });
        POST(app.route.ROUTE_MODIFY_ORDER_AGENT, param, this.placeOrderSuccess, true);
    },
    modify () {
        this.setState({ showType:SHOW_PLACE_VIEW, tabIndex:0, isModify:true, isUpdateView:false }, this.setPlaceTitle);
    },
    continuePlaceOrder () {
        this.setState({ tabIndex:0, showType:SHOW_PLACE_VIEW, isModify:false, isUpdateView:false, data:{} }, this.setPlaceTitle);
    },
    doPrintReceipt () {
        const { data } = this.state;
        const param = {
            userId: app.personal.info.userId,
            orderId: data.id,
        };
        POST(app.route.ROUTE_PRINT_ORDER_BILL, param, this.doPrintReceiptSuccess, true);
    },
    showConfirmPayBox () {
        app.showModal(
            <MessageBox
                onConfirm={() => { this.confirmCachPay(); }}
                content={'确认已现金收款'}
                width={sr.s(250)} />
        );
    },
    confirmCachPay () {
        const { data } = this.state;
        const param = {
            userId: app.personal.info.userId,
            orderId: data.id,
        };
        POST(app.route.ROUTE_CONFIRM_CACH_PAYED, param, this.confirmCachPaySuccess, false);
    },
    confirmCachPaySuccess (data) {
        if (data.success) {
            Toast('支付成功');
            this.setState({ data:data.context }, this.setContinueTitle);
        } else {
            Toast(data.msg);
        }
    },
    doPrintReceiptSuccess (data) {
        if (data.success) {
            Toast('打印完成');
            this.setState({ showType:SHOW_PLACE_VIEW, data:{}, isModify:false }, this.setPlaceTitle);
        } else {
            Toast(data.msg);
        }
    },
    doPrintQrcode () {
        const { data } = this.state;
        const param = {
            userId: app.personal.info.userId,
            orderId: data.id,
        };
        POST(app.route.ROUTE_PRINT_BAR_CODE, param, this.doPrintQrcodeSuccess, false);
    },
    doPrintQrcodeSuccess (data) {
        if (data.success) {
            Toast('打印成功');
            this.setState({
                data : data.context,
            }, data.context.state == OS.OS_READY_PAYMENT ? this.setModifyTitle : this.setContinueTitle);
        } else {
            Toast(data.msg);
        }
    },
    componentWillMount () {
        this.state.isUpdateView ? this.getLastestOrder() : this.setPlaceTitle();
    },
    onWillFocus () {
        this.state.isUpdateView ? this.getLastestOrder() : this.setPlaceTitle();
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    onClickRow (data) {
        this.setState({ tabIndex:0, showType:SHOW_PLACE_VIEW, data:data }, this.setPlaceTitle);
    },
    render () {
        const menus = ['现填单', '通过预下单填单'];
        const { tabIndex, showType, data } = this.state;
        return (
            <ScrollView style={styles.container}
                refreshControl={
                    <RefreshControl
                        refreshing={false}
                        onRefresh={this.getLastestOrder}
                        title='正在刷新...' />
                }>
                {
                    showType == SHOW_PLACE_VIEW ?
                        <View style={{ flex:1 }}>
                            <View style={styles.tabContainer}>
                                {
                                menus.map((item, i) => (
                                    <TouchableHighlight
                                        key={i}
                                        underlayColor='rgba(0, 0, 0, 0)'
                                        onPress={this.changeTab.bind(null, i)}
                                        style={styles.touchTab}>
                                        <View style={styles.tabButton}>
                                            <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                                {item}
                                            </Text>
                                            <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                        </View>
                                    </TouchableHighlight>
                                ))
                            }
                            </View>
                            {
                            tabIndex == 0 ? <PlaceOrder confirm={this.confirm} info={data} /> : <PreOrderList onClickRow={this.onClickRow} />
                        }
                        </View>
                    :
                        <OrderDetail data={data} showConfirmPayBox={this.showConfirmPayBox} doPrintReceipt={this.doPrintReceipt} doPrintQrcode={this.doPrintQrcode} />
                }
            </ScrollView>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
        backgroundColor: '#FFFFFF',
        marginBottom:10,
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonCenter: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonRight: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        borderRadius: 9,
    },
    tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/LogisticsRanking.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableOpacity,
} = ReactNative;

const BaidumapSelectPosition = require('../../baidumap');

module.exports = React.createClass({
    statics: {
        title: '填单',
        rightButton: { title: '保存', handler: () => { app.scene.prePlaceOrder&&app.scene.prePlaceOrder(); } },
    },
    prePlaceOrder () {

    },
    getInitialState () {
        return {
            set:0,
            position:'位置',
        };
    },
    toPrice () {
        this.setState({ set:1 });
    },
    toTime () {
        this.setState({ set:2 });
    },
    todistance () {
        this.setState({ set:3 });
    },
    toMap () {
        app.push({
            component: BaidumapSelectPosition,
            passProps:{
                confirm:(marker) => {
                    this.setState({ position:marker.title });
                },
            },
        });
    },
    render () {
        return (
            <View style={styles.container}>
                <View style={styles.containerTop}>
                    <Text style={styles.topText}>物流排名</Text>
                    <TouchableOpacity onPress={this.toMap}>
                        <Text style={styles.topTextRight}>{this.state.position}</Text>
                    </TouchableOpacity>
                </View>
                <View style={styles.top}>
                    <TouchableOpacity onPress={this.toPrice}>
                        <Text style={this.state.set == 1 ? styles.redText : styles.itemText}>价格最低</Text>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={this.toTime}>
                        <Text style={this.state.set == 2 ? styles.redText : styles.itemText}>时间最短</Text>
                    </TouchableOpacity>
                    <TouchableOpacity onPress={this.todistance}>
                        <Text style={this.state.set == 3 ? styles.redText : styles.itemText}>距离最短</Text>
                    </TouchableOpacity>
                </View>
                <View style={styles.info}>
                    <View style={styles.shop}>
                        <Text style={styles.shopInfo}>六面通物流公司 - 四面通物流大超市石板店</Text>
                        <Text style={styles.distance}>据我2km</Text>
                    </View>
                    <View style={styles.infoDetails}>
                        <View style={styles.unit}>
                            <Text style={styles.unitPriceNum}>¥20.00/吨</Text>
                            <Text style={styles.unitPrice}>运费单价</Text>
                        </View>
                        <View style={styles.unit}>
                            <Text style={styles.unitPriceNum}>¥150.00/吨</Text>
                            <Text style={styles.unitPrice}>送货上门</Text>
                        </View>
                        <View style={styles.unit}>
                            <Text style={styles.time}>2天</Text>
                            <Text style={styles.EstimateTime}>预计用时</Text>
                        </View>
                    </View>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    containerTop:{
        flexDirection: 'row',
        justifyContent:'space-between',
    },
    topText:{
        fontSize:16,
        marginTop:16,
        marginLeft:12,
        marginBottom:10,
        color:'#333333',
    },
    topTextRight:{
        marginRight:10,
        fontSize:16,
        marginTop:16,
        marginBottom:10,
        color:'#333333',
    },
    top:{
        height:35,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        justifyContent:'space-around',
        alignItems:'center',
    },
    itemText:{
        fontSize:13,
        color:'#333333',
    },
    redText:{
        fontSize:13,
        color:'#f11821',
    },
    info:{
        marginTop:1,
        height:95,
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
    },
    shop:{
        flexDirection: 'row',
    },
    shopInfo:{
        fontSize:14,
        color:'#333333',
        marginLeft:10,
    },
    distance:{
        fontSize:12,
        color:'#ff9517',
        marginTop:2,
        marginLeft:8,
    },
    infoDetails:{
        marginTop:10,
        backgroundColor:'#FFFFFF',
        flexDirection: 'row',
        justifyContent:'space-around',
    },
    unit:{
        justifyContent:'center',
        alignItems:'center',
    },
    unitPriceNum:{
        fontSize:15,
        color:'#f11821',
        fontWeight:'500',
        marginBottom:3,
    },
    unitPrice:{
        fontSize:12,
        color:'#333333',
        marginTop:3,
    },
    time:{
        paddingTop:2,
        fontSize:15,
        color:'#333333',
        marginBottom:3,
    },
    EstimateTime:{
        fontSize:12,
        color:'#333333',
        marginTop:3,
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/LookPreOrder.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableOpacity,
} = ReactNative;

const BaidumapSelectPosition = require('../../baidumap');
const MarketSearchResult = require('./MarketSearchResult');
module.exports = React.createClass({
    statics: {
        title: '填单详情',
    },
    getInitialState () {
        return {
            mapInfo:{ location:[0, 0], region:'' },
        };
    },
    doSelectPosition () {
        app.push({
            component: BaidumapSelectPosition,
            passProps:{
                confirm:(marker) => {
                    this.setState({ mapInfo:{ region:marker.province + marker.city, location:[marker.longitude, marker.latitude] } });
                },
            },
        });
    },
    render () {
        const { mapInfo } = this.state;
        const { param } = this.props;
        const { name, startPoint, receiverName, receiverPhone, endPoint, weight, size, proxyCharge, totalDesignatedFee, isInsuance, isSendDoor, id, totalNumbers, shop, agent } = param;
        return (
            <View style={styles.container}>
                <View style={styles.startPoint}>
                    <Text style={styles.itemBottom}>始发地：{shop && shop.name || agent && agent.name}</Text>
                </View>
                <View style={styles.item}>
                    <View style={styles.itemTop}>
                        <Text style={styles.itemName}>收货人：{receiverName}</Text>
                        <Text style={styles.itemphone}>{receiverPhone}</Text>
                    </View>
                    <Text style={styles.itemBottom}>收货地：{endPoint}</Text>
                </View>
                <View style={styles.itemNumber}>
                    <Text style={styles.weight}>货物名称:{name}</Text>
                    <Text style={styles.weight}>重量：{weight}吨</Text>
                    <Text style={styles.size}>方量：{size}m³</Text>
                    <Text style={styles.number}>件数：{totalNumbers}件</Text>
                </View>
                <View style={styles.itemFee}>
                    <Text style={styles.generationFee}>代收货款：¥{ proxyCharge == '' ? 0 : proxyCharge }元</Text>
                    <Text style={styles.designatedFee}>指定收款：¥{ totalDesignatedFee == '' ? 0 : totalDesignatedFee }元</Text>
                    <Text style={styles.isInsuance}>是否保价：{isInsuance ? '是' : '否' }</Text>
                    <Text style={styles.isSendDoor}>送货上门：{isSendDoor ? '是' : '否' }</Text>
                </View>
                <View style={styles.logisticsRankContainer}>
                    <Text style={styles.logisticsRankTitle}>物流排名</Text>
                    <TouchableOpacity onPress={this.doSelectPosition}>
                        <Text style={styles.logisticsRankPosition}>{mapInfo.region ? mapInfo.region : '位置'}</Text>
                    </TouchableOpacity>
                </View>
                <MarketSearchResult mapInfo={mapInfo} orderInfo={{ orderId:id }} />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    logisticsRankContainer:{
        flexDirection: 'row',
        justifyContent:'space-between',
    },
    logisticsRankTitle:{
        fontSize:15,
        marginTop:10,
        marginLeft:12,
        marginBottom:10,
        color:'#333333',
    },
    logisticsRankPosition:{
        marginRight:10,
        fontSize:15,
        marginTop:10,
        marginBottom:10,
        color:'#333333',
    },
    startPoint:{
        height:25,
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
        borderBottomWidth:1,
        borderColor:'#F4F4F4',
    },
    item:{
        height:50,
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
        borderBottomWidth:1,
        borderColor:'#F4F4F4',
    },
    itemTop:{
        flexDirection: 'row',
        justifyContent: 'space-between',
        marginBottom:2,
    },
    itemName:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
    },
    itemphone:{
        marginRight:10,
        fontSize:12,
        color:'#333333',
    },
    itemBottom:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
        marginTop:2,
    },
    itemNumber:{
        height:85,
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
    },
    weight:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
        marginBottom:2,
    },
    size:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
        marginTop:2,
    },
    number:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
        marginTop:2,
    },
    itemFee:{
        height:90,
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
        marginTop:1,
    },
    generationFee:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
        marginBottom:2,
    },
    designatedFee:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
        marginBottom:2,
        marginTop:2,
    },
    isInsuance:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
        marginTop:5,
    },
    isSendDoor:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
        marginTop:4,
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/LookPreOrderGroup.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    TouchableOpacity,
} = ReactNative;

const { SelectStartPointAddress } = COMPONENTS;
import { Geolocation } from '@remobile/react-native-baidu-map';

const BaidumapSelectPosition = require('../../baidumap');
const MarketSearchResultGroup = require('./MarketSearchResultGroup');

module.exports = React.createClass({
    mixins: [SceneMixin],
    statics: {
        title: '下单组详情',
        rightButton: () => app.scene.renderRightButton&&app.scene.renderRightButton(),
    },
    getInitialState () {
        const { param } = this.props;
        return {
            mapInfo:{ location:[0, 0], region:'' },
            startInfo:{ startPoint:param.startPoint, startPointLastCode:param.startPointLastCode,id:param.shop ? param.shop.id : param.agent ? param.agent.id : '' } || {},
            type:false,
            groupName:param.name,
        };
    },
    componentDidMount () {
        Geolocation.getCurrentPosition()
            .then(data => {
                console.warn(JSON.stringify(data));
                this.setState({ mapInfo:{ region:data.province + data.city, location:[data.longitude, data.latitude] } });
            })
            .catch(e => {
                console.warn(e, 'error');
            });
    },
    renderRightButton () {
        const { type } = this.state;
        if (type) {
            return { title: '确认', handler: () => { this.doConfirm(); } };
        } else {
            return { title: '修改', handler: () => { this.doEdit(); } };
        }
    },
    doEdit () {
        const { type } = this.state;
        this.setState({ type:!type }, () => {
            app.getCurrentRoute().rightButton = { title: '确认', handler: () => { this.doConfirm(); } };
            app.forceUpdateNavbar();
        });
    },
    doConfirm () {
        const { type, startInfo, groupName } = this.state;
        const { param } = this.props;
        this.setState({ type:!type }, () => {
            app.getCurrentRoute().rightButton = { title: '修改', handler: () => { this.doEdit(); } };
            app.forceUpdateNavbar();
        });
        if (!startInfo.startPoint) {
            Toast('请选择发货地');
            return;
        }
        const params = {
            userId:app.personal.info.userId,
            orderGroupId:param.id,
            groupName,
            ...startInfo,
        };
        POST(app.route.ROUTE_MODIFY_PRE_ORDER_GROUP, params, this.doConfirmSuccess, false);
    },
    doConfirmSuccess (data) {
        if (data.success) {
            this.props.refresh();
        } else {
            Toast(data.msg);
        }
    },
    doSelectPosition () {
        app.push({
            component: BaidumapSelectPosition,
            passProps:{
                confirm:(marker) => {
                    this.setState({ mapInfo:{ region:marker.province + marker.city, location:[marker.longitude, marker.latitude] } });
                },
            },
        });
    },
    showStartPlace () {
        const { startInfo } = this.state;
        app.push({
            component: SelectStartPointAddress,
            passProps:{
                isNeedSelectAll:true,
                endPointLastCode:startInfo.startPointLastCode,
                id:startInfo.id || startInfo.shopId || startInfo.agentId,
                confirmAddress:(obj) => {
                    this.setState({ startInfo:obj });
                },
            },
        });
    },
    render () {
        const { mapInfo, type, groupName, startInfo } = this.state;
        const { param } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.item}>
                    {
                        type ?
                            <View>
                                <View style={styles.itemRow}>
                                    <Text style={styles.itemText}>群组名：</Text>
                                    <TextInput style={styles.modifyItemText}
                                        underlineColorAndroid='transparent'
                                        defaultValue={groupName}
                                        onChangeText={(text) => this.setState({ groupName: text })} />
                                </View>
                                <TouchableOpacity
                                    activeOpacity={0.6}
                                    onPress={this.showStartPlace}>
                                    <View style={styles.itemRow}>
                                        <Text style={styles.itemText}>发货地</Text>
                                        <View style={styles.itemPoint}>
                                            <Text style={styles.modifyItemText}
                                                numberOfLines={1}
                                                ellipsizeMode='tail'>{startInfo.startPoint || ''}</Text>
                                        </View>
                                    </View>
                                </TouchableOpacity>
                            </View> :
                            <View>
                                <Text style={styles.itemText}>群组名：{groupName}</Text>
                                <Text style={styles.itemText}>发货地：{startInfo.startPoint || ''}</Text>
                            </View>
                    }
                    <Text style={styles.itemText}>总尺寸：{param.totalSize}m³</Text>
                    <Text style={styles.itemText}>总重量：{param.totalWeight}吨</Text>
                    <Text style={styles.itemText}>总件数：{param.totalNumbers}件</Text>
                    <Text style={styles.itemText}>总单数：{param.orderCount}单</Text>
                </View>
                <View style={styles.logisticsRankContainer}>
                    <Text style={styles.logisticsRankTitle}>物流排名</Text>
                    <TouchableOpacity onPress={this.doSelectPosition}>
                        <Text style={styles.logisticsRankPosition}>{mapInfo.region ? mapInfo.region : '位置'}</Text>
                    </TouchableOpacity>
                </View>
                <MarketSearchResultGroup mapInfo={mapInfo} orderInfo={{ orderGroupId:param.id }} />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    logisticsRankContainer:{
        flexDirection: 'row',
        justifyContent:'space-between',
    },
    logisticsRankTitle:{
        fontSize:15,
        marginTop:10,
        marginLeft:10,
        marginBottom:10,
        color:'#333333',
    },
    logisticsRankPosition:{
        marginRight:10,
        fontSize:15,
        marginTop:10,
        marginBottom:10,
        color:'#333333',
    },
    item:{
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
        borderBottomWidth:2,
        borderColor:'#F4F4F4',
        paddingBottom:10,
    },
    itemText:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
        paddingTop:5,
        paddingBottom:5,
    },
    modifyItemText:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
        paddingTop:5,
        paddingBottom:5,
        flex:1,
    },
    itemRow:{
        flexDirection:'row',
        alignItems:'center',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/MarketSearchResult.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

import { Geolocation } from '@remobile/react-native-baidu-map';
const SupermarketSearchResult = require('./SupermarketSearchResult');
const ReceivePointSearchResult = require('./ReceivePointSearchResult');
module.exports = React.createClass({
    getInitialState () {
        return {
            tabIndex: 0,
            context:{},
            tabMenus:[],
            mapInfo:{},
        };
    },
    componentWillReceiveProps (nextProps) {
        this.setState({ mapInfo: nextProps.mapInfo });
    },
    componentWillMount () {
        Geolocation.getCurrentPosition()
        .then(data => {
            console.warn(JSON.stringify(data));
            this.setState({ mapInfo:{ region:data.province + data.city, location:[data.longitude, data.latitude] } }, () => { this.getRoadMaps(); });
        })
        .catch(e => {
            console.warn(e, 'error');
        });
    },
    getRoadMaps () {
        const { mapInfo } = this.state;
        const { orderInfo } = this.props;
        const param = {
            userId: app.personal.info.userId,
            orderBy: 0,
            pageNo:0,
            pageSize:10,
            ...mapInfo,
            ...orderInfo };
        POST(app.route.ROUTE_GET_ROADMAP_LIST_WITH_PRE_ORDER, param, this.getRoadMapsSuccess, false);
    },
    getRoadMapsSuccess (data) {
        if (data.success) {
            this.setState({ tabMenus:(!data.context.agent || data.context.agent.roadmapList.length == 0) ? ['物流超市'] : ['物流超市', '揽收点'] });
            this.setState({ context:data.context });
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { tabIndex, tabMenus, context, mapInfo } = this.state;
        const { orderInfo } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.topTab}>
                    {
                        tabMenus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={() => {
                                    this.setState({ tabIndex:i });
                                }}
                                style={tabIndex != i ? styles.noTouchTab : styles.touchTab}>
                                <Text style={tabIndex != i ? styles.NoTouchText : styles.touchText}>
                                    {item}
                                </Text>
                            </TouchableHighlight>
                        ))
                    }
                </View>

                {
                    tabIndex === 0 ? <SupermarketSearchResult mapInfo={mapInfo} listName={'roadmapList'} orderInfo={orderInfo} data={context && context['shop']} /> : <ReceivePointSearchResult mapInfo={mapInfo} orderInfo={orderInfo} listName={'roadmapList'} data={context && context['agent']} />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        height: 30,
        width: 140,
        borderRadius:5,
        borderColor:'white',
        borderWidth:1,
        flexDirection: 'row',
        overflow: 'hidden',
    },
    topTab: {
        flexDirection:'row',
        height:30,
    },
    touchTab: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        backgroundColor:'#c81622',
    },
    noTouchTab: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        backgroundColor:'#E5E5E5',
    },
    NoTouchText: {
        color:'#c81622',
        fontWeight:'500',
    },
    touchText: {
        color:'#E5E5E5',
        fontWeight:'500',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/MarketSearchResultGroup.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

import { Geolocation } from '@remobile/react-native-baidu-map';
const SupermarketSearchResultGroup = require('./SupermarketSearchResultGroup');
const ReceivePointSearchResultGroup = require('./ReceivePointSearchResultGroup');
module.exports = React.createClass({
    getInitialState () {
        return {
            tabIndex: 0,
            context:{},
            tabMenus:[],
            mapInfo:{},
        };
    },
    componentWillReceiveProps (nextProps) {
        this.setState({ mapInfo: nextProps.mapInfo });
    },
    componentWillMount () {
        Geolocation.getCurrentPosition()
        .then(data => {
            console.warn(JSON.stringify(data));
            this.setState({ mapInfo:{ region:data.province + data.city, location:[data.longitude, data.latitude] } }, () => { this.getRoadMaps(); });
        })
        .catch(e => {
            console.warn(e, 'error');
        });
    },
    getRoadMaps () {
        const { mapInfo } = this.state;
        const { orderInfo } = this.props;
        const param = {
            userId: app.personal.info.userId,
            orderBy: 0,
            pageNo:0,
            pageSize:10,
            ...mapInfo,
            ...orderInfo };
        POST(app.route.ROUTE_GET_ROADMAP_LIST_WITH_PRE_ORDER_GROUP, param, this.getRoadMapsSuccess, false);
    },
    getRoadMapsSuccess (data) {
        if (data.success) {
            this.setState({ tabMenus:(!data.context.agentList || data.context.agentList.length == 0) ? ['物流超市'] : ['物流超市', '揽收点'] });
            this.setState({ context:data.context });
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { tabIndex, tabMenus, context, mapInfo } = this.state;
        const { orderInfo } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.topTab}>
                    {
                        tabMenus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={() => {
                                    this.setState({ tabIndex:i });
                                }}
                                style={tabIndex != i ? styles.noTouchTab : styles.touchTab}>
                                <Text style={tabIndex != i ? styles.NoTouchText : styles.touchText}>
                                    {item}
                                </Text>
                            </TouchableHighlight>
                        ))
                    }
                </View>

                {
                    tabIndex === 0 ? <SupermarketSearchResultGroup mapInfo={mapInfo} listName={'shopList'} orderInfo={orderInfo} data={context && context['shopList']} /> : <ReceivePointSearchResultGroup mapInfo={mapInfo} orderInfo={orderInfo} listName={'agentList'} data={context && context['agentList']} />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        height: 30,
        width: 140,
        borderRadius:5,
        borderColor:'white',
        borderWidth:1,
        flexDirection: 'row',
        overflow: 'hidden',
    },
    topTab: {
        flexDirection:'row',
        height:30,
    },
    touchTab: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        backgroundColor:'#c81622',
    },
    noTouchTab: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        backgroundColor:'#E5E5E5',
    },
    NoTouchText: {
        color:'#c81622',
        fontWeight:'500',
    },
    touchText: {
        color:'#E5E5E5',
        fontWeight:'500',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/PreOrderDetail.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TextInput,
    Image,
    ScrollView,
    TouchableOpacity,
} = ReactNative;
const { DImage, SelectRegionAddress, SelectStartPointAddress, SelectShortAddress, ActionSheet } = COMPONENTS;
const Camera = require('@remobile/react-native-camera');
const LookPreOrder = require('./LookPreOrder');

module.exports = React.createClass({
    statics: {
        title: '下单详情',
        leftButton:{ handler: () => { app.scene.goBack&&app.scene.goBack(); } },
    },
    goBack () {
        const { refresh } = this.props;
        !!refresh && refresh();
        app.navigator.popToTop();
    },
    getInitialState () {
        return {
            pageState:0, // 0,修改,1,保存
            ...this.props.info
        };
    },
    componentDidMount () {
        this.originData = this.state;
    },
    showStartPlace () {
        const { startPointLastCode, shopId, agentId } = this.state;
        app.push({
            component: SelectStartPointAddress,
            passProps:{
                isNeedSelectAll:true,
                endPointLastCode:startPointLastCode,
                id:shopId || agentId,
                confirmAddress:(obj) => {
                    this.setState({ ...obj });
                },
            },
        });
    },
    showEndPlace () {
        const { endPointLastCode } = this.state;
        app.push({
            component: SelectRegionAddress,
            title:'到货地',
            passProps:{
                endPointLastCode,
                confirmAddress:(endPoint, endPointLastCode) => {
                    if (endPointLastCode != this.state.endPointLastCode) {
                        this.setState({ sendDoorEndPointLastCode:0, sendDoorEndPoint:'', isSendDoor : false });
                    }
                    this.setState({ endPoint, endPointLastCode });
                },
            },
        });
    },
    showCityDistributeEndPlace () {
        const { endPointLastCode, startPointLastCode } = this.state;
        if (!startPointLastCode) {
            Toast('同城配送需先选择始发地');
            return;
        }
        app.push({
            component: SelectShortAddress,
            title:'同城配送地址',
            passProps:{
                endPointLastCode:endPointLastCode,
                parentCode : startPointLastCode,
                confirmAddress:(endPoint, endPointLastCode) => {
                    this.setState({ endPoint, endPointLastCode });
                },
            },
        });
    },
    showSendDoorEndPlace () {
        const { endPointLastCode, sendDoorEndPointLastCode } = this.state;
        if (!endPointLastCode) {
            Toast('请先选择到货地');
            return;
        }
        app.push({
            component: SelectShortAddress,
            title:'送货上门地址',
            passProps:{
                endPointLastCode:sendDoorEndPointLastCode,
                parentCode : endPointLastCode,
                confirmAddress:(sendDoorEndPoint, sendDoorEndPointLastCode) => {
                    this.setState({ sendDoorEndPoint, sendDoorEndPointLastCode });
                },
            },
        });
    },
    setIsInsuance () {
        this.setState({
            isInsuance: !this.state.isInsuance,
        });
    },
    setIsReachPay () {
        this.setState({
            isReachPay : !this.state.isReachPay,
        });
    },
    setIsSendDoor () {
        this.setState({
            isSendDoor: !this.state.isSendDoor,
        });
    },
    setIsCityDistribute () {
        this.setState({
            isCityDistribute: !this.state.isCityDistribute,
            endPoint:'',
            endPointLastCode:0
        });
    },
    render () {
        const {
            pageState,
            photo,
            startPoint,
            endPoint,
            receiverName,
            senderName,
            name,
            receiverPhone,
            totalNumbers,
            weight,
            size,
            isInsuance,
            isReachPay,
            isSendDoor,
            isCityDistribute,
            actionSheetVisible,
            proxyCharge,
            totalDesignatedFee,
            sendDoorEndPoint,
        } = this.state;
        return (
            <View style={styles.container}>
                {
                    pageState == 1 ?
                        <ScrollView>
                            { photo == '' ?
                                <TouchableOpacity
                                    onPress={this.doShowActionSheet}
                                    style={styles.topImage}>
                                    <DImage
                                        defaultSource={app.img.order_upload_picture}
                                        source={{ uri: photo }}
                                        style={styles.photo} />
                                    <Text style={styles.topImageText}>上传货物图片</Text>
                                </TouchableOpacity>
                            :
                                <TouchableOpacity
                                    onPress={this.doShowActionSheet}
                                    style={styles.topBigImage}>
                                    <Text style={{ color:'#666666' }}>货物图片</Text>
                                    <DImage
                                        source={{ uri: photo }}
                                        style={styles.bigPicture} />
                                </TouchableOpacity>
                            }
                            <View style={styles.itemStart}>
                                <Text style={styles.itemLeft}>发货人</Text>
                                <TextInput style={styles.inputText}
                                    underlineColorAndroid='transparent'
                                    placeholder='请输入发货人姓名'
                                    defaultValue={senderName}
                                    onChangeText={(text) => this.setState({ senderName: text })}
                                />
                            </View>
                            <View style={styles.item}>
                                <Text style={styles.itemLeft}>货物名称</Text>
                                <TextInput style={styles.inputText}
                                    underlineColorAndroid='transparent'
                                    placeholder='请输入货物名称'
                                    defaultValue={name}
                                    onChangeText={(text) => this.setState({ name: text })}
                                />
                            </View>
                            <View style={styles.item}>
                                <Text style={styles.itemLeft}>收货人</Text>
                                <TextInput style={styles.inputText}
                                    underlineColorAndroid='transparent'
                                    placeholder='请输入收货人姓名'
                                    defaultValue={receiverName}
                                    onChangeText={(text) => this.setState({ receiverName: text })}
                                />
                            </View>
                            <View style={styles.item}>
                                <Text style={styles.itemLeft}>收货人电话</Text>
                                <TextInput style={styles.inputText}
                                    underlineColorAndroid='transparent'
                                    keyboardType='numeric'
                                    placeholder='请输入收货人手机号码'
                                    defaultValue={receiverPhone}
                                    maxLength={11}
                                    onChangeText={(text) => this.setState({ receiverPhone: text })}
                                />
                            </View>
                            <TouchableOpacity
                                activeOpacity={0.6}
                                onPress={this.showStartPlace}>
                                <View style={styles.itemStart}>
                                    <Text style={styles.itemLeft}>始发地</Text>
                                    <View style={styles.itemTop}>
                                        <Text style={styles.itemRight}
                                            numberOfLines={1}
                                            ellipsizeMode='tail'>{startPoint || ''}</Text>
                                        <Image
                                            source={app.img.common_go}
                                            style={styles.goStyle} />
                                    </View>
                                </View>
                            </TouchableOpacity>
                            <TouchableOpacity onPress={this.setIsCityDistribute}>
                                <View style={styles.item}>
                                    <Text style={styles.itemLeft}>是否同城配送</Text>
                                    <View style={styles.itemTop}>
                                        <Text style={styles.itemRight}
                                            numberOfLines={1}
                                            ellipsizeMode='tail'>{isCityDistribute ? '是' : '否'}</Text>
                                        <Image
                                            source={app.img.common_go}
                                            style={styles.goStyle} />
                                    </View>
                                </View>
                            </TouchableOpacity>
                            {
                                isCityDistribute ?
                                    <TouchableOpacity
                                        activeOpacity={0.6}
                                        onPress={this.showCityDistributeEndPlace}>
                                        <View style={styles.item}>
                                            <Text style={styles.itemLeft}>同城配送地址</Text>
                                            <View style={styles.itemTop}>
                                                <Text style={[styles.itemRight, { paddingRight:5 }]}
                                                    numberOfLines={1}
                                                    ellipsizeMode='tail'>{endPoint}</Text>
                                                <Image
                                                    source={app.img.common_go}
                                                    style={styles.goStyle} />
                                            </View>
                                        </View>
                                    </TouchableOpacity>
                                :
                                <View>
                                    <TouchableOpacity
                                        activeOpacity={0.6}
                                        onPress={this.showEndPlace}>
                                        <View style={styles.item}>
                                            <Text style={styles.itemLeft}>到货地</Text>
                                            <View style={styles.itemTop}>
                                                <Text style={styles.itemRight}
                                                    numberOfLines={1}
                                                    ellipsizeMode='tail'>{endPoint}</Text>
                                                <Image
                                                    source={app.img.common_go}
                                                    style={styles.goStyle} />
                                            </View>
                                        </View>
                                    </TouchableOpacity>
                                    <TouchableOpacity onPress={this.setIsSendDoor}>
                                        <View style={styles.item}>
                                            <Text style={styles.itemLeft}>是否送货上门</Text>
                                            <View style={styles.itemTop}>
                                                <Text style={styles.itemRight}>{isSendDoor ? '是' : '否'}</Text>
                                                <Image
                                                    source={app.img.common_go}
                                                    style={styles.goStyle} />
                                            </View>
                                        </View>
                                    </TouchableOpacity>
                                    {
                                        isSendDoor &&
                                        <TouchableOpacity
                                            activeOpacity={0.6}
                                            onPress={this.showSendDoorEndPlace}>
                                            <View style={styles.item}>
                                                <Text style={styles.itemLeft}>送货上门地址</Text>
                                                <View style={styles.itemTop}>
                                                    <Text style={styles.itemRight}
                                                        numberOfLines={1}
                                                        ellipsizeMode='tail'>{sendDoorEndPoint}</Text>
                                                    <Image
                                                        source={app.img.common_go}
                                                        style={styles.goStyle} />
                                                </View>
                                            </View>
                                        </TouchableOpacity>
                                    }
                                </View>

                            }
                            <View style={styles.itemUnitTop}>
                                <Text style={styles.itemLeft}>重量</Text>
                                <TextInput style={styles.inputText}
                                    underlineColorAndroid='transparent'
                                    keyboardType='numeric'
                                    placeholder='请输入货物重量'
                                    defaultValue={weight + ''}
                                    onChangeText={(text) => this.setState({ weight: text })}
                                />
                                <Text style={styles.unit}>吨</Text>
                            </View>
                            <View style={styles.itemUnit}>
                                <Text style={styles.itemLeft}>方量</Text>
                                <TextInput style={styles.sizeInputText}
                                    underlineColorAndroid='transparent'
                                    keyboardType='numeric'
                                    placeholder='请输入货物方量'
                                    defaultValue={size + ''}
                                    onChangeText={(text) => this.setState({ size: text })}
                                />
                                <Text style={styles.unit}>m³</Text>
                            </View>
                            <View style={styles.itemUnit}>
                                <Text style={styles.itemLeft}>件数</Text>
                                <TextInput style={styles.inputText}
                                    underlineColorAndroid='transparent'
                                    keyboardType='numeric'
                                    placeholder='请输入货物件数'
                                    defaultValue={totalNumbers + ''}
                                    onChangeText={(text) => this.setState({ totalNumbers: text })}
                                />
                                <Text style={styles.unit}>件</Text>
                            </View>
                            <View style={styles.itemUnitTop}>
                                <Text style={styles.itemLeft}>代收货款</Text>
                                <TextInput style={styles.inputText}
                                    underlineColorAndroid='transparent'
                                    keyboardType='numeric'
                                    placeholder='0'
                                    defaultValue={proxyCharge + ''}
                                    onChangeText={(text) => this.setState({ proxyCharge : text })}
                                />
                                <Text style={styles.unit}>元</Text>
                            </View>
                            <View style={styles.itemUnit}>
                                <Text style={styles.itemLeft}>指定收款</Text>
                                <TextInput style={styles.inputText}
                                    underlineColorAndroid='transparent'
                                    keyboardType='numeric'
                                    placeholder='0'
                                    defaultValue={totalDesignatedFee + ''}
                                    onChangeText={(text) => this.setState({ totalDesignatedFee : text })}
                                />
                                <Text style={styles.unit}>元</Text>
                            </View>
                            <TouchableOpacity onPress={this.setIsInsuance}>
                                <View style={styles.item}>
                                    <Text style={styles.itemLeft}>是否保价</Text>
                                    <View style={styles.itemTop}>
                                        <Text style={styles.itemRight}>{isInsuance ? '是' : '否'}</Text>
                                        <Image
                                            source={app.img.common_go}
                                            style={styles.goStyle} />
                                    </View>
                                </View>
                            </TouchableOpacity>
                            <TouchableOpacity onPress={this.setIsReachPay}>
                                <View style={styles.item}>
                                    <Text style={styles.itemLeft}>是否到付</Text>
                                    <View style={styles.itemTop}>
                                        <Text style={styles.itemRight}>{isReachPay ? '是' : '否'}</Text>
                                        <Image
                                            source={app.img.common_go}
                                            style={styles.goStyle} />
                                    </View>
                                </View>
                            </TouchableOpacity>
                        </ScrollView> : <LookPreOrder param={this.state} />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    topImage:{
        width:sr.w,
        height:65,
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent: 'center',
    },
    topBigImage:{
        height:100,
        width:sr.w,
        padding:12,
        flexDirection:'row',
        alignItems:'center',
        justifyContent:'space-between',
        backgroundColor:'#FFFFFF',
    },
    photo:{
        height:23,
        width:25,
    },
    bigPicture:{
        height:80,
        width:120,
    },
    topImageText:{
        marginTop:10,
        fontSize:12,
        color:'#888888',
    },
    itemStart:{
        height:40,
        marginTop:8,
        justifyContent: 'space-between',
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    item:{
        height:40,
        marginTop:1,
        justifyContent: 'space-between',
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    itemTop:{
        flexDirection: 'row',
    },
    itemLeft:{
        flex:1,
        paddingLeft:12,
        fontSize:14,
        color:'#666666',
    },
    itemRight:{
        fontSize:14,
        color:'#333333',
        paddingRight:5,
        width:sr.w / 3 * 2,
        textAlign:'right',
    },
    goStyle: {
        height: 15,
        width: 8,
        marginRight: 12,
    },
    inputText:{
        flex:1,
        height:40,
        fontSize:14,
        width:sr.w / 2,
        marginRight:10,
        textAlign:'right',
    },
    sizeInputText:{
        flex:1,
        height:40,
        fontSize:14,
        width:sr.w / 2,
        marginRight:7,
        textAlign:'right',
    },
    itemUnitTop:{
        marginTop:8,
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    itemUnit:{
        marginTop:1,
        alignItems:'center',
        flexDirection: 'row',
        backgroundColor:'#FFFFFF',
    },
    unit:{
        paddingRight:12,
        fontSize:14,
        color:'#888888',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/PreOrderGroupList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;
const { PageList, Button, DImage } = COMPONENTS;
const PreOrderSelectList = require('./PreOrderSelectList');
const PreOrderListInGroup = require('./PreOrderListInGroup');
const LookPreOrderGroup = require('./LookPreOrderGroup');
const ConfirmMerge = require('../../common/ConfirmMerge');

module.exports = React.createClass({
    getInitialState () {
        return {
            list:[],
            isShowSelectAll:false,
        };
    },
    doSplitGroup (obj) {
        app.push({
            component:PreOrderListInGroup,
            passProps:{
                orderGroupId: obj.id,
                refresh: this.listView.refresh,
            },
        });
    },
    doLookDetail (obj) {
        app.push({
            component:LookPreOrderGroup,
            passProps:{
                param:obj,
                refresh: this.listView.refresh,
            },
        });
    },
    doChooseOne (obj, rowID) {
        const { list } = this.state;
        list[rowID].selected = !list[rowID].selected;
        this.setState({ list });
    },
    doSelectedAll () {
        const { list } = this.state;
        if (_.every(list, (o) => o.selected == true)) {
            _.forEach(list, o => { o.selected = false; });
        } else {
            _.forEach(list, o => { o.selected = true; });
        }
        this.setState({ list });
    },
    doMerge () {
        const { list } = this.state;
        const filterList = _.filter(list, o => o.selected);
        const orderGroupIdList = filterList.map(o => o.id);
        if (orderGroupIdList.length == 0) {
            Toast('请先勾选要合并的货单');
            return;
        }
        app.push({
            component:PreOrderSelectList,
            passProps:{
                doMergePreOrder: this.doMergePreOrder,
            },
        });
    },
    doMergePreOrder (orderIdList) {
        const { list } = this.state;
        const filterList = _.filter(list, o => o.selected);
        const orderGroupIdList = _.filter(filterList.map(o => o.id), undefined);
        if (!(orderGroupIdList.length <= 1 && orderIdList.length == 0)) {
            app.showModal(<ConfirmMerge doConfirm={(info) => {
                const param = {
                    userId: app.personal.info.userId,
                    orderGroupIdList,
                    orderIdList:orderIdList || [],
                    ...info,
                };
                POST(app.route.ROUTE_CREATE_PRE_ORDER_GROUP, param, this.doMergePreOrderSuccess, true);
            }} />);
        }
    },
    doMergePreOrderSuccess (data) {
        if (data.success) {
            const { list } = this.state;
            _.forEach(list, o => { o.selected = false; });
            this.setState({ list });
            this.listView.refresh();
            Toast('合并成功！');
        }
    },
    onGetList (data, pageNo) {
        if (!data.success || data.context.orderGroupList.length == 0 && pageNo == 0) {
            this.setState({ isShowSelectAll:false });
        } else {
            const { orderGroupList } = data.context;
            const list = _.uniqBy([...(_.map(orderGroupList, (v, k) => { return { id:v.id, selected:false }; })), ...this.state.list], 'id');
            this.setState({ isShowSelectAll:true, list });
        }
    },
    renderRow (obj, rowID) {
        const { list } = this.state;
        return (
            <View>
                <View style={styles.item}>
                    <TouchableOpacity style={styles.chooseView} onPress={this.doChooseOne.bind(null, obj, rowID)}>
                        <Image style={styles.chooseIcon} source={!list[rowID].selected ? app.img.order_no_choose : app.img.order_choose} />
                    </TouchableOpacity>
                    <TouchableOpacity style={styles.content} onPress={this.doLookDetail.bind(null, obj)}>
                        <DImage
                            resizeMode='stretch'
                            defaultSource={app.img.login_logo}
                            source={{ uri:obj.photo }}
                            style={styles.image}
                            />
                        <View>
                            <View style={styles.itemRow}>
                                <Text style={styles.itemRowText}>名称：{obj.name}</Text>
                                <Text style={styles.itemRowText}>总单数:{obj.orderCount}</Text>
                            </View>
                            <View style={styles.itemRow}>
                                <Text style={styles.itemRowText}>总方量:{obj.totalSize}</Text>
                                <Text style={styles.itemRowText}>总重量:{obj.totalWeight}</Text>
                            </View>
                            <View style={styles.itemRow}>
                                <Text style={styles.totalNumbers}>总数量:{obj.totalNumbers}</Text>
                                <Text style={styles.startPoint}
                                    numberOfLines={1}>始发地：{(obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : obj.startPoint)}</Text>
                            </View>
                        </View>
                    </TouchableOpacity>
                </View>
                <View style={styles.rowButtomContainer}>
                    <Button onPress={this.doSplitGroup.bind(null, obj)} style={styles.btnSplitGroup} textStyle={styles.btnSplitGroupText}>{'查看货单'}</Button>
                </View>
            </View>
        );
    },
    render () {
        const { list, isShowSelectAll } = this.state;
        const status = list.length != 0 && _.every(list, (o) => o.selected == true);
        return (
            <View style={styles.container}>
                <View style={styles.list}>
                    <PageList
                        style={styles.listStyle}
                        ref={(ref) => { this.listView = ref; }}
                        renderRow={this.renderRow}
                        onGetList={this.onGetList}
                        listParam={{ userId: app.personal.info.userId }}
                        listName={'orderGroupList'}
                        listUrl={app.route.ROUTE_GET_PRE_ORDER_GROUP_LIST}
                        refreshEnable
                        />
                </View>
                {
                    isShowSelectAll &&
                    <View style={styles.bottom}>
                        <TouchableOpacity style={styles.chooseAll} onPress={this.doSelectedAll}>
                            <Image
                                resizeMode='cover'
                                style={styles.imgChoose}
                                source={status ? app.img.order_choose : app.img.order_no_choose} />
                            <Text style={styles.chooseAllFont}>全选</Text>
                        </TouchableOpacity>
                        <Button onPress={this.doMerge}
                            style={styles.btnMerge}
                            textStyle={styles.btnMergeFont}>合并</Button>
                    </View>
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    item:{
        height:70,
        flexDirection: 'row',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
    },
    content:{
        flexDirection: 'row',
        alignItems:'center',
    },
    chooseView :{
        height:70,
        width:30,
        justifyContent:'center',
        alignItems:'center',
    },
    chooseIcon:{
        height:12,
        width:12,
    },
    image:{
        height:50,
        width:50,
        marginRight:5,
    },
    itemRow:{
        width:sr.w - 90,
        flexDirection: 'row',
        alignItems:'center',
        justifyContent:'space-between',
    },
    itemRowText:{
        fontSize:12,
        color:'#888888',
        marginTop:1,
        marginBottom:2,
    },
    totalNumbers:{
        fontSize:12,
        color:'#888888',
        marginTop:3,
        width:sr.w / 5,
    },
    startPoint:{
        fontSize:12,
        color:'#888888',
        marginTop:3,
        width:sr.w / 5 * 3,
    },
    bottom:{
        height:50,
        width:sr.w,
        flexDirection: 'row',
        alignItems:'center',
        justifyContent:'space-between',
        backgroundColor:'#FFFFFF',
    },
    chooseAllFont:{
        fontSize:14,
        marginLeft:10,
    },
    chooseAll:{
        alignItems:'center',
        flexDirection:'row',
    },
    imgChoose:{
        marginLeft:10,
        width:12,
        height:12,
    },
    btnMerge:{
        width:60,
        height:25,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#f21620',
        marginRight:15,
    },
    btnMergeFont:{
        color:'#f21620',
        fontSize:13,
        fontWeight:'400',
    },
    rowButtomContainer: {
        backgroundColor:'white',
        flexDirection: 'row',
        justifyContent: 'flex-end',
        paddingVertical:5,
    },
    btnSplitGroup: {
        height:20,
        width:73,
        borderRadius:5,
        borderColor:'#007ef7',
        borderWidth:1,
        marginLeft:20,
        marginRight:10,
        backgroundColor:'white',
    },
    btnSplitGroupText:{
        color:'#007ef7',
        fontSize:12,
        fontWeight:'300',
    },
    list:{
        width:sr.w,
        flex:1,
    },
    listStyle:{
        marginTop:10,
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/PreOrderList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

import Swipeout from 'react-native-swipeout';

const { PageList, Button, MessageBox, DImage } = COMPONENTS;
const PreOrderDetail = require('./PreOrderDetail');
const ConfirmMerge = require('../../common/ConfirmMerge');

module.exports = React.createClass({
    getInitialState () {
        return {
            list:[],
            isShowSelectAll:false,
        };
    },
    doDetail (obj, sectionID) {
        app.push({
            component:PreOrderDetail,
            passProps:{
                info:obj,
                updateList: (data) => {
                    this.listView.updateList((list) => {
                        list[sectionID] = data;
                        return list;
                    });
                },
            },
        });
    },
    doChooseOne (obj, rowID) {
        const { list } = this.state;
        list[rowID].selected = !list[rowID].selected;
        this.setState({ list });
    },
    doSelectedAll () {
        const { list } = this.state;
        if (_.every(list, (o) => o.selected == true)) {
            _.forEach(list, o => { o.selected = false; });
        } else {
            _.forEach(list, o => { o.selected = true; });
        }
        this.setState({ list });
    },
    doMergePreOrder () {
        const { list } = this.state;
        const filterList = _.filter(list, o => o.selected);
        const orderIdList = filterList.map(o => o.id);
        if (filterList.length == 0) {
            Toast('你还没选择需要合并的货单');
            return;
        }
        app.showModal(<ConfirmMerge doConfirm={(info) => {
            const param = {
                userId: app.personal.info.userId,
                orderIdList,
                ...info,
            };
            POST(app.route.ROUTE_CREATE_PRE_ORDER_GROUP, param, this.doMergePreOrderSuccess, true);
        }} />);
    },
    doMergePreOrderSuccess (data) {
        if (data.success) {
            const { list } = this.state;
            _.forEach(list, o => { o.selected = false; });
            this.setState({ list });
            this.listView.refresh();
            Toast('合并成功！');
        }
    },
    onGetList (data, pageNo) {
        if (!data.success || data.context.orderList.length == 0 && pageNo == 0) {
            this.setState({ isShowSelectAll:false });
        } else {
            const { orderList } = data.context;
            const list = _.uniqBy([...(_.map(orderList, (v, k) => { return { id:v.id, selected:false }; })), ...this.state.list], 'id');
            this.setState({ isShowSelectAll:true, list });
        }
    },
    refresh () {
        this.listView.refresh();
    },
    renderRow (obj, rowID) {
        let { list } = this.state;
        return (
                <View style={styles.item}>
                    <TouchableOpacity style={styles.chooseView} onPress={this.doChooseOne.bind(null, obj, rowID)}>
                        <Image style={styles.chooseIcon} source={!list[rowID].selected ? app.img.order_no_choose : app.img.order_choose} />
                    </TouchableOpacity>
                    <TouchableOpacity style={styles.content} onPress={this.doDetail.bind(null, obj, rowID)}>
                        <DImage
                            resizeMode='stretch'
                            defaultSource={app.img.common_default}
                            source={{ uri:obj.photo }}
                            style={styles.image}
                        />
                        <View style={styles.row}>
                            <View style={styles.itemTop}>
                                <Text style={styles.itemText}>始发地：{ (obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : obj.startPoint)}</Text>
                            </View>
                            <Text style={styles.itemText}>
                            收货人：{obj.receiverName ? obj.receiverName : obj.receiverName == '' ? obj.receiverPhone : obj.receiverName}
                            </Text>
                            <Text style={styles.itemText}
                                numberOfLines={1}>到货地：{obj.endPoint}</Text>
                        </View>
                    </TouchableOpacity>
                </View>
        );
    },
    render () {
        const { list, isShowSelectAll } = this.state;
        const status = list.length != 0 && _.every(list, (o) => o.selected == true);
        return (
            <View style={styles.container}>
                <View style={styles.list}>
                    <PageList
                        ref={(ref) => { this.listView = ref; }}
                        renderRow={this.renderRow}
                        onGetList={this.onGetList}
                        listParam={{ userId: app.personal.info.userId }}
                        listName={'orderList'}
                        listUrl={app.route.ROUTE_GET_PRE_ORDER_LIST}
                        refreshEnable
                        />
                </View>
                {
                        isShowSelectAll &&
                        <View style={styles.bottom}>
                            <TouchableOpacity style={styles.chooseAll} onPress={this.doSelectedAll}>
                                <Image
                                    style={styles.imgChoose}
                                    source={status ? app.img.order_choose : app.img.order_no_choose} />
                                <Text style={styles.chooseAllFont}>全选</Text>
                            </TouchableOpacity>
                            <Button onPress={this.doMergePreOrder}
                                style={styles.btnMerge}
                                textStyle={styles.btnMergeFont}>合并</Button>
                        </View>
                    }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
        paddingTop:10,
    },
    item:{
        height:70,
        flexDirection: 'row',
        alignItems:'center',
    },
    content:{
        flexDirection: 'row',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
    },
    contentFee: {
        flexDirection: 'row',
        flex:1,
        alignItems:'center',
        justifyContent:'flex-end',
    },
    chooseView :{
        height:70,
        width:30,
        justifyContent:'center',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
    },
    chooseIcon:{
        height:12,
        width:12,
    },
    image:{
        height:50,
        width:50,
        marginRight:5,
    },
    itemTop:{
        flex:1,
        flexDirection: 'row',
    },
    itemText:{
        fontSize:12,
        color:'#888888',
        flex:1,
    },
    rightText:{
        fontSize:12,
        color:'#333333',
    },
    rightMoney:{
        fontSize:14,
        color:'red',
        marginRight:10,
    },
    bottom:{
        height:50,
        width:sr.w,
        flexDirection: 'row',
        alignItems:'center',
        justifyContent:'space-between',
        backgroundColor:'#FFFFFF',
    },
    chooseAllFont:{
        fontSize:14,
        marginLeft:10,
    },
    chooseAll:{
        alignItems:'center',
        flexDirection:'row',
    },
    imgChoose:{
        width:12,
        height:12,
        marginLeft:10,
    },
    btnShipperFont:{
        color:'#f01725',
        fontSize:12,
    },
    btnMerge:{
        width:60,
        height:25,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#f21620',
        marginRight:15,
    },
    btnMergeFont:{
        color:'#f21620',
        fontSize:13,
        fontWeight:'400',
    },
    point:{
        flex:1,
        flexDirection:'row',
    },
    btnAddPreOrder:{
        width:150,
        height:35,
        backgroundColor:'#FE9917',
        borderRadius: 4,
        alignSelf:'center',
        marginBottom:10,
    },
    btnAddPreOrderFont:{
        fontWeight:'500',
    },
    list:{
        width:sr.w,
        flex:1,
    },
    row:{
        flex:1,
        height:60,
        marginTop:10,
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/PreOrderListInGroup.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    ListView,
    TouchableOpacity,
} = ReactNative;

import Swipeout from 'react-native-swipeout';

const { Button, MessageBox, DImage } = COMPONENTS;
const PreOrderDetail = require('./PreOrderDetail');

module.exports = React.createClass({
    statics: {
        title: '货单列表',
        leftButton: { handler: () => { app.scene.refresh&&app.scene.refresh(); } },
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            list:[],
            orderList:[],
        };
    },
    refresh () {
        this.props.refresh();
        app.pop();
    },
    doDetail (obj, sectionID) {
        app.push({
            component:PreOrderDetail,
            passProps:{
                info:obj,
            },
        });
    },
    componentWillMount () {
        const { orderGroupId } = this.props;
        this.getOrderListInGroup(orderGroupId);
    },
    getOrderListInGroup (orderGroupId) {
        const param = {
            userId: app.personal.info.userId,
            orderGroupId,
        };
        POST(app.route.ROUTE_GET_PRE_ORDER_LIST_IN_GROUP, param, this.getOrderListInGroupSuccess, true);
    },
    getOrderListInGroupSuccess (data) {
        if (data.success) {
            const { orderList } = data.context;
            const list = _.map(orderList, (v, k) => { return { id:v.id, selected:false }; });
            this.setState({ orderList, list });
        } else {
            Toast(data.msg);
        }
    },
    doConfirmDelete (obj) {
        const param = {
            userId: app.personal.info.userId,
            orderId: obj.id,
        };
        POST(app.route.ROUTE_REMOVE_PRE_ORDER, param, this.doRemovePreOrderSuccess.bind(null, obj.id), true);
    },
    doRemovePreOrderSuccess (orderId, data) {
        if (data.success) {
            var orderList = this.state.orderList;
            _.remove(orderList, function (o) {
                return o.id == orderId;
            });
            this.setState({ orderList });
            Toast('删除成功');
        } else {
            Toast(data.msg);
        }
    },
    showRemoveModel (obj) {
        app.showModal(
            <MessageBox onConfirm={this.doConfirmDelete.bind(null, obj)} content={'是否确定删除'} onCancel title={false} width={sr.s(250)} />
        );
    },
    doChooseOne (obj, rowID) {
        const { list } = this.state;
        list[rowID].selected = !list[rowID].selected;
        this.setState({ list });
    },
    doSelectedAll () {
        const { list } = this.state;
        if (_.every(list, (o) => o.selected == true)) {
            _.forEach(list, o => { o.selected = false; });
        } else {
            _.forEach(list, o => { o.selected = true; });
        }
        this.setState({ list });
    },
    doSplitPreOrder (orderGroupId) {
        const { list } = this.state;
        const filterList = _.filter(list, o => o.selected);
        const orderIdList = filterList.map(o => o.id);
        if (orderIdList.length == 0) {
            Toast('请选择要拆分的货单');
            return;
        }
        const param = {
            userId: app.personal.info.userId,
            orderIdList,
            orderGroupId,
        };
        POST(app.route.ROUTE_REMOVE_PRE_ORDER_LIST_FROM_GROUP, param, this.doSplitPreOrderSuccess.bind(null, orderIdList), true);
    },
    doSplitPreOrderSuccess (orderIdList, data) {
        if (data.success) {
            var { orderList, list } = this.state;
            _.remove(orderList, obj => { return _.find(orderIdList, o => { return o == obj.id; }); });
            _.forEach(list, o => { o.selected = false; });
            this.setState({ list, orderList });
            Toast('拆分成功！');
        }
    },
    renderRow (obj, sectionID, rowID) {
        const { list } = this.state;
        return (
            <Swipeout right={[
                {
                    text: '删除',
                    backgroundColor: '#c81622',
                    onPress: this.showRemoveModel.bind(null, obj),
                },
            ]}
                autoClose>
                <View style={styles.item}>
                    <TouchableOpacity style={styles.chooseView} onPress={this.doChooseOne.bind(null, obj, rowID)}>
                        <Image style={styles.chooseIcon} source={!list[rowID].selected ? app.img.order_no_choose : app.img.order_choose} />
                    </TouchableOpacity>
                    <TouchableOpacity style={styles.content} onPress={this.doDetail.bind(null, obj, rowID)}>
                        <DImage
                            resizeMode='stretch'
                            defaultSource={app.img.common_default}
                            source={{ uri:obj.photo }}
                            style={styles.image}
                        />
                        <View style={{ flex:1 }}>
                            <View style={styles.itemTop}>
                                <Text style={styles.itemText}>始发地：{(obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : obj.startPoint)}</Text>
                                <View style={styles.contentFee}>
                                    <Text style={styles.rightText}>运费:</Text>
                                    <Text style={styles.rightMoney}>¥{obj.proxyCharge ? obj.proxyCharge : 0}</Text>
                                </View>
                            </View>
                            <Text style={styles.itemText}>
                            收货人：{obj.receiverName ? obj.receiverName : obj.receiverName == '' ? obj.receiverPhone : obj.receiverName}
                            </Text>
                            <Text style={styles.itemText}>地址：{obj.endPoint}</Text>
                        </View>
                    </TouchableOpacity>
                </View>
            </Swipeout>

        );
    },
    render () {
        const { list, orderList } = this.state;
        const { orderGroupId } = this.props;
        const status = list.length != 0 && _.every(list, (o) => o.selected == true);
        return (
            <View style={styles.container}>
                <ListView
                    initialListSize={1}
                    onEndReachedThreshold={10}
                    enableEmptySections
                    dataSource={this.ds.cloneWithRows(orderList)}
                    renderRow={this.renderRow}
                    renderSeparator={this.renderSeparator} />
                <View style={styles.bottom}>
                    <TouchableOpacity style={styles.chooseAll} onPress={this.doSelectedAll}>
                        <Image
                            resizeMode='cover'
                            style={styles.imgChoose}
                            source={status ? app.img.order_choose : app.img.order_no_choose} />
                        <Text style={styles.chooseAllFont}>全选</Text>
                    </TouchableOpacity>
                    <Button onPress={this.doSplitPreOrder.bind(null, orderGroupId)}
                        style={styles.btnSplit}
                        textStyle={styles.btnSplitFont}>拆分</Button>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
        paddingTop:10,
    },
    item:{
        height:71,
        flexDirection: 'row',
        alignItems:'center',
    },
    content:{
        height:70,
        flex:1,
        flexDirection: 'row',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
    },
    contentFee: {
        flexDirection: 'row',
        justifyContent:'flex-end',
        flex:1,
        marginTop:-2,
    },
    chooseView :{
        height:70,
        width:30,
        justifyContent:'center',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
    },
    chooseIcon:{
        height:12,
        width:12,
    },
    image:{
        height:50,
        width:50,
        marginRight:5,
    },
    itemTop:{
        flexDirection: 'row',
        alignItems:'flex-end',
        marginBottom:8,
    },
    itemText:{
        fontSize:12,
        color:'#888888',
        flex:2,
    },
    rightText:{
        fontSize:12,
        color:'#333333',
        marginTop:8,
    },
    rightMoney:{
        fontSize:14,
        color:'red',
        marginRight:10,
        marginTop:6,
    },
    bottom:{
        position:'absolute',
        bottom:0,
        height:50,
        width:sr.w,
        flexDirection: 'row',
        alignItems:'center',
        justifyContent:'space-between',
        backgroundColor:'#FFFFFF',
    },
    chooseAllFont:{
        fontSize:14,
        marginLeft:10,
    },
    chooseAll:{
        alignItems:'center',
        flexDirection:'row',
    },
    imgChoose:{
        marginLeft:5,
        width:12,
        height:12,
    },
    imgLine:{
        marginLeft:10,
        width:35,
    },
    pointFont:{
        fontSize:14,
        marginLeft:10,
        width:100,
    },
    btnShipperFont:{
        color:'#f01725',
        fontSize:12,
    },
    btnSplit:{
        width:60,
        height:25,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#f21620',
        marginRight:15,
    },
    btnSplitFont:{
        color:'#f21620',
        fontSize:13,
        fontWeight:'400',
    },
    point:{
        flex:1,
        flexDirection:'row',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/PreOrderSelectList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
    TouchableOpacity,
} = ReactNative;

import Swipeout from 'react-native-swipeout';

const { PageList, Button, MessageBox, DImage } = COMPONENTS;
const PreOrderDetail = require('./PreOrderDetail');

module.exports = React.createClass({
    statics: {
        title: '选择要合并货单',
        leftButton: { handler: () => { app.scene.doConcelSelectPreOrder&&app.scene.doConcelSelectPreOrder(); } },
    },
    getInitialState () {
        return {
            list:[],
            isShowSelectAll:false,
        };
    },
    doDetail (obj, sectionID) {
        app.push({
            component:PreOrderDetail,
            passProps:{
                info:obj,
                updateList: (data) => {
                    this.listView.updateList((list) => {
                        list[sectionID] = data;
                        return list;
                    });
                },
            },
        });
    },
    doConfirmDelete (obj) {
        const param = {
            userId: app.personal.info.userId,
            orderId: obj.id,
        };
        POST(app.route.ROUTE_REMOVE_PRE_ORDER, param, this.doRemovePreOrderSuccess.bind(null, obj.id), true);
    },
    doRemovePreOrderSuccess (orderId, data) {
        if (data.success) {
            var list = this.listView.list;
            _.remove(list, function (o) {
                return o.id == orderId;
            });
            this.setState({ list });
            Toast('删除成功');
        } else {
            Toast(data.msg);
        }
    },
    showRemoveModel (obj) {
        app.showModal(
            <MessageBox onConfirm={this.doConfirmDelete.bind(null, obj)} content={'是否确定删除'} onCancel title={false} width={sr.s(250)} />
        );
    },
    doChooseOne (obj, rowID) {
        const { list } = this.state;
        list[rowID].selected = !list[rowID].selected;
        this.setState({ list });
    },
    doSelectedAll () {
        const { list } = this.state;
        if (_.every(list, (o) => o.selected == true)) {
            _.forEach(list, o => { o.selected = false; });
        } else {
            _.forEach(list, o => { o.selected = true; });
        }
        this.setState({ list });
    },
    doConfirmSelectPreOrder () {
        const { list } = this.state;
        const filterList = _.filter(list, o => o.selected);
        const orderIdList = filterList.map(o => o.id);
        this.doMergePreOrder(orderIdList);
    },
    doConcelSelectPreOrder () {
        this.doMergePreOrder([]);
    },
    doMergePreOrder (orderIdList) {
        this.props.doMergePreOrder(orderIdList);
        app.pop();
    },
    onGetList (data, pageNo) {
        if (!data.success || data.context.orderList.length == 0 && pageNo == 0) {
            this.setState({ isShowSelectAll:false });
        } else {
            const { orderList } = data.context;
            const list = _.uniqBy([...(_.map(orderList, (v, k) => { return { id:v.id, selected:false }; })), ...this.state.list], 'id');
            this.setState({ isShowSelectAll:true, list });
        }
    },
    renderRow (obj, rowID) {
        let { list } = this.state;
        return (
            <Swipeout right={[
                {
                    text: '删除',
                    backgroundColor: '#c81622',
                    onPress: this.showRemoveModel.bind(null, obj),
                },
            ]}
                autoClose>
                <View style={styles.item}>
                    <TouchableOpacity style={styles.chooseView} onPress={this.doChooseOne.bind(null, obj, rowID)}>
                        <Image style={styles.chooseIcon} source={!list[rowID].selected ? app.img.order_no_choose : app.img.order_choose} />
                    </TouchableOpacity>
                    <TouchableOpacity style={styles.content} onPress={this.doDetail.bind(null, obj, rowID)}>
                        <DImage
                            resizeMode='stretch'
                            defaultSource={app.img.common_default}
                            source={{ uri:obj.photo }}
                            style={styles.image}
                        />
                        <View>
                            <View style={styles.itemTop}>
                                <Text style={styles.itemText}
                                    numberOfLines={1}>始发地：{(obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : obj.startPoint)}</Text>
                                <View style={styles.contentFee}>
                                    <Text style={styles.rightText}>运费:</Text>
                                    <Text style={styles.rightMoney}>¥{obj.proxyCharge ? obj.proxyCharge : 0}</Text>
                                </View>
                            </View>
                            <Text style={styles.itemText}>
                            收货人：{obj.receiverName ? obj.receiverName : obj.receiverName == '' ? obj.receiverPhone : obj.receiverName}
                            </Text>
                            <Text style={styles.itemText}
                                numberOfLines={1}>地址：{obj.endPoint}</Text>
                        </View>
                    </TouchableOpacity>
                </View>
            </Swipeout>

        );
    },
    render () {
        const { list, isShowSelectAll } = this.state;
        const status = list.length != 0 && _.every(list, (o) => o.selected == true);
        return (
            <View style={styles.container}>
                <PageList
                    ref={(ref) => { this.listView = ref; }}
                    renderRow={this.renderRow}
                    onGetList={this.onGetList}
                    listParam={{ userId: app.personal.info.userId }}
                    listName={'orderList'}
                    listUrl={app.route.ROUTE_GET_PRE_ORDER_LIST}
                    refreshEnable
                    />
                {
                        isShowSelectAll &&
                        <View style={styles.bottom}>
                            <TouchableOpacity style={styles.chooseAll} onPress={this.doSelectedAll}>
                                <Image
                                    resizeMode='cover'
                                    style={styles.imgChoose}
                                    source={status ? app.img.order_choose : app.img.order_no_choose} />
                                <Text style={styles.chooseAllFont}>全选</Text>
                            </TouchableOpacity>
                            <Button onPress={this.doConcelSelectPreOrder}
                                style={styles.btnMerge}
                                textStyle={styles.btnMergeFont}>取消</Button>
                            <Button onPress={this.doConfirmSelectPreOrder}
                                style={styles.btnMerge}
                                textStyle={styles.btnMergeFont}>确定</Button>
                        </View>
                    }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
        paddingTop:10,
    },
    item:{
        height:70,
        flexDirection: 'row',
        alignItems:'center',
    },
    content:{
        height:70,
        flex:1,
        flexDirection: 'row',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
    },
    contentFee: {
        flexDirection: 'row',
        alignItems:'center',
        justifyContent:'flex-end',
    },
    chooseView :{
        height:70,
        width:30,
        justifyContent:'center',
        alignItems:'center',
        backgroundColor:'#FFFFFF',
    },
    chooseIcon:{
        height:12,
        width:12,
    },
    image:{
        height:50,
        width:50,
        marginRight:5,
    },
    itemTop:{
        flexDirection: 'row',
    },
    itemText:{
        fontSize:12,
        width:sr.w / 5 * 3,
        color:'#888888',
        marginTop:3,
    },
    rightText:{
        fontSize:12,
        color:'#333333',
    },
    rightMoney:{
        fontSize:14,
        color:'red',
        marginRight:10,
    },
    bottom:{
        position:'absolute',
        bottom:0,
        height:50,
        width:sr.w,
        flexDirection: 'row',
        alignItems:'center',
        justifyContent:'flex-end',
        backgroundColor:'#FFFFFF',
    },
    chooseAllFont:{
        fontSize:14,
        marginLeft:10,
        marginRight:sr.w - 210,
    },
    chooseAll:{
        alignItems:'center',
        flexDirection:'row',
    },
    imgChoose:{
        marginLeft:5,
        width:12,
        height:12,
    },
    imgLine:{
        marginLeft:10,
        width:35,
    },
    pointFont:{
        fontSize:14,
        marginLeft:10,
        width:100,
    },
    btnShipperFont:{
        color:'#f01725',
        fontSize:12,
    },
    btnMerge:{
        width:60,
        height:25,
        backgroundColor:'#FFFFFF',
        borderWidth:0.5,
        borderRadius:2,
        borderColor:'#f21620',
        marginRight:15,
    },
    btnMergeFont:{
        color:'#f21620',
        fontSize:13,
        fontWeight:'400',
    },
    point:{
        flex:1,
        flexDirection:'row',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/ReceivePointSearchList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const { PageList } = COMPONENTS;
let km = '';

module.exports = React.createClass({
    renderRow (obj, rowID) {
        if (obj.distance) {
            if (obj.distance > 1) {
                km = parseInt(obj.distance) + 'km';
            } else {
                km = parseInt((obj.distance) * 1000) + 'm';
            }
        }
        return (
            <View style={styles.row}>
                <View style={styles.rowChild}>
                    <View style={styles.address}>
                        <Text style={styles.rowChildFont} numberOfLines={1}>{obj.shipperName}</Text>
                        <Text >{'-'}</Text>
                        <Text style={styles.rowChildFont} numberOfLines={1}>{obj.endPoint}</Text>
                    </View>
                    <Text style={styles.rowChildDistance}>距我:{km}</Text>
                </View>
                <View style={styles.itemInfo}>
                    <View style={styles.item}>
                        <Text style={styles.rowChildNumber}>￥{app.utils.N(parseInt(obj.price)) + '/吨'}</Text>
                        <Text style={styles.rowChildPrice}>运费单价</Text>
                    </View>
                    {
                        obj.sendDoorPrice && <View style={styles.item}>
                            <Text style={styles.rowChildNumber}>￥{app.utils.N(parseInt(obj.sendDoorPrice)) + '/吨'}</Text>
                            <Text style={styles.rowChildTitle}>送货上门</Text>
                        </View>
                    }
                    <View style={styles.item}>
                        <Text style={styles.rowChildNumber}>￥{app.utils.N(parseInt(obj.selfSignRate)) + '/%'}</Text>
                        <Text style={styles.rowChildTitle}>亲签率</Text>
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.rowChildTime}>{obj.duration}天</Text>
                        <Text style={styles.rowChildTitle}>预计用时</Text>
                    </View>
                </View>
            </View>
        );
    },
    render () {
        const { index, tabIndex, mapInfo, orderInfo, listName, data } = this.props;
        return (
            <View style={index === tabIndex ? styles.container : { left:-sr.tw, top:0, position:'absolute' }}>
                {
                    !!data &&
                    <PageList
                        ref={(ref) => { this.listView = ref; }}
                        renderRow={this.renderRow}
                        listParam={{ userId: app.personal.info.userId, orderBy: index, ...mapInfo, ...orderInfo }}
                        listName={listName}
                        list={data.roadmapList}
                        autoLoad={false}
                        listUrl={app.route.ROUTE_GET_ROADMAP_LIST_WITH_PRE_ORDER}
                        refreshEnable
                        />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    row: {
        flex:1,
        paddingVertical: 20,
        paddingHorizontal: 2,
    },
    rowChild:{
        width:sr.w,
        flexDirection:'row',
        marginTop:3,
        marginBottom:3,
    },
    rowChildFont:{
        fontSize:14,
        marginLeft:5,
        fontWeight:'300',
    },
    rowChildDistance:{
        fontSize:12,
        color:'#FF9D24',
        fontWeight:'300',
        marginRight:5,
        flex:0.5,
    },
    rowChildNumber:{
        color:'#f61525',
        fontSize:15,
        fontWeight:'500',
    },
    rowChildTime:{
        fontSize:16,
        color:'#333333',
    },
    rowChildTitle:{
        flex:1,
        fontSize:12,
        color:'#333333',
    },
    rowChildPrice:{
        flex:1,
        fontSize:12,
        color:'#333333',
    },
    address:{
        flexDirection:'row',
        flex:1.5,
    },
    itemInfo:{
        flexDirection:'row',
        flex:1,
        alignItems:'center',
        justifyContent:'center',
        marginTop:3,
    },
    item:{
        flex:1,
        alignItems:'center',
        justifyContent:'center',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/ReceivePointSearchListGroup.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const { PageList } = COMPONENTS;
let km = '';

module.exports = React.createClass({
    renderRow (obj, rowID) {
        if (obj.distance) {
            if (obj.distance > 1) {
                km = parseInt(obj.distance) + 'km';
            } else {
                km = parseInt((obj.distance) * 1000) + 'm';
            }
        }
        return (
            <View style={styles.row}>
                <View style={styles.rowChild}>
                    <Text style={styles.shopName} numberOfLines={1}>{obj.name}</Text>
                    <View style={styles.price}>
                        <Text style={styles.rowChildFont} numberOfLines={1}>总运费:</Text>
                        <Text style={styles.rowChildFont} numberOfLines={1}>{app.utils.N(parseInt(obj.totalTransportFee)) + '元'}</Text>
                    </View>
                    <Text style={styles.rowChildDistance}>距我:{km}</Text>
                </View>
            </View>
        );
    },
    render () {
        const { index, tabIndex, mapInfo, orderInfo, listName, data } = this.props;
        return (
            <View style={index === tabIndex ? styles.container : { left:-sr.tw, top:0, position:'absolute' }}>
                {
                    !!data &&
                    <PageList
                        ref={(ref) => { this.listView = ref; }}
                        renderRow={this.renderRow}
                        listParam={{ userId: app.personal.info.userId, orderBy: index, ...mapInfo, ...orderInfo }}
                        listName={listName}
                        list={data}
                        autoLoad={false}
                        listUrl={app.route.ROUTE_GET_ROADMAP_LIST_WITH_PRE_ORDER_GROUP}
                        refreshEnable
                        />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    row: {
        flex:1,
        paddingVertical: 20,
        paddingHorizontal: 2,
        backgroundColor:'#FFFFFF',
        marginTop:1,
    },
    rowChild:{
        width:sr.w,
        flexDirection:'row',
        marginTop:3,
        marginBottom:3,
    },
    shopName:{
        flex:2,
        fontSize:14,
        marginLeft:5,
        fontWeight:'300',
    },
    rowChildFont:{
        fontSize:13,
        marginLeft:5,
        color:'#c81622',
        fontWeight:'300',
    },
    rowChildDistance:{
        fontSize:12,
        color:'#FF9D24',
        fontWeight:'300',
        marginRight:15,
        textAlign:'right',
        flex:1,
    },
    price:{
        flexDirection:'row',
        flex:1,
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/ReceivePointSearchResult.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

const ReceivePointSearchList = require('./ReceivePointSearchList');
module.exports = React.createClass({
    statics: {
        title: '查询结果',
    },
    getInitialState () {
        return {
            tabIndex: 0,
        };
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    render () {
        const menus = ['价格最低', '时间最短', '亲签率最高', '距离最短'];
        const { tabIndex } = this.state;
        const { mapInfo, orderInfo, listName, data } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.tabContainer}>
                    {
                        menus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={this.changeTab.bind(null, i)}
                                style={styles.touchTab}>
                                <View style={styles.tabButton}>
                                    <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                        {item}
                                    </Text>
                                    <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                </View>
                            </TouchableHighlight>
                        ))
                    }
                </View>
                {
                    menus.map((item, i) => (
                        <ReceivePointSearchList key={i} index={i} tabIndex={tabIndex} mapInfo={mapInfo} orderInfo={orderInfo} listName={'agent.' + listName} data={data} />
                    ))
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonCenter: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonRight: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        borderRadius: 9,
    },
    tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/ReceivePointSearchResultGroup.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

const ReceivePointSearchListGroup = require('./ReceivePointSearchListGroup');
module.exports = React.createClass({
    statics: {
        title: '查询结果',
    },
    getInitialState () {
        return {
            tabIndex: 0,
        };
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    render () {
        const menus = ['价格最低', '距离最短'];
        const { tabIndex } = this.state;
        const { mapInfo, orderInfo, listName, data } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.tabContainer}>
                    {
                        menus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={this.changeTab.bind(null, i)}
                                style={styles.touchTab}>
                                <View style={styles.tabButton}>
                                    <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                        {item}
                                    </Text>
                                    <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                </View>
                            </TouchableHighlight>
                        ))
                    }
                </View>
                {
                    menus.map((item, i) => (
                        <ReceivePointSearchListGroup key={i} index={i} tabIndex={tabIndex} mapInfo={mapInfo} orderInfo={orderInfo} listName={listName} data={data} />
                    ))
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonCenter: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonRight: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        borderRadius: 9,
    },
    tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/SupermarketSearchList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const { PageList } = COMPONENTS;
let km = '';

module.exports = React.createClass({
    renderRow (obj, rowID) {
        if (obj.distance) {
            if (obj.distance > 1) {
                km = parseInt(obj.distance) + 'km';
            } else {
                km = parseInt((obj.distance) * 1000) + 'm';
            }
        }
        return (
            <View style={styles.row}>
                <Text style={styles.rowChildCenter} numberOfLines={1}>{obj.shopName}</Text>
                <View style={styles.rowChild}>
                    <View style={styles.address}>
                        <Text style={styles.rowChildFont} numberOfLines={1}>{obj.shipperName}</Text>
                        <Text >{'-'}</Text>
                        <Text style={styles.rowChildFont} numberOfLines={1}>{obj.endPoint}</Text>
                    </View>
                    <Text style={styles.rowChildDistance}>距我:{km}</Text>
                </View>
                <View style={styles.itemInfo}>
                    <View style={styles.item}>
                        <Text style={styles.rowChildNumber}>￥{app.utils.N(parseInt(obj.transportFee))}</Text>
                        <Text style={styles.rowChildPrice}>运费</Text>
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.rowChildTime}>{app.utils.N(parseInt(obj.selfSignRate)) + '%'}</Text>
                        <Text style={styles.rowChildTitle}>亲签率</Text>
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.rowChildTime}>{obj.duration}天</Text>
                        <Text style={styles.rowChildTitle}>预计用时</Text>
                    </View>
                </View>
            </View>
        );
    },
    render () {
        const { index, tabIndex, mapInfo, orderInfo, listName, data } = this.props;
        return (
            <View style={index === tabIndex ? styles.container : { left:-sr.tw, top:0, position:'absolute' }}>
                {
                    !!data &&
                    <PageList
                        ref={(ref) => { this.listView = ref; }}
                        renderRow={this.renderRow}
                        listParam={{ userId: app.personal.info.userId, orderBy: index, ...mapInfo, ...orderInfo }}
                        listName={listName}
                        list={data && data.roadmapList}
                        autoLoad={false}
                        listUrl={app.route.ROUTE_GET_ROADMAP_LIST_WITH_PRE_ORDER}
                        refreshEnable
                        />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    row: {
        flex:1,
        paddingVertical: 20,
        paddingHorizontal: 2,
        backgroundColor:'#FFFFFF',
        marginTop:1,
    },
    rowChild:{
        width:sr.w,
        flexDirection:'row',
        marginTop:3,
        marginBottom:3,
    },
    rowChildCenter:{
        fontSize:14,
        paddingBottom:5,
        fontWeight:'300',
        alignSelf:'center',
    },
    rowChildFont:{
        fontSize:14,
        marginLeft:5,
        fontWeight:'300',
    },
    rowChildDistance:{
        fontSize:12,
        color:'#FF9D24',
        fontWeight:'300',
        marginRight:5,
        flex:0.5,
    },
    rowChildNumber:{
        color:'#f61525',
        fontSize:15,
        fontWeight:'500',
    },
    rowChildTime:{
        fontSize:16,
        color:'#333333',
    },
    rowChildTitle:{
        flex:1,
        fontSize:12,
        color:'#333333',
    },
    rowChildPrice:{
        flex:1,
        fontSize:12,
        color:'#333333',
    },
    address:{
        flexDirection:'row',
        flex:1.5,
    },
    itemInfo:{
        flexDirection:'row',
        flex:1,
        alignItems:'center',
        justifyContent:'center',
        marginTop:3,
    },
    item:{
        flex:1,
        alignItems:'center',
        justifyContent:'center',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/SupermarketSearchListGroup.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const { PageList } = COMPONENTS;
let km = '';

module.exports = React.createClass({
    renderRow (obj, rowID) {
        if (obj.distance) {
            if (obj.distance > 1) {
                km = parseInt(obj.distance) + 'km';
            } else {
                km = parseInt((obj.distance) * 1000) + 'm';
            }
        }
        return (
            <View style={styles.row}>
                <View style={styles.rowChild}>
                    <Text style={styles.shopName} numberOfLines={1}>{obj.name}</Text>
                    <View style={styles.price}>
                        <Text style={styles.rowChildFont} numberOfLines={1}>总运费:</Text>
                        <Text style={styles.rowChildFont} numberOfLines={1}>{app.utils.N(parseInt(obj.totalTransportFee)) + '元'}</Text>
                    </View>
                    <Text style={styles.rowChildDistance}>距我:{km}</Text>
                </View>
            </View>
        );
    },
    render () {
        const { index, tabIndex, mapInfo, orderInfo, listName, data } = this.props;
        return (
            <View style={index === tabIndex ? styles.container : { left:-sr.tw, top:0, position:'absolute' }}>
                {
                    !!data &&
                    <PageList
                        ref={(ref) => { this.listView = ref; }}
                        renderRow={this.renderRow}
                        listParam={{ userId: app.personal.info.userId, orderBy: index, ...mapInfo, ...orderInfo }}
                        listName={listName}
                        list={data}
                        disable
                        autoLoad={false}
                        listUrl={app.route.ROUTE_GET_ROADMAP_LIST_WITH_PRE_ORDER_GROUP}
                        refreshEnable
                        />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
    },
    row: {
        flex:1,
        paddingVertical: 20,
        paddingHorizontal: 2,
        backgroundColor:'#FFFFFF',
        marginTop:1,
    },
    rowChild:{
        width:sr.w,
        flexDirection:'row',
        marginTop:3,
        marginBottom:3,
    },
    shopName:{
        flex:2,
        fontSize:14,
        marginLeft:10,
        fontWeight:'300',
    },
    rowChildFont:{
        fontSize:13,
        marginLeft:5,
        color:'#c81622',
        fontWeight:'300',
    },
    rowChildDistance:{
        fontSize:12,
        color:'#FF9D24',
        fontWeight:'300',
        marginRight:15,
        flex:1,
        textAlign:'right',
    },
    price:{
        flexDirection:'row',
        flex:1,
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/SupermarketSearchResult.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

const SupermarketSearchList = require('./SupermarketSearchList');
module.exports = React.createClass({
    statics: {
        title: '查询结果',
    },
    getInitialState () {
        return {
            tabIndex: 0,
        };
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    render () {
        const menus = ['价格最低', '时间最短', '亲签率最高', '距离最短'];
        const { tabIndex } = this.state;
        const { mapInfo, orderInfo, listName, data } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.tabContainer}>
                    {
                        menus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={this.changeTab.bind(null, i)}
                                style={styles.touchTab}>
                                <View style={styles.tabButton}>
                                    <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                        {item}
                                    </Text>
                                    <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                </View>
                            </TouchableHighlight>
                        ))
                    }
                </View>
                {
                    menus.map((item, i) => (
                        <SupermarketSearchList key={i} index={i} tabIndex={tabIndex} mapInfo={mapInfo} orderInfo={orderInfo} listName={'shop.' + listName} data={data} />
                    ))
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonCenter: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonRight: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        borderRadius: 9,
    },
    tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/SupermarketSearchResultGroup.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

const SupermarketSearchListGroup = require('./SupermarketSearchListGroup');
module.exports = React.createClass({
    statics: {
        title: '查询结果',
    },
    getInitialState () {
        return {
            tabIndex: 0,
        };
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    render () {
        const menus = ['价格最低', '距离最短'];
        const { tabIndex } = this.state;
        const { mapInfo, orderInfo, listName, data } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.tabContainer}>
                    {
                        menus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={this.changeTab.bind(null, i)}
                                style={styles.touchTab}>
                                <View style={styles.tabButton}>
                                    <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                        {item}
                                    </Text>
                                    <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                </View>
                            </TouchableHighlight>
                        ))
                    }
                </View>
                {
                    menus.map((item, i) => (
                        <SupermarketSearchListGroup key={i} index={i} tabIndex={tabIndex} mapInfo={mapInfo} orderInfo={orderInfo} listName={listName} data={data} />
                    ))
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonCenter: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonRight: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        borderRadius: 9,
    },
    tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```

* PDClient/project/App/modules/receivePoint/preOrder/index.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    TouchableHighlight,
} = ReactNative;

const PreOrderGroupList = require('./PreOrderGroupList');
const PreOrderList = require('./PreOrderList');
const TimerMixin = require('react-timer-mixin');

module.exports = React.createClass({
    mixins: [TimerMixin],
    getInitialState () {
        return {
            tabIndex: 0,
        };
    },
    onWillFocus () {
        this.setTimeout(() => {
            app.getCurrentRoute().title = '发货';
            app.getCurrentRoute().rightButton = null;
            app.getCurrentRoute().leftButton = null;
            app.forceUpdateNavbar();
        }, 200);
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    render () {
        const menus = ['单一货单', '货单组'];
        const { tabIndex } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.tabContainer}>
                    {
                        menus.map((item, i) => (
                            <TouchableHighlight
                                key={i}
                                underlayColor='rgba(0, 0, 0, 0)'
                                onPress={this.changeTab.bind(null, i)}
                                style={styles.touchTab}>
                                <View style={styles.tabButton}>
                                    <Text style={[styles.tabText, tabIndex === i ? { color:'#DE3031' } : { color:'#666666' }]} >
                                        {item}
                                    </Text>
                                    <View style={[styles.tabLine, tabIndex === i ? { backgroundColor: '#DE3031' } : null]} />
                                </View>
                            </TouchableHighlight>
                        ))
                    }
                </View>
                {
                    tabIndex == 0 ? <PreOrderList /> : <PreOrderGroupList />
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    tabContainer: {
        width:sr.w,
        flexDirection: 'row',
    },
    touchTab: {
        flex: 1,
        paddingTop: 20,
    },
    tabButton: {
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonCenter: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
    },
    tabButtonRight: {
        flex: 1,
        alignItems:'center',
        justifyContent:'center',
        borderRadius: 9,
    },
    tabText: {
        fontSize: 13,
    },
    tabLine: {
        width: sr.w / 4,
        height: 2,
        marginTop: 10,
        alignSelf: 'center',
    },
});

```
