<template>
  <div class="goods">
    <div class="menu-wrapper" v-el:menu-wrapper>
      <ul>
        <li v-for="item in goods" track-by="$index" class="menu-item border-1px"
            :class="{'current':currentIndex===$index}"
            @click="selectMenu($index,$event)">
          <span class="text">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" v-el:foods-wrapper>
      <ul>
        <li v-for="item in goods" class="goods-list food-list-hook">
          <div class="title">
            <h1>{{item.name}}</h1>
          </div>
          <ul>
            <li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item">
              <div class="icon">
                <img width="57px" height="57px" :src="food.icon">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="description">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span>
                  <span>好评率：{{food.rating}}</span>
                </div>
                <div class="price">
                  <span>¥{{food.price}}</span>
                  <span v-show="food.oldPrice" class="old-price">¥{{food.oldPrice}}</span>
                </div>
                <div class="cartCtrl-wrapper">
                  <cartctrl :food="food"></cartctrl>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart v-ref:shopcart :select-foods="selectFoods" :delivery-price="seller.deliveryPrice"
              :min-price="seller.minPrice"></shopcart>
  </div>
  <food :food="selectedFood" v-ref:food></food>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import shopcart from 'components/shopcart/shopcart.vue';
  import cartCtrl from 'components/cartCtrl/cartCtrl.vue';
  import food from 'components/food/food.vue';

  const ERR_OK = 0;

  export default {
    //vueRouter
    props: {
      seller: {
        type: Object
      }
    },
    //把goods放在data里面，以后个能会操作
    data() {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      };
    },
    //vueRousource
    created() {
      this.classMap = ['decrease', 'discount', 'guarantee', 'invoice', 'special'];
      this.$http.get('api/goods').then((res) => {
        res = res.body;
        if (res.errno === ERR_OK) {
          this.goods = res.data;
          this.$nextTick(() => {
            this._initScroll();
            this._caculateHeight();
          });
        }
      });
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          //let offset=height2-height1;
          if (this.scrollY >= height1 && this.scrollY < height2) {
            return i;
          }
        }
        //如果找不到就返回index为0
        return 0;
      },
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        return foods;
      }

    },
    methods: {
      _drop(target) {
        // 体验优化，异步执行下落动画
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target);
        });
      },
      selectMenu(index, event) {
        // better-Scroll自带事件属性，这里用来过滤浏览器事件
        if (!event._constructed) {
          return;
        }
        let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index];

        // better-scroll的滚动接口
        this.foodsScroll.scrollToElement(el, 200);
      },
      selectFood(food, event) {
        if (!event._constructed) {
          return;
        }
        this.selectedFood = food;
        this.$refs.food.show();
      },
      _initScroll() {
        this.menuScroll = new BScroll(this.$els.menuWrapper, {
          click: true
        });

        this.foodsScroll = new BScroll(this.$els.foodsWrapper, {
          click: true,
          probeType: 3
        });

        this.foodsScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y));
        });
      },
      _caculateHeight() {
        let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook');
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let itemHeight = foodList[i].clientHeight;
          height += itemHeight;
          this.listHeight.push(height);
        }
      }
    },
    events: {
      'cart.add'(target) {
        this._drop(target);
      }
    },
    components: {
      'shopcart': shopcart,
      'cartctrl': cartCtrl,
      'food': food
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .goods
    display: flex
    position: absolute
    width: 100%
    top: 174px
    bottom: 46px
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px
      background: #f3f5f7
      .menu-item
        display: table
        height: 54px
        width: 56px
        line-height: 14px
        padding: 0 12px
        font-size: 0px
        border-1px(rgba(7, 17, 27, 0.1))
        &:last-child
          border-none()
        &.current
          background: white
          border-left: 5px solid #1E89E0
        &.current .text
          border-none()
        .icon
          display: inline-block
          vertical-align: top
          height: 12px
          width: 12px
          margin-right: 4px
          background-size: 12px 12px
          background-repeat: no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          display: table-cell
          width: 56px
          vertical-align: middle
          font-size: 12px
    .foods-wrapper
      flex: 1
      .goods-list
        .title
          border-left: 2px solid #1E89E0
          padding-left: 14px
          font-size: 12px
          line-height: 26px
          color: rgb(147, 153, 159)
          background: #f3f5f7
        .food-item
          display: flex
          margin: 18px
          padding-bottom: 18px
          border-1px(rgba(7, 17, 27, 0.1))
          &:last-child
            margin-bottom: 0
            border-none()
          .icon
            display: inline-block
            flex: 0 0 57px
          .content
            flex: 1
            margin-left: 10px
            display: inline-block
            .name
              margin: 2px auto 8px auto
              font-size: 14px
              line-height: 14px
              color: rgb(7, 17, 27)
            .description
              margin-bottom: 8px
              font-size: 10px
              line-height: 14px
              color: rgb(147, 153, 159)
          .extra
            margin-bottom: 8px
            font-size: 10px
            line-height: 12px
            color: rgb(7, 17, 27)
            .count
              margin-right: 12px
          .price
            font-size: 14px
            line-height: 14px
            font-weight: 700
            color: rgb(240, 20, 20)
            .old-price
              text-decoration: line-through
              color: rgb(147, 153, 159)
          .cartCtrl-wrapper
            position absolute
            right 0
            bottom 12px
</style>
