<!-- order.vue承载了支付订单相关页面的头部尾部，
     <router-view></router-view>嵌入其他页面的核心代码 -->
<template>
  <div>
    <router-view></router-view>
    <service-bar></service-bar>
    <nav-footer></nav-footer>
  </div>
</template>
<script>
// 加载组件
import ServiceBar from '@/components/ServiceBar'
import NavFooter from '@/components/NavFooter'
export default {
  name: 'order',
  components: {
    ServiceBar,
    NavFooter
  }
  // data() {
  //   return {
  //     title: '',
  //     tip: ''
  //   }
  // }
  // mounted() {
  //   // order-header组件，根据路由地址，动态改变title和tip
  //   // 获取当前页面路由地址
  //   let path = this.$route.path
  //   if (path == '/order/confirm') {
  //     this.title = '订单确认'
  //     this.tip = '请仔细核对收货地址'
  //   } else if (path == '/order/list') {
  //     this.title = '订单列表'
  //     this.tip = '请谨防钓鱼链接或诈骗电话，了解更多>'
  //   } else if (path == '/order/pay') {
  //     this.title = '订单支付'
  //     this.tip = '请谨防钓鱼链接或诈骗电话，了解更多>'
  //   } else {
  //     this.title = '订单支付'
  //     this.tip = '请谨防钓鱼链接或诈骗电话，了解更多>'
  //   }
  // }
}
</script>
