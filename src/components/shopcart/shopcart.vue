<template>
  <div class="shopcart">
    <div class="content">
      <div class="content-left">
        <div class="logo-wrapper">
          <div class="logo" :class="{'highlight': totalCount>0}">
            <i class="icon-shopping_cart" :class="{'highlight': totalCount>0}"></i>
          </div>
          <div class="num" v-show="totalCount>0">{{totalCount}}</div>
        </div>
        <div class="price" :class="{'highlight': totalPrice>0}">￥{{totalPrice}}</div>
        <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
      </div>
      <div class="content-right">
        <div class="pay" :class="payClass">
          {{payDesc}}
        </div>
      </div>
    </div>
    <div class="ball-container" name="drop">
      <div v-for="(ball, index) in balls" :key="index">
      <transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
        <div class="ball" v-show="ball.show">
          <div class="inner inner-hook"></div>
        </div>
      </transition>
      </div>
    </div>
    <div class="shopcart-list" v-show="listShow">
      <div class="list-header">
        <h1 class="title">购物车</h1>
        <span class="empty">清空</span>
      </div>
      <div class="list-content">
        <ul>
          <li class="food" v-for="(food, index) in selectFoods" :key="index">
            <span class="name">{{food.name}}</span>
            <div class="price">
              <span>￥{{food.price*food.count}}</span>
            </div>
            <div class="cartcontrol-rapper">
              <cartcontrol :food="food"></cartcontrol>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
import cartcontrol from 'components/cartcontrol/cartcontrol'
export default {
  props: {
    selectFoods: {
      type: Array,
      default () {
        return []
      }
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      balls: [
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        }
      ],
      dropBalls: []
    }
  },
  computed: {
    totalPrice () {
      let total = 0
      this.selectFoods.forEach((food) => {
        // 优化精度丢失解决方案
        let foodPrice = food.price
        let foodCount = food.count
        let newFoodPrice = 0
        // 解决精度丢失问题 V3.0
        newFoodPrice = this.accMul(foodPrice, foodCount)
        total = this.accAdd(total, newFoodPrice)
        // 解决精度丢失问题 V2.0
        // if (foodPrice >= 0 && foodPrice < 100) {
        //   total += ((foodPrice * 10) / 10) * foodCount
        // } else if (foodPrice >= 100 && foodPrice < 1000) {
        //   total += ((foodPrice * 100) / 100) * foodCount
        // } else {
        //   console.log('精度丢失')
        // }
        // 解决精度丢失问题 V1.0
        // total += ((food.price * 10) * food.count) / 10
      })
      return total
    },
    totalCount () {
      let count = 0
      this.selectFoods.forEach((food) => {
        count += food.count
      })
      return count
    },
    payDesc () {
      if (this.totalPrice === 0) {
        return `￥${this.minPrice}元起送`
      } else if (this.totalPrice < this.minPrice) {
        let diff = this.minPrice - this.totalPrice
        return `还差￥${diff}元起送`
      } else {
        return '去结算'
      }
    },
    payClass () {
      if (this.totalPrice < this.minPrice) {
        return 'not-enough'
      } else {
        return 'enough'
      }
    }
  },
  methods: {
    // 解决精度丢失问题(加减乘除)
    // 加法
    accAdd (arg1, arg2) {
      let len1 = 0
      let len2 = 0
      let times = 0
      let result = 0
      try {
        len1 = arg1.toString().split('.')[1].length
      } catch (e) {
        len1 = 0
      }
      try {
        len2 = arg2.toString().split('.')[1].length
      } catch (e) {
        len2 = 0
      }
      times = Math.pow(10, Math.max(len1, len2))
      result = (arg1 * times + arg2 * times) / times
      return result
    },
    // 减法
    accSub (arg1, arg2) {
      let len1 = 0
      let len2 = 0
      let times = 0
      let maxPrecision = 0
      let result = 0
      try {
        len1 = arg1.toString().split('.')[1].length
      } catch (e) {
        len1 = 0
      }
      try {
        len2 = arg2.toString().split('.')[1].length
      } catch (e) {
        len2 = 0
      }
      times = Math.pow(10, Math.max(len1, len2))
      maxPrecision = (len1 > len2) ? len1 : len2
      result = ((arg1 * times + arg2 * times) / times).toFixed(maxPrecision)
      return result
    },
    // 乘法
    accMul (arg1, arg2) {
      let s1 = arg1.toString()
      let s2 = arg2.toString()
      let times = 0
      let result = 0
      try {
        times += s1.split('.')[1].length
      } catch (e) {
      }
      try {
        times += s2.split('.')[1].length
      } catch (e) {
      }
      result = Number(s1.replace('.', '')) * Number(s2.replace('.', '')) / Math.pow(10, times)
      return result
    },
    // 除法
    accDiv (arg1, arg2) {
      let len1 = 0
      let len2 = 0
      let r1 = 0
      let r2 = 0
      let result = 0
      try {
        len1 = arg1.toString().split('.')[1].length
      } catch (e) {
      }
      try {
        len2 = arg2.toString().split('.')[1].length
      } catch (e) {
      }
      r1 = Number(arg1.toString().replace('.', ''))
      r2 = Number(arg2.toString().replace('.', ''))
      result = this.accMul((r1 / r2), Math.pow(10, len2 - len1))
      return result
    },
    drop (el) {
      for (let i = 0; i < this.balls.length; i++) {
        let ball = this.balls[i]
        if (!ball.show) {
          ball.show = true
          ball.el = el
          this.dropBalls.push(ball)
          return
        }
      }
    },
    beforeDrop (el) {
      let count = this.balls.length // 把所有true的小球取出来
      while (count--) {
        let ball = this.balls[count]
        if (ball.show) {
          let rect = ball.el.getBoundingClientRect()
          let x = rect.left - 32
          let y = -(window.innerHeight - rect.top - 22)
          el.style.display = ''
          el.style.webkitTransform = `translate3d(0, ${y}px, 0)`
          el.style.transform = `translate3d(0, ${y}px, 0)`
          let inner = el.getElementsByClassName('inner-hook')[0]
          inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`
          inner.style.Transform = `translate3d(${x}px, 0, 0)`
        }
      }
    },
    dropping (el) {
      /* eslint-disable no-unused-vars */
      let rf = el.offsetHeight //  HTMLElement.offsetHeight 是一个只读属性，它返回该元素的像素高度，高度包含该元素的垂直内边距和边框，且是一个整数。
      this.$nextTick(() => {
        el.style.webkitTransform = 'translate3d(0, 0, 0)'
        el.style.transform = 'translate3d(0, 0, 0)'
        let inner = el.getElementsByClassName('inner-hook')[0]
        inner.style.webkitTransform = 'translate3d(0, 0, 0)'
        inner.style.Transform = 'translate3d(0, 0, 0)'
      })
    },
    afterDrop (el) {
      let ball = this.dropBalls.shift()
      if (ball) {
        ball.show = false
        el.style.display = 'none'
      }
    }
  },
  components: {
    cartcontrol
  }
}
</script>
<style lang="stylus" rel="stylesheet/stylus">
.shopcart
  position: fixed
  left: 0
  bottom: 0
  height: 48px
  width: 100%
  z-index: 50
  .content
    display: flex
    background: #141d27
    font-size: 0
    color: rgba(255, 255, 255, 0.4)
    .content-left
      flex: 1
      .logo-wrapper
        display: inline-block
        position: relative
        top: -10px
        margin: 0 12px
        padding: 6px
        width: 56px
        height 56px
        box-sizing: border-box
        vertical-align: top
        border-radius: 50%
        background: #141d27
        .logo
          width: 100%
          height: 100%
          border-radius: 50%
          background: #2b343c
          text-align: center
          &.highlight
            background: rgb(0, 160, 220)
          .icon-shopping_cart
            line-height: 44px
            font-size: 24px
            color: #80858a
            &.highlight
              color: #ffffff
        .num
          position: absolute
          top: 0
          right: 0
          width: 24px
          height: 16px
          line-height: 16px
          text-align: center
          border-radius: 16px
          font-size: 9px
          font-weight: 700
          color: #ffffff
          background: rgb(240, 20, 20)
          box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4)
      .price
        display: inline-block
        vertical-align: top
        margin-top: 12px
        line-height: 24px
        padding-right: 12px
        box-sizing: border-box
        border-right: 1px solid rgba(255, 255, 255, 0.1)
        font-size: 16px
        font-weight: 700
        &.highlight
          color: #ffffff
      .desc
        display: inline-block
        vertical-align: top
        margin: 12px 0 0 12px
        line-height: 24px
        font-size: 10px
    .content-right
      flex: 0 0 105px
      width: 105px
      background: #2b333b
      .pay
        height: 48px
        line-height: 48px
        text-align: center
        font-size: 12px
        font-weight: 700
        &.not-enough
          background: #2b333b
        &.enough
          background: #00b43c
          color: #ffffff
  .ball-container
    .ball
      position: fixed
      left: 32px
      bottom: 22px
      z-index: 200
      transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
      .inner
        width: 16px
        height: 16px
        border-radius: 50%
        background: rgb(0, 160, 220)
        transition: all 0.4s linear
</style>
