* PDClient/project/App/index.js

```js
'use strict';

console.disableYellowBox = true;

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    Platform,
    BackAndroid,
    View,
    Text,
    Image,
    StatusBar,
} = ReactNative;

const CustomComponents = require('react-native-deprecated-custom-components');
global.Navigator = CustomComponents.Navigator;
global._ = require('lodash');
global.sr = require('./config/Screen');
const COLOR = require('./config/Color');
global._c = COLOR._c;
global._bc = COLOR._bc;
global._tc = COLOR._tc;
global.Toast = require('@remobile/react-native-toast').show;
global.CONSTANTS = require('./config/Constants');
global.AH = require('./config/Authority');
global.PS = require('./config/Position');
global.PT = require('./config/Partment');
global.OS = require('./config/OrderState');
global.TS = require('./config/TruckState');
const Utils = require('./utils');
global.POST = Utils.POST;
global.GET = Utils.GET;
global.UPLOAD = Utils.UPLOAD;
global.COMPONENTS = require('./components/index');
const TimerMixin = require('react-timer-mixin');
const Route = require('./config/Route');
const img = require('./resource/image');
const aud = require('./resource/audio');
const AuthorityMgr = require('./manager/AuthorityMgr');
const PersonalInfoMgr = require('./manager/PersonalInfoMgr');
const UpdateMgr = require('./manager/UpdateMgr');
const SettingMgr = require('./manager/SettingMgr');
const LoginMgr = require('./manager/LoginMgr');
const WXpayMgr = require('./manager/WXpayMgr');
const UPpayMgr = require('./manager/UPpayMgr');
const JpushMgr = require('./manager/JpushMgr');
const { ProgressHud, DelayTouchableOpacity, Modal } = COMPONENTS;

global.app = {
    route: Route,
    utils: Utils,
    img: img,
    aud: aud,
    data: {},
    authority: AuthorityMgr,
    personal: PersonalInfoMgr,
    setting: SettingMgr,
    updateMgr:UpdateMgr,
    login: LoginMgr,
    wxpay: WXpayMgr,
    uppay: UPpayMgr,
    jpush: JpushMgr,
    isandroid: Platform.OS === 'android',
};

global.SceneMixin = {
    componentWillMount () {
        app.scene = this;
    },
    onWillFocus () {
        app.scene = this;
    },
};

app.configureScene = function (route) {
    route = route || {};
    let sceneConfig = route.sceneConfig;
    if (sceneConfig) {
        return sceneConfig;
    }
    if (Platform.OS === 'android') {
        if (route.fromLeft) {
            sceneConfig = { ...Navigator.SceneConfigs.FloatFromLeft, gestures: null };
        } else {
            sceneConfig = Navigator.SceneConfigs.FadeAndroid;
        }
    } else {
        if (route.fromLeft) {
            sceneConfig = { ...Navigator.SceneConfigs.FloatFromLeft, gestures: null };
        } else {
            sceneConfig = { ...Navigator.SceneConfigs.HorizontalSwipeJump, gestures: null };
        }
    }
    return sceneConfig;
};

const Splash = require('./modules/splash/index');

const NavigationBarRouteMapper = {
    LeftButton (route, navigator, index, navState) {
        let leftButton = route.leftButton || route.component.leftButton;
        if (index === 0 && !leftButton) {
            return null;
        }
        if (typeof leftButton === 'function') {
            leftButton = leftButton();
        }
        const image = leftButton && leftButton.image || app.img.common_back;
        const title = leftButton && leftButton.title || '';
        const isLimitLength = leftButton && leftButton.isLimitLength || false;
        const handler = leftButton && leftButton.handler || navigator.pop;
        if (image && !title) {
            return (
                <DelayTouchableOpacity
                    onPress={handler}
                    style={styles.navBarButton}>
                    <Image
                        resizeMode='stretch'
                        source={image}
                        style={styles.navBarIcon} />
                </DelayTouchableOpacity>
            );
        } else {
            return (
                <DelayTouchableOpacity
                    onPress={handler}
                    style={styles.navBarButton}>
                    {
                        isLimitLength ? <Text numberOfLines={1} style={styles.navBarButtonLimitLengthText}>{leftButton.title}</Text>
                        : <Text style={styles.navBarButtonText}>{leftButton.title}</Text>
                    }

                </DelayTouchableOpacity>
            );
        }
    },
    RightButton (route, navigator, index, navState) {
        let rightButton = route.rightButton || route.component.rightButton;
        if (typeof rightButton === 'function') {
            rightButton = rightButton();
        }
        if (!rightButton) {
            return <View style={styles.navBarRightEmptyButton} />;
        }
        if (rightButton.image) {
            return (
                <DelayTouchableOpacity
                    onPress={rightButton.handler}
                    style={styles.navBarButton}>
                    <Image
                        resizeMode='stretch'
                        source={rightButton.image}
                        style={styles.navBarIcon} />
                </DelayTouchableOpacity>
            );
        } else {
            return (
                <DelayTouchableOpacity
                    onPress={rightButton.handler}
                    style={styles.navBarButton}>
                    <Text style={styles.navBarButtonText}>
                        {rightButton.title}
                    </Text>
                </DelayTouchableOpacity>
            );
        }
    },
    Title (route, navigator, index, navState) {
        const title = route.title || route.component.title;
        if (typeof title === 'function') {
            return (
                <View style={styles.titleContainer}>
                    <Text
                        numberOfLines={1}
                        style={styles.navBarTitleText}>
                        {title()}
                    </Text>
                </View>
            );
        } else if (typeof title === 'string') {
            return (
                <View style={styles.titleContainer}>
                    <Text
                        numberOfLines={1}
                        style={styles.navBarTitleText}>
                        {title}
                    </Text>
                </View>
            );
        } else {
            return (
                <View style={styles.titleContainer}>
                    {title}
                </View>
            );
        }
    },
};

module.exports = React.createClass({
    mixins: [TimerMixin],
    getInitialState () {
        return {
            showNavBar: false,
            modalShow: false,
            modalContent: null,
            modalBackgroundColor: null,
            modalTouchHide: false,
        };
    },
    componentWillMount () {
        // if (!app.isandroid) {
        //     NativeModules.AccessibilityManager.setAccessibilityContentSizeMultipliers({
        //         'extraSmall': 1,
        //         'small': 1,
        //         'medium': 1,
        //         'large': 1,
        //         'extraLarge': 1,
        //         'extraExtraLarge': 1,
        //         'extraExtraExtraLarge': 1,
        //         'accessibilityMedium': 1,
        //         'accessibilityLarge': 1,
        //         'accessibilityExtraLarge': 1,
        //         'accessibilityExtraExtraLarge': 1,
        //         'accessibilityExtraExtraExtraLarge': 1,
        //     });
        // }
        app.root = this;
        app.showLoading = () => {
            this.refs.progressHud.show();
        };
        app.hideLoading = () => {
            this.refs.progressHud.hide();
        };
        StatusBar.setBackgroundColor('transparent');
        StatusBar.setTranslucent(true);
        app.showModal = (view, options = {}) => {
            const { backgroundColor, touchHide } = options;
            this.setState({
                modalShow: true,
                modalContent: view,
                modalBackgroundColor: backgroundColor,
                modalTouchHide: touchHide,
            });
        };
        app.closeModal = () => {
            this.refs.modal.closeModal();
        };
        app.removeModal = () => {
            this.setState({
                modalShow: false,
            });
        };
        app.update = () => {
            this.setState({});
        };
        app.forceUpdateNavbar = () => {
            this.setState({
                showNavBar: true,
            });
        };
        app.toggleNavigationBar = (show) => {
            this.setImmediate(() => {
                this.setState({ showNavBar:show });
            });
        };
        app.getCurrentRoute = (index = 0) => {
            const { routeStack, presentedIndex } = app.navigator.state;
            return routeStack[presentedIndex - index];
        };
        app.push = (params) => {
            app.navigator.push(params);
        };
        app.pop = (step = 1) => {
            if (step === 1) {
                app.navigator.pop();
            } else {
                const routes = app.navigator.getCurrentRoutes();
                const index = routes.length - step - 1;
                if (index > 0) {
                    app.navigator.popToRoute(routes[index]);
                } else {
                    app.navigator.popToTop();
                }
            }
        };
        if (app.isandroid) {
            BackAndroid.addEventListener('hardwareBackPress', () => {
                if (this.refs.progressHud.visible) {
                    app.hideLoading();
                    return true;
                }
                if (this.state.modalShow) {
                    this.setState({ modalShow: false });
                    return true;
                }
                const routes = app.navigator.getCurrentRoutes();
                if (routes.length > 1) {
                    const leftButton = routes[routes.length - 1].component.leftButton;
                    if (leftButton && leftButton.handler) {
                        leftButton.handler();
                    } else {
                        app.pop();
                    }
                    return true;
                }
                if (!this.willExitAndroid) {
                    Toast('再按一次返回键退出程序');
                    this.willExitAndroid = true;
                    this.setTimeout(() => { this.willExitAndroid = false; }, 3000);
                    return true;
                }
                return false;
            });
        }
    },
    componentDidMount: function () {
        // app.net.register();
        app.jpush.register();
    },
    configureScene (route) {
        return app.configureScene(route);
    },
    renderScene (route, navigator) {
        return (
            <View style={{ flex: 1 }}>
                {this.state.showNavBar && <View style={[styles.navBarBack, { backgroundColor: app.setting.THEME_COLOR }]} />}
                <route.component
                    {...route.passProps}
                    ref={(ref) => { if (ref)route.ref = ref; }} />
            </View>
        );
    },
    render () {
        const initialRoute = {
            component: Splash,
        };
        const navigationBar = (
            <Navigator.NavigationBar
                routeMapper={NavigationBarRouteMapper}
                style={[styles.navBar, { backgroundColor: app.setting.THEME_COLOR }]}
                />
        );
        return (
            <View style={{ flex:1 }}>
                <Navigator
                    ref={(navigator) => {
                        if (navigator) {
                            app.navigator = navigator;
                        }
                    }}
                    debugOverlay={false}
                    style={styles.container}
                    initialRoute={initialRoute}
                    configureScene={this.configureScene}
                    renderScene={this.renderScene}
                    onDidFocus={(route) => {
                        if (route) {
                            const ref = route.ref;
                            const getChildScene = ref && ref.getChildScene;
                            // 注意：app.scene调用的时候一定需要使用封装函数，如：{handler: ()=>{app.scene.toggleEdit()}}，不能直接使用 handler: app.scene.toggleEdit.
                            // 在动画加载完成前 app.scene 还没有被赋值， 需要使用 SceneMixin 来设置 app.scene
                            const scene = app.scene = getChildScene ? getChildScene() : ref;
                            if (getChildScene && !scene.hasMouted) {
                                scene.hasMouted = true;
                                return;
                            }
                            scene && scene.onDidFocus && scene.onDidFocus();
                            // 如果时主页面，需要检测主页面和其子页面的回调
                            ref && ref !== scene && ref.onDidFocus && ref.onDidFocus();
                        }
                    }}
                    onWillFocus={(route) => {
                        if (route) {
                            const preRoute = app.navigator && app.getCurrentRoute();
                            if (preRoute) {
                                const preRef = preRoute.ref;
                                const preGetChildScene = preRef && preRef.getChildScene;
                                const preScene = preGetChildScene ? preGetChildScene() : preRef;
                                preScene && preScene.onWillHide && preScene.onWillHide();
                                // 如果时主页面，需要检测主页面和其子页面的回调
                                preRef && preRef !== preScene && preRef.onWillHide && preRef.onWillHide();
                            }
                            const ref = route.ref;
                            const getChildScene = ref && ref.getChildScene;
                            const scene = getChildScene ? getChildScene() : ref;
                            // 如果时主页面，需要检测主页面和其子页面的回调
                            scene && scene.onWillFocus && scene.onWillFocus();// 注意：在首次加载的时候页面没有被加载，route.ref为空，不会调用该函数，需要在该页面的componentWillMount里面处理首次逻辑，只有从上页面返回的时候才能被调用
                            ref && ref !== scene && ref.onWillFocus && ref.onWillFocus();
                        }
                    }}
                    navigationBar={this.state.showNavBar ? navigationBar : null}
                    />
                {
                    this.state.modalShow &&
                    <Modal ref='modal' backgroundColor={this.state.modalBackgroundColor} modalTouchHide={this.state.modalTouchHide}>
                        {this.state.modalContent}
                    </Modal>
                }
                <ProgressHud
                    ref='progressHud'
                    overlayColor='rgba(0, 0, 0, 0.6)'
                    color={app.setting.THEME_COLOR}
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex:1,
        backgroundColor:'#EEEEEE',
    },
    navBarBack: {
        height:sr.totalNavbarHeight,
    },
    navBar: {
        alignItems:'center',
        height: sr.totalNavbarHeight,
    },
    titleContainer: {
        width: sr.w,
        height: sr.navbarHeight,
        alignItems:'center',
        justifyContent: 'center',
        paddingTop: sr.translucent ? sr.statusBarHeight : 0,
    },
    navBarButtonLimitLengthText: {
        color: '#FFFFFF',
        fontSize: 14,
        width:80,
    },
    navBarButtonText: {
        color: '#FFFFFF',
        fontSize: 16,
    },
    navBarTitleText: {
        fontSize: 18,
        color: '#FFFFFF',
        textAlign: 'center',
        fontWeight: '500',
        width: sr.w / 2,
    },
    navBarButton: {
        flexDirection: 'row',
        paddingHorizontal: 10,
        height:sr.navbarHeight,
        alignItems: 'center',
        paddingTop: sr.translucent ? sr.statusBarHeight / 2 : 0,
    },
    navBarRightEmptyButton: {
        width: 80,
        height: sr.navbarHeight,
    },
    navBarIcon: {
        width: sr.navbarHeight * (sr.translucent ? 0.2 : 0.4),
        height: sr.navbarHeight * (sr.translucent ? 0.2 : 0.4),
    },
});

```

* PDClient/project/App/components/AutogrowInput.js

```js
import React, { Component } from 'react';
import {
    TextInput,
    StyleSheet,
} from 'react-native';

class Input extends Component {
    constructor () {
        super();
        this.state = {
            height: 35,
        };
    }
    componentWillMount () {
        const { maxLines, style, maxHeight } = this.props;
        const { height, lineHeight, fontSize } = StyleSheet.flatten(style);
        this.defaultHeight = height;
        this.maxHeight = maxLines ? (maxLines * (lineHeight || fontSize * 1.5)) : maxHeight || 99999;
        if (this.defaultHeight) {
            this.setState({ height:this.defaultHeight });
        }
    }
    handleChange (event) {
        const { onChange } = this.props;
        const { height } = event.nativeEvent.contentSize;

        if (this.state.height !== height && height < this.maxHeight) {
            this.setState({
                height: Math.min(Math.max(this.defaultHeight, height), this.maxHeight),
            });
        }
        onChange && onChange(event);
    }
    resetInputText () {
        this.refs.input.setNativeProps({ text: '' });
        this.setState({ height: this.defaultHeight });
    }
    render () {
        return (
            <TextInput
                ref='input'
                multiline
                {...this.props}
                style={[this.props.style, { height: this.state.height }]}
                onChange={this.handleChange.bind(this)}
                />
        );
    }
}

Input.propTypes = {
    style: React.PropTypes.oneOfType([
        React.PropTypes.number,
        React.PropTypes.array,
        React.PropTypes.object,
    ]),
    onChange: React.PropTypes.func,
};

module.exports = Input;

```

* PDClient/project/App/components/Button.js

```js
'use strict';

const React = require('react');const { PropTypes } = React;const ReactNative = require('react-native');
const {
    StyleSheet,
    Text,
    TouchableOpacity,
} = ReactNative;

const DEFAULT_OPACITY = 0.8;

module.exports = React.createClass({
    propTypes: {
        onPress: PropTypes.func,
        disable: PropTypes.bool,
        textStyle: PropTypes.style,
        activeOpacity: PropTypes.number,
    },
    render () {
        const touchableProps = {
            activeOpacity: this.props.disable ? 1 : this.props.activeOpacity ? this.props.activeOpacity : DEFAULT_OPACITY,
        };
        if (!this.props.disable) {
            touchableProps.onPress = this.props.onPress;
        }
        return (
            <TouchableOpacity
                style={[styles.container, { backgroundColor:app.setting.THEME_COLOR }, this.props.style]} {...touchableProps}
                testID={this.props.testID}>
                <Text style={[styles.text, this.props.disable ? styles.disableText : null, this.props.textStyle]}
                    numberOfLines={1}>
                    {this.props.children}
                </Text>
            </TouchableOpacity>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        borderRadius: 10,
        alignItems:'center',
        justifyContent:'center',
    },
    text: {
        color: '#FFFFFF',
        fontSize: 18,
        fontWeight: '100',
        textAlign: 'center',
        overflow: 'hidden',
    },
    disableText: {
        color: '#dcdcdc',
    },
});

```

* PDClient/project/App/components/ClipRect.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    Shape,
    Surface,
    Path,
} = ReactNative.ART;

const ClipRectIOS = React.createClass({
    render: function () {
        const style = ReactNative.StyleSheet.flatten(this.props.style);
        let { width, height, borderRadius, borderTopLeftRadius, borderTopRightRadius, borderBottomRightRadius, borderBottomLeftRadius, color } = style;
        borderRadius = borderRadius || 0;
        const tl = borderTopLeftRadius || borderRadius;
        const tr = borderTopRightRadius || borderRadius;
        const br = borderBottomRightRadius || borderRadius;
        const bl = borderBottomLeftRadius || borderRadius;

        const path = Path();

        path.move(0, tl);

        if (tl > 0) { path.arc(tl, -tl); }
        path.line(width - (tr + tl), 0);

        if (tr > 0) { path.arc(tr, tr); }
        path.line(0, height - (tr + br));

        if (br > 0) { path.arc(-br, br); }
        path.line(-width + (br + bl), 0);

        if (bl > 0) { path.arc(-bl, -bl); }
        path.line(0, bl)
        .line(width, 0)
        .line(0, -height)
        .line(-width, 0);

        return (
            <Surface width={width} height={height} style={{ backgroundColor:'transparent' }}>
                <Shape d={path} fill={color} />
            </Surface>
        );
    },
});

const TIMES = 30;
const _X = (r, d) => Math.cos(Math.PI / 180 * d) * r;
const _Y = (r, d) => Math.sin(Math.PI / 180 * d) * r;
const arc = (path, x, y, r, t) => {
    const offset = 90 / TIMES;
    for (let i = 0; i <= TIMES; i++) {
        path.lineTo(x - _X(r, 90 * t + offset * i), y - _Y(r, 90 * t + offset * i));
    }
};

const ClipRectAndroid = React.createClass({
    render: function () {
        const style = ReactNative.StyleSheet.flatten(this.props.style);
        let { width, height, borderRadius, borderTopLeftRadius, borderTopRightRadius, borderBottomRightRadius, borderBottomLeftRadius, color } = style;
        borderRadius = borderRadius || 0;
        const tl = borderTopLeftRadius || borderRadius;
        const tr = borderTopRightRadius || borderRadius;
        const br = borderBottomRightRadius || borderRadius;
        const bl = borderBottomLeftRadius || borderRadius;

        const path = Path();

        path.move(0, tl);

        if (tl > 0) { arc(path, tl, tl, tl, 0); }
        path.line(width - (tr + tl), 0);

        if (tr > 0) { arc(path, width - tr, tr, tr, 1); }
        path.line(0, height - (tr + br));

        if (br > 0) { arc(path, width - br, height - br, br, 2); }
        path.line(-width + (br + bl), 0);

        if (bl > 0) { arc(path, bl, height - bl, bl, 3); }
        path.line(0, bl)
        .line(width, 0)
        .line(0, -height)
        .line(-width, 0);

        return (
            <Surface width={width} height={height} style={{ backgroundColor:'transparent' }}>
                <Shape d={path} fill={color} />
            </Surface>
        );
    },
});

module.exports = ReactNative.Platform.OS === 'android' ? ClipRectAndroid : ClipRectIOS;

```

* PDClient/project/App/components/CustomMessageBox.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    Text,
    View,
    TouchableHighlight,
} = ReactNative;

module.exports = React.createClass({
    getDefaultProps: function () {
        return {
            title: '温馨提示',
            cancelText: '取消',
            confirmText: '确定',
            width: 312,
        };
    },
    doConfirm () {
        const { onConfirm } = this.props;
        if (!onConfirm || !onConfirm()) {
            app.closeModal();
        }
    },
    doCancel () {
        const { onCancel } = this.props;
        if (onCancel === true || !onCancel || !onCancel()) {
            app.closeModal();
        }
    },
    render () {
        const { title, width, onCancel, cancelText, confirmText } = this.props;
        return (
            <View style={[styles.container, { width }]}>
                { !!title && <Text style={styles.title}>{title}</Text> }
                { !!title && <Text style={[styles.redLine, { width: width * 0.8 }]} /> }
                <View style={styles.content}>
                    {this.props.children}
                </View>
                <Text style={[styles.H_Line, { width }]} />
                <View style={[styles.buttonViewStyle, { width: width - 20 }]}>
                    {!!onCancel &&
                    <TouchableHighlight
                        underlayColor='rgba(0, 0, 0, 0)'
                        onPress={this.doCancel}
                        style={styles.buttonStyleContain}>
                        <Text style={[styles.buttonStyle, { color: 'red' }]}>{cancelText}</Text>
                    </TouchableHighlight>
                        }
                    {!!onCancel &&
                    <Text style={styles.line} />
                        }
                    <TouchableHighlight
                        underlayColor='rgba(0, 0, 0, 0)'
                        onPress={this.doConfirm}
                        style={styles.buttonStyleContain}>
                        <Text style={[styles.buttonStyle, { color: '#0076FF' }]} >{confirmText}</Text>
                    </TouchableHighlight>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        alignItems:'center',
        justifyContent:'center',
        backgroundColor:'#FFFFFF',
        borderRadius:10,
    },
    buttonViewStyle: {
        flexDirection: 'row',
        height: 50,
    },
    H_Line: {
        marginTop: 10,
        height: 1,
        backgroundColor: '#b4b4b4',
    },
    redLine: {
        marginTop: 10,
        height: 1,
        backgroundColor: '#ff3c30',
    },
    line: {
        width: 1,
        height: 50,
        backgroundColor: '#b4b4b4',
    },
    buttonStyleContain: {
        height: 50,
        flex: 1,
        justifyContent:'center',
        alignItems:'center',
    },
    buttonStyle: {
        fontSize: 15,
        color: '#000000',
    },
    title: {
        color: '#ff3c30',
        fontSize: 16,
        fontWeight: '100',
        textAlign: 'center',
        overflow: 'hidden',
        marginTop:20,
    },
    content: {
        alignSelf:'center',
        color:'#000000',
        marginTop: 20,
        width:sr.w/2,
    },
});

```

* PDClient/project/App/components/CustomSheet.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    TouchableOpacity,
    View,
} = ReactNative;

const Overlay = require('./ActionSheet/overlay');
const Sheet = require('./ActionSheet/sheet');

module.exports = React.createClass({
    render () {
        const { visible, onCancel } = this.props;
        return (
            <Overlay visible={visible}>
                <View style={styles.actionSheetContainer}>
                    <TouchableOpacity
                        style={{ flex:1 }}
                        onPress={onCancel} />
                    <Sheet visible={visible}>
                        {this.props.children}
                    </Sheet>
                </View>
            </Overlay>
        );
    },
});

const styles = StyleSheet.create({
    actionSheetContainer: {
        flex: 1,
        backgroundColor: 'rgba(0, 0, 0, 0.5)',
    },
});

```

* PDClient/project/App/components/DImage.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    Image,
    View,
} = ReactNative;

const DImage = React.createClass({
    getInitialState () {
        return { showDefault: true };
    },
    onLoad () {
        this.setState({ showDefault: false });
    },
    componentWillReceiveProps (nextProps) {
        const pre = this.props.source, post = nextProps.source;
        const preuri = pre.uri, posturi = post.uri;
        const preT = typeof posturi === 'string', postT = typeof posturi === 'string';
        if ((!(preT ^ postT)) && preuri !== posturi) {
            this.setState({ showDefault: true });
        }
    },
    render () {
        const { source, defaultSource, ...other } = this.props;
        const { showDefault } = this.state;
        return (
            showDefault ?
                <View>
                    <Image source={defaultSource} {...other}>{this.props.children}</Image>
                    <Image style={{ left:-1, top:-1, position:'absolute', width:1, height:1 }} source={source} onLoad={this.onLoad} />
                </View>
            :
                <Image source={source} {...other} >{this.props.children}</Image>
        );
    },
});

module.exports = DImage;

```

* PDClient/project/App/components/DelayTouchableOpacity.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    TouchableOpacity,
} = ReactNative;

const TimerMixin = require('react-timer-mixin');

module.exports = React.createClass({
    mixins: [TimerMixin],
    componentWillMount () {
        this.enable = true;
        this.onPress = (e) => {
            if (this.enable) {
                this.enable = false;
                this.props.onPress(e);
                this.setTimeout(() => { this.enable = true; }, this.props.delayTime || 200);
            }
        };
    },
    render () {
        return (
            <TouchableOpacity
                {...this.props}
                onPress={this.onPress}
                >
                {this.props.children}
            </TouchableOpacity>
        );
    },
});

```

* PDClient/project/App/components/Label.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    Image,
} = ReactNative;

module.exports = React.createClass({
    render () {
        const { img, children, style, textStyle } = this.props;
        return (
            <View style={[styles.labelContainer, style]}>
                {!!img && <Image resizeMode='stretch' source={img} style={styles.labelIcon} />}
                <Text style={[styles.label, textStyle]}>{children}:</Text>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    labelContainer: {
        marginTop: 10,
        marginBottom: 6,
        flexDirection: 'row',
        overflow: 'hidden',
        alignItems:'center',
    },
    label: {
        color: '#969696',
        fontSize: 14,
    },
    labelIcon: {
        width: 20,
        height: 20,
        tintColor: '#A3A3A3',
    },
});

```

* PDClient/project/App/components/MessageBox.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    Text,
    View,
    TouchableHighlight,
} = ReactNative;

module.exports = React.createClass({
    getDefaultProps: function () {
        return {
            title: '温馨提示',
            content: '确定要执行操作吗？',
            cancelText: '取消',
            confirmText: '确定',
            width: 312,
        };
    },
    doConfirm () {
        const { onConfirm } = this.props;
        if (!onConfirm || !onConfirm()) {
            app.closeModal();
        }
    },
    doCancel () {
        const { onCancel } = this.props;
        if (onCancel === true || !onCancel || !onCancel()) {
            app.closeModal();
        }
    },
    render () {
        const { title, width, content, onCancel, cancelText, confirmText } = this.props;
        return (
            <View style={[styles.container, { width }]}>
                { !!title && <Text style={styles.title}>{title}</Text> }
                { !!title && <Text style={[styles.redLine, { width: width * 0.8 }]} /> }
                <Text style={styles.content}>
                    {content}
                </Text>
                <Text style={[styles.H_Line, { width }]} />
                <View style={[styles.buttonViewStyle, { width: width - 20 }]}>
                    {!!onCancel &&
                    <TouchableHighlight
                        underlayColor='rgba(0, 0, 0, 0)'
                        onPress={this.doCancel}
                        style={styles.buttonStyleContain}>
                        <Text style={[styles.buttonStyle, { color: 'red' }]}>{cancelText}</Text>
                    </TouchableHighlight>
                        }
                    {!!onCancel &&
                    <Text style={styles.line} />
                        }
                    <TouchableHighlight
                        underlayColor='rgba(0, 0, 0, 0)'
                        onPress={this.doConfirm}
                        style={styles.buttonStyleContain}>
                        <Text style={[styles.buttonStyle, { color: '#0076FF' }]} >{confirmText}</Text>
                    </TouchableHighlight>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        alignItems:'center',
        justifyContent:'center',
        backgroundColor:'#FFFFFF',
        borderRadius:10,
    },
    buttonViewStyle: {
        flexDirection: 'row',
        height: 50,
    },
    H_Line: {
        marginTop: 10,
        height: 1,
        backgroundColor: '#b4b4b4',
    },
    redLine: {
        marginTop: 10,
        height: 1,
        backgroundColor: '#ff3c30',
    },
    line: {
        width: 1,
        height: 50,
        backgroundColor: '#b4b4b4',
    },
    buttonStyleContain: {
        height: 50,
        flex: 1,
        justifyContent:'center',
        alignItems:'center',
    },
    buttonStyle: {
        fontSize: 15,
        color: '#000000',
    },
    title: {
        color: '#ff3c30',
        fontSize: 16,
        fontWeight: '100',
        textAlign: 'center',
        overflow: 'hidden',
        marginTop:20,
    },
    content: {
        alignSelf:'center',
        color:'#000000',
        margin: 20,
        marginTop: 30,
    },
});

```

* PDClient/project/App/components/Modal.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    Animated,
    PanResponder,
} = ReactNative;

module.exports = React.createClass({
    getDefaultProps () {
        return {
            backgroundColor: 'rgba(0, 0, 0, 0.5)',
        };
    },
    getInitialState () {
        return {
            opacity: new Animated.Value(0),
        };
    },
    componentWillMount () {
        this._panResponder = PanResponder.create({
            onStartShouldSetPanResponder: (e, gestureState) => true,
            onPanResponderGrant: (e, gestureState) => {
                this.closeModal();
            },
        });
    },
    componentDidMount () {
        Animated.timing(this.state.opacity, {
            toValue: 1,
            duration: 500,
        }
        ).start();
    },
    closeModal () {
        Animated.timing(this.state.opacity, {
            toValue: 0,
            duration: 500,
        }
        ).start(() => {
            app.removeModal();
        });
    },
    render () {
        const { modalTouchHide } = this.props;
        return (
            <Animated.View style={[styles.container, { backgroundColor: this.props.backgroundColor, opacity: this.state.opacity }]} {...(modalTouchHide ? this._panResponder.panHandlers : {})}>
                {this.props.children}
            </Animated.View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        position: 'absolute',
        top: 0,
        bottom: 0,
        left: 0,
        right: 0,
        alignItems:'center',
        justifyContent: 'center',
    },
});

```

* PDClient/project/App/components/PageList.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    View,
    Text,
    StyleSheet,
    FlatList,
} = ReactNative;

const STATUS = {
    /* loading more status change graph
    *
    * STATUS_TEXT_HIDE->[STATUS_HAVE_MORE, STATUS_START_LOAD]
    * STATUS_START_LOAD->[STATUS_TEXT_HIDE, STATUS_NO_DATA, STATUS_ALL_LOADED, STATUS_LOAD_ERROR]
    * STATUS_HAVE_MORE->[STATUS_TEXT_HIDE, STATUS_NO_DATA, STATUS_ALL_LOADED, STATUS_LOAD_ERROR]
    * STATUS_ALL_LOADED->[STATUS_TEXT_HIDE]
    */
    STATUS_TEXT_HIDE: 0,
    STATUS_START_LOAD: 1,
    STATUS_HAVE_MORE: 2,
    STATUS_NO_DATA: 3,
    STATUS_ALL_LOADED: 4,
    STATUS_LOAD_ERROR: 5,
};
const TEXT = {
    0: '',
    1: '',
    2: '正在加载更多...',
    3: '暂无数据!',
    4: '全部加载完成',
    5: '加载错误，请稍后再试',
};

const { STATUS_TEXT_HIDE, STATUS_START_LOAD, STATUS_HAVE_MORE, STATUS_NO_DATA, STATUS_ALL_LOADED, STATUS_LOAD_ERROR } = STATUS;

module.exports = React.createClass({
    getDefaultProps () {
        return {
            autoLoad: true,
            pageNo: 0,
            style: { flex: 1 },
            pageSize: CONSTANTS.PER_PAGE_COUNT,
        };
    },
    componentDidMount () {
        if (this.props.autoLoad) {
            this.getList();
        }
    },
    updateList (callback) {
        this.list = callback(this.list);
        this.setState({
            dataSource: this.list,
        });
    },
    getInitialState () {
        const { list, pageNo, autoLoad } = this.props;
        this.list = list || [];
        this.pageNo = pageNo;
        return {
            dataSource: this.list,
            infiniteLoadStatus: list ? (list.length === 0 ? STATUS_NO_DATA : list.length < CONSTANTS.PER_PAGE_COUNT ? STATUS_ALL_LOADED : STATUS_TEXT_HIDE) : !autoLoad ? STATUS_TEXT_HIDE : STATUS_START_LOAD,
        };
    },
    componentDidUpdate (prevProps, prevState) {
        if (!_.isEqual(prevProps.listParam, this.props.listParam)) {
            this.refresh();
        }
    },
    getList (wait) {
        const param = {
            ...this.props.listParam,
            pageNo: this.pageNo,
            pageSize: this.props.pageSize,
        };
        this.setState({ infiniteLoadStatus: this.pageNo === 0 ? STATUS_START_LOAD : STATUS_HAVE_MORE });
        POST(this.props.listUrl, param, this.getListSuccess, this.getListFailed, wait);
    },
    getListSuccess (data) {
        this.props.onGetList && this.props.onGetList(data, this.pageNo);
        if (data.success) {
            const list = _.get(data.context, this.props.listName);
            const infiniteLoadStatus = (!list.length && this.pageNo === 0) ? STATUS_NO_DATA : list.length < this.props.pageSize ? STATUS_ALL_LOADED : STATUS_TEXT_HIDE;
            this.list = this.list.concat(list);
            this.setState({
                dataSource: this.list,
                infiniteLoadStatus: infiniteLoadStatus,
            });
        } else {
            if (this.props.ListFailedText) {
                this.setState({ infiniteLoadStatus: this.props.ListFailedText });
            } else {
                this.getListFailed();
            }
        }
    },
    getListFailed () {
        this.pageNo--;
        this.setState({ infiniteLoadStatus: STATUS_LOAD_ERROR });
    },
    onEndReached () {
        if (this.state.infiniteLoadStatus !== STATUS_TEXT_HIDE || this.props.disable) {
            return;
        }
        this.pageNo++;
        if (this.pageNo > 0) {
            this.getList();
        }
    },
    refresh () {
        if (this.isRefreshing()) {
            return;
        }
        this.list = [];
        this.setState({
            dataSource: this.list,
        });
        this.pageNo = 0;
        this.getList();
    },
    isRefreshing () {
        return (this.state.infiniteLoadStatus === STATUS_START_LOAD || this.state.infiniteLoadStatus === STATUS_HAVE_MORE);
    },
    renderSeparator (sectionID, rowID) {
        return (
            <View
                style={styles.separator}
                key={sectionID + rowID} />
        );
    },
    renderFooter () {
        const status = this.state.infiniteLoadStatus;
        return (
            <View style={styles.listFooterContainer}>
                <Text style={styles.listFooter}>
                    {typeof status === 'string' ? status : status === STATUS_NO_DATA && this.props.ListFailedText ? this.props.ListFailedText : TEXT[status]}
                </Text>
            </View>
        );
    },
    render () {
        return (
            <FlatList
                onEndReached={this.onEndReached}
                onEndReachedThreshold={10}
                initialListSize={this.props.pageSize}
                enableEmptySections
                removeClippedSubviews={false}
                style={[{ alignSelf:'stretch' }, this.props.style]}
                data={this.state.dataSource}
                renderItem={({ item, index }) => this.props.renderRow(item, index)}
                ItemSeparatorComponent={this.props.renderSeparator === undefined ? this.renderSeparator : this.props.renderSeparator}
                ListFooterComponent={this.renderFooter}
                refreshing={this.state.infiniteLoadStatus === STATUS_START_LOAD}
                onRefresh={this.refresh}
                />
        );
    },
});
module.exports.STATUS = STATUS;

const styles = StyleSheet.create({
    listFooterContainer: {
        height: 60,
        alignItems: 'center',
        paddingTop: 10,
    },
    listFooter: {
        color: 'gray',
        fontSize: 12,
    },
    separator: {
        backgroundColor: '#DDDDDD',
        height: 1,
    },
});

```

* PDClient/project/App/components/PageListView.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    View,
    Text,
    StyleSheet,
    ListView,
    RefreshControl,
} = ReactNative;

const STATUS = {
    /* loading more status change graph
    *
    * STATUS_TEXT_HIDE->[STATUS_HAVE_MORE, STATUS_START_LOAD]
    * STATUS_START_LOAD->[STATUS_TEXT_HIDE, STATUS_NO_DATA, STATUS_ALL_LOADED, STATUS_LOAD_ERROR]
    * STATUS_HAVE_MORE->[STATUS_TEXT_HIDE, STATUS_NO_DATA, STATUS_ALL_LOADED, STATUS_LOAD_ERROR]
    * STATUS_ALL_LOADED->[STATUS_TEXT_HIDE]
    */
    STATUS_TEXT_HIDE: 0,
    STATUS_START_LOAD: 1,
    STATUS_HAVE_MORE: 2,
    STATUS_NO_DATA: 3,
    STATUS_ALL_LOADED: 4,
    STATUS_LOAD_ERROR: 5,
};
const TEXT = {
    0: '',
    1: '',
    2: '正在加载更多...',
    3: '暂无数据!',
    4: '全部加载完成',
    5: '加载错误，请稍后再试',
};

const { STATUS_TEXT_HIDE, STATUS_START_LOAD, STATUS_HAVE_MORE, STATUS_NO_DATA, STATUS_ALL_LOADED, STATUS_LOAD_ERROR } = STATUS;

module.exports = React.createClass({
    getDefaultProps () {
        return {
            autoLoad: true,
            pageNo: 0,
            infiniteLoadStatus: STATUS_START_LOAD,
            style: { flex: 1 },
            pageSize: CONSTANTS.PER_PAGE_COUNT,
        };
    },
    componentDidMount () {
        if (this.props.autoLoad) {
            this.getList();
        }
    },
    updateList (callback) {
        this.list = callback(this.list);
        this.setState({
            dataSource: this.ds.cloneWithRows(this.list),
        });
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        this.list = this.props.list || [];
        this.pageNo = this.props.pageNo;
        return {
            dataSource: this.ds.cloneWithRows(this.list),
            infiniteLoadStatus: this.props.infiniteLoadStatus,
        };
    },
    componentDidUpdate (prevProps, prevState) {
        if (!_.isEqual(prevProps.listParam, this.props.listParam)) {
            this.refresh();
        }
    },
    getList (wait) {
        const param = {
            ...this.props.listParam,
            pageNo: this.pageNo,
            pageSize: this.props.pageSize,
        };
        this.setState({ infiniteLoadStatus: this.pageNo === 0 ? STATUS_START_LOAD : STATUS_HAVE_MORE });
        POST(this.props.listUrl, param, this.getListSuccess, this.getListFailed, wait);
    },
    getListSuccess (data) {
        this.props.onGetList && this.props.onGetList(data, this.pageNo);
        if (data.success) {
            const list = _.get(data.context, this.props.listName);
            const infiniteLoadStatus = (!list.length && this.pageNo === 0) ? STATUS_NO_DATA : list.length < this.props.pageSize ? STATUS_ALL_LOADED : STATUS_TEXT_HIDE;
            this.list = this.list.concat(list);
            this.setState({
                dataSource: this.ds.cloneWithRows(this.list),
                infiniteLoadStatus: infiniteLoadStatus,
            });
        } else {
            if (this.props.ListFailedText) {
                this.setState({ infiniteLoadStatus: this.props.ListFailedText });
            } else {
                this.getListFailed();
            }
        }
    },
    getListFailed () {
        this.pageNo--;
        this.setState({ infiniteLoadStatus: STATUS_LOAD_ERROR });
    },
    onEndReached () {
        if (this.state.infiniteLoadStatus !== STATUS_TEXT_HIDE || this.props.disable) {
            return;
        }
        this.pageNo++;
        this.getList();
    },
    refresh () {
        if (this.isRefreshing()) {
            return;
        }
        this.list = [];
        this.setState({
            dataSource: this.ds.cloneWithRows(this.list),
        });
        this.pageNo = 0;
        this.getList();
    },
    isRefreshing () {
        return (this.state.infiniteLoadStatus === STATUS_START_LOAD || this.state.infiniteLoadStatus === STATUS_HAVE_MORE);
    },
    renderSeparator (sectionID, rowID) {
        return (
            <View
                style={styles.separator}
                key={sectionID + rowID} />
        );
    },
    renderFooter () {
        const status = this.state.infiniteLoadStatus;
        return (
            <View style={styles.listFooterContainer}>
                <Text style={styles.listFooter}>
                    {typeof status === 'string' ? status : status === STATUS_NO_DATA && this.props.ListFailedText ? this.props.ListFailedText : TEXT[status]}
                </Text>
            </View>
        );
    },
    render () {
        return (
            <ListView                onEndReached={this.onEndReached}
                onEndReachedThreshold={10}
                initialListSize={this.props.pageSize}
                enableEmptySections
                removeClippedSubviews={false}
                style={[{ alignSelf:'stretch' }, this.props.style]}
                dataSource={this.state.dataSource}
                renderRow={this.props.renderRow}
                renderSeparator={this.props.renderSeparator === undefined ? this.renderSeparator : this.props.renderSeparator}
                renderFooter={this.renderFooter}
                refreshControl={
                    this.props.refreshEnable ?
                        <RefreshControl
                            refreshing={this.state.infiniteLoadStatus === STATUS_START_LOAD}
                            onRefresh={this.refresh}
                            title='正在刷新...' />
                    : null
                }
                />
        );
    },
});
module.exports.STATUS = STATUS;

const styles = StyleSheet.create({
    listFooterContainer: {
        height: 60,
        alignItems: 'center',
        paddingTop: 10,
    },
    listFooter: {
        color: 'gray',
        fontSize: 12,
    },
    separator: {
        backgroundColor: '#DDDDDD',
        height: 1,
    },
});

```

* PDClient/project/App/components/Picker.js

```js
'use strict';

import Picker from 'react-native-picker';

module.exports = (pickerData, selectedValue, title) => {
    const color = Math.floor(app.setting.THEME_COLOR.replace('#', '0x'));
    const pickerToolBarBg = [(color >> 16) & 0xff, (color >> 8) & 0xff, color & 0xff, 1];
    return new Promise(async(resolve) => {
        Picker.isPickerShow((show) => {
            if (show) {
                Picker.hide();
            } else {
                Picker.init({
                    pickerConfirmBtnText: '确定',
                    pickerConfirmBtnColor: [255, 255, 255, 1],
                    pickerCancelBtnText: '取消',
                    pickerCancelBtnColor: [255, 255, 255, 1],
                    pickerTitleText: title || '',
                    pickerTitleColor: [255, 255, 255, 1],
                    pickerToolBarBg,
                    pickerBg: [255, 255, 255, 1],
                    pickerFontColor: [0, 0, 0, 1],
                    pickerToolBarFontSize: 16,
                    pickerFontSize: 16,
                    pickerData,
                    selectedValue,
                    onPickerConfirm: (value) => { resolve(value); },
                });
                Picker.show();
            }
        });
    });
};
module.exports.hide = () => {
    Picker.isPickerShow((show) => {
        Picker.hide();
    });
};

```

* PDClient/project/App/components/ProgressBar.js

```js
const React = require('react');const ReactNative = require('react-native');

const {
    StyleSheet,
    View,
} = ReactNative;

module.exports = React.createClass({
    getDefaultProps () {
        return {
            progress: 0,
        };
    },
    render () {
        const fillWidth = this.props.progress * this.props.style.width;
        return (
            <View style={[styles.background, this.props.backgroundStyle, this.props.style]}>
                <View style={[styles.fill, this.props.fillStyle, { width: fillWidth }]} />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    background: {
        backgroundColor: '#bbbbbb',
        height: 5,
        overflow: 'hidden',
    },
    fill: {
        backgroundColor: '#3b5998',
        height: 5,
    },
});

```

* PDClient/project/App/components/ProgressHud.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');

const {
    Image,
    StyleSheet,
    TouchableHighlight,
    View,
    Animated,
    Easing,
} = ReactNative;

const BACKGROUND_IMAGE = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyJpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoV2luZG93cykiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6NjBFOTY2OTRGMDlBMTFFNUFGQUVDMUUwNDA5REUwMzQiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6NjBFOTY2OTVGMDlBMTFFNUFGQUVDMUUwNDA5REUwMzQiPiA8eG1wTU06RGVyaXZlZEZyb20gc3RSZWY6aW5zdGFuY2VJRD0ieG1wLmlpZDo2MEU5NjY5MkYwOUExMUU1QUZBRUMxRTA0MDlERTAzNCIgc3RSZWY6ZG9jdW1lbnRJRD0ieG1wLmRpZDo2MEU5NjY5M0YwOUExMUU1QUZBRUMxRTA0MDlERTAzNCIvPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6UkRGPiA8L3g6eG1wbWV0YT4gPD94cGFja2V0IGVuZD0iciI/PrUpG+0AAATlSURBVHjazJpLjBVVEIa7+zYzlwGG4WF0RhkJj4EEY3yigiyU3STiYjbCRmQFC1gQEkIISzUxonHpRqOGBQnxsTImhFcIMYIhZKLyUgENyjPD8JCBud1WOX/r8aROnb7PvpV8uTOdvn3P31V1Tp3qDtM0DRpkITD/T8Fd4iZxhThHnCK+J/YTvzfkx+sUwoMtgexCqUDiOM78ROwmdhEXWi0kIiYZAnwkOc6pEF8Q7xDDzRbCHugk4pwCqhFinsve2YFQbLgQ9sBkJXy0ASc1fGeE2ER8mTdE8tgUYirOD4XEdiW8ZqmRV9LxbuJjYifRUa8QHtR0ouwZeCgc94mQBi8JW4tQm1JraGUi4irjPzGOJbjW2/BoP7GYmKt81/U73xFDxF/VCGERPciLvCLGQeK4s6b1EYPEa8Qy6zqamEPEauJ+XiHTkdh5RPBFx4y7X609Q2wnVuYU8wmxNY8QFjBDWOASYd6/I92dGu0V4n1ilifk2DYQX2lCeIF7EJOAzws36/CCy3qJT4knPd65QbxMXHLNWj1I7tBBBBGjTRDB9gfxKrHPE2LTEI7i9NuBE0KFcYhIg+YZh+vrxDFPjnIoPiYJmSHcfZMAq20zRWTGU+w6lCiJQDaGjbaQGPO8KcD2xggSvFX2J2YnV64myJNHTCHdlgDbG2OuhajJ9jXyRfPKkC3EDCk7vG4Exdm7ggBzIhg09xVlS4TpjXvwSFE2jPJE8gh/PkrMiVCMRQ5CrBdF2+fW4G2WxljJQ2OPHVoV6u02EHLIyovUqpwXRQgrl0eyQrBo4xnsvMMb/4SXKSQUZq77QfvYWSE/sr/7YqsksTc77STkglEW2YvytBizVt6dXJF2R9nrlGPPdjdqIyGpUlkkMVTGjhNKbSSkU6m4b8dQOcnRMJjcRkJ6MVZpnKMxVu4uofsR+joXLbY5hkfscf4Wo/yIHCd0waVjBYuYDY+4hJxnAbeEhbBkfM5qA288h7CqQEzFWhBPZ5VtpNDbBkJWKqs6c9zca7i8MhNlflE2QCwxBm17gxfKi1luXHaU8Nmzj4EChawxwqoilPL7zQXvkrEASnCL6KECRLxAPK2EVAW7yH+FcGhd8+TKEygwW2Uc0usFL5gcQWX8vxLkF48QXhyfb9Fqz1P+FjRENG/skmqpUTTItJYQz+fLlZKmUSI2E/OVpgNzAKW9WBSehFKto9KH6bCrCSK4t7YNjbdE2afz9vsj84tSE/th4lnBM1JT4tugjiexlnG/941g4kmA1DswuzvvEYd9QgIk9jzhIlLuXAwmnplfqVEAd0FWEY97miDZ517iQ/siLiH8pReD/zrzmphs4eQp/Azxa46GRTcWuaXEQqsk0kT8SLwl9RG0R2980ZeQ4JpHpPqM67friOVxHO/ElMolzwPGYusSYIv5mXjTVcD6Hk/z7LQCCe4TUxI+S8KAS45zNRE/EB8EE6+C1LSV5bt5ENOcnXAugkB/TJ0G+usedrOae787NRF5PGLaXKwhZcUz0h0uKcc1b3C18RlxNM/gqn2Fo4y9wUCOEIs8ArTk5l7vnmratbW+VMObraeIBcod1QbsEnCC+Cao4dWnel9z4kd1i8HsnF6yucobI4TQ9VoHEjbwxTMW1Y/plT3Wg6KvbPSR72JHeg13nafUkUb8+N8CDABdKT+Sgc4IvgAAAABJRU5ErkJggg==';

const SPIN_DURATION = 1000;

const ProgressHud = React.createClass({
    getDefaultProps () {
        return {
            color: '#000',
            overlayColor: 'rgba(0, 0, 0, 0)',
        };
    },
    getInitialState () {
        return {
            visible: false,
            rotate: new Animated.Value(0),
        };
    },
    showAnimate () {
        const { rotate, visible } = this.state;
        if (visible) {
            rotate.setValue(0);
            Animated.timing(rotate, {
                toValue: 1,
                duration: SPIN_DURATION,
                easing: Easing.linear,
                useNativeDriver: true,
            }).start(() => {
                this.showAnimate();
            });
        }
    },
    show () {
        this.setState({
            visible: true,
        }, () => {
            this.showAnimate();
        });
    },
    hide () {
        const { rotate } = this.state;
        rotate.stopAnimation(() => {
            rotate.setValue(0);
            this.setState({
                visible: false,
            });
        });
    },
    onPress () {
        if (this.props.close) {
            this.props.close();
        }
    },
    render () {
        const { visible, rotate } = this.state;
        if (!visible) {
            return <View />;
        }
        return (
            <TouchableHighlight
                key='ProgressHud'
                style={[styles.overlay, { backgroundColor: this.props.overlayColor }]}
                onPress={this.onPress}
                underlayColor={this.props.overlayColor}
                activeOpacity={1}
                >
                <View style={styles.container}>
                    <Animated.Image
                        style={[styles.spinner, {
                            transform: [{
                                rotate: rotate.interpolate({
                                    inputRange: [0, 1],
                                    outputRange: ['0deg', '360deg'],
                                }),
                            }],
                        }]}
                        source={{
                            uri: BACKGROUND_IMAGE,
                            isStatic: true,
                        }}
                         />
                    <Image
                        resizeMode='cover'
                        source={app.img.splash_logo}
                        style={styles.spinner_image} />
                </View>
            </TouchableHighlight>
        );
    },
});

const styles = StyleSheet.create({
    overlay: {
        alignItems: 'center',
        justifyContent: 'center',
        flex: 1,
        position: 'absolute',
        top: 0,
        left: 0,
        right: 0,
        bottom: 0,
    },
    container: {
        justifyContent: 'center',
        alignItems: 'center',
        width: 100,
        height: 100,
        borderRadius: 16,
    },
    spinner: {
        alignItems: 'center',
        justifyContent: 'center',
        width: 60,
        height: 60,
        borderRadius: 60 / 2,
    },
    spinner_image: {
        position: 'absolute',
        left: 26,
        top: 26,
        width: 48,
        height: 48,
        borderRadius: 48 / 2,
    },
});

module.exports = ProgressHud;

```

* PDClient/project/App/components/RText.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    Text,
    Platform,
} = ReactNative;

const RText = React.createClass({
    render () {
        const { style, ...props } = this.props;
        const s = StyleSheet.flatten(style);
        const { lineHeight, fontSize } = s;
        const paddingVertical = (lineHeight - fontSize) / 2;
        return (
            <Text
                style={[{ paddingVertical }, style]}
                testID={this.props.testID}
                {...props}
                >
                {this.props.children}
            </Text>
        );
    },
});

module.exports = Platform.OS === 'android' ? RText : Text;

```

* PDClient/project/App/components/ScoreSelect.js

```js
'use strict';
import React from 'react';
import {
    View,
    ScrollView,
    InteractionManager,
} from 'react-native';

const ViewPager = React.createClass({
    componentWillMount () {
        this.count = this.props.children.length;
    },
    componentDidMount () {
        InteractionManager.runAfterInteractions(() => {
            this.scrollView.scrollTo({ x: sr.s(160) });
        });
    },
    onScroll (e) {
        // android incompatible
        if (!e.nativeEvent.contentOffset) {
            e.nativeEvent.contentOffset = { x: e.nativeEvent.position * this.props.width };
        }
        this.updateIndex(e.nativeEvent.contentOffset.x);
    },
    updateIndex (x) {
        const { width, afterChange } = this.props;
        const selectedIndex = Math.round(x / width);
        if (this.lastSelectedIndex !== selectedIndex && selectedIndex >= 0 && selectedIndex <= this.count - 1) {
            this.lastSelectedIndex = selectedIndex;
            if (afterChange) {
                afterChange(selectedIndex);
            }
        }
    },
    render () {
        const { width, height } = this.props;
        const pages = this.props.children.map((page, i) => {
            return (<View style={{ width, height }} key={i}>{page}</View>);
        });
        return (
            <ScrollView ref={(scrollView) => { this.scrollView = scrollView; }}
                horizontal
                pagingEnabled={false}
                scrollEventThrottle={10}
                removeClippedSubviews={false}
                automaticallyAdjustContentInsets={false}
                directionalLockEnabled
                showsHorizontalScrollIndicator={false}
                showsVerticalScrollIndicator={false}
                contentContainerStyle={this.props.style}
                onScroll={this.onScroll}>
                <View style={{ width, height }} />
                {pages}
                <View style={{ width, height }} />
            </ScrollView>
        );
    },
});

module.exports = ViewPager;

```

* PDClient/project/App/components/SelectRegionAddress.js

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

// type可传 0，2，3；0: 长途地址只能选到市或县，2：所有地址可以选到镇, 3: 分店路线中存在的地址，4：收货点路线中存在的地址
module.exports = React.createClass({
    statics: {
        title: '所在地',
        rightButton: { title: '确定', handler: () => { app.scene.confirm&&app.scene.confirm(); } },
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            dataSource:[],
            allAddress:[],
        };
    },
    confirm (obj) {
        const { allAddress } = this.state;
        const lastObj = allAddress[allAddress.length - 1];
        const { confirmAddress, isNeedSelectAll } = this.props;
        if (isNeedSelectAll) {
            if (!!lastObj && lastObj.isLeaf) {
                confirmAddress(_.join(_.map(this.state.allAddress, 'name'), ''), allAddress.length >= 1 ? lastObj.code : 0);
                app.pop();
            } else {
                Toast('请选择完整到货地');
            }
        } else {
            confirmAddress(_.join(_.map(this.state.allAddress, 'name'), ''), allAddress.length >= 1 ? lastObj.code : 0);
            app.pop();
        }
    },
    componentWillMount () {
        const { endPointLastCode } = this.props;
        !endPointLastCode || endPointLastCode == '' ? this.getRegionAddress(0) : this.getAddressFromLastCode(endPointLastCode);
    },
    getRegionAddress (parentCode) {
        const type = this.props.type || 0;
        const param = { userId: app.personal.info.userId, parentCode, type };
        POST(app.route.ROUTE_GET_REGION_ADDRESS_LIST, param, this.getRegionAddressSuccess.bind(null, parentCode), false);
    },
    getRegionAddressSuccess (parentCode, data) {
        const { allAddress } = this.state;
        if (data.success) {
            parentCode == 0 && this.setState({ allAddress: [] });
            this.setState({ dataSource: data.context.addressList });
        } else {
            Toast(data.msg);
        }
    },
    getAddressFromLastCode (addressLastCode) {
        const type = this.props.type || 0;
        const param = { userId: app.personal.info.userId, addressLastCode, type };
        POST(app.route.ROUTE_GET_ADDRESS_FROM_LAST_CODE, param, this.getAddressFromLastCodeSuccess.bind(null, addressLastCode), false);
    },
    getAddressFromLastCodeSuccess (addressLastCode, data) {
        if (data.success) {
            this.setState({ allAddress: app.utils.getAddressTitleArray(data.context.addressList, addressLastCode), dataSource: data.context.addressList[0] || [] });
        }
    },
    getSubAddressList (obj) {
        let { allAddress } = this.state;
        const lastObj = allAddress[allAddress.length - 1];
        if ((!!lastObj && lastObj.isLeaf) || (!!lastObj && obj.parentCode==0 && lastObj.parentCode==0)) {
            allAddress.pop();
        }
        allAddress.push(obj);
        this.setState({ allAddress: _.uniqBy(allAddress, 'code') });
        if (obj.isLeaf) {
            this.props.confirmAddress(_.join(_.map(this.state.allAddress, 'name'), ''), obj.code);
            app.pop();
        } else {
            this.getRegionAddress(obj.code);
        }
    },
    onClickAddressTitle (obj, index) {
        this.state.allAddress = _.dropRight(this.state.allAddress, this.state.allAddress.length - index);
        this.getSubAddressList(obj);
    },
    renderRow (obj, rowID) {
        return (
            <TouchableOpacity
                key={rowID}
                activeOpacity={0.6}
                onPress={this.getSubAddressList.bind(null, obj)}
                style={styles.row}>
                <Text>{obj.name}</Text>
            </TouchableOpacity>
        );
    },
    renderSeparator (sectionID, rowID) {
        return (
            <View style={styles.separator} key={rowID} />
        );
    },
    render () {
        const { allAddress } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.infoStyle}>
                    <TouchableOpacity
                        activeOpacity={1}
                        onPress={this.getRegionAddress.bind(null, 0)}>
                        <Text style={styles.allAddress}>省市</Text>
                    </TouchableOpacity>
                    {
                        allAddress.map((item, i) => {
                            return (
                                <TouchableOpacity
                                    key={i}
                                    activeOpacity={1}
                                    onPress={this.onClickAddressTitle.bind(null, item, i)}>
                                    <Text style={styles.allAddress}>{item.name}</Text>
                                </TouchableOpacity>
                            );
                        })
                    }
                    {
                        <TouchableOpacity activeOpacity={1} style={styles.btnSelectAddress} >
                            <Text>选择地址</Text>
                        </TouchableOpacity>
                    }
                </View>
                <ListView
                    initialListSize={1}
                    onEndReachedThreshold={10}
                    enableEmptySections
                    dataSource={this.ds.cloneWithRows(this.state.dataSource)}
                    renderRow={this.renderRow}
                    renderSeparator={this.renderSeparator} />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    row: {
        flexDirection: 'row',
        paddingVertical: 10,
        paddingHorizontal: 12,
    },
    separator: {
        backgroundColor: '#EEEEEE',
        height: 1,
        width: sr.w,
    },
    infoStyle: {
        flexDirection: 'row',
        paddingTop:15,
        marginLeft:10,
    },
    allAddress:{
        fontSize:15,
        paddingVertical:5,
        paddingHorizontal:1,
    },
    btnSelectAddress:{
        paddingVertical:5,
        paddingHorizontal:2,
        borderColor:'red',
        borderBottomWidth:1,
    },
});

```

* PDClient/project/App/components/SelectShortAddress.js

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

module.exports = React.createClass({
    statics: {
        title: '短途地址',
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            dataSource:[],
            allAddress:[],
        };
    },
    componentWillMount () {
        const { endPointLastCode, parentCode } = this.props;
        this.getAddressFromLastCode(parentCode, endPointLastCode);
    },
    getRegionAddress (parentCode) {
        const param = { userId: app.personal.info.userId, parentCode, type:1 };
        POST(app.route.ROUTE_GET_REGION_ADDRESS_LIST, param, this.getRegionAddressSuccess.bind(null, parentCode), false);
    },
    getRegionAddressSuccess (parentCode, data) {
        const { allAddress } = this.state;
        if (data.success) {
            parentCode == this.props.parentCode && this.setState({ allAddress: [] });
            this.setState({ dataSource: data.context.addressList });
        } else {
            Toast(data.msg);
        }
    },
    getAddressFromLastCode (parentCode, addressLastCode) {
        const param = { userId: app.personal.info.userId, parentCode, addressLastCode };
        POST(app.route.ROUTE_GET_SEND_DOOR_ADDRESS_FROM_LAST_CODE, param, this.getAddressFromLastCodeSuccess.bind(null, addressLastCode), false);
    },
    getAddressFromLastCodeSuccess (addressLastCode, data) {
        if (data.success) {
            this.setState({ allAddress: app.utils.getAddressTitleArray(data.context.addressList, addressLastCode), dataSource:data.context.addressList[0] || [] });
        }
    },
    getSubAddressList (obj) {
        let { allAddress } = this.state;
        const { style } = this.props;
        const lastObj = allAddress[allAddress.length - 1];
        if (!!lastObj && lastObj.isLeaf) {
            allAddress.pop();
        }
        allAddress.push(obj);
        this.setState({ allAddress: _.uniqBy(allAddress, 'code') });
        if (obj.isLeaf) {
            this.props.confirmAddress(_.join(_.map(this.state.allAddress, 'name'), ''), obj.code);
            !style && app.pop();
        } else {
            this.getRegionAddress(obj.code);
        }
    },
    onClickAddressTitle (obj, index) {
        this.state.allAddress = _.dropRight(this.state.allAddress, this.state.allAddress.length - index);
        this.getSubAddressList(obj);
    },
    renderRow (obj, rowID) {
        return (
            <TouchableOpacity
                key={rowID}
                activeOpacity={0.6}
                onPress={this.getSubAddressList.bind(null, obj)}
                style={styles.row}>
                <Text>{obj.name}</Text>
            </TouchableOpacity>
        );
    },
    renderSeparator (sectionID, rowID) {
        return (
            <View style={styles.separator} key={rowID} />
        );
    },
    render () {
        const { allAddress } = this.state;
        const { parentCode,style } = this.props;
        return (
            <View style={style || styles.container}>
                <View style={styles.infoStyle}>
                    <TouchableOpacity
                        activeOpacity={1}
                        onPress={this.getRegionAddress.bind(null, parentCode)}>
                        <Text style={styles.allAddress}>区域</Text>
                    </TouchableOpacity>
                    {
                        allAddress.map((item, i) => {
                            return (
                                <TouchableOpacity
                                    key={i}
                                    activeOpacity={1}
                                    onPress={this.onClickAddressTitle.bind(null, item, i)}>
                                    <Text style={styles.allAddress}>{item.name}</Text>
                                </TouchableOpacity>
                            );
                        })
                    }
                    {
                        <TouchableOpacity
                            activeOpacity={1}
                            style={styles.btnSelectAddress}>
                            <Text>选择地址</Text>
                        </TouchableOpacity>
                    }
                </View>
                <ListView
                    initialListSize={1}
                    onEndReachedThreshold={10}
                    enableEmptySections
                    dataSource={this.ds.cloneWithRows(this.state.dataSource)}
                    renderRow={this.renderRow}
                    renderSeparator={this.renderSeparator} />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    row: {
        flexDirection: 'row',
        paddingVertical: 10,
        paddingHorizontal: 12,
    },
    separator: {
        backgroundColor: '#EEEEEE',
        height: 1,
        width: sr.w,
    },
    infoStyle: {
        flexDirection: 'row',
        paddingTop:15,
        marginLeft:10,
    },
    allAddress:{
        fontSize:15,
        paddingVertical:5,
        paddingHorizontal:1,
    },
    btnSelectAddress:{
        paddingVertical:5,
        paddingHorizontal:2,
        borderColor:'red',
        borderBottomWidth:1,
    },
});

```

* PDClient/project/App/components/SelectStartPointAddress.js

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

module.exports = React.createClass({
    statics: {
        title: '始发地',
        rightButton: { title: '确定', handler: () => { app.scene.confirm&&app.scene.confirm(); } },
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        this.lastLevel = null;
        return {
            dataSource:[],
            allAddress:[],
        };
    },
    componentWillMount () {
        const { endPointLastCode, id } = this.props;
        !endPointLastCode || endPointLastCode == '' ? this.getStartPointAddress(0) : this.getAddressFromLastCode(endPointLastCode, id);
    },
    getStartPointAddress (parentCode) {
        const param = { userId: app.personal.info.userId, parentCode };
        POST(app.route.ROUTE_GET_START_POINT_ADDRESS_LIST, param, this.getStartPointAddressSuccess.bind(null, parentCode), false);
    },
    getStartPointAddressSuccess (parentCode, data) {
        const { allAddress } = this.state;
        if (data.success) {
            parentCode == 0 && this.setState({ allAddress: [] });
            let dataSource = this.props.shopOnly&&_.includes([ 0, 4, 5, 6, 7 ], this.lastLevel)?_.filter(data.context.addressList, 'isShop', true):data.context.addressList;
            this.setState({ dataSource });
        } else {
            Toast(data.msg);
        }
    },
    getAddressFromLastCode (addressLastCode, id) {
        const param = { userId: app.personal.info.userId, addressLastCode, isLeaf: !!id };
        POST(app.route.ROUTE_GET_START_POINT_ADDRESS_FROME_LAST_CODE, param, this.getAddressFromLastCodeSuccess.bind(null, addressLastCode, id), false);
    },
    getAddressFromLastCodeSuccess (addressLastCode, id, data) {
        if (data.success) {
            this.setState({ allAddress: app.utils.getStartPointTitleArray(data.context.addressList, addressLastCode, id), dataSource:data.context.addressList[0]||[]});
        }
    },
    confirm () {
        const { allAddress } = this.state;
        const lastObj = allAddress[allAddress.length - 1];
        const { confirmAddress, isNeedSelectAll } = this.props;
        if (isNeedSelectAll) {
            if (!!lastObj && lastObj.isLeaf || _.includes([ 0, 4, 5, 6, 7 ], lastObj.level)) {
                if (lastObj.isShop) {
                    confirmAddress({ startPoint:_.join(_.map(allAddress, 'name'), ''), shopId:lastObj.id, startPointLastCode:lastObj.addressRegionLastCode });
                } else if (lastObj.isAgent) {
                    confirmAddress({ startPoint:_.join(_.map(allAddress, 'name'), ''), agentId:lastObj.id, startPointLastCode:lastObj.addressRegionLastCode });
                } else {
                    confirmAddress({ startPoint:_.join(_.map(allAddress, 'name'), ''), startPointLastCode: lastObj.code });
                }
                app.pop();
            } else {
                Toast('请选择完整到货地');
            }
        } else {
            console.log('lastObj.code',lastObj);
            confirmAddress({ startPoint:_.join(_.map(allAddress, 'name'), ''), startPointLastCode:lastObj && lastObj.code || null });
            app.pop();
        }
    },
    getSubAddressList (obj) {
        let { allAddress } = this.state;
        const lastObj = allAddress[allAddress.length - 1];
        if ((!!lastObj && lastObj.isLeaf) || (!!lastObj && obj.parentCode==0 && lastObj.parentCode==0)) {
            allAddress.pop();
        }
        allAddress.push(obj);
        this.setState({ allAddress: _.uniqBy(allAddress, 'code') });
        if (obj.isLeaf) { // 省+市+名字
            if (obj.isShop) {
                this.props.confirmAddress({ startPoint:obj.name, shopId:obj.id, startPointLastCode:obj.addressRegionLastCode });
            } else if (obj.isAgent) {
                this.props.confirmAddress({ startPoint:obj.name, agentId:obj.id, startPointLastCode:obj.addressRegionLastCode });
            } else {
                this.props.confirmAddress({ startPoint:_.join(_.map(this.state.allAddress, 'name'), ''), startPointLastCode:obj.code });
            }
            app.pop();
        } else {
            this.lastLevel = obj.level;
            this.getStartPointAddress(obj.code);
        }
    },
    onClickAddressTitle (obj, index) {
        this.state.allAddress = _.dropRight(this.state.allAddress, this.state.allAddress.length - index);
        this.getSubAddressList(obj);
    },
    renderRow (obj, rowID) {
        return (
            <TouchableOpacity
                key={rowID}
                activeOpacity={0.6}
                onPress={this.getSubAddressList.bind(null, obj)}
                style={styles.row}>
                <Text>{obj.name}</Text>
            </TouchableOpacity>
        );
    },
    renderSeparator (sectionID, rowID) {
        return (
            <View style={styles.separator} key={rowID} />
        );
    },
    render () {
        const { allAddress } = this.state;
        const { style } = this.props;
        return (
            <View style={style || styles.container}>
                <View style={styles.infoStyle}>
                    <TouchableOpacity
                        activeOpacity={1}
                        onPress={this.getStartPointAddress.bind(null, 0)}>
                        <Text style={styles.allAddress}>省市</Text>
                    </TouchableOpacity>
                    {
                        allAddress.map((item, i) => {
                            return (
                                <TouchableOpacity
                                    key={i}
                                    activeOpacity={1}
                                    onPress={this.onClickAddressTitle.bind(null, item, i)}>
                                    <Text style={styles.allAddress}>{item.name}</Text>
                                </TouchableOpacity>
                            );
                        })
                    }
                    {
                        allAddress.length != 4 &&
                            <TouchableOpacity
                                activeOpacity={1}
                                style={styles.btnSelectAddress}>
                                <Text>选择地址</Text>
                            </TouchableOpacity>
                    }
                    {
                        !!style &&
                            <TouchableOpacity
                                activeOpacity={0.6}
                                onPress={this.confirm}
                                style={styles.btnConfirmAddress}>
                                <Text>确定</Text>
                            </TouchableOpacity>
                    }
                </View>
                <ListView
                    initialListSize={1}
                    onEndReachedThreshold={10}
                    enableEmptySections
                    dataSource={this.ds.cloneWithRows(this.state.dataSource)}
                    renderRow={this.renderRow}
                    renderSeparator={this.renderSeparator} />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#FFFFFF',
    },
    row: {
        flexDirection: 'row',
        paddingVertical: 10,
        paddingHorizontal: 12,
    },
    separator: {
        backgroundColor: '#EEEEEE',
        height: 1,
        width: sr.w,
    },
    infoStyle: {
        flexDirection: 'row',
        paddingTop:15,
        marginLeft:10,
    },
    allAddress:{
        fontSize:15,
        paddingVertical:5,
        paddingHorizontal:1,
    },
    btnSelectAddress:{
        paddingVertical:5,
        paddingHorizontal:2,
        borderColor:'red',
        borderBottomWidth:1,
    },
    btnConfirmAddress:{
        paddingVertical:5,
        paddingHorizontal:10,
    },
});

```

* PDClient/project/App/components/Slider.js

```js
'use strict';

const React = require('react');const {    PropTypes,} = React;const ReactNative = require('react-native');
const {
  StyleSheet,
  PanResponder,
  View,
  Platform,
} = ReactNative;

const TRACK_SIZE = 4;
const THUMB_SIZE = 20;

function Rect (x, y, width, height) {
    this.x = x;
    this.y = y;
    this.width = width;
    this.height = height;
}

Rect.prototype.containsPoint = function (x, y) {
    return (x >= this.x
          && y >= this.y
          && x <= this.x + this.width
          && y <= this.y + this.height);
};

const Slider = React.createClass({
    propTypes: {
    /**
     * Initial value of the slider. The value should be between minimumValue
     * and maximumValue, which default to 0 and 1 respectively.
     * Default value is 0.
     *
     * *This is not a controlled component*, e.g. if you don't update
     * the value, the component won't be reset to its inital value.
     */
        value: PropTypes.number,

    /**
     * Initial minimum value of the slider. Default value is 0.
     */
        minimumValue: PropTypes.number,

    /**
     * Initial maximum value of the slider. Default value is 1.
     */
        maximumValue: PropTypes.number,

    /**
     * The color used for the track to the left of the button. Overrides the
     * default blue gradient image.
     */
        minimumTrackTintColor: PropTypes.string,

    /**
     * The color used for the track to the right of the button. Overrides the
     * default blue gradient image.
     */
        maximumTrackTintColor: PropTypes.string,

    /**
     * The color used for the thumb.
     */
        thumbTintColor: PropTypes.string,

    /**
     * The size of the touch area that allows moving the thumb.
     * The touch area has the same center has the visible thumb.
     * This allows to have a visually small thumb while still allowing the user
     * to move it easily.
     * The default is {width: 40, height: 40}.
     */
        thumbTouchSize: PropTypes.shape(
      { width: PropTypes.number, height: PropTypes.number }
    ),

    /**
     * Callback continuously called while the user is dragging the slider.
     */
        onValueChange: PropTypes.func,

    /**
     * Callback called when the user starts changing the value (e.g. when
     * the slider is pressed).
     */
        onSlidingStart: PropTypes.func,

    /**
     * Callback called when the user finishes changing the value (e.g. when
     * the slider is released).
     */
        onSlidingComplete: PropTypes.func,

    /**
     * The style applied to the slider container.
     */
        style: PropTypes.style,

    /**
     * The style applied to the track.
     */
        trackStyle: PropTypes.style,

    /**
     * The style applied to the thumb.
     */
        thumbStyle: PropTypes.style,

    /**
     * Set this to true to visually see the thumb touch rect in green.
     */
        debugTouchArea: PropTypes.bool,
    },
    getInitialState () {
        return {
            containerSize: {},
            trackSize: {},
            thumbSize: {},
            previousLeft: 0,
            value: this.props.value,
        };
    },
    getDefaultProps () {
        return {
            value: 0,
            minimumValue: 0,
            maximumValue: 1,
            minimumTrackTintColor: '#3f3f3f',
            maximumTrackTintColor: '#b3b3b3',
            thumbTintColor: '#343434',
            thumbTouchSize: { width: 40, height: 40 },
            debugTouchArea: false,
        };
    },
    componentWillMount () {
        this._panResponder = PanResponder.create({
            onStartShouldSetPanResponder: this._handleStartShouldSetPanResponder,
            onMoveShouldSetPanResponder: this._handleMoveShouldSetPanResponder,
            onPanResponderGrant: this._handlePanResponderGrant,
            onPanResponderMove: this._handlePanResponderMove,
            onPanResponderRelease: this._handlePanResponderEnd,
            onPanResponderTerminate: this._handlePanResponderEnd,
        });
    },
    componentWillReceiveProps (nextProps) {
        this.setState({ value: nextProps.value });
    },
    render () {
        const {
      minimumTrackTintColor,
      maximumTrackTintColor,
      thumbTintColor,
      styles,
      style,
      trackStyle,
      thumbStyle,
      debugTouchArea,
      ...other
    } = this.props;
        const { value, containerSize, trackSize, thumbSize } = this.state;
        const mainStyles = styles || defaultStyles;
        const thumbLeft = this._getThumbLeft(value);
        const valueVisibleStyle = {};
        if (containerSize.width === undefined
        || trackSize.width === undefined
        || thumbSize.width === undefined) {
            valueVisibleStyle.opacity = 0;
        }

        const minimumTrackStyle = {
            position: 'absolute',
            width: 300, // needed to workaround a bug for borderRadius
            marginTop: -trackSize.height,
            backgroundColor: minimumTrackTintColor,
            ...valueVisibleStyle,
        };

        if (thumbLeft >= 0 && thumbSize.width >= 0) {
            minimumTrackStyle.width = thumbLeft + thumbSize.width / 2;
        }

        const touchOverflowStyle = this._getTouchOverflowStyle();

        return (
            <View style={{ flex:1, justifyContent:'center' }}>
                <View
                    {...other}
                    style={[mainStyles.container, style]}
                    onLayout={this._measureContainer}>
                    <View
                        style={[{ backgroundColor: maximumTrackTintColor }, mainStyles.track, trackStyle]}
                        onLayout={this._measureTrack} />
                    <View style={[mainStyles.track, trackStyle, minimumTrackStyle]} />
                </View>
                <View
                    ref={(thumb) => { this.thumb = thumb; }}
                    onLayout={this._measureThumb}
                    style={[
                    { backgroundColor: thumbTintColor, marginTop: -(trackSize.height + thumbSize.height) / 2 },
                        mainStyles.thumb, thumbStyle, { left: thumbLeft, ...valueVisibleStyle },
                    ]}
                />
                <View
                    style={[defaultStyles.touchArea, touchOverflowStyle]}
                    {...this._panResponder.panHandlers}>
                    {debugTouchArea === true && this._renderDebugThumbTouchRect()}
                </View>
            </View>
        );
    },

    _handleStartShouldSetPanResponder (e: Object, /* gestureState: Object */): boolean {
    // Until the PR https://github.com/facebook/react-native/pull/3426 is merged, we need to always return "true" for android
        if (Platform.OS === 'android') {
            return true;
        }
    // Should we become active when the user presses down on the thumb?
        return this._thumbHitTest(e);
    },

    _handleMoveShouldSetPanResponder (/* e: Object, gestureState: Object */): boolean {
    // Should we become active when the user moves a touch over the thumb?
        return false;
    },

    _handlePanResponderGrant (/* e: Object, gestureState: Object */) {
        this.setState({ previousLeft: this._getThumbLeft(this.state.value) },
      this._fireChangeEvent.bind(this, 'onSlidingStart'));
    },
    _handlePanResponderMove (e: Object, gestureState: Object) {
        this.setState({ value: this._getValue(gestureState) },
      this._fireChangeEvent.bind(this, 'onValueChange'));
    },
    _handlePanResponderEnd (e: Object, gestureState: Object) {
        this.setState({ value: this._getValue(gestureState) },
      this._fireChangeEvent.bind(this, 'onSlidingComplete'));
    },

    _measureContainer (x: Object) {
        const { width, height } = x.nativeEvent.layout;
        const containerSize = { width: width, height: height };
        this.setState({ containerSize: containerSize });
    },

    _measureTrack (x: Object) {
        const { width, height } = x.nativeEvent.layout;
        const trackSize = { width: width, height: height };
        this.setState({ trackSize: trackSize });
    },

    _measureThumb (x: Object) {
        const { width, height } = x.nativeEvent.layout;
        const thumbSize = { width: width, height: height };
        this.setState({ thumbSize: thumbSize });
    },

    _getRatio (value: number) {
        return (value - this.props.minimumValue) / (this.props.maximumValue - this.props.minimumValue);
    },

    _getThumbLeft (value: number) {
        const ratio = this._getRatio(value);
        return ratio * (this.state.containerSize.width - this.state.thumbSize.width);
    },

    _getValue (gestureState: Object) {
        const length = this.state.containerSize.width - this.state.thumbSize.width;
        const thumbLeft = Math.min(length,
      Math.max(0, this.state.previousLeft + (this.props.vertical ? gestureState.dy : gestureState.dx)));

        const ratio = thumbLeft / length;
        return ratio * (this.props.maximumValue - this.props.minimumValue) + this.props.minimumValue;
    },

    _fireChangeEvent (event) {
        if (this.props[event]) {
            this.props[event](this.state.value);
        }
    },

    _getTouchOverflowSize () {
        const state = this.state;
        const props = this.props;

        const size = {};
        if (state.containerSize.width !== undefined
        && state.thumbSize.width !== undefined) {
            size.width = Math.max(0, props.thumbTouchSize.width - state.thumbSize.width);
            size.height = Math.max(0, props.thumbTouchSize.height - state.containerSize.height);
        }

        return size;
    },

    _getTouchOverflowStyle () {
        const { width, height } = this._getTouchOverflowSize();

        const touchOverflowStyle = {};
        if (width !== undefined && height !== undefined) {
            const verticalMargin = -height / 2;
            touchOverflowStyle.marginTop = verticalMargin;
            touchOverflowStyle.marginBottom = verticalMargin;

            const horizontalMargin = -width / 2;
            touchOverflowStyle.marginLeft = horizontalMargin;
            touchOverflowStyle.marginRight = horizontalMargin;
        }

        if (this.props.debugTouchArea === true) {
            touchOverflowStyle.backgroundColor = 'orange';
            touchOverflowStyle.opacity = 0.5;
        }

        return touchOverflowStyle;
    },

    _thumbHitTest (e: Object) {
        const nativeEvent = e.nativeEvent;
        const thumbTouchRect = this._getThumbTouchRect();
        return thumbTouchRect.containsPoint(nativeEvent.locationX, nativeEvent.locationY);
    },

    _getThumbTouchRect () {
        const state = this.state;
        const props = this.props;
        const touchOverflowSize = this._getTouchOverflowSize();

        return new Rect(
      touchOverflowSize.width / 2 + this._getThumbLeft(state.value) + (state.thumbSize.width - props.thumbTouchSize.width) / 2,
      touchOverflowSize.height / 2 + (state.containerSize.height - props.thumbTouchSize.height) / 2,
      props.thumbTouchSize.width,
      props.thumbTouchSize.height
    );
    },

    _renderDebugThumbTouchRect () {
        const thumbTouchRect = this._getThumbTouchRect();
        const positionStyle = {
            left: thumbTouchRect.x,
            top: thumbTouchRect.y,
            width: thumbTouchRect.width,
            height: thumbTouchRect.height,
        };

        return (
            <View
                style={[defaultStyles.debugThumbTouchArea, positionStyle]}
                pointerEvents='none'
      />
        );
    },
});

const defaultStyles = StyleSheet.create({
    container: {
        height: 40,
        justifyContent: 'center',
    },
    track: {
        height: TRACK_SIZE,
        borderRadius: TRACK_SIZE / 2,
    },
    thumb: {
        position: 'absolute',
        width: THUMB_SIZE,
        height: THUMB_SIZE,
        borderRadius: THUMB_SIZE / 2,
    },
    touchArea: {
        position: 'absolute',
        backgroundColor: 'transparent',
        top: 0,
        left: 0,
        right: 0,
        bottom: 0,
    },
    debugThumbTouchArea: {
        position: 'absolute',
        backgroundColor: 'green',
        opacity: 0.5,
    },
});

module.exports = Slider;

```

* PDClient/project/App/components/StarBar.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    Image,
    View,
    StyleSheet,
} = ReactNative;

const MAX_STAR_NUM = 5;

module.exports = React.createClass({
    render () {
        const value = this.props.value;
        const starNum = Math.floor(value);
        const rest = value - starNum;
        const list = [];
        let i;
        for (i = 0; i < starNum; i++) {
            list[i] = 1;
        }
        if (i < MAX_STAR_NUM) {
            list[i] = rest <= 0.31 ? 2 : rest >= 0.69 ? 1 : 3;
            while (++i < MAX_STAR_NUM) {
                list[i] = 2;
            }
        }
        return (
            <View style={this.props.style ? this.props.style : styles.scoreIconContainer}>
                {
                    list.map((item, i) => {
                        const imgSource = app.img['actualCombat_star_' + item];
                        return (
                            <Image
                                key={i}
                                resizeMode='stretch'
                                source={imgSource}
                                style={this.props.starStyle ? this.props.starStyle : styles.scoreIcon}
                                />
                        );
                    })
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    scoreIconContainer: {
        height: 30,
        flexDirection: 'row',
        justifyContent: 'center',
    },
    scoreIcon: {
        marginLeft: 3,
        width: 20,
        height: 20,
    },
});

```

* PDClient/project/App/components/TouchAbleLink.js

```js
import React, { Component } from 'react';
import {
    Text,
    Linking,
    TouchableOpacity,
} from 'react-native';

// 打电话：url->("tel:10086")
// 发邮件：url->("mailto:10086@qq.com")
// 打开网站:url->("http://www.baidu.com")
module.exports = React.createClass ({
    render () {
        const {url, style, children} = this.props;
        return (
            <TouchableOpacity style={style} onPress={() => {
                Linking.canOpenURL(url).then(supported => {
                    if (!supported) {
                        console.log('Can\'t handle url: ' + url);
                    } else {
                        return Linking.openURL(url);
                    }
                }).catch(err => console.error('An error occurred', err));
            }}>
                {children}
            </TouchableOpacity>

        );
    }
});

```

* PDClient/project/App/components/WebviewMessageBox.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    WebView,
    Text,
} = ReactNative;

const Button = require('./Button');

module.exports = React.createClass({
    render () {
        return (
            <View style={styles.overlayContainer}>
                {
                    !this.props.title &&
                    <View style={[styles.title, { backgroundColor: app.setting.THEME_COLOR }]}>
                        <View style={[styles.titleContainer, { marginTop: sr.trueStatusBarHeight }]}>
                            <Text style={styles.titleText}>
                                {this.props.title}
                            </Text>
                        </View>
                    </View>
                }
                <View style={[styles.container, { top: sr.totalNavbarHeight + sr.statusBarHeight }]}>
                    <WebView
                        style={styles.webview}
                        source={{ uri:this.props.webAddress }}
                        scalesPageToFit
                        />
                    <Button
                        onPress={app.closeModal}
                        style={styles.contentButton}
                        textStyle={styles.contentButtonText}>
                        返回</Button>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    overlayContainer: {
        position:'absolute',
        top: 0,
        bottom: 0,
        left:0,
        right: 0,
        backgroundColor: 'transparent',
    },
    title: {
        height:sr.totalNavbarHeight,
        width: sr.w,
    },
    titleContainer: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
    },
    titleText: {
        fontSize: 18,
        color: 'gray',
        fontWeight: '500',
    },
    container: {
        width:sr.w * 5 / 6,
        height:sr.h * 4 / 5,
        alignItems:'center',
        justifyContent:'center',
        borderRadius:10,
        position: 'absolute',
        left: sr.w * 1 / 12,
    },
    webview: {
        width:sr.w * 5 / 6,
        height:sr.h * 4 / 5 - 50,
    },
    contentButton: {
        width:sr.w * 5 / 6,
        height:50,
        borderRadius:0,
    },
    contentButtonText: {
        color: '#000000',
        fontWeight: '800',
    },
});

```

* PDClient/project/App/components/index.js

```js
module.exports = {
    Button: require('./Button'),
    PageList: require('./PageList'),
    ActionSheet: require('./ActionSheet/index'),
    ActionView: require('./ActionSheet/view'),
    CustomSheet: require('./CustomSheet'),
    ClipRect: require('./ClipRect'),
    Modal: require('./Modal'),
    DImage: require('./DImage'),
    Label: require('./Label'),
    ProgressBar: require('./ProgressBar'),
    Slider: require('./Slider'),
    MessageBox: require('./MessageBox'),
    CustomMessageBox: require('./CustomMessageBox'),
    WebviewMessageBox: require('./WebviewMessageBox'),
    DelayTouchableOpacity: require('./DelayTouchableOpacity'),
    StarBar: require('./StarBar'),
    ProgressHud: require('./ProgressHud'),
    RText: require('./RText'),
    ScoreSelect: require('./ScoreSelect'),
    AutogrowInput: require('./AutogrowInput'),
    TouchAbleLink: require('./TouchAbleLink'),
    Picker: require('./Picker'),
    SelectRegionAddress: require('./SelectRegionAddress'),
    SelectStartPointAddress: require('./SelectStartPointAddress'),
    SelectShortAddress: require('./SelectShortAddress'),
};

```

* PDClient/project/App/config/Authority.js

```js
'use strict';
// 注意：常量部分必须和服务器保持一致
// 总部权限
const AH_CREATE_BRANCH_SHOP = 0; // 创建分店的权限
const AH_MODIFY_BRANCH_SHOP = 1; // 修改分店信息的权限
const AH_REMOVE_BRANCH_SHOP = 2; // 删除分店的权限
const AH_LOOK_BRANCH_SHOP = 3; // 查看分店列表的权限
const AH_MODIFY_SETTING = 4; // 修改设置的权限
const AH_CREATE_AGENT = 5; // 创建收货点的权限
const AH_MODIFY_AGENT = 6; // 修改收货点的权限
const AH_REMOVE_AGENT = 7; // 删除收货点的权限
const AH_LOOK_AGENT = 8; // 查看收货点的权限

// 公共权限
const AH_MODIFY_OWN_SHOP = 10000; // 修改所在物流超市信息的权限
const AH_CREATE_MEMBER = 10001; // 创建成员的权限
const AH_MODIFY_MEMBER = 10002; // 修改成员信息的权限
const AH_REMOVE_MEMBER = 10003; // 删除成员的权限
const AH_LOOK_MEMBER = 10004; // 查看成员的权限
const AH_RECHARGE = 10005; // 充值的权限
const AH_WITHDRAW = 10006; // 提现的权限
const AH_LOOK_AMOUNT = 10007; // 查看余额的权限
const AH_MODIFY_ROADMAP_PROFIT_RATE = 10008; // 修改路线的提成
const AH_LOOK_STATISTICS = 10009; // 查看统计信息的权限
const AH_LOOK_ORDERS = 10010; // 查看综合货单的权限

// 分店权限
const AH_CREATE_SHIPPER = 20000; // 创建物流公司的权限
const AH_MODIFY_SHIPPER = 20001; // 修改物流公司的权限
const AH_REMOVE_SHIPPER = 20002; // 删除物流公司的权限
const AH_LOOK_SHIPPER = 20003; // 查看物流公司的权限
const AH_CREATE_PARTMENT = 20004; // 创建部门的权限
const AH_MODIFY_PARTMENT = 20005; // 修改部门信息的权限
const AH_REMOVE_PARTMENT = 20006; // 删除部门的权限
const AH_LOOK_PARTMENT = 20007; // 查看部门的权限
const AH_MODIFY_OWN_PARTMENT = 20008; // 修改所在部门信息的权限
const AH_RECEIVE_PARTMENT = 20009; // 收货的权限（收货部人员）
const AH_WARE_HOUSE_PARTMENT = 20010; // 库管的权限（库管部人员）
const AH_CARRY_PARTMENT = 20011; // 搬货的权限（搬运部人员）
const AH_SECURITY_CHECK_PARTMENT = 20012; // 安检的权限（保安部人员）
const AH_DISTRIBUTION_PARTMENT = 20013; // 配送的权限（配送部人员）
const AH_TO_EXAMINE_TRUCK = 20014; // 审核货车的权限

// 物流公司
const AH_MODIFY_SHIPPER_INFO = 30000; // 修改物流公司信息的权限
const AH_MODIFY_SHIPPER_MEMBER_AUTHORITY = 30001; // 修改物流公司成员权限的权限
const AH_CREATE_ROADMAP = 30002; // 创建路线的权限
const AH_MODIFY_ROADMAP = 30003; // 修改路线的权限
const AH_REMOVE_ROADMAP = 30004; // 删除路线的权限
const AH_LOOK_ROADMAP = 30005; // 查看路线的权限 (分店共享该权限)
const AH_CREATE_SEND_DOOR_ROADMAP = 30006; // 创建同城配送路线的权限
const AH_MODIFY_SEND_DOOR_ROADMAP = 30007; // 修改同城配送路线的权限
const AH_REMOVE_SEND_DOOR_ROADMAP = 30008; // 删除同城配送路线的权限
const AH_LOOK_SEND_DOOR_ROADMAP = 30009; // 查看同城配送路线的权限 (分店共享该权限)
const AH_CREATE_TRUCK = 30010; // 创建货车的权限
const AH_MODIFY_TRUCK = 30011; // 修改货车的权限
const AH_REMOVE_TRUCK = 30012; // 删除货车的权限
const AH_LOOK_TRUCK = 30013; // 查看货车的权限
const SELECT_CARRY_PARTMENT = 30014; // 选择搬运队的权限
const AH_SCAN_LOAD_TRUCK = 30015; // 扫描装车的权限
const AH_CITY_DISTRIBUTE = 30016; // 同城配送的权限

// 收货点
const AH_MODIFY_AGENT_INFO = 40000; // 修改收货点信息的权限
const AH_MODIFY_AGENT_MEMBER_AUTHORITY = 40001; // 修改收货点成员权限的权限
const AH_PLACE_ORDER = 40002; // 收货点下单的权限

module.exports = {
    // 总部权限
    AH_CREATE_BRANCH_SHOP,
    AH_MODIFY_BRANCH_SHOP,
    AH_REMOVE_BRANCH_SHOP,
    AH_LOOK_BRANCH_SHOP,
    AH_MODIFY_SETTING,
    AH_CREATE_AGENT,
    AH_MODIFY_AGENT,
    AH_REMOVE_AGENT,
    AH_LOOK_AGENT,

    // 公共权限
    AH_MODIFY_OWN_SHOP,
    AH_CREATE_MEMBER,
    AH_MODIFY_MEMBER,
    AH_REMOVE_MEMBER,
    AH_LOOK_MEMBER,
    AH_RECHARGE,
    AH_WITHDRAW,
    AH_LOOK_AMOUNT,
    AH_MODIFY_ROADMAP_PROFIT_RATE,
    AH_LOOK_STATISTICS,
    AH_LOOK_ORDERS,

    // 分店权限
    AH_CREATE_SHIPPER,
    AH_MODIFY_SHIPPER,
    AH_REMOVE_SHIPPER,
    AH_LOOK_SHIPPER,
    AH_CREATE_PARTMENT,
    AH_MODIFY_PARTMENT,
    AH_REMOVE_PARTMENT,
    AH_LOOK_PARTMENT,
    AH_MODIFY_OWN_PARTMENT,
    AH_RECEIVE_PARTMENT,
    AH_WARE_HOUSE_PARTMENT,
    AH_CARRY_PARTMENT,
    AH_SECURITY_CHECK_PARTMENT,
    AH_DISTRIBUTION_PARTMENT,
    AH_TO_EXAMINE_TRUCK,

    // 物流公司
    AH_MODIFY_SHIPPER_INFO,
    AH_MODIFY_SHIPPER_MEMBER_AUTHORITY,
    AH_CREATE_ROADMAP,
    AH_MODIFY_ROADMAP,
    AH_REMOVE_ROADMAP,
    AH_LOOK_ROADMAP,
    AH_CREATE_SEND_DOOR_ROADMAP,
    AH_MODIFY_SEND_DOOR_ROADMAP,
    AH_REMOVE_SEND_DOOR_ROADMAP,
    AH_LOOK_SEND_DOOR_ROADMAP,
    AH_CREATE_TRUCK,
    AH_MODIFY_TRUCK,
    AH_REMOVE_TRUCK,
    AH_LOOK_TRUCK,
    SELECT_CARRY_PARTMENT,
    AH_SCAN_LOAD_TRUCK,
    AH_CITY_DISTRIBUTE,

    // 收货点
    AH_MODIFY_AGENT_INFO,
    AH_MODIFY_AGENT_MEMBER_AUTHORITY,
    AH_PLACE_ORDER,

    // mapper
    MAP: {
        // 总部权限
        [AH_CREATE_BRANCH_SHOP]: '创建分店的权限',
        [AH_MODIFY_BRANCH_SHOP]: '修改分店信息的权限',
        [AH_REMOVE_BRANCH_SHOP]: '删除分店的权限',
        [AH_LOOK_BRANCH_SHOP]: '查看分店的权限',
        [AH_MODIFY_SETTING]: '修改设置的权限',
        [AH_CREATE_AGENT]: '创建收货点的权限',
        [AH_MODIFY_AGENT]: '修改收货点的权限',
        [AH_REMOVE_AGENT]: '删除收货点的权限',
        [AH_LOOK_AGENT]: '查看收货点的权限',

        // 公共权限
        [AH_MODIFY_OWN_SHOP]: '修改自身店铺信息的权限',
        [AH_CREATE_MEMBER]: '创建成员的权限',
        [AH_MODIFY_MEMBER]: '修改成员信息的权限',
        [AH_REMOVE_MEMBER]: '删除成员的权限',
        [AH_LOOK_MEMBER]: '查看成员的权限',
        [AH_RECHARGE]: '充值的权限',
        [AH_WITHDRAW]: '提现的权限',
        [AH_LOOK_AMOUNT]: '查看余额的权限',
        [AH_MODIFY_ROADMAP_PROFIT_RATE]: '修改路线的提成的权限',
        [AH_LOOK_STATISTICS]: '查看统计信息的权限',
        [AH_LOOK_ORDERS]: '查看综合货单的权限',

        // 分店权限
        [AH_CREATE_SHIPPER]: '创建物流公司',
        [AH_MODIFY_SHIPPER]: '修改物流公司',
        [AH_REMOVE_SHIPPER]: '删除物流公司',
        [AH_LOOK_SHIPPER]: '查看物流公司',
        [AH_CREATE_PARTMENT]: '创建部门的权限',
        [AH_MODIFY_PARTMENT]: '修改部门信息的权限',
        [AH_REMOVE_PARTMENT]: '删除部门的权限',
        [AH_LOOK_PARTMENT]: '查看部门的权限',
        [AH_MODIFY_OWN_PARTMENT]: '修改所在部门信息的权限',
        [AH_RECEIVE_PARTMENT]: '收货的权限',
        [AH_WARE_HOUSE_PARTMENT]: '库管的权限',
        [AH_CARRY_PARTMENT]: '搬货的权限',
        [AH_SECURITY_CHECK_PARTMENT]: '安检的权限',
        [AH_DISTRIBUTION_PARTMENT]: '配送的权限',
        [AH_TO_EXAMINE_TRUCK]: '审核货车的权限',

        // 物流公司
        [AH_MODIFY_SHIPPER_INFO]: '修改物流公司信息的权限',
        [AH_MODIFY_SHIPPER_MEMBER_AUTHORITY]: '修改物流公司成员权限的权限',
        [AH_CREATE_ROADMAP]: '创建路线的权限',
        [AH_MODIFY_ROADMAP]: '修改路线的权限',
        [AH_REMOVE_ROADMAP]: '删除路线的权限',
        [AH_LOOK_ROADMAP]: '查看路线的权限',
        [AH_CREATE_SEND_DOOR_ROADMAP]: '创建同城配送路线的权限',
        [AH_MODIFY_SEND_DOOR_ROADMAP]: '修改同城配送路线的权限',
        [AH_REMOVE_SEND_DOOR_ROADMAP]: '删除同城配送路线的权限',
        [AH_LOOK_SEND_DOOR_ROADMAP]: '查看同城配送路线的权限',
        [AH_CREATE_TRUCK]: '创建货车的权限',
        [AH_MODIFY_TRUCK]: '修改货车的权限',
        [AH_REMOVE_TRUCK]: '删除货车的权限',
        [AH_LOOK_TRUCK]: '查看货车的权限',
        [SELECT_CARRY_PARTMENT]: '选择搬运队的权限',
        [AH_SCAN_LOAD_TRUCK]: '扫描装车的权限',
        [AH_CITY_DISTRIBUTE]: '同城配送的权限',

        // 收货点
        [AH_MODIFY_AGENT_INFO]: '修改收货点信息的权限',
        [AH_MODIFY_AGENT_MEMBER_AUTHORITY]: '修改收货点成员权限的权限',
        [AH_PLACE_ORDER]: '收货点下单的权限',
    },
};

```

* PDClient/project/App/config/Color.js

```js
'use strict';

module.exports = {
    _c (style, color) {
        return [style, { color: color || app.setting.THEME_COLOR }];
    },
    _bc (style, color) {
        return [style, { backgroundColor:color || app.setting.THEME_COLOR }];
    },
    _tc (style, color) {
        return [style, { tintColor: color || app.setting.THEME_COLOR }];
    },
};

```

* PDClient/project/App/config/Constants.js

```js
'use strict';

// 该文件永远不要提交svn
// 发布正式服务器的配置，svn上面永远不要动这个配置，在发布测试服务器的时候将ISSUE改为false
// 如果发布ios时 ISSUE_IOS=true 其他的发布为 ISSUE_IOS=false
// CHANNEL为android渠道，发布百度市场时为baidu,其他的为default,ios忽略这个选项
const CONFIG = {
    ISSUE: false,
    ISSUE_IOS: false,
    CHANNEL: 'default',
};

// 发布测试服务器的配置，该配置只有 CONFIG.ISSUE 为 false 的时候才生效
// 如果是本地开发模式， CONFIG.ISSUE = false， TEST_CONFIG.ISSUE = false, 同时可以修改调试的服务器
// 如果是发布测试服务器， CONFIG.ISSUE = false，TEST_CONFIG.ISSUE = true
const TEST_CONFIG = {
    ISSUE: true,
    BASE_SERVER_INDEX: 0, // 只有 TEST_CONFIG.ISSUE 为 false时生效
};

// web服务器 依次是本地服务器， 测试服务器， 正式服务器
const BASE_SERVERS = ['192.168.1.222:3000', '192.168.1.189', '120.78.65.221'];
const BASE_SERVER = CONFIG.ISSUE ? BASE_SERVERS[2] : TEST_CONFIG.ISSUE ? BASE_SERVERS[1] : BASE_SERVERS[TEST_CONFIG.BASE_SERVER_INDEX];

module.exports = {
    APP_NAME: '四面通客户端',
    THEME_COLORS: ['#c81622', '#A62045', '#239FDB'],
    ISSUE_IOS: CONFIG.ISSUE_IOS,
    NOT_NEED_UPDATE_JS_START: !(CONFIG.ISSUE || TEST_CONFIG.ISSUE), // 启动时不需要更新小版本
    MINIFY: CONFIG.ISSUE, // 是否压缩js文件，我们采取测试服务器为了查找问题不用压缩js文件，正式服务器需要压缩js文件，并且不能看到调试信息
    CHANNEL: CONFIG.CHANNEL,
    // IOS的appid,1290180200
    IOS_APPID: '',
    BASE_SERVER: BASE_SERVER, // web服务器地址
    // 获取验证码的超时时间
    DEFAULT_CODE_TIMEOUT: 90,
    // 分页列表每页数据的条数
    PER_PAGE_COUNT: 10,
    NAME_MAX_LENGTH: 10,
    ADDRESS_MAX_LENGTH: 30,
    VERIFICATION_MAX_LENGTH: 6,
    AMOUNT_MAX_LENGTH: 7,
    FEEDBACK_MAX_LENGTH: 500,
    QRCODE_TYPE_ORDER:1,
};

```

* PDClient/project/App/config/OrderState.js

```js
// 注意：常量部分必须和服务器保持一致
// 订单的状态
/*
===========
长途运输状态图:
(0->)1->2->5    |
                |->6->7->8(->11)->14->15
(0->)1->3->4->5 |
===========
收货点状态图:
0->2->9    |
           |(->10)->11->14->15
0->3->4->9 |
===========
同城配送状态图: (当同城配送的在运输中，那么它之前的货单就在待交货状态)
(0->)1->2    |
             |->6->8->15
(0->)1->3->4 |
===========
上门自提状态图:(当上门自提的在运输中或者等待上门自提，那么它之前的货单就在待交货状态)
      |->13->15
1->12 |
      |->6->8->15
*/
const OS_PREORDER = -1; // 预下单状态，需要将货拉到分店后不显示（件数，重量，方量），由收货员填写确认后进入待选线路状态
const OS_READY_PRINT_BAR_CODE = 0; // 待打印二维码
const OS_READY_SELECT_ROADMAP = 1; // 待选物流公司
const OS_READY_DEDUCT_AND_PRINT_ORDER = 2; // 待扣款并打印打印货单（用来区别需要付款的情况）
const OS_READY_PAYMENT = 3; // 待付款
const OS_READY_PRINT_ORDER = 4; // 待打印货单
const OS_READY_STOCK = 5; // 待库存
const OS_READY_LOAD = 6; // 待装车
const OS_READY_START_OFF = 7; // 待出发
const OS_ON_THE_WAY = 8; // 运输中
const OS_READY_SEND = 9; // 待发货 (收货点专用)
const OS_ON_SENDING = 10; // 发货中 (收货点专用)
const OS_ON_TRANSFER = 11; // 中转中
const OS_READY_PHONE_CONFIRM = 12; // 待打电话确认
const OS_WAIT_CLIENT_PICK = 13; // 等待上门自提
const OS_READY_RECEIVE = 14; // 待交货
const OS_RECEIVE_SUCCESS = 15; // 交货成功

module.exports =  {
    OS_PREORDER,
    OS_READY_PRINT_BAR_CODE,
    OS_READY_SELECT_ROADMAP,
    OS_READY_DEDUCT_AND_PRINT_ORDER,
    OS_READY_PAYMENT,
    OS_READY_PRINT_ORDER,
    OS_READY_STOCK,
    OS_READY_LOAD,
    OS_READY_START_OFF,
    OS_ON_THE_WAY,
    OS_READY_SEND,
    OS_ON_SENDING,
    OS_ON_TRANSFER,
    OS_READY_PHONE_CONFIRM,
    OS_WAIT_CLIENT_PICK,
    OS_READY_RECEIVE,
    OS_RECEIVE_SUCCESS,

    // mapper
    MAP: {
        [OS_PREORDER]: '预下单状态',
        [OS_READY_PRINT_BAR_CODE]: '待打印二维码',
        [OS_READY_SELECT_ROADMAP]: '待选线路',
        [OS_READY_DEDUCT_AND_PRINT_ORDER]: '待打印收据',
        [OS_READY_PAYMENT]: '待付款',
        [OS_READY_PRINT_ORDER]: '待打印收据',
        [OS_READY_STOCK]: '待库存',
        [OS_READY_LOAD]: '待装车',
        [OS_READY_START_OFF]: '待出发',
        [OS_ON_THE_WAY]: '运输中',
        [OS_READY_SEND]: '待发货',
        [OS_ON_SENDING]: '发货中',
        [OS_ON_TRANSFER]: '运输中',
        [OS_READY_PHONE_CONFIRM]: '待打电话确认',
        [OS_WAIT_CLIENT_PICK]: '等待上门自提',
        [OS_READY_RECEIVE]: '待交货',
        [OS_RECEIVE_SUCCESS]: '交货成功',
    },
};

```

* PDClient/project/App/config/Partment.js

```js
'use strict';
// 注意：常量部分必须和服务器保持一致
const SP_RECEIVE_PARTMENT = 0; // 收货部
const SP_WARE_HOUSE_PARTMENT = 1; // 库管部
const SP_CARRY_PARTMENT = 2; // 搬运部
const SP_SECURITY_CHECK_PARTMENT = 3; // 保安部
const SP_DISTRIBUTION_PARTMENT = 4; // 同城配送部

module.exports = {
    SP_RECEIVE_PARTMENT,
    SP_WARE_HOUSE_PARTMENT,
    SP_CARRY_PARTMENT,
    SP_SECURITY_CHECK_PARTMENT,
    SP_DISTRIBUTION_PARTMENT,

    // mapper
    MAP: {
        [SP_RECEIVE_PARTMENT]: '收货部',
        [SP_WARE_HOUSE_PARTMENT]: '库管部',
        [SP_CARRY_PARTMENT]: '搬运部',
        [SP_SECURITY_CHECK_PARTMENT]: '保安部',
        [SP_DISTRIBUTION_PARTMENT]: '同城配送部',
    },
};

```

* PDClient/project/App/config/Position.js

```js
'use strict';
const PO_CHAIR_MAN = '董事长';
const PO_PARTMENT_CHARGE_MAN = '部门经理';
const PO_CEO = '总经理';
const PO_PARTMENT_LEADER = '部门主管';
const PO_STAFF = '员工';

module.exports = {
    PO_CHAIR_MAN,
    PO_PARTMENT_CHARGE_MAN,
    PO_CEO,
    PO_PARTMENT_LEADER,
    PO_STAFF,

    // 公司职位
    SHOP_MAP: [PO_STAFF, PO_CEO, PO_PARTMENT_LEADER],
    // 部门职位
    PARTMENT_MAP: [PO_STAFF],
};

```

* PDClient/project/App/config/Route.js

```js
'use strict';

const { BASE_SERVER } = CONSTANTS;
const ROOT_SERVER = 'http://' + BASE_SERVER + '/';
const API_SERVER = ROOT_SERVER + 'api/';
const SERVER = API_SERVER + 'client/';
const SHIPPER_SERVER = API_SERVER + 'shipper/';
const AGENT_SERVER = API_SERVER + 'agent/';

module.exports = {
    // 登录
    ROUTE_REGISTER: SERVER + 'register', // 注册
    ROUTE_LOGIN: SERVER + 'login', // 登录
    ROUTE_FIND_PASSWORD: SERVER + 'findPassword', // 忘记密码
    ROUTE_MODIFY_PASSWORD: SERVER + 'modifyPassword', // 修改密码
    ROUTE_REQUEST_SEND_VERIFY_CODE: API_SERVER + 'requestSendVerifyCode', // 获取验证码

    // 个人中心
    ROUTE_GET_PERSONAL_INFO: SERVER + 'getPersonalInfo', // 获取个人信息
    ROUTE_MODIFY_PERSONAL_INFO: SERVER + 'modifyPersonalInfo', // 修改个人信息
    ROUTE_SUBMIT_FEEDBACK: SERVER + 'submitFeedback', // 提交信息反馈
    ROUTE_GET_REGION_ADDRESS_LIST: API_SERVER + 'getRegionAddress', // 获取地址列表
    ROUTE_GET_ADDRESS_FROM_LAST_CODE: API_SERVER + 'getRegionAddressFromLastCode', // 通过lastCode获取具体的地址列表
    ROUTE_GET_START_POINT_ADDRESS_LIST: API_SERVER + 'getStartPointAddress', // 获取收货点地址列表
    ROUTE_GET_START_POINT_ADDRESS_FROME_LAST_CODE: API_SERVER + 'getStartPointAddressFromLastCode', // 通过lastCode获取收货点地址列表
    ROUTE_GET_SEND_DOOR_ADDRESS_FROM_LAST_CODE: API_SERVER + 'getRegionSendDoorAddressFromLastCode', // 通过lastCode获取短途地址列表
    ROUTE_GET_SHIPPERLIST_LIST: SERVER + 'shipperList', // 获取分店列表
    ROUTE_GET_BILL_LIST: SERVER + 'getBillList', // 获取交易明细列表
    ROUTE_RECHARGE:SERVER + 'recharge', // 充值
    ROUTE_WITHDRAW:SERVER + 'withdraw', // 提现
    ROUTE_GET_REMAIN_AMOUNT: SERVER + 'getRemainAmount', // 获取账户余额
    ROUTE_GET_UNIFIED_ORDER: SERVER + 'weixinPayGetUnifiedOrder', // 微信统一下单
    ROUTE_GET_UNION_UNIFIED_ORDER: SERVER + 'unionPayGetUnifiedOrder', // 银联统一下单
    ROUTE_SET_PAYMENT_PASSWORD: SERVER + 'setPaymentPassword', // 修改支付密码

    // 收货点
    ROUTE_PLACE_ORDER_AGENT: AGENT_SERVER + 'placeOrder', // 下单
    ROUTE_MODIFY_ORDER_AGENT: AGENT_SERVER + 'modifyOrder', // 修改货单
    ROUTE_GET_PRE_ORDER_LIST_AGENT: AGENT_SERVER + 'getPreOrderList', // 获取预下单列表
    ROUTE_PRINT_BAR_CODE: AGENT_SERVER + 'printBarCode', // 打印二维码
    ROUTE_PRINT_ORDER_BILL: AGENT_SERVER + 'printOrderBill', // 打印收据
    ROUTE_PRINT_ORDER_LIST_BILL: AGENT_SERVER + 'printOrderListBill', // 打印收据
    ROUTE_CONFIRM_CACH_PAYED: AGENT_SERVER + 'confirmCachPayed', // 为订单支付
    ROUTE_CONFIRM_CACH_PAYED_FOR_ORDER_LIST: AGENT_SERVER + 'confirmCachPayedForOrderList', // 为订单支付
    ROUTE_GET_LASTEST_ORDER: AGENT_SERVER + 'getLastestOrder', // 收货点获取未完成的货单
    ROUTE_GET_MEMBER_LIST_AGENT: AGENT_SERVER + 'getMemberList', // 收货点获取成员列表
    ROUTE_GET_MEMBER_BY_PHONE_AGENT: AGENT_SERVER + 'getMemberByPhone', // 收货点通过手机号获取成员信息
    ROUTE_MODIFY_MEMBER_AUTHORITY_AGENT: AGENT_SERVER + 'modifyMemberAuthority', // 收货点修改成员权限
    ROUTE_GET_ORDERS : AGENT_SERVER + 'getOrders', // 获取收货点货单列表
    ROUTE_GET_ORDER_DETAIL_AGENT : AGENT_SERVER + 'getOrderDetail', // 获取收货点货单详情
    ROUTE_GET_REGION_PROFIT_LIST : AGENT_SERVER + 'getRegionProfitList', // 收货点获取已设置路线列表
    ROUTE_REMOVE_REGION_PROFIT : AGENT_SERVER + 'removeRegionProfit', // 收货点删除路线提成
    ROUTE_ADD_REGION_PROFIT : AGENT_SERVER + 'addRegionProfit', // 收货点创建路线提成
    ROUTE_MODIFY_REGION_PROFIT : AGENT_SERVER + 'modifyRegionProfit', // 收货点修改路线提成
    ROUTE_SET_REGION_PROFIT_WITH_LIST : AGENT_SERVER + 'setRegionProfitWithList', // 收货点批量修改路线提成
    ROUTE_GET_REFER_SHOP_LIST: AGENT_SERVER + 'getReferShopList', // 收货点获取参考分店列表
    ROUTE_SET_REFER_SHOP: AGENT_SERVER + 'setReferShop', // 收货点设置参考分店
    ROUTE_PAY_AGENT_FOR_ORDER : SERVER + 'payForAgentOrder', // 客户为收货点货单付款

    // 物流公司
    ROUTE_GET_SHIPPER_LIST: SERVER + 'getShipperList', // 获取物流公司列表
    ROUTE_GET_SHIPPER_DETAIL: SERVER + 'getShipperDetail', // 获取物流公司详情
    ROUTE_CARE_SHIPPER: SERVER + 'careShipper', // 关注物流公司
    ROUTE_GET_CARED_SHIPPER_LIST: SERVER + 'getCaredShipperList', // 获取关注物流公司列表

    ROUTE_GET_NEED_SEND_ORDER_SUMMARY_LIST:SHIPPER_SERVER + 'getNeedSendOrderSummaryList', // 物流公司竟得货物
    ROUTE_GET_NEED_SEND_ORDER_LIST_BY_END_POINT:SHIPPER_SERVER + 'getNeedSendOrderListByEndPoint', // 物流公司竟得货物订单列表
    ROUTE_CREATE_TRUCK:SHIPPER_SERVER + 'createTruck', // 创建卡车
    ROUTE_GET_WORK_TRUCKLIST:SHIPPER_SERVER + 'getWorkTruckList', // 获取卡车列表
    ROUTE_GET_TRUCKS: SHIPPER_SERVER + 'getTrucks', // 获取已装货单卡车列表，用在物流公司货车页面
    ROUTE_GET_UNFINISH_RUCK: SHIPPER_SERVER + 'getUnfinishTruck', // 获取未装完货单卡车信息
    ROUTE_GET_ORDER_LIST_IN_RUCKS: SHIPPER_SERVER + 'getOrderListInTruck', // 获取货车中的装货单列表，用在物流公司货车页面
    ROUTE_GET_CARRY_PARTMENT_LIST: SHIPPER_SERVER + 'getCarryPartmentList', // 获取搬运队列表
    ROUTE_SCAN_LOAD_ORDER: SHIPPER_SERVER + 'scanLoadOrder', // 扫描货单
    ROUTE_SELECT_CARRY_PARTMENT: SHIPPER_SERVER + 'selectCarryPartment', // 选择搬运队
    ROUTE_START_SCAN_ORDER: SHIPPER_SERVER + 'startScanOrder', // 货车开始扫描
    ROUTE_CONTINUE_SCAN_ORDER: SHIPPER_SERVER + 'continueScanOrder', // 货车继续扫描
    ROUTE_FINISH_SCAN_ORDER: SHIPPER_SERVER + 'finishScanOrder', // 货车结束扫描
    ROUTE_GET_HISTORY_TRUCK_LIST: SHIPPER_SERVER + 'getHistoryTruckList', // 物流公司历史审核卡车信息
    ROUTE_PAY_FOR_CARRY_PARTMENT: SHIPPER_SERVER + 'payForCarryPartment', // 物流公司付搬运费
    ROUTE_GET_BRANCH_SHOP_INFO: SERVER + 'getBranchShopInfo', // 物流公司获取入驻分店详情包括担保公司
    ROUTE_GET_MEMBER_LIST: SHIPPER_SERVER + 'getMemberList', // 物流公司获取成员列表
    ROUTE_GET_MEMBER_BY_PHONE: SHIPPER_SERVER + 'getMemberByPhone', // 物流公司通过手机号获取成员信息
    ROUTE_MODIFY_MEMBER_AUTHORITY: SHIPPER_SERVER + 'modifyMemberAuthority', // 物流公司修改成员权限
    ROUTE_MODIFY_SHIPPER_INFO : SERVER + 'modifyShipperInfo',//修改物流公司信息

    // 路线
    ROUTE_GET_ROADMAP_LIST: SERVER + 'getRoadmapList', // 获取路线列表
    ROUTE_GET_ROADMAP_DETAIL: SERVER + 'getRoadmapDetail', // 获取路线详情
    ROUTE_GET_ROADMAP_LIST_WITH_END_POINT: SERVER + 'getRoadmapListWithEndPoint', // 获取路线行情列表
    ROUTE_GET_ROADMAP_LIST_WITH_PRE_ORDER: SERVER + 'getRoadmapListWithPreOrder', // 通过货单获取区域收货点和分店路线列表
    ROUTE_GET_ROADMAP_LIST_WITH_PRE_ORDER_GROUP: SERVER + 'getRoadmapListWithPreOrderGroup', // 通过货单组获取区域收货点和分店路线列表
    ROUTE_GET_LOGISTICS_LIST: SERVER + 'getLogisticsList', // 获取订单物流信息
    ROUTE_GET_ROADMAP_LIST_SHIPPER:SHIPPER_SERVER + 'getRoadmapList', // 获取路线列表
    ROUTE_PUBLISH_ROADMAP:SHIPPER_SERVER + 'publishRoadmap', // 发布路线
    ROUTE_REMOVE_ROADMAP:SHIPPER_SERVER + 'removeRoadmap', // 删除路线
    ROUTE_GET_SHIPPER_ROADMAP_DETAIL: SHIPPER_SERVER + 'getRoadmapDetail', // 物流公司获取路线详情
    ROUTE_MODIFY_ROADMAP: SHIPPER_SERVER + 'modifyRoadmap', // 物流公司修改路线
    ROUTE_SET_ROADMAP_PRICE: SHIPPER_SERVER + 'setRoadmapPrice', // 物流公司修改路线价格
    ROUTE_SET_ROADMAP_SEND_DOOR_ENABLE: SHIPPER_SERVER + 'setRoadmapSendDoorEnable', // 物流公司修改长途路线对应的送货上门路线状态
    ROUTE_SET_ROADMAP_ENABLE: SHIPPER_SERVER + 'setRoadmapEnable', // 物流公司修改路线状态(开通/停运)
    ROUTE_SET_ROADMAP_SEND_DOOR_PRICE: SHIPPER_SERVER + 'setRoadmapSendDoorPrice', // 物流公司修改路线对应送货上门路线价格
    ROUTE_SET_ROADMAP_SEND_DOOR_BASE_PRICE: SHIPPER_SERVER + 'setRoadmapSendDoorBasePrice', // 物流公司修改路线对应送货上门路线价格底价
    ROUTE_GET_ROADMAP_LIST_WITH_TRUCK: SERVER + 'getRoadmapListWithTruck', // 根据货车id和所选分店查询该货车货物中转价格

    // 用户
    ROUTE_GET_CLIENT_LIST: SERVER + 'getClientList', // 获取用户列表
    ROUTE_GET_DRIVER_INFO_BY_PHONE:SHIPPER_SERVER + 'getDriverInfoByPhone', // 通过手机号获取司机信息

    // 订单
    ROUTE_PLACE_PRE_ORDER: SERVER + 'placePreOrder', // 预下单
    ROUTE_GET_PRE_ORDER_LIST: SERVER + 'getPreOrderList', // 获取预下单列表
    ROUTE_CREATE_PRE_ORDER_GROUP:SERVER + 'createPreOrderGroup', // 合并预下单
    ROUTE_GET_PRE_ORDER_GROUP_LIST:SERVER + 'getPreOrderGroupList', // 获取预下单群组列表
    ROUTE_GET_PRE_ORDER_LIST_IN_GROUP:SERVER + 'getPreOrderListInGroup', // 获取某个群组中的预下单列表
    ROUTE_REMOVE_PRE_ORDER_LIST_FROM_GROUP:SERVER + 'removePreOrderFromGroup', // 删除某个群组中的预下单列表

    ROUTE_PLACE_ORDER: SERVER + 'placeOrder', // 下单
    ROUTE_GET_SEND_ORDER_LIST: SERVER + 'getSendOrders', // 获取发出订单列表
    ROUTE_GET_RECEIVE_ORDERS: SERVER + 'getReceiveOrders', // 获取收到订单列表
    ROUTE_GET_ORDER_LIST: SERVER + 'getOrderList', // 获取订单列表
    ROUTE_GET_ORDER_DETAIL: SERVER + 'getOrderDetail', // 获取订单详情
    ROUTE_MODIFY_ORDER: SERVER + 'modifyOrder', // 修改订单
    ROUTE_MODIFY_PRE_ORDER: SERVER + 'modifyPreOrder', // 修改预下单
    ROUTE_REMOVE_PRE_ORDER: SERVER + 'removePreOrder', // 删除订单
    ROUTE_PAY_FOR_ORDER_WHEN_SEND: SERVER + 'payForOrderWhenSend', // 线上立即支付(发货)
    ROUTE_PAY_FOR_ORDER_WHEN_RECEIVE: SERVER + 'payForOrderWhenReceive', // 线上立即支付（收货）
    ROUTE_FINISH_ORDER: SERVER + 'finishOrder', // 现付成功
    ROUTE_MODIFY_PRE_ORDER_GROUP: SERVER + 'modifyPreOrderGroup', // 修改预下单组群
    ROUTE_CONFIRM_HAND_OVER_ORDER: SHIPPER_SERVER + 'confirmHandOverOrder', // 确认货物到达，确认后收货人才能收货

    // 文件上传
    ROUTE_UPDATE_FILE: API_SERVER + 'uploadFile', // 上传文件

    // 网页地址
    ROUTE_USER_LICENSE: ROOT_SERVER + 'protocals/pdclient/user.html', // 用户协议
    ROUTE_SOFTWARE_LICENSE: ROOT_SERVER + 'protocals/pdclient/software.html', // 获取软件许可协议
    ROUTE_ABOUT_PAGE: ROOT_SERVER + 'protocals/pdclient/about.html', // 关于

    // 下载更新
    ROUTE_VERSION_INFO_URL: ROOT_SERVER + 'apps/pdclient/version.json', // 版本信息地址
    ROUTE_JS_ANDROID_URL: ROOT_SERVER + 'apps/pdclient/jsandroid.zip', // android jsbundle 包地址
    ROUTE_JS_IOS_URL: ROOT_SERVER + 'apps/pdclient/jsios.zip', // ios jsbundle 包地址
    ROUTE_APK_URL: ROOT_SERVER + 'apps/pdclient/pdclient.apk', // apk地址

    // 同城配送
    ROUTE_PLACE_STORAGE : SHIPPER_SERVER + 'placeStorage', // 短途物流公司扫描入库
    ROUTE_SET_CITY_TRUCK : SHIPPER_SERVER + 'setCityTruck', // 短途物流公司设置货车
    ROUTE_SCAN_LOAD_ORDER_FOR_CITY_DISTRIBUTE : SHIPPER_SERVER + 'scanLoadOrderForCityDistribute', //短途物流公司扫描装车
    ROUTE_GET_CITY_DISTRIBUTE_ORDERS : SHIPPER_SERVER + 'getCityDistributeOrders',// 短途物流公司获取同城配送货单
    ROUTE_SET_CLIENT_PICK_ORDER_TRANSPORT_FEE : SHIPPER_SERVER + 'setClientPickOrderTransportFee',//短途物流公司设置送货上门
    ROUTE_GET_CLIENT_PICK_SHIPPER_LIST : SHIPPER_SERVER + 'getClientPickShipperList',//获取分店设置了上门自提竞价的物流公司列表
};

```

* PDClient/project/App/config/Screen.js

```js
'use strict';

const ReactNative = require('react-native');
const {
    Dimensions,
    PixelRatio,
    Platform,
    NativeModules,
} = ReactNative;

const SCREEN_WIDTH_BASE = 375;
const { width, height } = Dimensions.get('screen') || Dimensions.get('window');
const pxielRatio = PixelRatio.get();
const { TotalNavHeight, NavBarHeight, StatusBarHeight } = Navigator.NavigationBar.Styles.General;
const translucent = NativeModules.SplashScreen.translucent;
const statusBarHeight = (Platform.OS === 'android' && !translucent) ? StatusBarHeight : 0;
const workHeight = height - statusBarHeight;

module.exports = {
    translucent: NativeModules.SplashScreen.translucent,
    tw: width, // 屏幕的真实宽度
    w: SCREEN_WIDTH_BASE, // 屏幕的宽度
    th: workHeight, // 屏幕的真实高度（android不包含状态栏）
    h: workHeight * SCREEN_WIDTH_BASE / width, // 屏幕的高度（android不包含状态栏）
    tfh: height, // 屏幕全屏的真实高度（android包含状态栏）
    fh: height * SCREEN_WIDTH_BASE / width, // 屏幕全屏的高度（android包含状态栏）
    tch: workHeight - TotalNavHeight, // 界面的真实高度
    ch: (workHeight - TotalNavHeight) * SCREEN_WIDTH_BASE / width, // 界面的高度
    trueStatusBarHeight: StatusBarHeight, // android状态栏真实高度
    statusBarHeight: StatusBarHeight * SCREEN_WIDTH_BASE / width, // android状态栏高度
    trueTotalNavHeight: NavBarHeight, // 导航栏的真实高度
    navbarHeight: NavBarHeight * SCREEN_WIDTH_BASE / width, // 导航栏的高度
    trueTotalNavbarHeight: TotalNavHeight, // 导航栏的真实总高度
    totalNavbarHeight: TotalNavHeight * SCREEN_WIDTH_BASE / width, // 导航栏的总高度
    pr: pxielRatio,
    s: (w) => { return w * width / SCREEN_WIDTH_BASE; },
    rs: (w) => { return w * SCREEN_WIDTH_BASE / width; },
};

```

* PDClient/project/App/config/TruckState.js

```js
'use strict';
// 注意：常量部分必须和服务器保持一致

const TS_READY_EXAMINE = 0; // 待审核
const TS_READY_ENTER_WAREHOUSE = 1; // 待入库
const TS_READY_SEARCH_CARRY_PARTMENT = 2; // 待找搬运队
const TS_READY_SCAN_LOAD = 3; // 待扫描装车
const TS_SCAN_LOADING = 4; // 装车中
const TS_READY_PAY_FOR_CARRY_PARTMENT = 5; // 待付搬运部的款
const TS_READY_EXIT_WAREHOUSE = 6; // 待出库
const TS_ON_THE_WAY = 7; // 运输中
const TS_RECEIVE_SUCCESS = 8; // 交货成功

module.exports = {
    TS_READY_EXAMINE,
    TS_READY_ENTER_WAREHOUSE,
    TS_READY_SEARCH_CARRY_PARTMENT,
    TS_READY_SCAN_LOAD,
    TS_SCAN_LOADING,
    TS_READY_PAY_FOR_CARRY_PARTMENT,
    TS_READY_EXIT_WAREHOUSE,
    TS_ON_THE_WAY,
    TS_RECEIVE_SUCCESS,

    // mapper
    MAP: {
        [TS_READY_EXAMINE]: '待审核',
        [TS_READY_ENTER_WAREHOUSE]: '待入库',
        [TS_READY_SEARCH_CARRY_PARTMENT]: '待找搬运队',
        [TS_READY_SCAN_LOAD]: '待扫描装车',
        [TS_SCAN_LOADING]: '装车中',
        [TS_READY_PAY_FOR_CARRY_PARTMENT]: '待付搬运部的款',
        [TS_READY_EXIT_WAREHOUSE]: '待出库',
        [TS_ON_THE_WAY]: '运输中',
        [TS_RECEIVE_SUCCESS]: '交货成功',
    },
};

```

* PDClient/project/App/data/city.js

```js
module.exports = [{ name:'北京', city:[{ name:'北京', area:['东城区', '西城区', '崇文区', '宣武区', '朝阳区', '丰台区', '石景山区', '海淀区', '门头沟区', '房山区', '通州区', '顺义区', '昌平区', '大兴区', '平谷区', '怀柔区', '密云县', '延庆县'] }] }, { name:'天津', city:[{ name:'天津', area:['和平区', '河东区', '河西区', '南开区', '河北区', '红桥区', '塘沽区', '汉沽区', '大港区', '东丽区', '西青区', '津南区', '北辰区', '武清区', '宝坻区', '宁河县', '静海县', '蓟  县'] }] }, { name:'河北', city:[{ name:'石家庄', area:['长安区', '桥东区', '桥西区', '新华区', '郊  区', '井陉矿区', '井陉县', '正定县', '栾城县', '行唐县', '灵寿县', '高邑县', '深泽县', '赞皇县', '无极县', '平山县', '元氏县', '赵  县', '辛集市', '藁', '晋州市', '新乐市', '鹿泉市'] }, { name:'唐山', area:['路南区', '路北区', '古冶区', '开平区', '新  区', '丰润县', '滦  县', '滦南县', '乐亭县', '迁西县', '玉田县', '唐海县', '遵化市', '丰南市', '迁安市'] }, { name:'秦皇岛', area:['海港区', '山海关区', '北戴河区', '青龙满族自治县', '昌黎县', '抚宁县', '卢龙县'] }, { name:'邯郸', area:['邯山区', '丛台区', '复兴区', '峰峰矿区', '邯郸县', '临漳县', '成安县', '大名县', '涉  县', '磁  县', '肥乡县', '永年县', '邱  县', '鸡泽县', '广平县', '馆陶县', '魏  县', '曲周县', '武安市'] }, { name:'邢台', area:['桥东区', '桥西区', '邢台县', '临城县', '内丘县', '柏乡县', '隆尧县', '任  县', '南和县', '宁晋县', '巨鹿县', '新河县', '广宗县', '平乡县', '威  县', '清河县', '临西县', '南宫市', '沙河市'] }, { name:'保定', area:['新市区', '北市区', '南市区', '满城县', '清苑县', '涞水县', '阜平县', '徐水县', '定兴县', '唐  县', '高阳县', '容城县', '涞源县', '望都县', '安新县', '易  县', '曲阳县', '蠡  县', '顺平县', '博野', '雄县', '涿州市', '定州市', '安国市', '高碑店市'] }, { name:'张家口', area:['桥东区', '桥西区', '宣化区', '下花园区', '宣化县', '张北县', '康保县', '沽源县', '尚义县', '蔚  县', '阳原县', '怀安县', '万全县', '怀来县', '涿鹿县', '赤城县', '崇礼县'] }, { name:'承德', area:['双桥区', '双滦区', '鹰手营子矿区', '承德县', '兴隆县', '平泉县', '滦平县', '隆化县', '丰宁满族自治县', '宽城满族自治县', '围场满族蒙古族自治县'] }, { name:'沧州', area:['新华区', '运河区', '沧  县', '青  县', '东光县', '海兴县', '盐山县', '肃宁县', '南皮县', '吴桥县', '献  县', '孟村回族自治县', '泊头市', '任丘市', '黄骅市', '河间市'] }, { name:'廊坊', area:['安次区', '固安县', '永清县', '香河县', '大城县', '文安县', '大厂回族自治县', '霸州市', '三河市'] }, { name:'衡水', area:['桃城区', '枣强县', '武邑县', '武强县', '饶阳县', '安平县', '故城县', '景  县', '阜城县', '冀州市', '深州市'] }] }, { name:'山西', city:[{ name:'太原', area:['小店区', '迎泽区', '杏花岭区', '尖草坪区', '万柏林区', '晋源区', '清徐县', '阳曲县', '娄烦县', '古交市'] }, { name:'大同', area:['城  区', '矿  区', '南郊区', '新荣区', '阳高县', '天镇县', '广灵县', '灵丘县', '浑源县', '左云县', '大同县'] }, { name:'阳泉', area:['城  区', '矿  区', '郊  区', '平定县', '盂  县'] }, { name:'长治', area:['城  区', '郊  区', '长治县', '襄垣县', '屯留县', '平顺县', '黎城县', '壶关县', '长子县', '武乡县', '沁  县', '沁源县', '潞城市'] }, { name:'晋城', area:['城  区', '沁水县', '阳城县', '陵川县', '泽州县', '高平市'] }, { name:'朔州', area:['朔城区', '平鲁区', '山阴县', '应  县', '右玉县', '怀仁县'] }, { name:'忻州', area:['忻府区', '原平市', '定襄县', '五台县', '代  县', '繁峙县', '宁武县', '静乐县', '神池县', '五寨县', '岢岚县', '河曲县', '保德县', '偏关县'] }, { name:'吕梁', area:['离石区', '孝义市', '汾阳市', '文水县', '交城县', '兴  县', '临  县', '柳林县', '石楼县', '岚  县', '方山县', '中阳县', '交口县'] }, { name:'晋中', area:['榆次市', '介休市', '榆社县', '左权县', '和顺县', '昔阳县', '寿阳县', '太谷县', '祁  县', '平遥县', '灵石县'] }, { name:'临汾', area:['临汾市', '侯马市', '霍州市', '曲沃县', '翼城县', '襄汾县', '洪洞县', '古  县', '安泽县', '浮山县', '吉  县', '乡宁县', '蒲  县', '大宁县', '永和县', '隰  县', '汾西县'] }, { name:'运城', area:['运城市', '永济市', '河津市', '芮城县', '临猗县', '万荣县', '新绛县', '稷山县', '闻喜县', '夏  县', '绛  县', '平陆县', '垣曲县'] }] }, { name:'内蒙古', city:[{ name:'呼和浩特', area:['新城区', '回民区', '玉泉区', '郊  区', '土默特左旗', '托克托县', '和林格尔县', '清水河县', '武川县'] }, { name:'包头', area:['东河区', '昆都伦区', '青山区', '石拐矿区', '白云矿区', '郊  区', '土默特右旗', '固阳县', '达尔罕茂明安联合旗'] }, { name:'乌海', area:['海勃湾区', '海南区', '乌达区'] }, { name:'赤峰', area:['红山区', '元宝山区', '松山区', '阿鲁科尔沁旗', '巴林左旗', '巴林右旗', '林西县', '克什克腾旗', '翁牛特旗', '喀喇沁旗', '宁城县', '敖汉旗'] }, { name:'呼伦贝尔', area:['海拉尔市', '满洲里市', '扎兰屯市', '牙克石市', '根河市', '额尔古纳市', '阿荣旗', '莫力达瓦达斡尔族自治旗', '鄂伦春自治旗', '鄂温克族自治旗', '新巴尔虎右旗', '新巴尔虎左旗', '陈巴尔虎旗'] }, { name:'兴安盟', area:['乌兰浩特市', '阿尔山市', '科尔沁右翼前旗', '科尔沁右翼中旗', '扎赉特旗', '突泉县'] }, { name:'通辽', area:['科尔沁区', '霍林郭勒市', '科尔沁左翼中旗', '科尔沁左翼后旗', '开鲁县', '库伦旗', '奈曼旗', '扎鲁特旗'] }, { name:'锡林郭勒盟', area:['二连浩特市', '锡林浩特市', '阿巴嘎旗', '苏尼特左旗', '苏尼特右旗', '东乌珠穆沁旗', '西乌珠穆沁旗', '太仆寺旗', '镶黄旗', '正镶白旗', '正蓝旗', '多伦县'] }, { name:'乌兰察布盟', area:['集宁市', '丰镇市', '卓资县', '化德县', '商都县', '兴和县', '凉城县', '察哈尔右翼前旗', '察哈尔右翼中旗', '察哈尔右翼后旗', '四子王旗'] }, { name:'伊克昭盟', area:['东胜市', '达拉特旗', '准格尔旗', '鄂托克前旗', '鄂托克旗', '杭锦旗', '乌审旗', '伊金霍洛旗'] }, { name:'巴彦淖尔盟', area:['临河市', '五原县', '磴口县', '乌拉特前旗', '乌拉特中旗', '乌拉特后旗', '杭锦后旗'] }, { name:'阿拉善盟', area:['阿拉善左旗', '阿拉善右旗', '额济纳旗'] }] }, { name:'辽宁', city:[{ name:'沈阳', area:['沈河区', '皇姑区', '和平区', '大东区', '铁西区', '苏家屯区', '东陵区', '于洪区', '新民市', '法库县', '辽中县', '康平县', '新城子区', '其他'] }, { name:'大连', area:['西岗区', '中山区', '沙河口区', '甘井子区', '旅顺口区', '金州区', '瓦房店市', '普兰店市', '庄河市', '长海县', '其他'] }, { name:'鞍山', area:['铁东区', '铁西区', '立山区', '千山区', '海城市', '台安县', '岫岩满族自治县', '其他'] }, { name:'抚顺', area:['顺城区', '新抚区', '东洲区', '望花区', '抚顺县', '清原满族自治县', '新宾满族自治县', '其他'] }, { name:'本溪', area:['平山区', '明山区', '溪湖区', '南芬区', '本溪满族自治县', '桓仁满族自治县', '其他'] }, { name:'丹东', area:['振兴区', '元宝区', '振安区', '东港市', '凤城市', '宽甸满族自治县', '其他'] }, { name:'锦州', area:['太和区', '古塔区', '凌河区', '凌海市', '黑山县', '义县', '北宁市', '其他'] }, { name:'营口', area:['站前区', '西市区', '鲅鱼圈区', '老边区', '大石桥市', '盖州市', '其他'] }, { name:'阜新', area:['海州区', '新邱区', '太平区', '清河门区', '细河区', '彰武县', '阜新蒙古族自治县', '其他'] }, { name:'辽阳', area:['白塔区', '文圣区', '宏伟区', '太子河区', '弓长岭区', '灯塔市', '辽阳县', '其他'] }, { name:'盘锦', area:['双台子区', '兴隆台区', '盘山县', '大洼县', '其他'] }, { name:'铁岭', area:['银州区', '清河区', '调兵山市', '开原市', '铁岭县', '昌图县', '西丰县', '其他'] }, { name:'朝阳', area:['双塔区', '龙城区', '凌源市', '北票市', '朝阳县', '建平县', '喀喇沁左翼蒙古族自治县', '其他'] }, { name:'葫芦岛', area:['龙港区', '南票区', '连山区', '兴城市', '绥中县', '建昌县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'吉林', city:[{ name:'长春', area:['朝阳区', '宽城区', '二道区', '南关区', '绿园区', '双阳区', '九台市', '榆树市', '德惠市', '农安县', '其他'] }, { name:'吉林', area:['船营区', '昌邑区', '龙潭区', '丰满区', '舒兰市', '桦甸市', '蛟河市', '磐石市', '永吉县', '其他'] }, { name:'四平', area:['铁西区', '铁东区', '公主岭市', '双辽市', '梨树县', '伊通满族自治县', '其他'] }, { name:'辽源', area:['龙山区', '西安区', '东辽县', '东丰县', '其他'] }, { name:'通化', area:['东昌区', '二道江区', '梅河口市', '集安市', '通化县', '辉南县', '柳河县', '其他'] }, { name:'白山', area:['八道江区', '江源区', '临江市', '靖宇县', '抚松县', '长白朝鲜族自治县', '其他'] }, { name:'松原', area:['宁江区', '乾安县', '长岭县', '扶余县', '前郭尔罗斯蒙古族自治县', '其他'] }, { name:'白城', area:['洮北区', '大安市', '洮南市', '镇赉县', '通榆县', '其他'] }, { name:'延边朝鲜族自治州', area:['延吉市', '图们市', '敦化市', '龙井市', '珲春市', '和龙市', '安图县', '汪清县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'黑龙江', city:[{ name:'哈尔滨', area:['松北区', '道里区', '南岗区', '平房区', '香坊区', '道外区', '呼兰区', '阿城区', '双城市', '尚志市', '五常市', '宾县', '方正县', '通河县', '巴彦县', '延寿县', '木兰县', '依兰县', '其他'] }, { name:'齐齐哈尔', area:['龙沙区', '昂昂溪区', '铁锋区', '建华区', '富拉尔基区', '碾子山区', '梅里斯达斡尔族区', '讷河市', '富裕县', '拜泉县', '甘南县', '依安县', '克山县', '泰来县', '克东县', '龙江县', '其他'] }, { name:'鹤岗', area:['兴山区', '工农区', '南山区', '兴安区', '向阳区', '东山区', '萝北县', '绥滨县', '其他'] }, { name:'双鸭山', area:['尖山区', '岭东区', '四方台区', '宝山区', '集贤县', '宝清县', '友谊县', '饶河县', '其他'] }, { name:'鸡西', area:['鸡冠区', '恒山区', '城子河区', '滴道区', '梨树区', '麻山区', '密山市', '虎林市', '鸡东县', '其他'] }, { name:'大庆', area:['萨尔图区', '红岗区', '龙凤区', '让胡路区', '大同区', '林甸县', '肇州县', '肇源县', '杜尔伯特蒙古族自治县', '其他'] }, { name:'伊春', area:['伊春区', '带岭区', '南岔区', '金山屯区', '西林区', '美溪区', '乌马河区', '翠峦区', '友好区', '上甘岭区', '五营区', '红星区', '新青区', '汤旺河区', '乌伊岭区', '铁力市', '嘉荫县', '其他'] }, { name:'牡丹江', area:['爱民区', '东安区', '阳明区', '西安区', '绥芬河市', '宁安市', '海林市', '穆棱市', '林口县', '东宁县', '其他'] }, { name:'佳木斯', area:['向阳区', '前进区', '东风区', '郊区', '同江市', '富锦市', '桦川县', '抚远县', '桦南县', '汤原县', '其他'] }, { name:'七台河', area:['桃山区', '新兴区', '茄子河区', '勃利县', '其他'] }, { name:'黑河', area:['爱辉区', '北安市', '五大连池市', '逊克县', '嫩江县', '孙吴县', '其他'] }, { name:'绥化', area:['北林区', '安达市', '肇东市', '海伦市', '绥棱县', '兰西县', '明水县', '青冈县', '庆安县', '望奎县', '其他'] }, { name:'大兴安岭地区', area:['呼玛县', '塔河县', '漠河县', '大兴安岭辖区', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'上海', city:[{ name:'上海', area:['黄浦区', '卢湾区', '徐汇区', '长宁区', '静安区', '普陀区', '闸北区', '虹口区', '杨浦区', '宝山区', '闵行区', '嘉定区', '松江区', '金山区', '青浦区', '南汇区', '奉贤区', '浦东新区', '崇明县', '其他'] }] }, { name:'江苏', city:[{ name:'南京', area:['玄武区', '白下区', '秦淮区', '建邺区', '鼓楼区', '下关区', '栖霞区', '雨花台区', '浦口区', '江宁区', '六合区', '溧水县', '高淳县', '其他'] }, { name:'苏州', area:['金阊区', '平江区', '沧浪区', '虎丘区', '吴中区', '相城区', '常熟市', '张家港市', '昆山市', '吴江市', '太仓市', '其他'] }, { name:'无锡', area:['崇安区', '南长区', '北塘区', '滨湖区', '锡山区', '惠山区', '江阴市', '宜兴市', '其他'] }, { name:'常州', area:['钟楼区', '天宁区', '戚墅堰区', '新北区', '武进区', '金坛市', '溧阳市', '其他'] }, { name:'镇江', area:['京口区', '润州区', '丹徒区', '丹阳市', '扬中市', '句容市', '其他'] }, { name:'南通', area:['崇川区', '港闸区', '通州市', '如皋市', '海门市', '启东市', '海安县', '如东县', '其他'] }, { name:'泰州', area:['海陵区', '高港区', '姜堰市', '泰兴市', '靖江市', '兴化市', '其他'] }, { name:'扬州', area:['广陵区', '维扬区', '邗江区', '江都市', '仪征市', '高邮市', '宝应县', '其他'] }, { name:'盐城', area:['亭湖区', '盐都区', '大丰市', '东台市', '建湖县', '射阳县', '阜宁县', '滨海县', '响水县', '其他'] }, { name:'连云港', area:['新浦区', '海州区', '连云区', '东海县', '灌云县', '赣榆县', '灌南县', '其他'] }, { name:'徐州', area:['云龙区', '鼓楼区', '九里区', '泉山区', '贾汪区', '邳州市', '新沂市', '铜山县', '睢宁县', '沛县', '丰县', '其他'] }, { name:'淮安', area:['清河区', '清浦区', '楚州区', '淮阴区', '涟水县', '洪泽县', '金湖县', '盱眙县', '其他'] }, { name:'宿迁', area:['宿城区', '宿豫区', '沭阳县', '泗阳县', '泗洪县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'浙江', city:[{ name:'杭州', area:['拱墅区', '西湖区', '上城区', '下城区', '江干区', '滨江区', '余杭区', '萧山区', '建德市', '富阳市', '临安市', '桐庐县', '淳安县', '其他'] }, { name:'宁波', area:['海曙区', '江东区', '江北区', '镇海区', '北仑区', '鄞州区', '余姚市', '慈溪市', '奉化市', '宁海县', '象山县', '其他'] }, { name:'温州', area:['鹿城区', '龙湾区', '瓯海区', '瑞安市', '乐清市', '永嘉县', '洞头县', '平阳县', '苍南县', '文成县', '泰顺县', '其他'] }, { name:'嘉兴', area:['秀城区', '秀洲区', '海宁市', '平湖市', '桐乡市', '嘉善县', '海盐县', '其他'] }, { name:'湖州', area:['吴兴区', '南浔区', '长兴县', '德清县', '安吉县', '其他'] }, { name:'绍兴', area:['越城区', '诸暨市', '上虞市', '嵊州市', '绍兴县', '新昌县', '其他'] }, { name:'金华', area:['婺城区', '金东区', '兰溪市', '义乌市', '东阳市', '永康市', '武义县', '浦江县', '磐安县', '其他'] }, { name:'衢州', area:['柯城区', '衢江区', '江山市', '龙游县', '常山县', '开化县', '其他'] }, { name:'舟山', area:['定海区', '普陀区', '岱山县', '嵊泗县', '其他'] }, { name:'台州', area:['椒江区', '黄岩区', '路桥区', '临海市', '温岭市', '玉环县', '天台县', '仙居县', '三门县', '其他'] }, { name:'丽水', area:['莲都区', '龙泉市', '缙云县', '青田县', '云和县', '遂昌县', '松阳县', '庆元县', '景宁畲族自治县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'安徽', city:[{ name:'合肥', area:['庐阳区', '瑶海区', '蜀山区', '包河区', '长丰县', '肥东县', '肥西县', '其他'] }, { name:'芜湖', area:['镜湖区', '弋江区', '鸠江区', '三山区', '芜湖县', '南陵县', '繁昌县', '其他'] }, { name:'蚌埠', area:['蚌山区', '龙子湖区', '禹会区', '淮上区', '怀远县', '固镇县', '五河县', '其他'] }, { name:'淮南', area:['田家庵区', '大通区', '谢家集区', '八公山区', '潘集区', '凤台县', '其他'] }, { name:'马鞍山', area:['雨山区', '花山区', '金家庄区', '当涂县', '其他'] }, { name:'淮北', area:['相山区', '杜集区', '烈山区', '濉溪县', '其他'] }, { name:'铜陵', area:['铜官山区', '狮子山区', '郊区', '铜陵县', '其他'] }, { name:'安庆', area:['迎江区', '大观区', '宜秀区', '桐城市', '宿松县', '枞阳县', '太湖县', '怀宁县', '岳西县', '望江县', '潜山县', '其他'] }, { name:'黄山', area:['屯溪区', '黄山区', '徽州区', '休宁县', '歙县', '祁门县', '黟县', '其他'] }, { name:'滁州', area:['琅琊区', '南谯区', '天长市', '明光市', '全椒县', '来安县', '定远县', '凤阳县', '其他'] }, { name:'阜阳', area:['颍州区', '颍东区', '颍泉区', '界首市', '临泉县', '颍上县', '阜南县', '太和县', '其他'] }, { name:'宿州', area:['埇桥区', '萧县', '泗县', '砀山县', '灵璧县', '其他'] }, { name:'巢湖', area:['居巢区', '含山县', '无为县', '庐江县', '和县', '其他'] }, { name:'六安', area:['金安区', '裕安区', '寿县', '霍山县', '霍邱县', '舒城县', '金寨县', '其他'] }, { name:'亳州', area:['谯城区', '利辛县', '涡阳县', '蒙城县', '其他'] }, { name:'池州', area:['贵池区', '东至县', '石台县', '青阳县', '其他'] }, { name:'宣城', area:['宣州区', '宁国市', '广德县', '郎溪县', '泾县', '旌德县', '绩溪县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'福建', city:[{ name:'福州', area:['鼓楼区', '台江区', '仓山区', '马尾区', '晋安区', '福清市', '长乐市', '闽侯县', '闽清县', '永泰县', '连江县', '罗源县', '平潭县', '其他'] }, { name:'厦门', area:['思明区', '海沧区', '湖里区', '集美区', '同安区', '翔安区', '其他'] }, { name:'莆田', area:['城厢区', '涵江区', '荔城区', '秀屿区', '仙游县', '其他'] }, { name:'三明', area:['梅列区', '三元区', '永安市', '明溪县', '将乐县', '大田县', '宁化县', '建宁县', '沙县', '尤溪县', '清流县', '泰宁县', '其他'] }, { name:'泉州', area:['鲤城区', '丰泽区', '洛江区', '泉港区', '石狮市', '晋江市', '南安市', '惠安县', '永春县', '安溪县', '德化县', '金门县', '其他'] }, { name:'漳州', area:['芗城区', '龙文区', '龙海市', '平和县', '南靖县', '诏安县', '漳浦县', '华安县', '东山县', '长泰县', '云霄县', '其他'] }, { name:'南平', area:['延平区', '建瓯市', '邵武市', '武夷山市', '建阳市', '松溪县', '光泽县', '顺昌县', '浦城县', '政和县', '其他'] }, { name:'龙岩', area:['新罗区', '漳平市', '长汀县', '武平县', '上杭县', '永定县', '连城县', '其他'] }, { name:'宁德', area:['蕉城区', '福安市', '福鼎市', '寿宁县', '霞浦县', '柘荣县', '屏南县', '古田县', '周宁县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'江西', city:[{ name:'南昌', area:['东湖区', '西湖区', '青云谱区', '湾里区', '青山湖区', '新建县', '南昌县', '进贤县', '安义县', '其他'] }, { name:'景德镇', area:['珠山区', '昌江区', '乐平市', '浮梁县', '其他'] }, { name:'萍乡', area:['安源区', '湘东区', '莲花县', '上栗县', '芦溪县', '其他'] }, { name:'九江', area:['浔阳区', '庐山区', '瑞昌市', '九江县', '星子县', '武宁县', '彭泽县', '永修县', '修水县', '湖口县', '德安县', '都昌县', '其他'] }, { name:'新余', area:['渝水区', '分宜县', '其他'] }, { name:'鹰潭', area:['月湖区', '贵溪市', '余江县', '其他'] }, { name:'赣州', area:['章贡区', '瑞金市', '南康市', '石城县', '安远县', '赣县', '宁都县', '寻乌县', '兴国县', '定南县', '上犹县', '于都县', '龙南县', '崇义县', '信丰县', '全南县', '大余县', '会昌县', '其他'] }, { name:'吉安', area:['吉州区', '青原区', '井冈山市', '吉安县', '永丰县', '永新县', '新干县', '泰和县', '峡江县', '遂川县', '安福县', '吉水县', '万安县', '其他'] }, { name:'宜春', area:['袁州区', '丰城市', '樟树市', '高安市', '铜鼓县', '靖安县', '宜丰县', '奉新县', '万载县', '上高县', '其他'] }, { name:'抚州', area:['临川区', '南丰县', '乐安县', '金溪县', '南城县', '东乡县', '资溪县', '宜黄县', '广昌县', '黎川县', '崇仁县', '其他'] }, { name:'上饶', area:['信州区', '德兴市', '上饶县', '广丰县', '鄱阳县', '婺源县', '铅山县', '余干县', '横峰县', '弋阳县', '玉山县', '万年县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'山东', city:[{ name:'济南', area:['市中区', '历下区', '天桥区', '槐荫区', '历城区', '长清区', '章丘市', '平阴县', '济阳县', '商河县', '其他'] }, { name:'青岛', area:['市南区', '市北区', '城阳区', '四方区', '李沧区', '黄岛区', '崂山区', '胶南市', '胶州市', '平度市', '莱西市', '即墨市', '其他'] }, { name:'淄博', area:['张店区', '临淄区', '淄川区', '博山区', '周村区', '桓台县', '高青县', '沂源县', '其他'] }, { name:'枣庄', area:['市中区', '山亭区', '峄城区', '台儿庄区', '薛城区', '滕州市', '其他'] }, { name:'东营', area:['东营区', '河口区', '垦利县', '广饶县', '利津县', '其他'] }, { name:'烟台', area:['芝罘区', '福山区', '牟平区', '莱山区', '龙口市', '莱阳市', '莱州市', '招远市', '蓬莱市', '栖霞市', '海阳市', '长岛县', '其他'] }, { name:'潍坊', area:['潍城区', '寒亭区', '坊子区', '奎文区', '青州市', '诸城市', '寿光市', '安丘市', '高密市', '昌邑市', '昌乐县', '临朐县', '其他'] }, { name:'济宁', area:['市中区', '任城区', '曲阜市', '兖州市', '邹城市', '鱼台县', '金乡县', '嘉祥县', '微山县', '汶上县', '泗水县', '梁山县', '其他'] }, { name:'泰安', area:['泰山区', '岱岳区', '新泰市', '肥城市', '宁阳县', '东平县', '其他'] }, { name:'威海', area:['环翠区', '乳山市', '文登市', '荣成市', '其他'] }, { name:'日照', area:['东港区', '岚山区', '五莲县', '莒县', '其他'] }, { name:'莱芜', area:['莱城区', '钢城区', '其他'] }, { name:'临沂', area:['兰山区', '罗庄区', '河东区', '沂南县', '郯城县', '沂水县', '苍山县', '费县', '平邑县', '莒南县', '蒙阴县', '临沭县', '其他'] }, { name:'德州', area:['德城区', '乐陵市', '禹城市', '陵县', '宁津县', '齐河县', '武城县', '庆云县', '平原县', '夏津县', '临邑县', '其他'] }, { name:'聊城', area:['东昌府区', '临清市', '高唐县', '阳谷县', '茌平县', '莘县', '东阿县', '冠县', '其他'] }, { name:'滨州', area:['滨城区', '邹平县', '沾化县', '惠民县', '博兴县', '阳信县', '无棣县', '其他'] }, { name:'菏泽', area:['牡丹区', '鄄城县', '单县', '郓城县', '曹县', '定陶县', '巨野县', '东明县', '成武县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'河南', city:[{ name:'郑州', area:['中原区', '金水区', '二七区', '管城回族区', '上街区', '惠济区', '巩义市', '新郑市', '新密市', '登封市', '荥阳市', '中牟县', '其他'] }, { name:'开封', area:['鼓楼区', '龙亭区', '顺河回族区', '禹王台区', '金明区', '开封县', '尉氏县', '兰考县', '杞县', '通许县', '其他'] }, { name:'洛阳', area:['西工区', '老城区', '涧西区', '瀍河回族区', '洛龙区', '吉利区', '偃师市', '孟津县', '汝阳县', '伊川县', '洛宁县', '嵩县', '宜阳县', '新安县', '栾川县', '其他'] }, { name:'平顶山', area:['新华区', '卫东区', '湛河区', '石龙区', '汝州市', '舞钢市', '宝丰县', '叶县', '郏县', '鲁山县', '其他'] }, { name:'安阳', area:['北关区', '文峰区', '殷都区', '龙安区', '林州市', '安阳县', '滑县', '内黄县', '汤阴县', '其他'] }, { name:'鹤壁', area:['淇滨区', '山城区', '鹤山区', '浚县', '淇县', '其他'] }, { name:'新乡', area:['卫滨区', '红旗区', '凤泉区', '牧野区', '卫辉市', '辉县市', '新乡县', '获嘉县', '原阳县', '长垣县', '封丘县', '延津县', '其他'] }, { name:'焦作', area:['解放区', '中站区', '马村区', '山阳区', '沁阳市', '孟州市', '修武县', '温县', '武陟县', '博爱县', '其他'] }, { name:'濮阳', area:['华龙区', '濮阳县', '南乐县', '台前县', '清丰县', '范县', '其他'] }, { name:'许昌', area:['魏都区', '禹州市', '长葛市', '许昌县', '鄢陵县', '襄城县', '其他'] }, { name:'漯河', area:['源汇区', '郾城区', '召陵区', '临颍县', '舞阳县', '其他'] }, { name:'三门峡', area:['湖滨区', '义马市', '灵宝市', '渑池县', '卢氏县', '陕县', '其他'] }, { name:'南阳', area:['卧龙区', '宛城区', '邓州市', '桐柏县', '方城县', '淅川县', '镇平县', '唐河县', '南召县', '内乡县', '新野县', '社旗县', '西峡县', '其他'] }, { name:'商丘', area:['梁园区', '睢阳区', '永城市', '宁陵县', '虞城县', '民权县', '夏邑县', '柘城县', '睢县', '其他'] }, { name:'信阳', area:['浉河区', '平桥区', '潢川县', '淮滨县', '息县', '新县', '商城县', '固始县', '罗山县', '光山县', '其他'] }, { name:'周口', area:['川汇区', '项城市', '商水县', '淮阳县', '太康县', '鹿邑县', '西华县', '扶沟县', '沈丘县', '郸城县', '其他'] }, { name:'驻马店', area:['驿城区', '确山县', '新蔡县', '上蔡县', '西平县', '泌阳县', '平舆县', '汝南县', '遂平县', '正阳县', '其他'] }, { name:'焦作', area:['济源市', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'湖北', city:[{ name:'武汉', area:['江岸区', '武昌区', '江汉区', '硚口区', '汉阳区', '青山区', '洪山区', '东西湖区', '汉南区', '蔡甸区', '江夏区', '黄陂区', '新洲区', '其他'] }, { name:'黄石', area:['黄石港区', '西塞山区', '下陆区', '铁山区', '大冶市', '阳新县', '其他'] }, { name:'十堰', area:['张湾区', '茅箭区', '丹江口市', '郧县', '竹山县', '房县', '郧西县', '竹溪县', '其他'] }, { name:'荆州', area:['沙市区', '荆州区', '洪湖市', '石首市', '松滋市', '监利县', '公安县', '江陵县', '其他'] }, { name:'宜昌', area:['西陵区', '伍家岗区', '点军区', '猇亭区', '夷陵区', '宜都市', '当阳市', '枝江市', '秭归县', '远安县', '兴山县', '五峰土家族自治县', '长阳土家族自治县', '其他'] }, { name:'襄樊', area:['襄城区', '樊城区', '襄阳区', '老河口市', '枣阳市', '宜城市', '南漳县', '谷城县', '保康县', '其他'] }, { name:'鄂州', area:['鄂城区', '华容区', '梁子湖区', '其他'] }, { name:'荆门', area:['东宝区', '掇刀区', '钟祥市', '京山县', '沙洋县', '其他'] }, { name:'孝感', area:['孝南区', '应城市', '安陆市', '汉川市', '云梦县', '大悟县', '孝昌县', '其他'] }, { name:'黄冈', area:['黄州区', '麻城市', '武穴市', '红安县', '罗田县', '浠水县', '蕲春县', '黄梅县', '英山县', '团风县', '其他'] }, { name:'咸宁', area:['咸安区', '赤壁市', '嘉鱼县', '通山县', '崇阳县', '通城县', '其他'] }, { name:'随州', area:['曾都区', '广水市', '其他'] }, { name:'恩施土家族苗族自治州', area:['恩施市', '利川市', '建始县', '来凤县', '巴东县', '鹤峰县', '宣恩县', '咸丰县', '其他'] }, { name:'仙桃', area:['仙桃'] }, { name:'天门', area:['天门'] }, { name:'潜江', area:['潜江'] }, { name:'神农架林区', area:['神农架林区'] }, { name:'其他', area:['其他'] }] }, { name:'湖南', city:[{ name:'长沙', area:['岳麓区', '芙蓉区', '天心区', '开福区', '雨花区', '浏阳市', '长沙县', '望城县', '宁乡县', '其他'] }, { name:'株洲', area:['天元区', '荷塘区', '芦淞区', '石峰区', '醴陵市', '株洲县', '炎陵县', '茶陵县', '攸县', '其他'] }, { name:'湘潭', area:['岳塘区', '雨湖区', '湘乡市', '韶山市', '湘潭县', '其他'] }, { name:'衡阳', area:['雁峰区', '珠晖区', '石鼓区', '蒸湘区', '南岳区', '耒阳市', '常宁市', '衡阳县', '衡东县', '衡山县', '衡南县', '祁东县', '其他'] }, { name:'邵阳', area:['双清区', '大祥区', '北塔区', '武冈市', '邵东县', '洞口县', '新邵县', '绥宁县', '新宁县', '邵阳县', '隆回县', '城步苗族自治县', '其他'] }, { name:'岳阳', area:['岳阳楼区', '云溪区', '君山区', '临湘市', '汨罗市', '岳阳县', '湘阴县', '平江县', '华容县', '其他'] }, { name:'常德', area:['武陵区', '鼎城区', '津市市', '澧县', '临澧县', '桃源县', '汉寿县', '安乡县', '石门县', '其他'] }, { name:'张家界', area:['永定区', '武陵源区', '慈利县', '桑植县', '其他'] }, { name:'益阳', area:['赫山区', '资阳区', '沅江市', '桃江县', '南县', '安化县', '其他'] }, { name:'郴州', area:['北湖区', '苏仙区', '资兴市', '宜章县', '汝城县', '安仁县', '嘉禾县', '临武县', '桂东县', '永兴县', '桂阳县', '其他'] }, { name:'永州', area:['冷水滩区', '零陵区', '祁阳县', '蓝山县', '宁远县', '新田县', '东安县', '江永县', '道县', '双牌县', '江华瑶族自治县', '其他'] }, { name:'怀化', area:['鹤城区', '洪江市', '会同县', '沅陵县', '辰溪县', '溆浦县', '中方县', '新晃侗族自治县', '芷江侗族自治县', '通道侗族自治县', '靖州苗族侗族自治县', '麻阳苗族自治县', '其他'] }, { name:'娄底', area:['娄星区', '冷水江市', '涟源市', '新化县', '双峰县', '其他'] }, { name:'湘西土家族苗族自治州', area:['吉首市', '古丈县', '龙山县', '永顺县', '凤凰县', '泸溪县', '保靖县', '花垣县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'广东', city:[{ name:'广州', area:['越秀区', '荔湾区', '海珠区', '天河区', '白云区', '黄埔区', '番禺区', '花都区', '南沙区', '萝岗区', '增城市', '从化市', '其他'] }, { name:'深圳', area:['福田区', '罗湖区', '南山区', '宝安区', '龙岗区', '盐田区', '其他'] }, { name:'东莞', area:['莞城', '常平', '塘厦', '塘厦', '塘厦', '其他'] }, { name:'中山', area:['中山'] }, { name:'潮州', area:['湘桥区', '潮安县', '饶平县', '其他'] }, { name:'揭阳', area:['榕城区', '揭东县', '揭西县', '惠来县', '普宁市', '其他'] }, { name:'云浮', area:['云城区', '新兴县', '郁南县', '云安县', '罗定市', '其他'] }, { name:'珠海', area:['香洲区', '斗门区', '金湾区', '其他'] }, { name:'汕头', area:['金平区', '濠江区', '龙湖区', '潮阳区', '潮南区', '澄海区', '南澳县', '其他'] }, { name:'韶关', area:['浈江区', '武江区', '曲江区', '乐昌市', '南雄市', '始兴县', '仁化县', '翁源县', '新丰县', '乳源瑶族自治县', '其他'] }, { name:'佛山', area:['禅城区', '南海区', '顺德区', '三水区', '高明区', '其他'] }, { name:'江门', area:['蓬江区', '江海区', '新会区', '恩平市', '台山市', '开平市', '鹤山市', '其他'] }, { name:'湛江', area:['赤坎区', '霞山区', '坡头区', '麻章区', '吴川市', '廉江市', '雷州市', '遂溪县', '徐闻县', '其他'] }, { name:'茂名', area:['茂南区', '茂港区', '化州市', '信宜市', '高州市', '电白县', '其他'] }, { name:'肇庆', area:['端州区', '鼎湖区', '高要市', '四会市', '广宁县', '怀集县', '封开县', '德庆县', '其他'] }, { name:'惠州', area:['惠城区', '惠阳区', '博罗县', '惠东县', '龙门县', '其他'] }, { name:'梅州', area:['梅江区', '兴宁市', '梅县', '大埔县', '丰顺县', '五华县', '平远县', '蕉岭县', '其他'] }, { name:'汕尾', area:['城区', '陆丰市', '海丰县', '陆河县', '其他'] }, { name:'河源', area:['源城区', '紫金县', '龙川县', '连平县', '和平县', '东源县', '其他'] }, { name:'阳江', area:['江城区', '阳春市', '阳西县', '阳东县', '其他'] }, { name:'清远', area:['清城区', '英德市', '连州市', '佛冈县', '阳山县', '清新县', '连山壮族瑶族自治县', '连南瑶族自治县', '其他'] }] }, { name:'广西', city:[{ name:'南宁', area:['青秀区', '兴宁区', '西乡塘区', '良庆区', '江南区', '邕宁区', '武鸣县', '隆安县', '马山县', '上林县', '宾阳县', '横县', '其他'] }, { name:'柳州', area:['城中区', '鱼峰区', '柳北区', '柳南区', '柳江县', '柳城县', '鹿寨县', '融安县', '融水苗族自治县', '三江侗族自治县', '其他'] }, { name:'桂林', area:['象山区', '秀峰区', '叠彩区', '七星区', '雁山区', '阳朔县', '临桂县', '灵川县', '全州县', '平乐县', '兴安县', '灌阳县', '荔浦县', '资源县', '永福县', '龙胜各族自治县', '恭城瑶族自治县', '其他'] }, { name:'梧州', area:['万秀区', '蝶山区', '长洲区', '岑溪市', '苍梧县', '藤县', '蒙山县', '其他'] }, { name:'北海', area:['海城区', '银海区', '铁山港区', '合浦县', '其他'] }, { name:'防城港', area:['港口区', '防城区', '东兴市', '上思县', '其他'] }, { name:'钦州', area:['钦南区', '钦北区', '灵山县', '浦北县', '其他'] }, { name:'贵港', area:['港北区', '港南区', '覃塘区', '桂平市', '平南县', '其他'] }, { name:'玉林', area:['玉州区', '北流市', '容县', '陆川县', '博白县', '兴业县', '其他'] }, { name:'百色', area:['右江区', '凌云县', '平果县', '西林县', '乐业县', '德保县', '田林县', '田阳县', '靖西县', '田东县', '那坡县', '隆林各族自治县', '其他'] }, { name:'贺州', area:['八步区', '钟山县', '昭平县', '富川瑶族自治县', '其他'] }, { name:'河池', area:['金城江区', '宜州市', '天峨县', '凤山县', '南丹县', '东兰县', '都安瑶族自治县', '罗城仫佬族自治县', '巴马瑶族自治县', '环江毛南族自治县', '大化瑶族自治县', '其他'] }, { name:'来宾', area:['兴宾区', '合山市', '象州县', '武宣县', '忻城县', '金秀瑶族自治县', '其他'] }, { name:'崇左', area:['江州区', '凭祥市', '宁明县', '扶绥县', '龙州县', '大新县', '天等县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'海南', city:[{ name:'海口', area:['龙华区', '秀英区', '琼山区', '美兰区', '其他'] }, { name:'三亚', area:['三亚市', '其他'] }, { name:'五指山', area:['五指山'] }, { name:'琼海', area:['琼海'] }, { name:'儋州', area:['儋州'] }, { name:'文昌', area:['文昌'] }, { name:'万宁', area:['万宁'] }, { name:'东方', area:['东方'] }, { name:'澄迈县', area:['澄迈县'] }, { name:'定安县', area:['定安县'] }, { name:'屯昌县', area:['屯昌县'] }, { name:'临高县', area:['临高县'] }, { name:'白沙黎族自治县', area:['白沙黎族自治县'] }, { name:'昌江黎族自治县', area:['昌江黎族自治县'] }, { name:'乐东黎族自治县', area:['乐东黎族自治县'] }, { name:'陵水黎族自治县', area:['陵水黎族自治县'] }, { name:'保亭黎族苗族自治县', area:['保亭黎族苗族自治县'] }, { name:'琼中黎族苗族自治县', area:['琼中黎族苗族自治县'] }, { name:'其他', area:['其他'] }] }, { name:'重庆', city:[{ name:'重庆', area:['渝中区', '大渡口区', '江北区', '南岸区', '北碚区', '渝北区', '巴南区', '长寿区', '双桥区', '沙坪坝区', '万盛区', '万州区', '涪陵区', '黔江区', '永川区', '合川区', '江津区', '九龙坡区', '南川区', '綦江县', '潼南县', '荣昌县', '璧山县', '大足县', '铜梁县', '梁平县', '开县', '忠县', '城口县', '垫江县', '武隆县', '丰都县', '奉节县', '云阳县', '巫溪县', '巫山县', '石柱土家族自治县', '秀山土家族苗族自治县', '酉阳土家族苗族自治县', '彭水苗族土家族自治县', '其他'] }] }, { name:'四川', city:[{ name:'成都', area:['青羊区', '锦江区', '金牛区', '武侯区', '成华区', '龙泉驿区', '青白江区', '新都区', '温江区', '都江堰市', '彭州市', '邛崃市', '崇州市', '金堂县', '郫县', '新津县', '双流县', '蒲江县', '大邑县', '其他'] }, { name:'自贡', area:['大安区', '自流井区', '贡井区', '沿滩区', '荣县', '富顺县', '其他'] }, { name:'攀枝花', area:['仁和区', '米易县', '盐边县', '东区', '西区', '其他'] }, { name:'泸州', area:['江阳区', '纳溪区', '龙马潭区', '泸县', '合江县', '叙永县', '古蔺县', '其他'] }, { name:'德阳', area:['旌阳区', '广汉市', '什邡市', '绵竹市', '罗江县', '中江县', '其他'] }, { name:'绵阳', area:['涪城区', '游仙区', '江油市', '盐亭县', '三台县', '平武县', '安县', '梓潼县', '北川羌族自治县', '其他'] }, { name:'广元', area:['元坝区', '朝天区', '青川县', '旺苍县', '剑阁县', '苍溪县', '市中区', '其他'] }, { name:'遂宁', area:['船山区', '安居区', '射洪县', '蓬溪县', '大英县', '其他'] }, { name:'内江', area:['市中区', '东兴区', '资中县', '隆昌县', '威远县', '其他'] }, { name:'乐山', area:['市中区', '五通桥区', '沙湾区', '金口河区', '峨眉山市', '夹江县', '井研县', '犍为县', '沐川县', '马边彝族自治县', '峨边彝族自治县', '其他'] }, { name:'南充', area:['顺庆区', '高坪区', '嘉陵区', '阆中市', '营山县', '蓬安县', '仪陇县', '南部县', '西充县', '其他'] }, { name:'眉山', area:['东坡区', '仁寿县', '彭山县', '洪雅县', '丹棱县', '青神县', '其他'] }, { name:'宜宾', area:['翠屏区', '宜宾县', '兴文县', '南溪县', '珙县', '长宁县', '高县', '江安县', '筠连县', '屏山县', '其他'] }, { name:'广安', area:['广安区', '华蓥市', '岳池县', '邻水县', '武胜县', '其他'] }, { name:'达州', area:['通川区', '万源市', '达县', '渠县', '宣汉县', '开江县', '大竹县', '其他'] }, { name:'雅安', area:['雨城区', '芦山县', '石棉县', '名山县', '天全县', '荥经县', '宝兴县', '汉源县', '其他'] }, { name:'巴中', area:['巴州区', '南江县', '平昌县', '通江县', '其他'] }, { name:'资阳', area:['雁江区', '简阳市', '安岳县', '乐至县', '其他'] }, { name:'阿坝藏族羌族自治州', area:['马尔康县', '九寨沟县', '红原县', '汶川县', '阿坝县', '理县', '若尔盖县', '小金县', '黑水县', '金川县', '松潘县', '壤塘县', '茂县', '其他'] }, { name:'甘孜藏族自治州', area:['康定县', '丹巴县', '炉霍县', '九龙县', '甘孜县', '雅江县', '新龙县', '道孚县', '白玉县', '理塘县', '德格县', '乡城县', '石渠县', '稻城县', '色达县', '巴塘县', '泸定县', '得荣县', '其他'] }, { name:'凉山彝族自治州', area:['西昌市', '美姑县', '昭觉县', '金阳县', '甘洛县', '布拖县', '雷波县', '普格县', '宁南县', '喜德县', '会东县', '越西县', '会理县', '盐源县', '德昌县', '冕宁县', '木里藏族自治县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'贵州', city:[{ name:'贵阳', area:['南明区', '云岩区', '花溪区', '乌当区', '白云区', '小河区', '清镇市', '开阳县', '修文县', '息烽县', '其他'] }, { name:'六盘水', area:['钟山区', '水城县', '盘县', '六枝特区', '其他'] }, { name:'遵义', area:['红花岗区', '汇川区', '赤水市', '仁怀市', '遵义县', '绥阳县', '桐梓县', '习水县', '凤冈县', '正安县', '余庆县', '湄潭县', '道真仡佬族苗族自治县', '务川仡佬族苗族自治县', '其他'] }, { name:'安顺', area:['西秀区', '普定县', '平坝县', '镇宁布依族苗族自治县', '紫云苗族布依族自治县', '关岭布依族苗族自治县', '其他'] }, { name:'铜仁地区', area:['铜仁市', '德江县', '江口县', '思南县', '石阡县', '玉屏侗族自治县', '松桃苗族自治县', '印江土家族苗族自治县', '沿河土家族自治县', '万山特区', '其他'] }, { name:'毕节地区', area:['毕节市', '黔西县', '大方县', '织金县', '金沙县', '赫章县', '纳雍县', '威宁彝族回族苗族自治县', '其他'] }, { name:'黔西南布依族苗族自治州', area:['兴义市', '望谟县', '兴仁县', '普安县', '册亨县', '晴隆县', '贞丰县', '安龙县', '其他'] }, { name:'黔东南苗族侗族自治州', area:['凯里市', '施秉县', '从江县', '锦屏县', '镇远县', '麻江县', '台江县', '天柱县', '黄平县', '榕江县', '剑河县', '三穗县', '雷山县', '黎平县', '岑巩县', '丹寨县', '其他'] }, { name:'黔南布依族苗族自治州', area:['都匀市', '福泉市', '贵定县', '惠水县', '罗甸县', '瓮安县', '荔波县', '龙里县', '平塘县', '长顺县', '独山县', '三都水族自治县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'云南', city:[{ name:'昆明', area:['盘龙区', '五华区', '官渡区', '西山区', '东川区', '安宁市', '呈贡县', '晋宁县', '富民县', '宜良县', '嵩明县', '石林彝族自治县', '禄劝彝族苗族自治县', '寻甸回族彝族自治县', '其他'] }, { name:'曲靖', area:['麒麟区', '宣威市', '马龙县', '沾益县', '富源县', '罗平县', '师宗县', '陆良县', '会泽县', '其他'] }, { name:'玉溪', area:['红塔区', '江川县', '澄江县', '通海县', '华宁县', '易门县', '峨山彝族自治县', '新平彝族傣族自治县', '元江哈尼族彝族傣族自治县', '其他'] }, { name:'保山', area:['隆阳区', '施甸县', '腾冲县', '龙陵县', '昌宁县', '其他'] }, { name:'昭通', area:['昭阳区', '鲁甸县', '巧家县', '盐津县', '大关县', '永善县', '绥江县', '镇雄县', '彝良县', '威信县', '水富县', '其他'] }, { name:'丽江', area:['古城区', '永胜县', '华坪县', '玉龙纳西族自治县', '宁蒗彝族自治县', '其他'] }, { name:'普洱', area:['思茅区', '普洱哈尼族彝族自治县', '墨江哈尼族自治县', '景东彝族自治县', '景谷傣族彝族自治县', '镇沅彝族哈尼族拉祜族自治县', '江城哈尼族彝族自治县', '孟连傣族拉祜族佤族自治县', '澜沧拉祜族自治县', '西盟佤族自治县', '其他'] }, { name:'临沧', area:['临翔区', '凤庆县', '云县', '永德县', '镇康县', '双江拉祜族佤族布朗族傣族自治县', '耿马傣族佤族自治县', '沧源佤族自治县', '其他'] }, { name:'德宏傣族景颇族自治州', area:['潞西市', '瑞丽市', '梁河县', '盈江县', '陇川县', '其他'] }, { name:'怒江傈僳族自治州', area:['泸水县', '福贡县', '贡山独龙族怒族自治县', '兰坪白族普米族自治县', '其他'] }, { name:'迪庆藏族自治州', area:['香格里拉县', '德钦县', '维西傈僳族自治县', '其他'] }, { name:'大理白族自治州', area:['大理市', '祥云县', '宾川县', '弥渡县', '永平县', '云龙县', '洱源县', '剑川县', '鹤庆县', '漾濞彝族自治县', '南涧彝族自治县', '巍山彝族回族自治县', '其他'] }, { name:'楚雄彝族自治州', area:['楚雄市', '双柏县', '牟定县', '南华县', '姚安县', '大姚县', '永仁县', '元谋县', '武定县', '禄丰县', '其他'] }, { name:'红河哈尼族彝族自治州', area:['蒙自县', '个旧市', '开远市', '绿春县', '建水县', '石屏县', '弥勒县', '泸西县', '元阳县', '红河县', '金平苗族瑶族傣族自治县', '河口瑶族自治县', '屏边苗族自治县', '其他'] }, { name:'文山壮族苗族自治州', area:['文山县', '砚山县', '西畴县', '麻栗坡县', '马关县', '丘北县', '广南县', '富宁县', '其他'] }, { name:'西双版纳傣族自治州', area:['景洪市', '勐海县', '勐腊县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'西藏', city:[{ name:'拉萨', area:['城关区', '林周县', '当雄县', '尼木县', '曲水县', '堆龙德庆县', '达孜县', '墨竹工卡县', '其他'] }, { name:'那曲地区', area:['那曲县', '嘉黎县', '比如县', '聂荣县', '安多县', '申扎县', '索县', '班戈县', '巴青县', '尼玛县', '其他'] }, { name:'昌都地区', area:['昌都县', '江达县', '贡觉县', '类乌齐县', '丁青县', '察雅县', '八宿县', '左贡县', '芒康县', '洛隆县', '边坝县', '其他'] }, { name:'林芝地区', area:['林芝县', '工布江达县', '米林县', '墨脱县', '波密县', '察隅县', '朗县', '其他'] }, { name:'山南地区', area:['乃东县', '扎囊县', '贡嘎县', '桑日县', '琼结县', '曲松县', '措美县', '洛扎县', '加查县', '隆子县', '错那县', '浪卡子县', '其他'] }, { name:'日喀则地区', area:['日喀则市', '南木林县', '江孜县', '定日县', '萨迦县', '拉孜县', '昂仁县', '谢通门县', '白朗县', '仁布县', '康马县', '定结县', '仲巴县', '亚东县', '吉隆县', '聂拉木县', '萨嘎县', '岗巴县', '其他'] }, { name:'阿里地区', area:['噶尔县', '普兰县', '札达县', '日土县', '革吉县', '改则县', '措勤县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'陕西', city:[{ name:'西安', area:['莲湖区', '新城区', '碑林区', '雁塔区', '灞桥区', '未央区', '阎良区', '临潼区', '长安区', '高陵县', '蓝田县', '户县', '周至县', '其他'] }, { name:'铜川', area:['耀州区', '王益区', '印台区', '宜君县', '其他'] }, { name:'宝鸡', area:['渭滨区', '金台区', '陈仓区', '岐山县', '凤翔县', '陇县', '太白县', '麟游县', '扶风县', '千阳县', '眉县', '凤县', '其他'] }, { name:'咸阳', area:['秦都区', '渭城区', '杨陵区', '兴平市', '礼泉县', '泾阳县', '永寿县', '三原县', '彬县', '旬邑县', '长武县', '乾县', '武功县', '淳化县', '其他'] }, { name:'渭南', area:['临渭区', '韩城市', '华阴市', '蒲城县', '潼关县', '白水县', '澄城县', '华县', '合阳县', '富平县', '大荔县', '其他'] }, { name:'延安', area:['宝塔区', '安塞县', '洛川县', '子长县', '黄陵县', '延川县', '富县', '延长县', '甘泉县', '宜川县', '志丹县', '黄龙县', '吴起县', '其他'] }, { name:'汉中', area:['汉台区', '留坝县', '镇巴县', '城固县', '南郑县', '洋县', '宁强县', '佛坪县', '勉县', '西乡县', '略阳县', '其他'] }, { name:'榆林', area:['榆阳区', '清涧县', '绥德县', '神木县', '佳县', '府谷县', '子洲县', '靖边县', '横山县', '米脂县', '吴堡县', '定边县', '其他'] }, { name:'安康', area:['汉滨区', '紫阳县', '岚皋县', '旬阳县', '镇坪县', '平利县', '石泉县', '宁陕县', '白河县', '汉阴县', '其他'] }, { name:'商洛', area:['商州区', '镇安县', '山阳县', '洛南县', '商南县', '丹凤县', '柞水县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'甘肃', city:[{ name:'兰州', area:['城关区', '七里河区', '西固区', '安宁区', '红古区', '永登县', '皋兰县', '榆中县', '其他'] }, { name:'嘉峪关', area:['嘉峪关市', '其他'] }, { name:'金昌', area:['金川区', '永昌县', '其他'] }, { name:'白银', area:['白银区', '平川区', '靖远县', '会宁县', '景泰县', '其他'] }, { name:'天水', area:['清水县', '秦安县', '甘谷县', '武山县', '张家川回族自治县', '北道区', '秦城区', '其他'] }, { name:'武威', area:['凉州区', '民勤县', '古浪县', '天祝藏族自治县', '其他'] }, { name:'酒泉', area:['肃州区', '玉门市', '敦煌市', '金塔县', '肃北蒙古族自治县', '阿克塞哈萨克族自治县', '安西县', '其他'] }, { name:'张掖', area:['甘州区', '民乐县', '临泽县', '高台县', '山丹县', '肃南裕固族自治县', '其他'] }, { name:'庆阳', area:['西峰区', '庆城县', '环县', '华池县', '合水县', '正宁县', '宁县', '镇原县', '其他'] }, { name:'平凉', area:['崆峒区', '泾川县', '灵台县', '崇信县', '华亭县', '庄浪县', '静宁县', '其他'] }, { name:'定西', area:['安定区', '通渭县', '临洮县', '漳县', '岷县', '渭源县', '陇西县', '其他'] }, { name:'陇南', area:['武都区', '成县', '宕昌县', '康县', '文县', '西和县', '礼县', '两当县', '徽县', '其他'] }, { name:'临夏回族自治州', area:['临夏市', '临夏县', '康乐县', '永靖县', '广河县', '和政县', '东乡族自治县', '积石山保安族东乡族撒拉族自治县', '其他'] }, { name:'甘南藏族自治州', area:['合作市', '临潭县', '卓尼县', '舟曲县', '迭部县', '玛曲县', '碌曲县', '夏河县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'青海', city:[{ name:'西宁', area:['城中区', '城东区', '城西区', '城北区', '湟源县', '湟中县', '大通回族土族自治县', '其他'] }, { name:'海东地区', area:['平安县', '乐都县', '民和回族土族自治县', '互助土族自治县', '化隆回族自治县', '循化撒拉族自治县', '其他'] }, { name:'海北藏族自治州', area:['海晏县', '祁连县', '刚察县', '门源回族自治县', '其他'] }, { name:'海南藏族自治州', area:['共和县', '同德县', '贵德县', '兴海县', '贵南县', '其他'] }, { name:'黄南藏族自治州', area:['同仁县', '尖扎县', '泽库县', '河南蒙古族自治县', '其他'] }, { name:'果洛藏族自治州', area:['玛沁县', '班玛县', '甘德县', '达日县', '久治县', '玛多县', '其他'] }, { name:'玉树藏族自治州', area:['玉树县', '杂多县', '称多县', '治多县', '囊谦县', '曲麻莱县', '其他'] }, { name:'海西蒙古族藏族自治州', area:['德令哈市', '格尔木市', '乌兰县', '都兰县', '天峻县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'宁夏', city:[{ name:'银川', area:['兴庆区', '西夏区', '金凤区', '灵武市', '永宁县', '贺兰县', '其他'] }, { name:'石嘴山', area:['大武口区', '惠农区', '平罗县', '其他'] }, { name:'吴忠', area:['利通区', '青铜峡市', '盐池县', '同心县', '其他'] }, { name:'固原', area:['原州区', '西吉县', '隆德县', '泾源县', '彭阳县', '其他'] }, { name:'中卫', area:['沙坡头区', '中宁县', '海原县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'新疆', city:[{ name:'乌鲁木齐', area:['天山区', '沙依巴克区', '新市区', '水磨沟区', '头屯河区', '达坂城区', '东山区', '乌鲁木齐县', '其他'] }, { name:'克拉玛依', area:['克拉玛依区', '独山子区', '白碱滩区', '乌尔禾区', '其他'] }, { name:'吐鲁番地区', area:['吐鲁番市', '托克逊县', '鄯善县', '其他'] }, { name:'哈密地区', area:['哈密市', '伊吾县', '巴里坤哈萨克自治县', '其他'] }, { name:'和田地区', area:['和田市', '和田县', '洛浦县', '民丰县', '皮山县', '策勒县', '于田县', '墨玉县', '其他'] }, { name:'阿克苏地区', area:['阿克苏市', '温宿县', '沙雅县', '拜城县', '阿瓦提县', '库车县', '柯坪县', '新和县', '乌什县', '其他'] }, { name:'喀什地区', area:['喀什市', '巴楚县', '泽普县', '伽师县', '叶城县', '岳普湖县', '疏勒县', '麦盖提县', '英吉沙县', '莎车县', '疏附县', '塔什库尔干塔吉克自治县', '其他'] }, { name:'克孜勒苏柯尔克孜自治州', area:['阿图什市', '阿合奇县', '乌恰县', '阿克陶县', '其他'] }, { name:'巴音郭楞蒙古自治州', area:['库尔勒市', '和静县', '尉犁县', '和硕县', '且末县', '博湖县', '轮台县', '若羌县', '焉耆回族自治县', '其他'] }, { name:'昌吉回族自治州', area:['昌吉市', '阜康市', '奇台县', '玛纳斯县', '吉木萨尔县', '呼图壁县', '木垒哈萨克自治县', '米泉市', '其他'] }, { name:'博尔塔拉蒙古自治州', area:['博乐市', '精河县', '温泉县', '其他'] }, { name:'石河子', area:['石河子'] }, { name:'阿拉尔', area:['阿拉尔'] }, { name:'图木舒克', area:['图木舒克'] }, { name:'五家渠', area:['五家渠'] }, { name:'伊犁哈萨克自治州', area:['伊宁市', '奎屯市', '伊宁县', '特克斯县', '尼勒克县', '昭苏县', '新源县', '霍城县', '巩留县', '察布查尔锡伯自治县', '塔城地区', '阿勒泰地区', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'台湾', city:[{ name:'台湾', area:['台北市', '高雄市', '台北县', '桃园县', '新竹县', '苗栗县', '台中县', '彰化县', '南投县', '云林县', '嘉义县', '台南县', '高雄县', '屏东县', '宜兰县', '花莲县', '台东县', '澎湖县', '基隆市', '新竹市', '台中市', '嘉义市', '台南市', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'澳门', city:[{ name:'澳门', area:['花地玛堂区', '圣安多尼堂区', '大堂区', '望德堂区', '风顺堂区', '嘉模堂区', '圣方济各堂区', '路凼', '其他'] }] }, { name:'香港', city:[{ name:'香港', area:['中西区', '湾仔区', '东区', '南区', '深水埗区', '油尖旺区', '九龙城区', '黄大仙区', '观塘区', '北区', '大埔区', '沙田区', '西贡区', '元朗区', '屯门区', '荃湾区', '葵青区', '离岛区', '其他'] }] }, { name:'钓鱼岛', city:[{ name:'钓鱼岛', area:['钓鱼岛'] }] }];

```

* PDClient/project/App/manager/AuthorityMgr.js

```js
'use strict';
const ReactNative = require('react-native');
const {
    AsyncStorage,
} = ReactNative;
const EventEmitter = require('EventEmitter');
const ITEM_NAME = 'authorityInfo';

class Manager extends EventEmitter {
    constructor () {
        super();
        this.get();
    }
    get () {
        return new Promise(async(resolve, reject) => {
            let info;
            try {
                const infoStr = await AsyncStorage.getItem(ITEM_NAME);
                info = JSON.parse(infoStr);
            } catch (e) {
            }
            this.info = info || {};
        });
    }
    set (info) {
        this.info = info;
        return new Promise(async(resolve, reject) => {
            await AsyncStorage.setItem(ITEM_NAME, JSON.stringify(info));
            resolve();
        });
    }
    getCanSelectRoles () {
        return _.reject(this.info.roleList, (o) => { return o == this.info.currentRole; });
    }
    changeRole (role) {
        this.info.currentRole = role;
        return new Promise(async(resolve, reject) => {
            await AsyncStorage.setItem(ITEM_NAME, JSON.stringify(this.info));
            resolve();
        });
    }
    clear () {
        this.info = {};
        AsyncStorage.removeItem(ITEM_NAME);
    }
}

module.exports = new Manager();

```

* PDClient/project/App/manager/JpushMgr.js

```js
'use strict';
const ReactNative = require('react-native');
const {
    NativeAppEventEmitter,
    Platform,
} = ReactNative;
const EventEmitter = require('EventEmitter');
import JPushModule from 'jpush-react-native';
class Manager extends EventEmitter {
    register () {
        if (Platform.OS === 'android') {
            JPushModule.notifyJSDidLoad((resultCode) => {
                if (resultCode === 0) {
                    console.log('--------------notifyJSDidLoad success');
                }
            });
            JPushModule.addReceiveNotificationListener((map) => {
                console.log('------------------addReceiveNotificationListener' + JSON.stringify(map));
                this.emit('receiveJpushNotification', { context:map.alertContent });
                   // var extra = JSON.parse(map.extras);
                   // console.log(extra.key + ": " + extra.value);
            });
        } else {
            this.subscription = NativeAppEventEmitter.addListener(
             'receiveNotification',
             (notification) => {
                 console.log('----------------notification', notification);
                 this.emit('receiveJpushNotification', { context:notification.aps.alert });
             }
           );
        }
    }
    clear () {
        if (this.subscription) {
            this.subscription.remove();
            this.subscription = null;
        }
        JPushModule.removeReceiveCustomMsgListener();
        JPushModule.removeReceiveNotificationListener();
    }
}

module.exports = new Manager();

```

* PDClient/project/App/manager/LoginMgr.js

```js
'use strict';
const ReactNative = require('react-native');
const {
    AsyncStorage,
} = ReactNative;
const EventEmitter = require('EventEmitter');

const ITEM_NAME = 'LOGIN_HISTORY_LIST';

class Manager extends EventEmitter {
    constructor () {
        super();
        this.list = [];
        this.get();
    }
    get () {
        return new Promise(async(resolve, reject) => {
            let list = [];
            try {
                const infoStr = await AsyncStorage.getItem(ITEM_NAME);
                list = JSON.parse(infoStr);
            } catch (e) {
            }
            this.list = list || [];
        });
    }
    set (list) {
        return new Promise(async(resolve, reject) => {
            this.list = list;
            await AsyncStorage.setItem(ITEM_NAME, JSON.stringify(list));
            resolve();
        });
    }
    savePhone (phone) {
        let list = this.list;
        if (_.includes(list, phone)) {
            list = _.without(list, phone);
        }
        list.unshift(phone);
        this.set(list);
    }
    clear () {
        this.list = [];
        AsyncStorage.removeItem(ITEM_NAME);
    }
}

module.exports = new Manager();

```

* PDClient/project/App/manager/NetMgr.js

```js
'use strict';
const ReactNative = require('react-native');
const {
    Platform,
    NetInfo,
} = ReactNative;
const EventEmitter = require('EventEmitter');

class Manager extends EventEmitter {
    constructor () {
        super();
        this._init = false;
        this.info = {
            connect: false, // 是否连接
            fee: false, // 是否收费
        };
    }
    register () {
        if (!this._init) {
            this._init = true;
            NetInfo.addEventListener(
                'change',
                this._handleConnectionInfoChange.bind(this)
            );
        }
    }
    unregister () {
        NetInfo.removeEventListener(
            'change',
            this._handleConnectionInfoChange.bind(this)
        );
        this._init = false;
    }
    _handleConnectionInfoChange (o) {
        this.updateConnectionInfo(o);
    }
    updateConnectionInfo (o) {
        if (Platform.OS === 'android') {
            if (o === 'WIFI' || o === 'ETHERNET' || o === 'DUMMY') {
                this.info = { connect: true, fee: false };
            } else if (/MOBILE.*/.test(o)) {
                this.info = { connect: true, fee: true };
            } else {
                this.info = { connect: false, fee: false };
            }
        } else {
            if (o === 'wifi') {
                this.info = { connect: true, fee: false };
            } else if (o === 'cell') {
                this.info = { connect: true, fee: true };
            } else {
                this.info = { connect: false, fee: false };
            }
        }
        app.connect = this.info.connect;
        if (!app.connect) {
            // Toast('当前设备已离线，请检查您的网络是否可用');
        }
    }
}

module.exports = new Manager();

```

* PDClient/project/App/manager/PersonalInfoMgr.js

```js
'use strict';
const ReactNative = require('react-native');
const {
    AsyncStorage,
} = ReactNative;
const EventEmitter = require('EventEmitter');
const ITEM_NAME = 'personalInfo';
const NEED_LOGIN_ITEM_NAME = 'personalInfoNeedLogin';
const REMAIN_AMOUNR = 'remainAmount';
const CURRENT_SHOP_INDEX = 'currentShopIndex';
const IS_FINISH_SCAN = 'isFinishScan';

class Manager extends EventEmitter {
    constructor () {
        super();
        this.get();
    }
    get () {
        return new Promise(async(resolve, reject) => {
            let info;
            try {
                const needLogin = await AsyncStorage.getItem(NEED_LOGIN_ITEM_NAME);
                const index = await AsyncStorage.getItem(CURRENT_SHOP_INDEX);
                const isFinishScan = await AsyncStorage.getItem(IS_FINISH_SCAN);
                this.needLogin = needLogin !== 'false';
                this.currentShopIndex = index || '0';
                this.isFinishScan = isFinishScan || 'false';
                this.remainAmount = await AsyncStorage.getItem(REMAIN_AMOUNR);
                const infoStr = await AsyncStorage.getItem(ITEM_NAME);
                info = JSON.parse(infoStr);
            } catch (e) {
                this.needLogin = true;
            }
            this.info = info || {};

            this.needLogin = this.needLogin || !info;
            this.currentShopIndex = this.currentShopIndex || '0';
            this.isFinishScan = this.isFinishScan || false;
            this.currentShopId = this.info.shipper && this.info.shipper.registerShopList[this.currentShopIndex].id || null;
            this.currentShopName = this.info.shipper && this.info.shipper.registerShopList[this.currentShopIndex].name || '';
            this.currentShopAddress = this.info.shipper && this.info.shipper.registerShopList[this.currentShopIndex].address || '';
            this.remainAmount = this.remainAmount || !info;
            this.initialized = true;
        });
    }
    set (info) {
        this.info = info;
        let roleList = info.agent ? ['发货人', '收货点'] : info.shipper ? ['发货人', '物流公司'] : ['发货人'];
        let authorityInfo = {
            currentRole: info.agent ? '收货点' : info.shipper ? '物流公司' : '发货人',
            roleList,
            isShipperChairman:!!(info.shipper && info.userId == info.shipper.chairMan.id),
            isAgentChairman:!!(info.agent && this.info.userId == info.agent.chairMan.id),
            authorityList:info.authority,
        };
        this.currentShopId = info.shipper && info.shipper.registerShopList[this.currentShopIndex].id || null;
        this.currentShopName = info.shipper && info.shipper.registerShopList[this.currentShopIndex].name || '';
        this.currentShopAddress = info.shipper && info.shipper.registerShopList[this.currentShopIndex].address || '';
        app.authority.clear();
        app.authority.set(authorityInfo);
        return new Promise(async(resolve, reject) => {
            await AsyncStorage.setItem(ITEM_NAME, JSON.stringify(info));
            resolve();
        });
    }
    setUserHead (head) {
        this.emit('USER_HEAD_CHANGE_EVENT', { head:head });
    }
    setCurrentShopIndex (index) {
        this.currentShopIndex = index;
        AsyncStorage.setItem(CURRENT_SHOP_INDEX, index + '');
    }
    setIsFinishScan (flag) {
        this.isFinishScan = flag;
        AsyncStorage.setItem(IS_FINISH_SCAN, flag + '');
    }
    setNeedLogin (flag) {
        this.needLogin = flag;
        AsyncStorage.setItem(NEED_LOGIN_ITEM_NAME, flag + '');
    }
    setRemainAmount (amount) {
        this.remainAmount = amount;
        AsyncStorage.setItem(REMAIN_AMOUNR, amount + '');
    }
    clear () {
        this.info = {};
        this.needLogin = true;
        this.currentShopIndex = '0';
        this.isFinishScan = false;
        this.currentShopId = null;
        this.currentShopName = '';
        this.remainAmount = 0;
        this.initialized = true;
        AsyncStorage.removeItem(ITEM_NAME);
    }
}

module.exports = new Manager();

```

* PDClient/project/App/manager/SettingMgr.js

```js
'use strict';
const ReactNative = require('react-native');
const {
    AsyncStorage,
} = ReactNative;
const EventEmitter = require('EventEmitter');

const ITEM_NAME = 'APP_SETTING_MANAGER';

class Manager extends EventEmitter {
    constructor () {
        super();
        this.data = {};
        this.get();
    }
    get () {
        return new Promise(async(resolve, reject) => {
            let data = {};
            try {
                const infoStr = await AsyncStorage.getItem(ITEM_NAME);
                data = JSON.parse(infoStr);
            } catch (e) {
                data = {};
            }
            this.data = data || {};
            this.THEME_COLOR = this.data.themeColor || CONSTANTS.THEME_COLORS[0];
        });
    }
    set (data) {
        return new Promise(async(resolve, reject) => {
            this.data = data;
            await AsyncStorage.setItem(ITEM_NAME, JSON.stringify(data));
            resolve();
        });
    }
    setOnlyWifiUpload (flag) {
        const data = this.data;
        data.onlyWifiUpload = flag;
        this.set(data);
    }
    setThemeColor (color) {
        const data = this.data;
        data.themeColor = color;
        this.set(data);
        this.THEME_COLOR = color;
        app.update();
    }
    clear () {
        this.data = {};
        AsyncStorage.removeItem(ITEM_NAME);
    }
}

module.exports = new Manager();

```

* PDClient/project/App/manager/UPpayMgr.js

```js
'use strict';
const { NativeAppEventEmitter, DeviceEventEmitter, Platform } = require('react-native');
const EventEmitter = require('EventEmitter');
const Uppay = require('../native/index.js').UPPay;
const gEventEmitter = Platform.OS === 'android' ? DeviceEventEmitter : NativeAppEventEmitter;

class Manager extends EventEmitter {

    /*
    * 注：amount为充值金额，单位为【元】。
    * body: 描述
    */
    doPay (amount, body, callbackSuccess, callbackError) {
        this.subscription = gEventEmitter.addListener('UNION_PAY', (obj) => { this.onPayResult(obj, callbackSuccess, callbackError); });
        var param = {
            userId:'59fc08416a14bc0456979059',//app.personal.info.userId,
            amount, // :1, //总金额 (元)
            body, // :'充值',
        };
        POST(app.route.ROUTE_GET_UNION_UNIFIED_ORDER, param, (data) => {
            if (data.success) {
                !data.context ? callbackSuccess(data) : Uppay.pay(data.context, callbackSuccess, callbackError);
            } else {
                Toast(data.msg);
            }
        });
    }
    onPayResult (data, callbackSuccess, callbackError) {
        if (this.subscription) {
            this.subscription.remove();
            this.subscription = null;
        }
        if (data.success == 'true') {
            callbackSuccess(data);
        } else {
            callbackError(data);
        }
    }
}
module.exports = new Manager();

```

* PDClient/project/App/manager/UpdateMgr.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    AsyncStorage,
} = ReactNative;
const EventEmitter = require('EventEmitter');

const ITEM_NAME = 'NEED_SHOW_SPLASH';
const Update = require('@remobile/react-native-update');
const UpdateInfoBox = require('../modules/update/UpdateInfoBox');

class Manager extends EventEmitter {
    constructor () {
        super();
        this.needShowSplash = true;
        this.getNeedShowSplash();
    }
    checkUpdate () {
        Update.checkVersion({
            versionUrl: app.route.ROUTE_VERSION_INFO_URL,
            iosAppId: CONSTANTS.IOS_APPID,
        }).then((options) => {
            if (!__DEV__ && options && options.newVersion && (app.scene || {}).pageName !== 'UpdatePage') {
                app.showModal(<UpdateInfoBox options={options} />, { backgroundColor:'rgba(0, 0, 0, 0.6)' });
            }
        });
    }
    getNeedShowSplash () {
        return new Promise(async(resolve, reject) => {
            let needShowSplash;
            try {
                const infoStr = await AsyncStorage.getItem(ITEM_NAME);
                needShowSplash = JSON.parse(infoStr);
            } catch (e) {
            }
            this.needShowSplash = needShowSplash == null ? true : needShowSplash;
            this.initialized = true;
        });
    }
    setNeedShowSplash (needShowSplash) {
        return new Promise(async(resolve, reject) => {
            this.needShowSplash = needShowSplash;
            await AsyncStorage.setItem(ITEM_NAME, JSON.stringify(needShowSplash));
            resolve();
        });
    }
    clear () {
        this.needShowSplash = null;
        AsyncStorage.removeItem(ITEM_NAME);
    }
}

module.exports = new Manager();

```

* PDClient/project/App/manager/WXpayMgr.js

```js
'use strict';
const { NativeAppEventEmitter, DeviceEventEmitter, Platform } = require('react-native');
const EventEmitter = require('EventEmitter');
const Wxpay = require('../native/index.js').WeixinPay;
const gEventEmitter = Platform.OS === 'android' ? DeviceEventEmitter : NativeAppEventEmitter;

class Manager extends EventEmitter {

    /*
    * 注：amount为充值金额，单位为【元】。
    * body: 描述
    * appid: 公众账号ID
    * noncestr: 随机字符串
    * package: 扩展字段
    * partnerid: 商户号
    * prepayid: 预支付交易会话ID
    * timestamp: 时间戳
    * sign: 签名
    */
    doPay (amount, body, callbackSuccess, callbackError) {
        this.subscription = gEventEmitter.addListener('WEIXIN_PAY', (obj) => { this.onPayResult(obj, callbackSuccess, callbackError); });
        var param = {
            userId:app.personal.info.userId,
            amount, // :1, //总金额 (元)
            body, // :'充值',
        };
        POST(app.route.ROUTE_GET_UNIFIED_ORDER, param, (data) => {
            if (data.success) {
                !data.context ? callbackSuccess(data) : Wxpay.pay(data.context, callbackSuccess, callbackError);
            } else {
                Toast(data.msg);
            }
        });
    }
    onPayResult (data, callbackSuccess, callbackError) {
        if (this.subscription) {
            this.subscription.remove();
            this.subscription = null;
        }
        if (data.success == 'true') {
            callbackSuccess(data);
        } else {
            callbackError(data);
        }
    }
}
module.exports = new Manager();

```

* PDClient/project/App/native/index.js

```js

const WeixinPay = require('./wxpay/index.js');
const UPPay = require('./uppay/index.js');
module.exports = {
    WeixinPay: WeixinPay,
    UPPay: UPPay,
};

```

* PDClient/project/App/utils/index.js

```js
const common = require('./common');
const net = require('./net');
const check = require('./check');
const date = require('./date');
const qrcode = require('./qrcode');

module.exports = {
    ...common,
    ...net,
    ...check,
    ...date,
    ...qrcode,
};

```

* PDClient/project/App/resource/audio.js

```js
module.exports = {};

```

* PDClient/project/App/resource/image.js

```js
module.exports = {
    cargo_balance_background:require('./image/cargo/balance_background.png'),
    cargo_cancel:require('./image/cargo/cancel.png'),
    cargo_endPoint:require('./image/cargo/endPoint.png'),
    cargo_point:require('./image/cargo/point.png'),
    common_back:require('./image/common/back.png'),
    common_close:require('./image/common/close.png'),
    common_default:require('./image/common/default.png'),
    common_go:require('./image/common/go.png'),
    common_home:require('./image/common/home.png'),
    common_left_menu:require('./image/common/left_menu.png'),
    common_radio:require('./image/common/radio.png'),
    common_search:require('./image/common/search.png'),
    common_search_button:require('./image/common/search_button.png'),
    home_cargo:require('./image/home/cargo.png'),
    home_cargo_press:require('./image/home/cargo_press.png'),
    home_main:require('./image/home/main.png'),
    home_main_press:require('./image/home/main_press.png'),
    home_mine:require('./image/home/mine.png'),
    home_mine_press:require('./image/home/mine_press.png'),
    home_order:require('./image/home/order.png'),
    home_order_press:require('./image/home/order_press.png'),
    home_pre_order:require('./image/home/pre_order.png'),
    home_pre_order_press:require('./image/home/pre_order_press.png'),
    home_receivePoint:require('./image/home/receivePoint.png'),
    home_receivePoint_press:require('./image/home/receivePoint_press.png'),
    home_roadmap:require('./image/home/roadmap.png'),
    home_roadmap_press:require('./image/home/roadmap_press.png'),
    home_truck:require('./image/home/truck.png'),
    home_truck_unselect:require('./image/home/truck_unselect.png'),
    login_alipay_button:require('./image/login/alipay_button.png'),
    login_logo:require('./image/login/logo.png'),
    login_password:require('./image/login/password.png'),
    login_password_check:require('./image/login/password_check.png'),
    login_password_look:require('./image/login/password_look.png'),
    login_phone:require('./image/login/phone.png'),
    login_qq_button:require('./image/login/qq_button.png'),
    login_weixin_button:require('./image/login/weixin_button.png'),
    order_add:require('./image/order/add.png'),
    order_choose:require('./image/order/choose.png'),
    order_choose_r:require('./image/order/choose_r.png'),
    order_delete:require('./image/order/delete.png'),
    order_down:require('./image/order/down.png'),
    order_logo:require('./image/order/logo.png'),
    order_no_choose:require('./image/order/no_choose.png'),
    order_no_choose_r:require('./image/order/no_choose_r.png'),
    order_pay_success:require('./image/order/pay_success.png'),
    order_point:require('./image/order/point.png'),
    order_point_green:require('./image/order/point_green.png'),
    order_upload_picture:require('./image/order/upload_picture.png'),
    personal_add:require('./image/personal/add.png'),
    personal_default_head:require('./image/personal/default_head.png'),
    personal_delete:require('./image/personal/delete.png'),
    personal_female:require('./image/personal/female.png'),
    personal_info:require('./image/personal/info.png'),
    personal_male:require('./image/personal/male.png'),
    personal_money:require('./image/personal/money.png'),
    personal_personal_center:require('./image/personal/personal_center.png'),
    personal_scan:require('./image/personal/scan.png'),
    personal_settings:require('./image/personal/settings.png'),
    personal_wallet:require('./image/personal/wallet.png'),
    roadmap_add_photos:require('./image/roadmap/add_photos.png'),
    roadmap_address:require('./image/roadmap/address.png'),
    roadmap_collection_normal:require('./image/roadmap/collection_normal.png'),
    roadmap_collection_press:require('./image/roadmap/collection_press.png'),
    roadmap_comment:require('./image/roadmap/comment.png'),
    roadmap_share:require('./image/roadmap/share.png'),
    shipper_care_normal:require('./image/shipper/care_normal.png'),
    shipper_care_press:require('./image/shipper/care_press.png'),
    shipper_phone:require('./image/shipper/phone.png'),
    shipper_tell_phone:require('./image/shipper/tell_phone.png'),
    shop_add_photos:require('./image/shop/add_photos.png'),
    shop_address:require('./image/shop/address.png'),
    shop_back:require('./image/shop/back.png'),
    shop_choice:require('./image/shop/choice.png'),
    shop_comment:require('./image/shop/comment.png'),
    shop_follow_normal:require('./image/shop/follow_normal.png'),
    shop_follow_press:require('./image/shop/follow_press.png'),
    shop_home:require('./image/shop/home.png'),
    shop_no_comment:require('./image/shop/no_comment.png'),
    shop_no_share:require('./image/shop/no_share.png'),
    shop_phone:require('./image/shop/phone.png'),
    shop_praise_normal:require('./image/shop/praise_normal.png'),
    shop_praise_press:require('./image/shop/praise_press.png'),
    shop_report_normal:require('./image/shop/report_normal.png'),
    shop_report_press:require('./image/shop/report_press.png'),
    shop_share:require('./image/shop/share.png'),
    shop_shop_down_left:require('./image/shop/shop_down_left.png'),
    shop_shop_down_right:require('./image/shop/shop_down_right.png'),
    shop_unchoice:require('./image/shop/unchoice.png'),
    shop_zan_normal:require('./image/shop/zan_normal.png'),
    shop_zan_press:require('./image/shop/zan_press.png'),
    splash_logo:require('./image/splash/logo.png'),
    splash_splash1:require('./image/splash/splash1.png'),
    splash_splash2:require('./image/splash/splash2.png'),
    splash_splash3:require('./image/splash/splash3.png'),
    splash_start:require('./image/splash/start.png'),
};

```

* PDClient/project/App/components/ActionSheet/button.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    Text,
    TouchableOpacity,
} = ReactNative;

module.exports = React.createClass({
    render () {
        return (
            <TouchableOpacity
                activeOpacity={0.5}
                style={[styles.button, this.props.buttonStyle]}
                onPress={this.props.onPress}>
                <Text style={[styles.buttonText, this.props.textStyle]}>
                    {this.props.children}
                </Text>
            </TouchableOpacity>
        );
    },
});

const styles = StyleSheet.create({
    buttonText: {
        color: '#A1A2A3',
        alignSelf: 'center',
        fontSize: 18,
    },
    button: {
        height: 50,
        backgroundColor: '#FEFFFF',
        borderColor: '#EDEEEF',
        borderBottomWidth: 1,
        alignSelf: 'stretch',
        justifyContent: 'center',
    },
});

```

* PDClient/project/App/components/ActionSheet/index.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    TouchableOpacity,
    View,
} = ReactNative;

const Button = require('./button');
const Overlay = require('./overlay');
const Sheet = require('./sheet');

module.exports = React.createClass({
    getDefaultProps () {
        return {
            cancelText: 'Cancel',
        };
    },
    render () {
        return (
            <Overlay visible={this.props.visible}>
                <View style={styles.actionSheetContainer}>
                    <TouchableOpacity
                        style={{ flex:1 }}
                        onPress={this.props.onCancel} />
                    <Sheet visible={this.props.visible}>
                        <View style={styles.buttonContainer}>
                            {this.props.children}
                        </View>
                        <Button
                            buttonStyle={styles.cancelButton}
                            textStyle={styles.cancelText}
                            onPress={this.props.onCancel}>{this.props.cancelText}</Button>
                    </Sheet>
                </View>
            </Overlay>
        );
    },
});
module.exports.Button = Button;

const styles = StyleSheet.create({
    actionSheetContainer: {
        flex: 1,
        padding: 10,
        paddingBottom: 6,
        justifyContent: 'flex-end',
        backgroundColor: 'rgba(0, 0, 0, 0.5)',
    },
    buttonContainer: {
        borderRadius:6,
        overflow: 'hidden',
    },
    cancelButton: {
        marginTop:6,
        marginBottom:6,
        borderRadius:6,
        backgroundColor: '#A0D26F',
        borderBottomWidth: 0,
    },
    cancelText: {
        color:'#FFFFFF',
    },
});

```

* PDClient/project/App/components/ActionSheet/overlay.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    Animated,
    StyleSheet,
} = ReactNative;

const DEFAULT_ANIMATE_TIME = 300;

module.exports = React.createClass({
    getInitialState () {
        return {
            fadeAnim: new Animated.Value(0),
            overlayStyle: styles.emptyOverlay, // on android opacity=0 also can cover screen, so use overlayStyle fix it
        };
    },
    onAnimatedEnd () {
        !this.props.visible && this.setState({ overlayStyle:styles.emptyOverlay });
    },
    componentWillReceiveProps (newProps) {
        newProps.visible && this.setState({ overlayStyle: styles.fullOverlay });
        return Animated.timing(this.state.fadeAnim, {
            toValue: newProps.visible ? 1 : 0,
            duration: DEFAULT_ANIMATE_TIME,
        }).start(this.onAnimatedEnd);
    },

    render () {
        return (
            <Animated.View style={[this.state.overlayStyle, { opacity: this.state.fadeAnim }]}>
                {this.props.children}
            </Animated.View>
        );
    },
});

const styles = StyleSheet.create({
    fullOverlay: {
        top: 0,
        bottom: 0,
        left: 0,
        right: 0,
        backgroundColor: 'transparent',
        position: 'absolute',
    },
    emptyOverlay: {
        backgroundColor: 'transparent',
        position: 'absolute',
        left: -sr.w,
    },
});

```

* PDClient/project/App/components/ActionSheet/sheet.js

```js
'use strict';

const React = require('react');const ReactNative = require('react-native');
const {
    Animated,
} = ReactNative;

const DEFAULT_BOTTOM = -300;
const DEFAULT_ANIMATE_TIME = 300;

module.exports = React.createClass({
    getInitialState () {
        return {
            bottom: new Animated.Value(DEFAULT_BOTTOM),
        };
    },
    componentWillReceiveProps (newProps) {
        return Animated.timing(this.state.bottom, {
            toValue: newProps.visible ? 0 : DEFAULT_BOTTOM,
            duration: DEFAULT_ANIMATE_TIME,
        }).start();
    },

    render () {
        return (
            <Animated.View style={{ bottom: this.state.bottom }}>
                {this.props.children}
            </Animated.View>
        );
    },
});

```

* PDClient/project/App/components/ActionSheet/view.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    TouchableOpacity,
    View,
} = ReactNative;

const Button = require('./button');
const Sheet = require('./sheet');

module.exports = React.createClass({
    getDefaultProps () {
        return {
            cancelText: 'Cancel',
        };
    },
    render () {
        return (
            <View style={styles.actionSheetContainer}>
                <TouchableOpacity
                    style={{ flex:1 }}
                    onPress={this.props.onCancel} />
                <View visible={this.props.visible}>
                    <View style={styles.buttonContainer}>
                        {this.props.children}
                    </View>
                    <Button
                        buttonStyle={styles.cancelButton}
                        textStyle={styles.cancelText}
                        onPress={this.props.onCancel}>{this.props.cancelText}</Button>
                </View>
            </View>
        );
    },
});
module.exports.Button = Button;

const styles = StyleSheet.create({
    actionSheetContainer: {
        flex: 1,
        width:sr.w,
        padding: 10,
        paddingBottom: 6,
        justifyContent: 'flex-end',
    },
    buttonContainer: {
        borderRadius:6,
        overflow: 'hidden',
    },
    cancelButton: {
        marginTop:6,
        marginBottom:6,
        borderRadius:6,
        backgroundColor: '#A0D26F',
        borderBottomWidth: 0,
    },
    cancelText: {
        color:'#FFFFFF',
    },
});

```

* PDClient/project/App/native/uppay/index.js

```js
const exec = require('@remobile/react-native-cordova').exec;

const UPPay = {
    pay (json, successFn, failureFn) {
        exec(successFn, failureFn, 'UPPay', 'payment', [json]);
    },
};

module.exports = UPPay;

```

* PDClient/project/App/native/wxpay/index.js

```js
const exec = require('@remobile/react-native-cordova').exec;

const WeixinPay = {
    pay (json, successFn, failureFn) {
        exec(successFn, failureFn, 'WeixinPay', 'payment', [json]);
    },
};

module.exports = WeixinPay;

```

* PDClient/project/App/utils/check/index.js

```js
module.exports = {
    checkPhone (phone) {
        return /^1\d{10}$/.test(phone);
    },
    checkIdentifyNumber (number) {
        return /^(^[1-9]\d{7}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}$)|(^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])((\d{4})|\d{3}[Xx])$)$/.test(number);
    },
    checkPassword (pwd) {
        return /^[\d\w_]{6,20}$/.test(pwd);
    },
    checkVerificationCode (code) {
        return /^\d{6}$/.test(code);
    },
    checkMailAddress (code) {
        const reg = /^([a-zA-Z0-9]+[_.]?)*[a-zA-Z0-9]+@([a-zA-Z0-9]+[_.]?)*[a-zA-Z0-9]+\.[a-zA-Z]{2,3}$/;
        return reg.test(code);
    },
    /**
     * 验证银行卡号(16位或19位数字)
    */
    checkBankCardCode (code) {
        return /^(\d{16}|\d{19})$/.test(code);
    },
    /**
     * 验证出生日期(平年日期格式为YYYY-MM-DD的正则表达式)
    */
    checkBirthDate (str) {
        var re = /^([0-9]{3}[1-9]|[0-9]{2}[1-9][0-9]{1}|[0-9]{1}[1-9][0-9]{2}|[1-9][0-9]{3})-(((0[13578]|1[02])-(0[1-9]|[12][0-9]|3[01]))|((0[469]|11)-(0[1-9]|[12][0-9]|30))|(02-(0[1-9]|[1][0-9]|2[0-8])))$/;
        if (re.test(str)) {
            return true;
        } else {
            return false;
        }
    },
    /**
    * 验证字符串是否包含特殊符号
    */
    checkStr (str) {
        let re = /^[\u4E00-\u9FA5A-Za-z0-9-]+$/;
        if (re.test(str)) {
            return true;
        } else {
            return false;
        }
    },
    /**
     * 验证数字(可以包含两个小数点)
    */
    check2PointNum (code) {
        return /^(0|[1-9][0-9]{0,9})(\.[0-9]{1,2})?$/.test(code);
    },
    /**
     * 验证数字(包含不限小数点)
    */
    checkNum (str) {
        var re = /^[0-9]+.?[0-9]*$/;
        if (re.test(str)) {
            return true;
        } else {
            return false;
        }
    },
    /**
     * 验证数字(正整数)
    */
    checkIntNum (str) {
        var re = /^[0-9]*[1-9][0-9]*$/;
        if (re.test(str)) {
            return true;
        } else {
            return false;
        }
    },
    /**
     * 验证非负整数
    */
    checkUnIntNum (str) {
        var re = /^\d+$/;
        if (re.test(str)) {
            return true;
        } else {
            return false;
        }
    },
    /**
     * 验证数字为正数且最多包含两位小数
    */
    checkInt2PointNum (str) {
        let re = /^[0-9]+(.[0-9]{1,2})?$/;
        if (re.test(str)) {
            return true;
        } else {
            return false;
        }
    },
    /**
     * 验证车牌
    */
    checkPlate (code) {
        return /^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领A-Z]{1}[A-Z]{1}[A-Z0-9]{4}[A-Z0-9挂学警港澳]{1}$/.test(code);
    },

};

```

* PDClient/project/App/utils/common/index.js

```js
module.exports = {
    until (test, iterator, callback) {
        if (!test()) {
            iterator((err) => {
                if (err) {
                    return callback(err);
                }
                this.until(test, iterator, callback);
            });
        } else {
            callback();
        }
    },
    N (num, rank = 2) {
        typeof num !== 'number' && (num = 0);
        return parseFloat(num.toFixed(rank));
    },
    toThousands (num) {
        return (num || 0).toString().replace(/(\d)(?=(?:\d{3})+$)/g, '$1,');
    },
    getPercentages (list) {
        const sum = _.sum(list);
        return list.map((v) => Math.round(v * 100 / sum) + '%');
    },
    getVisibleText (text, n) {
        let realLength = 0, len = text.length, preLen = -1, charCode = -1, needCut = false;
        for (let i = 0; i < len; i++) {
            charCode = text.charCodeAt(i);
            if (charCode >= 0 && charCode <= 128) {
                realLength += 1;
            } else {
                realLength += 2;
            }
            if (preLen === -1 && realLength >= n) {
                preLen = i + 1;
            } else if (realLength > n + 2) {
                needCut = true;
                break;
            }
        }
        if (needCut) {
            text = text.substr(0, preLen) + '..';
        }
        return text;
    },
    cutLimitText (text, n) {
        let realLength = 0, len = text.length, preLen = -1, charCode = -1, needCut = false;
        for (let i = 0; i < len; i++) {
            charCode = text.charCodeAt(i);
            if (charCode >= 0 && charCode <= 128) {
                realLength += 1;
            } else {
                realLength += 2;
            }
            if (preLen === -1 && realLength >= n) {
                preLen = i + 1;
            } else if (realLength > n) {
                needCut = true;
                break;
            }
        }
        if (needCut) {
            text = text.substr(0, preLen);
        }
        return text;
    },
    getAddressTitleArray (list = [], lastCode) {
        let options;
        let value = [];
        for (const item of list) {
            const parent = _.find(item, o => o.code === lastCode) || {};
            parent.children = options;
            lastCode = parent.parentCode;
            parent.name && (value.unshift(parent));
            options = item;
        }
        return value;
    },
    getStartPointTitleArray (list = [], lastCode, id) { // 如果始发地是分店或者收货点，则传id，否则不传，一旦传了id，lastCode则无效
        let options;
        let value = [];
        for (const item of list) {
            const parent = _.find(item, o => !id ? o.code === lastCode : o.id === id) || {};
            parent.children = options;
            lastCode = parent.isLeaf ? parent.addressRegionLastCode : parent.parentCode;
            parent && value.unshift(parent);
            options = item;
            id = undefined;
        }
        return value;
    },
};

```

* PDClient/project/App/utils/date/index.js

```js
const moment = require('moment');

module.exports = {
    createDateData (start, end) {
        let date = [];
        let iy = start.year(), im = start.month() + 1, id = start.date();
        let ey = end.year(), em = end.month() + 1, ed = end.date();
        for (let y = iy; y <= ey; y++) {
            let month = [];
            let mm = [0, 31, (!(y % 4) & (!!(y % 100))) | (!(y % 400)) ? 29 : 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
            let iim = (y === iy) ? im : 1;
            let eem = (y === ey) ? em : 12;
            for (let m = iim; m <= eem; m++) {
                let day = [];
                let iid = (y === iy && m === im) ? id : 1;
                let eed = (y === ey && m === em) ? ed : mm[m];
                for (let d = iid; d <= eed; d++) {
                    day.push(d + '日');
                }
                month.push({ [m + '月']: day });
            }
            date.push({ [y + '年']: month });
        }
        return date;
    },
    createTimeData (start, end, date, hasSeconds) {
        const d = date.startOf('day');
        const isStart = d.isSame(moment(start).startOf('day'));
        const isEnd = d.isSame(moment(end).startOf('day'));
        let time = [];
        let ih = start.hour(), it = start.minute(), is = start.second();
        let eh = end.hour(), et = end.minute(), es = start.second();
        let iih = isStart ? ih : 0;
        let eeh = isEnd ? eh : 24;

        for (let h = iih; h < eeh; h++) {
            let minute = [];
            let iit = (isStart && h === ih) ? it : 0;
            let eet = (isEnd && h === eh) ? et : 60;
            for (let t = iit; t < eet; t++) {
                if (!hasSeconds) {
                    minute.push(t + '分');
                } else {
                    let second = [];
                    let iis = (isStart && h === ih && t === it) ? is : 1;
                    let ees = (isEnd && h === eh && t === et) ? es : 60;
                    for (let s = iis; s <= ees; s++) {
                        second.push(s + '秒');
                    }
                    minute.push({ [t + '分']: second });
                }
            }
            time.push({ [h + '时']: minute });
        }
        return time;
    },
    showDatePicker (Picker, start, end, selected) {
        return new Promise(async(resolve) => {
            const selectedValue = [selected.year() + '年', (selected.month() + 1) + '月', selected.date() + '日'];
            const pickerData = this.createDateData(start, end);
            Picker(pickerData, selectedValue, '选择日期').then((value) => {
                const time = moment(value.join(''), 'YYYY年M月D日');
                const date = moment(time.year() + '年' + (time.month() + 1) + '月' + time.date() + '日' + selected.hour() + '时' + selected.minute() + '分', 'YYYY年MM月DD日HH时mm分');
                resolve(date);
            });
        });
    },
    showTimePicker (Picker, start, end, selected, hasSeconds) {
        return new Promise(async(resolve) => {
            const selectedValue = [selected.hour() + '时', selected.minute() + '分', selected.second() + '秒'];
            const pickerData = this.createTimeData(start, end, selected, hasSeconds);
            Picker(pickerData, selectedValue, '选择时间').then((value) => {
                const time = moment(value.join(''), 'HH时mm分');
                const date = moment(selected.year() + '年' + (selected.month() + 1) + '月' + selected.date() + '日' + time.hour() + '时' + time.minute() + '分', 'YYYY年MM月DD日HH时mm分');
                resolve(date);
            });
        });
    },
};

```

* PDClient/project/App/utils/net/Get.js

```js
'use strict';

function GET (url, success, error) {
    console.log('getSend:', url);
    app.showLoading();
    fetch(url, {
        method: 'get',
        headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json',
        },
    })
    .then((response) => response.json())
    .then((json) => {
        console.log('getRecv:', json);
        app.hideLoading();
        success && success(json);
    })
    .catch((err) => {
        app.hideLoading();
        if (!error || !error(err)) {
            Toast('网络出错');
        }
    });
}

module.exports = GET;

```

* PDClient/project/App/utils/net/Post.js

```js
'use strict';

module.exports = (url, parameter, success, failed, wait) => {
    console.log(url, 'send:', parameter);
    if (typeof failed === 'boolean') {
        wait = failed;
        failed = null;
    }
    if (wait) {
        app.showLoading();
    }
    fetch(url, {
        method: 'post',
        headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json',
        },
        body: JSON.stringify(parameter),
    })
    .then((response) => response.json())
    .then((data) => {
        console.log(url, 'recv:', data);
        success(data);
        if (wait) {
            app.hideLoading();
        }
    })
    .catch((error) => {
        if (!failed || !failed(error)) {
            Toast('网络错误');
            console.log(url + ':网络错误', error);
            if (wait) {
                app.hideLoading();
            }
        }
    });
};

```

* PDClient/project/App/utils/net/Upload.js

```js
'use strict';

const FileTransfer = require('@remobile/react-native-file-transfer');

function UPLOAD (filePath, url, options, onprogress, success, failed, wait) {
    console.log(url, 'send:', options);
    if (typeof failed === 'boolean') {
        wait = failed;
        failed = null;
    }
    if (wait) {
        app.showLoading();
    }
    const fileTransfer = new FileTransfer();
    fileTransfer.onprogress = onprogress;
    fileTransfer.upload(filePath, app.route.ROUTE_UPDATE_FILE, (res) => {
        let json = {};
        try {
            json = JSON.parse(res.response);
        } catch (error) {
            if (!failed || !failed(error)) {
                Toast('数据解析错误');
                if (wait) {
                    app.hideLoading();
                }
            }
        }
        console.log(url, 'recv:', json);
        app.hideLoading();
        success(json);
    }, (err) => {
        if (!failed || !failed(err)) {
            Toast('上传失败');
            if (wait) {
                app.hideLoading();
            }
        }
    }, options);
}

module.exports = UPLOAD;

```

* PDClient/project/App/utils/net/index.js

```js
'use strict';

module.exports = {
    POST: require('./Post'),
    GET: require('./Get'),
    UPLOAD: require('./Upload'),
};

```

* PDClient/project/App/utils/qrcode/index.js

```js
module.exports = {
    genQRString (type, id) {
        const str = JSON.stringify({ c: 's', t: type, d: id });
        let output = '';
        for (let x = 0, y = str.length, charCode, hexCode; x < y; ++x) {
            charCode = str.charCodeAt(x);
            if (charCode < 128) {
                charCode += 128;
            } else if (charCode > 127) {
                charCode -= 128;
            }
            charCode = 255 - charCode;
            hexCode = charCode.toString(16);
            if (hexCode.length < 2) {
                hexCode = '0' + hexCode;
            }
            output += hexCode;
        }
        return output;
    },
    getQRResult (str) {
        let output = '';
        for (let x = 0, y = str.length, charCode, hexCode; x < y; x += 2) {
            hexCode = str.substr(x, 2);
            charCode = parseInt(hexCode, 16);
            charCode = 255 - charCode;
            if (charCode < 128) {
                charCode += 128;
            } else if (charCode > 127) {
                charCode -= 128;
            }
            output += String.fromCharCode(charCode);
        }
        try {
            const result = JSON.parse(output);
            if (result.c !== 's') {
                return undefined;
            }
            return { type: result.t, id: result.d };
        } catch (e) {
            return undefined;
        }
    },
};

```
