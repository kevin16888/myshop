<template>
  <div>
    <!-- 收货人信息 -->
    <div class="cart-top">
      <div class="receive">
        <div class="name">收货人: {{address.userName}}</div>
        <div class="phonen-number">{{address.telNumber}}</div>
      </div>
      <div class="address">收货地址: {{joinAddress}}</div>
      <img src="../../../static/images/cart_border@2x.png" class="address-bar" mode="aspectFill">
    </div>
    <!-- 订单商品列表 -->
    <div class="list-title">商品列表</div>
    <div class="ware-list">
      <div class="ware-item" v-for="(item, key) in goodsData" :key="key">
        <!-- 左侧的图片 -->
        <navigator class="ware-image" url="#">
          <img :src="item.goods_small_logo" mode="aspectFill">
        </navigator>
        <!-- 右边的商品信息 -->
        <div class="ware-info">
          <navigator url="#" class="ware-title">{{item.goods_name}}</navigator>
          <div class="ware-info-btm">
            <div class="ware-price">
              <span>￥</span>{{item.goods_price}}
            </div>
            <div class="number">x {{item.num}}</div>
          </div>
        </div>
      </div>
    </div>
    <!-- 总价和支付 -->
    <div class="all-price">
      合计：<span>￥{{allPrice}}</span>
      <button type="primary" @click="payHandle">支付</button>
    </div>
  </div>
</template>

<script>
import request from '../../utils/request.js'
export default {
  data () {
    return {
      // 地址
      address: null,
      // 购物车信息
      cart: null,
      // 订单号
      orderNum: null
    }
  },
  computed: {
    allPrice () {
      // 计算选中商品的总价
      let sum = 0
      for (let key in this.cart) {
        let p = this.cart[key]
        if (p.checked) {
          // 选中的商品
          sum += p.goods_price * p.num
        }
      }
      return sum
    },
    goodsData () {
      // 从购物车中取出商品放到数组中
      let products = []
      for (let key in this.cart) {
        let p = this.cart[key]
        if (p.checked) {
          // 选中的商品
          products.push(p)
        }
      }
      return products
    },
    joinAddress () {
      // 拼接一个完成的地址
      let { provinceName, cityName, countyName, detailInfo } = this.address
      return `${provinceName}${cityName}${countyName}${detailInfo}`
    }
  },
  methods: {
    // 在mpvue的插值表达式中，不可以直接使用函数调用
    payHandle () {
      // 完成支付动作
      let that = this
      let token = mpvue.getStorageSync('mytoken')
      request(
        'my/orders/req_unifiedorder',
        'post',
        {
          order_number: this.orderNum
        },
        { Authorization: token }
      ).then(res => {
        // 这里返回的结果用于进行微信支付
        // console.log(res)
        let { message } = res.data
        wx.requestPayment({
          // 把pay对象中所有属性拆开放到这个位置
          ...message.pay,
          success (res) {
            // 清空已经支付的商品信息
            // 获取所有选中的商品并且全部删除
            let indexs = []
            for (let key in that.cart) {
              let p = that.cart[key]
              if (p.checked) {
                // 选中的商品
                indexs.push(p.goods_id)
              }
            }
            indexs.forEach(item => {
              // item就是商品id，根据商品id删除对象中的属性和值
              delete that.cart[item]
            })
            // 然后同步购物车最新数据
            mpvue.setStorageSync('mycart', that.cart)
            // 成功之后，跳回到购物车，还应清空已经支付的商品信息
            mpvue.navigateBack({
              delta: 1
            })
          }
        })
      })
    }
  },
  onLoad (param) {
    // 获取订单号
    // console.log(param)
    this.orderNum = param.order_num
  },
  onShow () {
    this.address = mpvue.getStorageSync('myAddress')
    // console.log(this.address)
    this.cart = mpvue.getStorageSync('mycart')
  }
}
</script>

<style scoped lang='scss'>
@import 'main.scss';
</style>
