# Vue 中的 v-cloak 解读  

## v-cloak 的作用和用法

用法：
> 这个指令保持在元素上直到关联实例结束编译。和 CSS 规则如 ```[v-cloak] { display: none }``` 一起用时，这个指令可以隐藏未编译的 Mustache 标签直到实例准备完毕。[官方API](https://cn.vuejs.org/v2/api/#v-cloak)


    <div id="app">
        {{msg}}
    </div>
    


HTML 绑定 Vue实例，在页面加载时会闪烁

![闪烁内容](http://upload-images.jianshu.io/upload_images/5256964-d5a2bde0dc1bd721.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 
  
然后才会出现 ```加载完成``` 字样，为了效果更明显，我们可以延后加载 Vue 实例

    setTimeout(() => {
        new Vue({
            el: '#app',
            data: {
                msg: 'hello'
            }
        })
    },2000)

v-cloak 可以解决这一问题，在 css 中加上

    [v-cloak] {
      display: none;
    }
    
在 html 中的加载点加上 v-cloak，就可以解决这一问题

    <div id="app" v-cloak>
        {{msg}}
    </div>
    
## Vue1.x 与 Vue2 中 v-cloak 的不同

Vue1 中，允许将 Vue 实例挂载在 body 上，而 Vue2 是不允许的，想对整个页面实例化，需要另外用一个 div 来容纳整个页面内容，对其进行实例化

这样在使用 v-cloak 时，同样需要用到这种方法

## 为什么我用的 v-cloak 无效？

在实际项目中，我们常通过 @import 来加载 css 文件

    @import "style.css"
    @import "index.css"

而 @import 是在页面 DOM 完全载入后才会进行加载，如果我们将 ```[v-cloak]``` 写在 @import 加载的 css 文件中，就会导致页面仍旧闪烁。

为了避免这种情况，我们可以将 ```[v-cloak]``` 写在 link 引入的 css 中，或者写一个内联 css 样式，这样就得到了解决。