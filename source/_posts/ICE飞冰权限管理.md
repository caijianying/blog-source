---
title: ICE飞冰权限管理
tags:
  - 教程
categories:
  - 前端技术
toc: false
date: 2019-04-07 21:14:43
---

## 前言
前端页面权限在日常开发中，主要有以下：

- 登录授权，用户没有登录只能访问登录页面，如果处于登录状态则跳转到当前用户的默认首页；

- 路由授权，当前登录用户的角色，如果对一个 URL 没有权限访问，则跳转到 403 页面；

- 数据授权，当访问一个没有权限的 API，则跳转到 403 页面；

- 操作授权，当页面中某个按钮或者区域没有权限访问则在页面中隐藏

本文主要针对ICE飞冰模板，介绍下如何开发权限管理，主要是使用，不涉及任何架构设计及原理，关于原理可以参考源码分析，或者参考本文参考链接。
## 实践

首先先安装依赖，因为飞冰权限是使用[Authorized](https://pro.ant.design/components/Authorized-cn/) 权限组件实现了基本的权限管理方案。
```
npm install antd --save
npm install ant-design-pro@latest --save
```
因为ant-design-pro依赖antd，因此需要首先安装antd

在开始之前需要新增两个util,一个设置登录权限，一个为了重新渲染权限

authority.js
```
// use localStorage to store the authority info, which might be sent from server in actual project.
export function getAuthority() {
  return localStorage.getItem('ice-pro-authority') || 'admin';
}

export function setAuthority(authority) {
  return localStorage.setItem('ice-pro-authority', authority);
}

export function removeAuthority() {
  return localStorage.removeItem('ice-pro-authority');
}

```

Authorized.js
```
import RenderAuthorized from 'ant-design-pro/lib/Authorized';
import { getAuthority } from './authority';

let Authorized = RenderAuthorized(getAuthority()); // eslint-disable-line

// 更新权限
const reloadAuthorized = () => {
  Authorized = RenderAuthorized(getAuthority());
};

export { reloadAuthorized };
export default Authorized;

```
### 登录授权

在登录页面，引入依赖，然后再login方法处设置权限，如下：
```
import { setAuthority } from '../../utils/authority';
import { reloadAuthorized } from '../../utils/Authorized';

// 这里设置权限为admin,添加该方法到login验证通过的方法处。
setAuthority('admin');
//重新渲染页面
reloadAuthorized();
```

退出
```
removeAuthority()
```

### 路由授权
编辑routerConfig.js,在需要设置权限的地方添加authority属性，值可以是string,数组。例如：
```
  {
    path: '/approve/tasks',
    component: ApproveTask,
    authority: 'admin',
  }
```
在MainRoutes.jsx 设置重定向地址
```
import {AuthorizedRoute} from 'ant-design-pro/lib/Authorized';
/**
   * 根据菜单取得重定向地址.
   */
  getRedirectData = () => {
    const redirectData = [];
    const getRedirect = (item) => {
      if (item && item.children) {
        if (item.children[0] && item.children[0].path) {
          redirectData.push({
            from: `${item.path}`,
            to: `${item.children[0].path}`,
          });
          item.children.forEach((children) => {
            getRedirect(children);
          });
        }
      }
    };

    asideMenuConfig.forEach(getRedirect);

    return redirectData;
  };

  /**
   * 渲染权限路由组件
   */
  renderAuthorizedRoute = (item, index) => {
    return item.component ? (
      <AuthorizedRoute
        key={index}
        path={item.path}
        component={item.component}
        exact={item.exact}
        authority={item.authority}
        redirectPath="/exception/403"
      />
    ) : null;
  };

  render() {
    const redirectData = this.getRedirectData();
    return (
      <Switch>
        {/* 渲染权限路由表 */}
        
        {routerData.map(this.renderAuthorizedRoute)}
        
        {/* 路由重定向，嵌套路由默认重定向到当前菜单的第一个路由 */}
        
        {redirectData.map((item, index) => {
          return <Redirect key={index} exact from={item.from} to={item.to} />;
        })}

        {/* 根路由默认重定向到 /dashboard */}
        <Redirect from="/" to="/user/login" />

        {/* 未匹配到的路由重定向到 NotFound */}
        <Route component={NotFound} />
      </Switch>
    );
  }
}
```
这里跳转到403页面，这里不再说明如何增加403页面
### 菜单授权
修改meConfig.js。在需要添加权限地方，设置角色名称，`authority: 'admin'`,子菜单会继承父菜单的权限。详细案例如下：
```
const asideMenuConfig = [
  {
    name: 'Topics',
    path: '/topics',
    icon: 'home2',
    children: [
      { name: 'Topic List', path: '/topics/list', authority: ['admin', 'member'] },
      { name: 'My Task', path: '/topics/task', authority: ['admin', 'member'] },
    ],
  },
  {
    name: 'Approve',
    path: '/approve',
    icon: 'publish',
    authority: 'admin',
    children: [
      { name: 'Task List', path: '/approve/tasks' },
    ],
  },
];
```
在`\src\layouts\BasicLayout\components\Aside`页面添加权限校验：
```
import RenderAuthorized from 'ant-design-pro/lib/Authorized';
import { getAuthority } from '../../../../utils/authority';
import { asideMenuConfig } from '../../../../menuConfig';

//省略部分方法
/**
   * 权限检查
   */
  checkPermissionItem = (authority, ItemDom) => {
    if (Authorized.check) {
      const { check } = Authorized;
      return check(authority, ItemDom);
    }

    return ItemDom;
  };


  /**
   * 获取菜单项数据
   */
  getNavMenuItems = (menusData) => {
    if (!menusData) {
      return [];
    }

    return menusData
      .filter(item => item.name && !item.hideInMenu)
      .map((item, index) => {
        const ItemDom = this.getSubMenuOrItem(item, index);
        return this.checkPermissionItem(item.authority, ItemDom);
      })
      .filter(item => item);
  };


  /**
   * 二级导航
   */
  getSubMenuOrItem = (item, index) => {
    if (item.children && item.children.some(child => child.name)) {
      const childrenItems = this.getNavMenuItems(item.children);

      if (childrenItems && childrenItems.length > 0) {
        return (
          <SubNav
            key={index}
            label={
              <span>
                {item.icon ? (
                  <FoundationSymbol size="small" type={item.icon} />
                        ) : null}
                <span className="ice-menu-collapse-hide">
                  {item.name}
                </span>
              </span>
                    }
          >
            {childrenItems}
          </SubNav>
        );
      }
      return null;
    }

    const linkProps = {};
    if (item.newWindow) {
      linkProps.href = item.path;
      linkProps.target = '_blank';
    } else if (item.external) {
      linkProps.href = item.path;
    } else {
      linkProps.to = item.path;
    }
    return (
      <Item key={item.path}>
        <Link {...linkProps}>
          <span>
            {item.icon ? (
              <FoundationSymbol size="small" type={item.icon} />
            ) : null}
            <span className="ice-menu-collapse-hide">{item.name}</span>
          </span>
        </Link>
      </Item>
    );
  };

  render() {
    const { location } = this.props;
    const { pathname } = location;

    return (
      <Layout.Aside width="240" theme="light" className="custom-aside">
        <Nav
          defaultSelectedKeys={[pathname]}
          selectedKeys={[pathname]}
          onOpen={this.onOpenChange}
          openKeys={this.state.openKeys}
          className="custom-menu"
        >
          {this.getNavMenuItems(asideMenuConfig)}
        </Nav>
        {/* 侧边菜单项 end */}
      </Layout.Aside>
    );
  }
```
### 操作授权
这里说的数据操作授权，即根据不同权限在页面显示不同的菜单,这里是实现只有管理员角色才能操作的按钮。
```
import RenderAuthorized from 'ant-design-pro/lib/Authorized';
import { getAuthority } from '../../utils/authority';
const Authorized = RenderAuthorized(getAuthority());

ReactDOM.render(
  <div>
    <Authorized authority="admin" noMatch={noMatch}>
      按钮....
    </Authorized>
  </div>,
  mountNode,
);
```
## 参考
1. [权限管理](https://alibaba.github.io/ice/docs/pro/authority)
2. [Ant Design Pro权限(Authorized)管理模块深入解读](https://wsonh.com/article/76.html)