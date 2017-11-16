* PDShop_Shop_PC/project/App/client/auth.js

```js
import routes from 'routers/auth';
import renderRoutes from './helpers/render-routes';

renderRoutes(routes);

```

* PDShop_Shop_PC/project/App/client/public.js

```js
import routes from 'routers/public';
import renderRoutes from './helpers/render-routes';

renderRoutes(routes);

```

* PDShop_Shop_PC/project/App/client/shop.js

```js
import 'babel-polyfill';

import routes from 'routers/shop';
import renderRoutes from './helpers/render-routes';

renderRoutes(routes);

```

* PDShop_Shop_PC/project/App/server/index.js

```js
import bodyParser from 'body-parser';
import express from 'express';
import graphqlHTTP from 'express-graphql';
import morgan from 'morgan';
import path from 'path';
import passport from 'passport';
import { Strategy as LocalStrategy } from 'passport-local';
import connectMongo from 'connect-mongo';
import session from 'express-session';

import { post, urls } from 'helpers/api';
import config from '../../config';
import routers from './routers';
import schema from './schema';

const app = express();

app.use(morgan('short'));
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json({ limit: 100000000 }));

// session
const MongoStore = connectMongo(session);
app.use(session({
    secret: '88164657simiantong_shop', // session的密码
    resave: true,
    saveUninitialized: false,
    store: new MongoStore({
        url: config.dbServer,
        collection: 'sessions_shop',
    }),
}));

// Passport
app.use(passport.initialize());
app.use(passport.session());
passport.use('local', new LocalStrategy({
    usernameField: 'phone',
}, async (phone, password, done) => {
    const user = { phone, password };
    console.log('[login]: LocalStrategy', user);
    const data = await post(urls.login, user) || { msg: '服务器错误' };
    if (!data.success) {
        return done({ message: data.msg });
    }

    return done(null, Object.assign({}, data.context));
}));
passport.serializeUser((user, done) => { // 保存 user 到 sessoon
    console.log('[login]: serializeUser', user, done);
    done(null, user);// 可以通过数据库方式操作
});
passport.deserializeUser((user, done) => { // 通过保存的 user 信息到 sessoon 获取 user
    console.log('[login]: deserializeUser', user, done);
    done(null, user);// 可以通过数据库方式操作
});

// Static files
app.use(express.static(path.resolve('public')));
app.use(['/shop/favicon.ico', '/shop/images*', '/shop/media*', '/shop/css*', '/shop/fonts*', '/shop/assets*'], (req, res) => {
    res.status(404).end();
});

// GraphqQL server
app.use('/shop/graphql', graphqlHTTP(req => ({
    schema: schema.getSchema(),
    rootValue: {
        isAuthenticated: req.isAuthenticated(),
        user: req.user,
        req: req,
    },
    graphiql: true,
})));

app.use(async (req, res, next) => {
    res.locals.header = [
        {
            tag: 'title',
            content: '四面通物流大超市行政端',
        },
    ];

    if (process.env.NODE_ENV !== 'production') {
        res.baseScriptsURL = `http://localhost:${config.devPort}`;
        res.locals.header.push({
            tag: 'script',
            props: {
                src: `${res.baseScriptsURL}/webpack-dev-server.js`,
            },
        });
    } else {
        res.baseScriptsURL = '';
    }

    // footer
    res.locals.footer = [{
        tag: 'script',
        props: {
            src: `${res.baseScriptsURL}/shop/assets/common.js`,
        },
    }, {
        tag: 'script',
        props: {
            src: `http://api.map.baidu.com/api?v=2.0&ak=${config.baiduMapSK}`,
        },
    } ];

    next();
});

app.use(routers.authRouter);
app.use(routers.shopRouter);
app.use(routers.publicRouter);

app.use((req, res) => {
    res.status(404).end();
});

app.use((error, req, res) => {
    const statusCode = error.statusCode || 500;
    const err = {
        error: statusCode,
        message: error.message,
    };
    if (!res.headersSent) {
        res.status(statusCode).send(err);
    }
});

export default app;

```

* PDShop_Shop_PC/project/App/server/schema.js

```js
import _ from 'lodash';
import {
    GraphQLObjectType,
    GraphQLSchema,
} from 'graphql';

import mutations from './graphql/mutations';
import queries from './graphql/queries';

class SchemaManager {
    constructor () {
        this.init();
    }

    async init () {
        this.queryFields = _.clone(queries);
        this.mutationFields = _.clone(mutations);

        this.rootQuery = new GraphQLObjectType({
            name: 'Query',
            fields: () => (this.queryFields),
        });
        this.rootMutation = new GraphQLObjectType({
            name: 'Mutation',
            fields: () => (this.mutationFields),
        });
    }

    getSchema () {
        const schema = {
            query: this.rootQuery,
        };

        if (Object.keys(this.mutationFields).length) {
            schema.mutation = this.rootMutation;
        }

        return new GraphQLSchema(schema);
    }
}

export default new SchemaManager();

```

* PDShop_Shop_PC/project/App/client/helpers/render-routes.js

```js
import configureStore from 'helpers/configure-store';
import createHistory from 'history/lib/createBrowserHistory';
import React from 'react';
import { render } from 'react-dom';
import { Provider } from 'react-redux';
import { reduxReactRouter, ReduxRouter } from 'redux-router';
import getMuiTheme from 'material-ui/styles/getMuiTheme';
import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';
import injectTapEventPlugin from 'react-tap-event-plugin';

injectTapEventPlugin();

export default function renderRoutes (routes) {
    const state = window.__initialState;
    state.router = undefined;
    const store = configureStore(
        reduxReactRouter({ createHistory, routes }),
        state
    );
    const muiTheme = getMuiTheme({ userAgent: navigator.userAgent });

    render(
        <MuiThemeProvider muiTheme={muiTheme}>
            <Provider store={store}>
                <ReduxRouter routes={routes} />
            </Provider>
        </MuiThemeProvider>,
        document.getElementById('view')
    );
}

```

* PDShop_Shop_PC/project/App/server/graphql/authorize.js

```js
export function authorize (root) {
    if (!root.user) {
        throw new Error('unauthorized');
    }
}

```

* PDShop_Shop_PC/project/App/server/routers/auth.js

```js
import getDefaultFavicon from 'helpers/default-favicon';
import getMarkup from 'helpers/get-markup';
import routeHandler from 'helpers/route-handler';
import routes from 'routers/auth';
import { Router } from 'express';

const authRouter = new Router();

function injectScript (req, res, next) {
    if (process.env.NODE_ENV === 'production') {
        res.locals.header.push({
            tag: 'link',
            props: {
                rel: 'stylesheet',
                type: 'text/css',
                href: '/shop/assets/auth.css',
            },
        });
        // res.locals.header.push({
        //     tag: 'link',
        //     props: {
        //         rel: 'stylesheet',
        //         type: 'text/css',
        //         href: '/shop/assets/common.js.css',
        //     },
        // });
    }
    res.locals.header.push(getDefaultFavicon(res));
    res.locals.footer.push({
        tag: 'script',
        props: {
            src: `${res.baseScriptsURL}/shop/assets/auth.js`,
        },
    });
    next();
}

authRouter.get('/shop/login', (req, res, next) => {
    if (req.isAuthenticated()) {
        res.redirect('/shop');
    } else {
        routeHandler(routes, req, res, next);
    }
});

authRouter.get('/shop/logout', (req, res) => {
    req.logout();
    res.redirect('/shop/login');
});

authRouter.get(/^\/shop\/(register|forgotPwd)$/, (req, res, next) => {
    routeHandler(routes, req, res, next);
});

// Register | ForgotPwd
authRouter.get(/^\/shop\/(register|forgotPwd)$/, injectScript, async (req, res, next) => {
    res.status(200).send(getMarkup(req, res));
});

// Login
authRouter.get('/shop/login', injectScript, (req, res) => {
    if (req.isAuthenticated()) {
        res.redirect('/shop');
    } else {
        res.status(200).send(getMarkup(req, res));
    }
});

export default authRouter;

```

* PDShop_Shop_PC/project/App/server/routers/index.js

```js
import shopRouter from './shop';
import authRouter from './auth';
import publicRouter from './public';

export default {
    shopRouter,
    authRouter,
    publicRouter,
};

```

* PDShop_Shop_PC/project/App/server/routers/public.js

```js
import { Router } from 'express';
import request from 'request';
import { urls } from 'helpers/api';

const publicRouter = new Router();

publicRouter.post('/api/uploadFile', (req, res) => {
    req.pipe(request(urls.uploadFile)).pipe(res);
});

export default publicRouter;

```

* PDShop_Shop_PC/project/App/server/routers/shop.js

```js
import getBaseComponent from 'helpers/get-base-component';
import getDefaultFavicon from 'helpers/default-favicon';
import renderHtml from 'helpers/render-html';
import routeHandler from 'helpers/route-handler';
import routes from 'routers/shop';
import { Router } from 'express';
import { graphql } from 'graphql';
import { getDataDependencies } from 'relatejs';

import schema from '../schema';

const shopRouter = new Router();

shopRouter.get(/^\/shop\/.+/, (req, res, next) => {
    if (req.isAuthenticated()) {
        res.redirect('/shop');
    } else {
        next();
    }
});

shopRouter.get('/shop*', (req, res, next) => {
    if (req.isAuthenticated()) {
        if (process.env.NODE_ENV === 'production') {
            res.locals.header.push(getDefaultFavicon(res));
            res.locals.header.push({
                tag: 'link',
                props: {
                    rel: 'stylesheet',
                    type: 'text/css',
                    href: '/shop/assets/shop.css',
                },
            });
            // res.locals.header.push({
            //     tag: 'link',
            //     props: {
            //         rel: 'stylesheet',
            //         type: 'text/css',
            //         href: '/shop/assets/common.js.css',
            //     },
            // });
        }
        res.locals.footer.push({
            tag: 'script',
            props: {
                src: `${res.baseScriptsURL}/shop/assets/shop.js`,
            },
        });
        next();
    } else {
        res.redirect('/shop/login');
    }
});

shopRouter.get('/shop*', (req, res, next) => {
    if (req.isAuthenticated()) {
        routeHandler(routes, req, res, next);
    } else {
        next();
    }
});

shopRouter.get('/shop*', async (req, res, next) => {
    if (req.isAuthenticated() && req.routerState) {
        // get component with redux provider and react router
        const component = getBaseComponent(req);
        // get relate js data dependencies
        await getDataDependencies(component, async (request) => await graphql(
            schema.getSchema(),
            request.query,
            {
                isAuthenticated: true,
                user: req.user,
            },
            request.variables
        ));

        // final render html
        res.status(200).send(renderHtml({
            component,
            store: req.store,
            locals: res.locals,
        }));
    } else {
        next();
    }
});

export default shopRouter;

```

* PDShop_Shop_PC/project/App/shared/actions/accounts.js

```js
import { apiQuery } from 'relatejs';

export function weixinPayGetUnifiedOrder (data, callback) {
    return () => {
        return apiQuery({
            fragments: {
                weixinPayGetUnifiedOrder: { success: 1, msg: 1, context: { qrcode: 1 } },
            },
            variables: {
                weixinPayGetUnifiedOrder: {
                    data: {
                        value: data,
                        type: 'accountInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.weixinPayGetUnifiedOrder);
        })();
    };
}
export function weixinPayWithDraw (data, callback) {
    return () => {
        return apiQuery({
            fragments: {
                weixinPayWithDraw: { success: 1, msg: 1, context: { qrcode: 1 } },
            },
            variables: {
                weixinPayWithDraw: {
                    data: {
                        value: data,
                        type: 'accountInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.weixinPayWithDraw);
        })();
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/agents.js

```js
import { mutation } from 'relatejs';

export function checkAgentByChairManPhone (chairManPhone, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                checkAgentByChairManPhone: { success: 1, msg: 1, context },
            },
            variables: {
                checkAgentByChairManPhone: {
                    chairManPhone: {
                        value: chairManPhone,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.checkAgentByChairManPhone);
        })(dispatch, getState);
    };
}
export function createAgent (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                createAgent: { success: 1, msg: 1, context },
            },
            variables: {
                createAgent: {
                    data: {
                        value: data,
                        type: 'agentInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.createAgent);
        })(dispatch, getState);
    };
}
export function modifyAgent (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyAgent: { success: 1, msg: 1, context },
            },
            variables: {
                modifyAgent: {
                    data: {
                        value: data,
                        type: 'agentInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyAgent);
        })(dispatch, getState);
    };
}
export function removeAgent (agentId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removeAgent: { success: 1, msg: 1 },
            },
            variables: {
                removeAgent: {
                    agentId: {
                        value: agentId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.removeAgent);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/clients.js

```js
import { apiQuery } from 'relatejs';

export function getClientNameByPhone (phone, callback) {
    return () => {
        return apiQuery({
            fragments: {
                getClientNameByPhone: { success: 1, msg: 1, context: { name: 1 } },
            },
            variables: {
                getClientNameByPhone: {
                    phone: {
                        value: phone,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.getClientNameByPhone);
        })();
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/members.js

```js
import { mutation } from 'relatejs';

export function createMember (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                createMember: { success: 1, msg: 1, context },
            },
            variables: {
                createMember: {
                    data: {
                        value: data,
                        type: 'memberInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.createMember);
        })(dispatch, getState);
    };
}
export function modifyMember (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyMember: { success: 1, msg: 1, context },
            },
            variables: {
                modifyMember: {
                    data: {
                        value: data,
                        type: 'memberInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyMember);
        })(dispatch, getState);
    };
}
export function removeMember (memberId, password, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removeMember: { success: 1, msg: 1 },
            },
            variables: {
                removeMember: {
                    memberId: {
                        value: memberId,
                        type: 'ID!',
                    },
                    password: {
                        value: password,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.removeMember);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/notifies.js

```js
import { mutation } from 'relatejs';

export function sendNotify (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                sendNotify: { success: 1, msg: 1, context },
            },
            variables: {
                sendNotify: {
                    data: {
                        value: data,
                        type: 'notifyInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.sendNotify);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/orders.js

```js
import { mutation } from 'relatejs';

export function placeOriginOrder (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                placeOriginOrder: { success: 1, msg: 1, context },
            },
            variables: {
                placeOriginOrder: {
                    data: {
                        value: data,
                        type: 'orderInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.placeOriginOrder);
        })(dispatch, getState);
    };
}
export function placeOriginOrderWithPreOrder (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                placeOriginOrderWithPreOrder: { success: 1, msg: 1, context },
            },
            variables: {
                placeOriginOrderWithPreOrder: {
                    data: {
                        value: data,
                        type: 'orderInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.placeOriginOrderWithPreOrder);
        })(dispatch, getState);
    };
}
export function modifyOriginOrder (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyOriginOrder: { success: 1, msg: 1, context },
            },
            variables: {
                modifyOriginOrder: {
                    data: {
                        value: data,
                        type: 'orderInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyOriginOrder);
        })(dispatch, getState);
    };
}
export function removeOrder (orderId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removeOrder: { success: 1, msg: 1 },
            },
            variables: {
                removeOrder: {
                    orderId: {
                        value: orderId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.removeOrder);
        })(dispatch, getState);
    };
}

export function selectRoadmap (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                selectRoadmap: { success: 1, msg: 1, context },
            },
            variables: {
                selectRoadmap: {
                    data: {
                        value: data,
                        type: 'orderInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.selectRoadmap);
        })(dispatch, getState);
    };
}
export function selectRoadmapForOrderList (orderIdList, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                selectRoadmapForOrderList: { success: 1, msg: 1 },
            },
            variables: {
                selectRoadmapForOrderList: {
                    orderIdList: {
                        value: orderIdList,
                        type: '[ID]!',
                    },
                },
            },
        }, (result) => {
            callback(result.selectRoadmapForOrderList);
        })(dispatch, getState);
    };
}
export function proxyPay (orderId, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                proxyPay: { success: 1, msg: 1, context },
            },
            variables: {
                proxyPay: {
                    orderId: {
                        value: orderId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.proxyPay);
        })(dispatch, getState);
    };
}
export function proxyPayForOrderList (orderIdList, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                proxyPayForOrderList: { success: 1, msg: 1 },
            },
            variables: {
                proxyPayForOrderList: {
                    orderIdList: {
                        value: orderIdList,
                        type: '[ID]!',
                    },
                },
            },
        }, (result) => {
            callback(result.proxyPayForOrderList);
        })(dispatch, getState);
    };
}
export function printOrderBill (orderId, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                printOrderBill: { success: 1, msg: 1, context },
            },
            variables: {
                printOrderBill: {
                    orderId: {
                        value: orderId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.printOrderBill);
        })(dispatch, getState);
    };
}
export function printOrderListBill (orderIdList, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                printOrderListBill: { success: 1, msg: 1 },
            },
            variables: {
                printOrderListBill: {
                    orderIdList: {
                        value: orderIdList,
                        type: '[ID]!',
                    },
                },
            },
        }, (result) => {
            callback(result.printOrderListBill);
        })(dispatch, getState);
    };
}
export function printBarCode (orderId, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                printBarCode: { success: 1, msg: 1, context },
            },
            variables: {
                printBarCode: {
                    orderId: {
                        value: orderId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.printBarCode);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/partments.js

```js
import { mutation } from 'relatejs';

export function createPartment (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                createPartment: { success: 1, msg: 1, context },
            },
            variables: {
                createPartment: {
                    data: {
                        value: data,
                        type: 'partmentInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.createPartment);
        })(dispatch, getState);
    };
}
export function modifyPartment (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyPartment: { success: 1, msg: 1, context },
            },
            variables: {
                modifyPartment: {
                    data: {
                        value: data,
                        type: 'partmentInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyPartment);
        })(dispatch, getState);
    };
}
export function removePartment (partmentId, password, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removePartment: { success: 1, msg: 1 },
            },
            variables: {
                removePartment: {
                    partmentId: {
                        value: partmentId,
                        type: 'ID!',
                    },
                    password: {
                        value: password,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.removePartment);
        })(dispatch, getState);
    };
}
export function modifyOwnPartment (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyOwnPartment: { success: 1, msg: 1, context },
            },
            variables: {
                modifyOwnPartment: {
                    data: {
                        value: data,
                        type: 'partmentInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyOwnPartment);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/personals.js

```js
import { apiQuery, apiMutation, mutation } from 'relatejs';

export function getVerifyCode (phone, callback) {
    return () => {
        return apiQuery({
            fragments: {
                getVerifyCode: { success: 1, msg: 1 },
            },
            variables: {
                getVerifyCode: {
                    phone: {
                        value: phone,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.getVerifyCode);
        })();
    };
}
export function register (data, callback) {
    return () => {
        return apiMutation({
            fragments: {
                register: { success: 1, msg: 1 },
            },
            variables: {
                register: {
                    data: {
                        value: data,
                        type: 'memberInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.register);
        })();
    };
}
export function login (phone, password, callback) {
    return () => {
        return apiQuery({
            fragments: {
                login: { success: 1, msg: 1 },
            },
            variables: {
                login: {
                    phone: {
                        value: phone,
                        type: 'String!',
                    },
                    password: {
                        value: password,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.login);
        })();
    };
}
export function forgotPwd (data, callback) {
    return () => {
        return apiMutation({
            fragments: {
                forgotPwd: { success: 1, msg: 1 },
            },
            variables: {
                forgotPwd: {
                    data: {
                        value: data,
                        type: 'memberInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.forgotPwd);
        })();
    };
}
export function setPaymentPassword (data, callback) {
    return () => {
        return apiMutation({
            fragments: {
                setPaymentPassword: { success: 1, msg: 1 },
            },
            variables: {
                setPaymentPassword: {
                    data: {
                        value: data,
                        type: 'memberInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.setPaymentPassword);
        })();
    };
}
export function feedback (content, email, callback) {
    return () => {
        return apiMutation({
            fragments: {
                feedback: { success: 1, msg: 1 },
            },
            variables: {
                feedback: {
                    content: {
                        value: content,
                        type: 'String!',
                    },
                    email: {
                        value: email,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.feedback);
        })();
    };
}
export function modifyPersonalInfo (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyPersonalInfo: { success: 1, msg: 1, context },
            },
            variables: {
                modifyPersonalInfo: {
                    data: {
                        value: data,
                        type: 'memberInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyPersonalInfo);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/roadmaps.js

```js
import { mutation } from 'relatejs';

export function setRoadmapProfit (data, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setRoadmapProfit: { success: 1, msg: 1 },
            },
            variables: {
                setRoadmapProfit: {
                    data: {
                        value: data,
                        type: 'roadmapInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.setRoadmapProfit);
        })(dispatch, getState);
    };
}
export function setRegionRateProfit (data, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setRegionRateProfit: { success: 1, msg: 1 },
            },
            variables: {
                setRegionRateProfit: {
                    data: {
                        value: data,
                        type: 'regionRateInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.setRegionRateProfit);
        })(dispatch, getState);
    };
}
export function addRoadmapRegionProfitRate (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                addRoadmapRegionProfitRate: { success: 1, msg: 1, context },
            },
            variables: {
                addRoadmapRegionProfitRate: {
                    data: {
                        value: data,
                        type: 'regionRateInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.addRoadmapRegionProfitRate);
        })(dispatch, getState);
    };
}
export function modifyRoadmapRegionProfitRate (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyRoadmapRegionProfitRate: { success: 1, msg: 1, context },
            },
            variables: {
                modifyRoadmapRegionProfitRate: {
                    data: {
                        value: data,
                        type: 'regionRateInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyRoadmapRegionProfitRate);
        })(dispatch, getState);
    };
}
export function removeRoadmapRegionProfitRate (regionRateId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removeRoadmapRegionProfitRate: { success: 1, msg: 1 },
            },
            variables: {
                removeRoadmapRegionProfitRate: {
                    regionRateId: {
                        value: regionRateId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.removeRoadmapRegionProfitRate);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/settings.js

```js
import { mutation } from 'relatejs';

export function modifySetting (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifySetting: { success: 1, msg: 1, context },
            },
            variables: {
                modifySetting: {
                    data: {
                        value: data,
                        type: 'settingInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifySetting);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/shippers.js

```js
import { mutation, apiQuery } from 'relatejs';

export function modifyShipper (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyShipper: { success: 1, msg: 1, context },
            },
            variables: {
                modifyShipper: {
                    data: {
                        value: data,
                        type: 'shipperInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyShipper);
        })(dispatch, getState);
    };
}
export function createAndRegisterShipper (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                createAndRegisterShipper: { success: 1, msg: 1, context },
            },
            variables: {
                createAndRegisterShipper: {
                    data: {
                        value: data,
                        type: 'shipperInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.createAndRegisterShipper);
        })(dispatch, getState);
    };
}
export function registerShipper (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                registerShipper: { success: 1, msg: 1, context },
            },
            variables: {
                registerShipper: {
                    data: {
                        value: data,
                        type: 'shipperInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.registerShipper);
        })(dispatch, getState);
    };
}
export function getShipperByChairManPhone (chairManPhone, context, callback) {
    return () => {
        return apiQuery({
            fragments: {
                getShipperByChairManPhone: { success: 1, msg: 1, context },
            },
            variables: {
                getShipperByChairManPhone: {
                    chairManPhone: {
                        value: chairManPhone,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.getShipperByChairManPhone);
        })();
    };
}
export function passExamineTruck (truckId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                passExamineTruck: { success: 1, msg: 1 },
            },
            variables: {
                passExamineTruck: {
                    truckId: {
                        value: truckId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.passExamineTruck);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/shops.js

```js
import { mutation } from 'relatejs';

export function modifyOwnShop (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyOwnShop: { success: 1, msg: 1, context },
            },
            variables: {
                modifyOwnShop: {
                    data: {
                        value: data,
                        type: 'shopInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyOwnShop);
        })(dispatch, getState);
    };
}

export function createBranchShop (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                createBranchShop: { success: 1, msg: 1, context },
            },
            variables: {
                createBranchShop: {
                    data: {
                        value: data,
                        type: 'shopInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.createBranchShop);
        })(dispatch, getState);
    };
}
export function modifyBranchShop (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyBranchShop: { success: 1, msg: 1, context },
            },
            variables: {
                modifyBranchShop: {
                    data: {
                        value: data,
                        type: 'shopInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyBranchShop);
        })(dispatch, getState);
    };
}
export function removeBranchShop (shopId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removeBranchShop: { success: 1, msg: 1 },
            },
            variables: {
                removeBranchShop: {
                    shopId: {
                        value: shopId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.removeBranchShop);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/actions/warehouses.js

```js
import { mutation } from 'relatejs';

export function createWarehouse (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                createWarehouse: { success: 1, msg: 1, context },
            },
            variables: {
                createWarehouse: {
                    data: {
                        value: data,
                        type: 'warehouseInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.createWarehouse);
        })(dispatch, getState);
    };
}
export function modifyWarehouse (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyWarehouse: { success: 1, msg: 1, context },
            },
            variables: {
                modifyWarehouse: {
                    data: {
                        value: data,
                        type: 'warehouseInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyWarehouse);
        })(dispatch, getState);
    };
}
export function removeWarehouse (regionRateId, password, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removeWarehouse: { success: 1, msg: 1 },
            },
            variables: {
                removeWarehouse: {
                    warehouseId: {
                        value: regionRateId,
                        type: 'ID!',
                    },
                    password:{
                        value: password,
                        type:'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.removeWarehouse);
        })(dispatch, getState);
    };
}

```

* PDShop_Shop_PC/project/App/shared/config/menu.js

```js
import AH from '../constants/authority';
import _ from 'lodash';

export default [
    {
        label: '部门',
        link: '/shop/partments',
        auth: [AH.AH_LOOK_PARTMENT, AH.AH_CREATE_PARTMENT, AH.AH_MODIFY_PARTMENT, AH.AH_REMOVE_PARTMENT],
    },
    {
        label: '分店',
        link: '/shop/branchShops',
        auth: [AH.AH_LOOK_BRANCH_SHOP, AH.AH_CREATE_BRANCH_SHOP, AH.AH_MODIFY_BRANCH_SHOP, AH.AH_REMOVE_BRANCH_SHOP],
    },
    {
        label: '收货点',
        link: '/shop/agents',
        auth: [AH.AH_LOOK_AGENT, AH.AH_CREATE_AGENT, AH.AH_MODIFY_AGENT, AH.AH_REMOVE_AGENT],
    },
    {
        label: '参数设置',
        link: '/shop/setting',
        auth: [AH.AH_MODIFY_SETTING],
    },
    {
        label: '物流公司',
        link: '/shop/shippers',
        auth: [AH.AH_LOOK_SHIPPER, AH.AH_CREATE_SHIPPER, AH.AH_MODIFY_SHIPPER, AH.AH_REMOVE_SHIPPER],
    },
    {
        label: '路线提成',
        link: '/shop/roadmaps',
        auth: [AH.AH_MODIFY_ROADMAP_PROFIT, AH.AH_LOOK_ROADMAP],
    },
    {
        label: '方向提成',
        link: '/shop/regionRates',
        auth: [AH.AH_MODIFY_ROADMAP_PROFIT],
    },
    {
        label: '货车审核',
        link: '/shop/trucks',
        auth: [AH.AH_TO_EXAMINE_TRUCK],
    },
    {
        label: '仓库管理',
        link: '/shop/warehouses',
        auth: [AH.AH_CREATE_WAREHOUSE, AH.AH_MODIFY_WAREHOUSE, AH.AH_REMOVE_WAREHOUSE, AH.AH_LOOK_WAREHOUSE],
    },
    {
        label: '货单',
        link: '/shop/orders',
        auth: member => !member.partment && _.includes(member.authority, AH.AH_LOOK_ORDERS),
    },
    {
        label: '下单',
        link: '/shop/placeOrder',
        auth: [AH.AH_RECEIVE_PARTMENT],
    },
    {
        label: '货单',
        link: '/shop/rporders',
        auth: [AH.AH_RECEIVE_PARTMENT],
    },
    {
        label: '货单',
        link: '/shop/whorders',
        auth: [AH.AH_WARE_HOUSE_PARTMENT],
    },
    {
        label: '客户',
        link: '/shop/clients',
        auth: false,
    },
    {
        label: '业务统计',
        link: '/shop/statistics',
        auth: member => !member.partment && _.includes(member.authority, AH.AH_LOOK_STATISTICS),
    },
];

```

* PDShop_Shop_PC/project/App/shared/config/topmenu.js

```js
import AH from '../constants/authority';

export default [
    {
        label: '人员管理',
        link: '/shop/members',
        auth: [AH.AH_LOOK_MEMBER, AH.AH_CREATE_MEMBER, AH.AH_MODIFY_MEMBER, AH.AH_REMOVE_MEMBER],
    },
    {
        label: '个人中心',
        link: '/shop/personal',
    },
    {
        label: '意见反馈',
        link: '/shop/feedback',
    },
    {
        label: '退出',
        link: '/shop/logout',
    },
];

```

* PDShop_Shop_PC/project/App/shared/constants/authority.js

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
const AH_WITHDRAW = 10006; // 提现的权限
const AH_LOOK_AMOUNT = 10007; // 查看余额的权限
const AH_MODIFY_ROADMAP_PROFIT = 10008; // 修改路线的提成
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
const AH_CREATE_WAREHOUSE = 20015; // 创建仓库的权限
const AH_MODIFY_WAREHOUSE = 20016; // 修改仓库信息的权限
const AH_REMOVE_WAREHOUSE = 20017; // 删除仓库的权限
const AH_LOOK_WAREHOUSE = 20018; // 查看仓库的权限

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

// 收货点
const AH_MODIFY_AGENT_INFO = 40000; // 修改收货点信息的权限
const AH_MODIFY_AGENT_MEMBER_AUTHORITY = 40001; // 修改收货点成员权限的权限
const AH_PLACE_ORDER = 40002; // 收货点下单的权限

export default {
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
    AH_MODIFY_ROADMAP_PROFIT,
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
    AH_CREATE_WAREHOUSE,
    AH_MODIFY_WAREHOUSE,
    AH_REMOVE_WAREHOUSE,
    AH_LOOK_WAREHOUSE,

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
        [AH_MODIFY_ROADMAP_PROFIT]: '修改路线的提成的权限',
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
        [AH_CREATE_WAREHOUSE]: '创建仓库的权限',
        [AH_MODIFY_WAREHOUSE]: '修改仓库信息的权限',
        [AH_REMOVE_WAREHOUSE]: '删除仓库的权限',
        [AH_LOOK_WAREHOUSE]: '查看仓库的权限',

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

        // 收货点
        [AH_MODIFY_AGENT_INFO]: '修改收货点信息的权限',
        [AH_MODIFY_AGENT_MEMBER_AUTHORITY]: '修改收货点成员权限的权限',
        [AH_PLACE_ORDER]: '收货点下单的权限',
    },
};

```

* PDShop_Shop_PC/project/App/shared/constants/common.js

```js
// 注意：常量部分必须和服务器保持一致

const ST_LONG = 0; // 长途物流公司
const ST_CITY = 1; // 同城配送物流公司

// 方向设置的类型
export const RRT_ALL = 0; // 所有
export const RRT_LONG = 1; // 长途
export const RRT_CITY = 2; // 同城配送

export default {
    ST_LONG,
    ST_CITY,

    RRT_ALL,
    RRT_LONG,
    RRT_CITY,

    // mapper
    SHIPPER_TYPE_MAP: {
        [ST_LONG]: '长途运输',
        [ST_CITY]: ' 同城配送',
    },

    RRT_MAP: {
        [RRT_ALL]: '长途 & 同城',
        [RRT_LONG]: ' 长途运输',
        [RRT_CITY]: ' 同城配送',
    },
};

```

* PDShop_Shop_PC/project/App/shared/constants/index.js

```js
export AH from './authority';
export PO from './post';
export SP from './partment';
export OS from './orderState';
export CO from './common';

```

* PDShop_Shop_PC/project/App/shared/constants/orderState.js

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
const OS_READY_STOCK = 5; // 待入库
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

export default {
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
        [OS_READY_STOCK]: '待入库',
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

* PDShop_Shop_PC/project/App/shared/constants/partment.js

```js
// 注意：常量部分必须和服务器保持一致
const SP_RECEIVE_PARTMENT = 0; // 收货部
const SP_WARE_HOUSE_PARTMENT = 1; // 库管部
const SP_CARRY_PARTMENT = 2; // 搬运部
const SP_SECURITY_CHECK_PARTMENT = 3; // 保安部

export default {
    SP_RECEIVE_PARTMENT,
    SP_WARE_HOUSE_PARTMENT,
    SP_CARRY_PARTMENT,
    SP_SECURITY_CHECK_PARTMENT,

    // mapper
    MAP: {
        [SP_RECEIVE_PARTMENT]: '收货部',
        [SP_WARE_HOUSE_PARTMENT]: '库管部',
        [SP_CARRY_PARTMENT]: '搬运部',
        [SP_SECURITY_CHECK_PARTMENT]: '保安部',
    },
};

```

* PDShop_Shop_PC/project/App/shared/constants/post.js

```js
const PO_CHAIR_MAN = '董事长';
const PO_PARTMENT_CHARGE_MAN = '部门经理';
const PO_CEO = '总经理';
const PO_PARTMENT_LEADER = '部门主管';
const PO_STAFF = '员工';

export default {
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

* PDShop_Shop_PC/project/App/shared/decorators/antd_form_create.js

```js
import { Form } from 'antd';
export default (target) => {
    const Component = Form.create()(target);
    Component.fragments = target.fragments;
    return Component;
};

```

* PDShop_Shop_PC/project/App/shared/helpers/configure-store.js

```js
import combineActionsMiddleware from 'redux-combine-actions';
import createLogger from 'redux-logger';
import reducer from 'reducers';
import thunkMiddleware from 'redux-thunk';
import { createStore, applyMiddleware, compose } from 'redux';

const middleware = [];

middleware.push(combineActionsMiddleware);
middleware.push(thunkMiddleware);

if (typeof window !== 'undefined' && module.hot) {
    middleware.push(createLogger());
}

export default function configureStore (routerMiddleware, initialState) {
    const store = compose(
    applyMiddleware(
      ...middleware
    ),
    routerMiddleware
  )(createStore)(reducer, initialState);

    if (module.hot) {
    // Enable Webpack hot module replacement for reducers
        module.hot.accept('../reducers', () => {
            const nextReducer = require('../reducers');
            store.replaceReducer(nextReducer);
        });
    }

    return store;
}

```

* PDShop_Shop_PC/project/App/shared/helpers/ga-send.js

```js
export default function gaSend () {
    if (typeof window !== 'undefined') {
        window.ga && window.ga('send', 'pageview');
    }
}

```

* PDShop_Shop_PC/project/App/shared/reducers/index.js

```js
import { combineReducers } from 'redux';
import { routerStateReducer as router } from 'redux-router';
import { relateReducer } from 'relatejs';

export const reducersToCombine = {
    relateReducer,
    router,
};
const rootReducer = combineReducers(reducersToCombine);

export default rootReducer;

```

* PDShop_Shop_PC/project/App/shared/relatejs/index.js

```js
import { mergeFragments } from './helpers/fragments';

import actionTypes from './actions/types';
import capture from './helpers/capture';
import dataConnect from './decorators/data-connect';
import getDataDependencies from './server/get-data-dependencies';
import mutation from './actions/mutation';
import rootDataConnect from './decorators/root-data-connect';
import { setHeader, removeHeader, setEndpoint, setBody, removeBody } from './actions/settings';
import { relateReducer, relateReducerInit } from './reducer/reducer';
import { apiQuery, apiMutation } from './redirect-graphql';
import KeepData from './store/keep-data';
import GlobalData from './store/global-data';

export {
    dataConnect,
    rootDataConnect,
    relateReducer,
    relateReducerInit,
    actionTypes,
    mutation,
    mergeFragments,
    setHeader,
    removeHeader,
    setEndpoint,
    setBody,
    removeBody,
    getDataDependencies,
    capture,
    apiQuery,
    apiMutation,
    KeepData,
    GlobalData,
};

export default {
    dataConnect,
    rootDataConnect,
    relateReducer,
    relateReducerInit,
    actionTypes,
    mutation,
    mergeFragments,
    setHeader,
    removeHeader,
    setEndpoint,
    setBody,
    removeBody,
    getDataDependencies,
    capture,
    apiQuery,
    apiMutation,
    KeepData,
    GlobalData,
};

```

* PDShop_Shop_PC/project/App/shared/routers/auth.js

```js
import React from 'react';
import { Route } from 'react-router';
import gaSend from 'helpers/ga-send';
import Auth from 'pages/auth';
import Login from 'pages/auth/pages/login';
import ForgotPwd from 'pages/auth/pages/forgotPwd';

export default [
    <Route component={Auth}>
        <Route path='/shop/login' component={Login} onEnter={gaSend} />
        <Route path='/shop/forgotPwd' component={ForgotPwd} onEnter={gaSend} />
    </Route>,
];

```

* PDShop_Shop_PC/project/App/shared/routers/public.js

```js
// import React from 'react';
// import {Route} from 'react-router';

export default [];

```

* PDShop_Shop_PC/project/App/shared/routers/shop.js

```js
import React from 'react';
import { Route, IndexRoute } from 'react-router';
import request from 'superagent';
import Shop from 'pages/shop';
import Home from 'pages/shop/pages/home';
import Personal from 'pages/shop/pages/personal';
import PaymentPwd from 'pages/shop/pages/personal/pages/paymentPwd';
import OwnShop from 'pages/shop/pages/ownShop';
import Partment from 'pages/shop/pages/partment';
import Partments from 'pages/shop/pages/partments';
import PartmentDetail from 'pages/shop/pages/partments/pages/detail';
import Members from 'pages/shop/pages/members';
import MemberDetail from 'pages/shop/pages/members/pages/detail';
import BranchShops from 'pages/shop/pages/branchShops';
import BranchShopDetail from 'pages/shop/pages/branchShops/pages/detail';
import Agents from 'pages/shop/pages/agents';
import AgentDetail from 'pages/shop/pages/agents/pages/detail';
import AgentRegister from 'pages/shop/pages/agents/pages/register';
import Shippers from 'pages/shop/pages/shippers';
import ShipperDetail from 'pages/shop/pages/shippers/pages/detail';
import ShipperRegister from 'pages/shop/pages/shippers/pages/register';
import Clients from 'pages/shop/pages/clients';
import ClientDetail from 'pages/shop/pages/clients/pages/detail';
import Roadmaps from 'pages/shop/pages/roadmaps';
import RoadmapDetail from 'pages/shop/pages/roadmaps/pages/detail';
import RegionRates from 'pages/shop/pages/regionRates';
import RegionRateDetail from 'pages/shop/pages/regionRates/pages/detail';
import PlaceOrder from 'pages/shop/pages/placeOrder';
import BROrders from 'pages/shop/pages/orders';
import BROrderDetail from 'pages/shop/pages/orders/pages/detail';
import RPOrders from 'pages/shop/pages/rporders';
import RPOrderDetail from 'pages/shop/pages/rporders/pages/detail';
import WHOrders from 'pages/shop/pages/whorders';
import WHOrderDetail from 'pages/shop/pages/whorders/pages/detail';
import Trucks from 'pages/shop/pages/trucks';
import Bills from 'pages/shop/pages/bills';
import Feedback from 'pages/shop/pages/feedback';
import Setting from 'pages/shop/pages/setting';
import Notify from 'pages/shop/pages/notifies';
import Statistics from 'pages/shop/pages/statistics';
import SendNotify from 'pages/shop/pages/notifies/pages/sendNotify';
import Warehouses from 'pages/shop/pages/warehouse';
import WarehouseDetail from 'pages/shop/pages/warehouse/pages/detail';

let firstEntry = true;

function authenticate (nextState, replaceState, callback) {
    if (typeof window !== 'undefined' && !firstEntry) {
        request
        .post('/shop/graphql')
        .set('Content-Type', 'application/json')
        .set('Accept', 'application/json')
        .send({
            query: 'query { session }',
        })
        .end((error, result) => {
            if (error || !result.body.data.session) {
                window.location.href = '/shop/login';
            } else {
                callback();
            }
        });
    } else {
        firstEntry = false;
        callback();
    }
}

export default [
    <Route name='shop' path='/shop' component={Shop}>
        <IndexRoute component={Home} onEnter={authenticate} />
        <Route name='shopPersonal' path='personal'>
            <IndexRoute component={Personal} onEnter={authenticate} />
            <Route name='shopPaymentPwd' path='paymentPwd' component={PaymentPwd} onEnter={authenticate} />
        </Route>
        <Route name='shopOwnShop' path='ownShop' component={OwnShop} onEnter={authenticate} />
        <Route name='shopPartment' path='partment' component={Partment} onEnter={authenticate} />
        <Route name='shopPartments' path='partments'>
            <IndexRoute component={Partments} onEnter={authenticate} />
            <Route name='shopPartmentDetail' path='detail' component={PartmentDetail} onEnter={authenticate} />
        </Route>
        <Route name='shopMembers' path='members'>
            <IndexRoute component={Members} onEnter={authenticate} />
            <Route name='shopMemberDetail' path='detail' component={MemberDetail} onEnter={authenticate} />
        </Route>
        <Route name='shopBranchShops' path='branchShops'>
            <IndexRoute component={BranchShops} onEnter={authenticate} />
            <Route name='shopBranchShopDetail' path='detail' component={BranchShopDetail} onEnter={authenticate} />
        </Route>
        <Route name='shopAgents' path='agents'>
            <IndexRoute component={Agents} onEnter={authenticate} />
            <Route name='shopAgentDetail' path='detail' component={AgentDetail} onEnter={authenticate} />
            <Route name='shopAgentRegister' path='register' component={AgentRegister} onEnter={authenticate} />
        </Route>
        <Route name='shopShippers' path='shippers'>
            <IndexRoute component={Shippers} onEnter={authenticate} />
            <Route name='shopShipperDetail' path='detail' component={ShipperDetail} onEnter={authenticate} />
            <Route name='shopShipperRegister' path='register' component={ShipperRegister} onEnter={authenticate} />
        </Route>
        <Route name='shopClients' path='clients'>
            <IndexRoute component={Clients} onEnter={authenticate} />
            <Route name='shopClientDetail' path='detail' component={ClientDetail} onEnter={authenticate} />
        </Route>
        <Route name='shopRoadmaps' path='roadmaps'>
            <IndexRoute component={Roadmaps} onEnter={authenticate} />
            <Route name='shopRoadmapDetail' path='detail' component={RoadmapDetail} onEnter={authenticate} />
        </Route>
        <Route name='shopRegionRates' path='regionRates'>
            <IndexRoute component={RegionRates} onEnter={authenticate} />
            <Route name='shopRegionRateDetail' path='detail' component={RegionRateDetail} onEnter={authenticate} />
        </Route>
        <Route name='shopPlaceOrder' path='placeOrder' component={PlaceOrder} onEnter={authenticate} />
        <Route name='shopBROrders' path='orders'>
            <IndexRoute component={BROrders} onEnter={authenticate} />
            <Route name='shopBROrderDetail' path='detail' component={BROrderDetail} onEnter={authenticate} />
        </Route>
        <Route name='shopRPOrders' path='rporders'>
            <IndexRoute component={RPOrders} onEnter={authenticate} />
            <Route name='shopRPOrderDetail' path='detail' component={RPOrderDetail} onEnter={authenticate} />
        </Route>
        <Route name='shopWHOrders' path='whorders'>
            <IndexRoute component={WHOrders} onEnter={authenticate} />
            <Route name='shopWHOrderDetail' path='detail' component={WHOrderDetail} onEnter={authenticate} />
        </Route>
        <Route name='shopTrucks' path='trucks' component={Trucks} onEnter={authenticate} />
        <Route name='shopBills' path='bills' component={Bills} onEnter={authenticate} />
        <Route name='shopSetting' path='setting' component={Setting} onEnter={authenticate} />
        <Route name='shopStatistics' path='statistics' component={Statistics} onEnter={authenticate} />
        <Route name='shopFeedback' path='feedback' component={Feedback} onEnter={authenticate} />
        <Route name='shopNotify' path='notifies'>
            <IndexRoute component={Notify} onEnter={authenticate} />
            <Route name='shopSendNotify' path='sendNotify' component={SendNotify} onEnter={authenticate} />
        </Route>
        <Route name='warehouse' path='warehouses' onEnter={authenticate} >
            <IndexRoute component={Warehouses} onEnter={authenticate} />
            <Route name='warehouseDetail' path='detail' component={WarehouseDetail} onEnter={authenticate} />
        </Route>
    </Route>,
];

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/index.js

```js
import * as personal from './personal';
import * as member from './member';
import * as shop from './shop';
import * as shipper from './shipper';
import * as agent from './agent';
import * as partment from './partment';
import * as order from './order';
import * as roadmap from './roadmap';
import * as setting from './setting';
import * as notify from './notify';
import * as warehouse from './warehouse';

export default {
    ...personal,
    ...member,
    ...shop,
    ...shipper,
    ...agent,
    ...partment,
    ...order,
    ...roadmap,
    ...setting,
    ...notify,
    ...warehouse,
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/index.js

```js
import * as common from './common';
import * as personal from './personal';
import * as account from './account';
import * as shop from './shop';
import * as partment from './partment';
import * as member from './member';
import * as shipper from './shipper';
import * as agent from './agent';
import * as client from './client';
import * as roadmap from './roadmap';
import * as order from './order';
import * as setting from './setting';
import * as statistic from './statistic';
import * as notify from './notify';
import * as warehouse from './warehouse';

export default {
    ...common,
    ...account,
    ...personal,
    ...shop,
    ...partment,
    ...member,
    ...shipper,
    ...agent,
    ...client,
    ...roadmap,
    ...order,
    ...setting,
    ...statistic,
    ...notify,
    ...warehouse,
};

```

* PDShop_Shop_PC/project/App/server/graphql/types/account.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
} from 'graphql';
import { makeResultType } from './common';

export const accountType = new GraphQLObjectType({
    name: 'accountType',
    fields: {
        amount: { type: GraphQLFloat },
        qrcode: { type: GraphQLString },
    },
});
export const accountResultType = makeResultType('accountResultType', accountType);

export const accountInputType = new GraphQLInputObjectType({
    name: 'accountInputType',
    fields: {
        amount: { type: GraphQLFloat },
        body: { type: GraphQLString },
        payPassword: { type: GraphQLString },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/agent.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
    GraphQLInt,
    GraphQLID,
    GraphQLList,
} from 'graphql';

import { makeResultType, makeListType, descriptType, descriptInputType } from './common';
import { clientType } from './client';

export const agentType = new GraphQLObjectType({
    name: 'agentType',
    fields: {
        // 列表的字段
        id: { type: GraphQLID },
        logo: { type: GraphQLString },
        image: { type: GraphQLString },
        addressRegion: { type: GraphQLString },
        addressRegionLastCode: { type: GraphQLInt },
        address: { type: GraphQLString },
        location: { type: new GraphQLList(GraphQLFloat) },
        phoneList:  { type: GraphQLString },
        name: { type: GraphQLString },
        chairMan: { type: clientType },
        // 基本信息
        sign: { type: GraphQLString },
        descriptList:  { type: new GraphQLList(descriptType) },
        registerTime: { type: GraphQLString },
        // 法人
        legalName: { type: GraphQLString },
        legalPhone: { type: GraphQLString },
        legalIDCard: { type: new GraphQLList(GraphQLString) },
    },
});
export const agentResultType = makeResultType('agentResultType', agentType);
export const agentListType = makeListType('agentListType', agentType);

export const agentInputType = new GraphQLInputObjectType({
    name: 'agentInputType',
    fields: {
        agentId: { type: GraphQLID },
        chairManId: { type: GraphQLString },
        name: { type: GraphQLString },
        logo: { type: GraphQLString },
        image: { type: GraphQLString },
        sign: { type: GraphQLString },
        addressRegion: { type: GraphQLString },
        addressRegionLastCode: { type: GraphQLInt },
        address: { type: GraphQLString },
        location: { type: new GraphQLList(GraphQLFloat) },
        phoneList:  { type: GraphQLString },
        descriptList:  { type: new GraphQLList(descriptInputType) },
        legalName: { type: GraphQLString },
        legalPhone: { type: GraphQLString },
        legalIDCard: { type: new GraphQLList(GraphQLString) },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/bill.js

```js
import {
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
} from 'graphql';

import { makeListType } from './common';

export const billType = new GraphQLObjectType({
    name: 'billType',
    fields: {
        tradeAmountYuan: { type: GraphQLFloat },
        tradeTime: { type: GraphQLString },
        thirdpartyAccount: { type: GraphQLString },
        remark: { type: GraphQLString },
    },
});
export const billItemListType = makeListType('billItemListType', billType, 'list');

```

* PDShop_Shop_PC/project/App/server/graphql/types/client.js

```js
import {
    GraphQLObjectType,
    GraphQLString,
    GraphQLInt,
    GraphQLID,
} from 'graphql';

import { makeResultType, makeListType } from './common';
export const clientType = new GraphQLObjectType({
    name: 'clientType',
    fields: {
        id: { type: GraphQLID },
        name: { type: GraphQLString },
        phone: { type: GraphQLString },
        email: { type: GraphQLString },
        head: { type: GraphQLString },
        birthday: { type: GraphQLString },
        age: { type: GraphQLInt },
        sex: { type: GraphQLInt },
        phoneList: { type: GraphQLString },
    },
});
export const clientResultType = makeResultType('clientResultType', clientType);
export const clientListType = makeListType('clientListType', clientType);

```

* PDShop_Shop_PC/project/App/server/graphql/types/common.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
    GraphQLBoolean,
    GraphQLInt,
    GraphQLList,
} from 'graphql';

export function makeResultType (name, type) {
    const base = {
        success: { type: GraphQLBoolean },
        msg: { type: GraphQLString },
    };
    const fields = type ? { ...base, context: { type: type } } : base;
    return new GraphQLObjectType({ name, fields });
}

export function makeListType (name, type, listName) {
    return new GraphQLObjectType({
        name,
        fields: {
            count: { type: GraphQLInt },
            [listName || name.replace(/Type$/, '')]: { type: new GraphQLList(type) },
        },
    });
}

export const successType = new GraphQLObjectType({
    name: 'successType',
    fields: {
        success: { type: GraphQLBoolean },
        msg: { type: GraphQLString },
    },
});

export const descriptType = new GraphQLObjectType({
    name: 'descriptType',
    fields: {
        img: { type: GraphQLString }, // 图片
        text: { type: GraphQLString }, // 描述
    },
});

export const descriptInputType = new GraphQLInputObjectType({
    name: 'descriptInputType',
    fields: {
        img: { type: GraphQLString }, // 图片
        text: { type: GraphQLString }, // 描述
    },
});

export const pointType = new GraphQLObjectType({
    name: 'pointType',
    fields: {
        name: { type: GraphQLString }, // 地名
        longitude: { type: GraphQLFloat }, // 经度
        latitude: { type: GraphQLFloat }, // 纬度
        locateTime: { type: GraphQLString }, // 定位时间
    },
});

export const pointInputType = new GraphQLInputObjectType({
    name: 'pointInputType',
    fields: {
        name: { type: GraphQLString }, // 地名
        longitude: { type: GraphQLFloat }, // 经度
        latitude: { type: GraphQLFloat }, // 纬度
    },
});

export const driverType = new GraphQLObjectType({
    name: 'driverType',
    fields: {
        name: { type: GraphQLString }, // 司机名字
        phone: { type: GraphQLString }, // 司机电话
    },
});

export const driverInputType = new GraphQLInputObjectType({
    name: 'driverInputType',
    fields: {
        name: { type: GraphQLString }, // 司机名字
        phone: { type: GraphQLString }, // 司机电话
    },
});

export const sizeType = new GraphQLObjectType({
    name: 'sizeType',
    fields: {
        length: { type: GraphQLFloat }, // 长度
        width: { type: GraphQLFloat }, // 宽度
        height: { type: GraphQLFloat }, // 高度
    },
});

export const sizeInputType = new GraphQLInputObjectType({
    name: 'sizeInputType',
    fields: {
        length: { type: GraphQLFloat }, // 长度
        width: { type: GraphQLFloat }, // 宽度
        height: { type: GraphQLFloat }, // 高度
    },
});

export const addressType = new GraphQLObjectType({
    name: 'addressType',
    fields: {
        parentCode: { type: GraphQLInt },
        code: { type: GraphQLInt },
        name: { type: GraphQLString },
        level: { type: GraphQLInt },
        isLeaf: { type: GraphQLBoolean },
    },
});

export const orderAddressType = new GraphQLObjectType({
    name: 'orderAddressType',
    fields: {
        addressList: { type: new GraphQLList(new GraphQLList(addressType)) },
        cityAddressList: { type: new GraphQLList(new GraphQLList(addressType)) },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/member.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLInt,
    GraphQLID,
    GraphQLList,
} from 'graphql';
import { makeResultType, makeListType } from './common';

const innerPartmentType = new GraphQLObjectType({
    name: 'innerPartmentType',
    fields: {
        id: { type: GraphQLID },
        name:  { type: GraphQLString },
        type:  { type: GraphQLInt },
    },
});

export const memberType = new GraphQLObjectType({
    name: 'memberType',
    fields: {
        id: { type: GraphQLID },
        userId: { type: GraphQLID },
        name: { type: GraphQLString },
        phone: { type: GraphQLString },
        email: { type: GraphQLString },
        head: { type: GraphQLString },
        age: { type: GraphQLInt },
        sex: { type: GraphQLInt },
        post: { type: GraphQLString },
        authority: { type: new GraphQLList(GraphQLInt) },
        birthday: { type: GraphQLString },
        phoneList: { type: GraphQLString },
        partment: { type: innerPartmentType },
    },
});
export const memberResultType = makeResultType('memberResultType', memberType);
export const memberListType = makeListType('memberListType', memberType);

export const memberInputType = new GraphQLInputObjectType({
    name: 'memberInputType',
    fields: {
        userId: { type: GraphQLString },
        memberId: { type: GraphQLID },
        name: { type: GraphQLString },
        phone: { type: GraphQLString },
        password: { type: GraphQLString },
        email: { type: GraphQLString },
        head: { type: GraphQLString },
        sex: { type: GraphQLInt },
        post: { type: GraphQLString },
        authority: { type: new GraphQLList(GraphQLInt) },
        birthday: { type: GraphQLString },
        phoneList: { type: GraphQLString },
        partmentId: { type: GraphQLID },
        shopId: { type: GraphQLID },
        verifyCode: { type: GraphQLString },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/notify.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLID,
} from 'graphql';
import { makeResultType, makeListType } from './common';

export const notifyType = new GraphQLObjectType({
    name: 'notifyType',
    fields: {
        id: { type: GraphQLID },
        title: { type: GraphQLString },
        content: { type: GraphQLString },
        time: { type: GraphQLString },
        source: { type: GraphQLString },
    },
});
export const notifyResultType = makeResultType('notifyResultType', notifyType);
export const notifyListType = makeListType('notifyListType', notifyType);

export const notifyInputType = new GraphQLInputObjectType({
    name: 'notifyInputType',
    fields: {
        notifyId: { type: GraphQLID },
        type: { type: GraphQLString },
        title: { type: GraphQLString },
        content: { type: GraphQLString },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/order.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
    GraphQLBoolean,
    GraphQLInt,
    GraphQLID,
    GraphQLList,
} from 'graphql';

import { makeResultType, makeListType } from './common';
import { memberType } from './member';

const stateListType = new GraphQLObjectType({
    name: 'stateListType',
    fields: {
        state: { type: GraphQLInt },
        count: { type: GraphQLInt },
    },
});
export const orderType = new GraphQLObjectType({
    name: 'orderType',
    fields: {
        id: { type: GraphQLID },
        senderPhone: { type: GraphQLString },
        senderName: { type: GraphQLString },
        receiverPhone: { type: GraphQLString },
        receiverName: { type: GraphQLString },
        name: { type: GraphQLString },
        endPoint: { type: GraphQLString },
        endPointLastCode: { type: GraphQLInt },
        sendDoorEndPoint: { type: GraphQLString },
        sendDoorEndPointLastCode: { type: GraphQLInt },
        totalNumbers: { type: GraphQLInt },
        weight: { type: GraphQLFloat },
        size: { type: GraphQLFloat },
        proxyCharge: { type: GraphQLFloat },
        proxyChargeProfit: { type: GraphQLFloat },
        totalDesignatedFee: { type: GraphQLFloat },
        isReachPay: { type: GraphQLBoolean },
        isSendDoor: { type: GraphQLBoolean },
        isClientPick: { type: GraphQLBoolean },
        state: { type: GraphQLInt },
        placeOrderTime: { type: GraphQLString },
        // 价格
        realFee: { type: GraphQLFloat }, // 实际运费
        designatedFee: { type: GraphQLFloat }, // 指定收款
        needPayTransportFee: { type: GraphQLFloat }, // 应该付的运费
        needPayInsuanceFee: { type: GraphQLFloat }, // 应该付的保险金额
        photo: { type: GraphQLString }, // 图片
        // 保险
        isInsuance: { type: GraphQLBoolean },
        insuanceFee: { type: GraphQLFloat },
        insuanceMount: { type: GraphQLFloat },
        // 提成
        profit: { type: GraphQLFloat },
        masterProfit: { type: GraphQLFloat },
        branchProfit: { type: GraphQLFloat },
        // 其他
        warehouse: { type: GraphQLString }, // 仓库
        isCityDistribute: { type: GraphQLBoolean }, // 是否是同城配送
        payMode: { type: GraphQLInt }, // 支付方式 0：现付（PM_IMMEDIATE），1：到付（PM_REACH），2：混合支付（PM_MIXED 指定收款的部分到付，其余的现付）
        isTransferOrder: { type: GraphQLBoolean }, // 是否是中转单
        receiveMember: { type: memberType }, // 收货员
        stateList: { type: new GraphQLList(stateListType) }, // 状态列表
        deductError: { type: GraphQLBoolean }, // 扣款是否失败
    },
});
export const orderResultType = makeResultType('orderResultType', orderType);
export const orderListType = makeListType('orderListType', orderType);
export const orderItemListType = makeListType('orderItemListType', orderType, 'list');

export const orderInputType = new GraphQLInputObjectType({
    name: 'orderInputType',
    fields: {
        orderId: { type: GraphQLID },
        preOrderId: { type: GraphQLID },
        senderPhone: { type: GraphQLString },
        senderName: { type: GraphQLString },
        receiverPhone: { type: GraphQLString },
        receiverName: { type: GraphQLString },
        name: { type: GraphQLString },
        endPoint: { type: GraphQLString },
        endPointLastCode: { type: GraphQLInt },
        sendDoorEndPoint: { type: GraphQLString },
        sendDoorEndPointLastCode: { type: GraphQLInt },
        totalNumbers: { type: GraphQLInt },
        weight: { type: GraphQLFloat },
        size: { type: GraphQLFloat },
        proxyCharge: { type: GraphQLInt },
        totalDesignatedFee: { type: GraphQLInt },
        isCityDistribute: { type: GraphQLBoolean },
        isReachPay: { type: GraphQLBoolean },
        isSendDoor: { type: GraphQLBoolean },
        isInsuance: { type: GraphQLBoolean },
        roadmapId: { type: GraphQLID },
        roadmapRankIndex: { type: GraphQLInt },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/partment.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
    GraphQLBoolean,
    GraphQLInt,
    GraphQLID,
} from 'graphql';
import { makeResultType, makeListType } from './common';
import { memberType } from './member';

export const partmentType = new GraphQLObjectType({
    name: 'partmentType',
    fields: {
        id: { type: GraphQLID },
        name:  { type: GraphQLString },
        descript: { type: GraphQLString },
        type:  { type: GraphQLInt },
        phoneList: { type: GraphQLString },
        chargeMan: { type: memberType },
        price: { type: GraphQLFloat },
        enable: { type: GraphQLBoolean },
    },
});
export const partmentResultType = makeResultType('partmentResultType', partmentType);
export const partmentListType = makeListType('partmentListType', partmentType);

export const partmentInputType = new GraphQLInputObjectType({
    name: 'partmentInputType',
    fields: {
        partmentId: { type: GraphQLID },
        name:  { type: GraphQLString },
        descript: { type: GraphQLString },
        type:  { type: GraphQLInt },
        phoneList: { type: GraphQLString },
        chargeManPhone: { type: GraphQLString },
        chargeManName: { type: GraphQLString },
        chargeManPassword: { type: GraphQLString },
        price: { type: GraphQLFloat },
        enable: { type: GraphQLBoolean },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/roadmap.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
    GraphQLInt,
    GraphQLID,
    GraphQLList,
} from 'graphql';

import { makeResultType, makeListType } from './common';
import { shipperType } from './shipper';
import { shopType } from './shop';

export const roadmapType = new GraphQLObjectType({
    name: 'roadmapType',
    fields: {
        id: { type: GraphQLID },
        // 选择物流公司的参数
        shipperName: { type: GraphQLString },
        transportFee: { type: GraphQLFloat },
        duration: { type: GraphQLFloat },
        price: { type: GraphQLFloat },
        minFee: { type: GraphQLFloat },
        sendDoorPrice: { type: GraphQLFloat },
        sendDoorMinFee: { type: GraphQLFloat },
        roadmapRankIndex: { type: GraphQLInt },
        // 路线列表的参数
        startPoint: { type: GraphQLString },
        endPoint: { type: GraphQLString },
        shipper: { type: shipperType },
        shop: { type: shopType },
        // 提成
        profitRate: { type: GraphQLFloat },
        defaultProfitRate: { type: GraphQLFloat },
        // 其他
        selfSignRate: { type: GraphQLFloat },
        createTime: { type: GraphQLString },
        setProfitRateTime: { type: GraphQLString },
    },
});
export const roadmapResultType = makeResultType('roadmapResultType', roadmapType);
export const roadmapListType = makeListType('roadmapListType', roadmapType);
export const roadmapInputType = new GraphQLInputObjectType({
    name: 'roadmapInputType',
    fields: {
        roadmapId: { type: GraphQLID },
        roadmapIdList: { type: new GraphQLList(GraphQLID) },
        profitRate: { type: GraphQLFloat },
    },
});

export const regionRateType = new GraphQLObjectType({
    name: 'regionRateType',
    fields: {
        id: { type: GraphQLID },
        // 路线列表的参数
        shop: { type: shopType },
        region: { type: GraphQLString },
        regionLastCode: { type: GraphQLInt },
        profitRate: { type: GraphQLFloat },
        type: { type: GraphQLInt },
        createTime: { type: GraphQLString },
        modifyTime: { type: GraphQLString },
    },
});
export const regionRateResultType = makeResultType('regionRateResultType', regionRateType);
export const regionRateListType = makeListType('regionRateListType', regionRateType);
export const regionRateInputType = new GraphQLInputObjectType({
    name: 'regionRateInputType',
    fields: {
        regionRateId: { type: GraphQLID },
        regionRateIdList: { type: new GraphQLList(GraphQLID) },
        shopId: { type: GraphQLID },
        region: { type: GraphQLString },
        regionLastCode: { type: GraphQLInt },
        profitRate: { type: GraphQLFloat },
        type: { type: GraphQLInt },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/setting.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLFloat,
    GraphQLList,
} from 'graphql';

import { makeResultType } from './common';

const gradientPriceType = new GraphQLObjectType({
    name: 'gradientPriceType',
    fields: {
        rate: { type: GraphQLFloat },
        min:  { type: GraphQLFloat },
    },
});

const gradientPriceInputType = new GraphQLInputObjectType({
    name: 'gradientPriceInputType',
    fields: {
        rate: { type: GraphQLFloat },
        min:  { type: GraphQLFloat },
    },
});

export const settingType = new GraphQLObjectType({
    name: 'settingType',
    fields: {
        sizeWeightRate: { type: GraphQLFloat }, // 方量和重量的计算比
        insuanceRate:  { type: GraphQLFloat }, // 担保保险的费用（客户实际付的钱）= 担保保险额度（需要赔偿用户的钱）* insuanceRate
        insuanceMountRate:  { type: GraphQLFloat }, // 保额相对于运费的多少倍，默认为10
        insuanceBaseValue:  { type: GraphQLFloat }, // 保险的最低值（默认为最低10元）
        proxyChargeProfitRate:  { type: GraphQLFloat }, //  手续费 = 代收货款 * proxyChargeProfitRate
        // 设置min必须从0开始，默认： [ {rate: 1.1, min: 0}, {rate: 1, min: 10}, {rate: 0.9, min: 20 } ] 如果物流公司设置的是100元/吨，表示0-10为110元/吨，10-20为100元/吨，20以上为90元/吨
        gradientPriceList: { type: new GraphQLList(gradientPriceType) }, // 每吨的价格，单位为元/吨（采用阶梯价格）
        additionalDeliverTime: { type: GraphQLFloat }, // 物流公司没有让货主确认附加的交易成功时间，单位小时(h)
        noticeShipperStorageWeight: { type: GraphQLFloat }, // 没多少吨货物放置后通知物流公司
        bondAmountWeightRate: { type: GraphQLFloat }, // 每吨货需要的保证金
        rankedMaskFirstRankWeight: { type: GraphQLFloat }, // 用来判断排名第一的物流公司的收购多少吨货被屏蔽的依据
        rankedMaskRateList: { type: new GraphQLList(GraphQLFloat) }, // 为了防止物流公司垄断，排在前3（数组的长度+1）名的需要轮流收货，第一名收货的量为rankedMaskFirstRankWeight*1，第二名为rankedMaskFirstRankWeight*0.8，第三名为rankedMaskFirstRankWeight*0.6。默认： [ 1, 0.8, 0.6 ]，屏蔽规则：见 RoadmapMaskModel
    },
});

export const settingInputType = new GraphQLInputObjectType({
    name: 'settingInputType',
    fields: {
        sizeWeightRate: { type: GraphQLFloat }, // 方量和重量的计算比
        insuanceRate:  { type: GraphQLFloat }, // 担保保险的费用（客户实际付的钱）= 担保保险额度（需要赔偿用户的钱）* insuanceRate
        insuanceMountRate:  { type: GraphQLFloat }, // 保额相对于运费的多少倍，默认为10
        insuanceBaseValue:  { type: GraphQLFloat }, // 保险的最低值（默认为最低10元）
        proxyChargeProfitRate:  { type: GraphQLFloat }, //  手续费 = 代收货款 * proxyChargeProfitRate
        // 设置min必须从0开始，默认： [ {rate: 1.1, min: 0}, {rate: 1, min: 10}, {rate: 0.9, min: 20 } ] 如果物流公司设置的是100元/吨，表示0-10为110元/吨，10-20为100元/吨，20以上为90元/吨
        gradientPriceList: { type: new GraphQLList(gradientPriceInputType) }, // 每吨的价格，单位为元/吨（采用阶梯价格）
        additionalDeliverTime: { type: GraphQLFloat }, // 物流公司没有让货主确认附加的交易成功时间，单位小时(h)
        noticeShipperStorageWeight: { type: GraphQLFloat }, // 没多少吨货物放置后通知物流公司
        bondAmountWeightRate: { type: GraphQLFloat }, // 每吨货需要的保证金
        rankedMaskFirstRankWeight: { type: GraphQLFloat }, // 用来判断排名第一的物流公司的收购多少吨货被屏蔽的依据
        rankedMaskRateList: { type: new GraphQLList(GraphQLFloat) }, // 为了防止物流公司垄断，排在前3（数组的长度+1）名的需要轮流收货，第一名收货的量为rankedMaskFirstRankWeight*1，第二名为rankedMaskFirstRankWeight*0.8，第三名为rankedMaskFirstRankWeight*0.6。默认： [ 1, 0.8, 0.6 ]，屏蔽规则：见 RoadmapMaskModel
    },
});
export const settingResultType = makeResultType('settingResultType', settingType);

```

* PDShop_Shop_PC/project/App/server/graphql/types/shipper.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
    GraphQLInt,
    GraphQLID,
    GraphQLList,
} from 'graphql';

import { makeResultType, makeListType, descriptType, descriptInputType } from './common';
import { clientType } from './client';

export const bondCompanyType = new GraphQLObjectType({
    name: 'bondCompanyType',
    fields: {
        capital: { type: GraphQLFloat }, // 担保公司注册资金
        bondAmount: { type: GraphQLFloat }, // 担保金额
        name: { type: GraphQLString }, // 担保公司名称
        address: { type: GraphQLString }, // 担保公司地址
        phoneList: { type: GraphQLString }, // 担保公司电话
        legalName: { type: GraphQLString }, // 法人姓名
        legalPhone: { type: GraphQLString }, // 法人电话
        certificate: { type: new GraphQLList(GraphQLString) }, // 担保公司资格证书
        legalIDCard: { type: new GraphQLList(GraphQLString) }, // 法人身份证
    },
});
export const shipperType = new GraphQLObjectType({
    name: 'shipperType',
    fields: {
        // 列表的字段
        id: { type: GraphQLID },
        shipperType: { type: GraphQLInt },
        shopId: { type: GraphQLID },
        shopName: { type: GraphQLString },
        name: { type: GraphQLString },
        chairMan: { type: clientType },
        acountAmount: { type: GraphQLFloat },
        totalBondAmount: { type: GraphQLFloat },
        remainBondAmount: { type: GraphQLFloat },
        capital: { type: GraphQLFloat },
        // 基本信息
        logo: { type: GraphQLString },
        image: { type: GraphQLString },
        sign: { type: GraphQLString },
        address: { type: GraphQLString },
        phoneList:  { type: GraphQLString },
        descriptList:  { type: new GraphQLList(descriptType) },
        registerTime: { type: GraphQLString },
        // 法人
        legalName: { type: GraphQLString },
        legalPhone: { type: GraphQLString },
        legalIDCard: { type: new GraphQLList(GraphQLString) },
        // 担保公司
        bondCompanyList: { type: new GraphQLList(bondCompanyType) },
    },
});
export const shipperResultType = makeResultType('shipperResultType', shipperType);
export const shipperListType = makeListType('shipperListType', shipperType);

export const bondCompanyInputType = new GraphQLInputObjectType({
    name: 'bondCompanyInputType',
    fields: {
        capital: { type: GraphQLFloat }, // 担保公司注册资金
        bondAmount: { type: GraphQLFloat }, // 担保金额
        name: { type: GraphQLString }, // 担保公司名称
        address: { type: GraphQLString }, // 担保公司地址
        phoneList: { type: GraphQLString }, // 担保公司电话
        legalName: { type: GraphQLString }, // 法人姓名
        legalPhone: { type: GraphQLString }, // 法人电话
        certificate: { type: new GraphQLList(GraphQLString) }, // 担保公司资格证书
        legalIDCard: { type: new GraphQLList(GraphQLString) }, // 法人身份证
    },
});
export const shipperInputType = new GraphQLInputObjectType({
    name: 'shipperInputType',
    fields: {
        shipperId: { type: GraphQLID },
        shipperType: { type: GraphQLInt },
        chairManId: { type: GraphQLString },
        name: { type: GraphQLString },
        logo: { type: GraphQLString },
        image: { type: GraphQLString },
        sign: { type: GraphQLString },
        address: { type: GraphQLString },
        phoneList:  { type: GraphQLString },
        descriptList:  { type: new GraphQLList(descriptInputType) },
        capital: { type: GraphQLFloat },
        legalName: { type: GraphQLString },
        legalPhone: { type: GraphQLString },
        legalIDCard: { type: new GraphQLList(GraphQLString) },
        bondCompanyList: { type: new GraphQLList(bondCompanyInputType) },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/shop.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLBoolean,
    GraphQLFloat,
    GraphQLInt,
    GraphQLID,
    GraphQLList,
} from 'graphql';
import { makeResultType, makeListType, descriptType, descriptInputType } from './common';

const memberInShopType = new GraphQLObjectType({
    name: 'memberInShopType',
    fields: {
        id: { type: GraphQLID },
        name: { type: GraphQLString },
        phone: { type: GraphQLString },
        head: { type: GraphQLString },
    },
});
export const shopType = new GraphQLObjectType({
    name: 'shopType',
    fields: {
        id: { type: GraphQLID },
        name: { type: GraphQLString },
        logo: { type: GraphQLString },
        image: { type: GraphQLString },
        sign: { type: GraphQLString },
        phoneList: { type: GraphQLString },
        addressRegion: { type: GraphQLString },
        addressRegionLastCode: { type: GraphQLInt },
        address: { type: GraphQLString },
        location: { type: new GraphQLList(GraphQLFloat) },
        descriptList: { type: new GraphQLList(descriptType) },
        chairMan: { type: memberInShopType },
        profitRate: { type: GraphQLFloat },
        isMasterShop: { type: GraphQLBoolean },
    },
});
export const shopResultType = makeResultType('shopResultType', shopType);
export const branchShopListType = makeListType('branchShopListType', shopType);

export const shopInputType = new GraphQLInputObjectType({
    name: 'shopInputType',
    fields: {
        shopId: { type: GraphQLID },
        name: { type: GraphQLString },
        logo: { type: GraphQLString },
        image: { type: GraphQLString },
        sign: { type: GraphQLString },
        phoneList: { type: GraphQLString },
        addressRegion: { type: GraphQLString },
        addressRegionLastCode: { type: GraphQLInt },
        address: { type: GraphQLString },
        location: { type: new GraphQLList(GraphQLFloat) },
        descriptList:  { type: new GraphQLList(descriptInputType) },
        profitRate: { type: GraphQLFloat },
        chairManPhone: { type: GraphQLString },
        chairManName: { type: GraphQLString },
        chairManPassword: { type: GraphQLString },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/statistic.js

```js
import {
    GraphQLObjectType,
    GraphQLFloat,
    GraphQLInt,
    GraphQLList,
} from 'graphql';

export const shopStatisticType = new GraphQLObjectType({
    name: 'shopStatisticType',
    fields: {
        count: { type: new GraphQLList(GraphQLInt) },
        isReachPay: { type: new GraphQLList(GraphQLInt) },
        isInsuance: { type: new GraphQLList(GraphQLInt) },
        insuanceMount: { type: new GraphQLList(GraphQLFloat) },
        insuanceFee: { type: new GraphQLList(GraphQLFloat) },
        realFee: { type: new GraphQLList(GraphQLFloat) },
        totalDesignatedFee: { type: new GraphQLList(GraphQLFloat) },
        proxyCharge: { type: new GraphQLList(GraphQLFloat) },
        masterProfit: { type: new GraphQLList(GraphQLFloat) },
        branchProfit: { type: new GraphQLList(GraphQLFloat) },
        weight: { type: new GraphQLList(GraphQLFloat) },
        size: { type: new GraphQLList(GraphQLFloat) },
    },
});

```

* PDShop_Shop_PC/project/App/server/graphql/types/truck.js

```js
import {
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
    GraphQLInt,
    GraphQLID,
} from 'graphql';

import { makeResultType, makeListType, driverType } from './common';

export const truckType = new GraphQLObjectType({
    name: 'truckType',
    fields: {
        id: { type: GraphQLID }, // Id
        // 基本信息
        plateNo: { type: GraphQLString }, // 车牌号码
        drivingLicense: { type: GraphQLString }, // 行驶证编号
        truckType: { type: GraphQLString }, // 车型
        insuanceMount: { type: GraphQLFloat }, // 保险
        driver: { type: driverType }, // 司机的Id
        state: { type: GraphQLInt }, // 货车状态
    },
});

export const truckResultType = makeResultType('truckResultType', truckType);
export const truckListType = makeListType('truckListType', truckType);

```

* PDShop_Shop_PC/project/App/server/graphql/types/warehouse.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLInt,
    GraphQLID,
    GraphQLList,
    GraphQLFloat,
} from 'graphql';
import { makeResultType, makeListType } from './common';
import { memberType } from './member';
import { shipperType } from './shipper';

export const warehouseType = new GraphQLObjectType({
    name: 'warehouseType',
    fields: {
        id: { type: GraphQLID },
        shopId: { type: GraphQLID },
        shipperList: { type: new GraphQLList(shipperType) },
        houseMan: { type: memberType },
        houseNo: { type: GraphQLString },
        maxStoreWeight: { type: GraphQLFloat },
        maxStoreSize: { type: GraphQLFloat },
        orderCount: { type: GraphQLInt },
        totalNumbers: { type: GraphQLInt },
        totalWeight: { type: GraphQLFloat },
        totalSize: { type: GraphQLString },
    },
});
export const warehouseResultType = makeResultType('warehouseResultType', warehouseType);
export const warehouseListType = makeListType('warehouseListType', warehouseType);

export const warehouseInputType = new GraphQLInputObjectType({
    name: 'warehouseInputType',
    fields: {
        warehouseId: { type: GraphQLID },
        shopId: { type: GraphQLID },
        shipperList: { type: new GraphQLList(GraphQLID) },
        houseManId: { type: GraphQLID },
        houseNo: { type: GraphQLString },
        maxStoreWeight: { type: GraphQLFloat },
        maxStoreSize: { type: GraphQLFloat },
        orderCount: { type: GraphQLInt },
        totalNumbers: { type: GraphQLInt },
        totalWeight: { type: GraphQLFloat },
        totalSize: { type: GraphQLString },
    },
});

```

* PDShop_Shop_PC/project/App/server/shared/helpers/bind-error-type.js

```js
import {
    GraphQLUnionType,
} from 'graphql';
import _ from 'lodash';
import { errorType } from '../../graphql/types/error';

export default function bindErrorType (obj) {
    let key = _.keys(obj)[0];
    let type = obj[key];
    return new GraphQLUnionType({
        name: key + 'BindErrorType',
        types: [type, errorType],
        resolveType: (value) => {
            if (value.error) {
                return errorType;
            }
            return type;
        },
    });
}

```

* PDShop_Shop_PC/project/App/server/shared/helpers/default-favicon.js

```js
export default function getDefaultIcon (res) {
    return {
        tag: 'link',
        props: {
            rel: 'shortcut icon',
            type: 'image/vnd.microsoft.icon',
            href: `${res.baseScriptsURL}/shop/favicon.ico`,
        },
    };
}

```

* PDShop_Shop_PC/project/App/server/shared/helpers/get-base-component.js

```js
import React from 'react';
import { Provider } from 'react-redux';
import { ReduxRouter } from 'redux-router';
import getMuiTheme from 'material-ui/styles/getMuiTheme';
import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';

export default (req) => {
    const { store, headers } = req;
    const muiTheme = getMuiTheme({ userAgent: headers['user-agent'] });
    return (
        <MuiThemeProvider muiTheme={muiTheme}>
            <Provider store={store}>
                <ReduxRouter />
            </Provider>
        </MuiThemeProvider>
    );
};

```

* PDShop_Shop_PC/project/App/server/shared/helpers/get-markup.js

```js
import serialize from 'serialize-javascript';
import Html from 'components/html';
import React from 'react';
import { renderToString } from 'react-dom/server';
import { Provider } from 'react-redux';
import { ReduxRouter } from 'redux-router';
import getMuiTheme from 'material-ui/styles/getMuiTheme';
import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';

export default function getMarkup (req, res) {
    const { store, headers } = req;
    const state = store.getState();
    const initialState = serialize(state);
    const muiTheme = getMuiTheme({ userAgent: headers['user-agent'] });

    const markup = renderToString(
        <MuiThemeProvider muiTheme={muiTheme}>
            <Provider store={store}>
                <ReduxRouter />
            </Provider>
        </MuiThemeProvider>
    );

    const htmlMarkup = renderToString(
        <Html
            body={markup}
            props={initialState}
            locals={res.locals}
            />
    );

    return htmlMarkup;
}

```

* PDShop_Shop_PC/project/App/server/shared/helpers/get-projection.js

```js
export function getProjection (fieldASTs) {
    return fieldASTs.selectionSet.selections.reduce((projections, selection) => {
        projections[selection.name.value] = 1;
        return projections;
    }, {});
}

export function getInlineProjection (fieldASTs) {
    return fieldASTs.selectionSet.selections[0].selectionSet.selections.reduce((projections, selection) => {
        projections[selection.name.value] = 1;
        return projections;
    }, {});
}

```

* PDShop_Shop_PC/project/App/server/shared/helpers/render-html.js

```js
import serialize from 'serialize-javascript';
import Html from 'components/html';
import React from 'react';
import { renderToString } from 'react-dom/server';

export default ({ component, store, locals }) => {
    const state = store.getState();
    const initialState = serialize(state);

    const markup = renderToString(component);

    const htmlMarkup = renderToString(
        <Html
            body={markup}
            props={initialState}
            locals={locals}
            />
    );

    return htmlMarkup;
};

```

* PDShop_Shop_PC/project/App/server/shared/helpers/route-handler.js

```js
import configureStore from 'helpers/configure-store';
import { createMemoryHistory } from 'history';
import { match, reduxReactRouter } from 'redux-router/server';

export default function routeHandler (routes, req, res, next) {
    const store = configureStore(reduxReactRouter({
        createHistory: createMemoryHistory,
        routes,
    }));
    const url = req.originalUrl;

    store.dispatch(match(url, async (error, redirectLocation, routerState) => {
        if (error) {
            next(error);
        } else if (redirectLocation) {
            res.redirect(302, redirectLocation.pathname + redirectLocation.search);
        } else if (routerState) {
            req.routerState = routerState;
            req.store = store;
            next();
        } else {
            res.status(404).send('Not found');
        }
    }));
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/index.js

```js
import React from 'react';
import { rootDataConnect } from 'relatejs';
import Shop from './contents';
import _ from 'lodash';

@rootDataConnect()
export default class ShopContainer extends React.Component {
    state = {
        personal: {},
        shop : {},
        printShow: false,
    };
    updatePersonal (personal) {
        this.setState({ personal });
    }
    updateShop (shop) {
        this.setState({ shop });
    }
    updatePrintShow (printShow) {
        this.setState({ printShow });
    }
    hasAuthority (...authority) {
        const { personal } = this.state;
        return !!_.intersection(personal.authority, authority).length;
    }
    render () {
        const { personal, shop, printShow } = this.state;
        return (
            <Shop {...this.props} printShow={printShow} rootPersonal={personal} rootShop={shop} hasAuthority={::this.hasAuthority}>
                {
                    React.cloneElement(this.props.children, {
                        rootPersonal: personal,
                        updatePersonal: ::this.updatePersonal,
                        rootShop: shop,
                        updateShop: ::this.updateShop,
                        updatePrintShow: ::this.updatePrintShow,
                        hasAuthority: ::this.hasAuthority,
                    })
                }
            </Shop>
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/actions/mutation.js

```js
import invariant from 'invariant';
import { buildQueryAndVariables } from '../helpers/fragments';

import actionTypes from './types';
import request from '../helpers/request';

export default function (options, callback = false) {
    return (dispatch, getState) => {
        invariant(options.fragments, 'Relate: Mutation needs fragments defined');

        const mutation = buildQueryAndVariables(options.fragments, options.variables, 'mutation');
        const { headers, endpoint, body, withCredentials } = getState().relateReducer;
        return request({
            dispatch,
            type: actionTypes.mutation,
            query: mutation.query,
            variables: mutation.variables,
            fragments: options.fragments,
            headers: options.headers || headers,
            body: options.body || body,
            endpoint: options.endpoint || endpoint,
            withCredentials,
        }).then((result) => {
            if (callback !== false) {
                callback(result, dispatch);
            }
        });
    };
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/actions/query.js

```js
import q from 'q';
import warning from 'warning';

import actionTypes from './types';
import request from '../helpers/request';

export default function graphql (
    { query, variables },
    fragments,
    connectors,
    scopes,
    config,
    relateSSR = false
) {
    return (dispatch, getState) => {
        let result;

        if (relateSSR) {
            result = q()
            .then(() => relateSSR({
                query,
                variables,
            }))
            .then(({ data, errors }) => {
                dispatch({
                    type: actionTypes.query,
                    data,
                    errors,
                    variables,
                    connectors,
                    scopes,
                    fragments,
                    isIntrospection: true,
                });
            })
            .catch((err) => {
                warning(false, err);
            });
        } else {
            const { headers, endpoint, body, withCredentials } = getState().relateReducer;
            result = request({
                dispatch,
                type: actionTypes.query,
                query,
                variables,
                connectors,
                scopes,
                fragments,
                headers: config && config.headers || headers,
                body: config && config.body || body,
                endpoint: config && config.endpoint || endpoint,
                withCredentials,
            });
        }

        return result;
    };
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/actions/remove-connector.js

```js
import actionTypes from './types';

export default function removeConnector (id) {
    return {
        type: actionTypes.removeConnector,
        id,
    };
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/actions/settings.js

```js
import actionTypes from './types';

export function setHeader (key, value) {
    return {
        type: actionTypes.setHeader,
        key,
        value,
    };
}

export function removeHeader (key) {
    return {
        type: actionTypes.removeHeader,
        key,
    };
}

export function setEndpoint (endpoint) {
    return {
        type: actionTypes.setEndpoint,
        endpoint,
    };
}

export function setBody (key, value) {
    return {
        type: actionTypes.setBody,
        key,
        value,
    };
}

export function removeBody (key) {
    return {
        type: actionTypes.removeBody,
        key,
    };
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/actions/types.js

```js
export default {
    query: 'RELATE_QUERY',
    mutation: 'RELATE_MUTATION',
    removeConnector: 'RELATE_REMOVE_CONNECTOR',
    setHeader: 'RELATE_SET_HEADER',
    removeHeader: 'RELATE_REMOVE_HEADER',
    setEndpoint: 'RELATE_SET_ENDPOINT',
    setBody: 'RELATE_SET_BODY',
    removeBody: 'RELATE_REMOVE_BODY',
};

```

* PDShop_Shop_PC/project/App/shared/relatejs/decorators/data-connect.js

```js
import React, { Component, PropTypes } from 'react';

import { connect } from 'react-redux';
import { generateConnectorId } from '../helpers/connectors-ids';
import getVariables from '../helpers/get-variables';
import hoistStatics from 'hoist-non-react-statics';
import invariant from 'invariant';
import removeConnector from '../actions/remove-connector';
import KeepData from '../store/keep-data';
import _ from 'lodash';

export default function dataConnect (...args) {
    const { length } = args;
    let getReduxState, getReduxDispatches, getBundle;
    invariant(length, 'Relate: a dataConnect does not have arguments specified');
    if (length === 1) {
        getBundle = args[0];
    } else {
        getReduxState = args[0];
        getReduxDispatches = args[1];
        getBundle = args[2];
    }

    return function wrapWithDataConnect (WrappedComponent) {
        class ConnectData extends Component {
            static propTypes = {
                relateConnectorData: PropTypes.object,
                CONNECTOR_ID: PropTypes.string.isRequired,
            };

            static contextTypes = {
                fetchData: PropTypes.func.isRequired,
                store: PropTypes.any.isRequired,
                relate_ssr: PropTypes.func,
            };

            static defaultProps = {
                relateConnectorData: {},
            };

            static relateIdentifier = 'DATA_CONNECT';

            constructor (props, context) {
                super(props, context);

                this.constructor.displayName = WrappedComponent.name;

                // Relate connector info
                this.relate = {
                    setKeepData: ::this.setKeepData,
                    refresh: ::this.refresh,
                    loadMore: ::this.loadMore,
                    loadPage: ::this.loadPage,
                    variables: {},
                    hasMore: true,
                    pageNo: 0,
                };

                // Set keepData
                if (props.keepData) {
                    this.setKeepData(props.keepData);
                }

                // get bundle
                this.initialBundle = this.processBundle(props);
                if (!props.relateConnectorData || Object.keys(props.relateConnectorData).length === 0) {
                    this.initialHasDataToFetch = !!(
                        this.initialBundle &&
                        !this.initialBundle.manualLoad &&
                        this.hasDataToFetch(this.initialBundle.fragments)
                    );
                    if (this.context.relate_ssr) {
                        this.initialFetchData();
                    }
                    // Set initial state
                    this.state = {
                        loading: this.initialHasDataToFetch,
                        error: false,
                    };
                } else {
                    this.state = {
                        loading: false,
                        error: false,
                    };
                }
            }

            componentWillMount () {
                this.initialFetchData();
            }

            shouldComponentUpdate (nextProps, nextState) {
                return (
                    !this.state.loading ||
                    this.state.loading && !nextState.loading ||
                    this.state.error !== nextState.error
                );
            }

            componentWillUnmount () {
                const { pathname } = this.props.location || {};
                // 如果有保留数据，则不删除 CONNECTOR_ID，否则会删除 CONNECTOR_ID， 同时会删除 keepData
                if (!KeepData.getHasKeepData(pathname)) {
                    this.context.store.dispatch(removeConnector(this.props.CONNECTOR_ID));
                }
            }

            /*
            * 如果进入其他页面需要保留该页面的数据的时候，使用该接口
            * keepData 为需要额外保留的数据，如果没有，使用 setKeepData(true);
            * 需要保留的额外数据的情况
            * relate.setKeepData({ lastSelectIndex: index, lastCurrent: current });
            * 如果想要手动删除 keepData, 使用 setKeepData();
            *
            * 如果想设置一个全局的保留数据，需要在 @dataConnect 里面设置，如下：
            * @dataConnect(
            *    (state, props) => ({ states: state.tests, keepData: true })
            * }
            *
            * 如果不设置pathname，则使用 location.pathname， 只有 IndexRoute 的时候才需要设置
            */
            setKeepData (keepData, pathname) {
                KeepData.update(this.props.location.pathname, keepData, this.props.CONNECTOR_ID, this.context.store, pathname);
            }

            initialFetchData () {
                if (this.initialHasDataToFetch) {
                    this.fetchData({
                        fragments: this.initialBundle.fragments,
                        variables: this.initialBundle.initialVariables,
                        mutations: this.initialBundle.mutations,
                    });
                }
            }

            /* 如果没有设置 property，则取 variables 的键值，如果不需要请求参数的时候使用 property ,需要的时候必须使用 variables
            * 不需要参数的情况
            * relate.refresh({
            *     property: 'personal',
            * });
            *
            * 要参数的情况
            * relate.refresh({
            *     variables: {
            *         clients: {
            *             pageNo: 0,
            *             pageSize,
            *             keyword,
            *         },
            *     },
            * });
            */
            refresh ({ variables, property, callback }) {
                const properties = [...(variables ? _.keys(variables) : []), ...(_.isArray(property) ? property : [property])];
                const bundle = this.processBundle(this.props, variables);

                // Fetch data
                if (bundle) {
                    this.setState({
                        loading: true,
                    }, () => {
                        this.fetchData({
                            fragments: _.pick(bundle.fragments, properties),
                            variables,
                            mutations: bundle.mutations,
                            callback,
                        });
                    });
                }
            }

            // 适合做上拉加载更多的列表
            loadMore ({ variables, property, callback }) {
                const key = _.keys(variables)[0];
                const bundle = this.processBundle(this.props, variables);
                // Fetch data
                if (bundle && !this.lock) {
                    this.lock = true;
                    this.setState({
                        loading: true,
                        loadingMore: true,
                    }, () => {
                        this.fetchData({
                            fragments: _.pick(bundle.fragments, key),
                            variables,
                            mutations: bundle.mutations,
                            loadingMoreProperty: property || true,
                            callback,
                        });
                    });
                }
            }

            // 适合做分页的表格
            loadPage ({ variables, property, callback }) {
                const key = _.keys(variables)[0];
                const bundle = this.processBundle(this.props, variables);
                const { pageNo, pageSize } = variables[key];
                // Fetch data
                if (bundle && !this.lock) {
                    this.lock = true;
                    this.setState({
                        loadingPage: true,
                    }, () => {
                        this.fetchData({
                            fragments: _.pick(bundle.fragments, key),
                            variables,
                            mutations: bundle.mutations,
                            loadingPageProperty: property || true,
                            loadingPageStartIndex: pageNo * pageSize,
                            callback,
                        });
                    });
                }
            }

            hasDataToFetch (fragments) {
                return typeof fragments === 'object' && Object.keys(fragments).length > 0;
            }

            processBundle (props, variables) {
                const bundle = getBundle && getBundle(props);

                this.variablesTypes = bundle && bundle.variablesTypes || {};
                this.relate.variables = variables || bundle && bundle.initialVariables || {};

                return bundle;
            }

            fetchData ({ fragments, variables, mutations, loadingMoreProperty = false, loadingPageProperty = false, loadingPageStartIndex = 0, callback }) {
                console.log('[relatejs] fetchData:', { fragments, variables, mutations, loadingMoreProperty, loadingPageProperty, loadingPageStartIndex });
                const { fetchData } = this.context;

                if (fetchData && this.hasDataToFetch(fragments)) {
                    const fetchOptions = {
                        fragments,
                        variables: getVariables({
                            variables,
                            variablesTypes: this.variablesTypes,
                            fragments,
                            displayName: typeof WrappedComponent !== 'undefined' && WrappedComponent.displayName,
                        }),
                        ID: this.props.CONNECTOR_ID,
                        mutations,
                        loadingMoreProperty,
                        loadingPageProperty,
                        loadingPageStartIndex,
                    };
                    if (this.context.relate_ssr) {
                        fetchData(fetchOptions);
                    } else {
                        fetchData(fetchOptions)
                        .then((data) => {
                            callback && callback(data);
                            const { loadingMore } = this.state;
                            if (loadingMoreProperty && loadingMore) {
                                const key = _.keys(variables)[0];
                                const { pageNo, pageSize } = variables[key];
                                const item = (loadingMoreProperty === true) ? data[key] : data[key][loadingMoreProperty];
                                if (item) {
                                    this.relate.hasMore = item.length >= pageSize;
                                    this.relate.pageNo = pageNo;
                                }
                            }

                            this.setState({
                                loading: false,
                                loadingMore: false,
                                loadingPage: false,
                                error: false,
                            });

                            this.lock = false;
                        })
                        .catch(() => {
                            this.setState({
                                loading: false,
                                loadingMore: false,
                                loadingPage: false,
                                error: true,
                            });
                        });
                    }
                } else {
                    if (!this.context.relate_ssr) {
                        this.setState({
                            loading: false,
                            loadingMore: false,
                            loadingPage: false,
                            error: false,
                        });
                    }
                }
            }

            render () {
                const { relateConnectorData, ...otherProps } = this.props;
                const { loading, loadingMore, loadingPage } = this.state;
                return (
                    <WrappedComponent
                        {...otherProps}
                        {...relateConnectorData}
                        relate={this.relate}
                        loading={loading}
                        loadingMore={loadingMore}
                        loadingPage={loadingPage}
                        />
                );
            }
        }

        const Connected = connect(
            () => function map (state, props) {
                const { pathname } = props.location || {};
                const refuxProps = getReduxState && getReduxState(state, props) || {};
                const keepData = KeepData.get(pathname, refuxProps);
                if (!this.CONNECTOR_ID) {
                    this.CONNECTOR_ID = keepData.connectorId;
                }
                if (!this.CONNECTOR_ID) {
                    if (state.relateReducer.server) {
                        const finalProps = Object.assign({}, props, refuxProps);
                        const initialBundle = getBundle(finalProps);
                        const thisCompare = {
                            fragments: initialBundle.fragments,
                            variables: getVariables({
                                variables: initialBundle.initialVariables,
                                variablesTypes: initialBundle.variablesTypes,
                                fragments: initialBundle.fragments,
                            }),
                        };

                        _.forEach(state.relateReducer.server, (compare, id) => {
                            if (_.isEqual(compare, thisCompare)) {
                                this.CONNECTOR_ID = id;
                                return false;
                            }
                            return true;
                        });

                        if (!this.CONNECTOR_ID) {
                            this.CONNECTOR_ID = generateConnectorId();
                        }
                    } else {
                        this.CONNECTOR_ID = generateConnectorId();
                    }
                }
                return Object.assign(
                    refuxProps,
                    {
                        relateConnectorData: state.relateReducer[this.CONNECTOR_ID],
                        CONNECTOR_ID: this.CONNECTOR_ID,
                    },
                    keepData.data,
                );
            },
            (dispatch) => Object.assign(
                getReduxDispatches && getReduxDispatches(dispatch) || {},
                {
                    removeConnector: dispatch(removeConnector),
                }
            )
        )(ConnectData);

        return hoistStatics(Connected, WrappedComponent, {
            relateIdentifier: true,
        });
    };
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/decorators/root-data-connect.js

```js
import hoistStatics from 'hoist-non-react-statics';
import warning from 'warning';
import Q from 'q';
import _ from 'lodash';
import React, { Component, PropTypes } from 'react';
import { mergeFragments, buildQueryAndVariables } from '../helpers/fragments';

import queryAction from '../actions/query';

export default function rootDataConnect (config) {
    return function wrapWithDataConnect (WrappedComponent) {
        class RootConnectData extends Component {
            static contextTypes = {
                store: PropTypes.any.isRequired,
                relate_ssr: PropTypes.func,
            };

            static childContextTypes = {
                fetchData: PropTypes.func.isRequired,
            };

            static relateIdentifier = 'ROOT_DATA_CONNECT';

            constructor (props, context) {
                super(props, context);
                this.bundle = {};
                this.childFetchDataBind = ::this.childFetchData;
                this.fetchDebounce = _.debounce(::this.fetchData, 10);
                this.scopeID = 0;
            }

            getChildContext () {
                return {
                    fetchData: this.childFetchDataBind,
                };
            }

            componentDidMount () {
                this.mounted = true;
                if (this.bundle && this.bundle.fragments) {
                    this.fetchData();
                }
            }

            childFetchData ({ fragments, variables = {}, ID, mutations, loadingMoreProperty = false, loadingPageProperty = false, loadingPageStartIndex = 0 }) {
                // Check for same query with different variables
                const scopes = {};
                const resultFragments = Object.assign({}, fragments || {});
                const resultVariables = Object.assign({}, variables || {});

                if (this.bundle.fragments) {
                    _.forEach(fragments, (fragment, queryName) => {
                        if (this.bundle.fragments[queryName]) {
                            // Same query name detected, will have to check if variables are the same
                            const sameVariables = _.isEqual(
                                variables && variables[queryName],
                                this.bundle.variables && this.bundle.variables[queryName]
                            );

                            if (!sameVariables) {
                                // Will have to scope it
                                const scope = `relate_${this.scopeID++}`;
                                const newQueryName = `${scope}: ${queryName}`;
                                resultFragments[newQueryName] = Object.assign({}, fragments[queryName]);
                                scopes[scope] = queryName;
                                delete resultFragments[queryName];

                                if (resultVariables[queryName]) {
                                    resultVariables[newQueryName] = Object.assign({}, variables[queryName]);
                                    delete resultVariables[queryName];
                                }
                            }
                        }
                    });
                }

                this.bundle = {
                    fragments: mergeFragments(this.bundle.fragments || {}, resultFragments),
                    variables: Object.assign(this.bundle.variables || {}, resultVariables),
                    connectors: Object.assign(this.bundle.connectors || {}, {
                        [ID]: { fragments, variables, mutations, scopes, loadingMoreProperty, loadingPageProperty, loadingPageStartIndex },
                    }),
                    scopes: Object.assign(this.bundle.scopes || {}, scopes),
                };

                let result = null;
                if (!this.context || !this.context.relate_ssr) {
                    if (this.fetchingData && this.deferred) {
                        if (!this.nextDeferred) {
                            this.nextDeferred = Q.defer();

                            this.deferred.promise.fin(() => {
                                this.deferred = this.nextDeferred;
                                this.nextDeferred = null;
                                this.fetchData();
                            });
                        }

                        result = this.nextDeferred.promise;
                    } else {
                        if (this.mounted) {
                            this.fetchDebounce();
                        }

                        this.deferred = this.deferred || Q.defer();
                        result = this.deferred.promise;
                    }
                }

                return result;
            }

            fetchData () {
                const { store, relate_ssr } = this.context;
                const { dispatch } = store;
                const { fragments, variables, connectors, scopes } = this.bundle;
                this.bundle = {};
                this.deferred = this.deferred || Q.defer();
                if (fragments && Object.keys(fragments).length) {
                    this.fetchingData = true;
                    dispatch(
                        queryAction(
                            buildQueryAndVariables(fragments, variables),
                            fragments,
                            connectors,
                            scopes,
                            typeof config === 'function' ? config(this.props) : config,
                            relate_ssr
                        )
                    ).then((data) => {
                        this.deferred.resolve(data);
                    }).catch((err) => {
                        warning(false, err);
                        this.deferred.reject();
                    }).fin(() => {
                        this.deferred = null;
                        this.fetchingData = false;
                    });
                } else {
                    this.deferred.resolve();
                    this.fetchingData = false;
                }

                return this.deferred.promise;
            }

            render () {
                return <WrappedComponent {...this.props} />;
            }
        }

        return hoistStatics(RootConnectData, WrappedComponent, {
            relateIdentifier: true,
        });
    };
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/helpers/capture.js

```js
import _ from 'lodash';

/**
* Captures a relate query or mutation values (concats scoped queries)
*
* @param {Object} action
* @param {String||Array} test
*/
export default (action, test) => {
    if (!test) {
        return null;
    }

    const querieTestNames = test.constructor === Array ? test : [test];
    let result = null;

    _.forEach(action.data, (queryValue, key) => {
        if (queryValue && querieTestNames.indexOf(key) !== -1 ||
        querieTestNames.indexOf(action.scopes[key]) !== -1) {
            // is in query test values

            if (queryValue.constructor === Array) {
                result = result && [...result, ...queryValue] || [...queryValue];
            } else {
                result = queryValue;
            }
        }
    });

    return result;
};

```

* PDShop_Shop_PC/project/App/shared/relatejs/helpers/connectors-ids.js

```js
let ID_COUNTER = 0;
let PREFIX = '';

export function generateConnectorId () {
    return `${PREFIX}connector_${ID_COUNTER++}`;
}

export function resetConnectorsIds (prefix = '') {
    PREFIX = prefix;
    ID_COUNTER = 0;
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/helpers/fragments.js

```js
import _ from 'lodash';

//
// Example usage:
//
//     mergeFragments({
//       page: {
//         _id: 1,
//         name: 1,
//         createdBy: {
//           _id: 1
//         }
//       }
//     }, {
//       page: {
//         title: 1,
//         createdBy: {
//           name: 1
//         }
//       }
//     }))
//
// Outputs:
//
//     { page: { _id: 1, name: 1, createdBy: { _id: 1, name: 1 }, title: 1 } }
//
export function mergeFragments () {
    return _.merge({}, ...arguments);
}

//
// Example usage:
//
//     fragmentToQL({
//       _id: 1,
//       name: 1,
//       createdBy: {
//         _id: 1
//       }
//     })
//
// Outputs:
//
//     _id,name,createdBy { _id }
//
export function fragmentToQL (fragment) {
    const iterate = (i) => {
        return Object
        .keys(i)
        .map((key) => {
            let result;
            const value = i[key];
            if (typeof value === 'object') {
                result = `${key} { ${iterate(value)} }`;
            } else {
                result = key;
            }
            return result;
        })
        .join(',');
    };
    return iterate(fragment);
}

// Example usage:
//
//     const {query, variables} = buildQueryAndVariables(
//       {
//         session: {
//           _id: 1,
//           email: 1
//         },
//         page: {
//           _id: 1,
//           title: 1,
//           slug: 1,
//           createdBy: {
//             name: 1
//           }
//         }
//       },
//       {
//         page: {
//           slug: {
//             value: 'landing',
//             type: 'String!'
//           }
//         }
//       }
//     );
//
// Outputs:
//
// {
//   query: `
//     query ($slug0: String!) {
//       session {
//         _id,
//         email
//       },
//       page (slug: $slug0) {
//         _id,
//         title,
//         slug,
//         createdBy {
//           name
//         }
//       }
//     }
//   `,
//   variables: {
//     slug0: 'landing'
//   }
// }
//
let iterator;
let variables;
let variablesTypes;

function _buildQuery (fragments, inputVariables = {}) {
    const queries = [];

    _.forIn(fragments, (fragment, key) => {
        let queryStr = '';
        const vars = inputVariables[key];
        const variablesMap = []; // slug: $slug
        const hasFragments = typeof fragment === 'object';

        // variables calculation
        if (vars) {
            _.forIn(vars, (varValue, varKey) => {
                if (varValue.hasOwnProperty('type') && varValue.hasOwnProperty('value') && typeof varValue.type === 'string') {
                    const name = varKey + iterator;
                    variablesTypes.push(`$${name}: ${varValue.type}`);
                    variablesMap.push(`${varKey}: $${name}`);
                    variables[name] = varValue.value;
                    iterator++;
                }
            });
        }

        // Make query variables string
        let variablesString = '';
        if (variablesMap.length) {
            variablesString = ` (${variablesMap.join(',')})`;
        }

        queryStr += `${key}${variablesString}`;

        if (hasFragments) {
            queryStr += ` { ${_buildQuery(fragment, vars)} }`;
        }

        queries.push(queryStr);
    });

    return queries.join(',');
}

export function buildQueryAndVariables (fragments, inputVariables = {}, type = 'query') {
    variables = {};
    variablesTypes = [];
    iterator = 0;

    const queries = _buildQuery(fragments, inputVariables);

    let query;
    if (variablesTypes.length) {
        query = `${type} (${variablesTypes.join(',')}) { ${queries} }`;
    } else {
        query = `${type} { ${queries} }`;
    }

    return {
        query,
        variables,
    };
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/helpers/get-variables.js

```js
import invariant from 'invariant';
import _ from 'lodash';

export default function getVariables ({ variables, fragments, variablesTypes, displayName }) {
    const resultVariables = {};
    if (variables) {
        _.forEach(variables, (vars, queryName) => {
            if (fragments[queryName]) { // if not in fragments, ignore
                resultVariables[queryName] = {};
                const queryVariablesTypes = variablesTypes[queryName];

                // No variables types defined for this query
                invariant(
                    queryVariablesTypes,
                    'Relate: Query to %s doesn\'t have variables types defined in %s!',
                    queryName,
                    displayName || 'a component'
                );

                // Check if every variable has a type
                _.forEach(vars, (value, variable) => {
                    invariant(
                        queryVariablesTypes[variable],
                        'Relate: Query to %s does not have variable "%s" type defined in %s!',
                        queryName,
                        variable,
                        displayName || 'a component'
                    );

                    // add variable prepared for query e.g. {type: 'String', value: 'something'}
                    const type = queryVariablesTypes[variable];

                    if (typeof type === 'object') {
                        // deep query
                        Object.assign(resultVariables[queryName], getVariables({
                            variables: { [variable]: value },
                            fragments: { [variable]: fragments[queryName] },
                            variablesTypes: { [variable]: type },
                        }));
                    } else {
                        resultVariables[queryName][variable] = {
                            type: queryVariablesTypes[variable],
                            value,
                        };
                    }
                });

                // Check if every required variable type is met
                _.forEach(queryVariablesTypes, (type, variable) => {
                    if (typeof type !== 'object') {
                        invariant(
                            type.slice(-1) !== '!' || vars[variable] !== undefined,
                            'Relate: Query to %s requires the variable "%s" in %s!',
                            queryName,
                            variable,
                            displayName || 'a component'
                        );
                    }
                });
            }
        });
    }
    return resultVariables;
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/helpers/request.js

```js
import request from 'superagent';
import _ from 'lodash';
import Q from 'q';

function omitAll (obj) {
    if (_.isPlainObject(obj)) {
        return _.mapValues(obj, (o) => omitAll(o));
    } else if (_.isArray(obj)) {
        return _.map(obj, (o) => omitAll(o));
    } else if (obj === null) {
        return undefined;
    }
    return obj;
}

export default function doRequest ({ dispatch, query, variables, type, endpoint, headers, body, ...params }) {
    console.log('[relatejs]: doRequest', { dispatch, query, variables, type, endpoint, headers, body, ...params });
    return new Q()
    .then(() => {
        const deferred = Q.defer();
        let promise = deferred.promise;
        const dataObj = { query, variables, ...body };
        const payload =
        headers['Content-Type'] === 'text/plain' ?
        JSON.stringify(dataObj) :
        dataObj;

        const req = request
        .post(endpoint || '/shop/graphql')
        .set(headers)
        .send(payload);

        if (params.withCredentials) {
            req.withCredentials();
        }

        req
        .end((error, res) => {
            if (error) {
                console.error('[relatejs]:', res.text);
                deferred.reject(error);
            } else {
                const data = omitAll(res.body);
                console.log('[relatejs]: body', data);
                const { errors } = data;
                if (errors && errors.length && _.find(errors, (item) => (item.message === 'unauthorized'))) {
                    window.location.href = '/shop/login';
                } else {
                    deferred.resolve(data);
                }
            }
        });

        if (dispatch) {
            console.log('[relatejs]: start dispatch');
            promise = promise.then(({ data, errors }) => {
                console.log('[relatejs]: dispatch', { type, data, errors, ...params });
                dispatch({ type, data, errors, ...params });
                return data;
            });
        }

        return promise;
    });
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/redirect-graphql/index.js

```js
import { buildQueryAndVariables } from '../helpers/fragments';
import request from '../helpers/request';

const headers = {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
};

export function apiQuery (options, callback) {
    const mutation = buildQueryAndVariables(options.fragments, options.variables, 'query');
    const params = { ...mutation, headers };
    return () => {
        return request(params).then(({ data }) => {
            callback(data);
        });
    };
}

export function apiMutation (options, callback) {
    const mutation = buildQueryAndVariables(options.fragments, options.variables, 'mutation');
    const params = { ...mutation, headers };
    return () => {
        return request(params).then(({ data }) => {
            callback(data);
        });
    };
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/reducer/reducer.js

```js
import actionTypes from '../actions/types';
import Store from '../store';
import KeepData from '../store/keep-data';
import _ from 'lodash';

const store = new Store();

// Default state
// state will be composed of connectors data, e.g.
// {
//   connector_1: {
//     pages: [{...}, {...}]
//   },
//   connector_2: {
//     page: {...}
//   }
// }
const defaultState = {
    headers: {
        'Content-Type': 'application/json',
        Accept: 'application/json',
    },
    body: {},
    endpoint: '/shop/graphql',
    withCredentials: false,
};

export function relateReducer (_state = defaultState, action = {}) {
    const isIntrospection = action.isIntrospection;
    let state = _state;

    // Server store hydration
    if (!isIntrospection && state.serverConnectors) {
        store.connectors.connectors = state.serverConnectors;
        state = Object.assign({}, state);
        delete state.serverConnectors;
    }

    if ((action.type === actionTypes.query || action.type === actionTypes.mutation) &&
    action.data &&
    action.fragments) {
        const isMutation = action.type === actionTypes.mutation;

        const changes = store.processIncoming({
            data: action.data,
            fragments: action.fragments,
            connectors: action.connectors,
            isMutation,
        });

        let result;
        if (isIntrospection) {
            // is introspection (server side), so save connectors mapping for later re hydration
            const connectorsMap = {};
            _.forEach(action.connectors, (connectorData, connectorId) => {
                connectorsMap[connectorId] = {
                    fragments: connectorData.fragments,
                    variables: connectorData.variables,
                };
            });
            result = Object.assign({}, state, changes, {
                server: connectorsMap,
                serverConnectors: store.connectors.connectors,
            });
        } else {
            // client side
            result = Object.assign({}, state, changes);
        }

        return result;
    }

    if (action.type === actionTypes.removeConnector) {
        const newState = Object.assign({}, state);
        delete newState[action.id];
        KeepData.removeConnectorId(action.id);
        store.deleteConnector(action.id);

        if (state.server) {
            // server mapping no longer needed
            delete newState.server;
        }

        // TODO Delete no longer needed data from state? or maintain for future cache?
        return newState;
    }

    if (action.type === actionTypes.setHeader) {
        return Object.assign({}, state, {
            headers: Object.assign({}, state.headers, {
                [action.key]: action.value,
            }),
        });
    }

    if (action.type === actionTypes.removeHeader) {
        const headers = Object.assign({}, state.headers);
        delete headers[action.key];
        return Object.assign({}, state, {
            headers,
        });
    }

    if (action.type === actionTypes.setEndpoint) {
        return Object.assign({}, state, {
            endpoint: action.endpoint,
        });
    }

    if (action.type === actionTypes.setBody) {
        return Object.assign({}, state, {
            body: Object.assign({}, state.body, {
                [action.key]: action.value,
            }),
        });
    }

    if (action.type === actionTypes.removeBody) {
        const body = Object.assign({}, state.body);
        delete body[action.key];
        return Object.assign({}, state, {
            body,
        });
    }

    return state;
}

export function relateReducerInit (settings) {
    Object.assign(defaultState, settings);
    return relateReducer;
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/server/get-data-dependencies.js

```js
import invariant from 'invariant';
import q from 'q';
import React from 'react';

import { resetConnectorsIds } from '../helpers/connectors-ids';

function processElement ({ element, context, rootDataConnectors, dataConnectors }) {
    try {
        if (element !== null) {
            const { props, type } = element;
            if (typeof type === 'function') {
                const ElementClass = element.type;
                let finalProps = props;
                if (ElementClass.defaultProps) {
                    finalProps = Object.assign({}, ElementClass.defaultProps, props);
                }
                // console.log("=============1", element);
                // console.log("=============2", ElementClass);
                const Element = new ElementClass(
                    finalProps,
                    context
                );

                if (type.relateIdentifier === 'ROOT_DATA_CONNECT') {
                    rootDataConnectors.push(Element);
                } else if (type.relateIdentifier === 'DATA_CONNECT') {
                    dataConnectors.push(Element);
                }

                // Generate context for children
                let newContext = context;
                if (Element.getChildContext) {
                    newContext = Object.assign({}, context, Element.getChildContext());
                }
                // console.log("=============3", !!Element.render);
                // go through children
                const renderResult = Element.render ? Element.render() : ElementClass(finalProps, context);
                // console.log("=============4", renderResult);
                processElement({
                    element: renderResult,
                    context: newContext,
                    rootDataConnectors,
                    dataConnectors,
                });
            } else if (props && props.children) {
                React.Children.forEach(props.children, (childElement) => {
                    processElement({
                        element: childElement,
                        context,
                        rootDataConnectors,
                        dataConnectors,
                    });
                });
            }
        }
    } catch (err) {
        console.log('=============5', err);
        invariant(false, 'Relate: error traversing components tree');
    }
}

export default function getAllDataDependencies (rootElement, getData) {
    const rootDataConnectors = [];
    const dataConnectors = [];

    // Ensure connectors ids are reset
    resetConnectorsIds('server_');

    // traverse tree
    processElement({
        element: rootElement,
        context: {
            relate_ssr: getData,
        },
        rootDataConnectors,
        dataConnectors,
    });

    // fetch data for each root data connector
    return q()
    .then(() => {
        let result;

        if (rootDataConnectors.length) {
            result = rootDataConnectors[0].fetchData();
        } else {
            result = null;
        }

        return result;
    })
    .then(() => resetConnectorsIds())
    .catch(() => {
        invariant(false, 'Relate: error getting data');
    });
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/store/connectors.js

```js
import _ from 'lodash';

export default class Connectors {
    constructor () {
        this.connectors = {};
    }

    connectorExists (connectorId) {
        return this.connectors[connectorId] && true;
    }

    generateConnectorData (connectorId) {
        const connector = this.connectors[connectorId];
        return connector.data;
    }

    // Process a query through the connectors that requested it
    processConnectors (queryConnectors, queryName, relativeNodes) {
        console.log('[relatejs]: processConnectors', { queryConnectors, queryName, relativeNodes });
        _.forEach(queryConnectors, (connectorQuery, connectorId) => {
            // Check if queryName is a scoped one
            const actualQueryName = connectorQuery.scopes && connectorQuery.scopes[queryName] || queryName;
            const isInConnectorFragments = connectorQuery.fragments[actualQueryName] && true;
            const isScopedQuery = actualQueryName !== queryName;
            const scopedOther =
            !isScopedQuery && // is not a scoped query
            connectorQuery.scopes && // has scopes
            _.values(connectorQuery.scopes).indexOf(actualQueryName) !== -1; // query has not been scoped

            // check if query was triggered by this connector
            // 1st case for scoped queries
            // 2nd case for non scope queries (they cannot be a value in scope)
            if ((isInConnectorFragments && isScopedQuery) ||
            (isInConnectorFragments && !isScopedQuery && !scopedOther)) {
                this.connectors[connectorId] = this.connectors[connectorId] || {
                    data: {},
                    fragments: {},
                    mutations: {},
                };
                const conn = this.connectors[connectorId];
                const { loadingMoreProperty, loadingPageProperty } = connectorQuery;
                // Add query data
                if (loadingMoreProperty) {
                    if (loadingMoreProperty === true) {
                        conn.data[actualQueryName] = [...conn.data[actualQueryName], ...relativeNodes];
                    } else {
                        _.set(conn.data[actualQueryName], loadingMoreProperty, [..._.get(conn.data[actualQueryName], loadingMoreProperty), ..._.get(relativeNodes, loadingMoreProperty)]);
                    }
                } else if (loadingPageProperty) {
                    let index = connectorQuery.loadingPageStartIndex;
                    if (loadingPageProperty === true) {
                        const list = conn.data[actualQueryName] || [];
                        _.forEach(relativeNodes, (item) => {
                            list[index++] = item;
                        });
                        for (let i = 0; i < index; i++) {
                            if (list[i] === undefined) {
                                list[i] = false;
                            }
                        }
                        conn.data[actualQueryName] = list;
                    } else {
                        const list = _.get(conn.data[actualQueryName], loadingPageProperty) || [];
                        _.forEach(_.get(relativeNodes, loadingPageProperty), (item) => {
                            list[index++] = item;
                        });
                        for (let i = 0; i < index; i++) {
                            if (list[i] === undefined) {
                                list[i] = false;
                            }
                        }
                        _.set(conn.data[actualQueryName], loadingPageProperty, list);
                    }
                } else {
                    Object.assign(conn.data, { [actualQueryName]: relativeNodes });
                }

                // fragments
                Object.assign(conn.fragments, {
                    [actualQueryName]: connectorQuery.fragments[actualQueryName],
                });

                // mutations listeners
                conn.mutations = connectorQuery.mutations || {};
            }
        });
    }

    // Check if any connector is listening to the mutation passed as argument
    checkMutationListeners (mutationName, relativeNodes) {
        const connsToUpdate = [];
        console.log('[relatejs]: checkMutationListeners', { mutationName, relativeNodes, connectors: this.connectors });
        _.forEach(this.connectors, (connector, connectorId) => {
            const mutation = connector.mutations && connector.mutations[mutationName];
            if (mutation && mutation.constructor === Function) {
                connsToUpdate.push(connectorId);
                mutation({ state:connector.data, data:relativeNodes, connector, _ });
            }
        });
        return connsToUpdate;
    }

    deleteConnector (connectorId) {
        this.connectors[connectorId] && delete this.connectors[connectorId];
    }
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/store/global-data.js

```js
export default {
    data: {},
    get (key, value) {
        const ret = this.data[key];
        if (value !== undefined) {
            this.data[key] = value;
        }
        return ret;
    },
    set (key, value) {
        this.data[key] = value;
    },
};

```

* PDShop_Shop_PC/project/App/shared/relatejs/store/index.js

```js
import _ from 'lodash';
import Connectors from './connectors';

export default class Store {
    constructor () {
        // Connectors interface
        this.connectors = new Connectors();
    }

    // #isMutation - Boolean
    processIncoming ({ data, fragments, connectors, isMutation }) {
        console.log('[relatejs]: processIncoming', { data, fragments, connectors, isMutation });
        let connectorsToUpdate = connectors && Object.keys(connectors) || [];

        _.forEach(fragments, (fragment, queryName) => {
            const relativeNodes = data[queryName];

            // Process data to connectors
            if (connectors) {
                this.connectors.processConnectors(
                    connectors,
                    queryName,
                    relativeNodes,
                );
            }

            // If mutation check if some connector is listening
            if (isMutation) {
                connectorsToUpdate = _.union(
                    connectorsToUpdate,
                    this.connectors.checkMutationListeners(queryName, relativeNodes)
                );
            }
        });

        // Calculate connectors that need to be changed
        const changes = {};
        _.forEach(connectorsToUpdate, (connectorId) => {
            if (this.connectors.connectorExists(connectorId)) {
                changes[connectorId] = this.connectors.generateConnectorData(connectorId);
            }
        });
        return changes;
    }

    // Remove a data connector
    deleteConnector (connectorId) {
        this.connectors.deleteConnector(connectorId);
    }
}

```

* PDShop_Shop_PC/project/App/shared/relatejs/store/keep-data.js

```js
import _ from 'lodash';
import removeConnector from '../actions/remove-connector';

export default {
    list: {},
    redirectPathnameList: {},
    update (location, data, connectorId, store, pathname) {
        if (pathname) {
            this.redirectPathnameList[location] = pathname;
            location = pathname;
        }
        if (data) {
            if (this.list[location] && typeof data === 'object') {
                this.list[location].data = Object.assign(this.list[location].data || {}, data, _.omitBy({ connectorId, store, hasKeepData: true }, _.isNil));
            } else {
                let obj = { connectorId, store, hasKeepData: true };
                if (typeof data === 'object') {
                    obj.data = data;
                }
                this.list[location] = obj;
            }
        } else if (this.list[location]) {
            delete this.list[location];
        }
    },
    get (location, props) {
        location = this.redirectPathnameList[location] || location;
        let obj = this.list[location] || {};
        if (!props.keepLastKeepData) {
            _.forEach(this.list, (o, key) => {
                if (key !== location) {
                    o.store.dispatch(removeConnector(o.connectorId));
                }
            });
        }
        return obj;
    },
    getHasKeepData (location) {
        location = this.redirectPathnameList[location] || location;
        return (this.list[location] || {}).hasKeepData;
    },
    getKeepData (location) {
        location = this.redirectPathnameList[location] || location;
        return (this.list[location] || {}).data;
    },
    removeConnectorId (connectorId) {
        const item = _.find(this.list, (o) => o.connectorId === connectorId);
        if (item) {
            if (item.onlyClearConnector) {
                delete item.onlyClearConnector;
                delete item.connectorId;
            } else {
                this.list = _.omitBy(this.list, (o) => o.connectorId === connectorId);
            }
        }
    },
    // 只清除location的所有数据
    forceClear (location) {
        location = this.redirectPathnameList[location] || location;
        const obj = this.list[location];
        if (obj) {
            obj.store.dispatch(removeConnector(obj.connectorId));
        }
    },
    // 只清除location的connectorId， 其他的数据保留
    forceClearConnector (location) {
        location = this.redirectPathnameList[location] || location;
        const obj = this.list[location];
        if (obj) {
            obj.onlyClearConnector = true;
            obj.store.dispatch(removeConnector(obj.connectorId));
        }
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/agent/checkAgentByChairManPhone.js

```js
import {
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { agentResultType } from '../../types/agent';
import { post, urls } from 'helpers/api';

export default {
    type: agentResultType,
    args: {
        chairManPhone: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.checkAgentByChairManPhone, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/agent/createAgent.js

```js
import { authorize } from '../../authorize';
import { agentInputType, agentResultType } from '../../types/agent';
import { post, urls } from 'helpers/api';

export default {
    type: agentResultType,
    args: {
        data: {
            type: agentInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.createAgent, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/agent/index.js

```js
export createAgent from './createAgent';
export modifyAgent from './modifyAgent';
export removeAgent from './removeAgent';
export checkAgentByChairManPhone from './checkAgentByChairManPhone';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/agent/modifyAgent.js

```js
import { authorize } from '../../authorize';
import { agentInputType, agentResultType } from '../../types/agent';
import { post, urls } from 'helpers/api';

export default {
    type: agentResultType,
    args: {
        data: {
            type: agentInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyAgent, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/agent/removeAgent.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        agentId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.removeAgent, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/member/createMember.js

```js
import { authorize } from '../../authorize';
import { memberInputType, memberResultType } from '../../types/member';
import { post, urls } from 'helpers/api';

export default {
    type: memberResultType,
    args: {
        data: {
            type: memberInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.createMember, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/member/index.js

```js
export createMember from './createMember';
export modifyMember from './modifyMember';
export removeMember from './removeMember';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/member/modifyMember.js

```js
import { authorize } from '../../authorize';
import { memberInputType, memberResultType } from '../../types/member';
import { post, urls } from 'helpers/api';

export default {
    type: memberResultType,
    args: {
        data: {
            type: memberInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyMember, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/member/removeMember.js

```js
import {
    GraphQLID,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        memberId: {
            type: GraphQLID,
        },
        password: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.removeMember, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/notify/index.js

```js
export sendNotify from './sendNotify';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/notify/sendNotify.js

```js
import { notifyResultType, notifyInputType } from '../../types/notify';
import { post, urls } from 'helpers/api';

export default {
    type: notifyResultType,
    args: {
        data: {
            type: notifyInputType,
        },
    },
    async resolve (root, params, options) {
        params.data.source = '物流超市平台';
        params.data.memberId = root.user.userId;
        return await post(urls.sendNotify, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/index.js

```js
export placeOriginOrder from './placeOriginOrder';
export placeOriginOrderWithPreOrder from './placeOriginOrderWithPreOrder';
export modifyOriginOrder from './modifyOriginOrder';
export selectRoadmap from './selectRoadmap';
export selectRoadmapForOrderList from './selectRoadmapForOrderList';
export proxyPay from './proxyPay';
export proxyPayForOrderList from './proxyPayForOrderList';
export printOrderBill from './printOrderBill';
export printOrderListBill from './printOrderListBill';
export printBarCode from './printBarCode';
export removeOrder from './removeOrder';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/modifyOriginOrder.js

```js
import { authorize } from '../../authorize';
import { orderInputType, orderResultType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderResultType,
    args: {
        data: {
            type: orderInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyOriginOrder, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/placeOriginOrder.js

```js
import { authorize } from '../../authorize';
import { orderInputType, orderResultType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderResultType,
    args: {
        data: {
            type: orderInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.placeOriginOrder, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/placeOriginOrderWithPreOrder.js

```js
import { authorize } from '../../authorize';
import { orderInputType, orderResultType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderResultType,
    args: {
        data: {
            type: orderInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.placeOriginOrderWithPreOrder, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/printBarCode.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderResultType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderResultType,
    args: {
        orderId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.printBarCode, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/printOrderBill.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderResultType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderResultType,
    args: {
        orderId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.printOrderBill, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/printOrderListBill.js

```js
import {
    GraphQLID,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        orderIdList: {
            type: new GraphQLList(GraphQLID),
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.printOrderListBill, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/proxyPay.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderResultType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderResultType,
    args: {
        orderId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.proxyPay, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/proxyPayForOrderList.js

```js
import {
    GraphQLID,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        orderIdList: {
            type: new GraphQLList(GraphQLID),
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.proxyPayForOrderList, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/removeOrder.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        orderId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.removeOrder, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/selectRoadmap.js

```js
import { authorize } from '../../authorize';
import { orderInputType, orderResultType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderResultType,
    args: {
        data: {
            type: orderInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.selectRoadmap, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/order/selectRoadmapForOrderList.js

```js
import {
    GraphQLID,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        orderIdList: {
            type: new GraphQLList(GraphQLID),
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.selectRoadmapForOrderList, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/partment/createPartment.js

```js
import { authorize } from '../../authorize';
import { partmentInputType, partmentResultType } from '../../types/partment';
import { post, urls } from 'helpers/api';

export default {
    type: partmentResultType,
    args: {
        data: {
            type: partmentInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.createPartment, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/partment/index.js

```js
export createPartment from './createPartment';
export modifyPartment from './modifyPartment';
export removePartment from './removePartment';
export modifyOwnPartment from './modifyOwnPartment';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/partment/modifyOwnPartment.js

```js
import { authorize } from '../../authorize';
import { partmentInputType, partmentResultType } from '../../types/partment';
import { post, urls } from 'helpers/api';

export default {
    type: partmentResultType,
    args: {
        data: {
            type: partmentInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyOwnPartment, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/partment/modifyPartment.js

```js
import { authorize } from '../../authorize';
import { partmentInputType, partmentResultType } from '../../types/partment';
import { post, urls } from 'helpers/api';

export default {
    type: partmentResultType,
    args: {
        data: {
            type: partmentInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyPartment, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/partment/removePartment.js

```js
import {
    GraphQLID,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        partmentId: {
            type: GraphQLID,
        },
        password: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.removePartment, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/personal/feedback.js

```js
import {
    GraphQLString,
} from 'graphql';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        content: {
            type: GraphQLString,
        },
        email: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.feedback, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/personal/forgotPwd.js

```js
import { successType } from '../../types/common';
import { memberInputType } from '../../types/member';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        data: {
            type: memberInputType,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.forgotPwd, params.data) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/personal/index.js

```js
export register from './register';
export forgotPwd from './forgotPwd';
export setPaymentPassword from './setPaymentPassword';
export modifyPersonalInfo from './modifyPersonalInfo';
export feedback from './feedback';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/personal/modifyPersonalInfo.js

```js
import { authorize } from '../../authorize';
import { memberResultType, memberInputType } from '../../types/member';
import { post, urls } from 'helpers/api';

export default {
    type: memberResultType,
    args: {
        data: {
            type: memberInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyPersonalInfo, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/personal/register.js

```js
import { successType } from '../../types/common';
import { memberInputType } from '../../types/member';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        data: {
            type: memberInputType,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.register, params.data) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/personal/setPaymentPassword.js

```js
import { successType } from '../../types/common';
import { memberInputType } from '../../types/member';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        data: {
            type: memberInputType,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.setPaymentPassword, params.data) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/roadmap/addRoadmapRegionProfitRate.js

```js
import { authorize } from '../../authorize';
import { regionRateInputType, regionRateResultType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: regionRateResultType,
    args: {
        data: {
            type: regionRateInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.addRoadmapRegionProfitRate, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/roadmap/index.js

```js
export setRoadmapProfit from './setRoadmapProfit';
export setRegionRateProfit from './setRegionRateProfit';
export addRoadmapRegionProfitRate from './addRoadmapRegionProfitRate';
export modifyRoadmapRegionProfitRate from './modifyRoadmapRegionProfitRate';
export removeRoadmapRegionProfitRate from './removeRoadmapRegionProfitRate';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/roadmap/modifyRoadmapRegionProfitRate.js

```js
import { authorize } from '../../authorize';
import { regionRateInputType, regionRateResultType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: regionRateResultType,
    args: {
        data: {
            type: regionRateInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyRoadmapRegionProfitRate, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/roadmap/removeRoadmapRegionProfitRate.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        regionRateId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.removeRoadmapRegionProfitRate, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/roadmap/setRegionRateProfit.js

```js
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { regionRateInputType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        data: {
            type: regionRateInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.setRegionRateProfit, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/roadmap/setRoadmapProfit.js

```js
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { roadmapInputType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        data: {
            type: roadmapInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.setRoadmapProfit, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/setting/index.js

```js
export modifySetting from './modifySetting';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/setting/modifySetting.js

```js
import { authorize } from '../../authorize';
import { settingInputType, settingResultType } from '../../types/setting';
import { post, urls } from 'helpers/api';

export default {
    type: settingResultType,
    args: {
        data: {
            type: settingInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifySetting, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/shipper/createAndRegisterShipper.js

```js
import { authorize } from '../../authorize';
import { shipperResultType, shipperInputType } from '../../types/shipper';
import { post, urls } from 'helpers/api';

export default {
    type: shipperResultType,
    args: {
        data: {
            type: shipperInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.createAndRegisterShipper, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/shipper/index.js

```js
export registerShipper from './registerShipper';
export createAndRegisterShipper from './createAndRegisterShipper';
export modifyShipper from './modifyShipper';
export passExamineTruck from './passExamineTruck';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/shipper/modifyShipper.js

```js
import { authorize } from '../../authorize';
import { shipperResultType, shipperInputType } from '../../types/shipper';
import { post, urls } from 'helpers/api';

export default {
    type: shipperResultType,
    args: {
        data: {
            type: shipperInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyShipper, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/shipper/passExamineTruck.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        truckId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.passExamineTruck, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/shipper/registerShipper.js

```js
import { authorize } from '../../authorize';
import { shipperResultType, shipperInputType } from '../../types/shipper';
import { post, urls } from 'helpers/api';

export default {
    type: shipperResultType,
    args: {
        data: {
            type: shipperInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.registerShipper, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/shop/createBranchShop.js

```js
import { authorize } from '../../authorize';
import { shopInputType, shopResultType } from '../../types/shop';
import { post, urls } from 'helpers/api';

export default {
    type: shopResultType,
    args: {
        data: {
            type: shopInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.createBranchShop, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/shop/index.js

```js
export modifyOwnShop from './modifyOwnShop';
export createBranchShop from './createBranchShop';
export modifyBranchShop from './modifyBranchShop';
export removeBranchShop from './removeBranchShop';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/shop/modifyBranchShop.js

```js
import { authorize } from '../../authorize';
import { shopInputType, shopResultType } from '../../types/shop';
import { post, urls } from 'helpers/api';

export default {
    type: shopResultType,
    args: {
        data: {
            type: shopInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyBranchShop, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/shop/modifyOwnShop.js

```js
import { authorize } from '../../authorize';
import { shopInputType, shopResultType } from '../../types/shop';
import { post, urls } from 'helpers/api';

export default {
    type: shopResultType,
    args: {
        data: {
            type: shopInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyOwnShop, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/shop/removeBranchShop.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        shopId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.removeBranchShop, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/warehouse/createWarehouse.js

```js
import { authorize } from '../../authorize';
import { warehouseInputType, warehouseResultType } from '../../types/warehouse';
import { post, urls } from 'helpers/api';

export default {
    type: warehouseResultType,
    args: {
        data: {
            type: warehouseInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.createWarehouse, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/warehouse/index.js

```js
export createWarehouse from './createWarehouse';
export modifyWarehouse from './modifyWarehouse';
export removeWarehouse from './removeWarehouse';

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/warehouse/modifyWarehouse.js

```js
import { authorize } from '../../authorize';
import { warehouseInputType, warehouseResultType } from '../../types/warehouse';
import { post, urls } from 'helpers/api';

export default {
    type: warehouseResultType,
    args: {
        data: {
            type: warehouseInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyWarehouse, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/mutations/warehouse/removeWarehouse.js

```js
import { authorize } from '../../authorize';
import { warehouseResultType } from '../../types/warehouse';
import { post, urls } from 'helpers/api';
import { GraphQLID, GraphQLString } from 'graphql';

export default {
    type: warehouseResultType,
    args: {
        warehouseId: {
            type: GraphQLID,
        },
        password: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.removeWarehouse, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/account/bills.js

```js
import {
    GraphQLObjectType,
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { billItemListType } from '../../types/bill';
import { post, urls } from 'helpers/api';

const billsType = new GraphQLObjectType({
    name: 'billsType',
    fields: {
        recharge: { type: billItemListType },
        withdraw: { type: billItemListType },
        income: { type: billItemListType },
        pay: { type: billItemListType },
    },
});

export default {
    type: billsType,
    args: {
        type: {
            type: GraphQLString,
        },
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
        startDate: {
            type: GraphQLString,
        },
        endDate: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.bills, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/account/index.js

```js
export remainAmount from './remainAmount';
export weixinPayGetUnifiedOrder from './weixinPayGetUnifiedOrder';
export weixinPayWithDraw from './weixinPayWithDraw';
export bills from './bills';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/account/remainAmount.js

```js
import { authorize } from '../../authorize';
import { accountType } from '../../types/account';
import { post, urls } from 'helpers/api';

export default {
    type: accountType,
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.remainAmount, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/account/weixinPayGetUnifiedOrder.js

```js
import { authorize } from '../../authorize';
import { accountResultType, accountInputType } from '../../types/account';
import { post, urls } from 'helpers/api';

export default {
    type: accountResultType,
    args: {
        data: {
            type: accountInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.weixinPayGetUnifiedOrder, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/account/weixinPayWithDraw.js

```js
import { authorize } from '../../authorize';
import { accountResultType, accountInputType } from '../../types/account';
import { post, urls } from 'helpers/api';

export default {
    type: accountResultType,
    args: {
        data: {
            type: accountInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.weixinPayWithDraw, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/agent/agent.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { agentType } from '../../types/agent';
import { post, urls } from 'helpers/api';

export default {
    type: agentType,
    args: {
        agentId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let agent = await post(urls.agent, params, root) || {};
        return agent.context || {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/agent/agents.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { agentListType } from '../../types/agent';
import { post, urls } from 'helpers/api';

export default {
    type: agentListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.agents, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/agent/index.js

```js
export agent from './agent';
export agents from './agents';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/client/client.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { clientType } from '../../types/client';
import { post, urls } from 'helpers/api';

export default {
    type: clientType,
    args: {
        clientId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.client, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/client/clients.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { clientListType } from '../../types/client';
import { post, urls } from 'helpers/api';

export default {
    type: clientListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.clients, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/client/getClientNameByPhone.js

```js
import {
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { clientResultType } from '../../types/client';
import { post, urls } from 'helpers/api';

export default {
    type: clientResultType,
    args: {
        phone: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.getClientNameByPhone, params, root) || {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/client/index.js

```js
export client from './client';
export clients from './clients';
export getClientNameByPhone from './getClientNameByPhone';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/common/address.js

```js
import {
    GraphQLInt,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { addressType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: new GraphQLList(addressType),
    args: {
        parentCode: {
            type: GraphQLInt,
        },
        type: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.address, params, root) || {};
        return ret.success ? ret.context.addressList : [];
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/common/addressFromLastCode.js

```js
import {
    GraphQLInt,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { addressType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: new GraphQLList(new GraphQLList(addressType)),
    args: {
        addressLastCode: {
            type: GraphQLInt,
        },
        type: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.addressFromLastCode, params, root) || {};
        return ret.success ? ret.context.addressList : [];
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/common/addressWithOrder.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderAddressType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: orderAddressType,
    args: {
        orderId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.addressWithOrder, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/common/index.js

```js
export address from './address';
export addressWithOrder from './addressWithOrder';
export addressFromLastCode from './addressFromLastCode';
export sendDoorAddressWithOrder from './sendDoorAddressWithOrder';
export sendDoorAddressFromLastCode from './sendDoorAddressFromLastCode';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/common/sendDoorAddressFromLastCode.js

```js
import {
    GraphQLList,
    GraphQLInt,
} from 'graphql';
import { authorize } from '../../authorize';
import { addressType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: new GraphQLList(new GraphQLList(addressType)),
    args: {
        addressLastCode: {
            type: GraphQLInt,
        },
        parentCode: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.sendDoorAddressFromLastCode, params, root) || {};
        return ret.success ? ret.context.addressList : [];
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/common/sendDoorAddressWithOrder.js

```js
import {
    GraphQLList,
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { addressType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: new GraphQLList(new GraphQLList(addressType)),
    args: {
        orderId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.sendDoorAddressWithOrder, params, root) || {};
        return ret.success ? ret.context.addressList : [];
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/member/index.js

```js
export members from './members';
export member from './member';
export wareHouseMembers from './wareHouseMembers';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/member/member.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { memberType } from '../../types/member';
import { post, urls } from 'helpers/api';

export default {
    type: memberType,
    args: {
        memberId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.member, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/member/members.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { memberListType } from '../../types/member';
import { post, urls } from 'helpers/api';

export default {
    type: memberListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.members, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/member/warehouseMembers.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { memberListType } from '../../types/member';
import { post, urls } from 'helpers/api';

export default {
    type: memberListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.warehouseMemberList, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/notify/index.js

```js
export notifies from './notifies';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/notify/notifies.js

```js
import {
    GraphQLObjectType,
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { notifyListType } from '../../types/notify';
import { post, urls } from 'helpers/api';

const notifiesType = new GraphQLObjectType({
    name: 'notifiesType',
    fields: {
        news: { type: notifyListType },
        publicity: { type: notifyListType },
        policy: { type: notifyListType },
        notice: { type: notifyListType },
    },
});

export default {
    type: notifiesType,
    args: {
        type: {
            type: GraphQLString,
        },
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.notifies, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/partment/index.js

```js
export partments from './partments';
export partment from './partment';
export ownPartment from './ownPartment';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/partment/ownPartment.js

```js
import { authorize } from '../../authorize';
import { partmentType } from '../../types/partment';
import { post, urls } from 'helpers/api';

export default {
    type: partmentType,
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.ownPartment, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/partment/partment.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { partmentType } from '../../types/partment';
import { post, urls } from 'helpers/api';

export default {
    type: partmentType,
    args: {
        partmentId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.partment, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/partment/partments.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { partmentListType } from '../../types/partment';
import { post, urls } from 'helpers/api';

export default {
    type: partmentListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.partments, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/order/index.js

```js
export order from './order';
export orders from './orders';
export rporder from './rporder';
export rporders from './rporders';
export whorders from './whorders';
export selectPreOrders from './selectPreOrders';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/order/order.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderType,
    args: {
        orderId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.order, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/order/orders.js

```js
import {
    GraphQLObjectType,
    GraphQLInt,
    GraphQLID,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderItemListType } from '../../types/order';
import { post, urls } from 'helpers/api';

const ordersType = new GraphQLObjectType({
    name: 'ordersType',
    fields: {
        toselectroadmap: { type: orderItemListType },
        toprintbarcode: { type: orderItemListType },
        topay: { type: orderItemListType },
        toprintbill: { type: orderItemListType },
        tostore: { type: orderItemListType },
        stored: { type: orderItemListType },
        tostartoff: { type: orderItemListType },
        onway: { type: orderItemListType },
        success: { type: orderItemListType },
    },
});

export default {
    type: ordersType,
    args: {
        type: {
            type: GraphQLString,
        },
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
        shopId: {
            type: GraphQLID,
        },
        startDate: {
            type: GraphQLString,
        },
        endDate: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.orders, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/order/rporder.js

```js
import { authorize } from '../../authorize';
import { orderType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderType,
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.rporder, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/order/rporders.js

```js
import {
    GraphQLObjectType,
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderItemListType } from '../../types/order';
import { post, urls } from 'helpers/api';

const rpordersType = new GraphQLObjectType({
    name: 'rpordersType',
    fields: {
        toselectroadmap: { type: orderItemListType },
        topay: { type: orderItemListType },
        toprintbill: { type: orderItemListType },
        tostore: { type: orderItemListType },
        stored: { type: orderItemListType },
    },
});

export default {
    type: rpordersType,
    args: {
        type: {
            type: GraphQLString,
        },
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
        startDate: {
            type: GraphQLString,
        },
        endDate: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.rporders, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/order/selectPreOrders.js

```js
import {
    GraphQLInt,
    GraphQLString,
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderListType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        senderPhone: {
            type: GraphQLID,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.selectPreOrders, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/order/whorders.js

```js
import {
    GraphQLObjectType,
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderItemListType } from '../../types/order';
import { post, urls } from 'helpers/api';

const whordersType = new GraphQLObjectType({
    name: 'whordersType',
    fields: {
        stored: { type: orderItemListType },
        loaded: { type: orderItemListType },
    },
});

export default {
    type: whordersType,
    args: {
        type: {
            type: GraphQLString,
        },
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
        startDate: {
            type: GraphQLString,
        },
        endDate: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.whorders, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/setting/index.js

```js
export setting from './setting';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/setting/setting.js

```js
import { authorize } from '../../authorize';
import { settingType } from '../../types/setting';
import { post, urls } from 'helpers/api';

export default {
    type: settingType,
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.setting, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/personal/getVerifyCode.js

```js
import {
    GraphQLString,
} from 'graphql';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        phone: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.getVerifyCode, params) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/personal/index.js

```js
export session from './session';
export login from './login';
export personal from './personal';
export getVerifyCode from './getVerifyCode';
export statistics from './statistics';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/personal/login.js

```js
import {
    GraphQLString,
} from 'graphql';
import passport from 'passport';
import { successType } from '../../types/common';

export default {
    type: successType,
    args: {
        phone: {
            type: GraphQLString,
        },
        password: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        return new Promise((resolve) => {
            const { req } = root;
            req.body = params;
            console.log('[login]: start', params);
            passport.authenticate('local', (err, user) => {
                console.log('[login]: authenticate', err, user);
                if (err) {
                    resolve({ msg: err.message });
                } else {
                    req.logIn(user, (error) => {
                        console.log('[login] logIn:', error);
                        if (error) {
                            resolve({ msg: '服务器错误' });
                        } else {
                            resolve({ success: true });
                        }
                    });
                }
            })(req);
        });
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/personal/personal.js

```js
import { authorize } from '../../authorize';
import { memberType } from '../../types/member';
import { post, urls } from 'helpers/api';

export default {
    type: memberType,
    async resolve (root, params, options) {
        authorize(root);
        let personal = await post(urls.personal, params, root) || {};
        return personal.context || {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/personal/session.js

```js
import { GraphQLBoolean } from 'graphql';

export default {
    type: GraphQLBoolean,
    resolve (root) {
        return root.isAuthenticated;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/personal/statistics.js

```js
import {
    GraphQLObjectType,
    GraphQLInt,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { post, urls } from 'helpers/api';

const releaseCardInfoType = new GraphQLObjectType({
    name: 'releaseCardInfoType',
    fields: {
        today: { type: GraphQLInt },
        past: { type: GraphQLInt },
        remain: { type: GraphQLInt },
    },
});
const consumeInfoType = new GraphQLObjectType({
    name: 'consumeInfoType',
    fields: {
        today: { type: new GraphQLList(GraphQLInt) },
        week: { type: new GraphQLList(GraphQLInt) },
        month: { type: new GraphQLList(GraphQLInt) },
    },
});
const userSexInfoType = new GraphQLObjectType({
    name: 'userSexInfoType',
    fields: {
        male: { type: GraphQLInt },
        female: { type: GraphQLInt },
    },
});
const statisticsType = new GraphQLObjectType({
    name: 'statisticsType',
    fields: {
        releaseCardInfo: { type: releaseCardInfoType },
        consumeInfo: { type: consumeInfoType },
        userSexInfo: { type: userSexInfoType },
        userAgeInfo: { type: new GraphQLList(GraphQLInt) },
    },
});

export default {
    type: statisticsType,
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.statistics, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/shipper/getShipperByChairManPhone.js

```js
import {
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { shipperResultType } from '../../types/shipper';
import { post, urls } from 'helpers/api';

export default {
    type: shipperResultType,
    args: {
        chairManPhone: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.getShipperByChairManPhone, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/shipper/index.js

```js
export shipper from './shipper';
export shippers from './shippers';
export getShipperByChairManPhone from './getShipperByChairManPhone';
export trucks from './trucks';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/shipper/shipper.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { shipperType } from '../../types/shipper';
import { post, urls } from 'helpers/api';

export default {
    type: shipperType,
    args: {
        shipperId: {
            type: GraphQLID,
        },
        shopId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.shipper, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/shipper/shippers.js

```js
import {
    GraphQLInt,
    GraphQLID,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { shipperListType } from '../../types/shipper';
import { post, urls } from 'helpers/api';

export default {
    type: shipperListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        shopId: {
            type: GraphQLID,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.shippers, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/shipper/trucks.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { truckListType } from '../../types/truck';
import { post, urls } from 'helpers/api';

export default {
    type: truckListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.trucks, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/shop/branchShop.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { shopType } from '../../types/shop';
import { post, urls } from 'helpers/api';

export default {
    type: shopType,
    args: {
        shopId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        if (!params.shopId) {
            return undefined;
        }
        let shop = await post(urls.branchShop, params, root) || {};
        return shop.context || undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/shop/branchShops.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { branchShopListType } from '../../types/shop';
import { post, urls } from 'helpers/api';

export default {
    type: branchShopListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.branchShops, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/shop/index.js

```js
export shop from './shop';
export branchShop from './branchShop';
export branchShops from './branchShops';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/shop/shop.js

```js
import { authorize } from '../../authorize';
import { shopType } from '../../types/shop';
import { post, urls } from 'helpers/api';

export default {
    type: shopType,
    async resolve (root, params, options) {
        authorize(root);
        let shop = await post(urls.shop, params, root) || {};
        return shop.context || {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/statistic/index.js

```js
export shopStatistics from './shopStatistics';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/statistic/shopStatistics.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { shopStatisticType } from '../../types/statistic';
import { post, urls } from 'helpers/api';

export default {
    type: shopStatisticType,
    args: {
        shopId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.shopStatistics, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/warehouse/index.js

```js
export warehouses from './warehouses';
export warehouse from './warehouse';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/warehouse/warehouse.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { warehouseType } from '../../types/warehouse';
import { post, urls } from 'helpers/api';

export default {
    type: warehouseType,
    args: {
        warehouseId:{
            type:GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.getWarehouseDetail, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/warehouse/warehouses.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { warehouseListType } from '../../types/warehouse';
import { post, urls } from 'helpers/api';

export default {
    type: warehouseListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.getWarehouseList, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/roadmap/index.js

```js
export roadmap from './roadmap';
export roadmaps from './roadmaps';
export onOrderRoadmaps from './onOrderRoadmaps';
export regionRate from './regionRate';
export regionRates from './regionRates';

```

* PDShop_Shop_PC/project/App/server/graphql/queries/roadmap/onOrderRoadmaps.js

```js
import {
    GraphQLInt,
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { roadmapListType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: roadmapListType,
    args: {
        orderId: {
            type: GraphQLID,
        },
        orderBy: {
            type: GraphQLInt,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.onOrderRoadmaps, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/roadmap/regionRate.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { regionRateType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: regionRateType,
    args: {
        regionRateId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.regionRate, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/roadmap/regionRates.js

```js
import {
    GraphQLInt,
    GraphQLID,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { regionRateListType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: regionRateListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        shopId: {
            type: GraphQLID,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.regionRates, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/roadmap/roadmap.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { roadmapType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: roadmapType,
    args: {
        roadmapId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.roadmap, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Shop_PC/project/App/server/graphql/queries/roadmap/roadmaps.js

```js
import {
    GraphQLInt,
    GraphQLID,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { roadmapListType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: roadmapListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        shopId: {
            type: GraphQLID,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.roadmaps, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Shop_PC/project/App/server/shared/helpers/api/index.js

```js
export urls from './urls';
export post from './post';

```

* PDShop_Shop_PC/project/App/server/shared/helpers/api/post.js

```js
import request from 'superagent';

export default function doRequest (url, data, root) {
    return new Promise((resolve) => {
        if (root && root.user) {
            Object.assign(data, { userId: root.user.userId, fromPC: true });
        }
        const req = request
        .post(url)
        .set({
            'Content-Type': 'application/json',
            'Accept': 'application/json',
        })
        .send(data);

        req.end((error, res) => {
            if (error) {
                console.error('recv error[' + url + ']:', error);
                resolve();
            } else {
                console.log('recv[' + url + ']:', res.body);
                resolve(res.body);
            }
        });
    });
}

```

* PDShop_Shop_PC/project/App/server/shared/helpers/api/urls.js

```js
const { apiServer } = require('../../../../../config.js');
const server = apiServer + 'shop/';
module.exports = {
    // common
    address: apiServer + 'getRegionAddress', // 获取地址列表
    addressFromLastCode: apiServer + 'getRegionAddressFromLastCode', // 通过最后一个code地址列表
    sendDoorAddressFromLastCode: server + 'getRegionSendDoorAddressFromLastCode', // 通过最后一个code获取送货上门地址列表

    // 个人中心
    getVerifyCode: apiServer + 'requestSendVerifyCode', // 请求发送验证码
    login: server + 'login', // 登录
    register: server + 'register', // 注册
    forgotPwd: server + 'findPassword', // 忘记密码
    setPaymentPassword: server + 'setPaymentPassword', // 设置支付密码
    modifyPassword: server + 'modifyPassword', // 修改密码
    personal: server + 'getPersonalInfo', // 获取物流公司信息
    modifyPersonalInfo: server + 'modifyPersonalInfo', // 修改个人信息
    feedback: server + 'submitFeedback', // 意见反馈

    // 账目
    remainAmount: server + 'getRemainAmount', // 获取余额
    weixinPayGetUnifiedOrder: server + 'weixinPayGetUnifiedOrderForPC', // 统一下单
    weixinPayWithDraw: server + 'withdraw', // 提现
    bills: server + 'getBills', // 获取账单

    // 店铺
    shop: server + 'getOwnShopDetail', // 获取自身所在物流超市详情（总部和分店）
    modifyOwnShop: server + 'modifyOwnShop', // 修改自身所在物流超市详情（总部和分店）
    branchShop: server + 'getBranchShopDetail', // 获取分店详情（总部）
    branchShops: server + 'getBranchShopList', // 获取分店列表（总部）
    createBranchShop: server + 'createBranchShop', // 创建分店（总部）
    modifyBranchShop: server + 'modifyBranchShop', // 修改分店（总部）
    removeBranchShop: server + 'removeBranchShop', // 删除分店（总部）

    // 货单
    orders: server + 'getOrders', // 分店获取货单综合列表
    order: server + 'getOrderDetail', // 获取货单详情

    // 部门
    partments: server + 'getPartmentList', // 获取部门列表
    ownPartment: server + 'getOwnPartmentInfo', // 获取所在部门的信息
    partment: server + 'getPartmentDetail', // 获取部门详情
    createPartment: server + 'createPartment', // 创建部门
    modifyPartment: server + 'modifyPartment', // 修改部门信息
    modifyOwnPartment: server + 'modifyOwnPartment', // 修改所在部门信息
    removePartment: server + 'removePartment', // 删除部门

    // 人员
    members: server + 'getMemberList', // 获取人员列表
    member: server + 'getMemberDetail', // 获取人员详情
    createMember: server + 'createMember', // 创建人员
    modifyMember: server + 'modifyMember', // 修改人员信息
    removeMember: server + 'removeMember', // 删除人员
    warehouseMemberList: server + 'getWarehouseMemberList',  // 获取分店库管人员

    // 设置
    setting: server + 'getSettingInfo', // 获取设置信息
    modifySetting: server + 'modifySettingInfo', // 修改设置信息

    // 物流公司
    shippers: server + 'getShipperList', // 获取物流公司列表
    shipper: server + 'getShipperDetail', // 获取物流公司详情
    registerShipper: server + 'registerShipper', // 注册物流公司
    createAndRegisterShipper: server + 'createAndRegisterShipper', // 创建并注册物流公司
    modifyShipper: server + 'modifyShipper', // 修改物流公司信息
    getShipperByChairManPhone: server + 'getShipperByChairManPhone', // 通过董事长电话获取物流公司
    passExamineTruck: server + 'passExamineTruck', // 通过汽车的审核

    // 收货点
    agents: server + 'getAgentList', // 获取收货点列表
    agent: server + 'getAgentDetail', // 获取收货点详情
    createAgent: server + 'createAgent', // 创建收货点
    modifyAgent: server + 'modifyAgent', // 修改收货点信息
    removeAgent: server + 'removeAgent', // 删除收货点
    checkAgentByChairManPhone: server + 'checkAgentByChairManPhone', // 检测收货点董事长
    trucks: server + 'getNeedExamineTruckList', // 获取需要审核的货车

    // 客户
    clients: server + 'getClientList', // 获取路线列表
    client: server + 'getClientDetail', // 获取路线详情
    getClientNameByPhone: apiServer + 'getClientNameByPhone', // 通过电话号码获取用户名

    // 路线
    onOrderRoadmaps: server + 'getRoadmapListWithOrderByReceivePartment', // 根据货单获取路线列表
    roadmap: server + 'getRoadmapDetail', // 获取路线详情
    roadmaps: server + 'getRoadmapList', // 获取路线列表
    setRoadmapProfit: server + 'setRoadmapProfit', // 批量修改路线提成比例
    regionRate: server + 'getRoadmapRegionProfitRateDetail', // 获取区域的路线提成详情
    regionRates: server + 'getRoadmapRegionProfitRateList', // 获取区域的路线提成列表
    addRoadmapRegionProfitRate: server + 'addRoadmapRegionProfitRate', // 添加区域的路线提成
    removeRoadmapRegionProfitRate: server + 'removeRoadmapRegionProfitRate', // 删除区域的路线提成
    modifyRoadmapRegionProfitRate: server + 'modifyRoadmapRegionProfitRate', // 修改区域的路线提成
    setRegionRateProfit: server + 'setRegionRateProfit', // 批量修改区域的路线提成

    // 收货部
    addressWithOrder: server + 'getRegionAddressWithOrder', // 通过货单获取地址列表
    sendDoorAddressWithOrder: server + 'getRegionSendDoorAddressWithOrder', // 通过货单获取送货上门地址列表
    rporders: server + 'getOrdersByReceivePartment', // 收货部获取货单综合列表
    rporder: server + 'getLastestOrderByReceivePartment', // 获取收货部需要的货单
    placeOriginOrder: server + 'placeOriginOrder', // 下初始货单
    placeOriginOrderWithPreOrder: server + 'placeOriginOrderWithPreOrder', // 通过预下单下初始货单
    modifyOriginOrder: server + 'modifyOriginOrder', // 修改初始货单
    removeOrder: server + 'removeOrder', // 删除货单
    selectRoadmap: server + 'selectRoadmap', // 选择物流公司
    selectRoadmapForOrderList: server + 'selectRoadmapForOrderList', // 批量为订单选择物流公司
    proxyPay: server + 'proxyPay', // 待支付
    proxyPayForOrderList: server + 'proxyPayForOrderList', // 待支付多个货单
    printOrderBill: server + 'printOrderBill', // 打印收据
    printOrderListBill: server + 'printOrderListBill', // 打印多个货单
    printBarCode: server + 'printBarCode', // 打印二维码
    selectPreOrders: server + 'getPreOrderList', // 收货部获取用户预下单列表

    // 仓库管理
    getWarehouseList: server + 'getWarehouseList', // 获取仓库列表
    getWarehouseDetail: server + 'getWarehouseDetail', // 获取仓库详细信息
    createWarehouse: server + 'createWarehouse', // 修改仓库信息
    modifyWarehouse: server + 'modifyWarehouse', // 修改仓库信息
    removeWarehouse: server + 'removeWarehouse', // 删除仓库信息

    // 库管部
    whorders: server + 'getOrdersByWareHousePartment', // 库管部获取货单综合列表

    // 统计
    shopStatistics: server + 'getStatistics', // 获取统计数据

    // 通知
    notifies: apiServer + 'getNotifies', // 获取通知列表
    sendNotify: apiServer + 'sendNotify', // 发布通知接口

    uploadFile: apiServer + 'uploadFile', // 上传文件

};

```

* PDShop_Shop_PC/project/App/shared/pages/shared/components/index.js

```js
// form
export FromGroup from './form/FromGroup';
export PlainFormItem from './form/PlainFormItem';
export TextFormItem from './form/TextFormItem';
export NumberFormItem from './form/NumberFormItem';
export ButtonFormItem from './form/ButtonFormItem';
export SelectFormItem from './form/SelectFormItem';
export EditFormItem from './form/EditFormItem';
export TableFormItem from './form/TableFormItem';
export ImageFormItem from './form/ImageFormItem';
export ImageListFormItem from './form/ImageListFormItem';
export LocateFormItem from './form/LocateFormItem';
export ChildFormItem from './form/ChildFormItem';
export DateFormItem from './form/DateFormItem';
export SizeFormItem from './form/SizeFormItem';
export CheckFormItem from './form/CheckFormItem';
export RadioFormItem from './form/RadioFormItem';
export AddressFormItem from './form/AddressFormItem';
export StartPointFormItem from './form/StartPointFormItem';
export PriceFormItem from './form/PriceFormItem';

// button
export DetailDeleteButton from './button/DetailDeleteButton';

// title
export TableContainer from './layout/TableContainer';
export DetailContainer from './layout/DetailContainer';

// table
export PlainTable from './table/PlainTable';
export TabsTable from './table/TabsTable';

// config
export * from './form/config';

```

* PDShop_Shop_PC/project/App/shared/pages/shared/utils/account.js

```js
export function getPayTool (str = '') {
    if (str === 'SIMULATION_NO') {
        return '模拟';
    }
    const options = {
        'A': '支付宝',
        'C': '财付通',
        'G': '工商银行',
        'J': '建设银行',
        'Z': '中国银行',
        'N': '农业银行',
    };
    const list = str.split('#');
    return options[list[0]];
}
export function getPayAccount (str = '') {
    if (str === 'SIMULATION_NO') {
        return 'SIMULATION_ACCOUNT';
    }
    const list = str.split('#');
    return list[2];
}
export function getPayBill (str = '') {
    if (str === 'SIMULATION_NO') {
        return 'SIMULATION_NO';
    }
    const list = str.split('#');
    return list[1];
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/utils/check.js

```js
import _ from 'lodash';

export function testTelePhone (phone) {
    return /^1\d{10}$/.test(phone);
}
export function testPhone (phone) {
    return /^(0\d{2}-?\d{8}(-?\d{1,4})?)|(0\d{3}-?\d{7,8}(-?\d{1,4})?)$/.test(phone);
}

export function checkTelePhone (rule, value, callback) {
    if (!testTelePhone(value)) {
        callback('请输入正确的手机号码');
    } else {
        callback();
    }
}
export function checkPhone (rule, value, callback) {
    if (!testPhone(value) && !testTelePhone(value)) {
        callback('请输入正确的电话号码');
    } else {
        callback();
    }
}

export function checkPhoneList (rule, value = '', callback) {
    const phoneList = _.reject(_.map(value.split(/;|；/), m => m.trim()), o => !o.length);
    if (phoneList.length && !_.every(phoneList, o => testPhone(o) || testTelePhone(o))) {
        callback('请输入正确的电话号码');
    } else {
        callback();
    }
}
export function checkEmail (rule, value, callback) {
    if (value && !/^(\w-*\.*)+@(\w-?)+(\.\w{2,})+$/.test(value)) {
        callback('请输入正确的邮箱地址');
    } else {
        callback();
    }
}
export function checkBankCard (rule, value, callback) {
    if (value && !/^(\d{16}|\d{19})$/.test(value)) {
        callback('请输入正确的银行卡卡号');
    } else {
        callback();
    }
}
export function checkVerifyCode (rule, value, callback) {
    if (value && !/^\d{6}$/.test(value)) {
        callback('请输入正确的验证码');
    } else {
        callback();
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/utils/common.js

```js
import _ from 'lodash';

export function _N (num, rank = 2) {
    typeof num !== 'number' && (num = 0);
    return parseFloat(num.toFixed(rank));
}
export function omitNil (obj) {
    return _.omitBy(obj, _.isNil);
}
export function needLoadPage (data, property, pageNo, pageSize) {
    const count = _.get(data, 'count');
    const list = _.get(data, property);
    const maxFullPage = Math.floor((count - 1) / pageSize);
    const maxDetectListSize = pageNo < maxFullPage ? pageSize : count - maxFullPage * pageSize;
    const detectList = _.slice(list, pageNo * pageSize, (pageNo + 1) * pageSize);
    let needLoad = true;
    if (detectList.length === maxDetectListSize) {
        needLoad = !_.every(detectList);
    }
    return needLoad;
}
export function until (test, iterator, callback) {
    if (!test()) {
        iterator((err) => {
            if (err) {
                return callback(err);
            }
            until(test, iterator, callback);
        });
    } else {
        callback();
    }
}
export function toThousands (num) {
    return (num || 0).toString().replace(/(\d)(?=(?:\d{3})+$)/g, '$1,');
}
export function getPercentages (list) {
    const sum = _.sum(list);
    return list.map((v) => Math.round(v * 100 / sum) + '%');
}
export function formatPhoneList (phoneList = '', phone = '') {
    phoneList = phone + ';' + phoneList.replace('；', ';');
    phoneList = _.uniq(_.filter(_.map(phoneList.split(';'), m => m.trim()), o => !!o));
    return phoneList.join(';');
}
export function getAddressOptions (list = [], lastCode, withTown) { // withTown为true，会一直选择到镇级和街道级
    let options;
    let value = [];
    for (const item of list) {
        const parent = _.find(item, o => o.code === lastCode) || {};
        parent.children = options;
        lastCode = parent.parentCode;
        parent.name && (value.unshift(parent.name));
        options = item.map(o => ({ value: o.name, label: o.name, code: o.code, level: o.level, isLeaf: !withTown ? o.isLeaf : o.level === 10, children: o.children }));
    }
    return {
        value,
        options: options || [],
    };
}
export function getStartPointOptions (list = [], lastCode, id) { // 如果始发地是分店或者收货点，则传id，否则不传，一旦传了id，lastCode则无效
    let options;
    let value = [];
    for (const item of list) {
        const parent = _.find(item, o => !id ? o.code === lastCode : o.id === id) || {};
        parent.children = options;
        lastCode = parent.isLeaf ? parent.addressRegionLastCode : parent.parentCode;
        (parent.id || parent.name) && (value.unshift(parent.id || parent.name));
        options = item.map(o => ({ value: o.id || o.name, label: o.name + (o.isShop ? '（分店）' : o.isAgent ? '（收货点）' : ''), code: o.code, level: o.level, isLeaf: o.isLeaf || false, isShop: o.isShop, isAgent: o.isAgent, children: o.children }));
        id = undefined;
    }
    return {
        value,
        options: options || [],
    };
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/utils/confirm.js

```js
import React from 'react';
import _ from 'lodash';
import { Modal, Input, notification } from 'antd';

export function confirm ({ pre, name, post = '' }, onOk) {
    Modal.confirm({
        title: '确认',
        content: (
            <div style={{ paddingTop: 10, fontSize: 14 }}>
                确定{pre}{name && <span style={{ color: 'red' }}>{name}</span>}{post}吗？
            </div>
        ),
        okText: '确定',
        cancelText: '取消',
        onOk,
    });
}
export function confirmWithPassword ({ pre, name, post = '', type = '登录' }, onOk) {
    let password = '';
    Modal.confirm({
        title: '确认',
        content: (
            <div style={{ paddingTop: 10, fontSize: 14 }}>
                {pre}{name && <span style={{ color: 'red' }}>{name}</span>}{post} 需要输入确认身份，请输入{type}密码。
                <Input style={{ marginTop: 10 }} placeholder={`请输入${type}密码`} maxLength={20}
                    type='password' autoComplete='off' onContextMenu={_.noop} onPaste={_.noop}
                    onCopy={_.noop} onCut={_.noop} onChange={(e) => {
                        password = e.target.value;
                    }} />
            </div>
        ),
        okText: '确定',
        cancelText: '取消',
        onOk: () => { onOk(password); },
    });
}
export function handleEditCancel () {
    Modal.confirm({
        title: '提示',
        content: (
            <div style={{ paddingTop: 10, fontSize: 14 }}>
                确定取消修改吗？
            </div>
        ),
        okText: '确定',
        cancelText: '取消',
        onOk: () => {
            this.setState({ pageData: _.cloneDeep(this.state.origin), editing: false });
        },
    });
}
export function showSuccess (info, title) {
    notification.success({ placement: 'bottomRight', key:'success', description: info, message: title });
}
export function showError (info, title) {
    notification.error({ placement: 'bottomRight', key:'error', description: info, message: title });
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/utils/index.js

```js
export * from './common';
export * from './check';
export * from './confirm';
export * from './account';
export * from './qrcode';

```

* PDShop_Shop_PC/project/App/shared/pages/shared/utils/qrcode.js

```js
export function genQRString (id, type) {
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
}
export function getQRResult (str) {
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
}

```

* PDShop_Shop_PC/project/App/shared/pages/auth/pages/forgotPwd/index.js

```js
import React from 'react';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import * as personalActions from 'actions/personals';
import ForgotPwd from './contents';

@connect(
    (state) => ({}),
    (dispatch) => ({
        actions : bindActionCreators(personalActions, dispatch),
    }),
)
export default class ForgotPwdContainer extends React.Component {
    render () {
        return (
            <ForgotPwd {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/auth/pages/login/index.js

```js
import { connect } from 'react-redux';
import React from 'react';
import { bindActionCreators } from 'redux';
import * as personalActions from 'actions/personals';
import Login from './contents';

@connect(
    (state) => ({ phone: state.router.location.state ? state.router.location.state.phone : '' }),
    (dispatch) => ({
        actions : bindActionCreators(personalActions, dispatch),
    }),
)
export default class LoginContainer extends React.Component {
    render () {
        return (
            <Login {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/auth/pages/register/index.js

```js
import React from 'react';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import * as personalActions from 'actions/personals';

import Register from './contents';

@connect(
    (state) => ({}),
    (dispatch) => ({
        actions : bindActionCreators(personalActions, dispatch),
    }),
)
export default class RegisterContainer extends React.Component {
    render () {
        return (
            <Register {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/components/form/config.js

```js
export function getFormItemLayout (layout, formGroup, hasOffset) {
    if (formGroup) {
        return {};
    }
    return {
        labelCol: { span: layout && layout[0] ? layout[0] : 4 },
        wrapperCol: { span: layout ? (layout[1] ? layout[1] : 16 - layout[0]) : 12, offset: hasOffset ? (layout && layout[0] ? layout[0] : 4) : 0 },
    };
}
export function getDefaultRules (label, required, rules = [], word = '填写') {
    const defaultRules = required ? [{ required, message: `请${word}${label}` }] : [];
    return [ ...defaultRules, ...rules ];
}
export function isNullValue (value) {
    if (!value) {
        if (value === 0) {
            return false;
        }
        return true;
    }
    return false;
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/components/table/config.js

```js
import React from 'react';
import { _N } from 'utils';
import _ from 'lodash';
import styles from './index.less';

export function fixColumns (columns) {
    columns = _.cloneDeep(columns);
    return columns.map((o) => {
        let unitRender;
        if (o.unit) {
            unitRender = (data = 0, record, index) => <span>{ _N(o.value ? o.value(data, record, index) : data) }<span className={styles.unit}>{_.isFunction(o.unit) ? o.unit(data, record, index) : o.unit}</span></span>;
        }

        let valuesRender;
        if (o.values) {
            valuesRender = (data, record, index) => {
                const values = _.reject(o.values(data, record, index), k => _.isNil(k));
                if (values[1]) {
                    return <div className={styles.baseValueContainer}><span>{values[0]}</span><span className={styles.baseValue}>{values[1]}</span></div>;
                }
                return <span>{values[0]}</span>;
            };
        }
        if (!o.render) {
            o.render = (data, record, index) => {
                if (valuesRender) {
                    data = valuesRender(data, record, index);
                } else if (unitRender) {
                    data = unitRender(data, record, index);
                } else if (o.value) {
                    data = o.value(data, record, index);
                } else if (_.isNil(data)) {
                    data = '无';
                }
                return data;
            };
        } else {
            const render = o.render;
            o.render = (data, record, index) => {
                data = render(data, record, index);
                if (o.unit) {
                    data = <span>{data}<span className={styles.unit}>{_.isFunction(o.unit) ? o.unit(data, record, index) : o.unit}</span></span>;
                }
                return data;
            };
        }

        if (o.totalUnit) {
            const title = o.title;
            o.title = <span>{title}<span className={styles.unit}>({o.totalUnit})</span></span>;
            delete o.totalUnit;
        }
        return o;
    });
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/pages/SelectBranchShop/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SelectBranchShop from './contents';

@dataConnect(
    (state) => ({ pageSize: 20, keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: SelectBranchShop.fragments,
        variablesTypes: {
            branchShops: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            branchShops: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class SelectBranchShopContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                branchShops: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.branchShops) {
                    showError('没有相关分店');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, branchShops } = this.props;
        const property = 'branchShopList';
        if (needLoadPage(branchShops, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    branchShops: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <SelectBranchShop {...this.props}
                refresh={::this.refresh}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/pages/SelectMember/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SelectMember from './contents';

@dataConnect(
    (state) => ({ states: state.members, pageSize: 20, keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: SelectMember.fragments,
        variablesTypes: {
            members: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            members: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class SelectMemberContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                members: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.members) {
                    showError('没有相关路线');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, members } = this.props;
        const property = 'memberList';
        if (needLoadPage(members, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    members: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <SelectMember {...this.props}
                refresh={::this.refresh}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/pages/SelectPartment/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SelectPartment from './contents';

@dataConnect(
    (state) => ({ states: state.partments, pageSize: 20, keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: SelectPartment.fragments,
        variablesTypes: {
            partments: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            partments: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class SelectPartmentContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                partments: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.partments) {
                    showError('没有相关路线');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, partments } = this.props;
        const property = 'partmentList';
        if (needLoadPage(partments, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    partments: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <SelectPartment {...this.props}
                refresh={::this.refresh}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/pages/SelectPreOrder/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SelectPreOrder from './contents';

@dataConnect(
    (state) => ({ pageSize: 20, keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: SelectPreOrder.fragments,
        variablesTypes: {
            selectPreOrders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                senderPhone: 'ID!',
                keyword: 'String',
            },
        },
        initialVariables: {
            selectPreOrders: {
                pageNo: 0,
                pageSize: props.pageSize,
                senderPhone: props.senderPhone,
            },
        },
    })
)
export default class SelectPreOrderContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize, senderPhone } = this.props;
        relate.refresh({
            variables: {
                selectPreOrders: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    senderPhone,
                },
            },
            callback (data) {
                if (!data.selectPreOrders) {
                    showError('没有相关货单');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, selectPreOrders, senderPhone } = this.props;
        const property = 'orderList';
        if (needLoadPage(selectPreOrders, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    selectPreOrders: {
                        pageNo,
                        pageSize,
                        keyword,
                        senderPhone,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <SelectPreOrder {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/pages/SelectRoadmap/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SelectRoadmap from './contents';

@dataConnect(
    (state) => ({ pageSize: 20, keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: SelectRoadmap.fragments,
        variablesTypes: {
            onOrderRoadmaps: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                orderId: 'ID!',
                orderBy: 'Int!',
            },
        },
        initialVariables: {
            onOrderRoadmaps: {
                pageNo: 0,
                pageSize: props.pageSize,
                orderId: props.orderId,
                orderBy: 0,
            },
        },
    })
)
export default class SelectRoadmapContainer extends React.Component {
    refresh (orderBy) {
        const { relate, pageSize, orderId } = this.props;
        relate.refresh({
            variables: {
                onOrderRoadmaps: {
                    pageNo: 0,
                    pageSize,
                    orderId,
                    orderBy,
                },
            },
            callback (data) {
                if (!data.onOrderRoadmaps) {
                    showError('没有相关路线');
                }
            },
        });
    }
    loadListPage (orderBy, pageNo) {
        const { relate, pageSize, orderId, onOrderRoadmaps } = this.props;
        const property = 'clientList';
        if (needLoadPage(onOrderRoadmaps, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    onOrderRoadmaps: {
                        pageNo,
                        pageSize,
                        orderId,
                        orderBy,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <SelectRoadmap {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/pages/SelectShipper/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SelectShipper from './contents';

@dataConnect(
    (state) => ({ pageSize: 20, keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: SelectShipper.fragments,
        variablesTypes: {
            shippers: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
                shopId: 'ID!',
            },
        },
        initialVariables: {
            shippers: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
                shopId: props.shopId,
            },
        },
    })
)
export default class SelectShipperShopContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                shippers: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    shopId: this.props.shopId,
                },
            },
            callback (data) {
                if (!data.shippers) {
                    showError('没有相关物流');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, shippers, shopId } = this.props;
        const property = 'shipperList';
        if (needLoadPage(shippers, shopId, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    shippers: {
                        pageNo,
                        pageSize,
                        keyword,
                        shopId,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            < SelectShipper {...this.props}
                refresh={::this.refresh}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shared/pages/SelectWareHouseMember/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SelectWareHouseMember from './contents';

@dataConnect(
    (state) => ({ states: state.wareHouseMembers, pageSize: 20, keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: SelectWareHouseMember.fragments,
        variablesTypes: {
            wareHouseMembers: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            wareHouseMembers: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class SelectWareHouseMemberContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                wareHouseMembers: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.wareHouseMembers) {
                    showError('没有相关成员');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, wareHouseMembers } = this.props;
        const property = 'memberList';
        if (needLoadPage(wareHouseMembers, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    wareHouseMembers: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <SelectWareHouseMember {...this.props}
                refresh={::this.refresh}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/agents/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as agentActions from 'actions/agents';
import { needLoadPage, showError } from 'utils';
import Agents from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(agentActions, dispatch),
    }),
    (props) => ({
        fragments: Agents.fragments,
        variablesTypes: {
            agents: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            agents: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class AgentsContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                agents: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.agents) {
                    showError('没有相关货车信息');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, agents } = this.props;
        const property = 'agentList';
        if (needLoadPage(agents, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    agents: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <Agents {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/bills/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import { needLoadPage, showError } from 'utils';
import Bills from './contents';

@dataConnect(
    (state) => ({ pageSize: 20, keepLastKeepData: true }),
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: Bills.fragments,
        variablesTypes: {
            bills: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                type: 'String',
                keyword: 'String',
                startDate: 'String',
                endDate: 'String',
            },
        },
        initialVariables: {
            bills: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class BillsContainer extends React.Component {
    refresh (keyword, startDate, endDate) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                bills: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    startDate,
                    endDate,
                },
            },
            callback (data) {
                if (!data.bills) {
                    showError('没有相关账单');
                }
            },
        });
    }
    loadListPage (keyword, type, pageNo, startDate, endDate) {
        const { relate, pageSize, bills } = this.props;
        if (needLoadPage(bills[type], 'list', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    bills: {
                        pageNo,
                        pageSize,
                        type,
                        keyword,
                        startDate,
                        endDate,
                    },
                },
                property: type + '.list',
            });
        }
    }
    render () {
        return (
            <Bills {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/branchShops/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as shopActions from 'actions/shops';
import { needLoadPage, showError } from 'utils';
import BranchShops from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(shopActions, dispatch),
    }),
    (props) => ({
        fragments: BranchShops.fragments,
        variablesTypes: {
            branchShops: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            branchShops: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class BranchShopsContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                branchShops: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.branchShops) {
                    showError('没有相关货车信息');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, branchShops } = this.props;
        const property = 'branchShopList';
        if (needLoadPage(branchShops, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    branchShops: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <BranchShops {...this.props}
                refresh={::this.refresh}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/clients/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as clientActions from 'actions/clients';
import { needLoadPage, showError } from 'utils';
import Clients from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(clientActions, dispatch),
    }),
    (props) => ({
        fragments: Clients.fragments,
        variablesTypes: {
            clients: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            clients: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class ClientsContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                clients: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.clients) {
                    showError('没有相关路线');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, clients } = this.props;
        const property = 'clientList';
        if (needLoadPage(clients, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    clients: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <Clients {...this.props}
                refresh={::this.refresh}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/feedback/index.js

```js
import React from 'react';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import * as personalActions from 'actions/personals';
import Feedback from './contents';

@connect(
    null,
    (dispatch) => ({
        actions : bindActionCreators(personalActions, dispatch),
    }),
)
export default class FeedbackContainer extends React.Component {
    render () {
        return (
            <Feedback {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/home/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import Home from './contents';

@dataConnect(
    (state) => ({}),
    null,
    (props) => ({
        fragments: Home.fragments,
    })
)
export default class HomeContainer extends React.Component {
    render () {
        return (
            <Home {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/members/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as memberActions from 'actions/members';
import { needLoadPage, showError } from 'utils';
import Members from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(memberActions, dispatch),
    }),
    (props) => ({
        fragments: Members.fragments,
        variablesTypes: {
            members: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            members: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class MembersContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                members: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.members) {
                    showError('没有相关货车信息');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, members } = this.props;
        const property = 'memberList';
        if (needLoadPage(members, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    members: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <Members {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/notifies/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import { needLoadPage } from 'utils';
import * as notifyActions from 'actions/notifies';
import Notify from './contents';

@dataConnect(
    (state) => {
        const { activeType } = state.router.location.state || state.router.location.query || { activeType: 'news' };
        return {
            activeType,
            pageSize: 20,
        };
    },
    (dispatch) => ({
        actions : bindActionCreators(notifyActions, dispatch),
    }),
    (props) => ({
        fragments: Notify.fragments,
        variablesTypes: {
            notifies: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                type: 'String!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            notifies: {
                pageNo: 0,
                pageSize: props.pageSize,
                type: '',
                keyword: '',
            },
        },
    })
)
export default class NotifyContainer extends React.Component {
    refresh (keyword = '') {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                notifies: {
                    pageNo: 0,
                    pageSize,
                    type: '',
                    keyword,
                },
            },
        });
    }
    loadListPage (keyword = '', type, pageNo) {
        const { relate, pageSize, notifies } = this.props;
        if (needLoadPage(notifies[type], pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    notifies: {
                        pageNo,
                        pageSize,
                        type,
                        keyword,
                    },
                },
                property: type + '.list',
            });
        }
    }
    render () {
        return (
            <Notify {...this.props}
                refresh={::this.refresh}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/orders/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import { needLoadPage, showError } from 'utils';
import BROrders from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: BROrders.fragments,
        variablesTypes: {
            orders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                shopId: 'ID',
                type: 'String',
                keyword: 'String',
                startDate: 'String',
                endDate: 'String',
            },
        },
        initialVariables: {
            orders: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class BROrdersContainer extends React.Component {
    refresh (keyword, shopId, startDate, endDate) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                orders: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    shopId,
                    startDate,
                    endDate,
                },
            },
            callback (data) {
                if (!data.orders) {
                    showError('没有相关货单');
                }
            },
        });
    }
    loadListPage (keyword, type, pageNo, shopId, startDate, endDate) {
        const { relate, pageSize, orders } = this.props;
        if (needLoadPage(orders[type], 'list', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    orders: {
                        pageNo,
                        pageSize,
                        type,
                        keyword,
                        shopId,
                        startDate,
                        endDate,
                    },
                },
                property: type + '.list',
            });
        }
    }
    render () {
        return (
            <BROrders {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/ownShop/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as shopActions from 'actions/shops';
import * as accountActions from 'actions/accounts';
import OwnShop from './contents';

@dataConnect(
    null,
    (dispatch) => ({
        actions : bindActionCreators({ ...shopActions, ...accountActions }, dispatch),
    }),
    (props) => ({
        fragments: OwnShop.fragments,
    })
)
export default class ShopContainer extends React.Component {
    refreshRemainAmount () {
        const { relate } = this.props;
        relate.refresh({
            property: 'remainAmount',
        });
    }
    render () {
        return (
            <OwnShop {...this.props} refreshRemainAmount={::this.refreshRemainAmount} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/partment/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as partmentActions from 'actions/partments';
import * as accountActions from 'actions/accounts';
import OwnPartment from './contents';

@dataConnect(
    (state) => ({}),
    (dispatch) => ({
        actions : bindActionCreators({ ...partmentActions, ...accountActions }, dispatch),
    }),
    (props) => ({
        fragments: OwnPartment.fragments,
    })
)
export default class OwnPartmentContainer extends React.Component {
    refreshRemainAmount () {
        const { relate } = this.props;
        relate.refresh({
            property: 'remainAmount',
        });
    }
    render () {
        return (
            <OwnPartment {...this.props} refreshRemainAmount={::this.refreshRemainAmount} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/partments/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as partmentActions from 'actions/partments';
import { needLoadPage, showError } from 'utils';
import Partments from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(partmentActions, dispatch),
    }),
    (props) => ({
        fragments: Partments.fragments,
        variablesTypes: {
            partments: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            partments: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class PartmentsContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                partments: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.partments) {
                    showError('没有相关货车信息');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, partments } = this.props;
        const property = 'partmentList';
        if (needLoadPage(partments, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    partments: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <Partments {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/personal/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as personalActions from 'actions/personals';
import Personal from './contents';

@dataConnect(
    (state) => ({}),
    (dispatch) => ({
        actions : bindActionCreators(personalActions, dispatch),
    }),
    (props) => ({
        fragments: Personal.fragments,
    })
)
export default class PersonalContainer extends React.Component {
    render () {
        return (
            <Personal {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/placeOrder/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import * as clientActions from 'actions/clients';
import PlaceOrder from './contents';

@dataConnect(
    null,
    (dispatch) => ({
        actions : bindActionCreators({ ...orderActions, ...clientActions }, dispatch),
    }),
    (props) => {
        return {
            fragments: PlaceOrder.fragments,
        };
    }
)
export default class PlaceOrderContainer extends React.Component {
    refresh () {
        const { relate } = this.props;
        relate.refresh({
            property: ['rporder', 'addressWithOrder', 'sendDoorAddressWithOrder'],
        });
    }
    render () {
        return (
            <PlaceOrder {...this.props} refresh={::this.refresh} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/regionRates/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as roadmapActions from 'actions/roadmaps';
import { needLoadPage, showError } from 'utils';
import RegionRates from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(roadmapActions, dispatch),
    }),
    (props) => ({
        fragments: RegionRates.fragments,
        variablesTypes: {
            regionRates: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String',
                shopId: 'ID',
            },
        },
        initialVariables: {
            regionRates: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class RegionRatesContainer extends React.Component {
    refresh (keyword, shopId) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                regionRates: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    shopId,
                },
            },
            callback (data) {
                if (!data.regionRates) {
                    showError('没有相关方向的设置');
                }
            },
        });
    }
    loadListPage (keyword, shopId, pageNo) {
        const { relate, pageSize, regionRates } = this.props;
        const property = 'list';
        if (needLoadPage(regionRates, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    regionRates: {
                        pageNo,
                        pageSize,
                        keyword,
                        shopId,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <RegionRates {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/roadmaps/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as roadmapActions from 'actions/roadmaps';
import { needLoadPage, showError } from 'utils';
import Roadmaps from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(roadmapActions, dispatch),
    }),
    (props) => ({
        fragments: Roadmaps.fragments,
        variablesTypes: {
            roadmaps: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String',
                shopId: 'ID',
            },
        },
        initialVariables: {
            roadmaps: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class RoadmapsContainer extends React.Component {
    refresh (keyword, shopId) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                roadmaps: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    shopId,
                },
            },
            callback (data) {
                if (!data.roadmaps) {
                    showError('没有相关路线');
                }
            },
        });
    }
    loadListPage (keyword, shopId, pageNo) {
        const { relate, pageSize, roadmaps } = this.props;
        const property = 'roadmapList';
        if (needLoadPage(roadmaps, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    roadmaps: {
                        pageNo,
                        pageSize,
                        keyword,
                        shopId,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <Roadmaps {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/rporders/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import { needLoadPage, showError } from 'utils';
import RPOrders from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: RPOrders.fragments,
        variablesTypes: {
            rporders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                type: 'String',
                keyword: 'String',
                startDate: 'String',
                endDate: 'String',
            },
        },
        initialVariables: {
            rporders: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class RPOrdersContainer extends React.Component {
    refresh (keyword, startDate, endDate) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                rporders: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    startDate,
                    endDate,
                },
            },
            callback (data) {
                if (!data.rporders) {
                    showError('没有相关货单');
                }
            },
        });
    }
    loadListPage (keyword, type, pageNo, startDate, endDate) {
        const { relate, pageSize, rporders } = this.props;
        if (needLoadPage(rporders[type], 'list', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    rporders: {
                        pageNo,
                        pageSize,
                        type,
                        keyword,
                        startDate,
                        endDate,
                    },
                },
                property: type + '.list',
            });
        }
    }
    render () {
        return (
            <RPOrders {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/setting/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as settingActions from 'actions/settings';
import Setting from './contents';

@dataConnect(
    null,
    (dispatch) => ({
        actions : bindActionCreators(settingActions, dispatch),
    }),
    (props) => {
        return {
            fragments: Setting.fragments,
        };
    }
)
export default class SettingContainer extends React.Component {
    refreshSetting () {
        const { relate } = this.props;
        relate.refresh({
            property: 'setting',
        });
    }
    render () {
        return (
            <Setting {...this.props} refreshSetting={::this.refreshSetting} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/shippers/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as shipperActions from 'actions/shippers';
import { needLoadPage, showError } from 'utils';
import Shippers from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(shipperActions, dispatch),
    }),
    (props) => ({
        fragments: Shippers.fragments,
        variablesTypes: {
            shippers: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
                shopId: 'ID!',
            },
        },
        initialVariables: {
            shippers: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
                shopId: '',
            },
        },
    })
)
export default class ShippersContainer extends React.Component {
    refresh (keyword, shopId = '') {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                shippers: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    shopId,
                },
            },
            callback (data) {
                if (!data.shippers) {
                    showError('没有相关路线');
                }
            },
        });
    }
    loadListPage (keyword, shopId = '', pageNo) {
        const { relate, pageSize, shippers } = this.props;
        const property = 'shipperList';
        if (needLoadPage(shippers, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    shippers: {
                        pageNo,
                        pageSize,
                        keyword,
                        shopId,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <Shippers {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/statistics/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import Statistics from './contents';

@dataConnect(
    null,
    null,
    (props) => ({
        fragments: Statistics.fragments,
        variablesTypes: {
            shopStatistics: {
                shopId: 'ID',
            },
        },
    })
)
export default class StatisticsContainer extends React.Component {
    refresh (shopId) {
        const { relate } = this.props;
        relate.refresh({
            variables: {
                shopStatistics: {
                    shopId,
                },
            },
        });
    }
    render () {
        return (
            <Statistics {...this.props} refresh={::this.refresh} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/trucks/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as shipperActions from 'actions/shippers';
import { needLoadPage, showError } from 'utils';
import Trucks from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(shipperActions, dispatch),
    }),
    (props) => ({
        fragments: Trucks.fragments,
        variablesTypes: {
            trucks: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            trucks: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class TrucksContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                trucks: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.trucks) {
                    showError('没有相关货车信息');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, trucks } = this.props;
        const property = 'truckList';
        if (needLoadPage(trucks, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    trucks: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <Trucks {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/warehouse/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as warehouseActions from 'actions/warehouses';
import { needLoadPage, showError } from 'utils';
import Warehouses from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(warehouseActions, dispatch),
    }),
    (props) => ({
        fragments: Warehouses.fragments,
        variablesTypes: {
            warehouses: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String',
            },
        },
        initialVariables: {
            warehouses: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class WarehouseContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                warehouses: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.warehouses) {
                    showError('没有相关仓库');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, warehouses } = this.props;
        const property = 'list';
        if (needLoadPage(warehouses, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    warehouses: {
                        pageNo,
                        pageSize,
                        keyword,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <Warehouses {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/whorders/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import { needLoadPage, showError } from 'utils';
import WHOrders from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: WHOrders.fragments,
        variablesTypes: {
            whorders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                type: 'String',
                keyword: 'String',
                startDate: 'String',
                endDate: 'String',
            },
        },
        initialVariables: {
            whorders: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class WHOrdersContainer extends React.Component {
    refresh (keyword, startDate, endDate) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                whorders: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    startDate,
                    endDate,
                },
            },
            callback (data) {
                if (!data.whorders) {
                    showError('没有相关货单');
                }
            },
        });
    }
    loadListPage (keyword, type, pageNo, startDate, endDate) {
        const { relate, pageSize, whorders } = this.props;
        if (needLoadPage(whorders[type], 'list', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    whorders: {
                        pageNo,
                        pageSize,
                        type,
                        keyword,
                        startDate,
                        endDate,
                    },
                },
                property: type + '.list',
            });
        }
    }
    render () {
        return (
            <WHOrders {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/agents/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as agentActions from 'actions/agents';
import AgentDetail from './contents';

@dataConnect(
    (state) => {
        const { addressRegionLastCode, agentId, record, agents } = state.router.location.state || state.router.location.query;
        return { addressRegionLastCode, agentId, record, agents, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(agentActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: !!props.agent || !props.agentId,
            fragments: AgentDetail.fragments,
            variablesTypes: {
                agent: {
                    agentId: 'ID!',
                },
                addressFromLastCode: {
                    addressLastCode: 'Int',
                },
            },
            initialVariables: {
                agent: {
                    agentId: props.agentId,
                },
                addressFromLastCode: {
                    addressLastCode: props.addressRegionLastCode,
                },
            },
            mutations: {
                modifyAgent ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removeAgent ({ state, data, _ }) {
                    if (data.success) {
                        props.agents.count--;
                        _.remove(props.agents.agentList, (item) => item.id === props.agentId);
                    }
                },
            },
        };
    }
)
export default class AgentDetailContainer extends React.Component {
    render () {
        return (
            <AgentDetail {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/agents/pages/register/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as agentActions from 'actions/agents';
import AgentRegister from './contents';

@dataConnect(
    (state) => {
        const { agentId, record, agents } = state.router.location.state || state.router.location.query;
        return { agentId, record, agents, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(agentActions, dispatch),
    }),
    null,
)
export default class AgentRegisterContainer extends React.Component {
    render () {
        return (
            <AgentRegister {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/branchShops/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as shopActions from 'actions/shops';
import BranchShopDetail from './contents';

@dataConnect(
    (state) => {
        const { operType, addressRegionLastCode, shopId, record, branchShops } = state.router.location.state || state.router.location.query;
        return { operType: operType * 1, addressRegionLastCode, shopId, record, branchShops, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(shopActions, dispatch),
    }),
    (props) => {
        return {
            fragments: BranchShopDetail.fragments,
            variablesTypes: {
                branchShop: {
                    shopId: 'ID',
                },
                addressFromLastCode: {
                    addressLastCode: 'Int',
                },
            },
            initialVariables: {
                branchShop: {
                    shopId: props.shopId,
                },
                addressFromLastCode: {
                    addressLastCode: props.addressRegionLastCode,
                },
            },
            mutations: {
                modifyBranchShop ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removeBranchShop ({ state, data, _ }) {
                    if (data.success) {
                        props.branchShops.count--;
                        _.remove(props.branchShops.branchShopList, (item) => item.id === props.shopId);
                    }
                },
            },
        };
    }
)
export default class BranchShopDetailContainer extends React.Component {
    render () {
        return (
            <BranchShopDetail {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/clients/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as clientActions from 'actions/clients';
import ClientDetail from './contents';

@dataConnect(
    (state) => {
        const { clientId } = state.router.location.state;
        return { clientId, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(clientActions, dispatch),
    }),
    (props) => {
        return {
            fragments: ClientDetail.fragments,
            variablesTypes: {
                client: {
                    clientId: 'ID!',
                },
            },
            initialVariables: {
                client: {
                    clientId: props.clientId,
                },
            },
        };
    }
)
export default class ClientDetailContainer extends React.Component {
    render () {
        return (
            <ClientDetail {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/members/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as memberActions from 'actions/members';
import MemberDetail from './contents';

@dataConnect(
    (state) => {
        const { operType, memberId, record, members } = state.router.location.state || state.router.location.query;
        return { operType: operType * 1, memberId, record, members, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(memberActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: !!props.member || !props.memberId,
            fragments: MemberDetail.fragments,
            variablesTypes: {
                member: {
                    memberId: 'ID!',
                },
            },
            initialVariables: {
                member: {
                    memberId: props.memberId,
                },
            },
            mutations: {
                modifyMember ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removeMember ({ state, data, _ }) {
                    if (data.success) {
                        props.members.count--;
                        _.remove(props.members.memberList, (item) => item.id === props.memberId);
                    }
                },
            },
        };
    }
)
export default class MemberDetailContainer extends React.Component {
    render () {
        return (
            <MemberDetail {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/notifies/pages/sendNotify/index.js

```js
import React from 'react';
import { connect } from 'react-redux';
import { bindActionCreators } from 'redux';
import * as notifyActions from 'actions/notifies';
import SendNotify from './contents';

@connect(
    (state) => {
        const { notifies, activeType } = state.router.location.state;
        return { notifies, activeType, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(notifyActions, dispatch),
    }),
)
export default class SendNotifyContainer extends React.Component {
    render () {
        return (
            <SendNotify {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/orders/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import RPOrderDetail from './contents';

@dataConnect(
    (state) => {
        const { orderId, activeType } = state.router.location.state || state.router.location.query;
        return { orderId, activeType, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: !!props.order || !props.orderId,
            fragments: RPOrderDetail.fragments,
            variablesTypes: {
                order: {
                    orderId: 'ID!',
                },
            },
            initialVariables: {
                order: {
                    orderId: props.orderId,
                },
            },
        };
    }
)
export default class RPOrderDetailContainer extends React.Component {
    refreshRPOrder () {
        const { relate, orderId } = this.props;
        relate.refresh({
            variables: {
                order: {
                    orderId,
                },
            },
        });
    }
    render () {
        return (
            <RPOrderDetail {...this.props} refreshRPOrder={::this.refreshRPOrder} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/personal/pages/paymentPwd/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as personalActions from 'actions/personals';
import PaymentPwd from './contents';

@dataConnect(
    null,
    (dispatch) => ({
        actions : bindActionCreators(personalActions, dispatch),
    })
)
export default class PaymentPwdContainer extends React.Component {
    render () {
        return (
            <PaymentPwd {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/partments/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as partmentActions from 'actions/partments';
import PartmentDetail from './contents';

@dataConnect(
    (state) => {
        const { operType, partmentId, record, partments } = state.router.location.state || state.router.location.query;
        return { operType: operType * 1, partmentId, record, partments, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(partmentActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: !!props.partment || !props.partmentId,
            fragments: PartmentDetail.fragments,
            variablesTypes: {
                partment: {
                    partmentId: 'ID!',
                },
            },
            initialVariables: {
                partment: {
                    partmentId: props.partmentId,
                },
            },
            mutations: {
                modifyPartment ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removePartment ({ state, data, _ }) {
                    if (data.success) {
                        props.partments.count--;
                        _.remove(props.partments.partmentList, (item) => item.id === props.partmentId);
                    }
                },
            },
        };
    }
)
export default class PartmentDetailContainer extends React.Component {
    render () {
        return (
            <PartmentDetail {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/regionRates/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as roadmapActions from 'actions/roadmaps';
import RegionRateDetail from './contents';

@dataConnect(
    (state) => {
        const { operType, regionLastCode, regionRateId, record, regionRates } = state.router.location.state || state.router.location.query;
        return { operType: operType * 1, regionLastCode, regionRateId, record, regionRates, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(roadmapActions, dispatch),
    }),
    (props) => {
        return {
            fragments: RegionRateDetail.fragments,
            variablesTypes: {
                regionRate: {
                    regionRateId: 'ID',
                },
                addressFromLastCode: {
                    addressLastCode: 'Int',
                    type: 'Int',
                },
            },
            initialVariables: {
                regionRate: {
                    regionRateId: props.regionRateId,
                },
                addressFromLastCode: {
                    addressLastCode: props.regionLastCode,
                    type: 2,
                },
            },
            mutations: {
                modifyRoadmapRegionProfitRate ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removeRoadmapRegionProfitRate ({ state, data, _ }) {
                    if (data.success) {
                        props.regionRates.count--;
                        _.remove(props.regionRates.regionRateList, (item) => item.id === props.regionRateId);
                    }
                },
            },
        };
    }
)
export default class RegionRateDetailContainer extends React.Component {
    render () {
        return (
            <RegionRateDetail {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/roadmaps/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import RoadmapDetail from './contents';

@dataConnect(
    (state) => {
        const { roadmapId } = state.router.location.state;
        return { roadmapId, keepLastKeepData: true };
    },
    null,
    (props) => {
        return {
            fragments: RoadmapDetail.fragments,
            variablesTypes: {
                roadmap: {
                    roadmapId: 'ID!',
                },
            },
            initialVariables: {
                roadmap: {
                    roadmapId: props.roadmapId,
                },
            },
        };
    }
)
export default class RoadmapDetailContainer extends React.Component {
    render () {
        return (
            <RoadmapDetail {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/rporders/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import RPOrderDetail from './contents';

@dataConnect(
    (state) => {
        const { orderId, record, rporders, activeType } = state.router.location.state || state.router.location.query;
        return { orderId, record, rporders, activeType, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: !!props.order || !props.orderId,
            fragments: RPOrderDetail.fragments,
            variablesTypes: {
                order: {
                    orderId: 'ID!',
                },
                addressWithOrder: {
                    orderId: 'ID!',
                },
                sendDoorAddressWithOrder: {
                    orderId: 'ID!',
                },
            },
            initialVariables: {
                order: {
                    orderId: props.orderId,
                },
                addressWithOrder: {
                    orderId: props.orderId,
                },
                sendDoorAddressWithOrder: {
                    orderId: props.orderId,
                },
            },
        };
    }
)
export default class RPOrderDetailContainer extends React.Component {
    refresh () {
        const { relate, orderId } = this.props;
        relate.refresh({
            variables: {
                order: {
                    orderId,
                },
                addressWithOrder: {
                    orderId,
                },
                sendDoorAddressWithOrder: {
                    orderId,
                },
            },
        });
    }
    render () {
        return (
            <RPOrderDetail {...this.props} refresh={::this.refresh} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/shippers/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as shipperActions from 'actions/shippers';
import ShipperDetail from './contents';

@dataConnect(
    (state) => {
        const { shipperId, record, shippers } = state.router.location.state || state.router.location.query;
        return { shipperId, record, shippers, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(shipperActions, dispatch),
    }),
    (props) => {
        return {
            fragments: ShipperDetail.fragments,
            variablesTypes: {
                shipper: {
                    shipperId: 'ID!',
                    shopId: 'ID!',
                },
            },
            initialVariables: {
                shipper: {
                    shipperId: props.shipperId,
                    shopId: props.record.shopId,
                },
            },
            mutations: {
                modifyShipper ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removeShipper ({ state, data, _ }) {
                    if (data.success) {
                        props.shippers.count--;
                        _.remove(props.shippers.shipperList, (item) => item.id === props.record.id);
                    }
                },
            },
        };
    }
)
export default class ShipperDetailContainer extends React.Component {
    render () {
        return (
            <ShipperDetail {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/shippers/pages/register/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as shipperActions from 'actions/shippers';
import ShipperRegister from './contents';

@dataConnect(
    (state) => {
        const { shippers } = state.router.location.state || state.router.location.query;
        return { shippers, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(shipperActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: true,
            fragments: ShipperRegister.fragments,
            variablesTypes: {
                shipper: {
                    shipperId: 'ID!',
                    shopId: 'ID!',
                },
            },
            initialVariables: {
                shipper: {
                    shipperId: '',
                    shopId: '',
                },
            },
        };
    }
)
export default class ShipperRegisterContainer extends React.Component {
    render () {
        return (
            <ShipperRegister {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/warehouse/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as warehouseActions from 'actions/warehouses';
import WarehouseDetail from './contents';

@dataConnect(
    (state) => {
        const { operType, warehouseId, record, warehouses } = state.router.location.state || state.router.location.query;
        return { operType: operType * 1, warehouseId, record, warehouses, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(warehouseActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: !!props.warehouse || !props.warehouseId,
            fragments: WarehouseDetail.fragments,
            variablesTypes: {
                warehouse: {
                    warehouseId: 'ID',
                },
            },
            initialVariables: {
                warehouse: {
                    warehouseId: props.warehouseId,
                },
            },
            mutations: {
                modifyWarehouse ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removeWarehouse ({ state, data, _ }) {
                    if (data.success) {
                        props.warehouses.count--;
                        _.remove(props.warehouses.warehouseList, (item) => item.id === props.warehouseId);
                    }
                },
            },
        };
    }
)
export default class WarehouseDetailContainer extends React.Component {
    render () {
        return (
            <WarehouseDetail {...this.props} />
        );
    }
}

```

* PDShop_Shop_PC/project/App/shared/pages/shop/pages/whorders/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import WHOrderDetail from './contents';

@dataConnect(
    (state) => {
        const { orderId, activeType } = state.router.location.state || state.router.location.query;
        return { orderId, activeType, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: !!props.order || !props.orderId,
            fragments: WHOrderDetail.fragments,
            variablesTypes: {
                order: {
                    orderId: 'ID!',
                },
            },
            initialVariables: {
                order: {
                    orderId: props.orderId,
                },
            },
        };
    }
)
export default class WHOrderDetailContainer extends React.Component {
    refreshWHOrder () {
        const { relate, orderId } = this.props;
        relate.refresh({
            variables: {
                order: {
                    orderId,
                },
            },
        });
    }
    render () {
        return (
            <WHOrderDetail {...this.props} refreshWHOrder={::this.refreshWHOrder} />
        );
    }
}

```
