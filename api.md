* PDDriver/project/App/index.js

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

* PDDriver/project/App/components/AutogrowInput.js

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

* PDDriver/project/App/components/Button.js

```js
'use strict';

const React = require('react');const {    PropTypes,} = React;const ReactNative = require('react-native');
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

* PDDriver/project/App/components/ClipRect.js

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

* PDDriver/project/App/components/CustomSheet.js

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

* PDDriver/project/App/components/DImage.js

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

* PDDriver/project/App/components/DelayTouchableOpacity.js

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

* PDDriver/project/App/components/Label.js

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

* PDDriver/project/App/components/MessageBox.js

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

* PDDriver/project/App/components/Modal.js

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

* PDDriver/project/App/components/PageList.js

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
            dataSource: this.list,
        });
    },
    getInitialState () {
        this.list = this.props.list || [];
        this.pageNo = this.props.pageNo;
        return {
            dataSource: this.list,
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
            <FlatList                onEndReached={this.onEndReached}
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

* PDDriver/project/App/components/PageListView.js

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

* PDDriver/project/App/components/Picker.js

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
                    pickerConfirmBtnColor: [0, 0, 255, 1],
                    pickerCancelBtnText: '取消',
                    pickerCancelBtnColor: [255 - pickerToolBarBg[0] / 2, 255 - pickerToolBarBg[1] / 2, 255 - pickerToolBarBg[2] / 2, 1],
                    pickerTitleText: title || '',
                    pickerTitleColor: [255, 255, 255, 1],
                    pickerToolBarBg,
                    pickerBg: [207, 207, 207, 1],
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

* PDDriver/project/App/components/ProgressBar.js

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

* PDDriver/project/App/components/ProgressHud.js

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

* PDDriver/project/App/components/RText.js

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
        const { children, style, ...props } = this.props;
        const s = StyleSheet.flatten(style);
        const { lineHeight, fontSize } = s;
        const paddingVertical = (lineHeight - fontSize) / 2;
        return (
            <Text
                style={[{ paddingVertical }, style]}
                testID={this.props.testID}
                {...props}
                >
                {children}
            </Text>
        );
    },
});

module.exports = Platform.OS === 'android' ? RText : Text;

```

* PDDriver/project/App/components/ScoreSelect.js

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

* PDDriver/project/App/components/SelectRegionAddress.js

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

* PDDriver/project/App/components/SelectShortAddress.js

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

* PDDriver/project/App/components/SelectStartPointAddress.js

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
        const param = { userId: app.personal.info.userId, addressLastCode, isLeaf: !!id };
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

* PDDriver/project/App/components/Slider.js

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

* PDDriver/project/App/components/StarBar.js

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

* PDDriver/project/App/components/TouchAbleCall.js

```js
import React, { Component } from 'react';
import {
    Text,
    Linking,
    TouchableOpacity,
} from 'react-native';
class TouchableCall extends Component {
    protoTypes:{
        url:React.ProtoTypes.string
    }
    render () {
        return (
            <TouchableOpacity onPress={() => {
                Linking.canOpenURL(this.props.url).then(supported => {
                    if (!supported) {
                        console.log('Can\'t handle url: ' + this.props.url);
                    } else {
                        return Linking.openURL(this.props.url);
                    }
                }).catch(err => console.error('An error occurred', err));
            }}>
                <Text>{this.props.url}</Text>
            </TouchableOpacity>

        );
    }
}
module.exports = TouchableCall;

```

* PDDriver/project/App/components/WebviewMessageBox.js

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
                        确定</Button>
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
        color: '#FFFFFF',
        fontWeight: '800',
    },
});

```

* PDDriver/project/App/components/index.js

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
    WebviewMessageBox: require('./WebviewMessageBox'),
    DelayTouchableOpacity: require('./DelayTouchableOpacity'),
    StarBar: require('./StarBar'),
    ProgressHud: require('./ProgressHud'),
    RText: require('./RText'),
    ScoreSelect: require('./ScoreSelect'),
    AutogrowInput: require('./AutogrowInput'),
    TouchableCall: require('./TouchAbleCall'),
    Picker: require('./Picker'),
    SelectRegionAddress: require('./SelectRegionAddress'),
    SelectStartPointAddress: require('./SelectStartPointAddress'),
    SelectShortAddress: require('./SelectShortAddress'),
};

```

* PDDriver/project/App/config/Color.js

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

* PDDriver/project/App/config/Constants.js

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
    APP_NAME: '四面通司机端',
    THEME_COLORS: ['#c81622', '#A62045', '#239FDB'],
    ISSUE_IOS: CONFIG.ISSUE_IOS,
    NOT_NEED_UPDATE_JS_START: !(CONFIG.ISSUE || TEST_CONFIG.ISSUE), // 启动时不需要更新小版本
    MINIFY: CONFIG.ISSUE, // 是否压缩js文件，我们采取测试服务器为了查找问题不用压缩js文件，正式服务器需要压缩js文件，并且不能看到调试信息
    CHANNEL: CONFIG.CHANNEL,
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
    FEEDBACK_MAX_LENGTH: 500,
};

```

* PDDriver/project/App/config/Route.js

```js
'use strict';

const { BASE_SERVER } = CONSTANTS;
const ROOT_SERVER = 'http://' + BASE_SERVER + '/';
const API_SERVER = ROOT_SERVER + 'api/';
const SERVER = API_SERVER + 'driver/';

module.exports = {
    // 登录
    ROUTE_REGISTER: SERVER + 'register', // 注册
    ROUTE_LOGIN: SERVER + 'login', // 登录
    ROUTE_FIND_PASSWORD: SERVER + 'findPassword', // 忘记密码
    ROUTE_MODIFY_PASSWORD: SERVER + 'modifyPassword', // 修改密码
    ROUTE_REQUEST_SEND_VERIFY_CODE: API_SERVER + 'requestSendVerifyCode', // 获取验证码

    // 路上卡车信息
    ROUTE_GET_ON_WAY_TRUCK: SERVER + 'getOnWayTruck', // 获取路上卡车信息
    ROUTE_UPLOAD_LOCATION: SERVER + 'uploadLocation', // 上传卡车位置信息
    ROUTE_CONFIRM_REACH: SERVER + 'confirmReach', // 确认目的地

    // 个人中心
    ROUTE_GET_PERSONAL_INFO: SERVER + 'getPersonalInfo', // 获取个人信息
    ROUTE_MODIFY_PERSONAL_INFO: SERVER + 'modifyPersonalInfo', // 修改个人信息
    ROUTE_SUBMIT_FEEDBACK: SERVER + 'submitFeedback', // 提交信息反馈

    // 文件上传
    ROUTE_UPDATE_FILE: API_SERVER + 'uploadFile', // 上传文件

    // 获取地址
    ROUTE_GET_REGION_ADDRESS_LIST: API_SERVER + 'getRegionAddress', // 获取地址列表
    ROUTE_GET_START_POINT_ADDRESS_LIST: API_SERVER + 'getStartPointAddress', // 获取地址列表

    // 网页地址
    ROUTE_USER_LICENSE: ROOT_SERVER + 'protocals/pddriver/user.html', // 用户协议
    ROUTE_SOFTWARE_LICENSE: ROOT_SERVER + 'protocals/pddriver/software.html', // 获取软件许可协议
    ROUTE_ABOUT_PAGE: ROOT_SERVER + 'protocals/pddriver/about.html', // 关于

    // 下载更新
    ROUTE_VERSION_INFO_URL: ROOT_SERVER + 'apps/pddriver/version.json', // 版本信息地址
    ROUTE_JS_ANDROID_URL: ROOT_SERVER + 'apps/pddriver/jsandroid.zip', // android jsbundle 包地址
    ROUTE_JS_IOS_URL: ROOT_SERVER + 'apps/pddriver/jsios.zip', // ios jsbundle 包地址
    ROUTE_APK_URL: ROOT_SERVER + 'apps/pddriver/pddriver.apk', // apk地址
};

```

* PDDriver/project/App/config/Screen.js

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

* PDDriver/project/App/manager/JpushMgr.js

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

* PDDriver/project/App/manager/LoginMgr.js

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

* PDDriver/project/App/manager/NetMgr.js

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

* PDDriver/project/App/manager/PersonalInfoMgr.js

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

* PDDriver/project/App/manager/SettingMgr.js

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

* PDDriver/project/App/manager/UpdateMgr.js

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

* PDDriver/project/App/resource/audio.js

```js
module.exports = {};

```

* PDDriver/project/App/resource/image.js

```js
module.exports = {
    common_back:require('./image/common/back.png'),
    common_close:require('./image/common/close.png'),
    common_default:require('./image/common/default.png'),
    common_go:require('./image/common/go.png'),
    common_home:require('./image/common/home.png'),
    common_left_menu:require('./image/common/left_menu.png'),
    common_logo:require('./image/common/logo.png'),
    common_radio:require('./image/common/radio.png'),
    common_search:require('./image/common/search.png'),
    common_search_button:require('./image/common/search_button.png'),
    home_mine:require('./image/home/mine.png'),
    home_mine_press:require('./image/home/mine_press.png'),
    home_position:require('./image/home/position.png'),
    home_position_press:require('./image/home/position_press.png'),
    home_qrcode:require('./image/home/qrcode.png'),
    home_qrcode_press:require('./image/home/qrcode_press.png'),
    login_alipay_button:require('./image/login/alipay_button.png'),
    login_choose:require('./image/login/choose.png'),
    login_logo:require('./image/login/logo.png'),
    login_password:require('./image/login/password.png'),
    login_password_check:require('./image/login/password_check.png'),
    login_password_look:require('./image/login/password_look.png'),
    login_phone:require('./image/login/phone.png'),
    login_qq_button:require('./image/login/qq_button.png'),
    login_unchoose:require('./image/login/unchoose.png'),
    login_weixin_button:require('./image/login/weixin_button.png'),
    personal_add:require('./image/personal/add.png'),
    personal_default_head:require('./image/personal/default_head.png'),
    personal_female:require('./image/personal/female.png'),
    personal_info:require('./image/personal/info.png'),
    personal_male:require('./image/personal/male.png'),
    personal_money:require('./image/personal/money.png'),
    personal_personal_center:require('./image/personal/personal_center.png'),
    personal_scan:require('./image/personal/scan.png'),
    personal_settings:require('./image/personal/settings.png'),
    personal_wallet:require('./image/personal/wallet.png'),
    position_endpoint:require('./image/position/endpoint.png'),
    position_point:require('./image/position/point.png'),
    position_point_green:require('./image/position/point_green.png'),
    splash_logo:require('./image/splash/logo.png'),
    splash_splash1:require('./image/splash/splash1.png'),
    splash_splash2:require('./image/splash/splash2.png'),
    splash_splash3:require('./image/splash/splash3.png'),
    splash_start:require('./image/splash/start.png'),
};

```

* PDDriver/project/App/utils/index.js

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

* PDDriver/project/App/components/ActionSheet/button.js

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

* PDDriver/project/App/components/ActionSheet/index.js

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

* PDDriver/project/App/components/ActionSheet/overlay.js

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

* PDDriver/project/App/components/ActionSheet/sheet.js

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

* PDDriver/project/App/modules/baidumap/index.js

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

* PDDriver/project/App/modules/baidumap/test.js

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

* PDDriver/project/App/modules/common/CellPhoneBox.js

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

* PDDriver/project/App/modules/common/ImageCrop.js

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

* PDDriver/project/App/modules/common/ImagePicker.js

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

* PDDriver/project/App/modules/common/SelectClient.js

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

* PDDriver/project/App/modules/common/SelectRoadmap.js

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

* PDDriver/project/App/modules/common/ShowBigImage.js

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

* PDDriver/project/App/modules/home/index.js

```js
'use strict';
const React = require('react');const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Image,
} = ReactNative;

import TabNavigator from 'react-native-tab-navigator';
import { Geolocation } from '@remobile/react-native-baidu-map';
const Position = require('../position');
const Qrcode = require('../qrcode');
const Person = require('../person');
const schedule = require('node-cron');

const INIT_ROUTE_INDEX = 0;
const ROUTE_STACK = [
    { index: 0, component: Position },
    { index: 1, component: Qrcode },
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
        this.setSchedule();
    },
    componentWillUnmount () {
        app.hasLoadMainPage = false;
        this.task.destroy();
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
    setSchedule () {
        // this.task = schedule.schedule('0 1,3,5,7,9,11,13,15,17,19,21,23 * * *', function () {
        this.task = schedule.schedule('*/30 * * * * *', function () {
            Geolocation.getCurrentPosition()
                .then(data => {
                    console.warn(JSON.stringify(data));
                    const param = {
                        userId: app.personal.info.userId,
                        address:data.address || '未知',
                        latitude:data.latitude,
                        longitude:data.longitude,
                    };
                    POST(app.route.ROUTE_UPLOAD_LOCATION, param, () => {}, null);
                })
                .catch(e => {
                    console.warn(e, 'error');
                });
        });
    },
    render () {
        const menus = [
            { index: 0, title: '位置', icon: app.img.home_position, selected: app.img.home_position_press },
            { index: 1, title: '二维码', icon: app.img.home_qrcode, selected: app.img.home_qrcode_press },
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

* PDDriver/project/App/modules/login/ForgetPassword.js

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
    render () {
        const { readSecond } = this.state;
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
                            maxLength={CONSTANTS.VERIFICATION_MAX_LENGTH}
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

* PDDriver/project/App/modules/login/Login.js

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
const home = require('../home/index');

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
        } else {
            Toast(data.msg);
            app.hideLoading();
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
            app.personal.get();
            app.navigator.replace({
                component: home,
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

* PDDriver/project/App/modules/login/PasswordConfirm.js

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
                <View style={[styles.ItemBg, { marginTop:10 }]}>
                    <View style={styles.infoStyle}>
                        <Image resizeMode='stretch'
                            source={app.img.login_password}
                            style={styles.icon_add} />
                        <TextInput placeholderTextColor='#aaaaaa'
                            placeholder='请输入新密码'
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
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
    },
    infoStyle: {
        width:sr.w,
        flexDirection: 'row',
        justifyContent: 'space-between',
    },
    icon_add:{
        width:25,
        height:25,
        marginLeft: 8,
    },
    itemNameText: {
        flex:1,
        fontSize: 18,
        color: '#444444',
        marginLeft: 10,
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

* PDDriver/project/App/modules/login/Register.js

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
        if (!app.utils.checkVerificationCode(verifyCode)) {
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
                            resizeMode='stretch'
                            source={this.state.protocalRead ? app.img.login_choose : app.img.login_unchoose}
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
        marginTop:5,
        padding:0,
        fontSize:14,
        height:40,
        width: 180,
    },
    text_input2: {
        marginLeft: 12,
        fontSize:14,
        marginTop:5,
        padding:0,
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

* PDDriver/project/App/modules/person/About.js

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

* PDDriver/project/App/modules/person/BalanceDetail.js

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

* PDDriver/project/App/modules/person/BalanceDetailList.js

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
                        <Text style={styles.rechargeText}>
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
        height:50,
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
    },
    rechargeDate:{
        color:'#aaaaaa',
        paddingVertical:5,
        paddingLeft:8,
    },
    rechargeMoney:{
        color:'#2F971C',
        fontSize:14,
        paddingRight:10,
    },
    expenditureMoney:{
        color:'#C81622',
        fontSize:14,
        paddingRight:10,
    },
});

```

* PDDriver/project/App/modules/person/BindBankcard.js

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
                        <Text style={styles.itemNameText}>{'账号'}</Text>
                        <TextInput style={styles.itemNameLeftText}
                            placeholderTextColor='#aaaaaa'
                            placeholder='请填入银行卡号'
                            onChangeText={this.onContentTextChange}
                            />
                        <TouchableOpacity
                            activeOpacity={0.6}
                            onPress={this.doScanBankcard}>
                            <Image
                                resizeMode='stretch'
                                source={app.img.personal_scan}
                                style={styles.icon_add} />
                        </TouchableOpacity>
                    </View>
                </View>
                <View style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <Text style={styles.itemNameText}>{'姓名'}</Text>
                        <TextInput style={styles.itemNameLeftText}
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

* PDDriver/project/App/modules/person/BindBankcardList.js

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

module.exports = React.createClass({
    statics: {
        title: '银行卡',
    },
    getInitialState () {
        return {
            ds: new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 }),
            list: [1, 2],
        };
    },
    doAddBankcard () {
        app.push({
            component: BindBankcard,
        });
    },
    doUnBindBankcard (rowID) {
        this.state.list.splice(rowID, 1);
        this.setState({
            dataSource: this.state.list,
        });
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
                        onPress={() => this.doUnBindBankcard(rowID)}>
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
                    dataSource={this.state.ds.cloneWithRows(this.state.list)}
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

* PDDriver/project/App/modules/person/CashBalance.js

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
    getInitialState () {
        return {
            amount:'',
            type:'1',
        };
    },
    componentWillMount () {
        this.setState({
            amount:app.personal.remainAmount,
        });
    },
    detail () {
        app.push({
            component:BalanceDetailList,
        });
    },
    setCash (cash) {
        app.personal.setRemainAmount(cash);
        this.setState({
            amount:app.personal.remainAmount,
        });
    },
    showWithdrawCash () {
        app.push({
            component: WithdrawCash,
            passProps:{
                money:this.state.amount,
                setType:this.setCash,
            },
        });
    },
    showRecharge () {
        app.push({
            component: Recharge,
            passProps:{
                setType:this.setCash,
            },
        });
    },
    render () {
        const { amount } = this.state;
        return (
            <View style={styles.container}>
                <View style={styles.TopView}>
                    <Text style={styles.CashBalanceTitle}>账户余额</Text>
                    <View style={styles.CashBalance}>
                        <Text style={styles.CashBalanceTitleLeft}>￥</Text><Text style={styles.CashBalanceContent}>{amount}</Text>
                    </View>
                </View>
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

* PDDriver/project/App/modules/person/CommonSetting.js

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

* PDDriver/project/App/modules/person/EditBirthday.js

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
const moment = require('moment');

module.exports = React.createClass({
    getInitialState () {
        const info = app.personal.info;
        return {
            birthday: info.birthday,
        };
    },
    goBack () {
        Picker.hide();
    },
    createBirthdayData (now) {
        const date = [];
        const iy = now.year(), im = now.month() + 1, id = now.date();
        for (let y = 1916; y <= iy; y++) {
            const month = [];
            const mm = [0, 31, (!(y % 4) & (!!(y % 100))) | (!(y % 400)) ? 29 : 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
            const iim = (y == iy) ? im : 12;
            for (let m = 1; m <= iim; m++) {
                const day = [];
                const iid = (y == iy && m == im) ? id : mm[m];
                for (let d = 1; d <= iid; d++) {
                    day.push(d + '日');
                }
                month.push({ [m + '月']: day });
            }
            date.push({ [y + '年']: month });
        }
        return date;
    },
    toEditBirth () {
        const date = moment(this.state.birthday, 'YYYY-MM-DD');
        const selectedValue = [date.year() + '年', (date.month() + 1) + '月', date.date() + '日'];
        const pickerData = this.createBirthdayData(moment());
        Picker(pickerData, selectedValue, '选择生日').then((value) => {
            const birthday = moment(value.join(''), 'YYYY年M月D日').format('YYYY-MM-DD');
            this.setState({ birthday }, () => {
                this.modifyPersonalInfo();
            });
        });
    },
    modifyPersonalInfo () {
        const info = app.personal.info;
        const param = {
            userId: info.userId,
            birthday: this.state.birthday,
        };
        POST(app.route.ROUTE_MODIFY_PERSONAL_INFO, param, this.modifyUserInfoSuccess, true);
    },
    modifyUserInfoSuccess (data) {
        if (data.success) {
            const info = app.personal.info;
            info.birthday = this.state.birthday;
            app.personal.set(info);
            this.props.setBirth(info.birthday);
            app.pop();
        } else {
            Toast(data.msg);
        }
    },
    render () {
        const { birthday } = this.state;
        return (
            <View style={styles.container}>
                <View style={[styles.separatorView, { marginTop: 30 }]} />
                <TouchableOpacity
                    onPress={this.toEditBirth}
                    style={styles.itemView}>
                    <Text style={styles.itemKeyText}>{birthday || '请选择您的生日'}</Text>
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

* PDDriver/project/App/modules/person/EditPersonInfo.js

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
            licenseNo:info.licenseNo,
            address: info.address,
            head: info.head,
            sex: info.sex === 0 ? '男' : '女',
            phone:info.phone,
            pickerData: ['男', '女'],
        };
    },
    toEditHead () {
        Picker.hide();
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
    toEditSingleInfo (name, tip, maxLength, keyboardType, checkFunc) {
        Picker.hide();
        app.push({
            title: '修改' + tip,
            component: EditSingleInfo,
            passProps: {
                keyName: name,
                placeholder: tip,
                maxLength,
                keyboardType,
                checkFunc,
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
        const { name, address, sex, head, phone, licenseNo } = this.state;
        return (
            <View style={styles.container}>
                <View style={[styles.headStyle, { marginTop: 10 }]} >
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
                    onPress={this.toEditSingleInfo.bind(null, 'licenseNo', '驾驶证号', 18, 'default', app.utils.checkIdentifyNumber)}
                    style={styles.itemView}>
                    <Text style={styles.itemKeyText}>驾驶证号</Text>
                    <View style={styles.rightView} >
                        <Text style={styles.itemValueText}>{licenseNo}</Text>
                        <Image
                            source={app.img.common_go}
                            style={styles.goStyle} />
                    </View>
                </TouchableOpacity>
                <TouchableOpacity
                    onPress={this.toEditSingleInfo.bind(null, 'name', '真实姓名', CONSTANTS.NAME_MAX_LENGTH, 'default', null)}
                    style={[styles.itemView, { marginTop: 8 }]} >
                    <Text style={styles.itemKeyText}>真实姓名</Text>
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
                    onPress={this.toEditSingleInfo.bind(null, 'address', '地址', CONSTANTS.ADDRESS_MAX_LENGTH, 'default', app.utils.checkStr)}
                    style={styles.itemView}>
                    <Text style={styles.itemKeyText}>家庭地址</Text>
                    <View style={styles.rightView} >
                        <Text style={styles.addressText}
                            numberOfLines={1}>{address}</Text>
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
    },
    headStyle: {
        width: sr.w,
        height:60,

    },
    head_itemView:{
        backgroundColor: 'white',
        flexDirection: 'row',
        height: 60,
        justifyContent: 'space-between',
        alignItems:'center',
    },
    itemView: {
        marginTop:1,
        backgroundColor: 'white',
        flexDirection: 'row',
        height: 42,
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
    addressText:{
        color: '#888888',
        fontSize: 14,
        marginRight: 10,
        width: sr.w / 2,
        textAlign:'right',
    },
    btnChangeRole:{
        marginTop:15,
        height:45,
        width:351,
        marginLeft:(sr.w - 351) / 2,
        borderRadius:2,
    },
    btnLogout:{
        marginTop:215,
        height:45,
        width:351,
        marginLeft:(sr.w - 351) / 2,
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

* PDDriver/project/App/modules/person/EditSingleInfo.js

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
        const { placeholder, checkFunc } = this.props;
        const { value } = this.state;
        const info = app.personal.info;
        if (!!checkFunc && !checkFunc(value)) {
            Toast('请填写正确的' + placeholder);
            return;
        } else {
            if (value.length == 0) {
                Toast(placeholder + '不能为空');
                return;
            }
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
        const { placeholder, keyboardType, maxLength } = this.props;
        return (
            <View style={styles.container}>
                <View style={styles.itemView}>
                    <Text style={styles.itemKeyText} />
                    <TextInput
                        ref={(ref) => { this.contentInput = ref; }}
                        style={styles.rightView}
                        onChangeText={(text) => this.setState({ value: text })}
                        multiline={false}
                        keyboardType={keyboardType}
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

* PDDriver/project/App/modules/person/Feedback.js

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
                    underlineColorAndroid='transparent'
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
        textAlignVertical: 'top',
        padding:3,
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

* PDDriver/project/App/modules/person/Help.js

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

* PDDriver/project/App/modules/person/ImageCrop.js

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

* PDDriver/project/App/modules/person/ImagePicker.js

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

* PDDriver/project/App/modules/person/ModifyPassword.js

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

* PDDriver/project/App/modules/person/NormalQuestions.js

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

* PDDriver/project/App/modules/person/Recharge.js

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

module.exports = React.createClass({
    statics: {
        title: '充值',
    },
    getInitialState () {
        return {
            bankCard:'请选择银行卡',
            amount:1,
            type:'',
        };
    },
    showBankList () {
        app.push({
            component:BankCardList,
            passProps: { setBankCard: this.setBankCard },
        });
    },
    doRecharge () {
        const param = {
            userId: app.personal.info.userId,
            amount:this.state.amount,
            thirdpartyAccount:'N1231242311',
        };
        POST(app.route.ROUTE_RECHARGE, param, this.doRechargeSuccess, true);
    },
    doRechargeSuccess (data) {
        if (data.success) {
            Toast('充值成功');
        }
        this.props.setType(data.context.amount);
    },
    setBankCard (bankCard) {
        this.setState({ bankCard });
    },
    render () {
        const { bankCard } = this.state;
        return (
            <View style={styles.container}>
                <TextInput
                    placeholderTextColor='#444444'
                    placeholder='充值金额'
                    onChangeText={(text) => this.setState({ amount: text })}
                    style={styles.ItemBgText} />
                <TouchableOpacity
                    activeOpacity={0.6}
                    onPress={this.showBankList}
                    style={styles.ItemBg}>
                    <View style={styles.infoStyle}>
                        <Text style={styles.itemNameText}>{bankCard}</Text>
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
    ItemBgText: {
        paddingLeft:10,
        height: 44,
        flexDirection: 'row',
        alignItems: 'center',
        justifyContent: 'space-between',
        backgroundColor:'white',
        fontSize: 15,
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

* PDDriver/project/App/modules/person/SelectBankCardList.js

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
    },
    getInitialState () {
        return {
            selectedRowID:'',
            bankCard:'农业银行',
        };
    },
    toChoose (obj, rowID) {
        this.setState({
            selectedRowID:rowID,
        });
        this.props.setBankCard(this.state.bankCard);
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

* PDDriver/project/App/modules/person/Software.js

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

* PDDriver/project/App/modules/person/WithdrawCash.js

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

module.exports = React.createClass({
    statics: {
        title: '提现',
    },
    showRecharge () {
        app.push({
            component:BankCardList,
            passProps: { setBankCard: this.setCardName },
        });
    },
    getInitialState () {
        return {
            bankName: '请选择银行卡',
            amount:1,
            type:'',
            money:'0.0',
        };
    },
    doWithdraw () {
        const param = {
            userId: app.personal.info.userId,
            amount:this.state.amount,
            thirdpartyAccount:'N1231242311',
        };
        POST(app.route.ROUTE_WITHDRAW, param, this.doWithdrawSuccess, true);
    },
    doWithdrawSuccess (data) {
        if (data.success) {
            Toast('提现成功');
        }
        this.props.setType(data.context.amount);
    },
    setCardName (cardName) {
        this.setState({ bankName:cardName });
    },
    WithdrawalsAllMoney () {
        this.setState({
            amount:this.props.money,
            money:this.props.money + '',
        });
    },
    render () {
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
                        placeholderTextColor='#444444'
                        placeholder={this.state.money}
                        onChangeText={(text) => this.setState({ amount: text })}
                        style={styles.drawCashContent} />
                </View>
                <View style={styles.balance}>
                    <View style={styles.balanceLeft}>
                        <Text style={styles.balanceTextLeft}>
                            账户余额：
                        </Text>
                        <Text style={styles.balanceTextRight}>
                            {this.props.money}元
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
                    onPress={this.showRecharge}
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
        alignItems:'flex-end',
    },
    drawCashLeft:{
        fontSize:12,
        paddingLeft:12,
        paddingBottom:3,
    },
    drawCashContent:{
        flex:1,
        height:65,
        paddingBottom:3,
        fontSize:20,
        paddingTop:10,
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

* PDDriver/project/App/modules/person/index.js

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

const EditPersonInfo = require('./EditPersonInfo');
const About = require('./About');
const Feedback = require('./Feedback');
const NormalQuestions = require('./NormalQuestions');
const UpdatePage = require('../update/UpdatePage');
const Software = require('./Software');
const ModifyPassword = require('./ModifyPassword');

const { DImage } = COMPONENTS;

const CHILD_PAGES_BOTTOM = [
    { strict:true, title:'修改密码', module: ModifyPassword, img:'' },
    { strict:true, title:'软件许可协议', module: Software, img:'' },
    { strict:true, title:'常见问题', module: NormalQuestions, img:'' },
    { strict:true, title:'在线更新', module: UpdatePage, img:'' },
    { strict:true, title:'关于软件', module: About, img:'' },
    { strict:true, title:'意见反馈', module: Feedback, img:'' },
];

const MenuItem = React.createClass({
    showChildPage () {
        const { module } = this.props.page;
        app.push({
            component: module,
        });
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
                <Image
                    resizeMode='stretch'
                    source={app.img.common_go}
                    style={styles.icon_go} />
                <View style={styles.lineView} />
            </TouchableOpacity>
        );
    },
});

module.exports = React.createClass({
    statics: {
        title: '个人中心',
    },
    getInitialState () {
        const info = app.personal.info;
        return {
            info,
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
                    <Text style={styles.name}>{info.name || info.phone}</Text>
                </View>
                <View>
                    <View style={styles.lineViewTop} />
                    {
                        CHILD_PAGES_BOTTOM.map((item, i) => {
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
        marginTop: 10,
        marginBottom: 5,
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
        marginTop: 20,
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

* PDDriver/project/App/modules/position/index.js

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
    ScrollView,
    RefreshControl,
    TouchableOpacity,
} = ReactNative;

import { Geolocation } from '@remobile/react-native-baidu-map';

module.exports = React.createClass({
    statics: {
        title: '首页',
    },
    componentWillMount () {
        this.getOnWayTruck();
    },
    getOnWayTruck () {
        const param = {
            userId: app.personal.info.userId,
        };
        POST(app.route.ROUTE_GET_ON_WAY_TRUCK, param, this.getOnWayTruckSuccess, false);
    },
    getOnWayTruckSuccess (data) {
        if (data.success) {
            this.setState({
                msg:null,
                data: data.context,
                list: data.context.locationList,
                // list: _.uniqBy(data.context.locationList, 'address'),
            });
        } else {
            this.setState({ msg:data.msg });
        }
    },
    doConfirmDestination () {
        Geolocation.getCurrentPosition()
        .then(data => {
            console.warn(JSON.stringify(data));
            const param = {
                userId: app.personal.info.userId,
                address:data.address || '未知',
                latitude:data.latitude,
                longitude:data.longitude,
            };
            POST(app.route.ROUTE_CONFIRM_REACH, param, this.confirmReach, false);
        })
        .catch(e => {
            console.warn(e, 'error');
        });
    },
    confirmReach (data) {
        if (data.success) {
            Toast('确认成功');
            this.getOnWayTruck();
        } else {
            Toast(data.msg);
        }
    },
    getInitialState () {
        return {
            ds: new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 !== r2 }),
            list: [],
            data: {},
            msg:'暂无运输中的货车',
        };
    },
    renderRow (obj, sectionID, rowId) {
        return (
            <View style={styles.row}>
                <View style={styles.item}>
                    <Image
                        resizeMode='stretch'
                        source={rowId == 0 ? app.img.position_point_green : app.img.position_point}
                        style={styles.imagePoint} />
                    <View style={styles.itemRight}>
                        <Text style={rowId == 0 ? styles.addressGreen : styles.address}>{obj.address}</Text>
                        <Text style={rowId == 0 ? styles.timeGreen : styles.time}>{obj.time}</Text>
                    </View>
                </View>
            </View>
        );
    },
    onScroll () {
        this.getOnWayTruck();
    },
    render () {
        const { data, list, msg } = this.state;
        return (
            <View style={styles.container}>
                <ScrollView
                    refreshControl={
                        <RefreshControl
                            refreshing={false}
                            onRefresh={this.onScroll}
                            title='正在刷新...' />
                }>
                    {
                    !msg ?
                        <View>
                            <View style={styles.top}>
                                <Text style={styles.driverText}>司机:{app.personal.info.name}</Text>
                                <Text style={styles.carNumber}>{data.plateNo}</Text>
                                <Text style={styles.truckType}>{data.truckType}</Text>
                                <Text style={styles.totalWeight}>总重量：{data.totalWeight}/吨</Text>
                                <Text style={styles.totalNumbers}>总件数：{data.totalNumbers}/件</Text>
                            </View>
                            <ListView
                                style={styles.listView}
                                dataSource={this.state.ds.cloneWithRows(list)}
                                renderRow={this.renderRow}
                            />
                        </View>
                    :
                        <View style={styles.msg}>
                            <Image
                                resizeMode='cover'
                                defaultSource={app.img.personal_default_head}
                                source={{ uri: app.personal.info.head }}
                                style={styles.headStyle} />
                            <Text style={styles.driverText}>司机:{app.personal.info.name}</Text>
                            <Text style={styles.truckMsg}>{msg}</Text>
                        </View>
                }
                </ScrollView>
                {
                !msg &&
                <TouchableOpacity
                    onPress={this.doConfirmDestination}
                    style={styles.doConfirm}>
                    <View style={styles.confirm}>
                        <Image
                            resizeMode='stretch'
                            source={app.img.position_endpoint}
                            style={styles.imageBtn} />
                        <Text style={styles.confirmText}>确认目的地</Text>
                    </View>
                </TouchableOpacity>
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
    row:{
        height:80,
    },
    item:{
        flexDirection:'row',
    },
    top:{
        height:135,
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent:'center',
    },
    msg:{
        height:175,
        backgroundColor:'#FFFFFF',
        alignItems:'center',
        justifyContent:'center',
    },
    carNumber:{
        fontSize:23,
        marginTop:5,
        color:'#f41729',
    },
    mid:{
        flexDirection:'row',
        marginTop:5,
    },
    driverText:{
        fontSize:14,
        fontWeight:'500',
        marginTop:5,
        color:'#333333',
        letterSpacing:2,
    },
    truckMsg:{
        fontSize:16,
        marginTop:15,
        color:'#c81622',
    },
    truckType:{
        fontSize:12,
        marginTop:3,
        color:'#888888',
    },
    totalWeight:{
        fontSize:12,
        color:'#333333',
        marginTop:5,
    },
    totalNumbers:{
        fontSize:12,
        color:'#333333',
        marginTop:5,
    },
    listView:{
        marginTop:25,
        backgroundColor:'#FFFFFF',
        paddingTop:30,
    },
    imagePoint:{
        width:8,
        height:80,
        marginLeft:25,
    },
    itemRight:{
        marginLeft:15,
    },
    address:{
        fontSize:12,
        color:'#888888',
    },
    time:{
        fontSize:12,
        color:'#888888',
        marginTop:3,
    },
    addressGreen:{
        fontSize:12,
        color:'green',
    },
    timeGreen:{
        fontSize:12,
        color:'green',
        marginTop:3,
    },
    doConfirm:{
        width:140,
        height:45,
        backgroundColor:'#aaaaaa',
        opacity:0.7,
        marginLeft:(sr.w - 125) / 2,
        position:'absolute',
        bottom:0,
        borderRadius:4,
        flexDirection:'row',
        alignItems:'center',
        justifyContent:'center',
    },
    confirm:{
        flexDirection:'row',
        alignItems:'center',
    },
    imageBtn:{
        height:20,
        width:15.5,
    },
    confirmText:{
        marginLeft:8,
        fontSize:15,
        fontWeight:'500',
        color:'#FFFFFF',
    },
    headStyle: {
        width: 87,
        height: 87,
        borderRadius: 43.5,
    },
});

```

* PDDriver/project/App/modules/qrcode/index.js

```js
'use strict';

const React = require('react');
const ReactNative = require('react-native');
const {
    StyleSheet,
    View,
    Text,
} = ReactNative;

import QRCode from 'react-native-qrcode';

module.exports = React.createClass({
    statics: {
        title: '我的二维码',
    },
    render () {
        const { info } = app.personal;
        return (
            <View style={styles.container}>
                <View style={styles.card}>
                    <Text style={styles.tip}>{info.name || info.phone}</Text>
                    <View style={styles.qrcode} pointerEvents={'none'}>
                        <QRCode
                            value={app.utils.genQRString(2, info.userId)}
                            size={230}
                            bgColor='black'
                            fgColor='white' />
                    </View>
                    <Text style={styles.tip}>扫一扫二维码过安检</Text>
                </View>
            </View>
        );
    },
});

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f4f4f4',
        alignItems: 'center',
    },
    card: {
        backgroundColor: 'white',
        alignItems: 'center',
        justifyContent: 'center',
        marginTop:30,
        height:420,
        width:330,
        borderRadius:10,
    },
    qrcode: {
        borderColor:'#d6ebfe',
        borderWidth: 10,
        borderRadius:20,
        padding:20,
    },
    tip: {
        fontSize:16,
        color:'#888888',
        paddingVertical:20,
    },
});

```

* PDDriver/project/App/modules/splash/index.js

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
const Home = require('../home/index');

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
            component: Home,
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

* PDDriver/project/App/modules/test/index.js

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

* PDDriver/project/App/modules/test/pageList.js

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

* PDDriver/project/App/modules/update/UpdateInfoBox.js

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
    ERROR_NULL = 0,
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
                androidApkDownloadDestPath:'/sdcard/pddriver.apk',
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

* PDDriver/project/App/modules/update/UpdatePage.js

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
    ERROR_NULL = 0,
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
                androidApkDownloadDestPath:'/sdcard/pddriver.apk',
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

* PDDriver/project/App/utils/check/index.js

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
};

```

* PDDriver/project/App/utils/date/index.js

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

* PDDriver/project/App/utils/common/index.js

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

* PDDriver/project/App/utils/net/Get.js

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

* PDDriver/project/App/utils/net/Post.js

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

* PDDriver/project/App/utils/net/Upload.js

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

* PDDriver/project/App/utils/net/index.js

```js
'use strict';

module.exports = {
    POST: require('./Post'),
    GET: require('./Get'),
    UPLOAD: require('./Upload'),
};

```

* PDDriver/project/App/utils/qrcode/index.js

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
