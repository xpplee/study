# react, dva, umijs学习，工作中的坑

一直比较喜欢阿里系的技术，所以这次微信网站选型用的是react, dva, umijs, ant desgin mobile

下面是工作中遇到的问题和的解决方案
1） 同一url，根据不同角色，跳转到不同的页面
因为dva框架默认会初始化2次对应的react组件，所以不能在对应的组件里面跳转，最好在model里subscriptions里跳转

 ```js
 //比如下面/home会根据不同登陆角色跳转不同页面
  subscriptions: {
    setup({ dispatch, history }) {
      history.listen(location => {
        if (location.pathname === '/home') {
          dispatch({
            type: 'updateState',
            payload: {
              selected: 'home',
            },
          });
          const loginUser = JSON.parse(localStorage.getItem("loginUser"));
          if (loginUser.role === 'COMPANY') {
            
          } else if (['SERVER', 'MANAGER'].includes(loginUser.role)) {
            dispatch(routerRedux.push({
              pathname: '/order/list',
            }));
          } else {
            localStorage.removeItem('loginUser');
            dispatch(routerRedux.push({
              pathname: '/login',
            }));
          }
          return;
        }
       
        }
        }
```
2) 页面2请求的数据是根据页面1选择的数据做为条件，如果页面要返回2可以返回页面1,再选择不一样的数据进入页面2，那么，刚进入面页2马上会显示上一次页面1选择的数据请求的结果，等一会取到最新一次页面1选择的数据请求的结果，页面2数据才会变正确。
 页面2 的model
 ```js
  namespace: 'selectServices',
  state: {
    services: { content: [] },
    selectProjectId: null,//页面1选择的参数
  }
```
 解决方法是在页面2的组件的mapStateToProps方法页面做验证，如果页面2的数据和页面1传的参数不一样，那么页面2的state就设置为空
 ```js
function mapStateToProps(state) {
  let {  services, selectProjectId } = state.selectServices;
  if (state.routing.location.state && selectProjectId) {
    if (state.routing.location.state.selectProjectId !== selectProjectId) {
      services = { content: [] };
      projects = [];
      selectProjectId = null;
    }
  }
  return {
    projects, services, selectProjectId
  };
}

export default connect(mapStateToProps)(createForm()(SelectService));
```
