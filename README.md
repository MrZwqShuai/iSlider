<template>
  <div id="wrapper">
    <div class="tabbar">
      <ul class="ul-list">
        <li class="list" v-for="(tab, index) in tabbar" :class="{active: activeIdx === index}" :key="tab.id">{{tab.title}}</li>
      </ul>
    </div>
    <div class="zwq-slide" ref="slide">
      <ul class="ul-list slide-active" ref="ulLst">
        <li class="slide-item" v-for="(content, idx) in contentLst" ref="listItem">
          {{idx+1}}.{{content.body}}
        </li>
      </ul>
    </div>
    <div class="addBtn">
      <div class="add" @click="add">增加</div>
      <div class="delete" @click="deleteItem">删除</div>
    </div>
  </div>
</template>
<script> 
import zwqSlide from '../utils/slide'
export default {
  name: 'test',
  props: {
    tabbar: {
      type: Array,
      default: function() {
        return [{
          id: 0,
          title: '首页'
        }, {
          id: 1,
          title: '热门'
        }, {
          id: 2,
          title: '收藏'
        }]
      }
    },
    contentLst: {
      type: Array,
      default: function() {
        return [{
          id: 0,
          body: '我是内容0'
        }, {
          id: 1,
          body: '我是内容1'
        }, {
          id: 2,
          body: '我是内容2'
        }]
      }
    }
  },
  data() {
    return {
      touchSlide: '',
      activeIdx: 0,
      apiConfig: [{
        url: 'http://www.baidu.com',
        data: {
          t: 'datalist'
        }
      },{
        url: 'http://www.baidu.com',
        data: {
          t: 'datalist1'
        }
      }]
    }
  },
  computed: {
  },
  methods: {
    apiCall () {

    },
    setSlideWidth() {
      if (this.zwqSlide) {
        this.zwqSlide = null
        this.activeIdx = 0
      }
      this.$nextTick(() => {
      this.children = this.$refs.ulLst.children
      let childWidth = this.$refs.slide.clientWidth
      let width = 0
      for (let i = 0; i < this.children.length; i++) {
        this.children[i].style.width = childWidth + 'px'
        width += childWidth
      }
      this.$refs.ulLst.style.width = width + 'px'
      this.zwqSlide = new zwqSlide({ el: this.$refs.ulLst, vm: this, slideEl: this.$refs.slide, activeClassName: 'active', activeIdx: this.activeIdx})
      console.log(this.zwqSlide.apiUpdate)
      let f1 = (apiConfig) => {
        this.zwqSlide.api(apiConfig)
      }
      this.zwqSlide.apiSubscrible(f1)
      this.zwqSlide.apiUpdate({
        url: 'http://www.baidu.com',
        data: {
          t: 'zwq'
        }
      })
      })
    },
    add () {
      this.contentLst.push({
        id: 0,
        body: '我是内容x'
      })
      this.tabbar.push({
          id: 0,
          title: 'tab'
      })
      this.setSlideWidth()
    },
    deleteItem () {
      this.contentLst.pop({
        id: 0,
        body: '我是内容x'
      })
      this.tabbar.pop({
          id: 0,
          title: 'tab'
      })
      this.setSlideWidth()
    }
  },
  watch: {
    activeIdx: function (val) {
      console.log('this.apiConfig[val]', this.apiConfig[val])
      this.zwqSlide.apiUpdate(this.apiConfig[val])
    }
  },
  mounted() {
    this.setSlideWidth()
  }
}
</script>
<style scoped>
#wrapper {
  font-size: .6rem;
}
.tabbar .ul-list {
  display: flex;
}
.tabbar .ul-list .list {
  flex: 1;
  height: 2rem;
  line-height: 2rem;
  display: inline-block;
  font-size: .6rem;
  text-align: center;
  border-bottom: 1px solid rgba(153, 153, 153, 0.18);
  box-sizing: border-box;
}

.zwq-slide {
  height: 12rem;
  overflow: hidden;
}

.zwq-slide ul {
  height: 100%;
  overflow: hidden;
}

.slide-active {
  transition-property: transform;
  transition-timing-function: cubic-bezier(0.165, 0.84, 0.44, 1);
  transition-duration: 300ms;
}

.slide-item {
  height: 100%;
  float: left;
}
.addBtn {
  position: fixed;
  bottom: 1rem;
  right: 1rem;
  font-size: .5rem;
}
.addBtn .add, .addBtn .delete{
    border: 1px solid #999;
    border-radius: 5px;
    padding: 6px;
    margin: 10px 0;
}
.active {
  color: orange;
}
</style>





class apiObserver {
  constructor () {
    this.apiFns = []
  }
  apiSubscrible (fn) {
    this.apiFns.push(fn)
  }
  apiUnSubscrible (fn) {
    this.apiFns = this.apiFns.filter((e) => {
      if (e !== fn) {
        return e
      }
    })
  }
  apiUpdate (coustom, Obj) {
    let scope = Obj || window
    this.apiFns.forEach((e) => {
      e.call(Obj, coustom)
    })
  }
}

export default class TouchSlide extends apiObserver{
  constructor(options) {
    super()
    this.options = options || window
    this.el = this.options.el
    this.vm = this.options.vm
    this.slideEl = this.options.slideEl
    this.width = 0
    this.startX = 0
    this.activeIdx = this.options.activeIdx = 0
    this.maxWidth = this.el.offsetWidth
    this.itemWidth = this.slideEl.offsetWidth
    this.cls = this.options.activeClassName
    this.start()
    this.move()
    this.end()
  }
  start() {
    // this.addClass('slide-active')
    this.el.addEventListener('touchstart', (e) => {
      let startTouch = e.targetTouches[0]
      this.startX = startTouch.pageX
    }, false)
  }
  move() {
    this.el.addEventListener('touchmove', (e) => {
      this.touch = e.targetTouches[0]
      let moveX = this.touch.pageX
      // console.log(moveX, this.el.style)
      let distance = moveX - this.startX
      this.el.style.webkitTransform = `translateX(${-this.width + distance}px)`
    }, false)
  }
  end() {
    // this.removeClass('zwq')
    this.el.addEventListener('touchend', (e) => {
      let endX = e.changedTouches[0].pageX
      let distance = endX - this.startX
      if (Math.abs(distance) > this.itemWidth / 2) {
        if (distance < 0) {
          if (this.width >= this.maxWidth - this.itemWidth) {
            this.el.style.webkitTransform = `translateX(${-this.width}px)`
          } else {
            this.width += this.itemWidth
            this.el.style.webkitTransform = `translateX(${-this.width}px)`
            this.vm.activeIdx ++
            console.log(222, this.vm.activeIdx)
          }
        } else {
          if (this.width <= 0) {
            this.el.style.webkitTransform = `translateX(${-this.width}px)`
          } else {
            this.width -= this.itemWidth
            this.el.style.webkitTransform = `translateX(${-this.width}px)`
            this.vm.activeIdx --
          }
        }
      } else {
        this.el.style.webkitTransform = `translateX(${-this.width}px)`
      }
    })
  }
  api (apiConfig) {
    this.$ajax({
      url: apiConfig.url,
      data: apiConfig.data,
      error: () => {
        this.$warn(`数据错误(this.apiConfig.data.t || this.apiConfig.data.q)`)
      }
    }).then(({data}) => {
      if (data.code === 0) {
        // do something
      } else {
        this.$warn(data.message)
      }
    })
  }
  addClass(cls) {
    let classN = this.el.className
    let blank = (classN != '') ? ' ' : ''
    let addClsObj = classN + blank + cls
    this.el.className = addClsObj
  }
  hasClass(cls) {
    return this.el.className.match(new RegExp('(\\s|^)' + cls + '(\\s|$)'))
  }
  removeClass(cls) {
    if (this.hasClass(cls)) {
      var reg = new RegExp('(\\s|^)' + cls + '(\\s|$)');
      this.el.className = this.el.className.replace(reg, ' ');
    }
  }
}
