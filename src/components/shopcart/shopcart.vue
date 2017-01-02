<template>
  <div class="shopcart">
    <div class="content">
      <div class="content-left">
        <div class="logoWrapper" @click.stop.prevent="toogleList()">
          <div class="logo" :class="{'highlight':totalCount>0}">
            <span class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></span>
          </div>
          <div class="num" v-show="totalCount>0" :class="caculateCount">{{caculateCount}}</div>
        </div>
        <div class="price" :class="{'highlight':totalCount>0}">¥{{totalPrice}}</div>
        <div class="desc">另需配送费¥{{deliveryPrice}}元</div>
      </div>
      <div class="content-right" @click.stop.prevent="pay">
        <div class="pay" :class="payClass">
          {{payDesc}}
        </div>
      </div>
    </div>
    <div class="ballcontainer">
      <div transition="drop" v-for="ball in balls" v-show="ball.show" class="ball">
        <div class="inner inner-hook"></div>
      </div>
    </div>
    <div class="shopcart-list" v-show="listShow" transition="fold">
      <div class="list-header">
        <h1 class="title">购物车</h1>
        <span class="empty" @click="emptyList">清空</span>
      </div>
      <div class="list-content" v-el:list-content>
        <ul>
          <li class="food-item" v-for="food in selectFoods">
            <span class="name">{{food.name}}</span>
            <div class="price">
              <span>¥{{food.price*food.count}}</span>
            </div>
            <div class="cartCtrl-wrapper">
              <cartctrl :food="food"></cartctrl>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
  <div class="list-mask" v-show="listShow" @click="hideList" transition="fade"></div>
</template>

<script type="text/ecmascript-6">
  import cartCtrl from 'components/cartCtrl/cartCtrl.vue';
  import BScroll from 'better-scroll';

  export default{
    props: {
      deliveryPrice: {
        type: Number,
        default: 0
      },
      minPrice: {
        type: Number,
        default: 0
      },
      selectFoods: {
        type: Array,
        default() {
          return [
            {
              price: 10,
              count: 1
            }
          ];
        }
      }
    },
    data() {
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
          }
        ],
        dropballs: [],
        showlist: false
      };
    },
    watch: {
      totalPrice: function () {
        if (this.totalPrice === 0 || !this.totalPrice) {
          this.showlist = false;
        }
      }
    },
    methods: {
      drop(el) {
        //console.log('shopcart----drop()');
        for (let i = 0; i < this.balls.length; i++) {
          let ball = this.balls[i];
          if (!ball.show) {
            ball.show = true;
            ball.el = el;
            this.dropballs.push(ball);
            return;
          }
        }
      },
      toogleList() {
        //if (!event._constructed) {
        //  return;
        //}
        if (!this.totalCount) {
          return;
        }
        this.showlist = !this.showlist;
      },
      emptyList() {
        console.log('1');
        this.selectFoods.forEach((food) => {
          food.count = 0;
        });
      },
      hideList() {
        this.showlist = false;
      },
      pay() {
        if (this.totalPrice < this.minPrice) {
          return;
        }
        window.alert(`支付${this.totalPrice}`);
      }
    },
    computed: {
      totalPrice() {
        let total = 0;
        this.selectFoods.forEach((food) => {
          total += food.price * food.count;
        });
        return total;
      },
      totalCount() {
        let count = 0;
        this.selectFoods.forEach((food) => {
          count += food.count;
        });
        return count;
      },
      payDesc() {
        if (this.totalPrice === 0) {
          return `¥${this.totalPrice}元起送`;
        } else if (this.totalPrice < this.minPrice) {
          let diff = Math.abs(this.minPrice - this.totalPrice);
          return `还差¥${diff}元起送`;
        } else {
          return '去结算';
        }
      },
      payClass() {
        if (this.totalPrice < this.minPrice) {
          return 'not-enough';
        } else {
          return 'enough';
        }
      },
      caculateCount() {
        let total = 0;
        for (let i = 0; i < this.selectFoods.length; i++) {
          total += this.selectFoods[i].count;
        }
        return total;
      },
      listShow() {
          if (!this.showlist) {
            return false;
          } else {
            //this.showlist = true;
            //BScroll初始化和刷新
            this.$nextTick(() => {
              if (!this.scroll) {
                this.scroll = new BScroll(this.$els.listContent, {
                  click: true
                });
              } else {
                this.scroll.refresh();
              }
            });
            return true;
          }
      }
    },
    transitions: {
      drop: {
        beforeEnter(el) {
          //console.log('shopcart---beforEnter()');
          let count = this.balls.length;
          while (count--) {
            let ball = this.balls[count];
            if (ball.show) {
              let rect = ball.el.getBoundingClientRect();
              let x = rect.left - 32;
              let y = -(window.innerHeight - rect.top - 22);
              el.style.display = '';
              el.style.webkitTransform = `translate3d(0,${y}px,0)`;
              el.style.transform = `translate3d(0,${y}px,0)`;
              let inner = el.getElementsByClassName('inner-hook')[0];
              inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
              inner.style.transform = `translate3d(${x}px,0,0)`;
            }
          }
        },
        enter(el) {
          //console.log('shopcart---Enter()');
          //触发浏览器重绘
          /* eslint-disable no-unused-vars */
          let rf = el.offsetHeight;
          this.$nextTick(() => {
            el.style.webkitTransform = 'translate3d(0,0,0)';
            el.style.transform = 'translate3d(0,0,0)';
            let inner = el.getElementsByClassName('inner-hook')[0];
            inner.style.webkitTransform = 'translate3d(0,0,0)';
            inner.style.transform = 'translate3d(0,0,0)';
          });
        },
        afterEnter(el) {
          let ball = this.dropballs.shift();
          if (ball) {
            ball.show = false;
            el.style.display = 'none';
          }
        }
      }
    },
    components: {
      'cartctrl': cartCtrl
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .shopcart
    position fixed
    bottom 0
    left 0
    height 48px
    width 100%
    z-index 50
    .content
      display: flex
      background #141d27
      .content-left
        flex: 1
        font-size 0
        .logoWrapper
          display inline-block
          position relative
          top -10px
          margin 0 12px
          padding 6px
          width 56px
          height 56px
          box-sizing border-box
          vertical-align top
          border-radius 50%
          background #141d27
          .logo
            width 100%
            height 100%
            border-radius 50%
            text-align center
            background #2b343c
            &.highlight
              background rgb(0, 160, 220)
            .icon-shopping_cart
              line-height 48px
              font-size 24px
              color #80858a
              &.highlight
                color rgb(255, 255, 255)
          .num
            position absolute
            top 0
            right 0px
            width 24px
            height 16px
            line-height 16px
            text-align center
            border-radius 16px
            font-size 9px
            font-weight 700
            color #fff
            background rgb(240, 20, 20)
            box-shadow 0 4px 8px 0 rgba(0, 0, 0, 0.4)
        .price
          display inline-block
          vertical-align top
          margin-top 12px
          line-height 24px
          padding-right 12px
          box-sizing border-box
          border-right 1px solid rgba(255, 255, 255, 0.1)
          font-size 16px
          font-weight 700px
          color rgba(255, 255, 255, 0.4)
          &.highlight
            color rgb(255, 255, 255)
        .desc
          display inline-block
          vertical-align top
          line-height 24px
          margin 12px 0 0 12px
          font-size 16px
          font-weight 700px
          color rgba(255, 255, 255, 0.4)
      .content-right
        flex: 0 0 105px
        width: 105px
        height: 46px
        .pay
          height 48px
          line-height 48px
          text-align center
          font-size 12px
          font-weight 700
          color #80858a
          &.not-enough
            background: #2b343c
          &.enough
            background: #00b43c
            color: #fff
    .ballcontainer
      .ball
        position fixed
        left 32px
        bottom 12px
        z-index 200
        &.drop-transition
          transition all 0.4s cubic-bezier(0.5, -0.29, 0.75, 0.4)
          .inner
            width 16px
            height 16px
            border-radius 50%
            background rgb(0, 160, 220)
            transition all 0.4s linear
    .shopcart-list
      position absolute
      left 0
      top 0
      z-index -10
      width 100%
      background blue
      transition all 0.5s ease
      &.fold-transition
        transition all 0.5s ease
        transform translate3d(0, -100%, 0)
      &.fold-enter, &.fold-leave
        transform translate3d(0, 0, 0)
      .list-header
        height 40px
        line-height 40px
        padding 0 18px
        background #f3f5f7
        border-bottom 1px solid rgba(7, 17, 27, 0.1)
        .title
          float left
          font-size 14px
          color rgb(7, 17, 27)
        .empty
          float right
          font-size 12px
          color rgb(0, 160, 220)
      .list-content
        padding 0 18px
        max-height 217px
        background #ffffff
        overflow hidden
        .food-item
          position relative
          padding 12px 0
          box-sizing border-box
          border-1px(rgba(7, 17, 27, 0.1))
          .name
            line-height 24px
            font-size 14px
            color rgb(7, 17, 27)
          .price
            position absolute
            right 90px
            bottom 12px
            line-height 24px
            font-size 14px
            font-weight 700
            color rgb(240, 20, 20)
          .cartCtrl-wrapper
            position absolute
            right 0
            bottom 6px
  .list-mask
    position fixed
    top 0
    left 0
    width 100%
    height 100%
    z-index 40
    backdrop-filter blur(10px)
    &.fade-transition
      transition all 0.5s
      opacity 1
      background rgba(7, 17, 27, 0.6)
    &.fade-enter, &.fade-leave
      opacity 0
</style>

