    在调研一中的关于Django+React.js调研，我
    们采用了项目django-React-redux-base。
    https://github.com/Seedstars/django-react-redux-base
## 项目介绍
这是一个基于djang、React、redux的web开发框架。他采用了docker作为开发环境配置。在本次调研中，我在ubuntu上安装了docker及其相关组件，并成功在本地架设了相关服务，打开了index，还有login页面，但没有账号可供我登录。由于后端数据库是采用了postgres，想起了这学期数据库，所以不想在折腾了，就进入了读源码的环节。
## 项目文件及工具链分析
#### /docker
在/docker目录下是关于django、nginx、postgres相关的配置文件，采用docker进行开发，方便我们快速搭建环境。文件中/docker-compose.yml,和/docker-compose.yml 就是docker services的配置  
#### /py-requirements
在/py-requirements目录下的一系列txt是关于python和Django及其相关插件扩展的版本要求。
#### /screenshots
在/screenshots目录下的是web页面的截图。
#### /scripts
在/scripts目录下的是空的  
#### /src
在/src目录下是比较关键的代码部分。/src/accounts、/src/base和/src/djangoReactredux 是python后端Django相关部分。而/src/lib里是几个基础的函数。/src/static里面就是跟React、redux相关的jsp和相关的html文件。   
这方面我们在后面的分析中会重点分析源代码。
#### /tests
在/tests目录下是相关的测试工作。
#### /webpack
在/webpack目录下就是一些config.js的打包。  

  
在项目中我们还有一些文件，如/license等在此不赘述。

## 源代码核心分析
#### redux
对于 redux 我们先了解几个概念  
##### action
action是纯声明式的数据结构，只提供事件的所有要素，不提供逻辑。考虑下面这个action，他有一个类型type，AUTH_LOGIN_USER_SUCCESS，用户auth连接成功，还有一个payload{token，user}。这个action就告诉了我们发生“发生了什么”。
~~~
{
        type: AUTH_LOGIN_USER_SUCCESS,
        payload: {
            token,
            user
        }
}
~~~
在实际使用时候，我们往往会调用一个actionFactoryFunction来返回一个action。如/src/static/actions/auth.js中的这段代码所示，工厂模式的使用是的action更简洁明了方便。
~~~
export function authLoginUserSuccess(token, user) {
    sessionStorage.setItem('token', token);
    sessionStorage.setItem('user', JSON.stringify(user));
    return {
        type: AUTH_LOGIN_USER_SUCCESS,
        payload: {
            token,
            user
        }
    };
}
~~~
##### state
state就是状态，web页面中有那么多的组件信息，有那么多的操作带来的操作结果，所以就有state。
如/src/static/reducers/auth.js中的initialState
~~~
const initialState = {
    token: null,
    userName: null,
    isAuthenticated: false,
    isAuthenticating: false,
    statusText: null
};
~~~
##### reducer
reducer是一个匹配函数，action的发送是全局的：所有的reducer都可以捕捉到并匹配与自己相关与否，相关就拿走action中的要素进行逻辑处理，修改store（store这个概念在会面会提起）中的状态，不相关就不对state做处理原样返回。
简单说就是一个reducer以一个state和action为参数，返回一个state。
(previousState, action) => newState
在/src/static/utils中有创建reducer的函数。
~~~
export function createReducer(initialState, reducerMap) {
    return (state = initialState, action) => {
        const reducer = reducerMap[action.type];
        return reducer ? reducer(state, action.payload) : state;
    };
}
~~~
我们再来看在/src/static/reducers/auth.js的实际调用。
~~~
export default createReducer(initialState, {
    [AUTH_LOGIN_USER_REQUEST]: (state, payload) => {
        return Object.assign({}, state, {
            isAuthenticating: true,
            statusText: null
        });
    },
    [AUTH_LOGIN_USER_SUCCESS]: (state, payload) => {
        return Object.assign({}, state, {
            isAuthenticating: false,
            isAuthenticated: true,
            token: payload.token,
            userName: payload.user.email,
            statusText: 'You have been successfully logged in.'
        });
    }
});
~~~
在创建好reducer的中，他们根据action.type来匹配这个是不是自己要处理的动作，如果是，再用action.payload和原来的state来创造一新的state返回。

##### store
store就是state的集合。store有store.getState()来获取store内存的state。而store.dispatch(action)就可以通过action来匹配reducer来修改状态。
我们看/src/static/store/configureStore.dev.js的的代码。参数里有reducer和state，至于middleware先忽略。
~~~
    const store = createStoreWithMiddleware(createStore)(rootReducer, initialState);

~~~

## React和redux
React中的view都是继承自component的。我们要将React和redux联系起来。“使用”redux的state机制，并利用reducer机制来获得新的state。
##### connect
在/src/static/containers/login/index.js中有
~~~
export default connect(mapStateToProps, mapDispatchToProps)(LoginView);
~~~
connect方法不会改变原来的组件类loginView，反而返回一个新的已与Redux store连接的组件类。

##### mapStateToProps
传入connect的mapStateToProps方法，是将 Redux 的状态映射到React组件的props属性。任何时候store中的state变了，组件类中对应的props属性就会发生改变。
在/src/static/containers/login/index.js中有
~~~
const mapStateToProps = (state) => {
    return {
        isAuthenticated: state.auth.isAuthenticated,
        isAuthenticating: state.auth.isAuthenticating,
        statusText: state.auth.statusText
    };
};
~~~
##### mapDispatchToProps
mapDispatchToProps则是将Store中的dispatch方法直接注入到组件里，一般会用到 Redux 的辅助函数bindActionCreators()绑定所有action以及参数的dispatch就可以作为属性在组件里面作为函数简单使用了，不需要手动dispatch。
在/src/static/containers/login/index.js中有
~~~
const mapDispatchToProps = (dispatch) => {
    return {
        dispatch,
        actions: bindActionCreators(actionCreators, dispatch)
    };
};
~~~  
    
就这样我们可以实现监听事件，事件有权利回到所有状态顶层影响状态。然后顶层分发状态，让React组件被动地渲染。Redux和React就可以工作了


## React 中的jsx和Virtual DOM
如/static/containers/Root/Root.dev.js中代码所示  
render function里的就是jsx代码   
jsx是一种嵌入式的类似XML的语法。 它可以被转换成合法的JavaScript，尽管转换的语义是依据不同的实现而定的。其中类XML部分语法和HTML很像。
~~~
export default class Root extends React.Component {


    render() {
        return (
            <div>
                <Provider store={this.props.store}>
                    <div>
                        <Router history={this.props.history}>
                            {routes}
                        </Router>
                        <DevTools />
                    </div>
                </Provider>
            </div>
        );
    }
}
~~~

我们先来讨论传统的DOM tree 和 HTML。HTML是Hypertext Markup Language的英文缩写, 即超文本标记语言,不需要编译,直接由浏 HTML语言是一种标记语言,不需要编译,直接由浏览器执行。浏览器将HTML解析成树形的数据结构。这个数据结构就叫做DOM tree。是浏览器渲染主流程中非常重要的一环。DOM 树的构建过程是一个深度遍历过程：当前节点的所有子节点都构建好后才会去构建当前节点的下一个兄弟节点。  
DOM树的构建是一个相当慢的过程。如果每次对网页操作都重新构建整个DOM树，在网页日益复杂的今天，是非常不合适的。   
所以就有了React 的 Virtual DOM的出现。有了Virtual DOM之后我们就不用每次update都更新整个DOM tree了。jsx代码的解析过程对于类xml部分就会调用React.createElement()来构建ReactElement对象。
如下图代码
~~~
React.render(
    <div>
        <div>
            <div>content</div>
        </div>
    </div>,
    document.getElementById('example')
);
等价于
React.render(
    React.createElement('div', null,
        React.createElement('div', null,
            React.createElement('div', null, 'content')
        )
    ),
    document.getElementById('example')
);
~~~

而jsx中这些xml结构也会被解析树形的数据结构，而树的节点就是ReactElement，而这棵树就是所谓的React Virtual DOM tree。当每次update后就会生成一颗新的React Virtual DOM tree。React 相关组件会自动diff两棵树，并将差别操作到真正的DOM tree。由于多数操作都是针对Virtual DOM tree的，而js object的操作效率是比构建DOM tree高的多了，所以我们节约了大量时间和计算资源原本相比更新整颗DOM tree。


## React的封装性
软件的封装与抽象可以有效降低一个软件的复杂度，利于软件的开发与维护。   
React里到处都是extends React.Component的组件，有较好的封装性。很多组件都可以在其他项目中得到良好的复用。   
React引入了Virtual DOM，这一层封装让开发人员不用小心翼翼搞定DOM操作，并且保证效率，在开发大型程序上这一点非常重要。  
React与redux相配合使用带来的应用的状态管理成本和复杂度大幅度下降。   
React还引入了ReactRouter来对不同的操作赋予不同的路由。
我们有路由表如下
~~~
export default(
    <Route path="/" component={App}>
        <IndexRoute component={HomeView} />
        <Route path="login" component={LoginView} />
        <Route path="protected" component={requireAuthentication(ProtectedView)} />
        <Route path="*" component={NotFoundView} />
    </Route>
);
~~~

在/src/static/containers/home/index.js里我们点击后就会触发渲染/protected，这也是很好的一个组织组件的方法。
~~~
<div className="margin-top-medium text-center">
                    <p>Attempt to access some <Link to="/protected"><b>protected content</b></Link>.</p>
~~~


## Django+React.js前后端协作的API
#### login时候的协作
在login页面，从代码角度讲也就是/src/static/containers/Login/index.js 里的LoginView Component 的props里有：
~~~
        this.state = {
            formValues: {
                email: '',
                password: ''
            },
            redirectTo: redirectRoute
        };
~~~

其中的formValues顾名思意就是表单值，用来存我们登录的email和password。在 render()里代码如下：
~~~
        <Form ref={(ref) => { this.loginForm = ref; }}
              type={Login}
              options={LoginFormOptions}
              value={this.state.formValues}
              onChange={this.onFormChange}
        />
~~~
当我们输入email和password的时候 会调用组件onFormChange函数更新组件里的formValues的值。
~~~
    onFormChange = (value) => {
        this.setState({ formValues: value });
    };
~~~
在点击了submit按钮后，他会调用组件的login()函数。
~~~
    login = (e) => {
        e.preventDefault();
        const value = this.loginForm.getValue();
        if (value) {
            this.props.actions.authLoginUser(value.email, value.password, this.state.redirectTo);
        }
    };
~~~
value被读取出来，当做component和redux 已经绑定的返回action 的工厂函数authLoginUser的参数。
我们来看一下action uthLoginUser做了什么。
~~~
export function authLoginUser(email, password, redirect = '/') {
    return (dispatch) => {
        dispatch(authLoginUserRequest());
~~~
>他首先dispatch一个action authLoginUserRequest()
~~~
export function authLoginUserRequest() {
    return {
        type: AUTH_LOGIN_USER_REQUEST
    };
}
~~~
aciton对应的reducer 用会更新store里的state,然后这些状态讲变化传递到绑定的component里的props。比如当state里的isAuthenticating为true时，我们再去按submit就没有作用了。
~~~
export default createReducer(initialState, {
    [AUTH_LOGIN_USER_REQUEST]: (state, payload) => {
        return Object.assign({}, state, {
            isAuthenticating: true,
            statusText: null
        });
    },
~~~
>action authLoginUser() 接着将传过来的email和password编码后得到auth
~~~
        const auth = btoa(`${email}:${password}`);
~~~
>然后他就准备开始向后端传送数据。
用的是fetch语句，指定了url和methed 还有报文头。我们的auth就放在头里。
~~~
        return fetch(`${SERVER_URL}/api/v1/accounts/login/`, {
            method: 'post',
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json',
                'Authorization': `Basic ${auth}`
            }
        })
~~~
>接下来后端收到了报文。
~~~
urlpatterns = [
    url(r'^api/v1/accounts/', include('accounts.urls', namespace='accounts')),
]
urlpatterns = [
    url(_(r'^login/$'),
        accounts.views.UserLoginView.as_view(),
        name='login'),
]
~~~
根据两级的url匹配。django调用UserLoginView的post函数来处理报文
>在post函数里django完成这个用户的登录，并返回这个token和user。 
~~~
class UserLoginView(GenericAPIView):
    serializer_class = UserSerializer
    authentication_classes = (BasicAuthentication,)
    permission_classes = (IsAuthenticated,)

    def post(self, request):
        """User login with username and password."""
        token = AuthToken.objects.create(request.user)
        return Response({
            'user': self.get_serializer(request.user).data,
            'token': token
        })

~~~
  

>这时候数据流就回到了前端的react。回到前面的返回action的工厂函数authLoginUser的后半部分。他收到包后先检查HttpStatus，然后对response进行json的解析。
~~~
            .then(checkHttpStatus)
            .then(parseJSON)
            .then((response) => {
                dispatch(authLoginUserSuccess(response.token, response.user));
                dispatch(push(redirect));
            })
            //中间略去一段异常处理的代码
                return Promise.resolve(); // TODO: we need a promise here because of the tests, find a better way
            });
    };
}
~~~
>然后用解析后得到的response.token, response.user做参数得到一个action。这个factory function 根据token和user设置了sessionStorage。然后返回的action被dispatch。 
~~~
export function authLoginUserSuccess(token, user) {
    sessionStorage.setItem('token', token);
    sessionStorage.setItem('user', JSON.stringify(user));
    return {
        type: AUTH_LOGIN_USER_SUCCESS,
        payload: {
            token,
            user
        }
    };
}
~~~
对应的reducer是
~~~
export default createReducer(initialState, {
    [AUTH_LOGIN_USER_SUCCESS]: (state, payload) => {
        return Object.assign({}, state, {
            isAuthenticating: false,
            isAuthenticated: true,
            token: payload.token,
            userName: payload.user.email,
            statusText: 'You have been successfully logged in.'
        });
    },
~~~
更新store里的state。而且state会向下传递到绑定的componen里的props。   
>返回action的工厂函数authLoginUser的最后部分是dispatch(push(redirect))。根据前端的react-route设置，网页会调回index。结束整个登录的过程。   

至此整个登录流程的前后端交互到此结束。    

#### Protected时候的协作
根据react router当path为"protected"而且isAuthenticated为true时，就会跳转到component ProtectedView。   
在组件渲染前会调用 componentWillMount()，会由factory function dataFetchProtectedData()产生一个action。
~~~
    componentWillMount() {
        const token = this.props.token;
        this.props.actions.dataFetchProtectedData(token);
    }
~~~
>factory function dataFetchProtectedData()先dispatch(dataFetchProtectedDataRequest())，这个会通过reducer是store里的isFetching为true
~~~
export function dataFetchProtectedData(token) {
    return (dispatch, state) => {
        dispatch(dataFetchProtectedDataRequest());
~~~
>然后就是通过fetch语句向服务端发报文
~~~
        return fetch(`${SERVER_URL}/api/v1/getdata/`, {
            credentials: 'include',
            headers: {
                Accept: 'application/json',
                Authorization: `Token ${token}`
            }
        })
~~~
>django收到报文后根据url匹配
~~~
urlpatterns = [
    url(r'',
        base_views.ProtectedDataView.as_view(),
        name='protected_data'),
]
~~~
>调用ProtectedDataView的get()返回了data
~~~
class ProtectedDataView(GenericAPIView):
    """Return protected data main page."""

    authentication_classes = (TokenAuthentication,)
    permission_classes = (IsAuthenticated,)

    def get(self, request):
        """Process GET request and return protected data."""

        data = {
            'data': 'THIS IS THE PROTECTED STRING FROM SERVER',
        }

        return Response(data, status=status.HTTP_200_OK)
~~~
>接着数据流回到前端。react factory function dataFetchProtectedData()接着执行。
check了HttpStatus，json解析后得到数据，产生了action并进行dispatch。
~~~
            .then(checkHttpStatus)
            .then(parseJSON)
            .then((response) => {
                dispatch(dataReceiveProtectedData(response.data));
            })
~~~
对应的reducer更新store内的state然后，component ProtectedView里的state也被更新。然后进行了render，收到的信息'THIS IS THE PROTECTED STRING FROM SERVER'被显示到页面上某一预想位置上。
# 总结和体会
项目一是Django+React.js调研，搭建docker环境的时候遇到了很多问题。在看源码的时候，由于不懂js和html也遇到了很多问题。React和redux的搭配使我印象深刻。顶层分发状态，让React组件被动地渲染。监听事件，事件有权利回到所有状态顶层影响状态。这个思想很巧妙。也非常利于我们工作上的使用。此外对react一些api有了了解。   
关于前后端api交互，react 做了很多路由的工作，所以与后端Django交互主要是交换信息。react通过一个action factory function里的fetch语句来往后台发送信息。经过Django里的url匹配后，后端调用对应的view class里的某些函数（如post，get）进行交互。信息发回到react端时，可以被扔进factory funcion里产生action再通过reducer来改变某些需要改变的state和组件信息。