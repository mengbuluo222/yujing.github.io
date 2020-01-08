vue-router 使用详情

# vue-router实现原理

SPA(single page application):单一页面应用程序，只有一个完整的页面；它在加载页面时，不会加载整个页面，而是只更新某个指定的容器中内容。单页面应用(SPA)的核心之一是:**更新视图而不重新请求页面**;vue-router在实现单页面前端路由时，提供了两种方式：Hash模式和History模式。

## 1、hash模式

随着 ajax 的流行，异步数据请求交互运行在不刷新浏览器的情况下进行。而异步交互体验的更高级版本就是 SPA —— 单页应用。单页应用不仅仅是在页面交互是无刷新的，连页面跳转都是无刷新的，为了实现单页应用，所以就有了前端路由。类似于服务端路由，前端路由实现起来其实也很简单，就是匹配不同的 url 路径，进行解析，然后动态的渲染出区域 html 内容。但是这样存在一个问题，就是 url 每次变化的时候，都会造成页面的刷新。那解决问题的思路便是在改变url的情况下，保证页面的不刷新。在 2014 年之前，大家是通过 hash 来实现路由，url hash 就是类似于：

```ruby
http://www.xxx.com/#/login
```

这种 #。后面 hash 值的变化，并不会导致浏览器向服务器发出请求，浏览器不发出请求，也就不会刷新页面。另外每次 hash 值的变化，还会触发hashchange 这个事件，通过这个事件我们就可以知道 hash 值发生了哪些变化。然后我们便可以监听hashchange来实现更新页面部分内容的操作：

```jsx
function matchAndUpdate () {
   // todo 匹配 hash 做 dom 更新操作
}

window.addEventListener('hashchange', matchAndUpdate)
```

## 2、history 模式

14年后，因为HTML5标准发布。多了两个 API，pushState 和 replaceState，通过这两个 API 可以改变 url 地址且不会发送请求。同时还有popstate事件。通过这些就能用另一种方式来实现前端路由了，但原理都是跟 hash 实现相同的。用了HTML5的实现，单页路由的url就不会多出一个#，变得更加美观。但因为没有 # 号，所以当用户刷新页面之类的操作时，浏览器还是会给服务器发送请求。为了避免出现这种情况，所以这个实现需要服务器的支持，需要把所有路由都重定向到根页面。

```jsx
function matchAndUpdate () {
   // todo 匹配路径 做 dom 更新操作
}

window.addEventListener('popstate', matchAndUpdate)
```

# vue-router的使用

## 1、动态路由匹配

我们经常需要把某种模式匹配到的所有路由，全都映射到同个组件。例如，我们有一个 User 组件，对于所有 ID 各不相同的用户，都要使用这个组件来渲染。那么，我们可以在 vue-router 的路由路径中使用“动态路径参数”(dynamic segment) 来达到这个效果



```csharp
const User = {
  template: '<div>User</div>'
}

const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})
```

现在呢，像 /user/foo 和 /user/bar 都将映射到相同的路由。

一个“路径参数”使用冒号 : 标记。当匹配到一个路由时，参数值会被设置到 this.$route.params，可以在每个组件内使用。于是，我们可以更新 User 的模板，输出当前用户的 ID：

```dart
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
```

你可以在一个路由中设置多段『路径参数』，对应的值都会设置到 $route.params 中。例如：

![image-20200106095618907](/Users/mxj/Library/Application Support/typora-user-images/image-20200106095618907.png)

**提醒一下，当使用路由参数时，例如从/user/foo导航到/user/bar，原来的组件实例会被复用。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。不过，这也意味着组件的生命周期钩子不会再被调用。复用组件时，想对路由参数的变化作出响应的话，你可以简单地 watch (监测变化) $route 对象**

```csharp
const User = {
  template: '...',
  watch: {
    '$route' (to, from) {
      // 对路由变化作出响应...
    }
  }
}
```

**或者使用 2.2 中引入的 beforeRouteUpdate 守卫：**

```csharp
const User = {
  template: '...',
  beforeRouteUpdate (to, from, next) {
    // react to route changes...
    // don't forget to call next()
  }
}
```

## 2、嵌套路由

实际生活中的应用界面，通常由多层嵌套的组件组合而成。同样地，URL 中各段动态路径也按某种结构对应嵌套的各层组件，例如

![image-20200106095715387](/Users/mxj/Library/Application Support/typora-user-images/image-20200106095715387.png)

借助 vue-router，使用嵌套路由配置，就可以很简单地表达这种关系

```xml
<div id="app">
  <router-view></router-view>
</div>
```

```dart
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

这里的``是最顶层的出口，渲染最高级路由匹配到的组件。同样地，一个被渲染组件同样可以包含自己的嵌套 ``。例如，在 User 组件的模板添加一个 `router-view`

```xml
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}
```

要在嵌套的出口中渲染组件，需要在 VueRouter 的参数中使用 children 配置：

```csharp
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

## 3、编程式导航

注意：在 Vue 实例内部，你可以通过 `$router`访问路由实例。因此你可以调用 `this.$router.push`。

![image-20200106095826163](/Users/mxj/Library/Application Support/typora-user-images/image-20200106095826163.png)

```参数可以是一个字符串路径，或者一个描述地址的对象:// 字符串router.push(&#39;home&#39;)// 对象router.push({ path: &#39;home&#39; })// 命名的路由router.push({ name: &#39;user&#39;, params: { userId: 123 }})// 带查询参数，变成 /register?plan=privaterouter.push({ path: &#39;register&#39;, query: { plan: &#39;private&#39; }})const userId = &#39;123&#39;router.push({ name: &#39;user&#39;, params: { userId }}) // -&gt; /user/123router.push({ path: `/user/${userId}` }) // -&gt; /user/123// 这里的 params 不生效router.push({ path: &#39;/user&#39;, params: { userId }}) // -&gt; /user//router.go(n)这个方法的参数是一个整数，意思是在 history 记录中向前或者后退多少步，类似 window.history.go(n)// 在浏览器记录中前进一步，等同于 history.forward()router.go(1)// 后退一步记录，等同于 history.back()router.go(-1)// 前进 3 步记录router.go(3)// 如果 history 记录不够用，那就默默地失败呗router.go(-100)router.go(100)go
参数可以是一个字符串路径，或者一个描述地址的对象:

// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: 123 }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})

const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 这里的 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user

//router.go(n)这个方法的参数是一个整数，意思是在 history 记录中向前或者后退多少步，类似 window.history.go(n)
// 在浏览器记录中前进一步，等同于 history.forward()
router.go(1)

// 后退一步记录，等同于 history.back()
router.go(-1)

// 前进 3 步记录
router.go(3)

// 如果 history 记录不够用，那就默默地失败呗
router.go(-100)
router.go(100)
```

## 4、命名视图

有时候想同时 (同级) 展示多个视图，而不是嵌套展示，例如创建一个布局，有 sidebar (侧导航) 和 main (主内容) 两个视图，这个时候命名视图就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 router-view 没有设置名字，那么默认为 default。

```jsx
//name对应的是组件名字
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 components 配置 (带上 s)：

```cpp
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,//Foo是组件名字
        a: Bar,//Bar是组件名字
        b: Baz//Baz是组件名字
      }
    }
  ]
})
```

**嵌套命名视图**

`UserSettings` 组件的`部分应该是类似下面的这段代码：



```xml
<!-- UserSettings.vue -->
<div>
  <h1>User Settings</h1>
  <NavBar/>
  <router-view/>
  <router-view name="helper"/>
</div>
```

然后你可以用这个路由配置完成该布局：

```css
{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [{
    path: 'emails',
    component: UserEmailsSubscriptions
  }, {
    path: 'profile',
    components: {
      default: UserProfile,
      helper: UserProfilePreview
    }
  }]
}
```

## 5、重定向与别名

重定向也是通过 routes 配置来完成，下面例子是从 /a 重定向到 /b：

```csharp
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})
```

重定向的目标也可以是一个命名的路由：

```csharp
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: { name: 'foo' }}
  ]
})
```

甚至是一个方法，动态返回重定向目标：

```jsx
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: to => {
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}
  ]
})
```

**别名：**/a 的别名是 /b，意味着，当用户访问 /b 时，URL 会保持为 /b，但是路由匹配则为 /a，就像用户访问 /a 一样。

```csharp
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```

**总结：**

- 『重定向』的意思是：当用户访问/a时，URL将会被替换成/b，然后匹配路由为/b。**（换药换汤）**
- 别名的意思是：/a的别名是/b，意味着，当用户访问/b时，URL会保持为/b，但是路由匹配则为/a，就像用户访问/a 一样。**（换汤不换药）**



作者：Edward_95b5
链接：https://www.jianshu.com/p/5dff6811252d
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

