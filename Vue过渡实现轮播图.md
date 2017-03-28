# Vue 过渡实现轮播图

## Vue 过渡

Vue 的过渡系统是内置的，在元素从 DOM 中插入或移除时自动应用过渡效果。

过渡的实现要在目标元素上使用 transition 属性，具体实现参考[Vue2 过渡](https://cn.vuejs.org/v2/guide/transitions.html)

下面例子中我们用到[列表过渡](https://cn.vuejs.org/v2/guide/transitions.html#列表过渡)，可以先学习一下官方的例子

要同时渲染整个列表，比如使用 v-for，我们需要用到 <transition-group> 组件

## Vue 轮播图

我们先看这样一个列表

    <ul>
      <li v-for="list in slideList">
        <img :src="list.image" :alt="list.desc">
      </li>
    </ul>
    
这个列表要从实例(见文章末尾)中获取了三张图片，要使其中的图片产生轮播，我们需要用 <transition-group> 组件替换其中的 ul 标签，从而实现过渡组件的功能，完整的组件 DOM 内容如下，下面分段解释一下

    <div class="carousel-wrap" id="carousel">
        // 轮播图列表
        <transition-group tag="ul" class='slide-ul' name="list">
          <li v-for="(list,index) in slideList" :key="index" v-show="index===currentIndex" @mouseenter="stop" @mouseleave="go">
            <a :href="list.clickUrl" >
              <img :src="list.image" :alt="list.desc">
            </a>
          </li>
        </transition-group>
        // 轮播图位置指示
        <div class="carousel-items">
          <span v-for="(item,index) in slideList.length" :class="{'active':index===currentIndex}" @mouseover="change(index)"></span>
        </div>
    </div>
    
对应的数据结构如下：
    
    data: {
        slideList: [
            {
                "clickUrl": "#",
                "desc": "nhwc",
                "image": "http://dummyimage.com/1745x492/f1d65b"
            },
            {
                "clickUrl": "#",
                "desc": "hxrj",
                "image": "http://dummyimage.com/1745x492/40b7ea"
            },
            {
                "clickUrl": "#",
                "desc": "rsdh",
                "image": "http://dummyimage.com/1745x492/e3c933"
            }
        ],
        currentIndex: 0,
        timer: ''
    },
        
在使用 v-for 时，应给对应的元素绑定一个 key 属性，相当于 index 标识，在 <transition-group> 组件中，key 是必须的，这样一个轮播图的 DOM 结构就完成了
    
接下来我们看看轮播函数的实现，再来看组件中的 li 元素
    
    <li v-for="(list,index) in slideList" :key="index">
        <a :href="list.clickUrl" >
          <img :src="list.image" :alt="list.desc">
        </a>
    </li>
      
上面通过 v-for 渲染了 li 列表，并在其中插入了包含可点击跳转的图片，接下来看看如何实现轮播，轮播图的样式直接在后面给出大家 sass 代码，父元素 ul 设置 ```position: relative;overflow: hidden``` 后，li 大小设为和父元素相同，absolute 定位固定在父元素中，要让 li 按照顺序显示，需要用到 v-show 或 v-if 处理，通过 index 值来改变当前显示的 li ，本例 v-show 绑定条件 ```index===currentIndex```，用定时器改变 currentIndex 实现轮播

    <li v-for="(list,index) in slideList" :key="index" v-show="index===currentIndex" @mouseenter="stop" @mouseleave="go">
        <a :href="list.clickUrl" >
          <img :src="list.image" :alt="list.desc">
        </a>
    </li>
    
实例中的方法：

    created() {
        //在DOM加载完成后，下个tick中开始轮播
        this.$nextTick(() => {
            this.timer = setInterval(() => {
                this.autoPlay()
            }, 4000)
        })
    },
    go() {
        this.timer = setInterval(() => {
            this.autoPlay()
        }, 4000)
    },
    stop() {
        clearInterval(this.timer)
        this.timer = null
    },
    change(index) {
        this.currentIndex = index
    },
    autoPlay() {
        this.currentIndex++
        if (this.currentIndex > this.slideList.length - 1) {
            this.currentIndex = 0
        }
    }
    
DOM 中为每个轮播 li 元素绑定事件 ```@mouseenter="stop" @mouseleave="go"``` 事件,使轮播鼠标移入时停止，移出时再次开始。
    
轮播图现在位置指示，绑定类名 active 改变颜色，绑定 change() 方法，鼠标移到指示点时跳转轮播图

    <div class="carousel-items">
      <span v-for="(item,index) in slideList.length" :class="{'active':index===currentIndex}" @mouseover="change(index)"></span>
    </div>
    


sass 样式代码

    .carousel-wrap {
      position: relative;
      height: 453px;
      width: 100%;
      overflow: hidden;
      // 删除
      background-color: #fff;
    }
    
    .slide-ul {
      width: 100%;
      height: 100%;
      li {
        position: absolute;
        width: 100%;
        height: 100%;
        img {
          width: 100%;
          height: 100%;
        }
      }
    }
    
    .carousel-items {
      position: absolute;
      z-index: 10;
      top: 380px;
      width: 100%;
      margin: 0 auto;
      text-align: center;
      font-size: 0;
      span {
        display: inline-block;
        height: 6px;
        width: 30px;
        margin: 0 3px;
        background-color: #b2b2b2;
        cursor: pointer;
      }
      .active {
        background-color: $btn-color;
      }
    }
    
滑动动画设置，知识点详见 Vue 教程中的 [过渡 css 类名](https://cn.vuejs.org/v2/guide/transitions.html#过渡的-CSS-类名)

    .list-enter-active {
      transition: all 1s ease;
      transform: translateX(0)
    }
    
    .list-leave-active {
      transition: all 1s ease;
      transform: translateX(-100%)
    }
    
    .list-enter {
      transform: translateX(100%)
    }
    
    .list-leave {
      transform: translateX(0)
    }

完整 Vue 实例如下

    new Vue({
      el: '#carousel',
      data: {
        slideList: [
            {
                "clickUrl": "#",
                "desc": "nhwc",
                "image": "http://dummyimage.com/1745x492/f1d65b"
            },
            {
                "clickUrl": "#",
                "desc": "hxrj",
                "image": "http://dummyimage.com/1745x492/40b7ea"
            },
            {
                "clickUrl": "#",
                "desc": "rsdh",
                "image": "http://dummyimage.com/1745x492/e3c933"
            }
        ],
        currentIndex: 0,
        timer: ''
      },
      methods: {
        this.$nextTick(() => {
          this.timer = setInterval(() => {
            this.autoPlay()
          },4000)
        }) 
        go() {
          this.timer = setInterval(() => {
            this.autoPlay()
          },4000)
        },
        stop() {
          clearInterval(this.timer)
          this.timer = null
        },
        change(index) {
          this.currentIndex = index
        },
        autoPlay() {
          this.currentIndex++
          if (this.currentIndex > this.slideList.length - 1) {
            this.currentIndex = 0
          }
        }
      }
    })

以上就是 Vue 过渡实现的轮播图