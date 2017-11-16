* PDShop/project/App/index.js

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
const PersonalInfoMgr = require('./manager/PersonalInfoMgr');
const UpdateMgr = require('./manager/UpdateMgr');
const SettingMgr = require('./manager/SettingMgr');
const LoginMgr = require('./manager/LoginMgr');
const ScanStoageOrderMgr = require('./manager/ScanStoageOrderMgr');
const JpushMgr = require('./manager/JpushMgr');
const { ProgressHud, DelayTouchableOpacity, Modal } = COMPONENTS;

global.app = {
    route: Route,
    utils: Utils,
    img: img,
    aud: aud,
    data: {},
    personal: PersonalInfoMgr,
    setting: SettingMgr,
    updateMgr:UpdateMgr,
    login: LoginMgr,
    jpush: JpushMgr,
    scanStoageOrder: ScanStoageOrderMgr,
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
                    <Text style={styles.navBarButtonText}>
                        {leftButton.title}
                    </Text>
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
                if (this.state.is_hud_visible) {
                    this.setState({ is_hud_visible: false });
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
    navBarButtonText: {
        color: '#FFFFFF',
        fontSize: 18,
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
        width: sr.navbarHeight * (sr.translucent ? 0.2 : 0.3),
        height: sr.navbarHeight * (sr.translucent ? 0.2 : 0.3),
    },
});

```

* PDShop/project/App/components/AutogrowInput.js

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

* PDShop/project/App/components/Button.js

```js
'use strict';

const React = require('react');const { PropTypes } = React;
const ReactNative = require('react-native');
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

* PDShop/project/App/components/ClipRect.js

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

* PDShop/project/App/components/CustomMessageBox.js

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
                <Text style={styles.content}>
                    {this.props.children}
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
        marginTop: 20,
    },
});

```

* PDShop/project/App/components/CustomSheet.js

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

* PDShop/project/App/components/DImage.js

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

* PDShop/project/App/components/DelayTouchableOpacity.js

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

* PDShop/project/App/components/Label.js

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

* PDShop/project/App/components/MessageBox.js

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

* PDShop/project/App/components/Modal.js

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

* PDShop/project/App/components/PageList.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
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
        const { list, pageNo } = this.props;
        this.list = list || [];
        this.pageNo = pageNo;
        return {
            dataSource: this.list,
            infiniteLoadStatus: list ? (list.length === 0 ? STATUS_NO_DATA : list.length < CONSTANTS.PER_PAGE_COUNT ? STATUS_ALL_LOADED : STATUS_TEXT_HIDE) : STATUS_START_LOAD,
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
        this.setState({ infiniteLoadStatus: this.pageNo === 0 ? STATUS_START_LOAD : STATUS_HAVE_MORE }, () => {
            POST(this.props.listUrl, param, this.getListSuccess, this.getListFailed, wait);
        });
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
        this.getList();
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

* PDShop/project/App/components/PageListView.js

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

* PDShop/project/App/components/Picker.js

```js
'use strict';

import Picker from 'react-native-picker';

module.exports = (pickerData, selectedValue, title) => {
    const color = Math.floor(app.setting.THEME_COLOR.replace('#', '0x'));
    const pickerToolBarBg = [(color >> 16) & 0xff, (color >> 8) & 0xff, color & 0xff, 1];
    return new Promise(async(resolve) => {
        Picker.isPickerShow((show) => {
            if (show) {
                app.closeModal();
                Picker.hide();
            } else {
                Picker.init({
                    pickerConfirmBtnText: '确定',
                    pickerConfirmBtnColor: [255, 255, 255, 1],
                    pickerCancelBtnText: '取消',
                    pickerCancelBtnColor:  [255, 255, 255, 1],
                    pickerTitleText: title || '',
                    pickerTitleColor: [255, 255, 255, 1],
                    pickerToolBarBg,
                    pickerBg: [255, 255, 255, 1],
                    pickerFontColor: [0, 0, 0, 1],
                    pickerToolBarFontSize: 16,
                    pickerFontSize: 16,
                    pickerData,
                    selectedValue,
                    onPickerCancel: () => { app.closeModal(); },
                    onPickerConfirm: (value) => { app.closeModal(); resolve(value); },
                });
                app.showModal();
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

* PDShop/project/App/components/ProgressBar.js

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

* PDShop/project/App/components/ProgressHud.js

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

* PDShop/project/App/components/RText.js

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

* PDShop/project/App/components/ScoreSelect.js

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

* PDShop/project/App/components/SelectRegionAddress.js

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
        title: '所在地',
        rightButton: { title: '确定', handler: () => { app.scene.confirm(); } },
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
        const param = { userId: app.personal.info.userId, parentCode, type:0 };
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
        const param = { userId: app.personal.info.userId, addressLastCode };
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
        if (!!lastObj && lastObj.isLeaf) {
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

* PDShop/project/App/components/SelectShortAddress.js

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
        const lastObj = allAddress[allAddress.length - 1];
        if (!!lastObj && lastObj.isLeaf) {
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
        const { parentCode } = this.props;
        return (
            <View style={styles.container}>
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

* PDShop/project/App/components/SelectStartPointAddress.js

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
        rightButton: { title: '确定', handler: () => { app.scene.confirm(); } },
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            dataSource:[],
            allAddress:[],
        };
    },
    componentWillMount () {
        const { endPointLastCode, id } = this.props;
        // const endPointLastCode = 520101;
        // const id = "59e426b1b6ee191fe5d4fbf8";
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
            this.setState({ dataSource: data.context.addressList });
        } else {
            Toast(data.msg);
        }
    },
    getAddressFromLastCode (addressLastCode, id) {
        const param = { userId: app.personal.info.userId, addressLastCode, isLeaf: id ? true : false };
        POST(app.route.ROUTE_GET_START_POINT_ADDRESS_FROME_LAST_CODE, param, this.getAddressFromLastCodeSuccess.bind(null, addressLastCode, id), false);
    },
    getAddressFromLastCodeSuccess (addressLastCode, id, data) {
        if (data.success) {
            this.setState({ allAddress: app.utils.getStartPointTitleArray(data.context.addressList, addressLastCode, id), dataSource:data.context.addressList[0] || [] });
        }
    },
    confirm () {
        const { allAddress } = this.state;
        const lastObj = allAddress[allAddress.length - 1];
        const { confirmAddress, isNeedSelectAll } = this.props;
        if (isNeedSelectAll) {
            if (!!lastObj && lastObj.isLeaf) {
                if (lastObj.isShop) {
                    confirmAddress({ startPoint:_.join(_.map(allAddress, 'name'), ''), shopId:lastObj.id, lastCode:lastObj.addressRegionLastCode });
                } else if (lastObj.isAgent) {
                    confirmAddress({ startPoint:_.join(_.map(allAddress, 'name'), ''), agentId:lastObj.id, lastCode:lastObj.addressRegionLastCode });
                }
                app.pop();
            } else {
                Toast('请选择完整到货地');
            }
        } else {
            confirmAddress({ startPoint:_.join(_.map(allAddress, 'name'), ''), lastCode:lastObj.code });
            app.pop();
        }
    },
    getSubAddressList (obj) {
        let { allAddress } = this.state;
        const lastObj = allAddress[allAddress.length - 1];
        if (!!lastObj && lastObj.isLeaf) {
            allAddress.pop();
        }
        allAddress.push(obj);
        this.setState({ allAddress: _.uniqBy(allAddress, 'code') });
        if (obj.isLeaf) { // 省+市+名字
            if (obj.isShop) {
                this.props.confirmAddress({ startPoint:_.join(_.map(this.state.allAddress, 'name'), ''), shopId:obj.id, lastCode:obj.addressRegionLastCode });
            } else if (obj.isAgent) {
                this.props.confirmAddress({ startPoint:_.join(_.map(this.state.allAddress, 'name'), ''), agentId:obj.id, lastCode:obj.addressRegionLastCode });
            } else {
                this.props.confirmAddress({ startPoint:_.join(_.map(this.state.allAddress, 'name'), ''), lastCode:obj.code });
            }
            app.pop();
        } else {
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

* PDShop/project/App/components/Slider.js

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

* PDShop/project/App/components/StarBar.js

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

* PDShop/project/App/components/TouchAbleLink.js

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

* PDShop/project/App/components/WebviewMessageBox.js

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

* PDShop/project/App/components/index.js

```js
module.exports = {
    Button: require('./Button'),
    PageList: require('./PageList'),
    ActionSheet: require('./ActionSheet/index'),
    CustomSheet: require('./CustomSheet'),    ClipRect: require('./ClipRect'),
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

* PDShop/project/App/config/Authority.js

```js
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
const AH_WITHDRAW = 10006; // 充值的权限
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
const AH_LOOK_ROADMAP = 30005; // 查看路线的权限
const AH_CREATE_TRUCK = 30006; // 创建货车的权限
const AH_MODIFY_TRUCK = 30007; // 修改货车的权限
const AH_REMOVE_TRUCK = 30008; // 删除货车的权限
const AH_LOOK_TRUCK = 30009; // 查看货车的权限
const SELECT_CARRY_PARTMENT = 30010; // 选择搬运队的权限
const AH_SCAN_LOAD_TRUCK = 30011; // 扫描装车的权限

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
    AH_CREATE_TRUCK,
    AH_MODIFY_TRUCK,
    AH_REMOVE_TRUCK,
    AH_LOOK_TRUCK,
    SELECT_CARRY_PARTMENT,
    AH_SCAN_LOAD_TRUCK,

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
        [AH_CREATE_TRUCK]: '创建货车的权限',
        [AH_MODIFY_TRUCK]: '修改货车的权限',
        [AH_REMOVE_TRUCK]: '删除货车的权限',
        [AH_LOOK_TRUCK]: '查看货车的权限',
        [SELECT_CARRY_PARTMENT]: '选择搬运队的权限',
        [AH_SCAN_LOAD_TRUCK]: '扫描装车的权限',

        // 收货点
        [AH_MODIFY_AGENT_INFO]: '修改收货点信息的权限',
        [AH_MODIFY_AGENT_MEMBER_AUTHORITY]: '修改收货点成员权限的权限',
        [AH_PLACE_ORDER]: '收货点下单的权限',
    },
};

```

* PDShop/project/App/config/Color.js

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

* PDShop/project/App/config/Constants.js

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
    APP_NAME: '四面通分店端',
    THEME_COLORS: ['#c81622', '#A62045', '#239FDB'],
    ISSUE_IOS: CONFIG.ISSUE_IOS,
    NOT_NEED_UPDATE_JS_START: !(CONFIG.ISSUE || TEST_CONFIG.ISSUE), // 启动时不需要更新小版本
    MINIFY: CONFIG.ISSUE, // 是否压缩js文件，我们采取测试服务器为了查找问题不用压缩js文件，正式服务器需要压缩js文件，并且不能看到调试信息
    CHANNEL: CONFIG.CHANNEL,
    RATE: 1,  //倍率
    // IOS的appid,1096525384
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
    QRCODE_TYPE_DRIVER:2,
};

```

* PDShop/project/App/config/OrderState.js

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

module.exports = {
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

* PDShop/project/App/config/Partment.js

```js
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

* PDShop/project/App/config/Position.js

```js
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

* PDShop/project/App/config/Route.js

```js
'use strict';

const { BASE_SERVER } = CONSTANTS;
const ROOT_SERVER = 'http://' + BASE_SERVER + '/';
const API_SERVER = ROOT_SERVER + 'api/';
const SERVER = API_SERVER + 'shop/';

module.exports = {
    // 登录
    ROUTE_REGISTER: SERVER + 'register', // 注册
    ROUTE_LOGIN: SERVER + 'login', // 登录
    ROUTE_FIND_PASSWORD: SERVER + 'findPassword', // 忘记密码
    ROUTE_MODIFY_PASSWORD: SERVER + 'modifyPassword', // 修改密码

    // 个人中心
    ROUTE_GET_PERSONAL_INFO: SERVER + 'getPersonalInfo', // 获取个人信息
    ROUTE_MODIFY_PERSONAL_INFO: SERVER + 'modifyPersonalInfo', // 修改个人信息
    ROUTE_MODIFY_OWN_PARTMENT: SERVER + 'modifyOwnPartment', // 修改搬运队信息
    ROUTE_SUBMIT_FEEDBACK: SERVER + 'submitFeedback', // 提交信息反馈
    ROUTE_GET_REGION_ADDRESS_LIST: API_SERVER + 'getRegionAddress', // 获取地址列表
    ROUTE_GET_START_POINT_ADDRESS_LIST: API_SERVER + 'getStartPointAddress', // 获取地址列表
    ROUTE_GET_GETANDOUT_LIST: SERVER + 'getAndOutList', // 获取收支列表
    ROUTE_GET_SHIPPERLIST_LIST: SERVER + 'shipperList', // 获取分店列表
    ROUTE_GET_BILL_LIST: SERVER + 'getBillList', // 获取交易明细列表
    ROUTE_RECHARGE:SERVER + 'recharge', // 充值
    ROUTE_WITHDRAW:SERVER + 'withdraw', // 提现
    ROUTE_GET_REMAIN_AMOUNT: SERVER + 'getRemainAmount', // 获取账户余额
    ROUTE_SET_PAYMENT_PASSWORD: SERVER + 'setPaymentPassword', // 修改支付密码
    ROUTE_REQUEST_SEND_VERIFY_CODE: API_SERVER + 'requestSendVerifyCode', // 获取验证码
    ROUTE_MODIFY_SETTING_INFO:SERVER + 'modifySettingInfo', // 董事长修改各项参数

    // 分店
    ROUTE_GET_ORDERS_BY_WAREHOUSE_PARTMENT: SERVER + 'getOrdersByWareHousePartment', // 库管部获取货单
    ROUTE_GET_ORDERS_BY_RECEIVE_PARTMENT: SERVER + 'getOrdersByReceivePartment', // 收货部获取货单
    ROUTE_PLACE_STORAGE: SERVER + 'placeStorage', // 库管部扫描入库
    ROUTE_GET_LASTEST_ORDER_BY_RECIEVE_PARTMENT: SERVER + 'getLastestOrderByReceivePartment', // 收货部获取未上传照片的货单
    ROUTE_SET_PHOTO_FOR_ORIGIN_ORDER: SERVER + 'setPhotoForOriginOrder', // 收货部为货单上传照片
    ROUTE_GET_MEMBER_LIST: SERVER + 'getMemberList', // 获取成员列表
    ROUTE_CREATE_MEMBER: SERVER + 'createMember', // 创建成员
    ROUTE_GET_OWN_PARTMENT_INFO: SERVER + 'getOwnPartmentInfo', // 获取部门信息
    ROUTE_GET_ORDER_DETAIL: SERVER + 'getOrderDetail', // 收货部货单详情
    ROUTE_PLACE_TRANSFER_ORDER: SERVER + 'placeTransferOrder', // 收货部扫描下中转货单
    ROUTE_CHECK_TRUCK_PASS: SERVER + 'checkTruckPass', // 保安部检查放行
    ROUTE_MODIFY_MEMBER: SERVER + 'modifyMember', // 分店创建成员
    ROUTE_GET_WAIT_SCAN_TRUCK_INFO: SERVER + 'getWaitScanTruckInfo', // 获取等待扫描装车的货车信息
    ROUTE_GET_CARRY_LIST: SERVER + 'getCarryList', // 获取搬运历史消息
    ROUTE_GET_ORDER_LIST_IN_RUCKS: SERVER + 'getOrderListInTruck', // 获取搬运货车货单列表
    ROUTE_GET_STATISTICS : SERVER + 'getStatistics', // 获取统计数据

    // 物流公司
    ROUTE_GET_SHIPPER_LIST: SERVER + 'getShipperList', // 获取物流公司列表
    ROUTE_GET_SHIPPER_DETAIL: SERVER + 'getShipperDetail', // 获取物流公司详情
    ROUTE_CARE_SHIPPER: SERVER + 'careShipper', // 关注物流公司
    ROUTE_GET_CARED_SHIPPER_LIST: SERVER + 'getCaredShipperList', // 获取关注物流公司列表

    // 路线
    ROUTE_GET_ROADMAP_LIST: SERVER + 'getRoadmapList', // 获取路线列表
    ROUTE_GET_ROADMAP_DETAIL: SERVER + 'getRoadmapDetail', // 获取路线详情
    ROUTE_GET_COLLECTION_ROADMAP_LIST: SERVER + 'getCollectionRoadmapList', // 获取收藏线路列表
    ROUTE_GET_ROADMAP_LIST_IN_SHOP: SERVER + 'getRoadmapListInShop', // 获取分店线路列表
    ROUTE_GET_ROADMAP_LIST_IN_AGENT: SERVER + 'getRoadmapListInAgent', // 获取收货点线路列表
    ROUTE_GET_ROADMAP_LIST_WITH_PRE_ORDER: SERVER + 'getRoadmapListWithPreOrder', // 获取区域收货点和分店路线列表
    ROUTE_GET_LOGISTICS_LIST: SERVER + 'getLogisticsList', // 获取订单物流信息

    // 用户
    ROUTE_GET_CLIENT_LIST: SERVER + 'getClientList', // 获取用户列表

    // 订单
    ROUTE_PLACE_PRE_ORDER: SERVER + 'placePreOrder', // 预下单
    ROUTE_PLACE_ORDER: SERVER + 'placeOrder', // 下单
    ROUTE_GET_SEND_ORDER_LIST: SERVER + 'getSendOrderList', // 获取发出订单列表
    ROUTE_GET_RECEIVE_ORDER_LIST: SERVER + 'getReceiveOrderList', // 获取发出订单列表
    ROUTE_GET_ORDER_LIST: SERVER + 'getOrderList', // 获取订单列表
    ROUTE_MODIFY_ORDER: SERVER + 'modifyOrder', // 修改订单
    ROUTE_MODIFY_PRE_ORDER: SERVER + 'modifyPreOrder', // 修改预下单
    ROUTE_REMOVE_PRE_ORDER: SERVER + 'removePreOrder', // 删除订单
    ROUTE_PAY_FOR_ORDER_WHEN_SEND: SERVER + 'payForOrderWhenSend', // 线上立即支付(发货)
    ROUTE_PAY_FOR_ORDER_WHEN_RECEIVE: SERVER + 'payForOrderWhenReceive', // 线上立即支付（收货）
    ROUTE_FINISH_ORDER: SERVER + 'finishOrder', // 现付成功

    // 文件上传
    ROUTE_UPDATE_FILE: API_SERVER + 'uploadFile', // 上传文件

    // 网页地址
    ROUTE_USER_LICENSE: ROOT_SERVER + 'protocals/pdshop/user.html', // 用户协议
    ROUTE_SOFTWARE_LICENSE: ROOT_SERVER + 'protocals/pdshop/software.html', // 获取软件许可协议
    ROUTE_ABOUT_PAGE: ROOT_SERVER + 'protocals/pdshop/about.html', // 关于

    // 下载更新
    ROUTE_VERSION_INFO_URL: ROOT_SERVER + 'apps/pdshop/version.json', // 版本信息地址
    ROUTE_JS_ANDROID_URL: ROOT_SERVER + 'apps/pdshop/jsandroid.zip', // android jsbundle 包地址
    ROUTE_JS_IOS_URL: ROOT_SERVER + 'apps/pdshop/jsios.zip', // ios jsbundle 包地址
    ROUTE_APK_URL: ROOT_SERVER + 'apps/pdshop/pdshop.apk', // apk地址
};

```

* PDShop/project/App/config/Screen.js

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

* PDShop/project/App/config/TruckState.js

```js
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

* PDShop/project/App/data/city.js

```js
module.exports = [{ name:'北京', city:[{ name:'北京', area:['东城区', '西城区', '崇文区', '宣武区', '朝阳区', '丰台区', '石景山区', '海淀区', '门头沟区', '房山区', '通州区', '顺义区', '昌平区', '大兴区', '平谷区', '怀柔区', '密云县', '延庆县'] }] }, { name:'天津', city:[{ name:'天津', area:['和平区', '河东区', '河西区', '南开区', '河北区', '红桥区', '塘沽区', '汉沽区', '大港区', '东丽区', '西青区', '津南区', '北辰区', '武清区', '宝坻区', '宁河县', '静海县', '蓟  县'] }] }, { name:'河北', city:[{ name:'石家庄', area:['长安区', '桥东区', '桥西区', '新华区', '郊  区', '井陉矿区', '井陉县', '正定县', '栾城县', '行唐县', '灵寿县', '高邑县', '深泽县', '赞皇县', '无极县', '平山县', '元氏县', '赵  县', '辛集市', '藁', '晋州市', '新乐市', '鹿泉市'] }, { name:'唐山', area:['路南区', '路北区', '古冶区', '开平区', '新  区', '丰润县', '滦  县', '滦南县', '乐亭县', '迁西县', '玉田县', '唐海县', '遵化市', '丰南市', '迁安市'] }, { name:'秦皇岛', area:['海港区', '山海关区', '北戴河区', '青龙满族自治县', '昌黎县', '抚宁县', '卢龙县'] }, { name:'邯郸', area:['邯山区', '丛台区', '复兴区', '峰峰矿区', '邯郸县', '临漳县', '成安县', '大名县', '涉  县', '磁  县', '肥乡县', '永年县', '邱  县', '鸡泽县', '广平县', '馆陶县', '魏  县', '曲周县', '武安市'] }, { name:'邢台', area:['桥东区', '桥西区', '邢台县', '临城县', '内丘县', '柏乡县', '隆尧县', '任  县', '南和县', '宁晋县', '巨鹿县', '新河县', '广宗县', '平乡县', '威  县', '清河县', '临西县', '南宫市', '沙河市'] }, { name:'保定', area:['新市区', '北市区', '南市区', '满城县', '清苑县', '涞水县', '阜平县', '徐水县', '定兴县', '唐  县', '高阳县', '容城县', '涞源县', '望都县', '安新县', '易  县', '曲阳县', '蠡  县', '顺平县', '博野', '雄县', '涿州市', '定州市', '安国市', '高碑店市'] }, { name:'张家口', area:['桥东区', '桥西区', '宣化区', '下花园区', '宣化县', '张北县', '康保县', '沽源县', '尚义县', '蔚  县', '阳原县', '怀安县', '万全县', '怀来县', '涿鹿县', '赤城县', '崇礼县'] }, { name:'承德', area:['双桥区', '双滦区', '鹰手营子矿区', '承德县', '兴隆县', '平泉县', '滦平县', '隆化县', '丰宁满族自治县', '宽城满族自治县', '围场满族蒙古族自治县'] }, { name:'沧州', area:['新华区', '运河区', '沧  县', '青  县', '东光县', '海兴县', '盐山县', '肃宁县', '南皮县', '吴桥县', '献  县', '孟村回族自治县', '泊头市', '任丘市', '黄骅市', '河间市'] }, { name:'廊坊', area:['安次区', '固安县', '永清县', '香河县', '大城县', '文安县', '大厂回族自治县', '霸州市', '三河市'] }, { name:'衡水', area:['桃城区', '枣强县', '武邑县', '武强县', '饶阳县', '安平县', '故城县', '景  县', '阜城县', '冀州市', '深州市'] }] }, { name:'山西', city:[{ name:'太原', area:['小店区', '迎泽区', '杏花岭区', '尖草坪区', '万柏林区', '晋源区', '清徐县', '阳曲县', '娄烦县', '古交市'] }, { name:'大同', area:['城  区', '矿  区', '南郊区', '新荣区', '阳高县', '天镇县', '广灵县', '灵丘县', '浑源县', '左云县', '大同县'] }, { name:'阳泉', area:['城  区', '矿  区', '郊  区', '平定县', '盂  县'] }, { name:'长治', area:['城  区', '郊  区', '长治县', '襄垣县', '屯留县', '平顺县', '黎城县', '壶关县', '长子县', '武乡县', '沁  县', '沁源县', '潞城市'] }, { name:'晋城', area:['城  区', '沁水县', '阳城县', '陵川县', '泽州县', '高平市'] }, { name:'朔州', area:['朔城区', '平鲁区', '山阴县', '应  县', '右玉县', '怀仁县'] }, { name:'忻州', area:['忻府区', '原平市', '定襄县', '五台县', '代  县', '繁峙县', '宁武县', '静乐县', '神池县', '五寨县', '岢岚县', '河曲县', '保德县', '偏关县'] }, { name:'吕梁', area:['离石区', '孝义市', '汾阳市', '文水县', '交城县', '兴  县', '临  县', '柳林县', '石楼县', '岚  县', '方山县', '中阳县', '交口县'] }, { name:'晋中', area:['榆次市', '介休市', '榆社县', '左权县', '和顺县', '昔阳县', '寿阳县', '太谷县', '祁  县', '平遥县', '灵石县'] }, { name:'临汾', area:['临汾市', '侯马市', '霍州市', '曲沃县', '翼城县', '襄汾县', '洪洞县', '古  县', '安泽县', '浮山县', '吉  县', '乡宁县', '蒲  县', '大宁县', '永和县', '隰  县', '汾西县'] }, { name:'运城', area:['运城市', '永济市', '河津市', '芮城县', '临猗县', '万荣县', '新绛县', '稷山县', '闻喜县', '夏  县', '绛  县', '平陆县', '垣曲县'] }] }, { name:'内蒙古', city:[{ name:'呼和浩特', area:['新城区', '回民区', '玉泉区', '郊  区', '土默特左旗', '托克托县', '和林格尔县', '清水河县', '武川县'] }, { name:'包头', area:['东河区', '昆都伦区', '青山区', '石拐矿区', '白云矿区', '郊  区', '土默特右旗', '固阳县', '达尔罕茂明安联合旗'] }, { name:'乌海', area:['海勃湾区', '海南区', '乌达区'] }, { name:'赤峰', area:['红山区', '元宝山区', '松山区', '阿鲁科尔沁旗', '巴林左旗', '巴林右旗', '林西县', '克什克腾旗', '翁牛特旗', '喀喇沁旗', '宁城县', '敖汉旗'] }, { name:'呼伦贝尔', area:['海拉尔市', '满洲里市', '扎兰屯市', '牙克石市', '根河市', '额尔古纳市', '阿荣旗', '莫力达瓦达斡尔族自治旗', '鄂伦春自治旗', '鄂温克族自治旗', '新巴尔虎右旗', '新巴尔虎左旗', '陈巴尔虎旗'] }, { name:'兴安盟', area:['乌兰浩特市', '阿尔山市', '科尔沁右翼前旗', '科尔沁右翼中旗', '扎赉特旗', '突泉县'] }, { name:'通辽', area:['科尔沁区', '霍林郭勒市', '科尔沁左翼中旗', '科尔沁左翼后旗', '开鲁县', '库伦旗', '奈曼旗', '扎鲁特旗'] }, { name:'锡林郭勒盟', area:['二连浩特市', '锡林浩特市', '阿巴嘎旗', '苏尼特左旗', '苏尼特右旗', '东乌珠穆沁旗', '西乌珠穆沁旗', '太仆寺旗', '镶黄旗', '正镶白旗', '正蓝旗', '多伦县'] }, { name:'乌兰察布盟', area:['集宁市', '丰镇市', '卓资县', '化德县', '商都县', '兴和县', '凉城县', '察哈尔右翼前旗', '察哈尔右翼中旗', '察哈尔右翼后旗', '四子王旗'] }, { name:'伊克昭盟', area:['东胜市', '达拉特旗', '准格尔旗', '鄂托克前旗', '鄂托克旗', '杭锦旗', '乌审旗', '伊金霍洛旗'] }, { name:'巴彦淖尔盟', area:['临河市', '五原县', '磴口县', '乌拉特前旗', '乌拉特中旗', '乌拉特后旗', '杭锦后旗'] }, { name:'阿拉善盟', area:['阿拉善左旗', '阿拉善右旗', '额济纳旗'] }] }, { name:'辽宁', city:[{ name:'沈阳', area:['沈河区', '皇姑区', '和平区', '大东区', '铁西区', '苏家屯区', '东陵区', '于洪区', '新民市', '法库县', '辽中县', '康平县', '新城子区', '其他'] }, { name:'大连', area:['西岗区', '中山区', '沙河口区', '甘井子区', '旅顺口区', '金州区', '瓦房店市', '普兰店市', '庄河市', '长海县', '其他'] }, { name:'鞍山', area:['铁东区', '铁西区', '立山区', '千山区', '海城市', '台安县', '岫岩满族自治县', '其他'] }, { name:'抚顺', area:['顺城区', '新抚区', '东洲区', '望花区', '抚顺县', '清原满族自治县', '新宾满族自治县', '其他'] }, { name:'本溪', area:['平山区', '明山区', '溪湖区', '南芬区', '本溪满族自治县', '桓仁满族自治县', '其他'] }, { name:'丹东', area:['振兴区', '元宝区', '振安区', '东港市', '凤城市', '宽甸满族自治县', '其他'] }, { name:'锦州', area:['太和区', '古塔区', '凌河区', '凌海市', '黑山县', '义县', '北宁市', '其他'] }, { name:'营口', area:['站前区', '西市区', '鲅鱼圈区', '老边区', '大石桥市', '盖州市', '其他'] }, { name:'阜新', area:['海州区', '新邱区', '太平区', '清河门区', '细河区', '彰武县', '阜新蒙古族自治县', '其他'] }, { name:'辽阳', area:['白塔区', '文圣区', '宏伟区', '太子河区', '弓长岭区', '灯塔市', '辽阳县', '其他'] }, { name:'盘锦', area:['双台子区', '兴隆台区', '盘山县', '大洼县', '其他'] }, { name:'铁岭', area:['银州区', '清河区', '调兵山市', '开原市', '铁岭县', '昌图县', '西丰县', '其他'] }, { name:'朝阳', area:['双塔区', '龙城区', '凌源市', '北票市', '朝阳县', '建平县', '喀喇沁左翼蒙古族自治县', '其他'] }, { name:'葫芦岛', area:['龙港区', '南票区', '连山区', '兴城市', '绥中县', '建昌县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'吉林', city:[{ name:'长春', area:['朝阳区', '宽城区', '二道区', '南关区', '绿园区', '双阳区', '九台市', '榆树市', '德惠市', '农安县', '其他'] }, { name:'吉林', area:['船营区', '昌邑区', '龙潭区', '丰满区', '舒兰市', '桦甸市', '蛟河市', '磐石市', '永吉县', '其他'] }, { name:'四平', area:['铁西区', '铁东区', '公主岭市', '双辽市', '梨树县', '伊通满族自治县', '其他'] }, { name:'辽源', area:['龙山区', '西安区', '东辽县', '东丰县', '其他'] }, { name:'通化', area:['东昌区', '二道江区', '梅河口市', '集安市', '通化县', '辉南县', '柳河县', '其他'] }, { name:'白山', area:['八道江区', '江源区', '临江市', '靖宇县', '抚松县', '长白朝鲜族自治县', '其他'] }, { name:'松原', area:['宁江区', '乾安县', '长岭县', '扶余县', '前郭尔罗斯蒙古族自治县', '其他'] }, { name:'白城', area:['洮北区', '大安市', '洮南市', '镇赉县', '通榆县', '其他'] }, { name:'延边朝鲜族自治州', area:['延吉市', '图们市', '敦化市', '龙井市', '珲春市', '和龙市', '安图县', '汪清县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'黑龙江', city:[{ name:'哈尔滨', area:['松北区', '道里区', '南岗区', '平房区', '香坊区', '道外区', '呼兰区', '阿城区', '双城市', '尚志市', '五常市', '宾县', '方正县', '通河县', '巴彦县', '延寿县', '木兰县', '依兰县', '其他'] }, { name:'齐齐哈尔', area:['龙沙区', '昂昂溪区', '铁锋区', '建华区', '富拉尔基区', '碾子山区', '梅里斯达斡尔族区', '讷河市', '富裕县', '拜泉县', '甘南县', '依安县', '克山县', '泰来县', '克东县', '龙江县', '其他'] }, { name:'鹤岗', area:['兴山区', '工农区', '南山区', '兴安区', '向阳区', '东山区', '萝北县', '绥滨县', '其他'] }, { name:'双鸭山', area:['尖山区', '岭东区', '四方台区', '宝山区', '集贤县', '宝清县', '友谊县', '饶河县', '其他'] }, { name:'鸡西', area:['鸡冠区', '恒山区', '城子河区', '滴道区', '梨树区', '麻山区', '密山市', '虎林市', '鸡东县', '其他'] }, { name:'大庆', area:['萨尔图区', '红岗区', '龙凤区', '让胡路区', '大同区', '林甸县', '肇州县', '肇源县', '杜尔伯特蒙古族自治县', '其他'] }, { name:'伊春', area:['伊春区', '带岭区', '南岔区', '金山屯区', '西林区', '美溪区', '乌马河区', '翠峦区', '友好区', '上甘岭区', '五营区', '红星区', '新青区', '汤旺河区', '乌伊岭区', '铁力市', '嘉荫县', '其他'] }, { name:'牡丹江', area:['爱民区', '东安区', '阳明区', '西安区', '绥芬河市', '宁安市', '海林市', '穆棱市', '林口县', '东宁县', '其他'] }, { name:'佳木斯', area:['向阳区', '前进区', '东风区', '郊区', '同江市', '富锦市', '桦川县', '抚远县', '桦南县', '汤原县', '其他'] }, { name:'七台河', area:['桃山区', '新兴区', '茄子河区', '勃利县', '其他'] }, { name:'黑河', area:['爱辉区', '北安市', '五大连池市', '逊克县', '嫩江县', '孙吴县', '其他'] }, { name:'绥化', area:['北林区', '安达市', '肇东市', '海伦市', '绥棱县', '兰西县', '明水县', '青冈县', '庆安县', '望奎县', '其他'] }, { name:'大兴安岭地区', area:['呼玛县', '塔河县', '漠河县', '大兴安岭辖区', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'上海', city:[{ name:'上海', area:['黄浦区', '卢湾区', '徐汇区', '长宁区', '静安区', '普陀区', '闸北区', '虹口区', '杨浦区', '宝山区', '闵行区', '嘉定区', '松江区', '金山区', '青浦区', '南汇区', '奉贤区', '浦东新区', '崇明县', '其他'] }] }, { name:'江苏', city:[{ name:'南京', area:['玄武区', '白下区', '秦淮区', '建邺区', '鼓楼区', '下关区', '栖霞区', '雨花台区', '浦口区', '江宁区', '六合区', '溧水县', '高淳县', '其他'] }, { name:'苏州', area:['金阊区', '平江区', '沧浪区', '虎丘区', '吴中区', '相城区', '常熟市', '张家港市', '昆山市', '吴江市', '太仓市', '其他'] }, { name:'无锡', area:['崇安区', '南长区', '北塘区', '滨湖区', '锡山区', '惠山区', '江阴市', '宜兴市', '其他'] }, { name:'常州', area:['钟楼区', '天宁区', '戚墅堰区', '新北区', '武进区', '金坛市', '溧阳市', '其他'] }, { name:'镇江', area:['京口区', '润州区', '丹徒区', '丹阳市', '扬中市', '句容市', '其他'] }, { name:'南通', area:['崇川区', '港闸区', '通州市', '如皋市', '海门市', '启东市', '海安县', '如东县', '其他'] }, { name:'泰州', area:['海陵区', '高港区', '姜堰市', '泰兴市', '靖江市', '兴化市', '其他'] }, { name:'扬州', area:['广陵区', '维扬区', '邗江区', '江都市', '仪征市', '高邮市', '宝应县', '其他'] }, { name:'盐城', area:['亭湖区', '盐都区', '大丰市', '东台市', '建湖县', '射阳县', '阜宁县', '滨海县', '响水县', '其他'] }, { name:'连云港', area:['新浦区', '海州区', '连云区', '东海县', '灌云县', '赣榆县', '灌南县', '其他'] }, { name:'徐州', area:['云龙区', '鼓楼区', '九里区', '泉山区', '贾汪区', '邳州市', '新沂市', '铜山县', '睢宁县', '沛县', '丰县', '其他'] }, { name:'淮安', area:['清河区', '清浦区', '楚州区', '淮阴区', '涟水县', '洪泽县', '金湖县', '盱眙县', '其他'] }, { name:'宿迁', area:['宿城区', '宿豫区', '沭阳县', '泗阳县', '泗洪县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'浙江', city:[{ name:'杭州', area:['拱墅区', '西湖区', '上城区', '下城区', '江干区', '滨江区', '余杭区', '萧山区', '建德市', '富阳市', '临安市', '桐庐县', '淳安县', '其他'] }, { name:'宁波', area:['海曙区', '江东区', '江北区', '镇海区', '北仑区', '鄞州区', '余姚市', '慈溪市', '奉化市', '宁海县', '象山县', '其他'] }, { name:'温州', area:['鹿城区', '龙湾区', '瓯海区', '瑞安市', '乐清市', '永嘉县', '洞头县', '平阳县', '苍南县', '文成县', '泰顺县', '其他'] }, { name:'嘉兴', area:['秀城区', '秀洲区', '海宁市', '平湖市', '桐乡市', '嘉善县', '海盐县', '其他'] }, { name:'湖州', area:['吴兴区', '南浔区', '长兴县', '德清县', '安吉县', '其他'] }, { name:'绍兴', area:['越城区', '诸暨市', '上虞市', '嵊州市', '绍兴县', '新昌县', '其他'] }, { name:'金华', area:['婺城区', '金东区', '兰溪市', '义乌市', '东阳市', '永康市', '武义县', '浦江县', '磐安县', '其他'] }, { name:'衢州', area:['柯城区', '衢江区', '江山市', '龙游县', '常山县', '开化县', '其他'] }, { name:'舟山', area:['定海区', '普陀区', '岱山县', '嵊泗县', '其他'] }, { name:'台州', area:['椒江区', '黄岩区', '路桥区', '临海市', '温岭市', '玉环县', '天台县', '仙居县', '三门县', '其他'] }, { name:'丽水', area:['莲都区', '龙泉市', '缙云县', '青田县', '云和县', '遂昌县', '松阳县', '庆元县', '景宁畲族自治县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'安徽', city:[{ name:'合肥', area:['庐阳区', '瑶海区', '蜀山区', '包河区', '长丰县', '肥东县', '肥西县', '其他'] }, { name:'芜湖', area:['镜湖区', '弋江区', '鸠江区', '三山区', '芜湖县', '南陵县', '繁昌县', '其他'] }, { name:'蚌埠', area:['蚌山区', '龙子湖区', '禹会区', '淮上区', '怀远县', '固镇县', '五河县', '其他'] }, { name:'淮南', area:['田家庵区', '大通区', '谢家集区', '八公山区', '潘集区', '凤台县', '其他'] }, { name:'马鞍山', area:['雨山区', '花山区', '金家庄区', '当涂县', '其他'] }, { name:'淮北', area:['相山区', '杜集区', '烈山区', '濉溪县', '其他'] }, { name:'铜陵', area:['铜官山区', '狮子山区', '郊区', '铜陵县', '其他'] }, { name:'安庆', area:['迎江区', '大观区', '宜秀区', '桐城市', '宿松县', '枞阳县', '太湖县', '怀宁县', '岳西县', '望江县', '潜山县', '其他'] }, { name:'黄山', area:['屯溪区', '黄山区', '徽州区', '休宁县', '歙县', '祁门县', '黟县', '其他'] }, { name:'滁州', area:['琅琊区', '南谯区', '天长市', '明光市', '全椒县', '来安县', '定远县', '凤阳县', '其他'] }, { name:'阜阳', area:['颍州区', '颍东区', '颍泉区', '界首市', '临泉县', '颍上县', '阜南县', '太和县', '其他'] }, { name:'宿州', area:['埇桥区', '萧县', '泗县', '砀山县', '灵璧县', '其他'] }, { name:'巢湖', area:['居巢区', '含山县', '无为县', '庐江县', '和县', '其他'] }, { name:'六安', area:['金安区', '裕安区', '寿县', '霍山县', '霍邱县', '舒城县', '金寨县', '其他'] }, { name:'亳州', area:['谯城区', '利辛县', '涡阳县', '蒙城县', '其他'] }, { name:'池州', area:['贵池区', '东至县', '石台县', '青阳县', '其他'] }, { name:'宣城', area:['宣州区', '宁国市', '广德县', '郎溪县', '泾县', '旌德县', '绩溪县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'福建', city:[{ name:'福州', area:['鼓楼区', '台江区', '仓山区', '马尾区', '晋安区', '福清市', '长乐市', '闽侯县', '闽清县', '永泰县', '连江县', '罗源县', '平潭县', '其他'] }, { name:'厦门', area:['思明区', '海沧区', '湖里区', '集美区', '同安区', '翔安区', '其他'] }, { name:'莆田', area:['城厢区', '涵江区', '荔城区', '秀屿区', '仙游县', '其他'] }, { name:'三明', area:['梅列区', '三元区', '永安市', '明溪县', '将乐县', '大田县', '宁化县', '建宁县', '沙县', '尤溪县', '清流县', '泰宁县', '其他'] }, { name:'泉州', area:['鲤城区', '丰泽区', '洛江区', '泉港区', '石狮市', '晋江市', '南安市', '惠安县', '永春县', '安溪县', '德化县', '金门县', '其他'] }, { name:'漳州', area:['芗城区', '龙文区', '龙海市', '平和县', '南靖县', '诏安县', '漳浦县', '华安县', '东山县', '长泰县', '云霄县', '其他'] }, { name:'南平', area:['延平区', '建瓯市', '邵武市', '武夷山市', '建阳市', '松溪县', '光泽县', '顺昌县', '浦城县', '政和县', '其他'] }, { name:'龙岩', area:['新罗区', '漳平市', '长汀县', '武平县', '上杭县', '永定县', '连城县', '其他'] }, { name:'宁德', area:['蕉城区', '福安市', '福鼎市', '寿宁县', '霞浦县', '柘荣县', '屏南县', '古田县', '周宁县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'江西', city:[{ name:'南昌', area:['东湖区', '西湖区', '青云谱区', '湾里区', '青山湖区', '新建县', '南昌县', '进贤县', '安义县', '其他'] }, { name:'景德镇', area:['珠山区', '昌江区', '乐平市', '浮梁县', '其他'] }, { name:'萍乡', area:['安源区', '湘东区', '莲花县', '上栗县', '芦溪县', '其他'] }, { name:'九江', area:['浔阳区', '庐山区', '瑞昌市', '九江县', '星子县', '武宁县', '彭泽县', '永修县', '修水县', '湖口县', '德安县', '都昌县', '其他'] }, { name:'新余', area:['渝水区', '分宜县', '其他'] }, { name:'鹰潭', area:['月湖区', '贵溪市', '余江县', '其他'] }, { name:'赣州', area:['章贡区', '瑞金市', '南康市', '石城县', '安远县', '赣县', '宁都县', '寻乌县', '兴国县', '定南县', '上犹县', '于都县', '龙南县', '崇义县', '信丰县', '全南县', '大余县', '会昌县', '其他'] }, { name:'吉安', area:['吉州区', '青原区', '井冈山市', '吉安县', '永丰县', '永新县', '新干县', '泰和县', '峡江县', '遂川县', '安福县', '吉水县', '万安县', '其他'] }, { name:'宜春', area:['袁州区', '丰城市', '樟树市', '高安市', '铜鼓县', '靖安县', '宜丰县', '奉新县', '万载县', '上高县', '其他'] }, { name:'抚州', area:['临川区', '南丰县', '乐安县', '金溪县', '南城县', '东乡县', '资溪县', '宜黄县', '广昌县', '黎川县', '崇仁县', '其他'] }, { name:'上饶', area:['信州区', '德兴市', '上饶县', '广丰县', '鄱阳县', '婺源县', '铅山县', '余干县', '横峰县', '弋阳县', '玉山县', '万年县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'山东', city:[{ name:'济南', area:['市中区', '历下区', '天桥区', '槐荫区', '历城区', '长清区', '章丘市', '平阴县', '济阳县', '商河县', '其他'] }, { name:'青岛', area:['市南区', '市北区', '城阳区', '四方区', '李沧区', '黄岛区', '崂山区', '胶南市', '胶州市', '平度市', '莱西市', '即墨市', '其他'] }, { name:'淄博', area:['张店区', '临淄区', '淄川区', '博山区', '周村区', '桓台县', '高青县', '沂源县', '其他'] }, { name:'枣庄', area:['市中区', '山亭区', '峄城区', '台儿庄区', '薛城区', '滕州市', '其他'] }, { name:'东营', area:['东营区', '河口区', '垦利县', '广饶县', '利津县', '其他'] }, { name:'烟台', area:['芝罘区', '福山区', '牟平区', '莱山区', '龙口市', '莱阳市', '莱州市', '招远市', '蓬莱市', '栖霞市', '海阳市', '长岛县', '其他'] }, { name:'潍坊', area:['潍城区', '寒亭区', '坊子区', '奎文区', '青州市', '诸城市', '寿光市', '安丘市', '高密市', '昌邑市', '昌乐县', '临朐县', '其他'] }, { name:'济宁', area:['市中区', '任城区', '曲阜市', '兖州市', '邹城市', '鱼台县', '金乡县', '嘉祥县', '微山县', '汶上县', '泗水县', '梁山县', '其他'] }, { name:'泰安', area:['泰山区', '岱岳区', '新泰市', '肥城市', '宁阳县', '东平县', '其他'] }, { name:'威海', area:['环翠区', '乳山市', '文登市', '荣成市', '其他'] }, { name:'日照', area:['东港区', '岚山区', '五莲县', '莒县', '其他'] }, { name:'莱芜', area:['莱城区', '钢城区', '其他'] }, { name:'临沂', area:['兰山区', '罗庄区', '河东区', '沂南县', '郯城县', '沂水县', '苍山县', '费县', '平邑县', '莒南县', '蒙阴县', '临沭县', '其他'] }, { name:'德州', area:['德城区', '乐陵市', '禹城市', '陵县', '宁津县', '齐河县', '武城县', '庆云县', '平原县', '夏津县', '临邑县', '其他'] }, { name:'聊城', area:['东昌府区', '临清市', '高唐县', '阳谷县', '茌平县', '莘县', '东阿县', '冠县', '其他'] }, { name:'滨州', area:['滨城区', '邹平县', '沾化县', '惠民县', '博兴县', '阳信县', '无棣县', '其他'] }, { name:'菏泽', area:['牡丹区', '鄄城县', '单县', '郓城县', '曹县', '定陶县', '巨野县', '东明县', '成武县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'河南', city:[{ name:'郑州', area:['中原区', '金水区', '二七区', '管城回族区', '上街区', '惠济区', '巩义市', '新郑市', '新密市', '登封市', '荥阳市', '中牟县', '其他'] }, { name:'开封', area:['鼓楼区', '龙亭区', '顺河回族区', '禹王台区', '金明区', '开封县', '尉氏县', '兰考县', '杞县', '通许县', '其他'] }, { name:'洛阳', area:['西工区', '老城区', '涧西区', '瀍河回族区', '洛龙区', '吉利区', '偃师市', '孟津县', '汝阳县', '伊川县', '洛宁县', '嵩县', '宜阳县', '新安县', '栾川县', '其他'] }, { name:'平顶山', area:['新华区', '卫东区', '湛河区', '石龙区', '汝州市', '舞钢市', '宝丰县', '叶县', '郏县', '鲁山县', '其他'] }, { name:'安阳', area:['北关区', '文峰区', '殷都区', '龙安区', '林州市', '安阳县', '滑县', '内黄县', '汤阴县', '其他'] }, { name:'鹤壁', area:['淇滨区', '山城区', '鹤山区', '浚县', '淇县', '其他'] }, { name:'新乡', area:['卫滨区', '红旗区', '凤泉区', '牧野区', '卫辉市', '辉县市', '新乡县', '获嘉县', '原阳县', '长垣县', '封丘县', '延津县', '其他'] }, { name:'焦作', area:['解放区', '中站区', '马村区', '山阳区', '沁阳市', '孟州市', '修武县', '温县', '武陟县', '博爱县', '其他'] }, { name:'濮阳', area:['华龙区', '濮阳县', '南乐县', '台前县', '清丰县', '范县', '其他'] }, { name:'许昌', area:['魏都区', '禹州市', '长葛市', '许昌县', '鄢陵县', '襄城县', '其他'] }, { name:'漯河', area:['源汇区', '郾城区', '召陵区', '临颍县', '舞阳县', '其他'] }, { name:'三门峡', area:['湖滨区', '义马市', '灵宝市', '渑池县', '卢氏县', '陕县', '其他'] }, { name:'南阳', area:['卧龙区', '宛城区', '邓州市', '桐柏县', '方城县', '淅川县', '镇平县', '唐河县', '南召县', '内乡县', '新野县', '社旗县', '西峡县', '其他'] }, { name:'商丘', area:['梁园区', '睢阳区', '永城市', '宁陵县', '虞城县', '民权县', '夏邑县', '柘城县', '睢县', '其他'] }, { name:'信阳', area:['浉河区', '平桥区', '潢川县', '淮滨县', '息县', '新县', '商城县', '固始县', '罗山县', '光山县', '其他'] }, { name:'周口', area:['川汇区', '项城市', '商水县', '淮阳县', '太康县', '鹿邑县', '西华县', '扶沟县', '沈丘县', '郸城县', '其他'] }, { name:'驻马店', area:['驿城区', '确山县', '新蔡县', '上蔡县', '西平县', '泌阳县', '平舆县', '汝南县', '遂平县', '正阳县', '其他'] }, { name:'焦作', area:['济源市', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'湖北', city:[{ name:'武汉', area:['江岸区', '武昌区', '江汉区', '硚口区', '汉阳区', '青山区', '洪山区', '东西湖区', '汉南区', '蔡甸区', '江夏区', '黄陂区', '新洲区', '其他'] }, { name:'黄石', area:['黄石港区', '西塞山区', '下陆区', '铁山区', '大冶市', '阳新县', '其他'] }, { name:'十堰', area:['张湾区', '茅箭区', '丹江口市', '郧县', '竹山县', '房县', '郧西县', '竹溪县', '其他'] }, { name:'荆州', area:['沙市区', '荆州区', '洪湖市', '石首市', '松滋市', '监利县', '公安县', '江陵县', '其他'] }, { name:'宜昌', area:['西陵区', '伍家岗区', '点军区', '猇亭区', '夷陵区', '宜都市', '当阳市', '枝江市', '秭归县', '远安县', '兴山县', '五峰土家族自治县', '长阳土家族自治县', '其他'] }, { name:'襄樊', area:['襄城区', '樊城区', '襄阳区', '老河口市', '枣阳市', '宜城市', '南漳县', '谷城县', '保康县', '其他'] }, { name:'鄂州', area:['鄂城区', '华容区', '梁子湖区', '其他'] }, { name:'荆门', area:['东宝区', '掇刀区', '钟祥市', '京山县', '沙洋县', '其他'] }, { name:'孝感', area:['孝南区', '应城市', '安陆市', '汉川市', '云梦县', '大悟县', '孝昌县', '其他'] }, { name:'黄冈', area:['黄州区', '麻城市', '武穴市', '红安县', '罗田县', '浠水县', '蕲春县', '黄梅县', '英山县', '团风县', '其他'] }, { name:'咸宁', area:['咸安区', '赤壁市', '嘉鱼县', '通山县', '崇阳县', '通城县', '其他'] }, { name:'随州', area:['曾都区', '广水市', '其他'] }, { name:'恩施土家族苗族自治州', area:['恩施市', '利川市', '建始县', '来凤县', '巴东县', '鹤峰县', '宣恩县', '咸丰县', '其他'] }, { name:'仙桃', area:['仙桃'] }, { name:'天门', area:['天门'] }, { name:'潜江', area:['潜江'] }, { name:'神农架林区', area:['神农架林区'] }, { name:'其他', area:['其他'] }] }, { name:'湖南', city:[{ name:'长沙', area:['岳麓区', '芙蓉区', '天心区', '开福区', '雨花区', '浏阳市', '长沙县', '望城县', '宁乡县', '其他'] }, { name:'株洲', area:['天元区', '荷塘区', '芦淞区', '石峰区', '醴陵市', '株洲县', '炎陵县', '茶陵县', '攸县', '其他'] }, { name:'湘潭', area:['岳塘区', '雨湖区', '湘乡市', '韶山市', '湘潭县', '其他'] }, { name:'衡阳', area:['雁峰区', '珠晖区', '石鼓区', '蒸湘区', '南岳区', '耒阳市', '常宁市', '衡阳县', '衡东县', '衡山县', '衡南县', '祁东县', '其他'] }, { name:'邵阳', area:['双清区', '大祥区', '北塔区', '武冈市', '邵东县', '洞口县', '新邵县', '绥宁县', '新宁县', '邵阳县', '隆回县', '城步苗族自治县', '其他'] }, { name:'岳阳', area:['岳阳楼区', '云溪区', '君山区', '临湘市', '汨罗市', '岳阳县', '湘阴县', '平江县', '华容县', '其他'] }, { name:'常德', area:['武陵区', '鼎城区', '津市市', '澧县', '临澧县', '桃源县', '汉寿县', '安乡县', '石门县', '其他'] }, { name:'张家界', area:['永定区', '武陵源区', '慈利县', '桑植县', '其他'] }, { name:'益阳', area:['赫山区', '资阳区', '沅江市', '桃江县', '南县', '安化县', '其他'] }, { name:'郴州', area:['北湖区', '苏仙区', '资兴市', '宜章县', '汝城县', '安仁县', '嘉禾县', '临武县', '桂东县', '永兴县', '桂阳县', '其他'] }, { name:'永州', area:['冷水滩区', '零陵区', '祁阳县', '蓝山县', '宁远县', '新田县', '东安县', '江永县', '道县', '双牌县', '江华瑶族自治县', '其他'] }, { name:'怀化', area:['鹤城区', '洪江市', '会同县', '沅陵县', '辰溪县', '溆浦县', '中方县', '新晃侗族自治县', '芷江侗族自治县', '通道侗族自治县', '靖州苗族侗族自治县', '麻阳苗族自治县', '其他'] }, { name:'娄底', area:['娄星区', '冷水江市', '涟源市', '新化县', '双峰县', '其他'] }, { name:'湘西土家族苗族自治州', area:['吉首市', '古丈县', '龙山县', '永顺县', '凤凰县', '泸溪县', '保靖县', '花垣县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'广东', city:[{ name:'广州', area:['越秀区', '荔湾区', '海珠区', '天河区', '白云区', '黄埔区', '番禺区', '花都区', '南沙区', '萝岗区', '增城市', '从化市', '其他'] }, { name:'深圳', area:['福田区', '罗湖区', '南山区', '宝安区', '龙岗区', '盐田区', '其他'] }, { name:'东莞', area:['莞城', '常平', '塘厦', '塘厦', '塘厦', '其他'] }, { name:'中山', area:['中山'] }, { name:'潮州', area:['湘桥区', '潮安县', '饶平县', '其他'] }, { name:'揭阳', area:['榕城区', '揭东县', '揭西县', '惠来县', '普宁市', '其他'] }, { name:'云浮', area:['云城区', '新兴县', '郁南县', '云安县', '罗定市', '其他'] }, { name:'珠海', area:['香洲区', '斗门区', '金湾区', '其他'] }, { name:'汕头', area:['金平区', '濠江区', '龙湖区', '潮阳区', '潮南区', '澄海区', '南澳县', '其他'] }, { name:'韶关', area:['浈江区', '武江区', '曲江区', '乐昌市', '南雄市', '始兴县', '仁化县', '翁源县', '新丰县', '乳源瑶族自治县', '其他'] }, { name:'佛山', area:['禅城区', '南海区', '顺德区', '三水区', '高明区', '其他'] }, { name:'江门', area:['蓬江区', '江海区', '新会区', '恩平市', '台山市', '开平市', '鹤山市', '其他'] }, { name:'湛江', area:['赤坎区', '霞山区', '坡头区', '麻章区', '吴川市', '廉江市', '雷州市', '遂溪县', '徐闻县', '其他'] }, { name:'茂名', area:['茂南区', '茂港区', '化州市', '信宜市', '高州市', '电白县', '其他'] }, { name:'肇庆', area:['端州区', '鼎湖区', '高要市', '四会市', '广宁县', '怀集县', '封开县', '德庆县', '其他'] }, { name:'惠州', area:['惠城区', '惠阳区', '博罗县', '惠东县', '龙门县', '其他'] }, { name:'梅州', area:['梅江区', '兴宁市', '梅县', '大埔县', '丰顺县', '五华县', '平远县', '蕉岭县', '其他'] }, { name:'汕尾', area:['城区', '陆丰市', '海丰县', '陆河县', '其他'] }, { name:'河源', area:['源城区', '紫金县', '龙川县', '连平县', '和平县', '东源县', '其他'] }, { name:'阳江', area:['江城区', '阳春市', '阳西县', '阳东县', '其他'] }, { name:'清远', area:['清城区', '英德市', '连州市', '佛冈县', '阳山县', '清新县', '连山壮族瑶族自治县', '连南瑶族自治县', '其他'] }] }, { name:'广西', city:[{ name:'南宁', area:['青秀区', '兴宁区', '西乡塘区', '良庆区', '江南区', '邕宁区', '武鸣县', '隆安县', '马山县', '上林县', '宾阳县', '横县', '其他'] }, { name:'柳州', area:['城中区', '鱼峰区', '柳北区', '柳南区', '柳江县', '柳城县', '鹿寨县', '融安县', '融水苗族自治县', '三江侗族自治县', '其他'] }, { name:'桂林', area:['象山区', '秀峰区', '叠彩区', '七星区', '雁山区', '阳朔县', '临桂县', '灵川县', '全州县', '平乐县', '兴安县', '灌阳县', '荔浦县', '资源县', '永福县', '龙胜各族自治县', '恭城瑶族自治县', '其他'] }, { name:'梧州', area:['万秀区', '蝶山区', '长洲区', '岑溪市', '苍梧县', '藤县', '蒙山县', '其他'] }, { name:'北海', area:['海城区', '银海区', '铁山港区', '合浦县', '其他'] }, { name:'防城港', area:['港口区', '防城区', '东兴市', '上思县', '其他'] }, { name:'钦州', area:['钦南区', '钦北区', '灵山县', '浦北县', '其他'] }, { name:'贵港', area:['港北区', '港南区', '覃塘区', '桂平市', '平南县', '其他'] }, { name:'玉林', area:['玉州区', '北流市', '容县', '陆川县', '博白县', '兴业县', '其他'] }, { name:'百色', area:['右江区', '凌云县', '平果县', '西林县', '乐业县', '德保县', '田林县', '田阳县', '靖西县', '田东县', '那坡县', '隆林各族自治县', '其他'] }, { name:'贺州', area:['八步区', '钟山县', '昭平县', '富川瑶族自治县', '其他'] }, { name:'河池', area:['金城江区', '宜州市', '天峨县', '凤山县', '南丹县', '东兰县', '都安瑶族自治县', '罗城仫佬族自治县', '巴马瑶族自治县', '环江毛南族自治县', '大化瑶族自治县', '其他'] }, { name:'来宾', area:['兴宾区', '合山市', '象州县', '武宣县', '忻城县', '金秀瑶族自治县', '其他'] }, { name:'崇左', area:['江州区', '凭祥市', '宁明县', '扶绥县', '龙州县', '大新县', '天等县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'海南', city:[{ name:'海口', area:['龙华区', '秀英区', '琼山区', '美兰区', '其他'] }, { name:'三亚', area:['三亚市', '其他'] }, { name:'五指山', area:['五指山'] }, { name:'琼海', area:['琼海'] }, { name:'儋州', area:['儋州'] }, { name:'文昌', area:['文昌'] }, { name:'万宁', area:['万宁'] }, { name:'东方', area:['东方'] }, { name:'澄迈县', area:['澄迈县'] }, { name:'定安县', area:['定安县'] }, { name:'屯昌县', area:['屯昌县'] }, { name:'临高县', area:['临高县'] }, { name:'白沙黎族自治县', area:['白沙黎族自治县'] }, { name:'昌江黎族自治县', area:['昌江黎族自治县'] }, { name:'乐东黎族自治县', area:['乐东黎族自治县'] }, { name:'陵水黎族自治县', area:['陵水黎族自治县'] }, { name:'保亭黎族苗族自治县', area:['保亭黎族苗族自治县'] }, { name:'琼中黎族苗族自治县', area:['琼中黎族苗族自治县'] }, { name:'其他', area:['其他'] }] }, { name:'重庆', city:[{ name:'重庆', area:['渝中区', '大渡口区', '江北区', '南岸区', '北碚区', '渝北区', '巴南区', '长寿区', '双桥区', '沙坪坝区', '万盛区', '万州区', '涪陵区', '黔江区', '永川区', '合川区', '江津区', '九龙坡区', '南川区', '綦江县', '潼南县', '荣昌县', '璧山县', '大足县', '铜梁县', '梁平县', '开县', '忠县', '城口县', '垫江县', '武隆县', '丰都县', '奉节县', '云阳县', '巫溪县', '巫山县', '石柱土家族自治县', '秀山土家族苗族自治县', '酉阳土家族苗族自治县', '彭水苗族土家族自治县', '其他'] }] }, { name:'四川', city:[{ name:'成都', area:['青羊区', '锦江区', '金牛区', '武侯区', '成华区', '龙泉驿区', '青白江区', '新都区', '温江区', '都江堰市', '彭州市', '邛崃市', '崇州市', '金堂县', '郫县', '新津县', '双流县', '蒲江县', '大邑县', '其他'] }, { name:'自贡', area:['大安区', '自流井区', '贡井区', '沿滩区', '荣县', '富顺县', '其他'] }, { name:'攀枝花', area:['仁和区', '米易县', '盐边县', '东区', '西区', '其他'] }, { name:'泸州', area:['江阳区', '纳溪区', '龙马潭区', '泸县', '合江县', '叙永县', '古蔺县', '其他'] }, { name:'德阳', area:['旌阳区', '广汉市', '什邡市', '绵竹市', '罗江县', '中江县', '其他'] }, { name:'绵阳', area:['涪城区', '游仙区', '江油市', '盐亭县', '三台县', '平武县', '安县', '梓潼县', '北川羌族自治县', '其他'] }, { name:'广元', area:['元坝区', '朝天区', '青川县', '旺苍县', '剑阁县', '苍溪县', '市中区', '其他'] }, { name:'遂宁', area:['船山区', '安居区', '射洪县', '蓬溪县', '大英县', '其他'] }, { name:'内江', area:['市中区', '东兴区', '资中县', '隆昌县', '威远县', '其他'] }, { name:'乐山', area:['市中区', '五通桥区', '沙湾区', '金口河区', '峨眉山市', '夹江县', '井研县', '犍为县', '沐川县', '马边彝族自治县', '峨边彝族自治县', '其他'] }, { name:'南充', area:['顺庆区', '高坪区', '嘉陵区', '阆中市', '营山县', '蓬安县', '仪陇县', '南部县', '西充县', '其他'] }, { name:'眉山', area:['东坡区', '仁寿县', '彭山县', '洪雅县', '丹棱县', '青神县', '其他'] }, { name:'宜宾', area:['翠屏区', '宜宾县', '兴文县', '南溪县', '珙县', '长宁县', '高县', '江安县', '筠连县', '屏山县', '其他'] }, { name:'广安', area:['广安区', '华蓥市', '岳池县', '邻水县', '武胜县', '其他'] }, { name:'达州', area:['通川区', '万源市', '达县', '渠县', '宣汉县', '开江县', '大竹县', '其他'] }, { name:'雅安', area:['雨城区', '芦山县', '石棉县', '名山县', '天全县', '荥经县', '宝兴县', '汉源县', '其他'] }, { name:'巴中', area:['巴州区', '南江县', '平昌县', '通江县', '其他'] }, { name:'资阳', area:['雁江区', '简阳市', '安岳县', '乐至县', '其他'] }, { name:'阿坝藏族羌族自治州', area:['马尔康县', '九寨沟县', '红原县', '汶川县', '阿坝县', '理县', '若尔盖县', '小金县', '黑水县', '金川县', '松潘县', '壤塘县', '茂县', '其他'] }, { name:'甘孜藏族自治州', area:['康定县', '丹巴县', '炉霍县', '九龙县', '甘孜县', '雅江县', '新龙县', '道孚县', '白玉县', '理塘县', '德格县', '乡城县', '石渠县', '稻城县', '色达县', '巴塘县', '泸定县', '得荣县', '其他'] }, { name:'凉山彝族自治州', area:['西昌市', '美姑县', '昭觉县', '金阳县', '甘洛县', '布拖县', '雷波县', '普格县', '宁南县', '喜德县', '会东县', '越西县', '会理县', '盐源县', '德昌县', '冕宁县', '木里藏族自治县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'贵州', city:[{ name:'贵阳', area:['南明区', '云岩区', '花溪区', '乌当区', '白云区', '小河区', '清镇市', '开阳县', '修文县', '息烽县', '其他'] }, { name:'六盘水', area:['钟山区', '水城县', '盘县', '六枝特区', '其他'] }, { name:'遵义', area:['红花岗区', '汇川区', '赤水市', '仁怀市', '遵义县', '绥阳县', '桐梓县', '习水县', '凤冈县', '正安县', '余庆县', '湄潭县', '道真仡佬族苗族自治县', '务川仡佬族苗族自治县', '其他'] }, { name:'安顺', area:['西秀区', '普定县', '平坝县', '镇宁布依族苗族自治县', '紫云苗族布依族自治县', '关岭布依族苗族自治县', '其他'] }, { name:'铜仁地区', area:['铜仁市', '德江县', '江口县', '思南县', '石阡县', '玉屏侗族自治县', '松桃苗族自治县', '印江土家族苗族自治县', '沿河土家族自治县', '万山特区', '其他'] }, { name:'毕节地区', area:['毕节市', '黔西县', '大方县', '织金县', '金沙县', '赫章县', '纳雍县', '威宁彝族回族苗族自治县', '其他'] }, { name:'黔西南布依族苗族自治州', area:['兴义市', '望谟县', '兴仁县', '普安县', '册亨县', '晴隆县', '贞丰县', '安龙县', '其他'] }, { name:'黔东南苗族侗族自治州', area:['凯里市', '施秉县', '从江县', '锦屏县', '镇远县', '麻江县', '台江县', '天柱县', '黄平县', '榕江县', '剑河县', '三穗县', '雷山县', '黎平县', '岑巩县', '丹寨县', '其他'] }, { name:'黔南布依族苗族自治州', area:['都匀市', '福泉市', '贵定县', '惠水县', '罗甸县', '瓮安县', '荔波县', '龙里县', '平塘县', '长顺县', '独山县', '三都水族自治县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'云南', city:[{ name:'昆明', area:['盘龙区', '五华区', '官渡区', '西山区', '东川区', '安宁市', '呈贡县', '晋宁县', '富民县', '宜良县', '嵩明县', '石林彝族自治县', '禄劝彝族苗族自治县', '寻甸回族彝族自治县', '其他'] }, { name:'曲靖', area:['麒麟区', '宣威市', '马龙县', '沾益县', '富源县', '罗平县', '师宗县', '陆良县', '会泽县', '其他'] }, { name:'玉溪', area:['红塔区', '江川县', '澄江县', '通海县', '华宁县', '易门县', '峨山彝族自治县', '新平彝族傣族自治县', '元江哈尼族彝族傣族自治县', '其他'] }, { name:'保山', area:['隆阳区', '施甸县', '腾冲县', '龙陵县', '昌宁县', '其他'] }, { name:'昭通', area:['昭阳区', '鲁甸县', '巧家县', '盐津县', '大关县', '永善县', '绥江县', '镇雄县', '彝良县', '威信县', '水富县', '其他'] }, { name:'丽江', area:['古城区', '永胜县', '华坪县', '玉龙纳西族自治县', '宁蒗彝族自治县', '其他'] }, { name:'普洱', area:['思茅区', '普洱哈尼族彝族自治县', '墨江哈尼族自治县', '景东彝族自治县', '景谷傣族彝族自治县', '镇沅彝族哈尼族拉祜族自治县', '江城哈尼族彝族自治县', '孟连傣族拉祜族佤族自治县', '澜沧拉祜族自治县', '西盟佤族自治县', '其他'] }, { name:'临沧', area:['临翔区', '凤庆县', '云县', '永德县', '镇康县', '双江拉祜族佤族布朗族傣族自治县', '耿马傣族佤族自治县', '沧源佤族自治县', '其他'] }, { name:'德宏傣族景颇族自治州', area:['潞西市', '瑞丽市', '梁河县', '盈江县', '陇川县', '其他'] }, { name:'怒江傈僳族自治州', area:['泸水县', '福贡县', '贡山独龙族怒族自治县', '兰坪白族普米族自治县', '其他'] }, { name:'迪庆藏族自治州', area:['香格里拉县', '德钦县', '维西傈僳族自治县', '其他'] }, { name:'大理白族自治州', area:['大理市', '祥云县', '宾川县', '弥渡县', '永平县', '云龙县', '洱源县', '剑川县', '鹤庆县', '漾濞彝族自治县', '南涧彝族自治县', '巍山彝族回族自治县', '其他'] }, { name:'楚雄彝族自治州', area:['楚雄市', '双柏县', '牟定县', '南华县', '姚安县', '大姚县', '永仁县', '元谋县', '武定县', '禄丰县', '其他'] }, { name:'红河哈尼族彝族自治州', area:['蒙自县', '个旧市', '开远市', '绿春县', '建水县', '石屏县', '弥勒县', '泸西县', '元阳县', '红河县', '金平苗族瑶族傣族自治县', '河口瑶族自治县', '屏边苗族自治县', '其他'] }, { name:'文山壮族苗族自治州', area:['文山县', '砚山县', '西畴县', '麻栗坡县', '马关县', '丘北县', '广南县', '富宁县', '其他'] }, { name:'西双版纳傣族自治州', area:['景洪市', '勐海县', '勐腊县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'西藏', city:[{ name:'拉萨', area:['城关区', '林周县', '当雄县', '尼木县', '曲水县', '堆龙德庆县', '达孜县', '墨竹工卡县', '其他'] }, { name:'那曲地区', area:['那曲县', '嘉黎县', '比如县', '聂荣县', '安多县', '申扎县', '索县', '班戈县', '巴青县', '尼玛县', '其他'] }, { name:'昌都地区', area:['昌都县', '江达县', '贡觉县', '类乌齐县', '丁青县', '察雅县', '八宿县', '左贡县', '芒康县', '洛隆县', '边坝县', '其他'] }, { name:'林芝地区', area:['林芝县', '工布江达县', '米林县', '墨脱县', '波密县', '察隅县', '朗县', '其他'] }, { name:'山南地区', area:['乃东县', '扎囊县', '贡嘎县', '桑日县', '琼结县', '曲松县', '措美县', '洛扎县', '加查县', '隆子县', '错那县', '浪卡子县', '其他'] }, { name:'日喀则地区', area:['日喀则市', '南木林县', '江孜县', '定日县', '萨迦县', '拉孜县', '昂仁县', '谢通门县', '白朗县', '仁布县', '康马县', '定结县', '仲巴县', '亚东县', '吉隆县', '聂拉木县', '萨嘎县', '岗巴县', '其他'] }, { name:'阿里地区', area:['噶尔县', '普兰县', '札达县', '日土县', '革吉县', '改则县', '措勤县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'陕西', city:[{ name:'西安', area:['莲湖区', '新城区', '碑林区', '雁塔区', '灞桥区', '未央区', '阎良区', '临潼区', '长安区', '高陵县', '蓝田县', '户县', '周至县', '其他'] }, { name:'铜川', area:['耀州区', '王益区', '印台区', '宜君县', '其他'] }, { name:'宝鸡', area:['渭滨区', '金台区', '陈仓区', '岐山县', '凤翔县', '陇县', '太白县', '麟游县', '扶风县', '千阳县', '眉县', '凤县', '其他'] }, { name:'咸阳', area:['秦都区', '渭城区', '杨陵区', '兴平市', '礼泉县', '泾阳县', '永寿县', '三原县', '彬县', '旬邑县', '长武县', '乾县', '武功县', '淳化县', '其他'] }, { name:'渭南', area:['临渭区', '韩城市', '华阴市', '蒲城县', '潼关县', '白水县', '澄城县', '华县', '合阳县', '富平县', '大荔县', '其他'] }, { name:'延安', area:['宝塔区', '安塞县', '洛川县', '子长县', '黄陵县', '延川县', '富县', '延长县', '甘泉县', '宜川县', '志丹县', '黄龙县', '吴起县', '其他'] }, { name:'汉中', area:['汉台区', '留坝县', '镇巴县', '城固县', '南郑县', '洋县', '宁强县', '佛坪县', '勉县', '西乡县', '略阳县', '其他'] }, { name:'榆林', area:['榆阳区', '清涧县', '绥德县', '神木县', '佳县', '府谷县', '子洲县', '靖边县', '横山县', '米脂县', '吴堡县', '定边县', '其他'] }, { name:'安康', area:['汉滨区', '紫阳县', '岚皋县', '旬阳县', '镇坪县', '平利县', '石泉县', '宁陕县', '白河县', '汉阴县', '其他'] }, { name:'商洛', area:['商州区', '镇安县', '山阳县', '洛南县', '商南县', '丹凤县', '柞水县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'甘肃', city:[{ name:'兰州', area:['城关区', '七里河区', '西固区', '安宁区', '红古区', '永登县', '皋兰县', '榆中县', '其他'] }, { name:'嘉峪关', area:['嘉峪关市', '其他'] }, { name:'金昌', area:['金川区', '永昌县', '其他'] }, { name:'白银', area:['白银区', '平川区', '靖远县', '会宁县', '景泰县', '其他'] }, { name:'天水', area:['清水县', '秦安县', '甘谷县', '武山县', '张家川回族自治县', '北道区', '秦城区', '其他'] }, { name:'武威', area:['凉州区', '民勤县', '古浪县', '天祝藏族自治县', '其他'] }, { name:'酒泉', area:['肃州区', '玉门市', '敦煌市', '金塔县', '肃北蒙古族自治县', '阿克塞哈萨克族自治县', '安西县', '其他'] }, { name:'张掖', area:['甘州区', '民乐县', '临泽县', '高台县', '山丹县', '肃南裕固族自治县', '其他'] }, { name:'庆阳', area:['西峰区', '庆城县', '环县', '华池县', '合水县', '正宁县', '宁县', '镇原县', '其他'] }, { name:'平凉', area:['崆峒区', '泾川县', '灵台县', '崇信县', '华亭县', '庄浪县', '静宁县', '其他'] }, { name:'定西', area:['安定区', '通渭县', '临洮县', '漳县', '岷县', '渭源县', '陇西县', '其他'] }, { name:'陇南', area:['武都区', '成县', '宕昌县', '康县', '文县', '西和县', '礼县', '两当县', '徽县', '其他'] }, { name:'临夏回族自治州', area:['临夏市', '临夏县', '康乐县', '永靖县', '广河县', '和政县', '东乡族自治县', '积石山保安族东乡族撒拉族自治县', '其他'] }, { name:'甘南藏族自治州', area:['合作市', '临潭县', '卓尼县', '舟曲县', '迭部县', '玛曲县', '碌曲县', '夏河县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'青海', city:[{ name:'西宁', area:['城中区', '城东区', '城西区', '城北区', '湟源县', '湟中县', '大通回族土族自治县', '其他'] }, { name:'海东地区', area:['平安县', '乐都县', '民和回族土族自治县', '互助土族自治县', '化隆回族自治县', '循化撒拉族自治县', '其他'] }, { name:'海北藏族自治州', area:['海晏县', '祁连县', '刚察县', '门源回族自治县', '其他'] }, { name:'海南藏族自治州', area:['共和县', '同德县', '贵德县', '兴海县', '贵南县', '其他'] }, { name:'黄南藏族自治州', area:['同仁县', '尖扎县', '泽库县', '河南蒙古族自治县', '其他'] }, { name:'果洛藏族自治州', area:['玛沁县', '班玛县', '甘德县', '达日县', '久治县', '玛多县', '其他'] }, { name:'玉树藏族自治州', area:['玉树县', '杂多县', '称多县', '治多县', '囊谦县', '曲麻莱县', '其他'] }, { name:'海西蒙古族藏族自治州', area:['德令哈市', '格尔木市', '乌兰县', '都兰县', '天峻县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'宁夏', city:[{ name:'银川', area:['兴庆区', '西夏区', '金凤区', '灵武市', '永宁县', '贺兰县', '其他'] }, { name:'石嘴山', area:['大武口区', '惠农区', '平罗县', '其他'] }, { name:'吴忠', area:['利通区', '青铜峡市', '盐池县', '同心县', '其他'] }, { name:'固原', area:['原州区', '西吉县', '隆德县', '泾源县', '彭阳县', '其他'] }, { name:'中卫', area:['沙坡头区', '中宁县', '海原县', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'新疆', city:[{ name:'乌鲁木齐', area:['天山区', '沙依巴克区', '新市区', '水磨沟区', '头屯河区', '达坂城区', '东山区', '乌鲁木齐县', '其他'] }, { name:'克拉玛依', area:['克拉玛依区', '独山子区', '白碱滩区', '乌尔禾区', '其他'] }, { name:'吐鲁番地区', area:['吐鲁番市', '托克逊县', '鄯善县', '其他'] }, { name:'哈密地区', area:['哈密市', '伊吾县', '巴里坤哈萨克自治县', '其他'] }, { name:'和田地区', area:['和田市', '和田县', '洛浦县', '民丰县', '皮山县', '策勒县', '于田县', '墨玉县', '其他'] }, { name:'阿克苏地区', area:['阿克苏市', '温宿县', '沙雅县', '拜城县', '阿瓦提县', '库车县', '柯坪县', '新和县', '乌什县', '其他'] }, { name:'喀什地区', area:['喀什市', '巴楚县', '泽普县', '伽师县', '叶城县', '岳普湖县', '疏勒县', '麦盖提县', '英吉沙县', '莎车县', '疏附县', '塔什库尔干塔吉克自治县', '其他'] }, { name:'克孜勒苏柯尔克孜自治州', area:['阿图什市', '阿合奇县', '乌恰县', '阿克陶县', '其他'] }, { name:'巴音郭楞蒙古自治州', area:['库尔勒市', '和静县', '尉犁县', '和硕县', '且末县', '博湖县', '轮台县', '若羌县', '焉耆回族自治县', '其他'] }, { name:'昌吉回族自治州', area:['昌吉市', '阜康市', '奇台县', '玛纳斯县', '吉木萨尔县', '呼图壁县', '木垒哈萨克自治县', '米泉市', '其他'] }, { name:'博尔塔拉蒙古自治州', area:['博乐市', '精河县', '温泉县', '其他'] }, { name:'石河子', area:['石河子'] }, { name:'阿拉尔', area:['阿拉尔'] }, { name:'图木舒克', area:['图木舒克'] }, { name:'五家渠', area:['五家渠'] }, { name:'伊犁哈萨克自治州', area:['伊宁市', '奎屯市', '伊宁县', '特克斯县', '尼勒克县', '昭苏县', '新源县', '霍城县', '巩留县', '察布查尔锡伯自治县', '塔城地区', '阿勒泰地区', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'台湾', city:[{ name:'台湾', area:['台北市', '高雄市', '台北县', '桃园县', '新竹县', '苗栗县', '台中县', '彰化县', '南投县', '云林县', '嘉义县', '台南县', '高雄县', '屏东县', '宜兰县', '花莲县', '台东县', '澎湖县', '基隆市', '新竹市', '台中市', '嘉义市', '台南市', '其他'] }, { name:'其他', area:['其他'] }] }, { name:'澳门', city:[{ name:'澳门', area:['花地玛堂区', '圣安多尼堂区', '大堂区', '望德堂区', '风顺堂区', '嘉模堂区', '圣方济各堂区', '路凼', '其他'] }] }, { name:'香港', city:[{ name:'香港', area:['中西区', '湾仔区', '东区', '南区', '深水埗区', '油尖旺区', '九龙城区', '黄大仙区', '观塘区', '北区', '大埔区', '沙田区', '西贡区', '元朗区', '屯门区', '荃湾区', '葵青区', '离岛区', '其他'] }] }, { name:'钓鱼岛', city:[{ name:'钓鱼岛', area:['钓鱼岛'] }] }];

```

* PDShop/project/App/manager/JpushMgr.js

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

* PDShop/project/App/manager/LoginMgr.js

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

* PDShop/project/App/manager/NetMgr.js

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

* PDShop/project/App/manager/PersonalInfoMgr.js

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
                this.needLogin = needLogin !== 'false';
                this.remainAmount = await AsyncStorage.getItem(REMAIN_AMOUNR);
                const infoStr = await AsyncStorage.getItem(ITEM_NAME);
                info = JSON.parse(infoStr);
            } catch (e) {
                this.needLogin = true;
            }
            this.info = info || {};
            this.needLogin = this.needLogin || !info;
            this.remainAmount = this.remainAmount || !info;
            this.initialized = true;
        });
    }
    set (info) {
        this.info = info;
        return new Promise(async(resolve, reject) => {
            await AsyncStorage.setItem(ITEM_NAME, JSON.stringify(info));
            resolve();
        });
    }
    setUserHead (head) {
        this.emit('USER_HEAD_CHANGE_EVENT', { head:head });
    }
    setNeedLogin (flag) {
        this.needLogin = flag;
        AsyncStorage.setItem(NEED_LOGIN_ITEM_NAME, flag + '');
    }
    setRemainAmount (amount) {
        this.remainAmount = amount;
        AsyncStorage.setItem(REMAIN_AMOUNR, amount + '');
    }
}

module.exports = new Manager();

```

* PDShop/project/App/manager/ScanStoageOrderMgr.js

```js
'use strict';
const ReactNative = require('react-native');
const {
    AsyncStorage,
} = ReactNative;
const EventEmitter = require('EventEmitter');

const ITEM_NAME = 'SCAN_STOAGE_ORDER_LIST';

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
            this.totalOrders = list.length;
            this.totalNumbers = _.sumBy(list, 'totalNumbers');
            this.totalSize = _.sumBy(list, 'size');
            this.totalWeight = _.sumBy(list, 'weight');
            this.stoageNumbers = _.sumBy(list, o => { return _.sumBy(o.stateList, item => { return item.state == 6 && item.count; }); });
            this.unStoageNumbers = _.sumBy(list, o => { return _.sumBy(o.stateList, item => { return item.state == 5 && item.count; }); });
            await AsyncStorage.setItem(ITEM_NAME, JSON.stringify(list));
            resolve();
        });
    }
    saveOrder (order) {
        let list = this.list;
        console.log('--------', order, list);
        if (_.find(list, o => { return o.id == order.id; })) {
            console.log('--------1', order, list);
            _.remove(list, o => { return o.id == order.id; });
            console.log('--------2', order, list);
        }
        list.unshift(order);
        console.log('--------3', order, list);
        this.set(list);
    }
    clear () {
        this.list = [];
        AsyncStorage.removeItem(ITEM_NAME);
    }
}

module.exports = new Manager();

```

* PDShop/project/App/manager/SettingMgr.js

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

* PDShop/project/App/manager/UpdateMgr.js

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

* PDShop/project/App/manager/WXpayMgr.js

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

* PDShop/project/App/resource/audio.js

```js
module.exports = {
};

```

* PDShop/project/App/resource/image.js

```js
module.exports = {
    cargo_balance_background:require('./image/cargo/balance_background.png'),
    cargo_cancel:require('./image/cargo/cancel.png'),
    cargo_point:require('./image/cargo/point.png'),
    chairman_date:require('./image/chairman/date.png'),
    chairman_selcet_mine:require('./image/chairman/selcet_mine.png'),
    chairman_select_incomestatistics:require('./image/chairman/select_incomestatistics.png'),
    chairman_select_orderstatistics:require('./image/chairman/select_orderstatistics.png'),
    chairman_un_selcet_mine:require('./image/chairman/un_selcet_mine.png'),
    chairman_un_select_incomestatistics:require('./image/chairman/un_select_incomestatistics.png'),
    chairman_un_select_orderstatistics:require('./image/chairman/un_select_orderstatistics.png'),
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
    home_cargo_order:require('./image/home/cargo_order.png'),
    home_cargo_order_press:require('./image/home/cargo_order_press.png'),
    home_cargo_press:require('./image/home/cargo_press.png'),
    home_main:require('./image/home/main.png'),
    home_main_press:require('./image/home/main_press.png'),
    home_mine:require('./image/home/mine.png'),
    home_mine_press:require('./image/home/mine_press.png'),
    home_order:require('./image/home/order.png'),
    home_order_press:require('./image/home/order_press.png'),
    home_pre_order:require('./image/home/pre_order.png'),
    home_pre_order_press:require('./image/home/pre_order_press.png'),
    home_roadmap:require('./image/home/roadmap.png'),
    home_roadmap_press:require('./image/home/roadmap_press.png'),
    home_storage:require('./image/home/storage.png'),
    home_storage_press:require('./image/home/storage_press.png'),
    home_warehouse_order:require('./image/home/warehouse_order.png'),
    home_warehouse_order_press:require('./image/home/warehouse_order_press.png'),
    login_alipay_button:require('./image/login/alipay_button.png'),
    login_logo:require('./image/login/logo.png'),
    login_password:require('./image/login/password.png'),
    login_password_check:require('./image/login/password_check.png'),
    login_password_look:require('./image/login/password_look.png'),
    login_phone:require('./image/login/phone.png'),
    login_qq_button:require('./image/login/qq_button.png'),
    login_weixin_button:require('./image/login/weixin_button.png'),
    moving_message:require('./image/moving/message.png'),
    moving_message_r:require('./image/moving/message_r.png'),
    moving_message_select:require('./image/moving/message_select.png'),
    moving_message_unselect:require('./image/moving/message_unselect.png'),
    moving_moving_select:require('./image/moving/moving_select.png'),
    moving_moving_unselect:require('./image/moving/moving_unselect.png'),
    moving_no_message:require('./image/moving/no_message.png'),
    moving_person_select:require('./image/moving/person_select.png'),
    moving_person_unselect:require('./image/moving/person_unselect.png'),
    moving_setting:require('./image/moving/setting.png'),
    order_add:require('./image/order/add.png'),
    order_choose:require('./image/order/choose.png'),
    order_choose_r:require('./image/order/choose_r.png'),
    order_delete:require('./image/order/delete.png'),
    order_down:require('./image/order/down.png'),
    order_logo:require('./image/order/logo.png'),
    order_no_choose:require('./image/order/no_choose.png'),
    order_no_choose_r:require('./image/order/no_choose_r.png'),
    order_pay_success:require('./image/order/pay_success.png'),
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
    receive_camera:require('./image/receive/camera.png'),
    receive_no_orders:require('./image/receive/no_orders.png'),
    receive_order:require('./image/receive/order.png'),
    receive_order_default:require('./image/receive/order_default.png'),
    receive_order_press:require('./image/receive/order_press.png'),
    receive_take_picture:require('./image/receive/take_picture.png'),
    receive_take_picture_press:require('./image/receive/take_picture_press.png'),
    roadmap_add_photos:require('./image/roadmap/add_photos.png'),
    roadmap_address:require('./image/roadmap/address.png'),
    roadmap_collection_normal:require('./image/roadmap/collection_normal.png'),
    roadmap_collection_press:require('./image/roadmap/collection_press.png'),
    roadmap_comment:require('./image/roadmap/comment.png'),
    roadmap_share:require('./image/roadmap/share.png'),
    security_scan_check_pass:require('./image/security/scan_check_pass.png'),
    shipper_care_normal:require('./image/shipper/care_normal.png'),
    shipper_care_press:require('./image/shipper/care_press.png'),
    shipper_phone:require('./image/shipper/phone.png'),
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
    storage_scan:require('./image/storage/scan.png'),
    storage_scan_code_storage:require('./image/storage/scan_code_storage.png'),
};

```

* PDShop/project/App/utils/index.js

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

* PDShop/project/App/components/ActionSheet/button.js

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

* PDShop/project/App/components/ActionSheet/index.js

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

* PDShop/project/App/components/ActionSheet/overlay.js

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

* PDShop/project/App/components/ActionSheet/sheet.js

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

* PDShop/project/App/modules/baidumap/index.js

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
                <View>
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
        height: sr.h - 150,
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
});

```

* PDShop/project/App/modules/baidumap/test.js

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

* PDShop/project/App/modules/chairman/IncomeStatistics.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
    TouchableOpacity,
    Text,
} = ReactNative;

const Statistics = require('./Statistics');

module.exports = React.createClass({
    statics :{
        title : '收入统计',
    },
    onWillFocus () {
        app.getCurrentRoute().title = '收入统计';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    render () {
        return (
            <View style={styles.container}>
                <TouchableOpacity style={styles.row} onPress={() => app.push({ component : Statistics, title : '整店收入',
                    passProps : { type : 'totalincome', buttonList : ['总提成', '总部提成', '分部提成', '手续费'] } })}>
                    <Text style={styles.rowText}>整店收入</Text>
                    <Image
                        source={app.img.common_go}
                        style={styles.goStyle} />
                </TouchableOpacity>
                <TouchableOpacity style={styles.row} onPress={() => app.push({ component : Statistics, title : '收货部',
                    passProps : { type : 'receiveincome', buttonList : ['库存', '入库', '重量', '方量'] } })}>
                    <Text style={styles.rowText}>收货部</Text>
                    <Image
                        source={app.img.common_go}
                        style={styles.goStyle} />
                </TouchableOpacity>
                <TouchableOpacity style={styles.row} onPress={() => app.push({ component : Statistics, title : '搬运部',
                    passProps : { type : 'movungincome', buttonList : ['总收入', '搬运价'] } })}>
                    <Text style={styles.rowText}>搬运部</Text>
                    <Image
                        source={app.img.common_go}
                        style={styles.goStyle} />
                </TouchableOpacity>
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
    row: {
        width:sr.w,
        height:42,
        flexDirection:'row',
        alignItems:'center',
        backgroundColor:'#ffffff',
        marginTop:1,
    },
    rowText: {
        marginLeft:10,
        fontSize:14,
        fontWeight:'300',
        flex:1,
    },
    goStyle: {
        height: 13,
        width: 8,
        marginRight: 20,
    },
});

```

* PDShop/project/App/modules/chairman/OrderStatistics.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
    TouchableOpacity,
    Text,
} = ReactNative;

const Statistics = require('./Statistics');

module.exports = React.createClass({
    statics :{
        title : '货单统计',
    },
    onWillFocus () {
        app.getCurrentRoute().title = '货单统计';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    render () {
        return (
            <View style={styles.container}>
                <TouchableOpacity style={styles.row} onPress={() => app.push({ component : Statistics, title : '总货单',
                    passProps : { type : 'totalorder', buttonList : ['库存', '入库', '重量', '方量'] } })}>
                    <Text style={styles.rowText}>总货单</Text>
                    <Image
                        source={app.img.common_go}
                        style={styles.goStyle} />
                </TouchableOpacity>
                <TouchableOpacity style={styles.row} onPress={() => app.push({ component : Statistics, title : '收货部',
                    passProps : { type : 'receiveorder', buttonList : ['库存', '入库', '重量', '方量'] } })}>
                    <Text style={styles.rowText}>收货部</Text>
                    <Image
                        source={app.img.common_go}
                        style={styles.goStyle} />
                </TouchableOpacity>
                <TouchableOpacity style={styles.row} onPress={() => app.push({ component : Statistics, title : '搬运部',
                    passProps : { type : 'movingorder', buttonList : ['搬运单数', '搬运价'] } })}>
                    <Text style={styles.rowText}>搬运部</Text>
                    <Image
                        source={app.img.common_go}
                        style={styles.goStyle} />
                </TouchableOpacity>
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
    row: {
        width:sr.w,
        height:42,
        flexDirection:'row',
        alignItems:'center',
        backgroundColor:'#ffffff',
        marginTop:1,
    },
    rowText: {
        marginLeft:10,
        fontSize:14,
        fontWeight:'300',
        flex:1,
    },
    goStyle: {
        height: 13,
        width: 8,
        marginRight: 20,
    },
});

```

* PDShop/project/App/modules/chairman/Statistics.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
    TouchableOpacity,
    Text,
} = ReactNative;

import moment from 'moment';

const { Button, Picker } = COMPONENTS;
const Line = require('../common/Line');

module.exports = React.createClass({
    getInitialState () {
        return {
            data : '',
            startDate : moment().format('YYYY年M月D日'),
            endDate : moment().format('YYYY年M月D日'),
            type : true,
            month : moment().format('YYYY年M月'),
        };
    },
    componentWillMount () {
        this.getStatistics();
    },
    getStatistics () {
        const param = {
            userId : app.personal.info.userId,
            shopId : app.personal.info.shop.id,
        };
        POST(app.route.ROUTE_GET_STATISTICS, param, this.getStatisticsSuccess, false);
    },
    getStatisticsSuccess (data) {
        const { buttonList, type } = this.props;
        if (data.success) {
            let dataSource = data.context;
            if (type == 'totalorder') {
                dataSource = [{
                    name: buttonList[0],
                    type: 'line',
                    data: dataSource.count,
                },
                {
                    name: buttonList[2],
                    type: 'line',
                    data: dataSource.weight,
                },
                {
                    name: buttonList[3],
                    type: 'line',
                    data: dataSource.size,
                }];
            } else if (type == 'receiveorder') {
                dataSource = [{
                    name: buttonList[1],
                    type: 'line',
                    data: dataSource.count,
                },
                {
                    name: buttonList[2],
                    type: 'line',
                    data: dataSource.weight,
                },
                {
                    name: buttonList[3],
                    type: 'line',
                    data: dataSource.size,
                }];
            } else if (type == 'movingorder') {
                dataSource = [{
                    name: buttonList[0],
                    type: 'line',
                    data: dataSource.count,
                },
                {
                    name: buttonList[1],
                    type: 'line',
                    data: [2, 6, 4, 3, 1, 6, 0],
                }];
            } else if (type == 'totalincome') {
                dataSource = [{
                    name: buttonList[0],
                    type: 'line',
                    data: dataSource.branchProfit,
                },
                {
                    name: buttonList[1],
                    type: 'line',
                    data: dataSource.masterProfit,
                },
                {
                    name: buttonList[2],
                    type: 'line',
                    data: dataSource.branchProfit,
                },
                {
                    name: buttonList[3],
                    type: 'line',
                    data: dataSource.proxyCharge,
                }];
            } else if (type == 'receiveincome') {
                dataSource = [{
                    name: buttonList[0],
                    type: 'line',
                    data: dataSource.profit,
                },
                {
                    name: buttonList[1],
                    type: 'line',
                    data: dataSource.masterProfit,
                },
                {
                    name: buttonList[2],
                    type: 'line',
                    data: dataSource.branchProfit,
                },
                {
                    name: buttonList[3],
                    type: 'line',
                    data: dataSource.proxyChargeProfit,
                }];
            } else if (type == 'movungincome') {
                dataSource = [{
                    name: buttonList[0],
                    type: 'line',
                    data: dataSource.count,
                },
                {
                    name: buttonList[1],
                    type: 'line',
                    data: null,
                }];
            }
            this.setState({ data : dataSource });
        } else {
            Toast(data.msg);
        }
    },
    showDatePicker (Picker, start, end, selected, type) {
        const status = this.state.type;
        const selectedValue = [selected.year() + '-', (selected.month() + 1) + '-', selected.date() + '-'];
        const pickerData = status ? app.utils.createDateData(start, end) : app.utils.createMonthData(start, end);
        Picker(pickerData, selectedValue, '选择日期').then((value) => {
            const time = moment(value.join(''), 'YYYY-M-D');
            const date = moment(time.year() + '-' + (time.month() + 1) + '-' + time.date() + '-', 'YYYY-MM-DD');
            if (type == 'start') {
                this.setState({ startDate : value });
            } else if (type == 'end') {
                this.setState({ endDate : value });
            } else if (type == 'month') {
                this.setState({ month : value });
            }
        });
    },
    render () {
        const { buttonList } = this.props;
        const { data, startDate, endDate, type, month } = this.state;
        const list = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
        const day = moment().day();
        const days = _.rangeRight(day, day - 7).map(o => list[(o + 7) % 7]);
        return (
            <View style={styles.container}>
                <View style={styles.title}>
                    <Image source={app.img.chairman_date} style={styles.dateImg} />
                    {
                        type && <View style={styles.title}>
                            <TouchableOpacity onPress={() => { this.showDatePicker(Picker, moment('2017-01-01'), moment('2050-12-31'), moment(), 'start'); }}>
                                <Text style={styles.titleText}>{startDate}</Text>
                            </TouchableOpacity>
                            <Text style={styles.titleText}>|</Text>
                            <TouchableOpacity onPress={() => { this.showDatePicker(Picker, moment(startDate), moment('2050-12-31'), moment(), 'end'); }}>
                                <Text style={styles.titleText}>{endDate}</Text>
                            </TouchableOpacity>
                        </View> || <View style={styles.title}>
                            <TouchableOpacity onPress={() => { this.showDatePicker(Picker, moment('2017-01'), moment('2050-12'), moment(), 'month'); }}>
                                <Text style={styles.titleText}>{month}</Text>
                            </TouchableOpacity>
                        </View>
                    }

                    <TouchableOpacity style={styles.chooseType} onPress={() => { this.setState({ type : !type }); }}>
                        <Text style={styles.chooseTypeText}>{type ? '按日期' : '按月份'}</Text>
                    </TouchableOpacity>

                </View>
                <View style={styles.textInfo}>
                    <View style={styles.info}>
                        <Text style={styles.infoText}>总库存: 16825单</Text>
                        <Text style={styles.infoText}>总入库: 16825单</Text>
                        <Text style={styles.infoText}>库存方量: {app.utils.N(13451.23)}m³</Text>
                        <Text style={styles.infoText}>库存重量: {app.utils.N(13451.23)}吨</Text>
                    </View>
                </View>
                <View style={styles.chart}>
                    <Line title={buttonList} data={data} day={days} />
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
    title:{
        flexDirection:'row',
        height:35,
        alignItems:'center',
        paddingLeft:10,
    },
    titleText:{
        fontSize:14,
        color:'#333333',
        marginLeft:10,
    },
    dateImg: {
        width:15,
        height:15,
    },
    chooseType: {
        flex:1,
        alignItems:'flex-end',
    },
    chooseTypeText: {
        fontSize:14,
        color:'#333333',
        marginRight:10,
    },
    textInfo:{
        width:sr.w,
        height:128,
        backgroundColor:'#ffffff',
        flexDirection:'row',
        justifyContent:'center',
        marginBottom:1,
    },
    info: {
        height:100,
        width:sr.w * 0.8,
        backgroundColor:'#F4F4F4',
        alignSelf:'center',
        paddingTop:5,
        paddingLeft:10,
    },
    infoText: {
        flex:1,
        fontSize:14,
        fontWeight:'300',
    },
    buttonList: {
        height:100,
        alignSelf:'center',
        marginLeft:20,
        justifyContent:'flex-start',
    },
    btn: {
        backgroundColor:'#ffffff',
        height:20,
        width:65,
        marginTop:5,
        borderRadius:1,
        borderColor:'#DDDDDD',
        borderWidth:1,
    },
    btnText: {
        fontSize:13,
        color:'#666666',
        fontWeight:'400',
    },
    chart: {
        flex:1,
        backgroundColor:'#ffffff',
    },
});

```

* PDShop/project/App/modules/chairman/index.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
} = ReactNative;

import TabNavigator from 'react-native-tab-navigator';

const OrderStatistics = require('./OrderStatistics');
const IncomeStatistics = require('./IncomeStatistics');
const Person = require('../person');

const INIT_ROUTE_INDEX = 0;
const ROUTE_STACK = [
    { index: 0, component: OrderStatistics },
    { index: 1, component: IncomeStatistics },
    { index: 2, component: Person },
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
            { index: 0, title: '货单统计', icon:app.img.chairman_un_select_orderstatistics, selected: app.img.chairman_select_orderstatistics },
            { index: 1, title: '收入统计', icon: app.img.chairman_un_select_incomestatistics, selected: app.img.chairman_select_incomestatistics },
            { index: 2, title: '我的', icon: app.img.chairman_un_selcet_mine, selected: app.img.chairman_selcet_mine },
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

* PDShop/project/App/modules/common/Bar.js

```js
'use strict';
var React = require('react');
var ReactNative = require('react-native');
var {
    Image,
    StyleSheet,
    Text,
    View,
} = ReactNative;

import Echarts from '@remobile/react-native-echarts';

module.exports = React.createClass({
    render () {
        const option = {
            xAxis: {
                data: ['5', '15', '25', '35', '45', '55'],
            },
            yAxis: {},
            series: [{
                name: '销量',
                type: 'bar',
                data: [5, 20, 36, 10, 10, 20],
            }],
        };
        return (
            <Echarts option={option} height={300} />
        );
    },
});

var styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: 'white',
    },
    chart: {
        width: sr.w,
        height: 100,
        marginBottom: 60,
    },
});

```

* PDShop/project/App/modules/common/CellPhoneBox.js

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

* PDShop/project/App/modules/common/Circle.js

```js
'use strict';
var React = require('react');
var ReactNative = require('react-native');
var {
    Image,
    StyleSheet,
    Text,
    View,
} = ReactNative;

import Echarts from '@remobile/react-native-echarts';

module.exports = React.createClass({
    render () {
        const option = {
            legend: {
                orient: 'vertical',
            //   bottom: '0',
                data:['今日数', '往日数', '剩余数'],
            },
            series: [{
                label: {
                    normal: {
                        show: false,
                        position: 'center',
                    },
                },
                name: '销量',
                type: 'pie',
                radius:[62, 88],
                data: [ { value:335, name:'今日数' },
                { value:310, name:'往日数' },
                { value:274, name:'剩余数' }],
            }],
        };
        return (
            <Echarts option={option} />
        );
    },
});

var styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: 'white',
    },
    chart: {
        width: sr.w,
        // height: 100,
        marginBottom: 60,
    },
});

```

* PDShop/project/App/modules/common/DigitalKeyboard.js

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
                                <Text style={styles.item}>{item}</Text>
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

* PDShop/project/App/modules/common/ImageCrop.js

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
            app.scene.cropImage();
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

* PDShop/project/App/modules/common/ImagePicker.js

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

* PDShop/project/App/modules/common/Line.js

```js
'use strict';
var React = require('react');
var ReactNative = require('react-native');
var {
    Image,
    StyleSheet,
    Text,
    ScrollView,
} = ReactNative;

import Echarts from '@remobile/react-native-echarts';

module.exports = React.createClass({

    render () {
        const { title, data, day } = this.props;
        const option = {
            grid:{ show:true },
            legend: {
                data:title,
            },
            toolbox: {
                show : true,
                orient : 'vertical',
                y : 130,
                x : 'right',
                feature : {
                    mark : { show: true },
                    magicType : {
                        show: true,
                        title : { line : '折线图', bar : '柱状图' },
                        type: ['line', 'bar'] },
                },
            },
            xAxis: {
                data: day,
            },
            yAxis : [
                {
                    type : 'value',
                },
            ],
            series: data,
        };
        return (
            <ScrollView style={styles.container}>
                <Echarts option={option} height={300} />
            </ScrollView>
        );
    },
});

var styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: 'white',
        marginBottom:50,
    },
});

```

* PDShop/project/App/modules/common/PayPassword.js

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

const list = [1, 2, 3, 4, 5, 6, 7, 8, 9];

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
    passwordItem (number) {
        const { password } = this.state;
        this.setState({ password: password + number });
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
        console.log('tmp', tmp);
        if (tmp.length == 6) {
            this.setState({
                type : !type,
            });
            this.toPay(tmp);
        } else {
            Toast('支付密码有误');
            this.toClear();
        }
    },
    toPay (tmp) {
        const { password } = this.state;
        let { param, routeName } = this.props;
        param['payPassword'] = tmp;
        POST(routeName, param, this.toPaySuccess, true);
    },
    toPaySuccess (data) {
        const { prompt, amount, doCloseActionSheet, money, alreadyPaid, refresh } = this.props;
        if (data.success) {
            app.showModal(
                <MessageBox
                    onConfirm={this.doConfirmDelete}
                    content={prompt}
                    width={sr.s(250)} />
            );
            !!doCloseActionSheet && doCloseActionSheet();
            !!amount && app.personal.setRemainAmount(parseInt(app.personal.remainAmount) + parseInt(amount));
            !!money && app.personal.setRemainAmount(parseInt(app.personal.remainAmount) - parseInt(money));
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
    toClear () {
        this.setState({
            password: '',
        });
    },
    render () {
        const max = [1, 2, 3, 4, 5, 6];
        const { password, type } = this.state;
        console.log('password', password);
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
                    !type && <View>
                        <DigitalKeyboard passwordItem={this.passwordItem}
                            list={list}
                            zero={this.zero}
                            delete={this.delete}
                            confirm={this.confirm} />
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

* PDShop/project/App/modules/common/Pie.js

```js
'use strict';
var React = require('react');
var ReactNative = require('react-native');
var {
    Image,
    StyleSheet,
    Text,
    View,
} = ReactNative;

import Echarts from '@remobile/react-native-echarts';

module.exports = React.createClass({
    render () {
        const option = {
            series: [{
                name: '销量',
                type: 'pie',
                data: [{ value:35, name:'男' },
                { value:310, name:'女' }],

            }],
        };
        return (
            <Echarts option={option} height={300} />
        );
    },
});

var styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: 'white',
    },
    chart: {
        width: sr.w,
        height: 100,
        marginBottom: 60,
    },
});

```

* PDShop/project/App/modules/common/ScanBarCode.js

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

* PDShop/project/App/modules/common/SelectClient.js

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
        title: (<Title doStartSearch={() => { app.scene.doStartSearch(); }} />),
        rightButton: { title: '搜索', handler: () => { app.scene.refresh(); } },
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

* PDShop/project/App/modules/common/SelectRoadmap.js

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
        title: (<Title doStartSearch={() => { app.scene.doStartSearch(); }} />),
        rightButton: { title: '搜索', handler: () => { app.scene.refresh(); } },
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

* PDShop/project/App/modules/common/ShowBigImage.js

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

* PDShop/project/App/modules/login/ForgetPassword.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
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
            button: false,
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
            this.setState({
                button: true,
            });
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
            Toast('请填写正确的验证码');
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
    doInput () {
        Toast('请先获取验证码');
    },
    render () {
        const { readSecond, button } = this.state;
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
                <Button onPress={button ? this.doNext : this.doInput}
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

* PDShop/project/App/modules/login/Login.js

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

const { Button } = COMPONENTS;
const ForgetPassword = require('./ForgetPassword');
const Register = require('./Register');
const Chairman = require('../chairman');
const WarehouseDepartment = require('../warehouseDepartment');
const ReceiveDepartment = require('../receiveDepartment');
const MovingDepartment = require('../movingDepartment');
const SecurityCheckDepartment = require('../securityCheckDepartment');
const ENTRY_HOME = [ ReceiveDepartment, WarehouseDepartment, MovingDepartment, SecurityCheckDepartment, Chairman ];

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
                component: ENTRY_HOME[app.personal.info.partment ? app.personal.info.partment.type : 4],
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
        const newData = _.filter(app.login.list, (item) => { const reg = new RegExp('^' + text + '.*'); return reg.test(item); });
        this.setState({
            phone: text,
            dataSource: dataSource.cloneWithRows(newData),
            showList: newData.length > 0 && text.length < 11,
        });
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
                            style={styles.input_icon} />
                        <TextInput
                            underlineColorAndroid='transparent'
                            placeholder='您的手机号码'
                            maxLength={11}
                            onChangeText={this.onPhoneTextChange}
                            defaultValue={this.state.phone}
                            style={styles.text_input}
                            keyboardType='phone-pad'
                            onFocus={this.onFocus}
                            onBlur={this.onBlur} />
                    </View>
                    <View style={styles.listView} />
                    <View style={styles.passwordContainer}>
                        <Image
                            resizeMode='stretch'
                            source={app.img.login_password}
                            style={styles.input_icon} />
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
                        keyboardShouldPersistTaps="always"
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
        marginLeft: (sr.w - 300) / 2,
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

* PDShop/project/App/modules/login/PasswordConfirm.js

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

* PDShop/project/App/modules/login/Register.js

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
        const { protocalRead, phone, password, rePassword, verification } = this.state;
        if (!protocalRead) {
            Toast('注册前请先阅读用户协议');
            return;
        }
        if (!app.utils.checkPhone(phone)) {
            Toast('请填写正确的手机号码');
            return;
        }
        if (!app.utils.checkVerificationCode(verification)) {
            Toast('请填写正确的验证码');
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
        // if (!app.utils.checkMailAddress(email)) {
        //     Toast('邮箱地址不规范，请重新输入');
        //     return;
        // }
        const param = {
            phone,
            verification,
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
            verification: '',
            protocalRead: true,
            overlayShow:false,
        };
    },
    changeProtocalState () {
        this.setState({ protocalRead: !this.state.protocalRead });
    },
    render () {
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
                        onChangeText={(text) => this.setState({ verification: text })}
                        style={styles.text_input2}
                        />
                    <TouchableOpacity>
                        <Text style={styles.text_verification}>获取验证码</Text>
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
    text_verification:{
        color:'#f99000',
        fontSize:15,
        marginRight:20,
    },
    text_input: {
        marginLeft:14,
        fontSize:15,
        height:40,
        width: 180,
        alignSelf: 'center',
    },
    text_input2: {
        marginLeft: 12,
        fontSize:15,
        height: 40,
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
});

```

* PDShop/project/App/modules/movingDepartment/MessageDetail.js

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
        title: '搬运消息',
    },

    render () {
        const { data } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.messageContainer}>
                    <Text style={styles.messageText}>搬运消息</Text>
                    <View style={styles.itemTop}>
                        <Text style={styles.itemLeft}>发起人：</Text>
                        <Text style={styles.itemRight}>{data.plateNo}</Text>
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.itemLeft}>时间：</Text>
                        <Text style={styles.itemRight}>{data.createTime}</Text>
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
        paddingTop:10,
    },
    messageContainer:{
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
    },
    messageText:{
        fontSize:14,
        color:'#333333',
        marginLeft:10,
        marginBottom:5,
        marginTop:7,
    },
    itemTop:{
        flexDirection:'row',
        marginTop:5,
    },
    item:{
        flexDirection:'row',
        marginTop:3,
        marginBottom:5,
    },
    itemLeft:{
        fontSize:12,
        color:'#888888',
        marginLeft:10,
    },
    itemRight:{
        fontSize:12,
        color:'#333333',
    },
    itemBottom:{
        flexDirection:'row',
        marginTop:2,
        marginBottom:5,
    },
});

```

* PDShop/project/App/modules/movingDepartment/MovingDetail.js

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

const { PageList } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '货单',
    },

    renderRow (obj, rowID) {
        const { stateList } = obj;
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
                <View style={styles.rowMiddleContainer} >
                    <Image
                        resizeMode='stretch'
                        source={{ uri: obj.photo }}
                        defaultSource={app.img.common_default}
                        style={styles.middleImage} />
                    <View style={styles.middleRight}>
                        <View style={styles.pointContainer}>
                            <View style={styles.placeContainer}>
                                <Text style={styles.pointName}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{obj.shop ? obj.shop.address : obj.agent ? obj.agent.address : ''}</Text>
                                <Text style={styles.pointName}>→</Text>
                                <Text style={styles.pointName}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{obj.endPoint}</Text>
                            </View>
                        </View>
                        <Text style={styles.commonInfo}>重量：{obj.weight}吨</Text>
                        <Text style={styles.commonInfo}>方量：{obj.size}m³</Text>
                        <Text style={styles.commonInfo}>件数：{obj.totalNumbers}件</Text>
                        <Text style={styles.commonInfo}>收货地址：{obj.endPoint}</Text>
                    </View>
                </View>
                <View style={styles.line} />
            </View>
        );
    },
    render () {
        const { truckId } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.separator} />
                <PageList
                    ref={(ref) => { this.listView = ref; }}
                    renderRow={this.renderRow}
                    listParam={{ userId: app.personal.info.userId, truckId }}
                    listName={'orderList'}
                    renderSeparator={null}
                    listUrl={app.route.ROUTE_GET_ORDER_LIST_IN_RUCKS}
                    refreshEnable />
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
    rowMiddleContainer: {
        flexDirection: 'row',
        alignItems: 'center',
        paddingVertical:11,
        backgroundColor:'#f8f8f8',
    },
    middleImage: {
        width: 80,
        height:80,
        marginLeft: 10,
    },
    middleRight: {
        paddingLeft: 10,
        width:sr.w - 100,
    },
    money: {
        fontSize: 14,
        color:'#f51b1a',
    },
    commonInfo: {
        fontSize: 13,
        color:'#888888',
    },
    pointName: {
        fontSize: 14,
        maxWidth: 70,
    },
    transformFee: {
        flex:1,
        fontSize: 14,
        color: 'gray',
        textAlign:'right',
    },
    pointContainer: {
        flexDirection: 'row',
    },
    placeContainer: {
        flex:1,
        flexDirection: 'row',
    },
    rowButtomContainer: {
        backgroundColor:'white',
        flexDirection: 'row',
        justifyContent: 'flex-end',
        paddingVertical:5,
    },
    ConfirmReach: {
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
    btnConfirmReachText:{
        color:'#ff6200',
        fontSize:12,
        fontWeight:'300',
    },
    btnDetallText:{
        color:'#007ef7',
        fontSize:12,
        fontWeight:'300',
    },
    line:{
        height:8,
        backgroundColor:'#FFFFFF',
    },
});

```

* PDShop/project/App/modules/movingDepartment/MovingHistory.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    ListView,
    Image,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '历史消息',
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            list: [],
        };
    },
    renderRow (obj) {
        return (
            <View style={styles.row}>
                <View style={styles.item}>
                    <Image source={app.img.moving_message} style={styles.message} />
                    <View style={styles.itemRight}>
                        <View style={styles.msg}>
                            <Text style={styles.itemRightTop}>搬运消息</Text>
                        </View>
                        <View style={styles.msgGray}>
                            <Text style={styles.itemRightBottom}
                                numberOfLines={1}>您收到来自{obj.plateNo}的一条消息，请点击查看。</Text>
                        </View>
                    </View>
                </View>
                <View style={styles.msg}>
                    <Text style={styles.itemDate}>{obj.createTime}</Text>
                </View>
            </View>
        );
    },
    render () {
        return (
            <View style={styles.container} />
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
        paddingTop:10,
    },
});

```

* PDShop/project/App/modules/movingDepartment/MovingMessage.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    ScrollView,
    RefreshControl,
    Image,
    TouchableOpacity,
} = ReactNative;

const MessageDetail = require('./MessageDetail');

module.exports = React.createClass({
    statics: {
        title: '消息',
        // rightButton: { title: '历史', handler: () => {app.scene.toHistory();} },
    },
    onWillFocus () {
        app.getCurrentRoute().title = '消息';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    // toHistory(){
    //     app.push({
    //         component:MovingHistory,
    //     });
    // },
    getInitialState () {
        return {
            data: {},
            nodata: false,
        };
    },
    componentWillMount () {
        this.getWaitScanTruckInfo();
    },
    getWaitScanTruckInfo () {
        const param = {
            userId: app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_WAIT_SCAN_TRUCK_INFO, param, this.getWaitScanTruckInfoSuccess, false);
    },
    getWaitScanTruckInfoSuccess (data) {
        if (data.success) {
            this.setState({
                nodata: false,
                data: data.context,
            });
        } else {
            this.setState({ nodata: true });
        }
    },
    messageDetail () {
        const { data } = this.state;
        app.push({
            component: MessageDetail,
            passProps:{
                data: data,
            },
        });
    },
    onScroll () {
        this.getWaitScanTruckInfo();
    },
    render () {
        const { data, nodata } = this.state;
        return (
            <ScrollView
                refreshControl={
                    <RefreshControl
                        refreshing={false}
                        onRefresh={this.onScroll}
                        title='正在刷新...' />
                }
                style={styles.container}>
                {
                    nodata ?
                        <View>
                            <Image source={app.img.moving_no_message} style={styles.onMessage} />
                            <Text style={styles.onMessageText}>暂无消息</Text>
                        </View>
                    :
                        <TouchableOpacity onPress={this.messageDetail}>
                            <View style={styles.row}>
                                <View style={styles.item}>
                                    <Image source={app.img.moving_message} style={styles.message} />
                                    <View style={styles.itemRight}>
                                        <View style={styles.msg}>
                                            <Text style={styles.itemRightTop}>搬运消息</Text>
                                        </View>
                                        <View style={styles.msgGray}>
                                            <Text style={styles.itemRightBottom}
                                                numberOfLines={1}>您收到来自{data.plateNo}的一条消息，请点击查看。</Text>
                                        </View>
                                    </View>
                                </View>
                                <View style={styles.msg}>
                                    <Text style={styles.itemDate}>{data.createTime}</Text>
                                </View>
                            </View>
                        </TouchableOpacity>
                }
            </ScrollView>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#F4F4F4',
        paddingTop:10,
    },
    onMessage:{
        height:120,
        width:126,
        marginLeft:(sr.w - 126) / 2,
        marginTop:65,
    },
    onMessageText:{
        fontSize:17,
        color:'#888888',
        marginTop:15,
        textAlign:'center',
    },
    row:{
        height:90,
        marginTop:1,
        backgroundColor:'#FFFFFF',
    },
    item:{
        height:60,
        flexDirection:'row',
    },
    msg:{
        justifyContent:'center',
        height:30,
    },
    msgGray:{
        justifyContent:'center',
        height:30,
        backgroundColor:'#f8f8f8',
    },
    message:{
        height:40,
        width:40,
        margin:10,
    },
    itemRight:{
        height:60,
        width:294,
        marginLeft:5,
    },
    itemRightTop:{
        padding:0,
        fontSize:15,
        textAlign:'justify',
    },
    itemRightBottom:{
        fontSize:12,
        padding:0,
        color:'#888888',
        paddingLeft:10,
        textAlign:'justify',
    },
    itemDate:{
        marginLeft:65,
        padding:0,
        color:'#888888',
        textAlign:'justify',
    },
    line:{
        height:10,
    },
});

```

* PDShop/project/App/modules/movingDepartment/MovingOrder.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    ListView,
    RefreshControl,
    TouchableOpacity,
} = ReactNative;

const MovingDetail = require('./MovingDetail');

module.exports = React.createClass({
    statics: {
        title: '搬运',
    },
    onWillFocus () {
        app.getCurrentRoute().title = '搬运';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    getInitialState () {
        return {
            ds: new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 }),
            list: [],
        };
    },
    componentWillMount () {
        this.getCarryList();
    },
    getCarryList () {
        const param = {
            userId: app.personal.info.userId,
            pageNo: 0,
            pageSize: 10,
        };
        POST(app.route.ROUTE_GET_CARRY_LIST, param, this.getCarryListSuccess, true);
    },
    getCarryListSuccess (data) {
        if (data.success) {
            this.setState({
                list: data.context.truckList,
            });
        } else {

        }
    },
    toMovingDetail (obj) {
        app.push({
            component:MovingDetail,
            passProps:{
                truckId: obj.id,
            },
        });
    },
    onScroll () {
        this.getCarryList();
    },
    renderRow (obj) {
        return (
            <TouchableOpacity onPress={this.toMovingDetail.bind(null, obj)}>
                <View style={styles.row}>
                    <View style={styles.itemTop}>
                        <Text style={styles.itemTopLeft}>{obj.plateNo}</Text>
                        <Text style={styles.itemTopRight}>¥{app.utils.N(obj.carryPrice * obj.totalWeight)}元</Text>
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.itemLeft}>单数：</Text>
                        <Text style={styles.itemRight}>{obj.orderCount}单</Text>
                    </View>
                    <View style={styles.item}>
                        <Text style={styles.itemLeft}>重量：</Text>
                        <Text style={styles.itemRight}>{app.utils.N(obj.totalWeight)}吨</Text>
                    </View>
                    <View style={styles.itemBottom}>
                        <Text style={styles.itemLeft}>时间：</Text>
                        <Text style={styles.itemRight}>{obj.createTime}</Text>
                    </View>
                </View>
            </TouchableOpacity>
        );
    },
    render () {
        return (
            <View style={styles.container}>
                <ListView dataSource={this.state.ds.cloneWithRows(this.state.list)}
                    enableEmptySections
                    renderRow={this.renderRow}
                    refreshControl={
                        <RefreshControl
                            refreshing={false}
                            onRefresh={this.onScroll}
                            title='正在刷新...' />
                   } />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        paddingTop:9,
    },
    row:{
        backgroundColor:'#FFFFFF',
        marginTop:1,
    },
    messageText:{
        fontSize:14,
        color:'#333333',
        margin:10,
    },
    itemTop:{
        flexDirection:'row',
        justifyContent:'space-between',
        marginTop:6,
        marginLeft:10,
        marginBottom:2,
    },
    itemTopLeft:{
        fontSize:14,
        color:'#333333',
    },
    itemTopRight:{
        fontSize:14,
        color:'red',
        marginRight:10,
    },
    item:{
        flexDirection:'row',
        marginTop:3,
    },
    itemLeft:{
        fontSize:12,
        color:'#888888',
        marginLeft:10,
    },
    itemRight:{
        fontSize:12,
        color:'#333333',
    },
    itemBottom:{
        flexDirection:'row',
        marginTop:3,
        marginBottom:5,
    },
});

```

* PDShop/project/App/modules/movingDepartment/index.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
} = ReactNative;

import TabNavigator from 'react-native-tab-navigator';

const MovingMessage = require('./MovingMessage');
const MovingOrder = require('./MovingOrder');
const Person = require('../person');

const INIT_ROUTE_INDEX = 0;
const ROUTE_STACK = [
    { index: 0, component: MovingMessage },
    { index: 1, component: MovingOrder },
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
            { index: 0, title: '消息', icon:app.img.moving_message_unselect, selected: app.img.moving_message_select },
            { index: 1, title: '搬运', icon: app.img.moving_moving_unselect, selected: app.img.moving_moving_select },
            { index: 3, title: '我的', icon: app.img.moving_person_unselect, selected: app.img.moving_person_select },
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

* PDShop/project/App/modules/person/About.js

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

* PDShop/project/App/modules/person/BalanceDetail.js

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
                    <Text style={styles.itemLeft}> 平台 </Text>
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

* PDShop/project/App/modules/person/BalanceDetailList.js

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
                    refreshEnable />
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

* PDShop/project/App/modules/person/BindBankcard.js

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
    render () {
        return (
            <View style={styles.container}>
                <View style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <Text style={styles.itemNameText}>{'账号'}</Text>
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
                        <Text style={styles.itemNameText}>{'姓名'}</Text>
                        <TextInput style={styles.itemNameLeftText}
                            underlineColorAndroid='transparent'
                            placeholderTextColor='#aaaaaa'
                            placeholder='请填入姓名'
                            onChangeText={this.onContentTextChange}
                            />
                    </View>
                    <View style={styles.lineView} />
                </View>
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
        height: 44,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
    },
    infoStyle: {
        flexDirection: 'row',
    },
    icon_add:{
        flex:1,
        width:20,
        height:30,
    },
    itemNameText: {
        fontSize: 15,
        color: '#444444',
        marginLeft: 8,
    },
    itemNameLeftText: {
        flex:1,
        fontSize: 15,
        color: '#444444',
        marginLeft: 8,
        padding:0,
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

* PDShop/project/App/modules/person/BindBankcardList.js

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

const { MessageBox } = COMPONENTS;
const BindBankcard = require('./BindBankcard');

module.exports = React.createClass({
    statics: {
        title: '银行卡',
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {
            list: [1, 2, 1, 1, 1, 1, 11, 1],
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
            <View>
                <View style={styles.line} />
                <View style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <View >
                            <Text style={styles.itemNameText}>{'农业银行'}</Text>
                            <Text style={styles.itemNumberText}>{'**0851' + '储蓄卡'}</Text>
                        </View>
                        <TouchableOpacity
                            activeOpacity={0.6}
                            onPress={() => this.unBindBankcard(rowID)}>
                            <Text style={styles.itemNameRightText}>{'解绑'}</Text>
                        </TouchableOpacity>
                    </View>
                </View>
            </View>
        );
    },
    renderFooter (obj, sectionID, rowID) {
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
    },
    line:{
        height:1,
        width:sr.w,
        backgroundColor:'#f4f4f4',
    },
    ListView:{
        marginTop:7,
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

* PDShop/project/App/modules/person/CashBalance.js

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
        rightButton: { title: '明细', handler: () => { app.scene.detail(); } },
    },
    detail () {
        app.push({
            component:BalanceDetailList,
        });
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
        POST(app.route.ROUTE_GET_REMAIN_AMOUNT, param, this.getRemainAmountSuccess, true);
    },
    getRemainAmountSuccess (data) {
        if (data.success) {
            const context = data.context;
            app.personal.setRemainAmount(context.amount);
            this.setState({ remainAmount:app.personal.remainAmount });
        }
    },
    showWithdrawCash () {
        app.push({
            component: WithdrawCash,
        });
    },
    showRecharge () {
        app.push({
            component: Recharge,
            passProps:{
                setRemainAmount:this.setRemainAmount,
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
                        <Text style={styles.CashBalanceTitleLeft}>￥</Text><Text style={styles.CashBalanceContent}>{remainAmount}</Text>
                    </View>
                </View>
                {
                    (_.intersection(app.personal.info.authority, [AH.AH_WITHDRAW]).length != 0) &&
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
                    (_.intersection(app.personal.info.authority, [AH.AH_RECHARGE]).length != 0) &&
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
        alignItems:'center',
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

* PDShop/project/App/modules/person/CommonSetting.js

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

* PDShop/project/App/modules/person/ConfirmPayPassword.js

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
        leftButton: { handler: () => { app.scene.goBack(); } },
    },
    goBack () {
        this.props.toClear();
        app.pop();
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
            this.toNext(tmp);
        } else {
            Toast('支付密码有误');
            this.toClear();
        }
    },
    toNext (password) {
        const { number, verifyCode } = this.props;
        if (number == password) {
            const param = {
                userId:app.personal.info.userId,
                verifyCode,
                phone:app.personal.info.phone,
                password,
            };
            POST(app.route.ROUTE_SET_PAYMENT_PASSWORD, param, this.modifySuccess, true);
        } else {
            Toast('两次输入密码不一致，请重试');
            this.goBack();
        }
    },
    modifySuccess (data) {
        if (data.success) {
            const info = app.personal.info;
            info.isSetPaymentPassword = 1;
            app.personal.set(info);
            Toast('修改成功');
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
                {
                   !type && <View>
                       <DigitalKeyboard list={list}
                           passwordItem={this.passwordItem}
                           zero={this.zero}
                           delete={this.delete}
                           confirm={this.confirm} />
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

* PDShop/project/App/modules/person/EditPersonInfo.js

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

const { DImage, Button, Picker } = COMPONENTS;

const ImagePicker = require('./ImagePicker');
const EditSingleInfo = require('./EditSingleInfo');

module.exports = React.createClass({
    mixins: [SceneMixin],
    statics: {
        title: '个人信息',
        leftButton: { handler: () => { app.scene.goBack(); } },
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
    toEditSingleInfo (name, tip, maxLength) {
        Picker.hide();
        app.push({
            title: '修改' + tip,
            component: EditSingleInfo,
            passProps: {
                keyName:name,
                placeholder:tip,
                maxLength,
                updateSingleState:this.updateSingleState,
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
            userId: app.personal.info.id,
            sex: sex == '男' ? 0 : 1,
        };
        POST(app.route.ROUTE_MODIFY_PERSONAL_INFO, param, this.modifyUserInfoSuccess, true);
    },
    modifyUserInfoSuccess (data) {
        if (data.success) {
            app.personal.set(data.context);
        } else {
            Toast(data.msg);
        }
    },
    doLogout () {
        app.navigator.resetTo({
            component: require('../login/Login'),
        }, 0);
        app.personal.setNeedLogin(true);
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
                    onPress={this.toEditSingleInfo.bind(null, 'name', '昵称', CONSTANTS.NAME_MAX_LENGTH)}
                    style={styles.itemView} >
                    <Text style={styles.itemKeyText}>昵称</Text>
                    <View style={styles.rightView} >
                        <Text style={styles.itemValueText}>{name}</Text>
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
                    onPress={this.toEditSingleInfo.bind(null, 'address', '地址', CONSTANTS.ADDRESS_MAX_LENGTH)}
                    style={styles.itemView}>
                    <Text style={styles.itemKeyText}>地址</Text>
                    <View style={styles.rightView} >
                        <Text style={styles.itemValueText} numberOfLines={1}>{address}</Text>
                        <Image
                            source={app.img.common_go}
                            style={styles.goStyle} />
                    </View>
                </TouchableOpacity>
                <View
                    style={styles.itemView}>
                    <Text style={styles.itemKeyText}>手机号码</Text>
                    <View style={styles.rightView} >
                        <Text style={styles.itemValueText}>{phone}</Text>
                    </View>
                </View>
                {
                    !app.personal.info.shipper &&
                    <Text />
                    ||
                    <TouchableOpacity
                        style={styles.itemView}>
                        <Text style={styles.itemKeyText}>所属物流公司</Text>
                        <View style={styles.rightView} >
                            <Text style={styles.itemValueText}>{app.personal.info.shipper.name}</Text>
                        </View>
                    </TouchableOpacity>
                }
                {
                    !app.personal.info.shipper &&
                    <Text />
                    ||
                    <TouchableOpacity
                        style={styles.itemView}>
                        <Text style={styles.itemKeyText}>职务</Text>
                        <View style={styles.rightView} >
                            <Text style={styles.itemValueText}>{app.personal.info.post}</Text>
                        </View>
                    </TouchableOpacity>
                }

                <Button onPress={this.doLogout} style={styles.btnLogout} textStyle={styles.btnLogoutText} >
                    退出登录
                </Button>
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
    btnLogout:{
        position:'absolute',
        top:sr.h - 250,
        height:45,
        width:sr.w - 25,
        borderRadius:2,
        backgroundColor:'#D9D9D9',
    },
    btnLogoutText:{
        color:'#000000',
        fontSize:16,
        fontWeight:'300',
    },
});

```

* PDShop/project/App/modules/person/EditSex.js

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
        leftButton: { handler: () => { app.scene.goBack(); } },
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
            const info = app.personal.info;
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

* PDShop/project/App/modules/person/EditSingleInfo.js

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
        rightButton: { title: '保存', handler: () => { app.scene.doSave(); } },
    },
    doSave () {
        const { placeholder } = this.props;
        const info = app.personal.info;
        const { value } = this.state;
        if (!!app.utils.checkStr && !app.utils.checkStr(value)) {
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
            const info = app.personal.info;
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

* PDShop/project/App/modules/person/Feedback.js

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
            content:text,
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
                    placeholderTextColor='#aaaaaa'
                    placeholder='请填入意见内容'
                    onChangeText={this.onContentTextChange}
                    multiline
                    underlineColorAndroid='transparent'
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

* PDShop/project/App/modules/person/Help.js

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

* PDShop/project/App/modules/person/ImageCrop.js

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
            app.scene.cropImage();
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
            const info = app.personal.info;
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

* PDShop/project/App/modules/person/ImagePicker.js

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

* PDShop/project/App/modules/person/ModifyPassword.js

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

* PDShop/project/App/modules/person/MovingTeamInfo.js

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
const MovingTeamSlogan = require('./MovingTeamSlogan');

module.exports = React.createClass({
    statics: {
        title: '搬运队信息',
        rightButton: { title: '确定', handler: () => { app.scene.doConfirm(); } },
        leftButton: { handler: () => { app.scene.doBack(); } },
    },
    doBack () {
        this.props.refresh();
        app.pop();
    },
    doConfirm () {
        const { phone1, phone2, phone3, slogan, price } = this.state;
        let list = (
            (phone1 == '' ? '' : phone1 + ';') +
            (phone2 == '' ? '' : phone2 + ';') +
            (phone3 == '' ? '' : phone3 + ';')
        );
        if (price ? price.length == 0 : true) {
            Toast('搬运价格不能为空');
            return;
        }
        const param = {
            userId: app.personal.info.userId,
            descript: slogan,
            phoneList: list,
            price: (price || 0),
        };
        POST(app.route.ROUTE_MODIFY_OWN_PARTMENT, param, this.modifyOwnPartmentSuccess, true);
    },
    modifyOwnPartmentSuccess (data) {
        if (data.success) {
            Toast('修改成功');
        } else {
            Toast('修改失败');
        }
    },
    getInitialState () {
        const { data } = this.props;
        let length = data.phoneList.split(';').length;
        return {
            chargeMan: data.chargeMan,
            phone1: (length >= 1 ? data.phoneList.split(';')[0] : ''),
            phone2: (length >= 2 ? data.phoneList.split(';')[1] : ''),
            phone3: (length >= 3 ? data.phoneList.split(';')[2] : ''),
            price: data.price,
            slogan: this.props.slogan,
        };
    },
    toModifySlogan () {
        const { slogan } = this.state;
        app.push({
            component:MovingTeamSlogan,
            passProps:{
                setSlogan:this.modifySlogan,
                slogan,
            },
        });
    },
    modifySlogan (newSlogan) {
        this.setState({
            slogan: newSlogan,
        });
    },
    render () {
        const { slogan, chargeMan, phone1, phone2, phone3, price } = this.state;
        const { data } = this.props;
        return (
            <ScrollView style={styles.container}>
                <View style={styles.TeamInfo}>
                    <View style={styles.rowGray}>
                        <Text style={styles.titleInfo}>负责人</Text>
                        <Text style={styles.info}>{chargeMan.name}</Text>
                    </View>
                    <View style={styles.rowGray}>
                        <Text style={styles.titleInfo}>负责人电话</Text>
                        <Text style={styles.slogan}>{chargeMan.phone}</Text>
                    </View>
                    <View style={styles.rowGray}>
                        <Text style={styles.titleInfo}>搬运队名字</Text>
                        <Text style={styles.info}>{data.name}</Text>
                    </View>
                    <View style={styles.rowGray}>
                        <Text style={styles.titleInfo}>人数</Text>
                        <Text style={styles.info}>{data.memberCount}</Text>
                    </View>
                    <View style={styles.rowLine} />
                    <View style={styles.row}>
                        <Text style={styles.titleInfo}>价格 / 吨</Text>
                        <TextInput
                            underlineColorAndroid='transparent'
                            style={styles.info}
                            keyboardType='numeric'
                            maxLength={5}
                            placeholderTextColor='#888888'
                            onChangeText={(text) => this.setState({ price: text })}
                            placeholder='请输入搬运队价格'
                            defaultValue={price + ''} />
                    </View>
                    <TouchableOpacity onPress={this.toModifySlogan}>
                        <View style={styles.row}>
                            <Text style={styles.titleInfo}>口号</Text>
                            <Text style={styles.slogan}>{slogan}</Text>
                            <Image source={app.img.common_go} style={styles.imgGo} resizeMode='contain' />
                        </View>
                    </TouchableOpacity>
                    <View style={styles.rowLine} />
                    <View style={styles.row} >
                        <Text style={styles.titleInfo}>联系电话</Text>
                        <TextInput
                            underlineColorAndroid='transparent'
                            style={styles.info}
                            onChangeText={(text) => this.setState({ phone1: text })}
                            maxLength={11}
                            keyboardType='phone-pad'
                            placeholderTextColor='#888888'
                            placeholder='请输入电话号码'
                            defaultValue={phone1} />
                    </View>
                    <View style={styles.row} >
                        <Text style={styles.titleInfo}>联系电话</Text>
                        <TextInput
                            underlineColorAndroid='transparent'
                            style={styles.info}
                            onChangeText={(text) => this.setState({ phone2: text })}
                            maxLength={11}
                            keyboardType='phone-pad'
                            placeholderTextColor='#888888'
                            placeholder='请输入电话号码'
                            defaultValue={phone2} />
                    </View>
                    <View style={styles.row} >
                        <Text style={styles.titleInfo}>联系电话</Text>
                        <TextInput
                            underlineColorAndroid='transparent'
                            style={styles.info}
                            onChangeText={(text) => this.setState({ phone3: text })}
                            maxLength={11}
                            keyboardType='phone-pad'
                            placeholderTextColor='#888888'
                            placeholder='请输入电话号码'
                            defaultValue={phone3} />
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
    TeamInfo:{
        marginTop:10,
    },
    row:{
        flexDirection:'row',
        height:42,
        alignItems:'center',
        backgroundColor:'#FFFFFF',
        marginTop:1,
    },
    rowGray:{
        flexDirection:'row',
        height:42,
        alignItems:'center',
        backgroundColor:'#fafafa',
        marginTop:1,
    },
    rowLine:{
        marginTop:5,
    },
    info:{
        flex:1,
        fontSize:12,
        color:'#333333',
        textAlign:'right',
        marginRight:10,
    },
    titleInfo:{
        fontSize:14,
        flex:1,
        marginLeft:10,
    },
    slogan:{
        flex:1,
        fontSize:14,
        fontWeight:'300',
        textAlign:'right',
        paddingRight:10,
    },
    imgGo:{
        width:15,
        height:15,
        marginRight:10,
        marginLeft:5,
    },
});

```

* PDShop/project/App/modules/person/MovingTeamSlogan.js

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

module.exports = React.createClass({
    statics: {
        title: '搬运队口号',
        rightButton: { title: '保存', handler: () => { app.scene.doSave(); } },
    },
    getInitialState () {
        return {
            content: this.props.slogan,
        };
    },
    doSave () {
        const { content } = this.state;

        this.props.setSlogan(content);
        app.pop();
    },
    onContentTextChange (text) {
        this.setState({
            content:text,
        });
    },
    render () {
        const { content } = this.state;
        return (
            <View style={styles.container}>
                <TextInput style={styles.item}
                    maxLength={20}
                    placeholder='请为您的搬运队起一个响亮的口号'
                    placeholderTextColor='#dddddd'
                    onChangeText={this.onContentTextChange}
                    underlineColorAndroid='transparent'
                    defaultValue={content}
                    multiline />
                <Text style={styles.wordnumber_text}>
                    {content ? content.length : '0'}/20
                </Text>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    item:{
        margin:10,
        paddingLeft:10,
        height:75,
        backgroundColor:'#FFFFFF',
        fontSize:14,
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
});

```

* PDShop/project/App/modules/person/MyRemovalTeam.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;
const { Button, CustomSheet } = COMPONENTS;
const TransportQuotation = require('./TransportQuotation');
const MovingTeamInfo = require('./MovingTeamInfo');
const StatusSelection = require('./StatusSelection');

module.exports = React.createClass({
    statics: {
        title: '搬运队',
        rightButton: { title: '搬运行情', handler: () => { app.scene.showtransportQuotation(); } },
    },
    getInitialState () {
        return {
            actionSheetVisible: false,
            sign:'',
            data : [],
        };
    },
    componentWillMount () {
        this.getOwnPartmentInfo();
    },
    getOwnPartmentInfo () {
        const param = { userId: app.personal.info.userId };
        POST(app.route.ROUTE_GET_OWN_PARTMENT_INFO, param, this.getOwnPartmentInfoSuccess, false);
    },
    getOwnPartmentInfoSuccess (data) {
        if (data.success) {
            this.setState({
                data: data.context,
                sign: data.context.enable,
            });
        }
    },
    showtransportQuotation () {
        app.push({
            component:TransportQuotation,
        });
    },
    refresh () {
        this.getOwnPartmentInfo();
    },
    modifyMovingTeamInfo () {
        const { data } = this.state;
        app.push({
            component:MovingTeamInfo,
            passProps:{
                data,
                refresh: this.refresh,
                slogan: data.descript,
            },
        });
    },
    doCloseActionSheet () {
        this.setState({ actionSheetVisible:false });
    },
    doShowActionSheet () {
        this.setState({ actionSheetVisible:true });
    },
    setStatus (status) {
        const param = {
            userId: app.personal.info.userId,
            enable: status,
        };
        POST(app.route.ROUTE_MODIFY_OWN_PARTMENT, param, this.modifyOwnPartmentSuccess.bind(null, status), true);
    },
    modifyOwnPartmentSuccess (status, data) {
        if (data.success) {
            this.setState({ sign: status });
            Toast('修改成功');
        } else {
            Toast('修改失败');
        }
    },
    render () {
        const { data, sign } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.row}>
                    <View style={styles.nameContainer}>
                        <Text style={styles.name}>{data.name}</Text>
                        <Text style={styles.status}>{sign ? '营业' : '暂停营业'}</Text>
                    </View>
                    <Text style={styles.infoTop}>人数:{data.memberCount}人</Text>
                    <Text style={styles.info}>电话:{data.phoneList}</Text>
                    <View style={styles.priceRow}>
                        <Text style={styles.info}>价格:</Text>
                        <Text style={styles.price}>¥{data.price ? data.price : '0'}/{'吨'}</Text>
                    </View>
                </View>
                <View style={styles.btnRow}>
                    <Button onPress={this.modifyMovingTeamInfo} style={styles.btnModifyInfo} textStyle={styles.modifyInfoFont}>修改信息</Button>
                    <Button onPress={this.doShowActionSheet} style={styles.btnModifyStatus} textStyle={styles.modifyStatusFont}>修改状态</Button>
                </View>
                <CustomSheet
                    visible={this.state.actionSheetVisible} >
                    <StatusSelection doClose={this.doCloseActionSheet} setStatus={this.setStatus} />
                </CustomSheet>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    row:{
        backgroundColor:'#FFFFFF',
        marginTop:8,
        paddingLeft:10,
        justifyContent:'center',
    },
    nameContainer:{
        flexDirection:'row',
        justifyContent:'space-between',
        marginTop:5,
    },
    name:{
        fontSize:14,
        color:'#333333',
        marginBottom:3,
    },
    status:{
        color:'#f01825',
        marginRight:10,
    },
    infoTop:{
        fontSize:12,
        color:'#888888',
        marginTop:4,
    },
    info:{
        marginTop:4,
        fontSize:12,
        color:'#888888',
    },
    priceRow:{
        flexDirection:'row',
        width:90,
        marginBottom:5,
    },
    price:{
        flex:1,
        fontSize:12,
        marginTop:4,
        marginLeft:5,
        color:'#e7222d',
        fontWeight:'500',
    },
    btn:{
        width:75,
        height:22,
        backgroundColor:'#FFFFFF',
        borderRadius:4,
        borderWidth:0.5,
        borderColor:'#ee1822',
        position:'absolute',
        right:15,
        bottom:10,
    },
    btnFont:{
        color:'#ee1822',
        fontSize:12,
    },
    btnRow:{
        flexDirection:'row',
        marginTop:1,
        backgroundColor:'#FFFFFF',
        height:30,
    },
    btnModifyInfo:{
        alignSelf:'center',
        height:20,
        width:65,
        borderWidth:0.5,
        borderRadius:4,
        borderColor:'#FE9917',
        backgroundColor:'#FFFFFF',
        marginLeft:sr.w - 160,
    },
    modifyInfoFont:{
        fontSize:12,
        color:'#FE9917',
    },
    btnModifyStatus:{
        alignSelf:'center',
        height:20,
        width:65,
        borderWidth:0.5,
        borderRadius:4,
        borderColor:'#c61a29',
        backgroundColor:'#FFFFFF',
        marginLeft:15,
    },
    modifyStatusFont:{
        fontSize:12,
        color:'#c61a29',
    },
});

```

* PDShop/project/App/modules/person/NormalQuestions.js

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

* PDShop/project/App/modules/person/ParameterSetting.js

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

module.exports = React.createClass({
    statics: {
        title: '参数设置',
        rightButton: { title: '保存', handler: () => { app.scene.doSave(); } },
    },
    doSave () {
        const {
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
        } = this.state;
        try {
            var TMP = JSON.stringify(rankedMaskRateList).replace(/,/g, ', ').replace(/"/g, '');
            var ARR = JSON.parse(TMP);
            if (!_.isArray(ARR) || !ARR.length) {
                Toast('您输入的排名梯度格式不正确');
                return;
            } else {
                if (!_.every(ARR, o => _.isNumber(o))) {
                    Toast('您输入的排名梯度格式不正确');
                    return;
                }
            }
        } catch (e) {
            Toast('您输入的排名梯度格式不正确');
            return;
        }
        try {
            var tmp = JSON.stringify(gradientPriceList).replace(/，/g, ', ').replace(/"/g, '').replace(/(\w+):/g, '"$1":');
            var arr = JSON.parse(tmp);
            if (!_.isArray(arr) || !arr.length) {
                Toast('您输入的价格梯度格式不正确');
                return;
            } else {
                if (!_.every(arr, o => _.isObject(o) && _.isNumber(o.min) && _.isNumber(o.rate) && _.keys(o).length === 2)) {
                    Toast('您输入的价格梯度格式不正确');
                    return;
                }
            }
        } catch (e) {
            Toast('您输入的价格梯度格式不正确');
            return;
        }
        const param = {
            sizeWeightRate: sizeWeightRate * 1,
            insuanceRate: insuanceRate * 1,
            insuanceMountRate: insuanceMountRate * 1,
            insuanceBaseValue: insuanceBaseValue * 1,
            proxyChargeProfitRate: proxyChargeProfitRate * 1,
            gradientPriceList: arr,
            additionalDeliverTime: additionalDeliverTime * 1,
            noticeShipperStorageWeight: noticeShipperStorageWeight * 1,
            bondAmountWeightRate: bondAmountWeightRate * 1,
            rankedMaskFirstRankWeight: rankedMaskFirstRankWeight * 1,
            rankedMaskRateList: ARR,
        };
        POST(app.route.ROUTE_MODIFY_SETTING_INFO, param, this.getModifySettingInfoSuccess, false);
    },
    getModifySettingInfoSuccess (data) {
        if (data.success) {
            Toast('保存成功');
        } else {
            Toast(data.msg);
        }
    },
    getInitialState () {
        return {
            sizeWeightRate: '',
            insuanceRate: '',
            insuanceMountRate: '',
            insuanceBaseValue: '',
            proxyChargeProfitRate: '',
            gradientPriceList: '',
            additionalDeliverTime: '',
            noticeShipperStorageWeight: '',
            bondAmountWeightRate: '',
            rankedMaskFirstRankWeight: '',
            rankedMaskRateList: '',
        };
    },
    render () {
        // const {
        //     sizeWeightRate,
        //     insuanceRate,
        //     insuanceMountRate,
        //     insuanceBaseValue,
        //     proxyChargeProfitRate,
        //     gradientPriceList,
        //     additionalDeliverTime,
        //     noticeShipperStorageWeight,
        //     bondAmountWeightRate,
        //     rankedMaskFirstRankWeight,
        //     rankedMaskRateList,
        // } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.row}>
                    <Text style={styles.rowText}>保险费与保价的比</Text>
                    <TextInput placeholderTextColor='#aaaaaa'
                        placeholder='请填写比例'
                        keyboardType='numeric'
                        onChangeText={(text) => this.setState({ insuanceRate: text })}
                        maxLength={10}
                        underlineColorAndroid='transparent'
                        style={styles.rowTextInput}
                        />
                </View>
                <View style={styles.row}>
                    <Text style={styles.rowText}>保价与运费的比</Text>
                    <TextInput placeholderTextColor='#aaaaaa'
                        placeholder='请填写比例'
                        keyboardType='numeric'
                        onChangeText={(text) => this.setState({ insuanceMountRate: text })}
                        maxLength={10}
                        underlineColorAndroid='transparent'
                        style={styles.rowTextInput}
                        />
                </View>
                <View style={styles.row}>
                    <Text style={styles.rowText}>保险的保底值</Text>
                    <TextInput placeholderTextColor='#aaaaaa'
                        placeholder='请填写最低保险金额'
                        keyboardType='numeric'
                        onChangeText={(text) => this.setState({ insuanceBaseValue: text })}
                        maxLength={10}
                        underlineColorAndroid='transparent'
                        style={styles.rowTextInput}
                        />
                </View>
                <View style={styles.row}>
                    <Text style={styles.rowText}>方量与重量的比</Text>
                    <TextInput placeholderTextColor='#aaaaaa'
                        placeholder='请填写比例'
                        keyboardType='numeric'
                        onChangeText={(text) => this.setState({ sizeWeightRate: text })}
                        maxLength={10}
                        underlineColorAndroid='transparent'
                        style={styles.rowTextInput}
                        />
                </View>
                <View style={styles.row}>
                    <Text style={styles.rowText}>代收货款的手续费比例</Text>
                    <TextInput placeholderTextColor='#aaaaaa'
                        placeholder='请填写比例'
                        keyboardType='numeric'
                        onChangeText={(text) => this.setState({ proxyChargeProfitRate: text })}
                        maxLength={10}
                        underlineColorAndroid='transparent'
                        style={styles.rowTextInput}
                        />
                </View>
                <View style={styles.row}>
                    <Text style={styles.rowText}>代签额外增加时间/小时</Text>
                    <TextInput placeholderTextColor='#aaaaaa'
                        placeholder='请填写时间'
                        keyboardType='numeric'
                        onChangeText={(text) => this.setState({ additionalDeliverTime: text })}
                        maxLength={10}
                        underlineColorAndroid='transparent'
                        style={styles.rowTextInput}
                        />
                </View>
                <View style={styles.row}>
                    <Text style={styles.rowText}>通知物流公司的重量/吨</Text>
                    <TextInput placeholderTextColor='#aaaaaa'
                        placeholder='请填写重量'
                        keyboardType='numeric'
                        onChangeText={(text) => this.setState({ noticeShipperStorageWeight: text })}
                        maxLength={10}
                        underlineColorAndroid='transparent'
                        style={styles.rowTextInput}
                        />
                </View>
                <View style={styles.row}>
                    <Text style={styles.rowText}>每吨货需要的保证金</Text>
                    <TextInput placeholderTextColor='#aaaaaa'
                        placeholder='请填写金额'
                        keyboardType='numeric'
                        onChangeText={(text) => this.setState({ bondAmountWeightRate: text })}
                        maxLength={10}
                        underlineColorAndroid='transparent'
                        style={styles.rowTextInput}
                        />
                </View>
                <View style={styles.row}>
                    <Text style={styles.rowText}>屏蔽排名的基数/吨</Text>
                    <TextInput placeholderTextColor='#aaaaaa'
                        placeholder='请填写重量'
                        keyboardType='numeric'
                        onChangeText={(text) => this.setState({ rankedMaskFirstRankWeight: text })}
                        maxLength={10}
                        underlineColorAndroid='transparent'
                        style={styles.rowTextInput}
                        />
                </View>
                <View style={styles.row}>
                    <Text style={styles.rowText}>屏蔽物流公司排名的梯度</Text>
                    <TextInput placeholderTextColor='#aaaaaa'
                        placeholder='请填写梯度,例:[1,2,3]'
                        keyboardType='numeric'
                        onChangeText={(text) => this.setState({ rankedMaskRateList: text })}
                        maxLength={30}
                        underlineColorAndroid='transparent'
                        style={styles.rowTextInput}
                        />
                </View>
                <View style={styles.row}>
                    <Text style={styles.rowText}>阶梯价格的梯度</Text>
                </View>
                <View style={styles.rowBottom}>
                    <TextInput style={styles.content_input}
                        placeholderTextColor='#aaaaaa'
                        keyboardType='email-address'
                        maxLength={200}
                        placeholder='例:[{min:0,rate:3},{min:0.1,rate:2},{min:0.3,rate:1.5},...]'
                        onChangeText={(text) => this.setState({ gradientPriceList: text })}
                        multiline
                        underlineColorAndroid='transparent'
                        />
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#f4f4f4',
        paddingTop:9,
    },
    row:{
        marginTop:1,
        height:42,
        backgroundColor:'#FFFFFF',
        flexDirection:'row',
        justifyContent:'space-between',
        alignItems:'center',
    },
    rowText:{
        marginLeft:10,
        fontSize:12,
        color:'#333333',
    },
    rowTextInput:{
        fontSize:12,
        textAlign:'right',
        marginRight:10,
        padding:0,
        width:sr.w / 2,
    },
    rowBottom:{
        height:100,
        backgroundColor:'#FFFFFF',
        justifyContent:'center',
        marginTop:1,
    },
    rowTextInputBottom:{
        marginLeft:10,
        fontSize:14,
        textAlignVertical: 'top',
        padding:3,
    },
    content_input:{
        fontSize:14,
        height:100,
        backgroundColor:'#FFFFFF',
        textAlignVertical: 'top',
        marginLeft:10,
        marginRight:10,
    },
});

```

* PDShop/project/App/modules/person/PayPasswordSetting.js

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
            this.toNext(tmp);
        } else {
            Toast('支付密码有误');
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
                                {
                                    password.length >= (i + 1) ?
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

* PDShop/project/App/modules/person/Permission.js

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
                    renderRow={this.renderRow}
                    enableEmptySections />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
    },
    listStyle:{
        flexDirection:'row', // 改变ListView的主轴方向
        flexWrap:'wrap', // 换行
        width:sr.w,
    },
    item:{
        width:sr.w / 2,
        height:25,
        paddingLeft:10,
    },
    press:{
        flexDirection:'row',
        alignItems:'center',
        height:16,
    },
    imgChoose:{
        width:14,
        height:14,
    },
    itemFont:{
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

* PDShop/project/App/modules/person/PermissionSetting.js

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
        rightButton : { title: '确认', handler: () => { app.scene.confirm(); } },
    },
    getInitialState () {
        return {
            list:this.props.employeeInfo && this.props.employeeInfo.authority || this.props.authority || [],
        };
    },
    confirm () {
        const { employeeInfo, isCreate } = this.props;
        const { list } = this.state;
        if (_.intersection(app.personal.info.authority, [AH.AH_MODIFY_MEMBER]).length == 0) {
            Toast('你没有修改成员信息的权限');
            return;
        }
        if (isCreate) {
            app.pop();
            this.props.backAuthority(list);
        } else {
            const param = {
                userId: app.personal.info.userId,
                memberId : employeeInfo.id,
                name: employeeInfo.name,
                phone: employeeInfo.phone,
                authority: list,
            };
            POST(app.route.ROUTE_MODIFY_MEMBER, param, this.getModifyMemberSuccess, true);
        }
    },
    getModifyMemberSuccess (data) {
        if (data.success) {
            Toast('修改成功');
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
                        <Text style={styles.name}>姓名：{employeeInfo.name}</Text>
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
        height:40,
        padding:10,
        paddingTop:13,
    },
    jurisdiction:{
        flex:1,
        backgroundColor:'#FFFFFF',
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

* PDShop/project/App/modules/person/Recharge.js

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
// const PayPassword = require('../common/PayPassword');

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
    toPaySuccess (data) {
        const { amount } = this.state;
        if (data.success) {
            app.personal.setRemainAmount(app.personal.remainAmount + amount * CONSTANTS.RATE);
            this.props.setRemainAmount(app.personal.remainAmount);
            app.pop();
            app.showModal(
                <MessageBox
                    onConfirm={this.doConfirmDelete}
                    content={'成功充值' + amount + '元'}
                    width={sr.s(250)} />
                );
        } else {
            Toast(data.msg);
        }
    },
    doRecharge () {
        const { amount } = this.state;
        if (!app.utils.checkInt2PointNum(amount)) {
            Toast('请输入数字且小数点后最多2位');
            return;
        }
        const param = {
            userId: app.personal.info.userId,
            amount,
            thirdpartyAccount:'N1231242311',
        };
        POST(app.route.ROUTE_RECHARGE, param, this.toPaySuccess, true);
        // app.wxpay.doPay(amount,'充值',(data)=>{
        //         if (data.success) {
        //             app.personal.setRemainAmount(parseInt(app.personal.remainAmount)+parseInt(amount));
        //             app.pop();
        //             Toast('充值成功');
        //         }
        //
        //     },(data)=>{
        //         Toast(data.errStr);
        //     });
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
                        maxLength={CONSTANTS.AMOUNT_MAX_LENGTH}
                        defaultValue={amount}
                        onChangeText={(text) => this.setState({ amount: text })}
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

* PDShop/project/App/modules/person/SelectBankCardList.js

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
        leftButton: { handler: () => { app.scene.goBack(); } },
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
        const { selectedRowID, bankCard } = this.state;
        return (
            <View style={styles.container} >
                <View style={styles.line} />
                <TouchableOpacity onPress={this.toChoose.bind(null, obj, rowID)}>
                    <View style={styles.item}>
                        <Text style={styles.nameText}>
                            {bankCard}
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
                    renderSeparator={null}
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
    line:{
        height:1,
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

* PDShop/project/App/modules/person/SelectEmployees.js

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
        rightButton : { title: '添加员工', handler: () => { app.scene.addEmployee(); } },
    },
    getInitialState () {
        return {
            myself: {},
        };
    },
    addEmployee () {
        if (_.intersection(app.personal.info.authority, [AH.AH_CREATE_MEMBER]).length == 0) {
            Toast('你没有添加员工的权限');
            return;
        }
        app.push({
            component:addEmployee,
            passProps:{
                refresh:this.refresh,
            },
        });
    },
    refresh () {
        this.listView.refresh();
    },
    showPermissionSetting (obj) {
        app.push({
            component:PermissionSetting,
            passProps:{
                employeeInfo:obj,
                selfAuthority:app.personal.info.authority,
            },
        });
    },
    renderRow (obj, rowId) {
        if (obj.id == app.personal.info.userId) {
            return null;
        } else {
            return (
                app.personal.info.partment.chargeMan.id != obj.id &&
                <TouchableOpacity onPress={this.showPermissionSetting.bind(null, obj)}>
                    <View style={styles.item}>
                        <Text style={styles.itemNum}>{obj.name ? obj.name : ''}</Text>
                        <View style={styles.itemRight}>
                            <Text style={styles.itemNum}>{obj.authority.length}项</Text>
                            <Image resizeMode='contain' source={app.img.common_go} style={styles.itemImg} />
                        </View>
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
                    renderSeparator={null}
                    listName={'memberList'}
                    listUrl={app.route.ROUTE_GET_MEMBER_LIST}
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
        width:sr.w,
        marginTop:1,
        flexDirection:'row',
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent:'space-between',
    },
    itemNum:{
        fontWeight:'300',
        marginLeft:10,
        marginRight:5,
    },
    itemRight:{
        marginRight:10,
        flexDirection:'row',
        alignItems:'center',
    },
    itemImg:{
        width:15,
        height:15,
    },
});

```

* PDShop/project/App/modules/person/Software.js

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

* PDShop/project/App/modules/person/StatusSelection.js

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

module.exports = React.createClass({
    statics: {
        title: '搬运状态',
    },
    render () {
        return (
            <View style={styles.container}>
                <View style={styles.top}>
                    <Text style={styles.title}>状态选择</Text>
                </View>
                <View style={styles.chooseStatus}>
                    <TouchableOpacity style={styles.item}
                        onPress={this.props.setStatus.bind(null, true)}
                        onPressOut={this.props.doClose}>
                        <Text style={styles.text}>营业</Text>
                    </TouchableOpacity>
                    <TouchableOpacity style={styles.item}
                        onPress={this.props.setStatus.bind(null, false)}
                        onPressOut={this.props.doClose}>
                        <Text style={styles.text}>暂停营业</Text>
                    </TouchableOpacity>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container:{
        flex: 1,
        backgroundColor:'#dddddd',
    },
    top:{
        height:50,
        backgroundColor:'#ffffff',
        alignItems:'center',
        justifyContent:'center',
    },
    title:{
        fontSize:15,
        fontWeight:'600',
    },
    text:{
        fontSize:12,
        fontWeight:'300',
    },
    item:{
        marginTop:1,
        alignItems:'center',
        justifyContent:'center',
        height:50,
        backgroundColor:'#ffffff',
    },
});

```

* PDShop/project/App/modules/person/TransportQuotation.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    ListView,
} = ReactNative;

module.exports = React.createClass({
    statics: {
        title: '搬运行情',
    },
    getInitialState () {
        return {
            ds: new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 }),
            list: [1, 2],
        };
    },
    renderRow (obj, rowId) {
        return (
            <View style={styles.row}>
                <Text style={styles.name}>{'小虎搬运队'}</Text>
                <Text style={styles.peopleInfo}>人数:{'12'}人</Text>
                <Text style={styles.peopleInfo}>电话:{'18888888888'}</Text>
                <Text style={styles.price}>¥{'50.00'}/吨</Text>
            </View>
        );
    },
    render () {
        return (
            <View style={styles.container}>
                <ListView
                    dataSource={this.state.ds.cloneWithRows(this.state.list)}
                    renderRow={this.renderRow}
                />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    row:{
        backgroundColor:'#FFFFFF',
        height:75,
        marginTop:8,
        paddingLeft:10,
    },
    name:{
        fontSize:14,
        color:'#333333',
        flex:1,
        marginTop:8,
    },
    peopleInfo:{
        fontSize:12,
        color:'#888888',
        flex:1,

    },
    price:{
        position:'absolute',
        fontSize:13,
        right:15,
        top:8,
        color:'#e7222d',
    },
    btn:{
        width:75,
        height:22,
        backgroundColor:'#FFFFFF',
        borderRadius:4,
        borderWidth:0.5,
        borderColor:'#ee1822',
        position:'absolute',
        right:15,
        bottom:10,
    },
    btnFont:{
        color:'#ee1822',
        fontSize:12,
    },
});

```

* PDShop/project/App/modules/person/VerificationCode.js

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
        Keyboard.dismiss();
        app.push({
            component:PayPasswordSetting,
            passProps:{
                verifyCode,
            },
        });
    },
    doInput () {
        Toast('请先获取验证码');
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

* PDShop/project/App/modules/person/WithdrawCash.js

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
const VerificationCode = require('./VerificationCode');

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
        if (money == 0) {
            Toast('提现金额不能为0');
            return;
        }
        const param = {
            userId: app.personal.info.userId,
            amount:money,
            thirdpartyAccount:'N1231242311',
        };
        if (app.personal.info.isSetPaymentPassword == 0) {
            Toast('请先设置支付密码');
            app.push({
                component: VerificationCode,
            });
            return;
        }
        app.push({
            component: PayPassword,
            passProps: {
                param,
                routeName:app.route.ROUTE_WITHDRAW,
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
                        maxLength={CONSTANTS.AMOUNT_MAX_LENGTH}
                        placeholder='请输入提现金额'
                        defaultValue={money}
                        onChangeText={(text) => this.setState({ money: text })}
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

* PDShop/project/App/modules/person/addEmployee.js

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
    TouchableOpacity,
} = ReactNative;

 const { Button } = COMPONENTS;
 const PermissionSetting = require('./PermissionSetting');

 module.exports = React.createClass({
     statics: {
         title: '添加员工',
     },
     getInitialState () {
         return {
             authority:[],
             phone:'',
             name:'',
         };
     },
     getAuthority () {
         const { authority } = this.state;
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
     createMember () {
         const { name, phone, authority } = this.state;
         if (app.utils.checkPhone(phone)) {
             const param = {
                 userId: app.personal.info.userId,
                 name,
                 phone,
                 authority,
             };
             POST(app.route.ROUTE_CREATE_MEMBER, param, (data) => { data.success ? Toast('创建成功！') : Toast(data.msg); this.props.refresh(); app.pop(); }, true);
         } else {
             Toast('请填写正确的手机号');
         }
     },
     render () {
         const { authority } = this.state;
         return (
             <View style={styles.container}>
                 <View style={styles.top}>
                     <View style={styles.row}>
                         <Text style={styles.rowFont}>手机号码</Text>
                         <TextInput style={styles.rowRightFont}
                             placeholderColor='#aaaaaa'
                             maxLength={11}
                             underlineColorAndroid='transparent'
                             onChangeText={(text) => this.setState({ phone: text })}
                             placeholder='请输入手机号码' />
                     </View>
                     <View style={styles.row}>
                         <Text style={styles.rowFont}>姓名</Text>
                         <TextInput style={styles.rowRightFont}
                             underlineColorAndroid='transparent'
                             onChangeText={(text) => this.setState({ name: text })}
                             maxLength={CONSTANTS.NAME_MAX_LENGTH}
                             placeholderColor='#aaaaaa'
                             placeholder='请输入姓名' />
                     </View>
                 </View>
                 <TouchableOpacity style={styles.row} onPress={this.getAuthority}>
                     <Text style={styles.rowFont}>权限设置</Text>
                     <Text style={styles.rowRightFont}>{authority.length}项</Text>
                     <Image resizeMode='contain' style={styles.imgGo} source={app.img.common_go} />
                 </TouchableOpacity>
                 <Button onPress={this.createMember} style={styles.btnConfirm} textStyle={styles.btnFont}>确认添加</Button>
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
         marginBottom:10,
     },
     row:{
         backgroundColor:'#FFFFFF',
         height:42,
         flexDirection:'row',
         alignItems:'center',
         paddingRight:10,
     },
     rowFont:{
         marginLeft:10,
         fontWeight:'300',
         flex:1,
     },
     rowRightFont:{
         flex:1,
         fontSize:14,
         textAlign:'right',
     },
     imgGo:{
         width:15,
         height:15,
         marginLeft:10,
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

* PDShop/project/App/modules/person/index.js

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

const ParameterSetting = require('./ParameterSetting');
const CashBalance = require('./CashBalance');
const Bankcards = require('./BindBankcardList');
const VerificationCode = require('./VerificationCode');

const EditPersonInfo = require('./EditPersonInfo');
const About = require('./About');
const Feedback = require('./Feedback');
const NormalQuestions = require('./NormalQuestions');
const UpdatePage = require('../update/UpdatePage');
const Software = require('./Software');
const ModifyPassword = require('./ModifyPassword');
const SelectEmployees = require('./SelectEmployees');
const MyRemovalTeam = require('./MyRemovalTeam');

const { DImage } = COMPONENTS;

const CHILD_PAGES_TOP = [
    { strict:true, title:'余额', module: CashBalance, img:'', clickEnable:true },
    { strict:true, title:'银行卡', module: Bankcards, img:'', clickEnable:true },
    { strict:true, title:'支付密码设置', module: VerificationCode, img:'', clickEnable:true },
    { strict:true, title:'修改登录密码', module: ModifyPassword, img:'', clickEnable:true },
];
// const CHILD_PAGES_MIDDLE = [
//     { strict:true, title:'我的搬运队', module:MyRemovalTeam, img:'', clickEnable:true },
//     { strict:true, title:'员工设置', module: SelectEmployees, img:'', clickEnable:true },
// ];
const CHILD_PAGES_BOTTOM = [
    { strict:true, title:'软件许可协议', module: Software, img:'', clickEnable:true },
    { strict:true, title:'常见问题', module: NormalQuestions, img:'', clickEnable:true },
    { strict:true, title:'在线更新', module: UpdatePage, img:'', clickEnable:true },
    { strict:true, title:'关于软件', module: About, img:'', clickEnable:true },
    { strict:true, title:'意见反馈', module: Feedback, img:'', clickEnable:true },
];
const MenuItem = React.createClass({
    showChildPage () {
        const { module } = this.props.page;
        app.push({
            component: module,
        });
    },
    render () {
        const { title, clickEnable } = this.props.page;
        return (
            <TouchableOpacity
                activeOpacity={0.6}
                onPress={clickEnable ? this.showChildPage : null}
                style={styles.ItemBg}>
                <View style={styles.infoStyle}>
                    <Text style={styles.itemNameText}>{title}</Text>
                </View>
                {
                    clickEnable &&
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
        app.getCurrentRoute().title = '个人中心';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    getInitialState () {
        return {
            info:app.personal.info,
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
    judgePermission (employeeSetting) {
        if (_.intersection(app.personal.info.authority, employeeSetting).length != 0) {
            return true;
        } else {
            return false;
        }
    },
    componentWillMount () {
        const employeeSetting = [AH.AH_LOOK_MEMBER, AH.AH_CREATE_MEMBER, AH.AH_REMOVE_MEMBER, AH.AH_MODIFY_MEMBER];
        if (app.personal.info.partment && app.personal.info.partment.type == PT.SP_RECEIVE_PARTMENT) {
            this.page = {
                top:CHILD_PAGES_TOP,
                middle:[
                    { strict:true, title:app.personal.info.post || '普通员工', module:null, img:'', clickEnable:false },
                    { strict:true, title:'员工设置', module: SelectEmployees, img:'', clickEnable: this.judgePermission(employeeSetting) },

                ],
                bottom:CHILD_PAGES_BOTTOM,
            };
        } else if (app.personal.info.partment && app.personal.info.partment.type == PT.SP_WARE_HOUSE_PARTMENT) {
            this.page = {
                top:null,
                middle:[
                    { strict:true, title:app.personal.info.post || '普通员工', module:null, img:'', clickEnable:false },
                    { strict:true, title:'员工设置', module: SelectEmployees, img:'', clickEnable: this.judgePermission(employeeSetting) },
                ],
                bottom:CHILD_PAGES_BOTTOM,
            };
        } else if (app.personal.info.partment && app.personal.info.partment.type == PT.SP_CARRY_PARTMENT) {
            this.page = {
                top:CHILD_PAGES_TOP,
                middle:[
                    { strict:true, title:'我的搬运队', module:MyRemovalTeam, img:'', clickEnable: this.judgePermission([AH.AH_MODIFY_OWN_PARTMENT]) },
                    { strict:true, title:'员工设置', module: SelectEmployees, img:'', clickEnable: this.judgePermission(employeeSetting) },
                ],
                bottom:CHILD_PAGES_BOTTOM,
            };
        } else if (app.personal.info.partment && app.personal.info.partment.type == PT.SP_SECURITY_CHECK_PARTMENT) {
            this.page = {
                top:null,
                middle:[
                    { strict:true, title:app.personal.info.post || '普通员工', module:null, img:'', clickEnable:false },
                    { strict:true, title:'员工设置', module: SelectEmployees, img:'', clickEnable: this.judgePermission(employeeSetting) },
                ],
                bottom:CHILD_PAGES_BOTTOM,
            };
        } else if (!app.personal.info.partment) {
            this.page = {
                top:[
                    { strict:true, title:app.personal.info.post || '普通员工', module:null, img:'', clickEnable:false },
                    { strict:true, title:'参数设置', module: ParameterSetting, img:'', clickEnable: true },
                ],
                middle:CHILD_PAGES_TOP,
                bottom:CHILD_PAGES_BOTTOM,
            };
        } else {
            this.page = {
                top:CHILD_PAGES_TOP,
                middle:[
                    { strict:true, title:app.personal.info.post || '普通员工', module:null, img:'', clickEnable:false },
                    { strict:true, title:'员工设置', module: SelectEmployees, img:'', clickEnable: this.judgePermission(employeeSetting) },
                ],
                bottom:CHILD_PAGES_BOTTOM,
            };
        }
    },
    render () {
        const { info } = this.state;
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
                            source={app.img.personal_female}
                            style={styles.careStyle} />
                    </TouchableOpacity>
                    <Text style={styles.name}>{info.name ? info.name : '张三'}</Text>
                </View>
                <View>
                    {this.page.top && <View style={styles.lineViewTop} />}
                    {
                        this.page.top && this.page.top.map((item, i) => {
                            if (!app.personal.info.phone && item.strict) {
                                return null;
                            }
                            if (item.title == '余额') {
                                if (_.intersection(app.personal.info.authority, [AH.AH_LOOK_AMOUNT, AH.AH_RECHARGE, AH.AH_WITHDRAW]).length != 0) {
                                    return (<MenuItem page={item} key={i} />);
                                } else {
                                    return null;
                                }
                            } else if (item.title == '支付密码设置') {
                                if (_.intersection(app.personal.info.authority, [AH.AH_WITHDRAW]).length != 0) {
                                    return (<MenuItem page={item} key={i} />);
                                } else {
                                    return null;
                                }
                            } else {
                                return (<MenuItem page={item} key={i} />);
                            }
                        })
                    }
                    {this.page.middle && <View style={styles.lineViewTop} />}
                    {
                        this.page.middle && this.page.middle.map((item, i) => {
                            if (!app.personal.info.phone && item.strict) {
                                return null;
                            }
                            return (
                                !item.hidden &&
                                <MenuItem page={item} key={i} />
                            );
                        })
                    }
                    {this.page.bottom && <View style={styles.lineViewTop} />}
                    {
                        this.page.bottom && this.page.bottom.map((item, i) => {
                            if (!app.personal.info.phone && item.strict) {
                                return null;
                            }
                            return (
                                !item.hidden &&
                                <MenuItem page={item} key={i} />
                            );
                        })
                    }
                    <View style={styles.line} />
                </View>
            </ScrollView>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex:1,
        marginBottom:0,
        backgroundColor:'white',
    },
    headImgBg:{
        height: 160,
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
    headStyle: {
        width: 87,
        height: 87,
        marginTop: 15,
        marginBottom:5,
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
        width: 20,
        height: 20,
        marginLeft:60,
        marginTop:-25,
    },
    itemNameText: {
        fontSize: 15,
        color: '#444444',
        marginLeft: 8,
    },
    name: {
        marginTop: 15,
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
});

```

* PDShop/project/App/modules/receiveDepartment/OrderDetail.js

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

const { Button, DImage } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '货单详情',
    },
    onWillFocus () {
        app.getCurrentRoute().title = '货单详情';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    getInitialState () {
        return {
            choose:false,
            canPay:false || this.props.canPay,
            actionSheetVisible: false,
        };
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
        const param = {
            userId: app.personal.info.userId,
            orderId: this.props.data.id,
        };
        POST(app.route.ROUTE_PAY_FOR_ORDER_WHEN_SEND, param, this.getPaySuccess, true);
    },
    getPaySuccess (data) {
        if (data.success) {
            this.setState({ actionSheetVisible:false });
            Toast('支付成功');
        } else {
            Toast('支付失败');
        }
    },
    doLookTransport () {

    },
    toPayMode (payMode) {
        if (payMode == 0) {
            return '现金付款';
        } else if (payMode == 1) {
            return '货到付款';
        } else {
            return '混合付款';
        }
    },
    render () {
        const { data } = this.props;
        const { stateList } = data;
        return (
            <View style={styles.container}>
                <View style={styles.containerReceive}>
                    <Text style={styles.textAddress}> 收货地址：{data.endPoint} </Text>
                    <Text style={styles.textName}> 收货人：{data.receiverName ? data.receiverPhone : '' + data.receiverPhone} </Text>
                </View>
                <View style={styles.containerMarket}>
                    <Image style={styles.marketLogo}
                        source={app.img.order_logo}
                        resizeMode='stretch'
                        />
                    <Text style={styles.textMarket}>{data.shop ? data.shop.name : data.agent ? data.agent.name : ''}</Text>
                    <Text style={styles.orderState}>{stateList ? OS.MAP[stateList[0].state] : ''}</Text>
                </View>
                <View style={styles.containerDetails}>
                    <DImage style={styles.imageGoods}
                        resizeMode='stretch'
                        defaultSource={app.img.common_default}
                        source={{ uri: data.photo }}
                        />
                    <View style={styles.goodsDetails}>
                        <View style={styles.goodsDetailsTop}>
                            <Text style={styles.detailsLeft}
                                numberOfLines={1}
                                ellipsizeMode='tail'>{data.shop ? data.shop.address : data.agent ? data.agent.address : ''}</Text>
                            <Text>-</Text>
                            <Text style={styles.detailsLeft}
                                numberOfLines={1}
                                ellipsizeMode='tail'>{data.endPoint}</Text>
                            <View style={styles.detailsTopRight}>
                                <Text style={styles.moneyLeft}>运费：</Text>
                                <Text style={styles.moneyRight}>¥{data.realFee}</Text>
                            </View>
                        </View>
                        <Text style={styles.textDetailsItem}>重量：{data.weight}吨</Text>
                        <Text style={styles.textDetailsItem}>方量：{data.size}m³</Text>
                        <Text style={styles.textDetailsItem}>件数：{data.totalNumbers}件</Text>
                        <Text style={styles.textDetailsItem}>下单时间：{data.placeOrderTime}</Text>
                    </View>
                </View>
                <View style={styles.containerWhite} />
                <View style={styles.containerOrderNumber}>
                    <Text style={styles.textOrderNumber}>订单号：{data.id}</Text>
                    <Text style={styles.textDate}>创建时间：{data.createTime}</Text>
                </View>
                <View style={styles.containerInfo}>
                    <Text style={styles.textInfo}>运费：¥{data.realFee}</Text>
                    <Text style={styles.textInfo}>支付方式：{this.toPayMode(data.payMode)}</Text>
                    <Text style={styles.textInfo}>保险费：¥{data.needPayInsuanceFee}</Text>
                    <Text style={styles.textTotal}>总计：¥{data.realFee + data.needPayInsuanceFee}</Text>
                </View>
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
                { this.state.canPay &&
                    <View style={styles.bottomView}>
                        <View style={styles.bottomInfo}>
                            <View style={styles.infoLeft}>
                                <Text style={styles.moneyLeft}>总计:</Text>
                                <Text style={styles.moneyRight}>¥{data.realFee}</Text>
                            </View>
                            <Button onPress={this.doShowActionSheet} style={styles.btnPayNow} textStyle={styles.btnPayNowText}>
                                立即支付
                            </Button>
                        </View>
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
        height:50,
        backgroundColor:'#ffffff',
        justifyContent: 'center',
    },
    textAddress:{
        marginLeft:10,
        fontSize:14,
        marginBottom:3,
        color:'#333333',
    },
    textName:{
        marginLeft:10,
        fontSize:12,
        marginTop:3,
        color:'#333333',
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
        backgroundColor:'#f8f8f8',
        flexDirection: 'row',
    },
    containerWhite:{
        height:8,
        backgroundColor:'#ffffff',
    },
    imageGoods:{
        height:90,
        width:90,
        marginLeft:12,
        marginTop:11,
    },
    goodsDetails:{
        marginLeft:8,
        height:95,
        marginTop:5,
        paddingBottom:8,
        justifyContent: 'center',
    },
    goodsDetailsTop:{
        width:sr.w - 99,
        flexDirection: 'row',
        marginTop:8,
        justifyContent: 'space-between',
    },
    detailsTopRight:{
        flexDirection: 'row',
        alignItems: 'center',
        marginRight:10,
    },
    detailsLeft:{
        fontSize:14,
        color:'#333333',
        width:60,
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
        fontSize:12,
    },
    containerInfo:{
        height:90,
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingLeft:10,
    },
    textInfo:{
        color:'#888888',
        fontSize:12,
        marginTop:5,
    },
    textTotal:{
        marginTop:5,
        fontSize:12,
        color:'#333333',
        fontWeight:'400',
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
        justifyContent:'center',
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
});

```

* PDShop/project/App/modules/receiveDepartment/OrderList.js

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

const { PageList, DImage } = COMPONENTS;
const SendOrderDetail = require('./OrderDetail');

module.exports = React.createClass({
    doLookDetail (obj, index) {
        const param = {
            userId: app.personal.info.userId,
            orderId: obj.id,
        };
        POST(app.route.ROUTE_GET_ORDER_DETAIL, param, this.getOrderDetailSuccess.bind(null, index), true);
    },
    getOrderDetailSuccess (index, data) {
        const { success, context, msg } = data;
        if (success) {
            app.push({
                component: SendOrderDetail,
                passProps: {
                    data: context,
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
    renderRow (obj, sectionID, rowID) {
        const { stateList } = obj;
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
                <View style={styles.line} />
                <TouchableOpacity style={styles.rowMiddleContainer} onPress={this.doLookDetail.bind(null, obj, rowID)}>
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
                                <Text style={styles.pointName}>→</Text>
                                <Text style={styles.pointName}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{obj.endPoint}</Text>
                            </View>
                            <Text style={styles.transformFee}>{'运费：'}<Text style={styles.money}>￥{obj.needPayTransportFee}</Text></Text>
                        </View>
                        <Text style={styles.commonInfo}>重量：{obj.weight}吨</Text>
                        <Text style={styles.commonInfo}>方量：{obj.size}m³</Text>
                        <Text style={styles.commonInfo}>件数：{obj.totalNumbers}件</Text>
                        <Text style={styles.commonInfo}>仓库号：{obj.warehouse}件</Text>
                        <Text style={styles.commonInfo}>下单时间：{obj.createTime}</Text>
                    </View>
                </TouchableOpacity>
            </View>
        );
    },
    render () {
        const { index, tabIndex, type, data } = this.props;
        return (
            <View style={index === tabIndex ? styles.container : { left:-sr.tw, top:0, position:'absolute' }}>
                <View style={styles.separator} />
                {
                    !!data &&
                    <PageList
                        ref={(ref) => { this.listView = ref; }}
                        autoLoad={false}
                        renderRow={this.renderRow}
                        listParam={{ userId: app.personal.info.userId, type: type }}
                        listName={type + '.list'}
                        list={data.list}
                        listUrl={app.route.ROUTE_GET_ORDERS_BY_RECEIVE_PARTMENT}
                        refreshEnable />
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
        height: 5,
    },
    line:{
        height:1,
        width:sr.w,
        backgroundColor:'#f8f8f8',
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
    rowMiddleContainer: {
        flexDirection: 'row',
        alignItems: 'center',
        paddingVertical:5,
        backgroundColor:'#ffffff',
    },
    middleImage: {
        width: 90,
        height:90,
        borderRadius: 10,
        marginLeft: 10,
    },
    middleRight: {
        paddingLeft: 10,
        width:sr.w - 100,
    },
    money: {
        fontSize: 14,
        color:'#f51b1a',
    },
    commonInfo: {
        fontSize: 13,
        color:'#888888',
    },
    pointName: {
        fontSize: 16,
        maxWidth: 70,
    },
    transformFee: {
        flex:1,
        fontSize: 14,
        color: 'gray',
        textAlign:'right',
        marginRight:5,
    },
    pointContainer: {
        flexDirection: 'row',
    },
    placeContainer: {
        flex:1,
        flexDirection: 'row',
    },
});

```

* PDShop/project/App/modules/receiveDepartment/Orders.js

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

module.exports = React.createClass({
    statics: {
        title: '货单',
    },
    onWillFocus () {
        app.getCurrentRoute().title = '货单';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    getInitialState () {
        return {
            tabIndex: 0,
            context:{},
        };
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    componentWillMount () {
        this.getOrders();
    },
    getOrders () {
        const param = {
            userId: app.personal.info.userId,
            pageNo:0,
            pageSize:10,
        };
        POST(app.route.ROUTE_GET_ORDERS_BY_RECEIVE_PARTMENT, param, this.getOrdersSuccess, false);
    },
    getOrdersSuccess (data) {
        if (data.success) {
            this.setState({ context:data.context });
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const menus = [{ name:'待入库', type:'tostore' }, { name:'已入库', type:'stored' }];
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
                        <OrderList key={i} index={i} type={item.type} tabIndex={tabIndex} data={context && context[item.type]} />
                    ))
                }
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#ffffff',
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

* PDShop/project/App/modules/receiveDepartment/ScanResult.js

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
} = ReactNative;

const { Button } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '扫描结果',
    },
    onWillFocus () {
        app.getCurrentRoute().title = '扫描结果';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    getInitialState () {
        return {
            context: {},
            totalNumbers:0,
            size:0,
            weight:0,
        };
    },
    componentWillMount () {
        const orderId = this.props.orderId;
        this.getOrderDetailByReceivePartment(orderId);
    },
    getOrderDetailByReceivePartment (orderId) {
        const param = {
            userId : app.personal.info.userId,
            orderId,
        };
        POST(app.route.ROUTE_GET_ORDER_DETAIL, param, this.getOrderDetailSuccess, false);
    },
    getOrderDetailSuccess (data) {
        if (data.success) {
            this.setState({ context:data.context });
        } else {
            Toast(data.msg);
        }
    },
    placeTransferOrder () {
        const { context, totalNumbers, size, weight } = this.state;
        if (!app.utils.checkNum(weight)) {
            Toast('请填写货物重量');
            return;
        }
        if (!app.utils.checkNum(size)) {
            Toast('请填写货物方量');
            return;
        }
        if (!app.utils.checkIntNum(totalNumbers)) {
            Toast('请填写货物件数');
            return;
        }
        const param = {
            userId : app.personal.info.userId,
            orderId: context.id,
            totalNumbers,
            size,
            weight,
        };
        POST(app.route.ROUTE_PLACE_TRANSFER_ORDER, param, this.placeTransferOrderSuccess, false);
    },
    placeTransferOrderSuccess (data) {
        if (data.success) {
            Toast('下单成功！');
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { context, size, weight, endPoint } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.topContainer}>
                    <Image
                        resizeMode='stretch'
                        defaultSource={app.img.receive_order_default}
                        source={{ uri: context.photo }}
                        style={styles.topLeft} />
                    <View style={styles.topRight}>
                        <Text numberOfLines={1} style={styles.topRightText}>始发地址：{context.shop && context.shop.address}</Text>
                        <Text style={styles.topRightText}>重量：{context.weight}吨</Text>
                        <Text style={styles.topRightText}>方量：{context.size}m³</Text>
                        <Text style={styles.topRightText}>件数：{context.totalNumbers}件</Text>
                        <Text style={styles.topRightText}>收货地址：{context.endPoint}</Text>
                        {
                            context.warehouse &&
                            <Text style={styles.topRightText}>仓库：{context.warehouse}</Text>
                        }

                    </View>
                </View>
                <View>
                    <Text style={styles.attachInfoTitle}>附件信息</Text>
                    <View style={styles.attachItem}>
                        <Text style={styles.attachItemLeft}>重量</Text>
                        <View style={styles.textRight}>
                            <TextInput style={styles.attachItemRight}
                                underlineColorAndroid='transparent'
                                placeholderTextColor='#888888'
                                keyboardType='numeric'
                                placeholder='请输入货物重量'
                                maxLength={5}
                                defaultValue={context.weight + ''}
                                onChangeText={(text) => this.setState({ weight: text })}
                                />
                            <Text style={styles.attachItemRightUnit}>吨</Text>
                        </View>
                    </View>
                    <View style={styles.attachItem}>
                        <Text style={styles.attachItemLeft}>方量</Text>
                        <View style={styles.textRight}>
                            <TextInput style={styles.attachItemRight}
                                underlineColorAndroid='transparent'
                                keyboardType='numeric'
                                placeholderTextColor='#888888'
                                maxLength={5}
                                placeholder={'请输入货物方量'}
                                defaultValue={context.size + ''}
                                onChangeText={(text) => this.setState({ size: text })}
                            />
                            <Text style={styles.attachItemRightUnit}>m³</Text>
                        </View>
                    </View>
                    <View style={styles.attachItem}>
                        <Text style={styles.attachItemLeft}>件数</Text>
                        <View style={styles.textRight}>
                            <TextInput style={styles.attachItemRight}
                                underlineColorAndroid='transparent'
                                keyboardType='numeric'
                                placeholderTextColor='#888888'
                                maxLength={5}
                                placeholder='请输入货物件数'
                                defaultValue={context.totalNumbers + ''}
                                onChangeText={(text) => this.setState({ totalNumbers: text })}
                            />
                            <Text style={styles.attachItemRightUnit}>件</Text>
                        </View>
                    </View>
                    <View style={styles.attachItem}>
                        <Text style={styles.attachItemLeft}>操作人</Text>
                        <Text style={styles.attachItemRightUnit}>{app.personal.info.name}</Text>
                    </View>
                </View>
                <Button onPress={this.placeTransferOrder} style={styles.btnConfirmModify} textStyle={styles.btnConfirmModifyText} >
                    下中转单
                </Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor:'#f4f4f4',
        marginBottom:10,
    },
    topContainer: {
        flexDirection: 'row',
        alignItems: 'center',
        backgroundColor:'white',
        paddingTop:8,
        paddingBottom:8,
    },
    topLeft: {
        width: 90,
        height:90,
        borderRadius: 10,
        marginLeft: 10,
    },
    topRight: {
        paddingLeft: 10,
        width:sr.w - 100,
    },
    topRightText: {
        fontSize: 13,
        color:'#888888',
        maxWidth:300,
    },
    btnConfirmModify:{
        marginTop:115,
        height:45,
        width:300,
        marginLeft:(sr.w - 300) / 2,
        borderRadius:2,
    },
    btnConfirmModifyText:{
        color:'white',
        fontSize:16,
        fontWeight:'300',
    },
    noDataImg:{
        height:120,
        width:126,
        marginLeft:(sr.w - 126) / 2,
        marginTop:65,
    },
    onDataText:{
        fontSize:17,
        color:'#888888',
        marginTop:15,
        textAlign:'center',
    },
    attachInfoTitle:{
        fontSize:16,
        width:sr.w,
        paddingLeft:8,
        marginTop:8,
        marginBottom:3,
    },
    attachItem:{
        paddingLeft:10,
        width:sr.w,
        backgroundColor:'white',
        flexDirection:'row',
        justifyContent:'space-between',
        borderBottomWidth:1,
        borderBottomColor:'#f4f4f4',
        alignItems:'center',
        height:40,
    },
    attachItemLeft:{
        fontSize:14,
        textAlign:'center',
        color:'#333333',
    },
    attachItemRight:{
        width:sr.w / 2,
        padding:0,
        fontSize:14,
        textAlign:'right',
    },
    attachItemRightUnit:{
        marginRight:10,
        marginLeft:8,
        textAlign:'right',
        fontSize:14,
        color:'#333333',
    },
    textRight:{
        flexDirection:'row',
        alignItems:'center',
    },
});

```

* PDShop/project/App/modules/receiveDepartment/UploadPhoto.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
    ScrollView,
    Image,
    RefreshControl,
} = ReactNative;

const { Button } = COMPONENTS;
const Camera = require('@remobile/react-native-camera');
const ScanBarCode = require('../common/ScanBarCode');
const ScanResult = require('./ScanResult');

module.exports = React.createClass({
    statics: {
        title: '拍照',
        rightButton: { title: '扫描', handler: () => { app.scene.scanBarCode(); } },
    },
    onWillFocus () {
        app.getCurrentRoute().title = '拍照';
        app.getCurrentRoute().rightButton = { title: '扫描', handler: () => { app.scene.scanBarCode(); } };
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
        this.getNeedUploadPhotoOrder();
    },
    scanBarCode () {
        app.showModal(
            <ScanBarCode onBarCodeRead={this.onBarCodeRead} />
        );
    },
    onBarCodeRead (data) {
        if (!data || data.type != CONSTANTS.QRCODE_TYPE_ORDER) { // type==1代表货单二维码
            Toast('无效的二维码');
        } else {
            app.push({
                component:ScanResult,
                passProps:{
                    orderId:data.id,
                },
            });
        }
    },
    getInitialState () {
        return {
            context: null,
        };
    },
    componentWillMount () {
        this.getNeedUploadPhotoOrder();
    },
    getNeedUploadPhotoOrder () {
        const param = {
            userId: app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_LASTEST_ORDER_BY_RECIEVE_PARTMENT, param, this.getNeedUploadPhotoOrderSuccess, false);
    },
    getNeedUploadPhotoOrderSuccess (data) {
        if (data.success) {
            this.setState({ context:data.context });
        } else {
            this.setState({ context:null });
        }
    },
    setPhotoForOrder (photo) {
        const param = {
            userId: app.personal.info.userId,
            orderId:this.state.context.id,
            photo,
        };

        POST(app.route.ROUTE_SET_PHOTO_FOR_ORIGIN_ORDER, param, this.setPhotoForOrderSuccess.bind(null, photo), false);
    },
    setPhotoForOrderSuccess (photo) {
        Toast('照片更新成功');
        let { context } = this.state;
        context.photo = photo;
        this.setState({ context });
    },
    doTakePhoto () {
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
        }, (err) => {
            console.log('操作失败========', err);
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
            this.setPhotoForOrder(context.url);
            this.getNeedUploadPhotoOrder();
        } else {
            Toast('上传失败');
        }
        this.uploadOn = false;
    },
    uploadErrorCallback () {
        this.uploadOn = false;
        this.setState({ photo: app.personal.info.head });
    },
    onScroll () {
        this.getNeedUploadPhotoOrder();
    },
    render () {
        const { context } = this.state;
        const { photo, shop, totalNumbers, size, weight, endPoint } = context || {};
        return (
            <ScrollView
                refreshControl={
                    <RefreshControl
                        refreshing={false}
                        onRefresh={this.onScroll}
                        title='正在刷新...' />
                }
                style={styles.container}>
                {
                    !!context &&
                    <View>
                        <View style={styles.topContainer}>
                            <Image
                                resizeMode='stretch'
                                source={photo ? { uri: photo } : app.img.receive_order_default}
                                style={styles.topLeft} />
                            <View style={styles.topRight}>
                                <Text numberOfLines={1} style={styles.topRightText}>始发地址：{shop && shop.address}</Text>
                                <Text style={styles.topRightText}>重量：{weight}吨</Text>
                                <Text style={styles.topRightText}>方量：{size}m³</Text>
                                <Text style={styles.topRightText}>件数：{totalNumbers}件</Text>
                                <Text style={styles.topRightText}>收货地址：{endPoint}</Text>
                            </View>
                        </View>
                        <Button onPress={this.doTakePhoto} style={styles.btnTakePhoto} textStyle={styles.btnTakePhotoText} >
                            拍照
                        </Button>
                    </View>
                    ||
                    <View style={styles.noDataContainer}>
                        <Image
                            resizeMode='stretch'
                            source={app.img.receive_no_orders}
                            style={styles.noDataImg} />
                        <Text style={styles.onDataText}>暂无货单</Text>
                        <View style={styles.btnTakePhotoGray}>
                            <View style={styles.imgContainer}>
                                <Image style={styles.photoImage}
                                    source={app.img.receive_camera} />
                                <Text style={styles.imgText}>拍照</Text>
                            </View>
                        </View>
                    </View>
                }
            </ScrollView>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f4f4f4',
    },
    topContainer: {
        flexDirection: 'row',
        alignItems: 'center',
        paddingVertical:11,
        backgroundColor:'#f8f8f8',
    },
    topLeft: {
        width: 80,
        height:80,
        borderRadius: 10,
        marginLeft: 10,
    },
    topRight: {
        paddingLeft: 10,
        width:sr.w - 100,
    },
    topRightText: {
        paddingVertical:2,
        fontSize: 13,
        color:'#888888',
        maxWidth:300,
    },
    btnTakePhoto:{
        marginTop:38,
        height:35,
        width:150,
        borderRadius:4,
        backgroundColor:'#c61a29',
        justifyContent:'center',
        alignItems:'center',
        alignSelf:'center',
    },
    btnTakePhotoGray:{
        marginTop:38,
        height:35,
        width:150,
        borderRadius:4,
        backgroundColor:'#aaaaaa',
        justifyContent:'center',
        alignItems:'center',
        alignSelf:'center',
    },
    btnTakePhotoText:{
        color:'white',
        fontSize:16,
        fontWeight:'300',
    },
    noDataContainer:{
        justifyContent:'center',
        alignItems:'center',
    },
    noDataImg:{
        height:80,
        width:80.5,
        marginTop:50,
    },
    onDataText:{
        fontSize:21,
        fontWeight:'400',
        color:'#aaaaaa',
        marginTop:15,
    },
    photoImage:{
        height:14,
        width:15.5,
        marginRight:3,
    },
    imgContainer:{
        flexDirection:'row',
        justifyContent:'center',
        alignItems:'center',
    },
    imgText:{
        fontSize:16,
        fontWeight:'400',
        color:'#FFFFFF',
        marginLeft:3,
    },
});

```

* PDShop/project/App/modules/receiveDepartment/index.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
} = ReactNative;

import TabNavigator from 'react-native-tab-navigator';

const UploadPhoto = require('./UploadPhoto');
const Orders = require('./Orders');
const Person = require('../person');

const INIT_ROUTE_INDEX = 0;
const ROUTE_STACK = [
    { index: 0, component: UploadPhoto },
    { index: 1, component: Orders },
    { index: 2, component: Person },
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
            { index: 0, title: '拍照', icon:app.img.receive_take_picture, selected: app.img.receive_take_picture_press },
            { index: 1, title: '货单', icon: app.img.receive_order, selected: app.img.receive_order_press },
            { index: 2, title: '我的', icon: app.img.home_mine, selected: app.img.home_mine_press },
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

* PDShop/project/App/modules/securityCheckDepartment/CheckPass.js

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

const ScanResult = require('./ScanResult');
const ScanBarCode = require('../common/ScanBarCode');

module.exports = React.createClass({
    statics: {
        title: '扫描安检',
    },
    onWillFocus () {
        app.getCurrentRoute().title = '扫描安检';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    scanBarCode () {
        app.showModal(
            <ScanBarCode onBarCodeRead={this.onBarCodeRead} />
        );
    },
    onBarCodeRead (data) {
        if (!data || data.type != CONSTANTS.QRCODE_TYPE_DRIVER) { // type==2代表司机二维码
            Toast('无效的二维码');
        } else {
            app.push({
                component:ScanResult,
                passProps:{
                    driverId:data.id,
                },
            });
        }
    },
    getInitialState () {
        return {
            context: {},
        };
    },
    render () {
        return (
            <View style={styles.container}>
                <View>
                    <Image source={app.img.security_scan_check_pass}
                        style={styles.imgScanCode}
                        resizeMode='contain' />
                    <TouchableOpacity style={styles.btnScan} onPress={this.scanBarCode}>
                        <Image source={app.img.storage_scan}
                            style={styles.imgscan}
                            resizeMode='contain' />
                        <Text style={styles.scanFont}>扫一扫</Text>
                    </TouchableOpacity>
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
    imgScanCode:{
        width:sr.w - 205,
        marginLeft:205 / 2,
        height:sr.h - 475,
        marginTop:sr.h - 620,
    },
    btnScan:{
        marginTop:90,
        flexDirection:'row',
        width:sr.w - 225,
        marginLeft:225 / 2,
        height:35,
        backgroundColor:'#c61a29',
        alignItems:'center',
        justifyContent:'center',
        borderRadius:4,
    },
    imgscan:{
        width:14,
        height:14,
    },
    scanFont:{
        color:'#ffffff',
        marginLeft:5,
        fontWeight:'500',
    },
});

```

* PDShop/project/App/modules/securityCheckDepartment/ScanResult.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

const { DImage } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '扫描结果',
    },
    onWillFocus () {
        app.getCurrentRoute().title = '扫描结果';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    getInitialState () {
        return {
            context:null,
            tip:'',
        };
    },
    componentWillMount () {
        this.checkTruckPass();
    },
    checkTruckPass () {
        const param = {
            userId: app.personal.info.userId,
            driverId:this.props.driverId,
        };
        POST(app.route.ROUTE_CHECK_TRUCK_PASS, param, this.checkTruckPassSuccess, true);
    },
    checkTruckPassSuccess (data) {
        if (data.success) {
            this.setState({ context:data.context });
        } else {
            this.setState({ context:null, tip:data.msg });
        }
    },
    render () {
        const { context, tip } = this.state;
        return (
            <View style={styles.container}>
                {
                    !context ?
                        <Text style={styles.tipCenter}>{tip}</Text>
                    :
                        <View style={styles.driverInfo}>
                            <View style={styles.top}>
                                <Text style={styles.topLeft}>驾驶信息</Text>
                                <Text style={styles.topRight}>{context && context.isEnter ? '入库' : '出库'}</Text>
                            </View>
                            <DImage
                                resizeMode='cover'
                                defaultSource={app.img.personal_default_head}
                                source={{ uri: context.driver.head }}
                                style={styles.headStyle} />
                            <Text style={styles.textCenterName}>{context && context.driver && context.driver.name}</Text>
                            <Text style={styles.textCenter}>驾驶证：{context && context.driver && context.driver.licenseNo}</Text>
                            <Text style={styles.textCenter}>车牌{context && context.plateNo}</Text>
                        </View>

                }

            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    driverInfo: {
        height:220,
        marginTop:10,
        backgroundColor:'white',
    },
    top: {
        flexDirection:'row',
        justifyContent:'space-between',
        padding:10,
    },
    topLeft: {
        fontSize:16,
        color:'#333333',
    },
    topRight: {
        fontSize:16,
        color:'#c81622',
    },
    headStyle: {
        width: 87,
        height: 87,
        borderRadius: 43.5,
        alignSelf:'center',
    },
    textCenterName: {
        alignSelf:'center',
        fontSize:16,
        paddingVertical:5,
        color:'#333333',
    },
    tipCenter: {
        alignSelf:'center',
        fontSize:16,
        color:'#888888',
    },
    textCenter: {
        alignSelf:'center',
        fontSize:16,
        paddingVertical:5,
        color:'#888888',
    },
});

```

* PDShop/project/App/modules/securityCheckDepartment/index.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
} = ReactNative;

import TabNavigator from 'react-native-tab-navigator';

const CheckPass = require('./CheckPass');
const Person = require('../person');

const INIT_ROUTE_INDEX = 0;
const ROUTE_STACK = [
    { index: 0, component: CheckPass },
    { index: 1, component: Person },
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
            { index: 0, title: '安检', icon:app.img.home_pre_order, selected: app.img.home_pre_order_press },
            { index: 1, title: '我的', icon: app.img.home_mine, selected: app.img.home_mine_press },
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

* PDShop/project/App/modules/splash/index.js

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
const Chairman = require('../chairman');
const WarehouseDepartment = require('../warehouseDepartment');
const ReceiveDepartment = require('../receiveDepartment');
const MovingDepartment = require('../movingDepartment');
const SecurityCheckDepartment = require('../securityCheckDepartment');
const ENTRY_HOME = [ ReceiveDepartment, WarehouseDepartment, MovingDepartment, SecurityCheckDepartment, Chairman ];
module.exports = React.createClass({
    mixins: [ TimerMixin ],
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
            component: ENTRY_HOME[app.personal.info.partment ? app.personal.info.partment.type : 4],
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

* PDShop/project/App/modules/test/Echarts.js

```js
'use strict';

var React = require('react');
var ReactNative = require('react-native');
var {
    Image,
    StyleSheet,
    Text,
    View,
    TouchableOpacity,
    TouchableHighlight,
    ScrollView,
    WebView,
} = ReactNative;

var { Button } = COMPONENTS;
var Circle = require('../common/Circle.js');
var Line = require('../common/Line.js');
var Pie = require('../common/Pie.js');
var Bar = require('../common/Bar.js');
const CHILD_ITEM = [
    '环形图',
    '折线图',
    '饼状图',
    '柱状图',
];

module.exports = React.createClass({
    statics: {
        title: CONSTANTS.APP_NAME,
    },
    getInitialState () {
        return {
            currentIndex: 0,
        };
    },
    onPress (index) {
        this.setState({ currentIndex: index });
    },
    headItem (itemStr, i) {
        return (
            <TouchableOpacity
                onPress={this.onPress.bind(null, i)}
                style={styles.itemView}>
                <Text style={this.state.currentIndex === i ? styles.selectText : styles.text}>
                    {itemStr}
                </Text>
                {this.state.currentIndex === i && <View style={styles.lineView} />}
            </TouchableOpacity>
        );
    },
    headView () {
        return (
            <View style={styles.textView}>
                {this.headItem(CHILD_ITEM[0], 0)}
                {this.headItem(CHILD_ITEM[1], 1)}
                {this.headItem(CHILD_ITEM[2], 2)}
                {this.headItem(CHILD_ITEM[3], 3)}
            </View>
        );
    },
    render () {
        return (
            <View style={styles.container}>
                <this.headView />
                {
                    this.state.currentIndex === 0 &&
                    <Circle
                        style={styles.containerWebView}
                        source={{ uri:'http://www.baidu.com' }}
                        scalesPageToFit
                        />
                }
                {
                    this.state.currentIndex === 1 &&
                    <Line
                        style={styles.containerWebView}
                        source={{ uri:'http://www.sohu.com' }}
                        scalesPageToFit
                        />
                }
                {
                    this.state.currentIndex === 2 &&
                    <Pie
                        style={styles.containerWebView}
                        source={{ uri:'http://www.sina.com' }}
                        scalesPageToFit
                        />
                }
                {
                    this.state.currentIndex === 3 &&
                    <Bar
                        style={styles.containerWebView}
                        source={{ uri:'http://www.hao123.com' }}
                        scalesPageToFit
                        />
                }
            </View>
        );
    },
});

var styles = StyleSheet.create({
    container: {
        flex:1,
        backgroundColor: '#E6EBEC',
    },
    containerWebView: {
        width: sr.w,
        height: sr.h - 45,
    },
    textView: {
        marginTop: 80,
        height: 45,
        flexDirection: 'row',
    },
    itemView: {
        width: sr.w / 4,
        height: 43,
    },
    lineView: {
        marginTop: 10,
        height: 2,
        backgroundColor: '#34a9b1',
    },
    selectText: {
        fontSize: 14,
        color: '#34a9b1',
        alignSelf: 'center',
    },
    text: {
        fontSize: 14,
        color: '#666666',
        alignSelf: 'center',
    },
    blankView: {
        width: sr.w,
        height: 30,
    },
    ItemBg: {
        marginTop: 1,
        padding: 10,
        height: 45,
        flexDirection: 'row',
        backgroundColor: '#FFFFFF',
        alignItems: 'center',
        justifyContent: 'space-between',
    },
    icon_go: {
        width: 8,
        height: 15,
    },
    infoStyle: {
        flex: 3,
        flexDirection: 'row',
        alignSelf: 'flex-start',
        alignItems: 'center',
    },
    icon_item: {
        width: 25,
        height: 25,
    },
    itemNameText: {
        fontSize: 15,
        color: 'gray',
        alignSelf: 'center',
        marginLeft: 10,
    },
});

```

* PDShop/project/App/modules/test/index.js

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

* PDShop/project/App/modules/test/pageList.js

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

* PDShop/project/App/modules/update/UpdateInfoBox.js

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
                androidApkDownloadDestPath:'/sdcard/pdshop.apk',
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

* PDShop/project/App/modules/update/UpdatePage.js

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
                androidApkDownloadDestPath:'/sdcard/pdshop.apk',
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

* PDShop/project/App/modules/warehouseDepartment/Order.js

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

const WarehouseOrderList = require('./OrderList');

module.exports = React.createClass({
    getInitialState () {
        return {
            tabIndex: 0,
        };
    },
    onWillFocus () {
        app.getCurrentRoute().title = '货单';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    componentDidMount () {
        this.getOrders();
    },
    changeTab (tabIndex) {
        this.setState({ tabIndex });
    },
    getOrders () {
        const param = {
            userId: app.personal.info.userId,
            type:'',
            pageNo:0,
            pageSize:3,
        };
        POST(app.route.ROUTE_GET_ORDERS_BY_WAREHOUSE_PARTMENT, param, this.getOrdersSuccess, false);
    },
    getOrdersSuccess (data) {
        if (data.success) {
            this.setState({ context:data.context });
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const menus = [{ name:'已入库', type: 'stored' }, { name:'已出库', type: 'loaded' }];
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
                        <WarehouseOrderList key={i} index={i} type={item.type} tabIndex={tabIndex} data={context && context[item.type]} listUrl={app.route.ROUTE_GET_ORDERS_BY_WAREHOUSE_PARTMENT} />
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

* PDShop/project/App/modules/warehouseDepartment/OrderDetail.js

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

const { DImage } = COMPONENTS;
import moment from 'moment';

module.exports = React.createClass({
    statics: {
        title: '货单详情',
    },
    render () {
        const obj = this.props.data;
        return (
            <View style={styles.container}>
                <View style={styles.containerReceive}>
                    <Text style={styles.textAddress}> 收货地址：{obj.endPoint} </Text>
                    <View style={styles.receiveMemberRow}>
                        <Text style={styles.receiveMember}> 收货人：{!obj.receiverName ? '' : obj.receiverName}</Text>
                        <Text style={styles.receiveMember}>{obj.receiverPhone}</Text>
                    </View>
                </View>
                <View style={styles.containerMarket}>
                    <View style={styles.containerMarketLeft}>
                        <Image style={styles.imageLogo}
                            source={app.img.order_logo}
                            resizeMode='stretch'
                            />
                        <Text style={styles.textMarket}>{obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : ''}</Text>
                    </View>
                </View>
                <View style={styles.containerDetails}>
                    <DImage style={styles.imageGoods}
                        resizeMode='stretch'
                        defaultSource={app.img.common_default}
                        source={{ uri: obj.photo }}
                        />
                    <View style={styles.goodsDetails}>
                        <View style={styles.goodsDetailsTop}>
                            <View style={styles.detailsTopLeft}>
                                <Text style={styles.detailsAddress}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{obj.shop ? obj.shop.address : obj.agent ? obj.agent.address : ''}</Text>
                                <Text >-</Text>
                                <Text style={styles.detailsAddress}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{obj.endPoint}</Text>
                            </View>
                            <View style={styles.detailsTopRight}>
                                <Text style={styles.moneyLeft}>运费：</Text>
                                <Text style={styles.moneyRight}>¥{obj.needPayInsuanceFee + obj.needPayTransportFee}</Text>
                            </View>
                        </View>
                        <Text style={styles.textDetailsItem}>重量：{obj.weight}吨</Text>
                        <Text style={styles.textDetailsItem}>方量：{obj.size}m³</Text>
                        <Text style={styles.textDetailsItem}>件数：{obj.totalNumbers}件</Text>
                        <Text style={styles.textDetailsItem}>下单时间：{moment().format('YYYY-MM-DD')}</Text>
                    </View>
                </View>
                <View style={styles.containerWhite} />
                <View style={styles.containerNumber}>
                    <Text style={styles.textNumber}>订单号：{obj.id}</Text>
                    <Text style={styles.textDate}>创建时间：{moment(obj.createTime).format('YYYY-MM-DD')}</Text>
                </View>
                <View style={styles.containerInfo}>
                    <Text style={styles.textInfoTop}>运费：¥{obj.needPayTransportFee}</Text>
                    <Text style={styles.textInfo}>保险费：¥{obj.needPayInsuanceFee}</Text>
                    <Text style={styles.textInfo}>
                        支付方式：{'货到付款'}
                    </Text>
                    <Text style={styles.textTotal}>总计：¥{obj.needPayInsuanceFee + obj.needPayTransportFee}</Text>
                </View>
                <TouchableOpacity>
                    <View style={styles.logInfo}>
                        <Text style={styles.textLogInfo}>操作人</Text>
                        <View style={styles.logInfoRight}>
                            <Text style={styles.textLogInfoRight}>{obj.receiveMember.name}</Text>
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
        height:50,
        backgroundColor:'#ffffff',
        justifyContent: 'center',
    },
    textAddress:{
        marginLeft:10,
        fontSize:14,
        marginBottom:3,
        color:'#333333',
    },
    receiveMemberRow:{
        flexDirection:'row',
    },
    receiveMember:{
        marginLeft:10,
        fontSize:12,
        marginTop:3,
        color:'#333333',
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
        backgroundColor:'#f4f4f4',
        flexDirection: 'row',
        alignItems:'center',
    },
    containerWhite:{
        height:8,
        backgroundColor:'#ffffff',
    },
    imageGoods:{
        height:75,
        width:75,
        marginLeft:10,
    },
    goodsDetails:{
        marginLeft:8,
        justifyContent: 'center',
    },
    goodsDetailsTop:{
        width:sr.w - 99,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
    },
    detailsTopLeft:{
        flexDirection: 'row',
        alignItems: 'center',
    },
    detailsTopRight:{
        flexDirection: 'row',
        alignItems: 'center',
    },
    detailsAddress:{
        fontSize:14,
        width:60,
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
        fontSize:12,
    },
    containerInfo:{
        backgroundColor:'#ffffff',
        marginTop:8,
        paddingLeft:10,
        paddingBottom:4,
    },
    textInfoTop:{
        marginTop:4,
        color:'#888888',
        fontSize:12,
    },
    textInfo:{
        marginTop:2,
        color:'#888888',
        fontSize:12,
    },
    textTotal:{
        marginTop:5,
        fontSize:12,
        color:'#333333',
        fontWeight:'400',
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
        color:'#aaaaaa',
        marginRight:5,
    },
    imageCommon_go:{
        width: 8,
        height: 15,
        marginRight: 7,
    },
});

```

* PDShop/project/App/modules/warehouseDepartment/OrderList.js

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

const { PageList, DImage } = COMPONENTS;
const WarehouseOrderDetail = require('./OrderDetail');
import moment from 'moment';

module.exports = React.createClass({
    statics: {
        title: '货单',
    },
    doLookDetail (obj) {
        app.push({
            component: WarehouseOrderDetail,
            passProps:{
                data:obj,
            },
        });
    },
    renderSeparator (sectionID, rowID) {
        return (
            <View style={styles.separator} key={sectionID + rowID} />
        );
    },
    renderRow (obj, rowID) {
        return (
            <View style={styles.row}>
                <View style={styles.rowTopContainer}>
                    <Image
                        resizeMode='stretch'
                        source={app.img.order_logo}
                        style={styles.topImage} />
                    <Text style={styles.topLeftText}>{obj.shop ? obj.shop.name : obj.agent ? obj.agent.name : ''}</Text>
                </View>
                <TouchableOpacity style={styles.rowMiddleContainer} onPress={this.doLookDetail.bind(null, obj)}>
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
                                <Text style={styles.pointName}>→</Text>
                                <Text style={styles.pointName}
                                    numberOfLines={1}
                                    ellipsizeMode='tail'>{obj.endPoint}</Text>
                            </View>
                            <Text style={styles.time}>{moment(obj.createTime).format('YYYY-MM-DD')}</Text>
                        </View>
                        <Text style={styles.commonInfo}>{'重量：'}{obj.weight}吨</Text>
                        <Text style={styles.commonInfo}>{'方量：'}{obj.size}m³</Text>
                        <Text style={styles.commonInfo}>{'件数：'}{obj.totalNumbers}件</Text>
                        {
                            this.props.type == 'stored' && obj.warehouse &&
                            <Text style={styles.commonInfo}>仓库号：{obj.warehouse}</Text>
                        }

                        <Text style={styles.commonInfo}>{'收货地址：'}{obj.endPoint}</Text>
                    </View>
                </TouchableOpacity>
            </View>
        );
    },
    render () {
        const { index, tabIndex, type, listUrl, data } = this.props;
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
                        autoLoad={false}
                        list={data.list}
                        listUrl={listUrl}
                        refreshEnable />
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
    typeContainer:{
        backgroundColor:'#c81622',
        height:80,
        width:60,
        position:'absolute',
        top:0,
        right:3,
    },
    typeButton:{
        height:25,
    },
    typeText:{
        fontSize:14,
        fontWeight:'400',
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
    rowMiddleContainer: {
        flexDirection: 'row',
        alignItems: 'center',
        paddingVertical:11,
        backgroundColor:'#f8f8f8',
    },
    middleImage: {
        width: 80,
        height:80,
        borderRadius: 10,
        marginLeft: 10,
    },
    middleRight: {
        paddingLeft: 10,
        width:sr.w - 100,
    },
    money: {
        fontSize: 14,
        color:'#f51b1a',
    },
    commonInfo: {
        fontSize: 13,
        color:'#888888',
    },
    pointName: {
        fontSize: 16,
        maxWidth: 60,
    },
    time: {
        flex:1,
        fontSize: 14,
        color: 'gray',
        textAlign:'right',
    },
    pointContainer: {
        flexDirection: 'row',
    },
    placeContainer: {
        flex:1,
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
    confirmReceive: {
        height:20,
        width:73,
        borderRadius:5,
        borderColor:'#c81622',
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
    btnReceiveText:{
        color:'#c81622',
        fontSize:12,
        fontWeight:'300',
    },
});

```

* PDShop/project/App/modules/warehouseDepartment/ScanOrderDetail.js

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

const { Button, DImage } = COMPONENTS;
import moment from 'moment';

module.exports = React.createClass({
    statics: {
        title: '扫描订单详情',
    },
    getInitialState () {
        return {
            data:[],
            count:1,
            force:false,
            unStoageNumbers:0,
        };
    },
    componentWillMount () {
        this.getOrderDetail();
    },
    getOrderDetail () {
        const { orderId } = this.props;
        const { force } = this.state;
        const param = {
            userId:app.personal.info.userId,
            orderId,
            count:0,
        };
        POST(app.route.ROUTE_PLACE_STORAGE, param, this.postScanOrderSuccess, true);
    },
    postScanOrderSuccess (data) {
        if (data.success) {
            this.setState({
                data: data.order,
                unStoageNumbers: _.find(data.order.stateList, o => o.state == 5 && o.count).count,
            });
            app.scanStoageOrder.saveOrder(data.order);
        } else {
            if (data.retry) {
                this.setState({force:!!data.retry,msg:data.msg,data:data.order,unStoageNumbers: _.find(data.order.stateList, o => o.state == 5 && o.count).count});
            } else {
                this.setState({force:!!data.retry,msg:''});
                Toast(data.msg);
                app.pop();
            }

        }
    },
    minusCount () {
        const { count } = this.state;
        if (count > 0) {
            this.setState({ count:parseInt(count) - 1 });
        } else {
            Toast('装车数量不能小于0');
        }
    },
    addCount () {
        const { count, unStoageNumbers } = this.state;
        if (unStoageNumbers > 0) {
            if (count < unStoageNumbers) {
                this.setState({ count:parseInt(count) + 1 });
            } else {
                Toast('装车数量不能大于未入库件数');
            }
        } else {
            Toast('该货单已全部入库');
        }
    },
    showScanResult () {
        const { orderId } = this.props;
        const { count, data, unStoageNumbers, force } = this.state;
        if (count > unStoageNumbers) {
            Toast('装车数量不能大于未入库件数');
            return;
        }
        app.navigator.replace({
            component:require('./ScanResult'),
            passProps:{
                orderId,
                force,
                count,
                data,
            },
        });
    },
    doCancel () {
        app.navigator.popToTop();
    },
    render () {
        const { data, count, unStoageNumbers, force, msg } = this.state;
        console.log('data', data);
        return (
            <View style={styles.container}>
                <View style={styles.containerDetails}>
                    <DImage style={styles.imageGoods}
                        defaultSource={app.img.common_default}
                        source={{ uri: data.photo }} />
                    <View style={styles.goodsDetails}>
                        <Text style={styles.textDetailsItem}>重量：{data.weight}m³</Text>
                        <Text style={styles.textDetailsItem}>方量：{data.size}吨</Text>
                        <Text style={styles.textDetailsItem}>未入库件数：{unStoageNumbers}件</Text>
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
                        maxLength={4}
                        onChangeText={(text) => this.setState({ count:text })}
                        keyboardType='numeric' />
                    <TouchableOpacity onPress={this.addCount}>
                        <Text style={styles.imgSymbol}>＋</Text>
                    </TouchableOpacity>
                </View>
                {
                    force&&<Text style={styles.msg}>{msg}</Text>
                }
                <View style={styles.bottom}>
                    <Button style={styles.btnConfirm} textStyle={styles.btnFont} onPress={this.showScanResult}>{force?'强制入库':'确认'}</Button>
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
        marginTop:10,
        height:125,
        paddingBottom:8,
        justifyContent: 'center',
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
        fontSize:20,
        fontWeight:'400',
        marginRight:10,
    },
    countFont:{
        padding:0,
        fontSize:14,
        width:30,
        height:20,
        color:'#666666',
        backgroundColor:'#FFFFFF',
        marginRight:10,
        alignSelf:'center',
        textAlign:'right',
    },
    msg :{
        color:'red',
        textAlign:'center',
        fontSize:16,
        marginTop:100
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

* PDShop/project/App/modules/warehouseDepartment/ScanResult.js

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

const { Button, DImage } = COMPONENTS;
const StorageDetail = require('./StorageDetail');
const ScanBarCode = require('../common/ScanBarCode');

import moment from 'moment';

module.exports = React.createClass({
    statics: {
        title: '扫描结果',
        leftButton: { handler: () => { app.scene.goBack(); } },
    },
    goBack () {
        app.navigator.popToTop();
    },
    getInitialState () {
        return {
            list: [],
            status:true,
        };
    },
    confirmScan () {
        app.push({
            component: StorageDetail,
        });
    },
    componentWillMount () {

        this.getPlaceStorage();
    },
    getPlaceStorage () {
        const { orderId, count, force } = this.props;
        const param = {
            userId : app.personal.info.userId,
            orderId,
            force,
            count:count * 1,
        };
        POST(app.route.ROUTE_PLACE_STORAGE, param, this.getPlaceStorageSuccess, false);
    },
    getPlaceStorageSuccess (data) {
        if (data.success) {
            this.setState({
                list:data.order,
            });
            app.scanStoageOrder.saveOrder(data.order);
        } else {
            Toast(data.msg);
        }
    },
    continueScanOrder () {
        app.showModal(
            <ScanBarCode onBarCodeRead={this.onBarCodeRead} />
        );
    },
    onBarCodeRead (data) {
        if (!data || data.type != CONSTANTS.QRCODE_TYPE_ORDER) { // type==1代表货单二维码
            Toast('无效的二维码');
        } else {
            app.navigator.replace({
                component:require('./ScanOrderDetail'),
                passProps:{
                    orderId:data.id,
                },
            });
        }
    },
    render () {
        const { list } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.header}>
                    <View style={styles.top}>
                        <View style={styles.headerItem}>
                            <Text style={styles.headerFontTop}>{app.scanStoageOrder.stoageNumbers ? app.utils.N(app.scanStoageOrder.stoageNumbers) : 0}单</Text>
                            <Text style={styles.headerFont}>入库单数</Text>
                        </View>
                        <View style={styles.headerItem}>
                            <Text style={styles.headerFontTop}>{app.scanStoageOrder.unStoageNumbers ? app.utils.N(app.scanStoageOrder.unStoageNumbers) : 0}单</Text>
                            <Text style={styles.headerFont}>未入库单数</Text>
                        </View>
                        <View style={styles.headerItem}>
                            <Text style={styles.headerFontTop}>{app.utils.N(app.scanStoageOrder.totalSize)}m³</Text>
                            <Text style={styles.headerFont}>总方量</Text>
                        </View>
                        <View style={styles.headerItem}>
                            <Text style={styles.headerFontTop}>{app.utils.N(app.scanStoageOrder.totalWeight)}吨</Text>
                            <Text style={styles.headerFont}>总重量</Text>
                        </View>
                    </View>
                    <View style={styles.order}>
                        <DImage
                            resizeMode='cover'
                            defaultSource={app.img.common_default}
                            source={{ uri: list.photo }}
                            style={styles.icon} />
                        <View style={styles.rowInfo}>
                            <Text style={styles.id}
                                numberOfLines={1}
                                ellipsizeMode='tail'>货单号:{list.id}</Text>
                            <Text style={styles.info}>重量:{list.weight}吨</Text>
                            <Text style={styles.info}>方量:{list.size}m³</Text>
                            <Text style={styles.info}>收货地址:{list.endPoint}</Text>
                        </View>
                        <Text style={styles.timeInfo}>{moment(list.createTime).format('YYYY-MM-DD')}</Text>
                    </View>
                </View>
                <Button style={styles.btnContinue} textStyle={styles.btnContinueFont} onPress={this.continueScanOrder}>继续扫描</Button>
                <Button style={styles.btnConfirm} textStyle={styles.btnConfirmFont} onPress={this.confirmScan}>确认扫描</Button>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor:'#f4f4f4',
    },
    header:{
        height:200,
    },
    top:{
        flexDirection:'row',
        height:60,
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent:'center',
    },
    headerItem:{
        flex:1,
        alignItems:'center',
    },
    headerFontTop:{
        fontSize:14,
        color:'#333333',
        marginBottom:2,
    },
    headerFont:{
        fontSize:14,
        color:'#888888',
        marginTop:2,
    },
    order:{
        flexDirection:'row',
        height:90,
        backgroundColor:'#FFFFFF',
        marginTop:10,
        alignItems:'center',
    },
    headText:{
        color:'#333333',
        marginTop:15,
        marginLeft:10,
        fontSize:14,
        flexDirection:'row',
    },
    unload:{
        height:40,
        backgroundColor:'#FFFFFF',
        marginTop:5,
        flexDirection:'row',
        alignItems:'center',
        justifyContent:'space-between',
    },
    unloadLeft:{
        flexDirection:'row',
    },
    carNumber:{
        fontSize:14,
        color:'#333333',
        marginLeft:10,
    },
    goodsNumber:{
        fontSize:12,
        marginTop:2,
        marginLeft:6,
        color:'#f29c00',
    },
    unloadRight:{
        fontSize:14,
        color:'red',
        marginRight:10,
    },
    previous:{
        fontSize:15,
        color:'#ec202b',
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
    infoRed:{
        fontSize:12,
        color:'red',
        marginTop:5,
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
        width:sr.w - 24,
        height:45,
        borderRadius:4,
        backgroundColor:'#FE9917',
        alignSelf:'center',
        marginTop:sr.h - 565,
    },
    btnConfirm:{
        width:sr.w - 24,
        height:45,
        borderRadius:4,
        backgroundColor:'#dddddd',
        alignSelf:'center',
        marginTop:20,
    },
    btnContinueFont:{
        color:'#FFFFFF',
        fontWeight:'400',
        fontSize:16,

    },
    btnConfirmFont:{
        color:'#333333',
        fontWeight:'400',
        fontSize:16,
    },
});

```

* PDShop/project/App/modules/warehouseDepartment/Storage.js

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

const ScanOrderDetail = require('./ScanOrderDetail');
const ScanBarCode = require('../common/ScanBarCode');

module.exports = React.createClass({
    statics: {
        title: '扫描入库',
    },
    onWillFocus () {
        app.getCurrentRoute().title = '扫描入库';
        app.getCurrentRoute().rightButton = null;
        app.getCurrentRoute().leftButton = null;
        app.forceUpdateNavbar();
    },
    scanBarCode () {
        app.showModal(
            <ScanBarCode onBarCodeRead={this.onBarCodeRead} />
        );
    },
    onBarCodeRead (data) {
        if (!data || data.type != CONSTANTS.QRCODE_TYPE_ORDER) { // type==1代表货单二维码
            Toast('无效的二维码');
        } else {
            app.push({
                component:ScanOrderDetail,
                passProps:{
                    orderId:data.id,
                },
            });
        }
    },
    render () {
        return (
            <View style={styles.container}>
                <View style={styles.container}>
                    <Image />
                    <Image source={app.img.storage_scan_code_storage}
                        style={styles.imgScanCode}
                        resizeMode='contain' />
                    <TouchableOpacity style={styles.btnScan} onPress={this.scanBarCode}>
                        <Image source={app.img.storage_scan}
                            style={styles.imgscan}
                            resizeMode='contain' />
                        <Text style={styles.scanFont}>扫一扫</Text>
                    </TouchableOpacity>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        alignItems:'center',
    },
    container1: {
        flex: 1,
        width: sr.w,
    },
    preview: {
        flex: 1,
        justifyContent: 'flex-end',
        alignItems: 'center',
    },
    imgScanCode:{
        width:sr.w - 205,
        height:sr.h - 475,
        marginTop:sr.h - 620,
    },
    btnScan:{
        marginTop:90,
        flexDirection:'row',
        width:sr.w - 225,
        height:35,
        backgroundColor:'#c61a29',
        alignItems:'center',
        justifyContent:'center',
        borderRadius:4,
    },
    imgscan:{
        width:14,
        height:14,
    },
    scanFont:{
        color:'#ffffff',
        marginLeft:5,
        fontWeight:'500',
    },
});

```

* PDShop/project/App/modules/warehouseDepartment/StorageDetail.js

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

const { DImage } = COMPONENTS;

module.exports = React.createClass({
    statics: {
        title: '入库详情',
        leftButton: { handler: () => { app.scene.goBack(); } },
    },
    goBack () {
        app.scanStoageOrder.clear();
        app.navigator.popToTop();
    },
    getInitialState () {
        this.ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 });
        return {};
    },
    renderRow (obj, sectionID, rowID) {
        console.log('obj', obj);
        return (
            <View style={styles.row}>
                <DImage
                    resizeMode='contain'
                    defaultSource={app.img.common_default}
                    source={{ uri: obj.photo }}
                    style={styles.img} />
                <View style={styles.infoRow}>
                    <Text style={styles.id}>货单号：{obj.id}</Text>
                    <Text style={styles.info}>重量：{obj.weight}吨</Text>
                    <Text style={styles.info}>方量：{obj.size}m³</Text>
                    <Text style={styles.infoBottom}>收货地址：{obj.endPoint}</Text>
                </View>
            </View>
        );
    },
    render () {
        return (
            <View style={styles.container}>
                <View style={styles.top}>
                    <View style={styles.headerItem}>
                        <Text style={styles.headerFontTop}>{app.scanStoageOrder.stoageNumbers ? app.utils.N(app.scanStoageOrder.stoageNumbers) : 0}单</Text>
                        <Text style={styles.headerFont}>入库单数</Text>
                    </View>
                    <View style={styles.headerItem}>
                        <Text style={styles.headerFontTop}>{app.scanStoageOrder.unStoageNumbers ? app.utils.N(app.scanStoageOrder.unStoageNumbers) : 0}单</Text>
                        <Text style={styles.headerFont}>未入库单数</Text>
                    </View>
                    <View style={styles.headerItem}>
                        <Text style={styles.headerFontTop}>{app.utils.N(app.scanStoageOrder.totalSize)}m³</Text>
                        <Text style={styles.headerFont}>入库总方量</Text>
                    </View>
                    <View style={styles.headerItem}>
                        <Text style={styles.headerFontTop}>{app.utils.N(app.scanStoageOrder.totalWeight)}吨</Text>
                        <Text style={styles.headerFont}>入库总重量</Text>
                    </View>
                </View>
                <ListView
                    style={styles.ListView}
                    dataSource={this.ds.cloneWithRows(app.scanStoageOrder.list)}
                    renderRow={this.renderRow}
                    />
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    row:{
        backgroundColor:'#FFFFFF',
        height:100,
        marginTop:8,
        paddingLeft:10,
        flexDirection:'row',
        alignItems:'center',
    },
    top:{
        flexDirection:'row',
        height:60,
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent:'center',
    },
    headerItem:{
        flex:1,
        alignItems:'center',
    },
    headerFontTop:{
        fontSize:14,
        color:'#333333',
    },
    headerFont:{
        fontSize:14,
        color:'#888888',
    },
    id:{
        marginBottom:6,
        fontSize:14,
        color:'#333333',
    },
    info:{
        marginTop:2,
        fontSize:12,
        color:'#888888',
    },
    infoBottom:{
        marginTop:2,
        marginBottom:1,
        fontSize:12,
        color:'#888888',
    },
    img:{
        width:80,
        height:80,
    },
    infoRow:{
        marginLeft:10,
        justifyContent:'center',
    },
});

```

* PDShop/project/App/modules/warehouseDepartment/index.js

```js
'use strict';
const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
} = ReactNative;

import TabNavigator from 'react-native-tab-navigator';

const Storage = require('./Storage');
const Order = require('./Order');
const Person = require('../person');

const INIT_ROUTE_INDEX = 0;
const ROUTE_STACK = [
    { index: 0, component: Storage },
    { index: 1, component: Order },
    { index: 2, component: Person },
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
            { index: 0, title: '入库', icon: app.img.home_storage, selected: app.img.home_storage_press },
            { index: 1, title: '货单', icon: app.img.home_warehouse_order, selected: app.img.home_warehouse_order_press },
            { index: 2, title: '我的', icon: app.img.home_mine, selected: app.img.home_mine_press },
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

* PDShop/project/App/utils/check/index.js

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
     * 验证二维码数据是否合法
    */
    checkQRCodeData (str) {
        var re = /[a-z0-9]{24}/;
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
};

```

* PDShop/project/App/utils/common/index.js

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
};

```

* PDShop/project/App/utils/date/index.js

```js
const moment = require('moment');

module.exports = {
    createDateData (start, end) {
        let date = [];
        let iy = start.year(), im = start.month() == 0 ? start.month() + 1 : start.month(), id = start.date();
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
    createMonthData (start, end) {
        let date = [];
        let iy = start.year(), im = start.month() == 0 ? start.month() + 1 : start.month();
        let ey = end.year(), em = end.month() + 1;
        for (let y = iy; y <= ey; y++) {
            let month = [];
            let iim = (y === iy) ? im : 1;
            let eem = (y === ey) ? em : 12;
            for (let m = iim; m <= eem; m++) {
                month.push(m + '月');
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

* PDShop/project/App/utils/net/Get.js

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

* PDShop/project/App/utils/net/Post.js

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

* PDShop/project/App/utils/net/Upload.js

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

* PDShop/project/App/utils/net/index.js

```js
'use strict';

module.exports = {
    POST: require('./Post'),
    GET: require('./Get'),
    UPLOAD: require('./Upload'),
};

```

* PDShop/project/App/utils/qrcode/index.js

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
