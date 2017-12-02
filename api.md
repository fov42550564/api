* PDShop_Client_PC/project/App/client/auth.js

```js
import routes from 'routers/auth';
import renderRoutes from './helpers/render-routes';

renderRoutes(routes);

```

* PDShop_Client_PC/project/App/client/client.js

```js
import 'babel-polyfill';

import routes from 'routers/client';
import renderRoutes from './helpers/render-routes';

renderRoutes(routes);

```

* PDShop_Client_PC/project/App/client/public.js

```js
import routes from 'routers/public';
import renderRoutes from './helpers/render-routes';

renderRoutes(routes);

```

* PDShop_Client_PC/project/App/server/index.js

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
    secret: '88164657simiantong_shipper', // session的密码
    resave: true,
    saveUninitialized: false,
    store: new MongoStore({
        url: config.dbServer,
        collection: 'sessions_shipper',
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
app.use(['/client/favicon.ico', '/client/images*', '/client/media*', '/client/css*', '/client/fonts*', '/client/assets*'], (req, res) => {
    res.status(404).end();
});

// GraphqQL server
app.use('/client/graphql', graphqlHTTP(req => ({
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
            content: '四面通物流大超市客户端',
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
            src: `${res.baseScriptsURL}/client/assets/common.js`,
        },
    }, /* {
        tag: 'script',
        props: {
            src: `http://api.map.baidu.com/api?v=2.0&ak=${config.baiduMapSK}`,
        },
    }*/];

    next();
});

app.use(routers.authRouter);
app.use(routers.clientRouter);
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

* PDShop_Client_PC/project/App/server/schema.js

```js
import clone from 'lodash/clone';
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
        this.queryFields = clone(queries);
        this.mutationFields = clone(mutations);

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

* PDShop_Client_PC/project/App/client/helpers/render-routes.js

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

* PDShop_Client_PC/project/App/server/graphql/authorize.js

```js
export function authorize (root) {
    if (!root.user) {
        throw new Error('unauthorized');
    }
}

```

* PDShop_Client_PC/project/App/server/routers/auth.js

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
                href: '/client/assets/auth.css',
            },
        });
        // res.locals.header.push({
        //     tag: 'link',
        //     props: {
        //         rel: 'stylesheet',
        //         type: 'text/css',
        //         href: '/client/assets/common.js.css',
        //     },
        // });
    }
    res.locals.header.push(getDefaultFavicon(res));
    res.locals.footer.push({
        tag: 'script',
        props: {
            src: `${res.baseScriptsURL}/client/assets/auth.js`,
        },
    });
    next();
}

authRouter.get('/client/login', (req, res, next) => {
    if (req.isAuthenticated()) {
        res.redirect('/client');
    } else {
        routeHandler(routes, req, res, next);
    }
});

authRouter.get('/client/logout', (req, res) => {
    req.logout();
    res.redirect('/client/login');
});

authRouter.get(/^\/client\/(register|forgotPwd)$/, (req, res, next) => {
    routeHandler(routes, req, res, next);
});

// Register | ForgotPwd
authRouter.get(/^\/client\/(register|forgotPwd)$/, injectScript, async (req, res, next) => {
    res.status(200).send(getMarkup(req, res));
});

// Login
authRouter.get('/client/login', injectScript, (req, res) => {
    if (req.isAuthenticated()) {
        res.redirect('/client');
    } else {
        res.status(200).send(getMarkup(req, res));
    }
});

export default authRouter;

```

* PDShop_Client_PC/project/App/server/routers/client.js

```js
import getBaseComponent from 'helpers/get-base-component';
import getDefaultFavicon from 'helpers/default-favicon';
import renderHtml from 'helpers/render-html';
import routeHandler from 'helpers/route-handler';
import routes from 'routers/client';
import { Router } from 'express';
import { graphql } from 'graphql';
import { getDataDependencies } from 'relatejs';

import schema from '../schema';

const clientRouter = new Router();

clientRouter.get(/^\/client\/.+/, (req, res, next) => {
    if (req.isAuthenticated()) {
        res.redirect('/client');
    } else {
        next();
    }
});

clientRouter.get('/client*', (req, res, next) => {
    if (req.isAuthenticated()) {
        if (process.env.NODE_ENV === 'production') {
            res.locals.header.push(getDefaultFavicon(res));
            res.locals.header.push({
                tag: 'link',
                props: {
                    rel: 'stylesheet',
                    type: 'text/css',
                    href: '/client/assets/client.css',
                },
            });
            // res.locals.header.push({
            //     tag: 'link',
            //     props: {
            //         rel: 'stylesheet',
            //         type: 'text/css',
            //         href: '/client/assets/common.js.css',
            //     },
            // });
        }
        res.locals.footer.push({
            tag: 'script',
            props: {
                src: `${res.baseScriptsURL}/client/assets/client.js`,
            },
        });
        next();
    } else {
        res.redirect('/client/login');
    }
});

clientRouter.get('/client*', (req, res, next) => {
    if (req.isAuthenticated()) {
        routeHandler(routes, req, res, next);
    } else {
        next();
    }
});

clientRouter.get('/client*', async (req, res, next) => {
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

export default clientRouter;

```

* PDShop_Client_PC/project/App/server/routers/index.js

```js
import clientRouter from './client';
import authRouter from './auth';
import publicRouter from './public';

export default {
    clientRouter,
    authRouter,
    publicRouter,
};

```

* PDShop_Client_PC/project/App/server/routers/public.js

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

* PDShop_Client_PC/project/App/shared/actions/accounts.js

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

* PDShop_Client_PC/project/App/shared/actions/agentOrders.js

```js
import { mutation } from 'relatejs';

export function placeClientOrder (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                placeClientOrder: { success: 1, msg: 1, context },
            },
            variables: {
                placeClientOrder: {
                    data: {
                        value: data,
                        type: 'orderInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.placeClientOrder);
        })(dispatch, getState);
    };
}
export function placeClientOrderWithPreOrder (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                placeClientOrderWithPreOrder: { success: 1, msg: 1, context },
            },
            variables: {
                placeClientOrderWithPreOrder: {
                    data: {
                        value: data,
                        type: 'orderInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.placeClientOrderWithPreOrder);
        })(dispatch, getState);
    };
}
export function modifyClientOrder (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyClientOrder: { success: 1, msg: 1, context },
            },
            variables: {
                modifyClientOrder: {
                    data: {
                        value: data,
                        type: 'orderInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyClientOrder);
        })(dispatch, getState);
    };
}
export function removeClientOrder (orderId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removeClientOrder: { success: 1, msg: 1 },
            },
            variables: {
                removeClientOrder: {
                    orderId: {
                        value: orderId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.removeClientOrder);
        })(dispatch, getState);
    };
}
export function confirmCachPayed (orderId, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                confirmCachPayed: { success: 1, msg: 1, context },
            },
            variables: {
                confirmCachPayed: {
                    orderId: {
                        value: orderId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.confirmCachPayed);
        })(dispatch, getState);
    };
}
export function confirmCachPayedForOrderList (orderIdList, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                confirmCachPayedForOrderList: { success: 1, msg: 1 },
            },
            variables: {
                confirmCachPayedForOrderList: {
                    orderIdList: {
                        value: orderIdList,
                        type: '[ID]!',
                    },
                },
            },
        }, (result) => {
            callback(result.confirmCachPayedForOrderList);
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

* PDShop_Client_PC/project/App/shared/actions/clients.js

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

* PDShop_Client_PC/project/App/shared/actions/members.js

```js
import { mutation, apiQuery } from 'relatejs';

export function modifyShipperMemberAuthority (memberId, authority, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyShipperMemberAuthority: { success: 1, msg: 1, context },
            },
            variables: {
                modifyShipperMemberAuthority: {
                    memberId: {
                        value: memberId,
                        type: 'ID!',
                    },
                    authority: {
                        value: authority,
                        type: '[Int]!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyShipperMemberAuthority);
        })(dispatch, getState);
    };
}

export function getShipperMemberByPhone (memberPhone, context, callback) {
    return () => {
        return apiQuery({
            fragments: {
                getShipperMemberByPhone: { success: 1, msg: 1, context },
            },
            variables: {
                getShipperMemberByPhone: {
                    memberPhone: {
                        value: memberPhone,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.getShipperMemberByPhone);
        })();
    };
}

// 收货点
export function modifyAgentMemberAuthority (memberId, authority, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyAgentMemberAuthority: { success: 1, msg: 1, context },
            },
            variables: {
                modifyAgentMemberAuthority: {
                    memberId: {
                        value: memberId,
                        type: 'ID!',
                    },
                    authority: {
                        value: authority,
                        type: '[Int]!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyAgentMemberAuthority);
        })(dispatch, getState);
    };
}

export function getAgentMemberByPhone (memberPhone, context, callback) {
    return () => {
        return apiQuery({
            fragments: {
                getAgentMemberByPhone: { success: 1, msg: 1, context },
            },
            variables: {
                getAgentMemberByPhone: {
                    memberPhone: {
                        value: memberPhone,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.getAgentMemberByPhone);
        })();
    };
}

```

* PDShop_Client_PC/project/App/shared/actions/orders.js

```js
import { mutation } from 'relatejs';

export function placePreOrder (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                placePreOrder: { success: 1, msg: 1, context },
            },
            variables: {
                placePreOrder: {
                    data: {
                        value: data,
                        type: 'orderInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.placePreOrder);
        })(dispatch, getState);
    };
}
export function modifyPreOrder (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyPreOrder: { success: 1, msg: 1, context },
            },
            variables: {
                modifyPreOrder: {
                    data: {
                        value: data,
                        type: 'orderInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyPreOrder);
        })(dispatch, getState);
    };
}
export function removePreOrder (orderId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removePreOrder: { success: 1, msg: 1 },
            },
            variables: {
                removePreOrder: {
                    orderId: {
                        value: orderId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.removePreOrder);
        })(dispatch, getState);
    };
}
export function createPreOrderGroup (data, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                createPreOrderGroup: { success: 1, msg: 1 },
            },
            variables: {
                createPreOrderGroup: {
                    data: {
                        value: data,
                        type: 'orderGroupInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.createPreOrderGroup);
        })(dispatch, getState);
    };
}
export function modifyPreOrderGroup (data, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyPreOrderGroup: { success: 1, msg: 1 },
            },
            variables: {
                modifyPreOrderGroup: {
                    data: {
                        value: data,
                        type: 'orderGroupInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyPreOrderGroup);
        })(dispatch, getState);
    };
}
export function removePreOrderFromGroup (orderIdList, orderGroupId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removePreOrderFromGroup: { success: 1, msg: 1 },
            },
            variables: {
                removePreOrderFromGroup: {
                    orderIdList: {
                        value: orderIdList,
                        type: '[ID]!',
                    },
                    orderGroupId: {
                        value: orderGroupId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.removePreOrderFromGroup);
        })(dispatch, getState);
    };
}
export function payForOrderWhenSend (orderIdList, payPassword, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                payForOrderWhenSend: { success: 1, msg: 1 },
            },
            variables: {
                payForOrderWhenSend: {
                    orderIdList: {
                        value: orderIdList,
                        type: '[ID]!',
                    },
                    payPassword: {
                        value: payPassword,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.payForOrderWhenSend);
        })(dispatch, getState);
    };
}
export function finishOrder (orderId, verifyCode, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                finishOrder: { success: 1, msg: 1 },
            },
            variables: {
                finishOrder: {
                    orderId: {
                        value: orderId,
                        type: 'ID!',
                    },
                    verifyCode: {
                        value: verifyCode,
                        type: 'String',
                    },
                },
            },
        }, (result) => {
            callback(result.finishOrder);
        })(dispatch, getState);
    };
}
export function setClientPickOrderTransportFee ({ orderId, isTransport, endPoint, transportFee }, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setClientPickOrderTransportFee: { success: 1, msg: 1 },
            },
            variables: {
                setClientPickOrderTransportFee: {
                    orderId: {
                        value: orderId,
                        type: 'ID!',
                    },
                    isTransport: {
                        value: isTransport,
                        type: 'Boolean!',
                    },
                    endPoint: {
                        value: endPoint,
                        type: 'String',
                    },
                    transportFee: {
                        value: transportFee,
                        type: 'Float',
                    },
                },
            },
        }, (result) => {
            callback(result.setClientPickOrderTransportFee);
        })(dispatch, getState);
    };
}

```

* PDShop_Client_PC/project/App/shared/actions/personals.js

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
                        type: 'personalInfoInputType!',
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
                        type: 'personalInfoInputType!',
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
                        type: 'personalInfoInputType!',
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
                        type: 'personalInfoInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyPersonalInfo);
        })(dispatch, getState);
    };
}
export function modifyShipperInfo (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyShipperInfo: { success: 1, msg: 1, context },
            },
            variables: {
                modifyShipperInfo: {
                    data: {
                        value: data,
                        type: 'shipperInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyShipperInfo);
        })(dispatch, getState);
    };
}
export function modifyAgentInfo (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyAgentInfo: { success: 1, msg: 1, context },
            },
            variables: {
                modifyAgentInfo: {
                    data: {
                        value: data,
                        type: 'agentInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyAgentInfo);
        })(dispatch, getState);
    };
}
export function getUserNameByPhone (phone, callback) {
    return () => {
        return apiQuery({
            fragments: {
                getUserNameByPhone: { success: 1, msg: 1, context: { name: 1 } },
            },
            variables: {
                getUserNameByPhone: {
                    phone: {
                        value: phone,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.getUserNameByPhone);
        })();
    };
}

export function setReferShop (shopId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setReferShop: { success: 1, msg: 1 },
            },
            variables: {
                setReferShop: {
                    shopId: {
                        value: shopId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.setReferShop);
        })(dispatch, getState);
    };
}

```

* PDShop_Client_PC/project/App/shared/actions/roadmaps.js

```js
import { mutation } from 'relatejs';

export function publishRoadmap (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                publishRoadmap: { success: 1, msg: 1, context },
            },
            variables: {
                publishRoadmap: {
                    data: {
                        value: data,
                        type: 'roadmapInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.publishRoadmap);
        })(dispatch, getState);
    };
}
export function modifyRoadmap (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyRoadmap: { success: 1, msg: 1, context },
            },
            variables: {
                modifyRoadmap: {
                    data: {
                        value: data,
                        type: 'roadmapInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyRoadmap);
        })(dispatch, getState);
    };
}
export function removeRoadmap (roadmapId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removeRoadmap: { success: 1, msg: 1 },
            },
            variables: {
                removeRoadmap: {
                    roadmapId: {
                        value: roadmapId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.removeRoadmap);
        })(dispatch, getState);
    };
}
export function setRoadmapPrice (data, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setRoadmapPrice: { success: 1, msg: 1 },
            },
            variables: {
                setRoadmapPrice: {
                    data: {
                        value: data,
                        type: 'setRoadmapPriceType!',
                    },
                },
            },
        }, (result) => {
            callback(result.setRoadmapPrice);
        })(dispatch, getState);
    };
}
export function setRoadmapSendDoorPrice (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setRoadmapSendDoorPrice: { success: 1, msg: 1, context },
            },
            variables: {
                setRoadmapSendDoorPrice: {
                    data: {
                        value: data,
                        type: 'setRoadmapSendDoorPriceType!',
                    },
                },
            },
        }, (result) => {
            callback(result.setRoadmapSendDoorPrice);
        })(dispatch, getState);
    };
}
export function setRoadmapSendDoorBasePrice (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setRoadmapSendDoorBasePrice: { success: 1, msg: 1, context },
            },
            variables: {
                setRoadmapSendDoorBasePrice: {
                    data: {
                        value: data,
                        type: 'setRoadmapSendDoorPriceType!',
                    },
                },
            },
        }, (result) => {
            callback(result.setRoadmapSendDoorBasePrice);
        })(dispatch, getState);
    };
}
export function setRoadmapEnable (roadmapIdList, enable, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setRoadmapEnable: { success: 1, msg: 1 },
            },
            variables: {
                setRoadmapEnable: {
                    roadmapIdList: {
                        value: roadmapIdList,
                        type: '[ID]!',
                    },
                    enable: {
                        value: enable,
                        type: 'Boolean!',
                    },
                },
            },
        }, (result) => {
            callback(result.setRoadmapEnable);
        })(dispatch, getState);
    };
}
export function setRoadmapSendDoorEnable (roadmapId, lastCodeList, enable, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setRoadmapSendDoorEnable: { success: 1, msg: 1 },
            },
            variables: {
                setRoadmapSendDoorEnable: {
                    roadmapId: {
                        value: roadmapId,
                        type: 'ID!',
                    },
                    lastCodeList: {
                        value: lastCodeList,
                        type: '[Int]!',
                    },
                    enable: {
                        value: enable,
                        type: 'Boolean!',
                    },
                },
            },
        }, (result) => {
            callback(result.setRoadmapSendDoorEnable);
        })(dispatch, getState);
    };
}

export function addRegionProfit (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                addRegionProfit: { success: 1, msg: 1, context },
            },
            variables: {
                addRegionProfit: {
                    data: {
                        value: data,
                        type: 'regionRateInputType',
                    },
                },
            },
        }, (result) => {
            callback(result.addRegionProfit);
        })(dispatch, getState);
    };
}

export function modifyRegionProfit (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyRegionProfit: { success: 1, msg: 1, context },
            },
            variables: {
                modifyRegionProfit: {
                    data: {
                        value: data,
                        type: 'regionRateInputType',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyRegionProfit);
        })(dispatch, getState);
    };
}

export function removeRegionProfit (regionId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removeRegionProfit: { success: 1, msg: 1 },
            },
            variables: {
                removeRegionProfit: {
                    regionId: {
                        value: regionId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.removeRegionProfit);
        })(dispatch, getState);
    };
}

export function setRegionProfitWithList (data, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setRegionProfitWithList: { success: 1, msg: 1 },
            },
            variables: {
                setRegionProfitWithList: {
                    regionIdList: {
                        value: data.regionIdList,
                        type: '[ID]!',
                    },
                    profitRate: {
                        value: data.profitRate,
                        type: 'Float',
                    },
                    profitCount: {
                        value: data.profitCount,
                        type: 'Float',
                    },
                },
            },
        }, (result) => {
            callback(result.setRegionProfitWithList);
        })(dispatch, getState);
    };
}


```

* PDShop_Client_PC/project/App/shared/actions/trucks.js

```js
import { mutation, apiQuery, apiMutation } from 'relatejs';

export function createTruck (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                createTruck: { success: 1, msg: 1, context },
            },
            variables: {
                createTruck: {
                    data: {
                        value: data,
                        type: 'truckInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.createTruck);
        })(dispatch, getState);
    };
}
export function modifyTruck (data, context, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                modifyTruck: { success: 1, msg: 1, context },
            },
            variables: {
                modifyTruck: {
                    data: {
                        value: data,
                        type: 'truckInputType!',
                    },
                },
            },
        }, (result) => {
            callback(result.modifyTruck);
        })(dispatch, getState);
    };
}
export function removeTruck (truckId, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                removeTruck: { success: 1, msg: 1 },
            },
            variables: {
                removeTruck: {
                    truckId: {
                        value: truckId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.removeTruck);
        })(dispatch, getState);
    };
}
export function getDriverNameByPhone (phone, callback) {
    return () => {
        return apiQuery({
            fragments: {
                getDriverNameByPhone: { id: 1, name: 1, phone: 1 },
            },
            variables: {
                getDriverNameByPhone: {
                    phone: {
                        value: phone,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.getDriverNameByPhone);
        })();
    };
}
export function selectCarryPartment (truckId, carryPartmentId, callback) {
    return () => {
        return apiMutation({
            fragments: {
                selectCarryPartment: { success: 1, msg: 1 },
            },
            variables: {
                selectCarryPartment: {
                    truckId: {
                        value: truckId,
                        type: 'ID!',
                    },
                    carryPartmentId: {
                        value: carryPartmentId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.selectCarryPartment);
        })();
    };
}
export function payforCarryPartment (truckId, callback) {
    return () => {
        return apiMutation({
            fragments: {
                payforCarryPartment: { success: 1, msg: 1 },
            },
            variables: {
                payforCarryPartment: {
                    truckId: {
                        value: truckId,
                        type: 'ID!',
                    },
                },
            },
        }, (result) => {
            callback(result.payforCarryPartment);
        })();
    };
}
export function setCityTruck (data, callback) {
    return (dispatch, getState) => {
        return mutation({
            fragments: {
                setCityTruck: { success: 1, msg: 1 },
            },
            variables: {
                setCityTruck: {
                    plateNo: {
                        value: data,
                        type: 'String!',
                    },
                },
            },
        }, (result) => {
            callback(result.setCityTruck);
        })(dispatch, getState);
    };
}

```

* PDShop_Client_PC/project/App/shared/config/menu.js

```js
import AH from '../constants/authority';
import CO from '../constants/common';
import _ from 'lodash';

export default [
    {
        label: '下单',
        link: '/client/placeOrder',
        auth: [AH.AH_PLACE_ORDER],
    },
    {
        label: '货单列表',
        link: '/client/rcorders',
        auth: [AH.AH_PLACE_ORDER],
    },
    {
        label: '路线设置',
        link: '/client/roadmaps',
        auth: [AH.AH_LOOK_ROADMAP, AH.AH_CREATE_ROADMAP, AH.AH_MODIFY_ROADMAP, AH.AH_REMOVE_ROADMAP],
    },
    {
        label: '路线设置',
        link: '/client/sendDoorRoadmaps',
        auth: [AH.AH_LOOK_SEND_DOOR_ROADMAP, AH.AH_CREATE_SEND_DOOR_ROADMAP, AH.AH_MODIFY_SEND_DOOR_ROADMAP, AH.AH_REMOVE_SEND_DOOR_ROADMAP],
    },
    {
        label: '竞得货物',
        link: '/client/needSendOrders',
        auth: [AH.AH_LOOK_TRUCK, AH.AH_CREATE_TRUCK, AH.AH_MODIFY_TRUCK, AH.AH_REMOVE_TRUCK],
    },
    {
        label: '我的货车',
        link: '/client/trucks',
        auth: [AH.AH_LOOK_TRUCK, AH.AH_CREATE_TRUCK, AH.AH_MODIFY_TRUCK, AH.AH_REMOVE_TRUCK],
    },
    {
        label: '方向提成',
        link: '/client/regionRates',
        auth: [AH.AH_MODIFY_ROADMAP_PROFIT],
    },
    {
        label: '承运货单',
        link: '/client/scorders',
        auth: (user) => user.shipper && user.shipper.shipperType === CO.ST_CITY && _.includes(user.authority, AH.AH_LOOK_ORDERS),
    },
    {
        label: '承运货单',
        link: '/client/sporders',
        auth: (user) => user.shipper && user.shipper.shipperType === CO.ST_LONG && _.includes(user.authority, AH.AH_LOOK_ORDERS),
    },

    {
        label: '业务统计',
        link: '/client/statistics',
        auth: [AH.AH_LOOK_STATISTICS],
    },
    {
        label: '我要发货',
        link: '/client/preOrders',
    },
    {
        label: '查看行情',
        link: '/client/quotations',
    },
    {
        label: '发货货单',
        link: '/client/sendOrders',
    },
    {
        label: '收货货单',
        link: '/client/receiveOrders',
    },
];

```

* PDShop_Client_PC/project/App/shared/config/topmenu.js

```js
import AH from '../constants/authority';

export default [
    {
        label: '人员管理',
        link: '/client/shipperMembers',
        auth: [AH.AH_MODIFY_SHIPPER_MEMBER_AUTHORITY],
    },
    {
        label: '人员管理',
        link: '/client/agentMembers',
        auth: [AH.AH_MODIFY_AGENT_MEMBER_AUTHORITY],
    },
    {
        label: '个人中心',
        link: '/client/personal',
    },
    {
        label: '意见反馈',
        link: '/client/feedback',
    },
    {
        label: '退出',
        link: '/client/logout',
    },
];

```

* PDShop_Client_PC/project/App/shared/constants/authority.js

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
export const AH_CITY_DISTRIBUTE = 30016; // 同城配送的权限

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

* PDShop_Client_PC/project/App/shared/constants/common.js

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

* PDShop_Client_PC/project/App/shared/constants/index.js

```js
export AH from './authority';
export OS from './orderState';
export TS from './truckState';
export CO from './common';

```

* PDShop_Client_PC/project/App/shared/constants/orderState.js

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

* PDShop_Client_PC/project/App/shared/constants/truckState.js

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

export default {
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

* PDShop_Client_PC/project/App/shared/helpers/configure-store.js

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

* PDShop_Client_PC/project/App/shared/helpers/ga-send.js

```js
export default function gaSend () {
    if (typeof window !== 'undefined') {
        window.ga && window.ga('send', 'pageview');
    }
}

```

* PDShop_Client_PC/project/App/shared/decorators/antd_form_create.js

```js
import { Form } from 'antd';
export default (target) => {
    const Component = Form.create()(target);
    Component.fragments = target.fragments;
    return Component;
};

```

* PDShop_Client_PC/project/App/shared/reducers/index.js

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

* PDShop_Client_PC/project/App/shared/relatejs/index.js

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

* PDShop_Client_PC/project/App/shared/routers/auth.js

```js
import React from 'react';
import { Route } from 'react-router';
import gaSend from 'helpers/ga-send';
import Auth from 'pages/auth';
import Register from 'pages/auth/pages/register';
import Login from 'pages/auth/pages/login';
import ForgotPwd from 'pages/auth/pages/forgotPwd';

export default [
    <Route component={Auth}>
        <Route path='/client/register' component={Register} onEnter={gaSend} />
        <Route path='/client/login' component={Login} onEnter={gaSend} />
        <Route path='/client/forgotPwd' component={ForgotPwd} onEnter={gaSend} />
    </Route>,
];

```

* PDShop_Client_PC/project/App/shared/routers/client.js

```js
import React from 'react';
import { Route, IndexRoute } from 'react-router';
import request from 'superagent';
import Client from 'pages/client';
import Home from 'pages/client/pages/home';
import Personal from 'pages/client/pages/personal';
import PaymentPwd from 'pages/client/pages/personal/pages/paymentPwd';
import Shipper from 'pages/client/pages/personal/pages/shipper';
import Agent from 'pages/client/pages/personal/pages/agent';
import ShipperMembers from 'pages/client/pages/shipperMembers';
import ShipperMemberDetail from 'pages/client/pages/shipperMembers/pages/detail';
import ShipperMemberRegister from 'pages/client/pages/shipperMembers/pages/register';
import Roadmaps from 'pages/client/pages/roadmaps';
import RoadmapDetail from 'pages/client/pages/roadmaps/pages/detail';
import RoadmapSendDoorList from 'pages/client/pages/roadmaps/pages/sendDoorList';
import SendDoorRoadmaps from 'pages/client/pages/sendDoorRoadmaps';
import SendDoorRoadmapDetail from 'pages/client/pages/sendDoorRoadmaps/pages/detail';
import Trucks from 'pages/client/pages/trucks';
import TruckDetail from 'pages/client/pages/trucks/pages/detail';
import PayForCarryPartment from 'pages/client/pages/trucks/pages/payForCarryPartment';
import SelectCarryPartment from 'pages/client/pages/trucks/pages/selectCarryPartment';
import TruckShowDetail from 'pages/client/pages/trucks/pages/showDetail';
import OrderListInTruck from 'pages/client/pages/trucks/pages/orderList';
import LookTruckFee from 'pages/client/pages/trucks/pages/lookTruckFee';
import PreOrders from 'pages/client/pages/preOrders';
import PreOrderDetail from 'pages/client/pages/preOrders/pages/detail';
import PreOrderGroupDetail from 'pages/client/pages/preOrders/pages/groupDetail';
import LookPreOrderFee from 'pages/client/pages/preOrders/pages/lookPreOrderFee';
import LookOrderFee from 'pages/client/pages/preOrders/pages/lookOrderFee';
import PreOrderLookGroupFee from 'pages/client/pages/preOrders/pages/lookGroupFee';
import Quotations from 'pages/client/pages/quotations';
import NeedSendOrders from 'pages/client/pages/needSendOrders';
import OrderListByEndPoint from 'pages/client/pages/needSendOrders/pages/orderListByEndPoint';
import OrderDetail from 'pages/client/pages/orderDetail';
import SPOrders from 'pages/client/pages/sporders';
import SCOrders from 'pages/client/pages/scorders';
import Bills from 'pages/client/pages/bills';
import Statistics from 'pages/client/pages/statistics';
import Feedback from 'pages/client/pages/feedback';
import Notify from 'pages/client/pages/notify';
import AgentMembers from 'pages/client/pages/agentMembers';
import AgentMemberDetail from 'pages/client/pages/agentMembers/pages/detail';
import AgentMemberRegister from 'pages/client/pages/agentMembers/pages/register';
import RegionRates from 'pages/client/pages/regionRates';
import RegionRate from 'pages/client/pages/regionRates/pages/detail';
import PlaceOrder from 'pages/client/pages/placeOrder';
import RCOrders from 'pages/client/pages/rcorders';
import RCOrderDetail from 'pages/client/pages/rcorders/pages/detail';
import SendOrders from 'pages/client/pages/sendOrders';
import SendOrderDetail from 'pages/client/pages/sendOrders/pages/detail';
import ReceiveOrders from 'pages/client/pages/receiveOrders';
import ReceiveOrderDetail from 'pages/client/pages/receiveOrders/pages/detail';

let firstEntry = true;

function authenticate (nextState, replaceState, callback) {
    if (typeof window !== 'undefined' && !firstEntry) {
        request
        .post('/client/graphql')
        .set('Content-Type', 'application/json')
        .set('Accept', 'application/json')
        .send({
            query: 'query { session }',
        })
        .end((error, result) => {
            if (error || !result.body.data.session) {
                window.location.href = '/client/login';
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
    <Route name='client' path='/client' component={Client}>
        <IndexRoute component={Home} onEnter={authenticate} />
        <Route name='clientPersonal' path='personal'>
            <IndexRoute component={Personal} onEnter={authenticate} />
            <Route name='clientPaymentPwd' path='paymentPwd' component={PaymentPwd} onEnter={authenticate} />
            <Route name='clientShipper' path='shipper' component={Shipper} onEnter={authenticate} />
            <Route name='clientAgent' path='agent' component={Agent} onEnter={authenticate} />
        </Route>
        <Route name='shipperMembers' path='shipperMembers'>
            <IndexRoute component={ShipperMembers} onEnter={authenticate} />
            <Route name='shipperMemberDetail' path='detail' component={ShipperMemberDetail} onEnter={authenticate} />
            <Route name='shipperMemberRegister' path='register' component={ShipperMemberRegister} onEnter={authenticate} />
        </Route>
        <Route name='clientRoadmaps' path='roadmaps'>
            <IndexRoute component={Roadmaps} onEnter={authenticate} />
            <Route name='clientRoadmapDetail' path='detail' component={RoadmapDetail} onEnter={authenticate} />
            <Route name='clientRoadmapSendDoorList' path='sendDoorList' component={RoadmapSendDoorList} onEnter={authenticate} />
        </Route>
        <Route name='clientSendDoorRoadmaps' path='sendDoorRoadmaps'>
            <IndexRoute component={SendDoorRoadmaps} onEnter={authenticate} />
            <Route name='clientSendDoorRoadmapDetail' path='detail' component={SendDoorRoadmapDetail} onEnter={authenticate} />
        </Route>
        <Route name='clientTrucks' path='trucks'>
            <IndexRoute component={Trucks} onEnter={authenticate} />
            <Route name='clientTruckDetail' path='detail' component={TruckDetail} onEnter={authenticate} />
            <Route name='clientPayForCarryPartment' path='payForCarryPartment' component={PayForCarryPartment} onEnter={authenticate} />
            <Route name='clientSelectCarryPartment' path='selectCarryPartment' component={SelectCarryPartment} onEnter={authenticate} />
            <Route name='clientTruckShowDetail' path='showDetail' component={TruckShowDetail} onEnter={authenticate} />
            <Route name='clientOrderListInTruck' path='orderList' component={OrderListInTruck} onEnter={authenticate} />
            <Route name='clientLookTruckFee' path='lookTruckFee' component={LookTruckFee} onEnter={authenticate} />
        </Route>
        <Route name='clientPreOrders' path='preOrders'>
            <IndexRoute component={PreOrders} onEnter={authenticate} />
            <Route name='clientPreOrderDetail' path='detail' component={PreOrderDetail} onEnter={authenticate} />
            <Route name='clientPreOrderGroupDetail' path='groupDetail' component={PreOrderGroupDetail} onEnter={authenticate} />
            <Route name='clientLookPreOrderFee' path='lookPreOrderFee' component={LookPreOrderFee} onEnter={authenticate} />
            <Route name='clientLookOrderFee' path='lookOrderFee' component={LookOrderFee} onEnter={authenticate} />
            <Route name='clientPreOrderLookGroupFee' path='lookGroupFee' component={PreOrderLookGroupFee} onEnter={authenticate} />
        </Route>
        <Route name='clientQuotations' path='quotations' component={Quotations} onEnter={authenticate} />
        <Route name='clientNeedSendOrders' path='needSendOrders' >
            <IndexRoute component={NeedSendOrders} onEnter={authenticate} />
            <Route name='clientOrderListByEndPoint' path='orderListByEndPoint' component={OrderListByEndPoint} onEnter={authenticate} />
        </Route>
        <Route name='clientOrderDetail' path='orderDetail' component={OrderDetail} onEnter={authenticate} />
        <Route name='clientBills' path='bills' component={Bills} onEnter={authenticate} />
        <Route name='clientSPOrders' path='sporders' component={SPOrders} onEnter={authenticate} />
        <Route name='clientSCOrders' path='scorders' component={SCOrders} onEnter={authenticate} />
        <Route name='clientStatistics' path='statistics' component={Statistics} onEnter={authenticate} />
        <Route name='clientFeedback' path='feedback' component={Feedback} onEnter={authenticate} />
        <Route name='clientNotify' path='notify' component={Notify} onEnter={authenticate} />
        <Route name='agentMembers' path='agentMembers'>
            <IndexRoute component={AgentMembers} onEnter={authenticate} />
            <Route name='agentMemberDetail' path='detail' component={AgentMemberDetail} onEnter={authenticate} />
            <Route name='agentMemberRegister' path='register' component={AgentMemberRegister} onEnter={authenticate} />
        </Route>
        <Route name='agentRegionRates' path='regionRates'>
            <IndexRoute component={RegionRates} onEnter={authenticate} />
            <Route name={'regionRateDetail'} path={'detail'} component={RegionRate} onEnter={authenticate} />
        </Route>
        <Route name='clientPlaceOrder' path='placeOrder' component={PlaceOrder} onEnter={authenticate} />
        <Route name='clientRCOrders' path='rcorders'>
            <IndexRoute component={RCOrders} onEnter={authenticate} />
            <Route name='clientRCOrderDetail' path='detail' component={RCOrderDetail} onEnter={authenticate} />
        </Route>
        <Route name='clientSendOrders' path='sendOrders'>
            <IndexRoute component={SendOrders} onEnter={authenticate} />
            <Route name='clientSendOrderDetail' path='detail' component={SendOrderDetail} onEnter={authenticate} />
        </Route>
        <Route name='clientReceiveOrders' path='receiveOrders'>
            <IndexRoute component={ReceiveOrders} onEnter={authenticate} />
            <Route name='clientReceiveOrderDetail' path='detail' component={ReceiveOrderDetail} onEnter={authenticate} />
        </Route>
    </Route>,
];

```

* PDShop_Client_PC/project/App/shared/routers/public.js

```js
// import React from 'react';
// import {Route} from 'react-router';

export default [];

```

* PDShop_Client_PC/project/App/server/graphql/mutations/index.js

```js
import * as personal from './personal';
import * as member from './member';
import * as roadmap from './roadmap';
import * as truck from './truck';
import * as order from './order';
import * as agentOrder from './agentOrder';
import * as regionRate from './regionRate';

export default {
    ...personal,
    ...member,
    ...truck,
    ...roadmap,
    ...order,
    ...agentOrder,
    ...regionRate,
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/index.js

```js
import * as common from './common';
import * as personal from './personal';
import * as account from './account';
import * as member from './member';
import * as roadmap from './roadmap';
import * as order from './order';
import * as agentOrder from './agentOrder';
import * as quotation from './quotation';
import * as truck from './truck';
import * as statistic from './statistic';
import * as client from './client';
import * as regionRate from './regionRate';

export default {
    ...common,
    ...personal,
    ...account,
    ...member,
    ...roadmap,
    ...order,
    ...agentOrder,
    ...quotation,
    ...truck,
    ...statistic,
    ...client,
    ...regionRate,
};

```

* PDShop_Client_PC/project/App/server/graphql/types/account.js

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

* PDShop_Client_PC/project/App/server/graphql/types/agent.js

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

import { makeResultType, descriptType, descriptInputType } from './common';
import { clientType } from './client';
import { shopType } from './shop';

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
        // 分店
        referShop: { type: shopType },
    },
});
export const agentResultType = makeResultType('agentResultType', agentType);

export const agentInputType = new GraphQLInputObjectType({
    name: 'agentInputType',
    fields: {
        logo: { type: GraphQLString },
        image: { type: GraphQLString },
        sign: { type: GraphQLString },
        addressRegion: { type: GraphQLString },
        addressRegionLastCode: { type: GraphQLInt },
        address: { type: GraphQLString },
        location: { type: new GraphQLList(GraphQLFloat) },
        phoneList:  { type: GraphQLString },
        descriptList:  { type: new GraphQLList(descriptInputType) },

    },
});

```

* PDShop_Client_PC/project/App/server/graphql/types/bill.js

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

* PDShop_Client_PC/project/App/server/graphql/types/client.js

```js
import {
    GraphQLObjectType,
    GraphQLString,
    GraphQLInt,
    GraphQLID,
    GraphQLList,
} from 'graphql';

import { makeResultType, makeListType } from './common';

export const clientType = new GraphQLObjectType({
    name: 'clientType',
    fields: {
        id: { type: GraphQLID },
        name: { type: GraphQLString },
        phone: { type: GraphQLString },
        email: { type: GraphQLString },
        address: { type: GraphQLString },
        phoneList: { type: GraphQLString },
        authority: { type: new GraphQLList(GraphQLInt) },
    },
});
export const memberResultType = makeResultType('memberResultType', clientType);
export const memberListType = makeListType('memberListType', clientType);

```

* PDShop_Client_PC/project/App/server/graphql/types/common.js

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

export const startPointType = new GraphQLObjectType({
    name: 'startPointType',
    fields: {
        parentCode: { type: GraphQLInt },
        code: { type: GraphQLInt },
        level: { type: GraphQLInt },
        name: { type: GraphQLString },
        id: { type: GraphQLID },
        isShop: { type: GraphQLBoolean },
        isAgent: { type: GraphQLBoolean },
        addressRegionLastCode: { type: GraphQLInt },
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

* PDShop_Client_PC/project/App/server/graphql/types/order.js

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
import { clientType } from './client';
import { shopType } from './shop';
import { agentType } from './agent';
import { shipperType } from './shipper';

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
        startPoint: { type: GraphQLString },
        startPointLastCode: { type: GraphQLInt },
        shop: { type: shopType },
        shipper: { type: shipperType },
        agent: { type: agentType },
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
        isInsuance: { type: GraphQLBoolean },
        state: { type: GraphQLInt },
        placeOrderTime: { type: GraphQLString },
        // 价格
        fee: { type: GraphQLFloat }, // 物流公司的运费
        realFee: { type: GraphQLFloat }, // 实际运费
        designatedFee: { type: GraphQLFloat }, // 指定收款
        needPayTransportFee: { type: GraphQLFloat }, // 应该付的运费
        needPayInsuanceFee: { type: GraphQLFloat }, // 应该付的保险金额
        photo: { type: GraphQLString }, // 图片
        // 提成
        profit: { type: GraphQLFloat },
        // 其他
        isCityDistribute: { type: GraphQLBoolean }, // 是否是同城配送
        payMode: { type: GraphQLInt }, // 支付方式 0：现付（PM_IMMEDIATE），1：到付（PM_REACH），2：混合支付（PM_MIXED 指定收款的部分到付，其余的现付）
        isTransferOrder: { type: GraphQLBoolean }, // 是否是中转单
        receiveAgentMember: { type: clientType }, // 收货员
        stateList: { type: new GraphQLList(stateListType) }, // 状态列表
        deductError: { type: GraphQLBoolean }, // 扣款是否失败
        // 附加信息
        unloadNumber: { type: GraphQLInt }, // 未装载的数量
    },
});
export const orderResultType = makeResultType('orderResultType', orderType);
export const orderListType = makeListType('orderListType', orderType);
export const orderItemListType = makeListType('orderItemListType', orderType, 'list');

export const orderInputType = new GraphQLInputObjectType({
    name: 'orderInputType',
    fields: {
        orderId: { type: GraphQLID },
        orderGroupId: { type: GraphQLID },
        preOrderId: { type: GraphQLID },
        senderPhone: { type: GraphQLString },
        senderName: { type: GraphQLString },
        receiverPhone: { type: GraphQLString },
        receiverName: { type: GraphQLString },
        name: { type: GraphQLString },
        startPoint: { type: GraphQLString },
        startPointLastCode: { type: GraphQLInt },
        shopId: { type: GraphQLID },
        agentId: { type: GraphQLID },
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
    },
});

export const orderFeeType = new GraphQLObjectType({
    name: 'orderFeeType',
    fields: {
        id: { type: GraphQLID },
        endPoint: { type: GraphQLString },
        selfSignRate: { type: GraphQLFloat },
        duration: { type: GraphQLFloat },
        distance: { type: GraphQLFloat },
        shipperName: { type: GraphQLString },
        shopName: { type: GraphQLString },
        shopAddress: { type: GraphQLString },
        agentName: { type: GraphQLString },
        agentAddress: { type: GraphQLString },
        transportFee: { type: GraphQLFloat },
        price: { type: GraphQLFloat },
        minFee: { type: GraphQLFloat },
        sendDoorPrice: { type: GraphQLFloat },
        sendDoorMinFee: { type: GraphQLFloat },
    },
});
export const orderFeeListType = makeListType('orderFeeListType', orderFeeType, 'roadmapList');
export const orderFeesType = new GraphQLObjectType({
    name: 'orderFeesType',
    fields: {
        shop: { type: orderFeeListType },
        agent: { type: orderFeeListType },
    },
});

export const needSendOrderType = new GraphQLObjectType({
    name: 'needSendOrderType',
    fields: {
        endPoint: { type: GraphQLString },
        orderCount: { type: GraphQLInt },
        totalNumbers: { type: GraphQLInt },
        totalWeight: { type: GraphQLFloat },
        totalSize: { type: GraphQLFloat },
        totalFee: { type: GraphQLFloat },
    },
});
export const needSendOrderListType = makeListType('needSendOrderListType', needSendOrderType, 'orderList');

```

* PDShop_Client_PC/project/App/server/graphql/types/orderGroup.js

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
import { orderType } from './order';
import { shopType } from './shop';
import { agentType } from './agent';

export const orderGroupType = new GraphQLObjectType({
    name: 'orderGroupType',
    fields: {
        id: { type: GraphQLID },
        startPoint: { type: GraphQLString },
        startPointLastCode: { type: GraphQLInt },
        shop: { type: shopType },
        agent: { type: agentType },
        name: { type: GraphQLString },
        orderCount: { type: GraphQLInt },
        totalNumbers: { type: GraphQLInt },
        totalWeight: { type: GraphQLFloat },
        totalSize: { type: GraphQLFloat },
    },
});
export const orderGroupResultType = makeResultType('orderGroupResultType', orderGroupType);
export const orderGroupListType = makeListType('orderGroupListType', orderGroupType);

export const orderGroupInputType = new GraphQLInputObjectType({
    name: 'orderGroupInputType',
    fields: {
        orderGroupId: { type: GraphQLID },
        groupName: { type: GraphQLString },
        startPoint: { type: GraphQLString },
        startPointLastCode: { type: GraphQLInt },
        shopId: { type: GraphQLID },
        agentId: { type: GraphQLID },
        orderGroupIdList:  { type: new GraphQLList(GraphQLID) },
        orderIdList:  { type: new GraphQLList(GraphQLID) },
    },
});

export const orderGroupFeeType = new GraphQLObjectType({
    name: 'orderGroupFeeType',
    fields: {
        id: { type: GraphQLID },
        totalTransportFee: { type: GraphQLFloat },
        name: { type: GraphQLString },
        address: { type: GraphQLString },
        distance: { type: GraphQLFloat },
        unfindOrderList: { type: new GraphQLList(orderType) },
    },
});
export const orderGroupFeesType = new GraphQLObjectType({
    name: 'orderGroupFeesType',
    fields: {
        shopList: { type: new GraphQLList(orderGroupFeeType) },
        agentList: { type: new GraphQLList(orderGroupFeeType) },
    },
});

```

* PDShop_Client_PC/project/App/server/graphql/types/personal.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLID,
    GraphQLString,
    GraphQLInt,
    GraphQLList,
} from 'graphql';
import { makeResultType } from './common';
import { shipperType } from './shipper';
import { agentType } from './agent';
import { truckType } from './truck';

export const personalInfoType = new GraphQLObjectType({
    name: 'personalInfoType',
    fields: {
        userId: { type: GraphQLID },
        phone: { type: GraphQLString },
        email: { type: GraphQLString },
        name: { type: GraphQLString },
        sex: { type: GraphQLInt },
        head: { type: GraphQLString },
        birthday: { type: GraphQLString },
        age: { type: GraphQLInt },
        address: { type: GraphQLString },
        phoneList: { type: GraphQLString },
        authority: { type: new GraphQLList(GraphQLInt) },
        shipper: { type: shipperType },
        agent: { type: agentType },
        truck: { type: truckType },
    },
});
export const personalInfoResultType = makeResultType('personalInfoResultType', personalInfoType);

export const personalInfoInputType = new GraphQLInputObjectType({
    name: 'personalInfoInputType',
    fields: {
        userId: { type: GraphQLID },
        phone: { type: GraphQLString },
        email: { type: GraphQLString },
        name: { type: GraphQLString },
        sex: { type: GraphQLInt },
        head: { type: GraphQLString },
        birthday: { type: GraphQLString },
        address: { type: GraphQLString },
        phoneList: { type: GraphQLString },
        password: { type: GraphQLString },
        verifyCode: { type: GraphQLString },
    },
});

```

* PDShop_Client_PC/project/App/server/graphql/types/roadmap.js

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
import { shopType } from './shop';

const sendDoorRoadmapType = new GraphQLObjectType({
    name: 'sendDoorRoadmapType',
    fields: {
        enable: { type: GraphQLBoolean },
        sendDoorEndPoint: { type: GraphQLString },
        sendDoorEndPointLastCode: { type: GraphQLInt },
        sendDoorPrice: { type: GraphQLFloat },
        baseSendDoorPrice: { type: GraphQLFloat },
        updateSendDoorPriceTime: { type: GraphQLString },
        sendDoorMinFee: { type: GraphQLFloat },
        baseSendDoorMinFee: { type: GraphQLFloat },
        updateSendDoorMinFeeTime: { type: GraphQLString },
    },
});

export const roadmapType = new GraphQLObjectType({
    name: 'roadmapType',
    fields: {
        id: { type: GraphQLID },
        startPoint: { type: GraphQLString },
        endPoint: { type: GraphQLString },
        endPointLastCode: { type: GraphQLInt },
        price: { type: GraphQLFloat },
        basePrice: { type: GraphQLFloat },
        updatePriceTime: { type: GraphQLString },
        minFee: { type: GraphQLFloat },
        baseMinFee: { type: GraphQLFloat },
        updateMinFeeTime: { type: GraphQLString },
        duration: { type: GraphQLFloat },
        selfSignRate: { type: GraphQLFloat },
        modifyTime: { type: GraphQLString },
        enable: { type: GraphQLBoolean },
        sendDoorEnable: { type: GraphQLBoolean },
        sendDoorList: { type: new GraphQLList(sendDoorRoadmapType) },
    },
});
export const roadmapResultType = makeResultType('roadmapResultType', roadmapType);
export const roadmapListType = makeListType('roadmapListType', roadmapType);

export const roadmapInputType = new GraphQLInputObjectType({
    name: 'roadmapInputType',
    fields: {
        roadmapId: { type: GraphQLID },
        shopId: { type: GraphQLID },
        enable: { type: GraphQLBoolean },
        endPoint: { type: GraphQLString },
        endPointLastCode: { type: GraphQLInt },
        price: { type: GraphQLFloat },
        basePrice: { type: GraphQLFloat },
        minFee: { type: GraphQLFloat },
        baseMinFee: { type: GraphQLFloat },
        isCityDistribute: { type: GraphQLBoolean },
    },
});
export const setRoadmapPriceType = new GraphQLInputObjectType({
    name: 'setRoadmapPriceType',
    fields: {
        roadmapIdList: { type: new GraphQLList(GraphQLID) },
        mode: { type: GraphQLInt },
        typeList: { type: new GraphQLList(GraphQLInt) },
        value: { type: GraphQLFloat },
    },
});
export const setRoadmapSendDoorPriceType = new GraphQLInputObjectType({
    name: 'setRoadmapSendDoorPriceType',
    fields: {
        roadmapId: { type: GraphQLID },
        lastCodeList: { type: new GraphQLList(GraphQLInt) },
        mode: { type: GraphQLInt },
        typeList: { type: new GraphQLList(GraphQLInt) },
        value: { type: GraphQLFloat },
    },
});

export const regionRateType = new GraphQLObjectType({
    name: 'regionRateType',
    fields: {
        id: { type: GraphQLID },
        // 路线列表的参数
        shop: { type: shopType },
        region: { type: GraphQLString },
        type: { type: GraphQLInt },
        regionLastCode: { type: GraphQLInt },
        profitRate: { type: GraphQLFloat },
        profitCount: { type: GraphQLFloat },
        createTime: { type: GraphQLString },
        modifyTime: { type: GraphQLString },
    },
});
export const regionRateResultType = makeResultType('regionRateResultType', regionRateType);
export const regionRateListType = makeListType('regionRateListType', regionRateType);
export const regionRateInputType = new GraphQLInputObjectType({
    name: 'regionRateInputType',
    fields: {
        regionId:  { type: GraphQLID },
        regionRateIdList: { type: new GraphQLList(GraphQLID) },
        shopId: { type: GraphQLID },
        region: { type: GraphQLString },
        regionLastCode: { type: GraphQLInt },
        profitRate: { type: GraphQLFloat },
        profitCount: { type: GraphQLFloat },
        type: { type: GraphQLInt },
    },
});

```

* PDShop_Client_PC/project/App/server/graphql/types/shipper.js

```js
import {
    GraphQLInputObjectType,
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
    GraphQLID,
    GraphQLInt,
    GraphQLList,
    GraphQLBoolean,
} from 'graphql';

import { makeResultType, makeListType, descriptType, descriptInputType } from './common';
import { clientType } from './client';
import { shopType } from './shop';

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
        // 入驻分店性信息
        registerShopList: { type: new GraphQLList(shopType) },
        // 上门自提
        clientPickPrice: { type: GraphQLFloat },
        clientPickEnable: { type: GraphQLBoolean },
        // 担保公司
        bondCompanyList: { type: new GraphQLList(bondCompanyType) },
    },
});
export const shipperListType = makeListType('shipperListType', shipperType);
export const shipperResultType = makeResultType('shipperResultType', shipperType);

export const shipperInputType = new GraphQLInputObjectType({
    name: 'shipperInputType',
    fields: {
        shopId: { type: GraphQLID },
        logo: { type: GraphQLString },
        image: { type: GraphQLString },
        sign: { type: GraphQLString },
        address: { type: GraphQLString },
        phoneList:  { type: GraphQLString },
        descriptList:  { type: new GraphQLList(descriptInputType) },
        clientPickPrice: { type: GraphQLFloat },
        clientPickEnable: { type: GraphQLBoolean },
    },
});

```

* PDShop_Client_PC/project/App/server/graphql/types/shop.js

```js
import {
    GraphQLObjectType,
    GraphQLString,
    GraphQLFloat,
    GraphQLInt,
    GraphQLID,
    GraphQLList,
} from 'graphql';
import { makeListType } from './common';

export const carryPartmentType = new GraphQLObjectType({
    name: 'carryPartmentType',
    fields: {
        id: { type: GraphQLID },
        name: { type: GraphQLString },
        price: { type: GraphQLFloat },
        memberCount: { type: GraphQLInt },
        descript: { type: GraphQLString },
        phoneList: { type: GraphQLString },
    },
});

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
        chairMan: { type: memberInShopType },
        // 担保公司信息
        totalBondAmount: { type: GraphQLFloat }, // 总的担保金额
        remainBondAmount: { type: GraphQLFloat }, // 剩余可用的担保金额
        registerTime: { type: GraphQLString }, // 注册时间
    },
});

export const branchShopListType = makeListType('branchShopListType', shopType);

```

* PDShop_Client_PC/project/App/server/graphql/types/statistic.js

```js
import {
    GraphQLObjectType,
    GraphQLFloat,
    GraphQLInt,
    GraphQLList,
} from 'graphql';

export const shipperStatisticType = new GraphQLObjectType({
    name: 'shipperStatisticType',
    fields: {
        count: { type: new GraphQLList(GraphQLInt) },
        isReachPay: { type: new GraphQLList(GraphQLInt) },
        fee: { type: new GraphQLList(GraphQLFloat) },
        totalDesignatedFee: { type: new GraphQLList(GraphQLFloat) },
        proxyCharge: { type: new GraphQLList(GraphQLFloat) },
        weight: { type: new GraphQLList(GraphQLFloat) },
        size: { type: new GraphQLList(GraphQLFloat) },
    },
});

```

* PDShop_Client_PC/project/App/server/graphql/types/truck.js

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
import { carryPartmentType } from './shop';
import { clientType } from './client';
import { orderType } from './order';

export const truckTypeType = new GraphQLObjectType({
    name: 'truckTypeType',
    fields: {
        name: { type: GraphQLString }, // 名称
        list: { type: new GraphQLList(GraphQLString) }, // 类型
    },
});
export const driverType = new GraphQLObjectType({
    name: 'driverType',
    fields: {
        id: { type: GraphQLID }, // Id
        name: { type: GraphQLString }, // 姓名
        phone: { type: GraphQLString }, // 手机号码
    },
});

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
        // 搬运信息
        orderCount: { type: GraphQLInt }, // 总单数
        totalNumbers: { type: GraphQLInt }, // 总件数
        totalWeight: { type: GraphQLFloat }, // 总重量
        totalSize: { type: GraphQLFloat }, // 总方量
        carryPrice: { type: GraphQLFloat }, // 搬运价格
        unloadAllOrderList: { type: new GraphQLList(orderType) }, // 只上了一部分订单列表
        scanner: { type: clientType }, // 扫描员
        carryPartment: { type: carryPartmentType }, // 选择的搬运部
    },
});

export const truckResultType = makeResultType('truckResultType', truckType);
export const truckItemListType = makeListType('truckItemListType', truckType, 'list');
export const truckListType = makeListType('truckListType', truckType, 'truckList');
export const trucksType = new GraphQLObjectType({
    name: 'trucksType',
    fields: {
        onway: { type: truckItemListType },
        complete: { type: truckItemListType },
    },
});

export const truckInputType = new GraphQLInputObjectType({
    name: 'truckInputType',
    fields: {
        truckId: { type: GraphQLID }, // 货车 Id
        shopId: { type: GraphQLID }, // 所在分店 Id
        plateNo: { type: GraphQLString }, // 车牌号码
        drivingLicense: { type: GraphQLString }, // 行驶证编号
        truckType: { type: GraphQLString }, // 车型
        insuanceMount: { type: GraphQLFloat }, // 保险
        driverId: { type: GraphQLID }, // 司机的Id
    },
});

```

* PDShop_Client_PC/project/App/server/shared/helpers/bind-error-type.js

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

* PDShop_Client_PC/project/App/server/shared/helpers/default-favicon.js

```js
export default function getDefaultIcon (res) {
    return {
        tag: 'link',
        props: {
            rel: 'shortcut icon',
            type: 'image/vnd.microsoft.icon',
            href: `${res.baseScriptsURL}/client/favicon.ico`,
        },
    };
}

```

* PDShop_Client_PC/project/App/server/shared/helpers/get-base-component.js

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

* PDShop_Client_PC/project/App/server/shared/helpers/get-markup.js

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

* PDShop_Client_PC/project/App/server/shared/helpers/get-projection.js

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

* PDShop_Client_PC/project/App/server/shared/helpers/render-html.js

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

* PDShop_Client_PC/project/App/server/shared/helpers/route-handler.js

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

* PDShop_Client_PC/project/App/shared/pages/client/index.js

```js
import React from 'react';
import { rootDataConnect } from 'relatejs';
import Client from './contents';
import _ from 'lodash';

@rootDataConnect()
export default class ClientContainer extends React.Component {
    state = {
        personal: {},
        currentShop: {},
        printShow: false,
    };
    updatePersonal (personal) {
        this.setState({ personal });
    }
    updateCurrentShop (currentShop) {
        this.setState({ currentShop });
    }
    updatePrintShow (printShow) {
        this.setState({ printShow });
    }
    hasAuthority (...authority) {
        const { personal } = this.state;
        return !!_.intersection(personal.authority, authority).length;
    }
    render () {
        const { personal, currentShop, printShow } = this.state;
        return (
            <Client {...this.props} printShow={printShow} rootPersonal={personal} rootShop={currentShop} hasAuthority={::this.hasAuthority}>
                {
                    React.cloneElement(this.props.children, {
                        rootPersonal: personal,
                        updatePersonal: ::this.updatePersonal,
                        rootShop: currentShop,
                        updateCurrentShop: ::this.updateCurrentShop,
                        updatePrintShow: ::this.updatePrintShow,
                        hasAuthority: ::this.hasAuthority,
                    })
                }
            </Client>
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/relatejs/actions/mutation.js

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

* PDShop_Client_PC/project/App/shared/relatejs/actions/query.js

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

* PDShop_Client_PC/project/App/shared/relatejs/actions/remove-connector.js

```js
import actionTypes from './types';

export default function removeConnector (id) {
    return {
        type: actionTypes.removeConnector,
        id,
    };
}

```

* PDShop_Client_PC/project/App/shared/relatejs/actions/settings.js

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

* PDShop_Client_PC/project/App/shared/relatejs/actions/types.js

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

* PDShop_Client_PC/project/App/shared/relatejs/decorators/data-connect.js

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

* PDShop_Client_PC/project/App/shared/relatejs/decorators/root-data-connect.js

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

* PDShop_Client_PC/project/App/shared/relatejs/helpers/capture.js

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

* PDShop_Client_PC/project/App/shared/relatejs/helpers/connectors-ids.js

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

* PDShop_Client_PC/project/App/shared/relatejs/helpers/fragments.js

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

* PDShop_Client_PC/project/App/shared/relatejs/helpers/get-variables.js

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

* PDShop_Client_PC/project/App/shared/relatejs/helpers/request.js

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
        .post(endpoint || '/client/graphql')
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
                    window.location.href = '/client/login';
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

* PDShop_Client_PC/project/App/shared/relatejs/redirect-graphql/index.js

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

* PDShop_Client_PC/project/App/shared/relatejs/reducer/reducer.js

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
    endpoint: '/client/graphql',
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

* PDShop_Client_PC/project/App/shared/relatejs/server/get-data-dependencies.js

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

* PDShop_Client_PC/project/App/shared/relatejs/store/connectors.js

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

* PDShop_Client_PC/project/App/shared/relatejs/store/global-data.js

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

* PDShop_Client_PC/project/App/shared/relatejs/store/index.js

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

* PDShop_Client_PC/project/App/shared/relatejs/store/keep-data.js

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

* PDShop_Client_PC/project/App/server/graphql/mutations/agentOrder/confirmCachPayed.js

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
        return await post(urls.confirmCachPayed, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/agentOrder/confirmCachPayedForOrderList.js

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
        return await post(urls.confirmCachPayedForOrderList, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/agentOrder/index.js

```js
export placeClientOrder from './placeClientOrder';
export placeClientOrderWithPreOrder from './placeClientOrderWithPreOrder';
export modifyClientOrder from './modifyClientOrder';
export confirmCachPayed from './confirmCachPayed';
export confirmCachPayedForOrderList from './confirmCachPayedForOrderList';
export printOrderBill from './printOrderBill';
export printOrderListBill from './printOrderListBill';
export printBarCode from './printBarCode';
export removeClientOrder from './removeClientOrder';

```

* PDShop_Client_PC/project/App/server/graphql/mutations/agentOrder/modifyClientOrder.js

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
        return await post(urls.modifyClientOrder, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/agentOrder/placeClientOrder.js

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
        return await post(urls.placeClientOrder, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/agentOrder/placeClientOrderWithPreOrder.js

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
        return await post(urls.placeClientOrderWithPreOrder, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/agentOrder/printBarCode.js

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

* PDShop_Client_PC/project/App/server/graphql/mutations/agentOrder/printOrderBill.js

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

* PDShop_Client_PC/project/App/server/graphql/mutations/agentOrder/printOrderListBill.js

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

* PDShop_Client_PC/project/App/server/graphql/mutations/agentOrder/removeClientOrder.js

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
        return await post(urls.removeClientOrder, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/member/index.js

```js
export modifyShipperMemberAuthority from './modifyShipperMemberAuthority';
export modifyAgentMemberAuthority from './modifyAgentMemberAuthority';

```

* PDShop_Client_PC/project/App/server/graphql/mutations/member/modifyAgentMemberAuthority.js

```js
import {
    GraphQLInt,
    GraphQLID,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { memberResultType } from '../../types/client';
import { post, urls } from 'helpers/api';

export default {
    type: memberResultType,
    args: {
        memberId: {
            type: GraphQLID,
        },
        authority: {
            type: new GraphQLList(GraphQLInt),
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let res = await post(urls.modifyAgentMemberAuthority, params, root) || { msg: '服务器错误' };

        return res;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/member/modifyshipperMemberAuthority.js

```js
import {
    GraphQLInt,
    GraphQLID,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { memberResultType } from '../../types/client';
import { post, urls } from 'helpers/api';

export default {
    type: memberResultType,
    args: {
        memberId: {
            type: GraphQLID,
        },
        authority: {
            type: new GraphQLList(GraphQLInt),
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyShipperMemberAuthority, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/order/createPreOrderGroup.js

```js
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { orderGroupInputType } from '../../types/orderGroup';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        data: {
            type: orderGroupInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.createPreOrderGroup, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/order/finishOrder.js

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
        orderId: {
            type: GraphQLID,
        },
        verifyCode: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.finishOrder, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/order/index.js

```js
export placePreOrder from './placePreOrder';
export modifyPreOrder from './modifyPreOrder';
export removePreOrder from './removePreOrder';
export createPreOrderGroup from './createPreOrderGroup';
export modifyPreOrderGroup from './modifyPreOrderGroup';
export removePreOrderFromGroup from './removePreOrderFromGroup';
export payForOrderWhenSend from './payForOrderWhenSend';
export finishOrder from './finishOrder';
export setClientPickOrderTransportFee from './setClientPickOrderTransportFee';

```

* PDShop_Client_PC/project/App/server/graphql/mutations/order/modifyPreOrder.js

```js
import { authorize } from '../../authorize';
import { orderResultType, orderInputType } from '../../types/order';
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
        return await post(urls.modifyPreOrder, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/order/modifyPreOrderGroup.js

```js
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { orderGroupInputType } from '../../types/orderGroup';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        data: {
            type: orderGroupInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyPreOrderGroup, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/order/payForOrderWhenSend.js

```js
import {
    GraphQLID,
    GraphQLList,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        payPassword: {
            type: GraphQLString,
        },
        orderIdList: {
            type: new GraphQLList(GraphQLID),
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.payForOrderWhenSend, params, root) || { msg: '服务器繁忙,请稍后重试' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/order/placePreOrder.js

```js
import { authorize } from '../../authorize';
import { orderResultType, orderInputType } from '../../types/order';
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
        return await post(urls.placePreOrder, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/order/removePreOrder.js

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
        return await post(urls.removePreOrder, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/order/removePreOrderFromGroup.js

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
        orderGroupId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.removePreOrderFromGroup, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/order/setClientPickOrderTransportFee.js

```js
import {
    GraphQLID,
    GraphQLBoolean,
    GraphQLString,
    GraphQLFloat,
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
        isTransport: {
            type: GraphQLBoolean,
        },
        endPoint: {
            type: GraphQLString,
        },
        transportFee: {
            type: GraphQLFloat,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.setClientPickOrderTransportFee, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/regionRate/addRegionProfit.js

```js
import { authorize } from '../../authorize';
import { regionRateResultType, regionRateInputType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: regionRateResultType,
    args: {
        data:{
            type:regionRateInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.addRegionProfit, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/regionRate/index.js

```js
export addRegionProfit from './addRegionProfit';
export modifyRegionProfit from './modifyRegionProfit';
export removeRegionProfit from './removeRegionProfit';
export setRegionProfitWithList from './setRegionProfitWithList';

```

* PDShop_Client_PC/project/App/server/graphql/mutations/regionRate/modifyRegionProfit.js

```js
import { authorize } from '../../authorize';
import { regionRateResultType, regionRateInputType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: regionRateResultType,
    args: {
        data:{
            type:regionRateInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyRegionProfit, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/regionRate/removeRegionProfit.js

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
        regionId:{
            type:GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.removeRegionProfit, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/regionRate/setRegionProfitWithList.js

```js
import {
    GraphQLFloat,
    GraphQLID,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        regionIdList: {
            type: new GraphQLList(GraphQLID),
        },
        profitRate: {
            type: GraphQLFloat,
        },
        profitCount: {
            type: GraphQLFloat,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.setRegionProfitWithList, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/personal/feedback.js

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

* PDShop_Client_PC/project/App/server/graphql/mutations/personal/forgotPwd.js

```js
import { successType } from '../../types/common';
import { personalInfoInputType } from '../../types/personal';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        data: {
            type: personalInfoInputType,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.forgotPwd, params.data) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/personal/index.js

```js
export register from './register';
export forgotPwd from './forgotPwd';
export setPaymentPassword from './setPaymentPassword';
export modifyPersonalInfo from './modifyPersonalInfo';
export modifyShipperInfo from './modifyShipperInfo';
export modifyAgentInfo from './modifyAgentInfo';
export feedback from './feedback';
export setReferShop from './setReferShop';

```

* PDShop_Client_PC/project/App/server/graphql/mutations/personal/modifyAgentInfo.js

```js
import { authorize } from '../../authorize';
import { agentResultType, agentInputType } from '../../types/agent';
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
        return await post(urls.modifyAgentInfo, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/personal/modifyPersonalInfo.js

```js
import { authorize } from '../../authorize';
import { personalInfoResultType, personalInfoInputType } from '../../types/personal';
import { post, urls } from 'helpers/api';

export default {
    type: personalInfoResultType,
    args: {
        data: {
            type: personalInfoInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyPersonalInfo, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/personal/modifyShipperInfo.js

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
        return await post(urls.modifyShipperInfo, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/personal/register.js

```js
import { successType } from '../../types/common';
import { personalInfoInputType } from '../../types/personal';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        data: {
            type: personalInfoInputType,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.register, params.data) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/personal/setPaymentPassword.js

```js
import { successType } from '../../types/common';
import { personalInfoInputType } from '../../types/personal';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        data: {
            type: personalInfoInputType,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.setPaymentPassword, params.data) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/personal/setReferShop.js

```js
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';
import {
    GraphQLID,
} from 'graphql';

export default {
    type: successType,
    args: {
        shopId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.setReferShop, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/roadmap/index.js

```js
export publishRoadmap from './publishRoadmap';
export modifyRoadmap from './modifyRoadmap';
export removeRoadmap from './removeRoadmap';
export setRoadmapPrice from './setRoadmapPrice';
export setRoadmapSendDoorPrice from './setRoadmapSendDoorPrice';
export setRoadmapSendDoorBasePrice from './setRoadmapSendDoorBasePrice';
export setRoadmapEnable from './setRoadmapEnable';
export setRoadmapSendDoorEnable from './setRoadmapSendDoorEnable';

```

* PDShop_Client_PC/project/App/server/graphql/mutations/roadmap/modifyRoadmap.js

```js
import { authorize } from '../../authorize';
import { roadmapInputType, roadmapResultType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: roadmapResultType,
    args: {
        data: {
            type: roadmapInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyRoadmap, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/roadmap/publishRoadmap.js

```js
import { authorize } from '../../authorize';
import { roadmapInputType, roadmapResultType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: roadmapResultType,
    args: {
        data: {
            type: roadmapInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.publishRoadmap, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/roadmap/removeRoadmap.js

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
        roadmapId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.removeRoadmap, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/roadmap/setRoadmapEnable.js

```js
import {
    GraphQLID,
    GraphQLBoolean,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        roadmapIdList: {
            type: new GraphQLList(GraphQLID),
        },
        enable: {
            type: GraphQLBoolean,
        },

    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.setRoadmapEnable, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/roadmap/setRoadmapPrice.js

```js
import { authorize } from '../../authorize';
import { setRoadmapPriceType, roadmapResultType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: roadmapResultType,
    args: {
        data: {
            type: setRoadmapPriceType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.setRoadmapPrice, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/roadmap/setRoadmapSendDoorBasePrice.js

```js
import { authorize } from '../../authorize';
import { setRoadmapSendDoorPriceType, roadmapResultType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: roadmapResultType,
    args: {
        data: {
            type: setRoadmapSendDoorPriceType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.setRoadmapSendDoorBasePrice, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/roadmap/setRoadmapSendDoorEnable.js

```js
import {
    GraphQLID,
    GraphQLInt,
    GraphQLBoolean,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        roadmapId: {
            type: GraphQLID,
        },
        lastCodeList: {
            type: new GraphQLList(GraphQLInt),
        },
        enable: {
            type: GraphQLBoolean,
        },

    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.setRoadmapSendDoorEnable, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/roadmap/setRoadmapSendDoorPrice.js

```js
import { authorize } from '../../authorize';
import { setRoadmapSendDoorPriceType, roadmapResultType } from '../../types/roadmap';
import { post, urls } from 'helpers/api';

export default {
    type: roadmapResultType,
    args: {
        data: {
            type: setRoadmapSendDoorPriceType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.setRoadmapSendDoorPrice, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/truck/createTruck.js

```js
import { authorize } from '../../authorize';
import { truckInputType, truckResultType } from '../../types/truck';
import { post, urls } from 'helpers/api';

export default {
    type: truckResultType,
    args: {
        data: {
            type: truckInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.createTruck, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/truck/index.js

```js
export createTruck from './createTruck';
export modifyTruck from './modifyTruck';
export removeTruck from './removeTruck';
export selectCarryPartment from './selectCarryPartment';
export payforCarryPartment from './payforCarryPartment';
export setCityTruck from './setCityTruck';

```

* PDShop_Client_PC/project/App/server/graphql/mutations/truck/modifyTruck.js

```js
import { authorize } from '../../authorize';
import { truckInputType, truckResultType } from '../../types/truck';
import { post, urls } from 'helpers/api';

export default {
    type: truckResultType,
    args: {
        data: {
            type: truckInputType,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.modifyTruck, params.data, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/truck/payforCarryPartment.js

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
        return await post(urls.payforCarryPartment, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/truck/removeTruck.js

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
        return await post(urls.removeTruck, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/truck/selectCarryPartment.js

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
        carryPartmentId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.selectCarryPartment, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/mutations/truck/setCityTruck.js

```js
import {
    GraphQLString,
} from 'graphql';
import { successType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: successType,
    args: {
        plateNo: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.setCityTruck, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/account/bills.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/account/index.js

```js
export remainAmount from './remainAmount';
export weixinPayGetUnifiedOrder from './weixinPayGetUnifiedOrder';
export weixinPayWithDraw from './weixinPayWithDraw';
export bills from './bills';

```

* PDShop_Client_PC/project/App/server/graphql/queries/account/remainAmount.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/account/weixinPayGetUnifiedOrder.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/account/weixinPayWithDraw.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/agentOrder/addressWithOrder.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/agentOrder/index.js

```js
export rcorder from './rcorder';
export rcorders from './rcorders';
export selectPreOrders from './selectPreOrders';
export addressWithOrder from './addressWithOrder';
export sendDoorAddressWithOrder from './sendDoorAddressWithOrder';

```

* PDShop_Client_PC/project/App/server/graphql/queries/agentOrder/rcorder.js

```js
import { authorize } from '../../authorize';
import { orderType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderType,
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.rcorder, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/agentOrder/rcorders.js

```js
import {
    GraphQLObjectType,
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderItemListType } from '../../types/order';
import { post, urls } from 'helpers/api';

const rcordersType = new GraphQLObjectType({
    name: 'rcordersType',
    fields: {
        topay: { type: orderItemListType },
        toprintbill: { type: orderItemListType },
        tosend: { type: orderItemListType },
        onway: { type: orderItemListType },
        success: { type: orderItemListType },
    },
});

export default {
    type: rcordersType,
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
        let ret = await post(urls.rcorders, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/agentOrder/selectPreOrders.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/agentOrder/sendDoorAddressWithOrder.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/client/getClientNameByPhone.js

```js
import {
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { memberResultType } from '../../types/client';
import { post, urls } from 'helpers/api';

export default {
    type: memberResultType,
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

* PDShop_Client_PC/project/App/server/graphql/queries/client/getClientPickShipperList.js

```js
import { GraphQLID } from 'graphql';
import { authorize } from '../../authorize';
import { shipperListType } from '../../types/shipper';
import { post, urls } from 'helpers/api';

export default {
    type: shipperListType,
    args: {
        shopId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.getClientPickShipperList, params, root) || {};
        console.log('ret', ret);
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/client/index.js

```js
export getClientNameByPhone from './getClientNameByPhone';
export clientPickList from './getClientPickShipperList';

```

* PDShop_Client_PC/project/App/server/graphql/queries/common/address.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/common/addressFromLastCode.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/common/index.js

```js
export address from './address';
export addressFromLastCode from './addressFromLastCode';
export sendDoorAddressFromLastCode from './sendDoorAddressFromLastCode';
export startPointAddress from './startPointAddress';
export startPointAddressFromLastCode from './startPointAddressFromLastCode';

```

* PDShop_Client_PC/project/App/server/graphql/queries/common/sendDoorAddressFromLastCode.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/common/startPointAddress.js

```js
import {
    GraphQLInt,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { startPointType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: new GraphQLList(startPointType),
    args: {
        parentCode: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.startPointAddress, params, root) || {};
        return ret.success ? ret.context.addressList : [];
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/common/startPointAddressFromLastCode.js

```js
import {
    GraphQLInt,
    GraphQLList,
    GraphQLBoolean,
} from 'graphql';
import { authorize } from '../../authorize';
import { startPointType } from '../../types/common';
import { post, urls } from 'helpers/api';

export default {
    type: new GraphQLList(new GraphQLList(startPointType)),
    args: {
        addressLastCode: {
            type: GraphQLInt,
        },
        isLeaf: {
            type: GraphQLBoolean,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.startPointAddressFromLastCode, params, root) || {};
        return ret.success ? ret.context.addressList : [];
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/member/agentMember.js

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
        memberId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.agentMember, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/member/agentMembers.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { memberListType } from '../../types/client';
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
        let ret = await post(urls.agentMembers, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/member/getAgentMemberByPhone.js

```js
import {
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { memberResultType } from '../../types/client';
import { post, urls } from 'helpers/api';

export default {
    type: memberResultType,
    args: {
        memberPhone: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.getAgentMemberByPhone, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/member/getShipperMemberByPhone.js

```js
import {
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { memberResultType } from '../../types/client';
import { post, urls } from 'helpers/api';

export default {
    type: memberResultType,
    args: {
        memberPhone: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        return await post(urls.getShipperMemberByPhone, params, root) || { msg: '服务器错误' };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/member/index.js

```js
// shipper
export shipperMembers from './shipperMembers';
export shipperMember from './shipperMember';
export getShipperMemberByPhone from './getShipperMemberByPhone';

// agent
export agentMembers from './agentMembers';
export agentMember from './agentMember';
export getAgentMemberByPhone from './getAgentMemberByPhone';

```

* PDShop_Client_PC/project/App/server/graphql/queries/member/shipperMember.js

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
        memberId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.shipperMember, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/member/shipperMembers.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { memberListType } from '../../types/client';
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
        let ret = await post(urls.shipperMembers, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/clientReceiveOrders.js

```js
import { GraphQLString, GraphQLInt, GraphQLObjectType } from 'graphql';
import { authorize } from '../../authorize';
import { orderItemListType } from '../../types/order';
import { post, urls } from 'helpers/api';

const receiveOrdersType = new GraphQLObjectType({
    name: 'receiveOrdersType',
    fields: {
        unsend: { type: orderItemListType },
        toreceive: { type: orderItemListType },
        onway: { type: orderItemListType },
        success: { type: orderItemListType },
    },
});

export default {
    type: receiveOrdersType,
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
        const res = await post(urls.receiveOrders, params, root) || { msg: '服务器繁忙请稍后再试!!!' };
        return res.success ? res.context : { msg: res.msg };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/clientSendOrders.js

```js
import { GraphQLString, GraphQLInt, GraphQLID, GraphQLObjectType } from 'graphql';
import { authorize } from '../../authorize';
import { orderItemListType } from '../../types/order';
import { post, urls } from 'helpers/api';

const sendOrdersType = new GraphQLObjectType({
    name: 'sendOrdersType',
    fields: {
        topay: { type: orderItemListType },
        onway: { type: orderItemListType },
        success: { type: orderItemListType },
    },
});

export default {
    type: sendOrdersType,
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
        senderId: {
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
        const res = await post(urls.sendOrders, params, root) || { msg: '服务器繁忙请稍后再试!!!' };
        return res.success ? res.context : { msg: res.msg };
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/inGroupPreOrders.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderListType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderListType,
    args: {
        orderGroupId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.inGroupPreOrders, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/index.js

```js
export order from './order';
export sporders from './sporders';
export scorders from './scorders';
export preOrders from './preOrders';
export preOrderGroups from './preOrderGroups';
export inGroupPreOrders from './inGroupPreOrders';
export lookPreOrderFee from './lookPreOrderFee';
export lookOrderFee from './lookOrderFee';
export lookGroupFee from './lookGroupFee';
export lookTruckOrderFee from './lookTruckOrderFee';
export needSendOrders from './needSendOrders';
export orderListByEndPoint from './orderListByEndPoint';
export sendOrders from './clientSendOrders';
export receiveOrders from './clientReceiveOrders';

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/lookGroupFee.js

```js
import {
    GraphQLID,
    GraphQLFloat,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderGroupFeesType } from '../../types/orderGroup';
import { post, urls } from 'helpers/api';

export default {
    type: orderGroupFeesType,
    args: {
        orderGroupId: {
            type: GraphQLID,
        },
        location: {
            type: new GraphQLList(GraphQLFloat),
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.lookGroupFee, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/lookOrderFee.js

```js
import {
    GraphQLID,
    GraphQLFloat,
    GraphQLInt,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderFeeListType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderFeeListType,
    args: {
        orderId: {
            type: GraphQLID,
        },
        startPointLastCode: {
            type: GraphQLInt,
        },
        shopId: {
            type: GraphQLID,
        },
        agentId: {
            type: GraphQLID,
        },
        location: {
            type: new GraphQLList(GraphQLFloat),
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
        let ret = await post(urls.lookOrderFee, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/lookPreOrderFee.js

```js
import {
    GraphQLID,
    GraphQLFloat,
    GraphQLInt,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderFeesType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderFeesType,
    args: {
        orderId: {
            type: GraphQLID,
        },
        location: {
            type: new GraphQLList(GraphQLFloat),
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
        let ret = await post(urls.lookPreOrderFee, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/lookTruckOrderFee.js

```js
import {
    GraphQLID,
    GraphQLInt,
    GraphQLFloat,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderGroupFeeType } from '../../types/orderGroup';
import { post, urls } from 'helpers/api';

export default {
    type: new GraphQLList(orderGroupFeeType),
    args: {
        truckId: {
            type: GraphQLID,
        },
        startPointLastCode: {
            type: GraphQLInt,
        },
        shopId: {
            type: GraphQLID,
        },
        location: {
            type: new GraphQLList(GraphQLFloat),
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.lookTruckOrderFee, params, root) || {};
        return ret.success ? ret.context.shopList : [];
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/needSendOrders.js

```js
import {
    GraphQLInt,
    GraphQLString,
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { needSendOrderListType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: needSendOrderListType,
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
        let ret = await post(urls.needSendOrders, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/order.js

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
        if (!params.orderId) {
            return undefined;
        }
        let ret = await post(urls.order, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/orderListByEndpoint.js

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
        endPoint: {
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
        let ret = await post(urls.orderListByEndPoint, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/preOrderGroups.js

```js
import { authorize } from '../../authorize';
import { orderGroupListType } from '../../types/orderGroup';
import { post, urls } from 'helpers/api';

export default {
    type: orderGroupListType,
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.preOrderGroups, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/preOrders.js

```js
import {
    GraphQLInt,
    GraphQLString,
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
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.preOrders, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/scorders.js

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

const scordersType = new GraphQLObjectType({
    name: 'scordersType',
    fields: {
        tostore: { type: orderItemListType },
        toconfirm: { type: orderItemListType },
        waitpick: { type: orderItemListType },
        toload: { type: orderItemListType },
        onway: { type: orderItemListType },
        success: { type: orderItemListType },
    },
});

export default {
    type: scordersType,
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
        let ret = await post(urls.scorders, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/order/sporders.js

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

const spordersType = new GraphQLObjectType({
    name: 'spordersType',
    fields: {
        tostore: { type: orderItemListType },
        stored: { type: orderItemListType },
        tostartoff: { type: orderItemListType },
        onway: { type: orderItemListType },
        success: { type: orderItemListType },
    },
});

export default {
    type: spordersType,
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
        let ret = await post(urls.sporders, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/quotation/index.js

```js
export lookRoadmap from './lookRoadmap';

```

* PDShop_Client_PC/project/App/server/graphql/queries/quotation/lookRoadmap.js

```js
import {
    GraphQLID,
    GraphQLFloat,
    GraphQLInt,
    GraphQLBoolean,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderFeesType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderFeesType,
    args: {
        startPointLastCode: {
            type: GraphQLInt,
        },
        shopId: {
            type: GraphQLID,
        },
        agentId: {
            type: GraphQLID,
        },
        endPointLastCode: {
            type: GraphQLInt,
        },
        sendDoorEndPointLastCode: {
            type: GraphQLInt,
        },
        isSendDoor: {
            type: GraphQLBoolean,
        },
        location: {
            type: new GraphQLList(GraphQLFloat),
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
        if (!params.startPointLastCode || !params.endPointLastCode) {
            return undefined;
        }
        let ret = await post(urls.lookRoadmap, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/personal/agent.js

```js
import { authorize } from '../../authorize';
import { agentType } from '../../types/agent';
import { post, urls } from 'helpers/api';

export default {
    type: agentType,
    async resolve (root, params, options) {
        authorize(root);
        let agent = await post(urls.agent, params, root) || {};
        return agent.context || {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/personal/getUserNameByPhone.js

```js
import {
    GraphQLString,
} from 'graphql';
import { personalInfoResultType } from '../../types/personal';
import { post, urls } from 'helpers/api';

export default {
    type: personalInfoResultType,
    args: {
        phone: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        return await post(urls.getUserNameByPhone, params) || {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/personal/getVerifyCode.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/personal/index.js

```js
export session from './session';
export getVerifyCode from './getVerifyCode';
export login from './login';
export personal from './personal';
export shipper from './shipper';
export agent from './agent';
export getUserNameByPhone from './getUserNameByPhone';
export notify from './notify';
export statistics from './statistics';
export referShops from './referShops';

```

* PDShop_Client_PC/project/App/server/graphql/queries/personal/login.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/personal/notify.js

```js
import {
    GraphQLObjectType,
    GraphQLInt,
    GraphQLString,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { post, urls } from 'helpers/api';

const newsType = new GraphQLObjectType({
    name: 'newsType',
    fields: {
        title: { type: GraphQLString },
        time: { type: GraphQLString },
        source: { type: GraphQLString },
    },
});

const newsListType = new GraphQLObjectType({
    name: 'newsListType',
    fields: {
        count: { type: GraphQLInt },
        list: { type: new GraphQLList(newsType) },
    },
});

const notifyType = new GraphQLObjectType({
    name: 'notifyType',
    fields: {
        news: { type: newsListType },
        publicity: { type: newsListType },
        policy: { type: newsListType },
        notice: { type: newsListType },
    },
});

export default {
    type: notifyType,
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
        let ret = await post(urls.notify, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/personal/personal.js

```js
import { authorize } from '../../authorize';
import { personalInfoType } from '../../types/personal';
import { post, urls } from 'helpers/api';

export default {
    type: personalInfoType,
    async resolve (root, params, options) {
        authorize(root);
        const personal = await post(urls.personal, params, root) || {};
        return personal.context || {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/personal/referShops.js

```js
import {
    GraphQLString,
    GraphQLInt,
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
        let agent = await post(urls.getReferShopList, params, root) || {};
        return agent.context || {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/personal/session.js

```js
import { GraphQLBoolean } from 'graphql';

export default {
    type: GraphQLBoolean,
    resolve (root) {
        return root.isAuthenticated;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/personal/shipper.js

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
        shopId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let shipper = await post(urls.shipper, params, root) || {};
        return shipper.context || {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/personal/statistics.js

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

* PDShop_Client_PC/project/App/server/graphql/queries/regionRate/addressWithRegion.js

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
        regionId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.addressWithRegion, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/regionRate/index.js

```js
export regionRates from './regionRates';
export regionRate from './regionRate';
export addressWithRegion from './addressWithRegion';

```

* PDShop_Client_PC/project/App/server/graphql/queries/regionRate/regionRate.js

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
        regionId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.getRegionProfitDetail, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/regionRate/regionRates.js

```js
import {
    GraphQLInt,
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
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.getRegionProfitList, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/roadmap/index.js

```js
export roadmap from './roadmap';
export roadmaps from './roadmaps';

```

* PDShop_Client_PC/project/App/server/graphql/queries/roadmap/roadmap.js

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
        if (!params.roadmapId) {
            return undefined;
        }
        let ret = await post(urls.roadmap, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/roadmap/roadmaps.js

```js
import {
    GraphQLInt,
    GraphQLString,
    GraphQLID,
    GraphQLBoolean,
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
        isCityDistribute: {
            type: GraphQLBoolean,
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

* PDShop_Client_PC/project/App/server/graphql/queries/statistic/index.js

```js
export shipperStatistics from './shipperStatistics';

```

* PDShop_Client_PC/project/App/server/graphql/queries/statistic/shipperStatistics.js

```js
import { authorize } from '../../authorize';
import { shipperStatisticType } from '../../types/statistic';
import { post, urls } from 'helpers/api';

export default {
    type: shipperStatisticType,
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.shipperStatistics, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/truck/carryPartments.js

```js
import {
    GraphQLID,
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { carryPartmentType } from '../../types/shop';
import { post, urls } from 'helpers/api';

export default {
    type: new GraphQLList(carryPartmentType),
    args: {
        truckId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.carryPartments, params, root) || {};
        return ret.success ? ret.context.partmenList : [];
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/truck/getDriverNameByPhone.js

```js
import {
    GraphQLString,
} from 'graphql';
import { driverType } from '../../types/truck';
import { post, urls } from 'helpers/api';

export default {
    type: driverType,
    args: {
        phone: {
            type: GraphQLString,
        },
    },
    async resolve (root, params, options) {
        const ret = await post(urls.getDriverNameByPhone, params) || {};
        return ret.context;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/truck/historyTrucks.js

```js
import {
    GraphQLInt,
    GraphQLString,
    GraphQLID,
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
        type: {
            type: GraphQLString,
        },
        pageNo: {
            type: GraphQLInt,
        },
        pageSize: {
            type: GraphQLInt,
        },
        shopId:{
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.historyTrucks, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/truck/index.js

```js
export truck from './truck';
export truckTypes from './truckTypes';
export trucks from './trucks';
export workTruckList from './workTruckList';
export getDriverNameByPhone from './getDriverNameByPhone';
export carryPartments from './carryPartments';
export orderListInTruck from './orderListInTruck';
export historyTrucks from './historyTrucks';

```

* PDShop_Client_PC/project/App/server/graphql/queries/truck/orderListInTruck.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { orderListType } from '../../types/order';
import { post, urls } from 'helpers/api';

export default {
    type: orderListType,
    args: {
        truckId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.orderListInTruck, params, root) || {};
        return ret.success ? ret.context : {};
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/truck/truck.js

```js
import {
    GraphQLID,
} from 'graphql';
import { authorize } from '../../authorize';
import { truckType } from '../../types/truck';
import { post, urls } from 'helpers/api';

export default {
    type: truckType,
    args: {
        truckId: {
            type: GraphQLID,
        },
    },
    async resolve (root, params, options) {
        authorize(root);
        if (!params.truckId) {
            return undefined;
        }
        let ret = await post(urls.truck, params, root) || {};
        return ret.success ? ret.context : undefined;
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/truck/truckTypes.js

```js
import {
    GraphQLList,
} from 'graphql';
import { authorize } from '../../authorize';
import { truckTypeType } from '../../types/truck';
import { post, urls } from 'helpers/api';

export default {
    type: new GraphQLList(truckTypeType),
    async resolve (root, params, options) {
        authorize(root);
        let ret = await post(urls.truckType, params, root) || {};
        return ret.success ? ret.context : [];
    },
};

```

* PDShop_Client_PC/project/App/server/graphql/queries/truck/trucks.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { trucksType } from '../../types/truck';
import { post, urls } from 'helpers/api';

export default {
    type: trucksType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        type: {
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

* PDShop_Client_PC/project/App/server/graphql/queries/truck/workTruckList.js

```js
import {
    GraphQLInt,
    GraphQLString,
} from 'graphql';
import { authorize } from '../../authorize';
import { truckItemListType } from '../../types/truck';
import { post, urls } from 'helpers/api';

export default {
    type: truckItemListType,
    args: {
        keyword: {
            type: GraphQLString,
        },
        type: {
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
        let ret = await post(urls.workTruckList, params, root) || {};
        if (ret.success) {
            return {
                count: ret.context.count,
                list: ret.context.truckList,
            };
        }
        return undefined;
    },
};

```

* PDShop_Client_PC/project/App/server/shared/helpers/api/index.js

```js
export urls from './urls';
export post from './post';

```

* PDShop_Client_PC/project/App/server/shared/helpers/api/post.js

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

* PDShop_Client_PC/project/App/server/shared/helpers/api/urls.js

```js
const { apiServer } = require('../../../../../config.js');
const clientServer = apiServer + 'client/';
const shipperServer = apiServer + 'shipper/';
const agentServer = apiServer + 'agent/';

module.exports = {
    // common
    address: apiServer + 'getRegionAddress', // 获取地址列表
    addressFromLastCode: apiServer + 'getRegionAddressFromLastCode', // 通过最后一个code地址列表
    sendDoorAddressFromLastCode: apiServer + 'getRegionSendDoorAddressFromLastCode', // 通过最后一个code送货上门地址列表
    startPointAddress: apiServer + 'getStartPointAddress', // 获取始发地的地址列表
    startPointAddressFromLastCode: apiServer + 'getStartPointAddressFromLastCode', // 通过最后一个code获取始发地的地址列表

    // 个人中心
    getVerifyCode: apiServer + 'requestSendVerifyCode', // 请求发送验证码
    login: clientServer + 'login', // 登录
    register: clientServer + 'register', // 注册
    forgotPwd: clientServer + 'findPassword', // 忘记密码
    setPaymentPassword: clientServer + 'setPaymentPassword', // 设置支付密码
    modifyPassword: clientServer + 'modifyPassword', // 修改密码
    personal: clientServer + 'getPersonalInfo', // 获取个人信息
    modifyPersonalInfo: clientServer + 'modifyPersonalInfo', // 修改个人信息
    shipper: clientServer + 'getShipperInfo', // 获取物流公司信息
    modifyShipperInfo: clientServer + 'modifyShipperInfo', // 修改物流公司信息
    agent: clientServer + 'getAgentInfo', // 获取收货点信息
    modifyAgentInfo: clientServer + 'modifyAgentInfo', // 修改收货点信息
    getUserNameByPhone: clientServer + 'getUserNameByPhone', // 通过电话号码获取用户名
    feedback: clientServer + 'submitFeedback', // 意见反馈
    getReferShopList:agentServer + 'getReferShopList', // 收货点获取分店列表
    setReferShop:agentServer + 'setReferShop', // 收货点设置分店

    // 账目
    remainAmount: clientServer + 'getRemainAmount', // 获取余额
    weixinPayGetUnifiedOrder: clientServer + 'weixinPayGetUnifiedOrderForPC', // 统一下单
    weixinPayWithDraw: clientServer + 'withdraw', // 提现
    bills: clientServer + 'getBills', // 获取账单

    // 物流人员
    shipperMembers: shipperServer + 'getMemberList', // 获取人员列表
    shipperMember: shipperServer + 'getMemberDetail', // 获取人员详情
    getShipperMemberByPhone: shipperServer + 'getMemberByPhone', // 通过电话号码获取成员信息
    modifyShipperMemberAuthority: shipperServer + 'modifyMemberAuthority', // 修改人员权限

    // 同城配送物流公司获取同行上门自提行情
    getClientPickShipperList: shipperServer + 'getClientPickShipperList',

    // 收货点人员
    agentMembers: agentServer + 'getMemberList', // 获取人员列表
    agentMember: agentServer + 'getMemberDetail', // 获取人员详情
    getAgentMemberByPhone: agentServer + 'getMemberByPhone', // 通过电话号码获取成员信息
    modifyAgentMemberAuthority: agentServer + 'modifyMemberAuthority', // 修改人员权限

    // 路线
    roadmaps: shipperServer + 'getRoadmapList', // 获取路线列表
    roadmap: shipperServer + 'getRoadmapDetail', // 获取路线详情
    publishRoadmap: shipperServer + 'publishRoadmap', // 发布路线
    modifyRoadmap: shipperServer + 'modifyRoadmap', // 修改路线
    removeRoadmap: shipperServer + 'removeRoadmap', // 删除路线
    setRoadmapPrice: shipperServer + 'setRoadmapPrice', // 设置路线价格
    setRoadmapSendDoorPrice: shipperServer + 'setRoadmapSendDoorPrice', // 设置路线送货上门价格
    setRoadmapSendDoorBasePrice: shipperServer + 'setRoadmapSendDoorBasePrice', // 设置路线送货上门底价
    setRoadmapEnable: shipperServer + 'setRoadmapEnable', // 设置路线停运或恢复
    setRoadmapSendDoorEnable: shipperServer + 'setRoadmapSendDoorEnable', // 设置路线送货上门停运或恢复

    // 区域提成
    getRegionProfitList: agentServer + 'getRegionProfitList', // 收货点获取区域路线提成列表
    addRegionProfit: agentServer + 'addRegionProfit', // 收货点添加区域的路线提成
    removeRegionProfit: agentServer + 'removeRegionProfit', // 收货点删除区域的路线提成
    modifyRegionProfit: agentServer + 'modifyRegionProfit', // 收货点修改区域的路线提成
    getRegionProfitDetail: agentServer + 'getRegionProfitDetail', // 收货点获取区域的路线提成详情
    setRegionProfitWithList: agentServer + 'setRegionProfitWithList', // 批量设置区域的路线提成
    addressWithRegion: agentServer + 'getRegionAddressWithRegion', // 通过区域获取地址列表

    // 货单
    preOrders: clientServer + 'getPreOrderList', // 获取预下单列表
    preOrderGroups: clientServer + 'getPreOrderGroupList', // 获取预下单批次列表
    placePreOrder: clientServer + 'placePreOrder', // 预下单
    modifyPreOrder: clientServer + 'modifyPreOrder', // 修改预下单
    removePreOrder: clientServer + 'removePreOrder', // 删除预下单
    createPreOrderGroup: clientServer + 'createPreOrderGroup', // 创建预下单批次
    modifyPreOrderGroup: clientServer + 'modifyPreOrderGroup', // 修改预下单批次
    removePreOrderFromGroup: clientServer + 'removePreOrderFromGroup', // 移除预下单批次的货单
    inGroupPreOrders: clientServer + 'getPreOrderListInGroup', // 获取批次中的预下单列表
    lookGroupFee: clientServer + 'getRoadmapListWithPreOrderGroup', // 通过预下单批次获取收货点和分店路线列表
    lookPreOrderFee: clientServer + 'getRoadmapListWithPreOrder', // 通过预下单获取收货点和分店路线列表
    lookOrderFee: clientServer + 'getRoadmapListWithOrder', // 通过货单获取收货点和分店路线列表
    lookTruckOrderFee: clientServer + 'getRoadmapListWithTruck', // 通过货车获取分店路线列表
    sporders: shipperServer + 'getOrders', // 物流公司获取订单综合列表
    scorders: shipperServer + 'getCityDistributeOrders', // 同城配送物流公司获取订单综合列表
    order: clientServer + 'getOrderDetail', // 获取订单详情
    sendOrders: clientServer + 'getSendOrders', // 获取发货订单
    receiveOrders: clientServer + 'getReceiveOrders', // 获取收货订单
    payForOrderWhenSend: clientServer + 'payForOrderWhenSend', // 支付货单
    finishOrder: clientServer + 'finishOrder', // 完成货单
    setClientPickOrderTransportFee: shipperServer + 'setClientPickOrderTransportFee', // 设置上门自提的信息

    // 收货部货单
    addressWithOrder: agentServer + 'getRegionAddressWithOrder', // 通过货单获取地址列表
    sendDoorAddressWithOrder: agentServer + 'getRegionSendDoorAddressWithOrder', // 通过货单获取送货上门地址列表
    rcorders: agentServer + 'getOrders', // 收货点获取货单综合列表
    rcorder: agentServer + 'getLastestOrder', // 获取收货点需要的货单
    placeClientOrder: agentServer + 'placeOrder', // 下初始货单
    placeClientOrderWithPreOrder: agentServer + 'placeOrderWithPreOrder', // 通过预下单下初始货单
    modifyClientOrder: agentServer + 'modifyOrder', // 修改初始货单
    removeClientOrder: agentServer + 'removeOrder', // 删除货单
    confirmCachPayed: agentServer + 'confirmCachPayed', // 待支付
    confirmCachPayedForOrderList: agentServer + 'confirmCachPayedForOrderList', // 待支付多个货单
    printOrderBill: agentServer + 'printOrderBill', // 打印收据
    printOrderListBill: agentServer + 'printOrderListBill', // 打印多个货单
    printBarCode: agentServer + 'printBarCode', // 打印二维码
    selectPreOrders: agentServer + 'getPreOrderList', // 收货部获取用户预下单列表

    // 行情
    lookRoadmap: clientServer + 'getRoadmapListWithEndPoint', // 查询行情路线

    // 竞得货物
    needSendOrders: shipperServer + 'getNeedSendOrderSummaryList', // 获取竞得货物汇总列表
    orderListByEndPoint: shipperServer + 'getNeedSendOrderListByEndPoint', // 通过到货地获取竞得货物汇总列表

    // 货车
    trucks: shipperServer + 'getTrucks', // 获取货车综合列表
    workTruckList: shipperServer + 'getWorkTruckList', // 获取工作状态的货车列表
    truck: shipperServer + 'getTruckDetail', // 获取货车详情
    createTruck: shipperServer + 'createTruck', // 创建货车
    modifyTruck: shipperServer + 'modifyTruck', // 修改货车
    removeTruck: shipperServer + 'removeTruck', // 删除货车
    getDriverNameByPhone: shipperServer + 'getDriverInfoByPhone', // 通过电话号码获取司机姓名
    truckType: shipperServer + 'getTruckType', // 通过电话号码获取司机姓名
    carryPartments: shipperServer + 'getCarryPartmentList', // 获取搬运队列表
    selectCarryPartment: shipperServer + 'selectCarryPartment', // 选择搬运队
    payforCarryPartment: shipperServer + 'payforCarryPartment', // 支付搬运队搬运费
    orderListInTruck: shipperServer + 'getOrderListInTruck', // 获取货车里面的订单
    setCityTruck: shipperServer + 'setCityTruck', // 设置同城配送的货车
    historyTrucks: shipperServer + 'getHistoryTruckList', // 获取历史货车列表

    // 统计
    shipperStatistics: shipperServer + 'getStatistics', // 获取物流公司的统计数据

    // 客户
    getClientNameByPhone: apiServer + 'getClientNameByPhone', // 通过电话号码获取用户名

    // 其他
    statistics: apiServer + 'getStatistics', // 获取统计数据
    uploadFile: apiServer + 'uploadFile', // 上传文件
    notify: apiServer + 'notify', // 通知接口
};

```

* PDShop_Client_PC/project/App/shared/pages/shared/components/index.js

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

* PDShop_Client_PC/project/App/shared/pages/shared/utils/account.js

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

* PDShop_Client_PC/project/App/shared/pages/shared/utils/check.js

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
export function testPlateNo (plateNo) {
    return /^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领A-Z]{1}[A-Z]{1}[A-Z0-9]{4}[A-Z0-9挂学警港澳]{1}$/.test(plateNo);
}
export function checkPlateNo (rule, value, callback) {
    if (!testPlateNo(value)) {
        callback('请输入正确的车牌');
    } else {
        callback();
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/shared/utils/common.js

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

* PDShop_Client_PC/project/App/shared/pages/shared/utils/confirm.js

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
        onOk: () => onOk(password),
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

* PDShop_Client_PC/project/App/shared/pages/shared/utils/index.js

```js
export * from './common';
export * from './check';
export * from './confirm';
export * from './account';
export * from './qrcode';
export * from './socket';

```

* PDShop_Client_PC/project/App/shared/pages/shared/utils/qrcode.js

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

* PDShop_Client_PC/project/App/shared/pages/shared/utils/socket.js

```js
import io from 'socket.io-client';
import { apiServer } from '../../../../../config.js';

export function createSocketIO (userId) {
    const server = apiServer.replace('http://', '').replace('/api/', '');
    const socket = io.connect('ws://' + server + '?userId=' + userId, { path: '/api/socket' });
    socket.on('connect', obj => {
        console.log('connect to server');
    }).on('disconnect', obj => {
        console.log('disconnect to server');
    }).on('connect_error', obj => {
        console.error('connect to server error');
    }).on('connect_timeout', obj => {
        console.error('connect to server timeout');
    }).on('reconnect', obj => {
        console.log('reconnect to server');
    }).on('reconnect_failed', obj => {
        console.error('reconnect to server failed');
    });
    return socket;
}

```

* PDShop_Client_PC/project/App/shared/pages/auth/pages/forgotPwd/index.js

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

* PDShop_Client_PC/project/App/shared/pages/auth/pages/login/index.js

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

* PDShop_Client_PC/project/App/shared/pages/auth/pages/register/index.js

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

* PDShop_Client_PC/project/App/shared/pages/shared/components/form/config.js

```js
function getNotEmptyValidator (label, word) {
    return (rule, value, callback) => {
        if (typeof value === 'string' && !value.trim()) {
            callback(`请${word}${label}`);
        } else {
            callback();
        }
    };
}
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
    const defaultRules = required ? [ { required, message: `请${word}${label}` }, { validator: getNotEmptyValidator(label, word) } ] : [];
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

* PDShop_Client_PC/project/App/shared/pages/shared/components/table/config.js

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

* PDShop_Client_PC/project/App/shared/pages/shared/pages/SelectHistoryTruck/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SelectHistoryTruck from './contents';

@dataConnect(
    (state) => ({ pageSize: 20, keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: SelectHistoryTruck.fragments,
        variablesTypes: {
            historyTrucks: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
                shopId: 'ID!',
            },
        },
        initialVariables: {
            historyTrucks: {
                pageNo: 0,
                pageSize: props.pageSize,
                shopId: props.shopId,
                keyword: '',
            },
        },
    })
)
export default class SelectTruckContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize, shopId } = this.props;
        relate.refresh({
            variables: {
                historyTrucks: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    shopId,
                },
            },
            callback (data) {
                if (!data.historyTrucks) {
                    showError('没有相关货车');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, historyTrucks } = this.props;
        const property = 'truckList';
        if (needLoadPage(historyTrucks, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    historyTrucks: {
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
            <SelectHistoryTruck {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/shared/pages/SelectPreOrder/index.js

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

* PDShop_Client_PC/project/App/shared/pages/shared/pages/SelectReferShop/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SelectReferShop from './contents';

@dataConnect(
    (state) => ({ pageSize: 20, keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: SelectReferShop.fragments,
        variablesTypes: {
            referShops: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            referShops: {
                pageNo: 0,
                pageSize: props.pageSize,
                keyword: '',
            },
        },
    })
)
export default class SelectReferShopContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                referShops: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.referShops) {
                    showError('没有相关分店');
                }
            },
        });
    }

    loadListPage (keyword, pageNo) {
        const { relate, pageSize, referShopList } = this.props;
        const property = 'referShops';
        if (needLoadPage(referShopList, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    referShopList: {
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
            <SelectReferShop {...this.props}
                refresh={::this.refresh}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/shared/pages/SelectTruck/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SelectTruck from './contents';

@dataConnect(
    (state) => ({ pageSize: 20, keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: SelectTruck.fragments,
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
export default class SelectTruckContainer extends React.Component {
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
                    showError('没有相关路线');
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
            <SelectTruck {...this.props}
                refresh={::this.refresh}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/shared/pages/ShowClientPicker/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import ShowClientPicker from './contents';

@dataConnect(
    (state) => ({ keepLastKeepData: true }),
    null,
    (props) => ({
        fragments: ShowClientPicker.fragments,
        variablesTypes: {
            clientPickList: {
                shopId: 'ID!',
            },
        },
        initialVariables: {
            clientPickList: {
                shopId: props.shopId,
            },
        },
    })
)
export default class ShowClientPickerContainer extends React.Component {
    refresh () {
        const { relate, shopId } = this.props;
        relate.refresh({
            variables: {
                clientPickList: {
                    shopId,
                },
            },
            callback (data) {
                if (!data.clientPickList) {
                    showError(data.msg);
                }
            },
        });
    }
    loadListPage () {
        const { relate, clientPickList, shopId } = this.props;
        const property = 'clientPickList';
        if (needLoadPage(clientPickList)) {
            relate.loadPage({
                variables: {
                    clientPickList: {
                        shopId,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <ShowClientPicker {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/agentMembers/index.js

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
            agentMembers: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            agentMembers: {
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
                agentMembers: {
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
                    agentMembers: {
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

* PDShop_Client_PC/project/App/shared/pages/client/pages/bills/index.js

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

* PDShop_Client_PC/project/App/shared/pages/client/pages/feedback/index.js

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

* PDShop_Client_PC/project/App/shared/pages/client/pages/home/index.js

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
    refreshHome () {
        const { relate } = this.props;
        relate.refresh({
            property: 'personal',
        });
    }
    render () {
        return (
            <Home {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/needSendOrders/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import NeedSendOrders from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    null,
    (props) => ({
        fragments: NeedSendOrders.fragments,
        variablesTypes: {
            needSendOrders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String',
                shopId: 'ID!',
            },
        },
        initialVariables: {
            needSendOrders: {
                pageNo: 0,
                pageSize: props.pageSize,
                shopId: props.rootShop.id,
            },
        },
    })
)
export default class NeedSendOrdersContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize, rootShop } = this.props;
        relate.refresh({
            variables: {
                needSendOrders: {
                    pageNo: 0,
                    pageSize,
                    shopId: rootShop.id,
                    keyword,
                },
            },
            callback (data) {
                if (!data.needSendOrders) {
                    showError('没有相关货单信息');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, needSendOrders, rootShop } = this.props;
        const property = 'orderList';
        if (needLoadPage(needSendOrders, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    needSendOrders: {
                        pageNo,
                        pageSize,
                        keyword,
                        shopId: rootShop.id,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <NeedSendOrders {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/notify/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import { needLoadPage } from 'utils';
import * as personalActions from 'actions/personals';
import Notify from './contents';

@dataConnect(
    (state) => {
        const { activeType } = state.router.location.state || { activeType: 'news' };
        return {
            activeType,
            pageSize: 20,
        };
    },
    (dispatch) => ({
        actions : bindActionCreators(personalActions, dispatch),
    }),
    (props) => ({
        fragments: Notify.fragments,
        variablesTypes: {
            notify: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                type: 'String!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            notify: {
                pageNo: 0,
                pageSize: props.pageSize,
                type: '',
                keyword: '',
            },
        },
    })
)
export default class NotifyContainer extends React.Component {
    getNotify (keyword = '') {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                notify: {
                    pageNo: 0,
                    pageSize,
                    type: '',
                    keyword,
                },
            },
        });
    }
    loadListPage (keyword = '', type, pageNo) {
        const { relate, pageSize, notify } = this.props;
        if (needLoadPage(notify[type], pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    notify: {
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
                getNotify={::this.getNotify}
                loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/orderDetail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import OrderDetail from './contents';

@dataConnect(
    (state) => {
        const { orderId } = state.router.location.state || state.router.location.query;
        return { orderId, keepLastKeepData: true };
    },
    null,
    (props) => {
        return {
            fragments: OrderDetail.fragments,
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
export default class OrderDetailContainer extends React.Component {
    render () {
        return (
            <OrderDetail {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/personal/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as personalActions from 'actions/personals';
import * as accountActions from 'actions/accounts';
import * as trucksActions from 'actions/trucks';
import Personal from './contents';

@dataConnect(
    (state) => ({}),
    (dispatch) => ({
        actions : bindActionCreators({ ...personalActions, ...accountActions, ...trucksActions }, dispatch),
    }),
    (props) => ({
        fragments: Personal.fragments,
    })
)
export default class PersonalContainer extends React.Component {
    refreshRemainAmount () {
        const { relate } = this.props;
        relate.refresh({
            property: 'remainAmount',
        });
    }
    render () {
        return (
            <Personal {...this.props} refreshRemainAmount={::this.refreshRemainAmount} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/placeOrder/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/agentOrders';
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
            property: ['rcorder', 'addressWithOrder', 'sendDoorAddressWithOrder'],
        });
    }
    render () {
        return (
            <PlaceOrder {...this.props} refresh={::this.refresh} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/preOrders/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import { needLoadPage, showError } from 'utils';
import PreOrders from './contents';

@dataConnect(
    (state) => ({ pageSize: 20, keepData: true }),
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: PreOrders.fragments,
        variablesTypes: {
            preOrders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
            },
        },
        initialVariables: {
            preOrders: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class PreOrdersContainer extends React.Component {
    refresh () {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                preOrders: {
                    pageNo: 0,
                    pageSize,
                },
            },
            property: 'preOrderGroups',
            callback (data) {
                if (!data.preOrders) {
                    showError('没有相关货车信息');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, preOrders } = this.props;
        const property = 'preOrderList';
        if (needLoadPage(preOrders, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    preOrders: {
                        pageNo,
                        pageSize,
                    },
                },
                property,
            });
        }
    }
    render () {
        return <PreOrders {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />;
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/quotations/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import Quotations from './contents';

@dataConnect(
    (state) => ({
        pageSize: 20,
        location: [106.6886, 26.566621],
        orderBy: 0,
    }),
    null,
    (props) => ({
        fragments: Quotations.fragments,
        variablesTypes: {
            lookRoadmap: {
                startPointLastCode: 'Int',
                shopId: 'ID',
                agentId: 'ID',
                endPointLastCode: 'Int',
                sendDoorEndPointLastCode: 'Int',
                isSendDoor: 'Boolean',
                location: '[Float]!',
                orderBy: 'Int!',
                pageNo: 'Int!',
                pageSize: 'Int!',
            },
            startPointAddressFromLastCode: {
                addressLastCode: 'Int',
                isLeaf: 'Boolean',
            },
            addressFromLastCode: {
                addressLastCode: 'Int',
            },
        },
        initialVariables: {
            lookRoadmap: {
                location: props.location,
                orderBy: props.orderBy,
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class QuotationsContainer extends React.Component {
    refresh (startPointLastCode, shopId, agentId, endPointLastCode, sendDoorEndPointLastCode, isSendDoor, location, orderBy) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                lookRoadmap: {
                    startPointLastCode,
                    shopId,
                    agentId,
                    endPointLastCode,
                    sendDoorEndPointLastCode,
                    isSendDoor,
                    location,
                    orderBy,
                    pageNo: 0,
                    pageSize,
                },
            },
            callback (data) {
                if (!data.lookRoadmap) {
                    showError('没有相关路线');
                }
            },
        });
    }
    loadListPage (startPointLastCode, shopId, endPointLastCode, sendDoorEndPointLastCode, isSendDoor, location, orderBy, type, pageNo) {
        const { relate, pageSize, lookRoadmap } = this.props;
        if (needLoadPage(lookRoadmap[type], 'roadmapList', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    lookRoadmap: {
                        startPointLastCode,
                        shopId,
                        endPointLastCode,
                        sendDoorEndPointLastCode,
                        isSendDoor,
                        location,
                        orderBy,
                        pageNo,
                        pageSize,
                    },
                },
                property: type + '.roadmapList',
            });
        }
    }
    render () {
        return (
            <Quotations {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/rcorders/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/agentOrders';
import { needLoadPage, showError } from 'utils';
import RCOrders from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: RCOrders.fragments,
        variablesTypes: {
            rcorders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                type: 'String',
                keyword: 'String',
                startDate: 'String',
                endDate: 'String',
            },
        },
        initialVariables: {
            rcorders: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class RCOrdersContainer extends React.Component {
    refresh (keyword, startDate, endDate) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                rcorders: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    startDate,
                    endDate,
                },
            },
            callback (data) {
                if (!data.rcorders) {
                    showError('没有相关货单');
                }
            },
        });
    }
    loadListPage (keyword, type, pageNo, startDate, endDate) {
        const { relate, pageSize, rcorders } = this.props;
        if (needLoadPage(rcorders[type], 'list', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    rcorders: {
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
            <RCOrders {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/receiveOrders/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import { needLoadPage, showError } from 'utils';
import Orders from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: Orders.fragments,
        variablesTypes: {
            receiveOrders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                type: 'String',
                keyword: 'String',
                startDate: 'String',
                endDate: 'String',
            },
        },
        initialVariables: {
            receiveOrders: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
        },
    })
)
export default class OrdersContainer extends React.Component {
    refresh (keyword, startDate, endDate) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                receiveOrders: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    startDate,
                    endDate,
                },
            },
            callback (data) {
                if (!data.receiveOrders) {
                    showError('没有相关货单');
                }
            },
        });
    }
    loadListPage (keyword, type, pageNo, startDate, endDate) {
        const { relate, pageSize, receiveOrders } = this.props;
        if (needLoadPage(receiveOrders[type], 'list', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    receiveOrders: {
                        pageNo,
                        pageSize,
                        type,
                        keyword,
                        startDate,
                        endDate,
                    },
                },
                property: type + 'list',
            });
        }
    }
    render () {
        return (
            <Orders {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/regionRates/index.js

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
    refresh (keyword) {
        const { relate, pageSize } = this.props;
        relate.refresh({
            variables: {
                regionRates: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                },
            },
            callback (data) {
                if (!data.regionRates) {
                    showError('没有相关方向的设置');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, regionRates } = this.props;
        const property = 'list';
        if (needLoadPage(regionRates, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    regionRates: {
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
            <RegionRates {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/roadmaps/index.js

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
                shopId: 'ID!',
            },
        },
        initialVariables: {
            roadmaps: {
                pageNo: 0,
                pageSize: props.pageSize,
                shopId: props.rootShop.id,
            },
        },
    })
)
export default class RoadmapsContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize, rootShop } = this.props;
        relate.refresh({
            variables: {
                roadmaps: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    shopId: rootShop.id,
                },
            },
            callback (data) {
                if (!data.roadmaps) {
                    showError('没有相关路线');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, roadmaps, rootShop } = this.props;
        const property = 'roadmapList';
        if (needLoadPage(roadmaps, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    roadmaps: {
                        pageNo,
                        pageSize,
                        keyword,
                        shopId: rootShop.id,
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

* PDShop_Client_PC/project/App/shared/pages/client/pages/scorders/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import { needLoadPage, showError } from 'utils';
import * as orderActions from 'actions/orders';
import * as personalActions from 'actions/personals';
import SPOrders from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators({ ...orderActions, ...personalActions }, dispatch),
    }),
    (props) => ({
        fragments: SPOrders.fragments,
        variablesTypes: {
            scorders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                shopId: 'ID!',
                type: 'String',
                keyword: 'String',
                startDate: 'String',
                endDate: 'String',
            },
        },
        initialVariables: {
            scorders: {
                pageNo: 0,
                pageSize: props.pageSize,
                shopId: props.rootShop.id,
            },
        },
    })
)
export default class SPOrdersContainer extends React.Component {
    refresh (keyword, startDate, endDate) {
        const { relate, pageSize, rootShop } = this.props;
        relate.refresh({
            variables: {
                scorders: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    shopId: rootShop.id,
                    startDate,
                    endDate,
                },
            },
            callback (data) {
                if (!data.scorders) {
                    showError('没有相关货单');
                }
            },
        });
    }
    loadListPage (keyword, type, pageNo, startDate, endDate) {
        const { relate, pageSize, scorders, rootShop } = this.props;
        if (needLoadPage(scorders[type], 'list', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    scorders: {
                        pageNo,
                        pageSize,
                        type,
                        keyword,
                        shopId: rootShop.id,
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
            <SPOrders {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/sendDoorRoadmaps/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as roadmapActions from 'actions/roadmaps';
import { needLoadPage, showError } from 'utils';
import SendDoorRoadmaps from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(roadmapActions, dispatch),
    }),
    (props) => ({
        fragments: SendDoorRoadmaps.fragments,
        variablesTypes: {
            roadmaps: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String',
                shopId: 'ID!',
                isCityDistribute: 'Boolean',
            },
        },
        initialVariables: {
            roadmaps: {
                pageNo: 0,
                pageSize: props.pageSize,
                shopId: props.rootShop.id,
                isCityDistribute: true,
            },
        },
    })
)
export default class SendDoorRoadmapsContainer extends React.Component {
    refresh (keyword) {
        const { relate, pageSize, rootShop } = this.props;
        relate.refresh({
            variables: {
                roadmaps: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    shopId: rootShop.id,
                    isCityDistribute: true,
                },
            },
            callback (data) {
                if (!data.roadmaps) {
                    showError('没有相关路线');
                }
            },
        });
    }
    loadListPage (keyword, pageNo) {
        const { relate, pageSize, roadmaps, rootShop } = this.props;
        const property = 'roadmapList';
        if (needLoadPage(roadmaps, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    roadmaps: {
                        pageNo,
                        pageSize,
                        keyword,
                        shopId: rootShop.id,
                        isCityDistribute: true,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <SendDoorRoadmaps {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/sendOrders/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import { needLoadPage, showError } from 'utils';
import Orders from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: Orders.fragments,
        variablesTypes: {
            sendOrders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                senderId: 'ID!',
                type: 'String',
                keyword: 'String',
                startDate: 'String',
                endDate: 'String',
            },
        },
        initialVariables: {
            sendOrders: {
                pageNo: 0,
                pageSize: props.pageSize,
                senderId: props.rootPersonal.userId,
            },
        },
    })
)
export default class OrdersContainer extends React.Component {
    refresh (keyword, startDate, endDate) {
        const { relate, pageSize, rootPersonal } = this.props;
        relate.refresh({
            variables: {
                sendOrders: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    senderId: rootPersonal.userId,
                    startDate,
                    endDate,
                },
            },
            callback (data) {
                if (!data.sendOrders) {
                    showError('没有相关货单');
                }
            },
        });
    }
    loadListPage (keyword, type, pageNo, startDate, endDate) {
        const { relate, pageSize, sendOrders, rootPersonal } = this.props;
        if (needLoadPage(sendOrders[type], 'list', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    sendOrders: {
                        pageNo,
                        pageSize,
                        type,
                        keyword,
                        senderId: rootPersonal.userId,
                        startDate,
                        endDate,
                    },
                },
                property: type + 'list',
            });
        }
    }
    render () {
        return (
            <Orders {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/shipperMembers/index.js

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
            shipperMembers: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String!',
            },
        },
        initialVariables: {
            shipperMembers: {
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
                shipperMembers: {
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
                    shipperMembers: {
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

* PDShop_Client_PC/project/App/shared/pages/client/pages/sporders/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage, showError } from 'utils';
import SPOrders from './contents';

@dataConnect(
    (state) => ({ pageSize: 20 }),
    null,
    (props) => ({
        fragments: SPOrders.fragments,
        variablesTypes: {
            sporders: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                shopId: 'ID!',
                type: 'String',
                keyword: 'String',
                startDate: 'String',
                endDate: 'String',
            },
        },
        initialVariables: {
            sporders: {
                pageNo: 0,
                pageSize: props.pageSize,
                shopId: props.rootShop.id,
            },
        },
    })
)
export default class SPOrdersContainer extends React.Component {
    refresh (keyword, startDate, endDate) {
        const { relate, pageSize, rootShop } = this.props;
        relate.refresh({
            variables: {
                sporders: {
                    pageNo: 0,
                    pageSize,
                    keyword,
                    shopId: rootShop.id,
                    startDate,
                    endDate,
                },
            },
            callback (data) {
                if (!data.sporders) {
                    showError('没有相关货单');
                }
            },
        });
    }
    loadListPage (keyword, type, pageNo, startDate, endDate) {
        const { relate, pageSize, sporders, rootShop } = this.props;
        if (needLoadPage(sporders[type], 'list', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    sporders: {
                        pageNo,
                        pageSize,
                        type,
                        keyword,
                        shopId: rootShop.id,
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
            <SPOrders {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/statistics/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import Statistics from './contents';

@dataConnect(
    null,
    null,
    (props) => ({
        fragments: Statistics.fragments,
    })
)
export default class StatisticsContainer extends React.Component {
    refresh (shopId) {
        const { relate } = this.props;
        relate.refresh({
            property: 'shipperStatistics',
        });
    }
    render () {
        return (
            <Statistics {...this.props} refresh={::this.refresh} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/trucks/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as truckActions from 'actions/trucks';
import { needLoadPage, showError } from 'utils';
import Trucks from './contents';

@dataConnect(
    (state) => ({ pageSize: 20, keepData: true }),
    (dispatch) => ({
        actions : bindActionCreators(truckActions, dispatch),
    }),
    (props) => ({
        fragments: Trucks.fragments,
        variablesTypes: {
            trucks: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                type: 'String',
                keyword: 'String',
            },
            workTruckList: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                keyword: 'String',
            },
        },
        initialVariables: {
            trucks: {
                pageNo: 0,
                pageSize: props.pageSize,
            },
            workTruckList: {
                pageNo: 0,
                pageSize: props.pageSize,
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
                workTruckList: {
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
    loadListPage (keyword, type, pageNo) {
        const { relate, pageSize, trucks, workTruckList } = this.props;
        if (type === 'tosend') {
            if (needLoadPage(workTruckList, 'list', pageNo, pageSize)) {
                relate.loadPage({
                    variables: {
                        workTruckList: {
                            pageNo,
                            pageSize,
                            keyword,
                        },
                    },
                    property: 'list',
                });
            }
        } else {
            if (needLoadPage(trucks[type], 'list', pageNo, pageSize)) {
                relate.loadPage({
                    variables: {
                        trucks: {
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
    }
    render () {
        return (
            <Trucks {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/agentMembers/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as memberActions from 'actions/members';
import MemberDetail from './contents';

@dataConnect(
    (state) => {
        const { memberId, record, agentMembers } = state.router.location.state || state.router.location.query;
        return { memberId, record, agentMembers, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(memberActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: !!props.member || !props.memberId,
            fragments: MemberDetail.fragments,
            variablesTypes: {
                agentMember: {
                    memberId: 'ID!',
                },
            },
            initialVariables: {
                agentMember: {
                    memberId: props.memberId,
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

* PDShop_Client_PC/project/App/shared/pages/client/pages/agentMembers/pages/register/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as memberActions from 'actions/members';
import MemberRegister from './contents';

@dataConnect(
    (state) => {
        const { agentMembers } = state.router.location.state || state.router.location.query;
        return { agentMembers, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions: bindActionCreators(memberActions, dispatch),
    }),
)
export default class MemberRegisterContainer extends React.Component {
    render () {
        return (
            <MemberRegister {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/needSendOrders/pages/orderListByEndpoint/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { needLoadPage } from 'utils';
import OrderListByEndPoint from './contents';

@dataConnect(
    (state) => {
        const { endPoint } = state.router.location.state || state.router.location.query;
        return { endPoint, pageSize: 20, keepLastKeepData: true };
    },
    null,
    (props) => ({
        fragments: OrderListByEndPoint.fragments,
        variablesTypes: {
            orderListByEndPoint: {
                pageNo: 'Int!',
                pageSize: 'Int!',
                endPoint: 'String!',
                shopId: 'ID!',
            },
        },
        initialVariables: {
            orderListByEndPoint: {
                pageNo: 0,
                pageSize: props.pageSize,
                shopId: props.rootShop.id,
                endPoint: props.endPoint,
            },
        },
    })
)
export default class OrderListByEndPointContainer extends React.Component {
    loadListPage (unused, pageNo) {
        const { relate, rootShop, endPoint, pageSize, orderListByEndPoint } = this.props;
        const property = 'orderList';
        if (needLoadPage(orderListByEndPoint, property, pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    orderListByEndPoint: {
                        pageNo,
                        pageSize,
                        endPoint,
                        shopId: rootShop.id,
                    },
                },
                property,
            });
        }
    }
    render () {
        return (
            <OrderListByEndPoint {...this.props} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/personal/pages/agent/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as personalActions from 'actions/personals';
import * as accountActions from 'actions/accounts';
import Agent from './contents';

@dataConnect(
    (state) => ({}),
    (dispatch) => ({
        actions : bindActionCreators({ ...personalActions, ...accountActions }, dispatch),
    }),
    (props) => ({
        fragments: Agent.fragments,
    }),
)
export default class AgentContainer extends React.Component {
    refreshRemainAmount () {
        const { relate } = this.props;
        relate.refresh({
            property: 'remainAmount',
        });
    }
    render () {
        return (
            <Agent {...this.props} refreshRemainAmount={::this.refreshRemainAmount} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/personal/pages/paymentPwd/index.js

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

* PDShop_Client_PC/project/App/shared/pages/client/pages/personal/pages/shipper/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as personalActions from 'actions/personals';
import * as accountActions from 'actions/accounts';
import Shipper from './contents';

@dataConnect(
    (state) => ({}),
    (dispatch) => ({
        actions : bindActionCreators({ ...personalActions, ...accountActions }, dispatch),
    }),
    (props) => ({
        fragments: Shipper.fragments,
        variablesTypes: {
            shipper: {
                shopId: 'ID',
            },
        },
        initialVariables: {
            shipper: {
                shopId: props.rootShop.id,
            },
        },
    }),
)
export default class ShipperContainer extends React.Component {
    refreshRemainAmount () {
        const { relate } = this.props;
        relate.refresh({
            property: 'remainAmount',
        });
    }
    render () {
        return (
            <Shipper {...this.props} refreshRemainAmount={::this.refreshRemainAmount} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/preOrders/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import { getUserNameByPhone } from 'actions/personals';
import PreOrderDetail from './contents';

@dataConnect(
    (state) => {
        const { orderGroupId, orderGroupStartPoint, operType, orderId, record, preOrders, activeType } = state.router.location.state || state.router.location.query;
        return { orderGroupId, orderGroupStartPoint, operType: operType * 1, orderId, record: record || { startPoint: '贵州省贵阳市' }, preOrders, activeType, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators({ ...orderActions, getUserNameByPhone }, dispatch),
    }),
    (props) => {
        return {
            fragments: PreOrderDetail.fragments,
            variablesTypes: {
                order: {
                    orderId: 'ID!',
                },
                startPointAddressFromLastCode: {
                    addressLastCode: 'Int',
                    isLeaf: 'Boolean',
                },
                addressFromLastCode: {
                    addressLastCode: 'Int!',
                    type: 'Int',
                },
                sendDoorAddressFromLastCode: {
                    addressLastCode: 'Int',
                },
            },
            initialVariables: {
                order: {
                    orderId: props.orderId || '',
                },
                startPointAddressFromLastCode: {
                    addressLastCode: props.record ? (props.record.startPointLastCode || 0) : 0,
                    isLeaf: props.record ? !!(props.record.shop || props.record.agent) : false,
                },
                addressFromLastCode: {
                    addressLastCode: props.record ? (props.record.endPointLastCode || 0) : 0,
                    type: props.record && props.record.isCityDistribute ? 1 : 0,
                },
                sendDoorAddressFromLastCode: {
                    addressLastCode: props.record ? (props.record.sendDoorEndPointLastCode || 0) : 0,
                },
            },
            mutations: {
                modifyPreOrder ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removePreOrder ({ state, data, _ }) {
                    if (data.success) {
                        props.preOrders.count--;
                        _.remove(props.preOrders.orderList, (item) => item.id === props.orderId);
                    }
                },
            },
        };
    }
)
export default class PreOrderDetailContainer extends React.Component {
    render () {
        return (
            <PreOrderDetail {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/preOrders/pages/groupDetail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import InGroupPreOrderList from './contents';

@dataConnect(
    (state) => {
        const { orderGroupId, orderGroupStartPoint, record } = state.router.location.state || state.router.location.query;
        return { orderGroupId, orderGroupStartPoint, record, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: InGroupPreOrderList.fragments,
        variablesTypes: {
            inGroupPreOrders: {
                orderGroupId: 'ID!',
            },
        },
        initialVariables: {
            inGroupPreOrders: {
                orderGroupId: props.orderGroupId,
            },
        },
    })
)
export default class InGroupPreOrderListContainer extends React.Component {
    render () {
        return (
            <InGroupPreOrderList {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/preOrders/pages/lookGroupFee/index.js

```js
    import React from 'react';
    import { dataConnect } from 'relatejs';
    import { bindActionCreators } from 'redux';
    import * as orderActions from 'actions/orders';
    import LookGroupFee from './contents';

@dataConnect(
    (state) => {
        const { orderGroupId, record } = state.router.location.state || state.router.location.query;
        return { orderGroupId, record, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: LookGroupFee.fragments,
        variablesTypes: {
            lookGroupFee: {
                orderGroupId: 'ID!',
                location: '[Float]!',
            },
        },
        initialVariables: {
            lookGroupFee: {
                orderGroupId: props.orderGroupId,
                location: [106.6886, 26.566621],
            },
        },
    })
)
    export default class LookGroupFeeContainer extends React.Component {
        render () {
            return (
                <LookGroupFee {...this.props} />
            );
        }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/preOrders/pages/lookOrderFee/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import { needLoadPage } from 'utils';
import LookOrderFee from './contents';

@dataConnect(
    (state) => {
        const { orderId, record, startPointLastCode, shopId, agentId } = state.router.location.state || state.router.location.query;
        return { orderId, record, startPointLastCode, shopId, agentId, orderBy: 0, pageSize: 20, location: [106.6886, 26.566621], keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: LookOrderFee.fragments,
        variablesTypes: {
            lookOrderFee: {
                orderId: 'ID!',
                location: '[Float]!',
                pageNo: 'Int!',
                pageSize: 'Int!',
                orderBy: 'Int!',
                startPointLastCode: 'Int',
                shopId: 'ID',
                agentId: 'ID',
            },
        },
        initialVariables: {
            lookOrderFee: {
                orderId: props.orderId,
                location: props.location,
                pageNo: 0,
                pageSize: props.pageSize,
                orderBy: props.orderBy,
                startPointLastCode: props.startPointLastCode,
                shopId: props.shopId,
                agentId: props.agentId,
            },
        },
    })
)
export default class LookOrderFeeContainer extends React.Component {
    refresh (orderBy, location) {
        const { relate, pageSize, orderId } = this.props;
        relate.refresh({
            variables: {
                lookOrderFee: {
                    orderId,
                    location,
                    pageNo: 0,
                    pageSize,
                    orderBy,
                },
            },
        });
    }
    loadListPage (orderBy, location, type, pageNo) {
        const { relate, lookOrderFee, pageSize, orderId } = this.props;
        if (needLoadPage(lookOrderFee[type], 'roadmapList', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    lookOrderFee: {
                        orderId,
                        location,
                        pageNo,
                        pageSize,
                        orderBy,
                    },
                },
                property: type + '.roadmapList',
            });
        }
    }
    render () {
        return (
            <LookOrderFee {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/preOrders/pages/lookPreOrderFee/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import { needLoadPage } from 'utils';
import LookPreOrderFee from './contents';

@dataConnect(
    (state) => {
        const { orderId, record } = state.router.location.state || state.router.location.query;
        return { orderId, record, orderBy: 0, pageSize: 20, location: [106.6886, 26.566621], keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: LookPreOrderFee.fragments,
        variablesTypes: {
            lookPreOrderFee: {
                orderId: 'ID!',
                location: '[Float]!',
                pageNo: 'Int!',
                pageSize: 'Int!',
                orderBy: 'Int!',
            },
        },
        initialVariables: {
            lookPreOrderFee: {
                orderId: props.orderId,
                location: props.location,
                pageNo: 0,
                pageSize: props.pageSize,
                orderBy: props.orderBy,
            },
        },
    })
)
export default class LookPreOrderFeeContainer extends React.Component {
    refresh (orderBy, location) {
        const { relate, pageSize, orderId } = this.props;
        relate.refresh({
            variables: {
                lookPreOrderFee: {
                    orderId,
                    location,
                    pageNo: 0,
                    pageSize,
                    orderBy,
                },
            },
        });
    }
    loadListPage (orderBy, location, type, pageNo) {
        const { relate, lookPreOrderFee, pageSize, orderId } = this.props;
        if (needLoadPage(lookPreOrderFee[type], 'roadmapList', pageNo, pageSize)) {
            relate.loadPage({
                variables: {
                    lookPreOrderFee: {
                        orderId,
                        location,
                        pageNo,
                        pageSize,
                        orderBy,
                    },
                },
                property: type + '.roadmapList',
            });
        }
    }
    render () {
        return (
            <LookPreOrderFee {...this.props} refresh={::this.refresh} loadListPage={::this.loadListPage} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/rcorders/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/agentOrders';
import RCOrderDetail from './contents';

@dataConnect(
    (state) => {
        const { orderId, record, rcorders, activeType } = state.router.location.state || state.router.location.query;
        return { orderId, record, rcorders, activeType, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: !!props.order || !props.orderId,
            fragments: RCOrderDetail.fragments,
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
export default class RCOrderDetailContainer extends React.Component {
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
            <RCOrderDetail {...this.props} refresh={::this.refresh} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/receiveOrders/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import OrderDetail from './contents';

@dataConnect(
    (state) => {
        const { orderId } = state.router.location.state || state.router.location.query;
        return { orderId, keepLastKeepData: true };
    },
    null,
    (props) => {
        return {
            fragments: OrderDetail.fragments,
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
export default class OrderDetailContainer extends React.Component {
    render () {
        return (
            <OrderDetail {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/roadmaps/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as roadmapActions from 'actions/roadmaps';
import RoadmapDetail from './contents';

@dataConnect(
    (state) => {
        const { operType, endPointLastCode, roadmapId, record, fromChild, roadmaps } = state.router.location.state || state.router.location.query;
        return { operType: operType * 1, roadmapId, endPointLastCode, record, fromChild, roadmaps, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(roadmapActions, dispatch),
    }),
    (props) => {
        return {
            fragments: RoadmapDetail.fragments,
            variablesTypes: {
                roadmap: {
                    roadmapId: 'ID',
                },
                addressFromLastCode: {
                    addressLastCode: 'Int',
                },
            },
            initialVariables: {
                roadmap: {
                    roadmapId: props.roadmapId,
                },
                addressFromLastCode: {
                    addressLastCode: props.endPointLastCode,
                },
            },
            mutations: {
                modifyRoadmap ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removeRoadmap ({ state, data, _ }) {
                    if (data.success) {
                        props.roadmaps.count--;
                        _.remove(props.roadmaps.roadmapList, (item) => item.id === props.roadmapId);
                    }
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

* PDShop_Client_PC/project/App/shared/pages/client/pages/regionRates/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as roadmapActions from 'actions/roadmaps';
import RegionRateDetail from './contents';

@dataConnect(
    (state) => {
        const { operType, regionLastCode, regionType, regionId, record, regionRates } = state.router.location.state || state.router.location.query;
        return { operType: operType * 1, regionLastCode, regionType, regionId, record, regionRates, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(roadmapActions, dispatch),
    }),
    (props) => {
        return {
            fragments: RegionRateDetail.fragments,
            variablesTypes: {
                regionRate: {
                    regionId: 'ID',
                },
                addressWithRegion: {
                    regionId: 'ID',
                },
            },
            initialVariables: {
                regionRate: {
                    regionId: props.regionId,
                },
                addressWithRegion: {
                    regionId: props.regionId,
                },
            },
            mutations: {
                modifyRegionProfit ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removeRegionProfit ({ state, data, _ }) {
                    if (data.success) {
                        props.regionRates.count--;
                        _.remove(props.regionRates.regionRateList, (item) => item.id === props.regionId);
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

* PDShop_Client_PC/project/App/shared/pages/client/pages/roadmaps/pages/sendDoorList/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as roadmapActions from 'actions/roadmaps';
import SendDoorLists from './contents';

@dataConnect(
    (state) => {
        const { record, endPointLastCode, roadmaps } = state.router.location.state || state.router.location.query;
        return { record, endPointLastCode, roadmaps, pageSize: 20, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(roadmapActions, dispatch),
    }),
)
export default class SendDoorListsContainer extends React.Component {
    render () {
        return (
            <SendDoorLists {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/sendDoorRoadmaps/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as roadmapActions from 'actions/roadmaps';
import SendDoorRoadmapDetail from './contents';

@dataConnect(
    (state) => {
        const { operType, endPointLastCode, roadmapId, record, roadmaps } = state.router.location.state || state.router.location.query;
        return { operType: operType * 1, roadmapId, endPointLastCode, record, roadmaps, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(roadmapActions, dispatch),
    }),
    (props) => {
        return {
            fragments: SendDoorRoadmapDetail.fragments,
            variablesTypes: {
                roadmap: {
                    roadmapId: 'ID',
                },
                sendDoorAddressFromLastCode: {
                    addressLastCode: 'Int',
                    parentCode: 'Int',
                },
            },
            initialVariables: {
                roadmap: {
                    roadmapId: props.roadmapId,
                },
                sendDoorAddressFromLastCode: {
                    addressLastCode: props.endPointLastCode,
                    parentCode: props.rootShop.addressRegionLastCode,
                },
            },
            mutations: {
                modifyRoadmap ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removeRoadmap ({ state, data, _ }) {
                    if (data.success) {
                        props.roadmaps.count--;
                        _.remove(props.roadmaps.roadmapList, (item) => item.id === props.roadmapId);
                    }
                },
            },
        };
    }
)
export default class SendDoorRoadmapDetailContainer extends React.Component {
    render () {
        return (
            <SendDoorRoadmapDetail {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/shipperMembers/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as memberActions from 'actions/members';
import MemberDetail from './contents';

@dataConnect(
    (state) => {
        const { memberId, record, members } = state.router.location.state || state.router.location.query;
        return { memberId, record, members, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(memberActions, dispatch),
    }),
    (props) => {
        return {
            manualLoad: !!props.member || !props.memberId,
            fragments: MemberDetail.fragments,
            variablesTypes: {
                shipperMember: {
                    memberId: 'ID!',
                },
            },
            initialVariables: {
                shipperMember: {
                    memberId: props.memberId,
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

* PDShop_Client_PC/project/App/shared/pages/client/pages/sendOrders/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import OrderDetail from './contents';

@dataConnect(
    (state) => {
        const { orderId } = state.router.location.state || state.router.location.query;
        return { orderId, keepLastKeepData: true };
    },
    null,
    (props) => {
        return {
            fragments: OrderDetail.fragments,
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
export default class OrderDetailContainer extends React.Component {
    render () {
        return (
            <OrderDetail {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/shipperMembers/pages/register/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as memberActions from 'actions/members';
import MemberRegister from './contents';

@dataConnect(
    (state) => {
        const { shipperMembers } = state.router.location.state || state.router.location.query;
        return { shipperMembers, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(memberActions, dispatch),
    }),
)
export default class MemberRegisterContainer extends React.Component {
    render () {
        return (
            <MemberRegister {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/trucks/pages/detail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as truckActions from 'actions/trucks';
import TruckDetail from './contents';

@dataConnect(
    (state) => {
        const { operType, truckId, record, trucks, workTruckList } = state.router.location.state || state.router.location.query;
        return { operType: operType * 1, truckId, record, trucks, workTruckList, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(truckActions, dispatch),
    }),
    (props) => {
        return {
            fragments: TruckDetail.fragments,
            variablesTypes: {
                truck: {
                    truckId: 'ID',
                },
            },
            initialVariables: {
                truck: {
                    truckId: props.truckId,
                },
            },
            mutations: {
                modifyTruck ({ state, data }) {
                    if (data.success) {
                        Object.assign(props.record, data.context);
                    }
                },
                removeTruck ({ state, data, _ }) {
                    if (data.success) {
                        props.workTruckList.count--;
                        _.remove(props.workTruckList.list, (item) => item.id === props.truckId);
                    }
                },
            },
        };
    }
)
export default class TruckDetailContainer extends React.Component {
    render () {
        return (
            <TruckDetail {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/trucks/pages/lookTruckFee/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import LookTruckOrderFee from './contents';

@dataConnect(
    (state) => {
        const { truckId, startPointLastCode, shopId } = state.router.location.state || state.router.location.query;
        return { truckId, startPointLastCode, shopId, pageSize: 20, location: [106.6886, 26.566621], keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: LookTruckOrderFee.fragments,
        variablesTypes: {
            lookTruckOrderFee: {
                truckId: 'ID!',
                location: '[Float]!',
                startPointLastCode: 'Int',
                shopId: 'ID',
            },
        },
        initialVariables: {
            lookTruckOrderFee: {
                truckId: props.truckId,
                location: props.location,
                startPointLastCode: props.startPointLastCode,
                shopId: props.shopId,
            },
        },
    })
)
export default class LookTruckOrderFeeContainer extends React.Component {
    render () {
        return (
            <LookTruckOrderFee {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/trucks/pages/orderList/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as orderActions from 'actions/orders';
import OrderListInTruck from './contents';

@dataConnect(
    (state) => {
        const { truckId } = state.router.location.state || state.router.location.query;
        return { truckId, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(orderActions, dispatch),
    }),
    (props) => ({
        fragments: OrderListInTruck.fragments,
        variablesTypes: {
            orderListInTruck: {
                truckId: 'ID!',
            },
        },
        initialVariables: {
            orderListInTruck: {
                truckId: props.truckId,
            },
        },
    })
)
export default class OrderListInTruckContainer extends React.Component {
    render () {
        return (
            <OrderListInTruck {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/trucks/pages/payForCarryPartment/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as truckActions from 'actions/trucks';
import PayForCarryPartment from './contents';

@dataConnect(
    (state) => {
        const { truckId, record } = state.router.location.state || state.router.location.query;
        return { truckId, record, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(truckActions, dispatch),
    }),
    (props) => {
        return {
            fragments: PayForCarryPartment.fragments,
            variablesTypes: {
                truck: {
                    truckId: 'ID!',
                },
            },
            initialVariables: {
                truck: {
                    truckId: props.truckId,
                },
            },
        };
    }
)
export default class PayForCarryPartmentContainer extends React.Component {
    render () {
        return (
            <PayForCarryPartment {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/trucks/pages/selectCarryPartment/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as truckActions from 'actions/trucks';
import SelectCarryPartment from './contents';

@dataConnect(
    (state) => {
        const { truckId, record } = state.router.location.state || state.router.location.query;
        return { truckId, record, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(truckActions, dispatch),
    }),
    (props) => {
        return {
            fragments: SelectCarryPartment.fragments,
            variablesTypes: {
                carryPartments: {
                    truckId: 'ID!',
                },
            },
            initialVariables: {
                carryPartments: {
                    truckId: props.truckId,
                },
            },
        };
    }
)
export default class SelectCarryPartmentContainer extends React.Component {
    render () {
        return (
            <SelectCarryPartment {...this.props} />
        );
    }
}

```

* PDShop_Client_PC/project/App/shared/pages/client/pages/trucks/pages/showDetail/index.js

```js
import React from 'react';
import { dataConnect } from 'relatejs';
import { bindActionCreators } from 'redux';
import * as truckActions from 'actions/trucks';
import ShowDetail from './contents';

@dataConnect(
    (state) => {
        const { truckId, record, workTruckList } = state.router.location.state || state.router.location.query;
        return { truckId, record, workTruckList, keepLastKeepData: true };
    },
    (dispatch) => ({
        actions : bindActionCreators(truckActions, dispatch),
    }),
    (props) => {
        return {
            fragments: ShowDetail.fragments,
            variablesTypes: {
                truck: {
                    truckId: 'ID!',
                },
            },
            initialVariables: {
                truck: {
                    truckId: props.truckId,
                },
            },
        };
    }
)
export default class ShowDetailContainer extends React.Component {
    render () {
        return (
            <ShowDetail {...this.props} />
        );
    }
}

```
