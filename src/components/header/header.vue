<template>
  <div class="header">
    <div class="content-wrapper">
      <div class="avatar">
        <img height="64px" width="64px" :src="seller.avatar">
      </div>
      <div class="content">
        <div class="title">
          <span class="brand">

          </span>
          <div class="name">
            {{seller.name}}
          </div>
        </div>
        <div class="description">
          {{seller.description}}/{{seller.deliveryTime}}分钟送达
        </div>
        <div class="supports" v-if="seller.supports">
          <span class="icon" :class="classMap[seller.supports[0].type]"></span>
          <div class="text">{{seller.supports[0].description}}</div>
        </div>
      </div>
      <div v-if="seller.supports" class="supports-count" @click="showDetail">
        <span class="count">{{seller.supports.length}}个</span>
        <span class="icon-keyboard_arrow_right"></span>
      </div>
    </div>
    <div class="bulletin-wrapper" @click="showDetail">
      <span class="bulletin-title"></span><span class="bulletin-text">{{seller.bulletin}}</span>
      <i class="icon-keyboard_arrow_right"></i>
    </div>
    <div class="background">
      <img :src="seller.avatar" height="100%" width="100%">
    </div>
    <div v-show="detailShow" class="detail" transition="fade">
      <div class="detail-wrapper clearfix">
        <div class="detail-main">
          <h1 class="name">{{seller.name}}</h1>
          <div class="star-wrapper">
            <star :size="48" :score="seller.score"></star>
          </div>
          <div class="title">
            <div class="line"></div>
            <div class="text">优惠信息</div>
            <div class="line"></div>
          </div>
          <ul v-link="seller.supports" class="supports">
            <li v-for="item in seller.supports" class="supports-item">
              <span class="icon" :class="classMap[seller.supports[$index].type]"></span>
              <span class="text">{{seller.supports[$index].description}}</span>
            </li>
          </ul>
          <div class="title">
            <div class="line"></div>
            <div class="text">商家公告</div>
            <div class="line"></div>
          </div>
          <div class="bulletin">
            <span class="text">{{seller.bulletin}}</span>
          </div>
        </div>
      </div>
      <div class="detail-close" @click="closeDetail">
        <i class="icon-close"></i>
      </div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import star from 'components/star/star.vue';

  export default{
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        detailShow: false
      };
    },
    methods: {
      showDetail() {
        this.detailShow = true;
      },
      closeDetail() {
        this.detailShow = false;
      }
    },
    created() {
      this.classMap = ['decrease', 'discount', 'guarantee', 'invoice', 'special'];
    },
    components: {
      'star': star
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .header
    position: relative
    background-color: rgba(7, 17, 27, 0.5)
    overflow: hidden
    color: #fff
    .content-wrapper
      position: relative
      padding: 24px 16px 18px 24px
      font-size: 0
      .avatar
        display: inline-block
        vertical-align: top
        margin-right: 16px
        img
          border-radius: 2px
      .content
        display: inline-block
        vertical-align: top
        .title
          margin: 2px 0 8px 0
          font-size: 16px
          .brand
            display: inline-block
            vertical-align: top
            height: 18px
            width: 30px
            bg-image('brand')
            background-size: 30px 18px
            background-repeat: no-repeat
          .name
            display: inline-block
            vertical-align: top
            font-weight: bold
            line-height: 18px
        .description
          padding-bottom: 10px
          font-size: 12px
        .supports
          .icon
            display: inline-block
            vertical-align: top
            height: 12px
            width: 12px
            margin-right: 4px
            background-size: 12px 12px
            background-repeat: no-repeat
            &.decrease
              bg-image('decrease_1')
            &.discount
              bg-image('discount_1')
            &.guarantee
              bg-image('guarantee_1')
            &.invoice
              bg-image('invoice_1')
            &.special
              bg-image('special_1')
          .text
            display: inline-block
            vertical-align: top
            line-height: 12px
            margin-top: 1px
            font-size: 10px
      .supports-count
        position: absolute
        right: 12px
        bottom: 14px
        padding: 0 8px
        height: 24px
        line-height: 24px
        border-radius: 14px
        background-color: rgba(0, 0, 0, 0.2)
        text-align: center
        .count
          vertical-align: top
          font-size: 10px
        .icon-keyboard_arrow_right
          margin-left: 2px
          font-size: 10px
          line-height: 24px
    .bulletin-wrapper
      position: relative
      height: 28px
      line-height: 18px
      padding: 0 22px 0 12px
      background-color: rgba(7, 17, 27, 0.2)
      white-space: nowrap
      overflow: hidden
      text-overflow: ellipsis
      .bulletin-title
        display: inline-block
        vertical-align: top
        height: 12px
        width: 22px
        margin-top: 6px
        bg-image('bulletin')
        background-size: 22px 12px
        background-repeat: no-repeat
      .bulletin-text
        vertical-align: top
        margin: 0 21px 0 4px
        line-height: 28px
        font-size: 10px
      .icon-keyboard_arrow_right
        position: absolute
        right: 12px
        line-height: 28px
        font-size: 10px
    .background
      position: absolute
      left: 0
      top: 0
      width: 100%
      z-index: -1
      filter: blur(10px)
    .detail
      position: fixed
      left: 0
      top: 0
      height: 100%
      width: 100%
      z-index: 100
      overflow: auto
      background-color: rgba(7, 17, 27, 0.8)
      backdrop-filter: blur(10px)
      transition: all 0.5s
      &.fade-transiion
        opacity: 1
        background: rgba(7, 17, 27, 0.8)
      &.fade-enter, &.fade-leave
        opacity: 0
        background: rgba(7, 17, 27, 0)
      .detail-wrapper
        width: 100%
        min-height: 100%
        .detail-main
          margin-top: 64px
          padding-bottom: 64px
          .name
            line-height: 32px
            font-size: 16px
            font-weight: 700
            text-align: center
          .star-wrapper
            margin: 12px auto 28px auto
            text-align: center
          .title
            display: flex
            width: 80%
            margin: 28px auto 24px auto
            .line
              flex: 1
              position: relative
              top: -6px
              border-bottom: 1px solid rgba(255, 255, 255.0 .2)
            .text
              padding: 0 12px
              font-size: 14px
              font-weight: 700
          .supports
            width: 80%
            margin: 0 auto
            .supports-item
              padding: 0 12px
              margin-bottom: 12px
              font-size: 0
              &:last-child
                margin-bottom: 0
              .icon
                display: inline-block
                vertical-align: top
                margin-right: 6px
                height: 16px
                width: 16px
                background-size: 16px 16px
                background-repeat: no-repeat
                &.decrease
                  bg-image('decrease_2')
                &.discount
                  bg-image('discount_2')
                &.guarantee
                  bg-image('guarantee_2')
                &.invoice
                  bg-image('invoice_2')
                &.special
                  bg-image('special_2')
              .text
                font-size: 16px
                color: rgb(255, 255, 255)
                line-height 18px
          .bulletin
            width: 80%
            margin: 24px auto 24px auto
            .text
              font-size: 16px
              line-height: 32px
      .detail-close
        position: relative
        height: 32px
        width: 32px
        margin: -64px auto 0 auto
        clear: both
        font-size: 32px
</style>
