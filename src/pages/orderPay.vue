<template>
  <div class="order-pay">
    <order-header title="订单支付">
      <template v-slot:tip>
        <span>请谨防钓鱼链接或诈骗电话，了解更多></span>
      </template>
    </order-header>
    <div class="wrapper">
      <div class="container">
        <div class="order-wrap">
          <div class="item-order">
            <div class="icon-succ"></div>
            <div class="order-info">
              <h2>订单提交成功！去付款咯～</h2>
              <p>请在<span>30分</span>内完成支付, 超时后将取消订单</p>
              <p>收货信息：{{userInfo}}</p>
            </div>
            <div class="order-total">
              <p>应付总额：<span>{{payment}}</span>元</p>
              <p>订单详情<em class="icon-down" :class="{'up':showDetail}" @click="showDetail=!showDetail"></em></p>
            </div>
          </div>
          <div class="item-detail" v-show="showDetail">
            <div class="item">
              <div class="detail-title">订单号：</div>
              <div class="detail-info theme-color">{{orderNo}}</div>
            </div>
            <div class="item">
              <div class="detail-title">收货信息：</div>
              <div class="detail-info">{{userInfo}}</div>
            </div>
            <div class="item good">
              <div class="detail-title">商品名称：</div>
              <div class="detail-info">
                <ul>
                  <li v-for="(item,index) in orderItemVoList" :key="index">
                    <img v-lazy="item.productImage" />{{item.productName}}
                  </li>
                </ul>
              </div>
            </div>
            <div class="item">
              <div class="detail-title">发票信息：</div>
              <div class="detail-info">电子发票 个人</div>
            </div>
          </div>
        </div>
        <div class="item-pay">
          <h3>选择以下支付方式付款</h3>
          <div class="pay-way">
            <p>支付平台</p>
            <div class="pay pay-ali" :class="{'checked':payType==1}" @click="paySubmit(1)"></div>
            <div class="pay pay-wechat" :class="{'checked':payType==2}" @click="paySubmit(2)"></div>
          </div>
        </div>
      </div>
    </div>
    <scan-pay-code v-if="showPay" :img="payImg" @close="closePay"></scan-pay-code>
    <modal title="支付确认" btnType="3" confirmText="查看订单" cancelText="未支付" :showModal="showModal" @submit="goOrderList" @cancel="showModal=false">
      <template v-slot:body>
        <p>您是否已完成支付?</p>
      </template>
    </modal>
  </div>
</template>
<script>
import QRCode from 'qrcode'
import OrderHeader from '@/components/OrderHeader'
import ScanPayCode from '@/components/ScanPayCode'
import Modal from '@/components/Modal'

export default {
  name: 'order-pay',
  components: { OrderHeader, ScanPayCode, Modal },
  data() {
    return {
      orderNo: this.$route.query.orderNo,
      userInfo: '', //收货人信息
      orderItemVoList: [], //订单项列表
      payment: 0, //订单总金额
      showDetail: false, //是否显示订单详情
      payType: '', //支付类型，1支付宝，2微信
      showPay: false, //是否显示微信二维码弹框
      payImg: '', //生成的二维码图片
      showModal: false, //是否展示'确认支付'弹框
      T: '' //定时器id
    }
  },
  created() {
    this.getOrderDetail()
  },
  methods: {
    getOrderDetail() {
      this.axios.get(`/orders/${this.orderNo}`).then(res => {
        let item = res.shippingVo
        this.userInfo = `${item.receiverName} ${item.receiverMobile} ${item.receiverProvince} ${item.receiverCity} ${item.receiverDistrict} ${item.receiverAddress}`
        this.orderItemVoList = res.orderItemVoList
        this.payment = res.payment
      })
    },
    paySubmit(payType) {
      this.payType = payType //用于选中高亮显示
      if (payType == 1) {
        // 支付宝支付,打开alipay中间页
        window.open('/#/order/alipay?orderId=' + this.orderNo, '_blank')
      } else {
        // 微信支付, 此时已拿到订单总金额payment, 不需要async/await异步处理了
        this.axios
          .post('/pay', {
            orderId: this.orderNo,
            orderName: 'phonemall支付订单', //扫码支付时订单名称
            amount: this.payment, //订单总金额
            payType: 2 //1支付宝，2微信
          })
          .then(res => {
            // 把返回的content,通过qrcode插件转换成url地址
            QRCode.toDataURL(res.content)
              .then(url => {
                // 弹出二维码弹框
                this.showPay = true
                this.payImg = url
                this.loopOrderState() //轮询订单支付状态
              })
              .catch(() => {
                this.$message.error('微信二维码生成失败,请稍后重试')
              })
          })
      }
    },
    closePay() {
      // 关闭二维码弹框,打开'确认支付'弹框,清理轮询定时器
      this.showPay = false
      this.showModal = true
      clearInterval(this.T)
    },
    loopOrderState() {
      // 轮询当前订单的支付状态,每1s就拉取一次订单状态
      // setInterval重复调用,setTimeout只调用一次
      this.T = setInterval(() => {
        this.axios.get(`/orders/${this.orderNo}`).then(res => {
          if (res.status == 20) {
            // status为20时,表示已支付,就自动跳转到订单列表
            clearInterval(this.T)
            this.goOrderList()
          }
        })
      }, 1000)
    },
    goOrderList() {
      // 用户点击查看订单按钮
      this.$router.push('/order/list')
    }
  }
}
</script>
<style lang="scss">
.order-pay {
  .wrapper {
    background-color: #f5f5f5;
    padding-top: 30px;
    padding-bottom: 61px;
    .order-wrap {
      padding: 62px 50px;
      background-color: #fff;
      font-size: 14px;
      margin-bottom: 30px;
      .item-order {
        display: flex;
        align-items: center;
        .icon-succ {
          width: 90px;
          height: 90px;
          border-radius: 50%;
          background: url('/public/imgs/icon-gou.png') #80c58a no-repeat center;
          background-size: 60px;
          margin-right: 40px;
        }
        .order-info {
          margin-right: 248px;
          h2 {
            font-size: 24px;
            color: #333333;
            margin-bottom: 20px;
          }
          p {
            color: #666666;
            span {
              color: #ff6700;
            }
          }
        }
        .order-total {
          & > p:first-child {
            margin-bottom: 30px;
          }
          span {
            font-size: 28px;
            color: #ff6700;
          }
          .icon-down {
            display: inline-block;
            width: 14px;
            height: 10px;
            background: url('/public/imgs/icon-down.png') no-repeat center;
            background-size: contain;
            margin-left: 9px;
            transition: all 0.5s;
            cursor: pointer;
            &.up {
              transform: rotate(180deg);
            }
          }
        }
      }
      .item-detail {
        border-top: 1px solid #d7d7d7;
        padding-top: 47px;
        padding-left: 130px;
        font-size: 14px;
        margin-top: 45px;
        .item {
          margin-bottom: 19px;
          .detail-title {
            float: left;
            width: 100px;
          }
          .detail-info {
            display: inline-block;
            img {
              width: 30px;
              vertical-align: middle;
            }
          }
        }
      }
    }
    .item-pay {
      padding: 26px 50px 72px;
      background-color: #ffffff;
      color: #333333;
      h3 {
        font-size: 20px;
        font-weight: 200;
        color: #333333;
        padding-bottom: 24px;
        border-bottom: 1px solid #d7d7d7;
        margin-bottom: 26px;
      }
      .pay-way {
        font-size: 18px;
        .pay {
          display: inline-block;
          width: 188px;
          height: 64px;
          border: 1px solid #d7d7d7;
          cursor: pointer;
          &:last-child {
            margin-left: 20px;
          }
          &.checked {
            border: 1px solid #ff6700;
          }
        }
        .pay-ali {
          background: url('/public/imgs/pay/icon-ali.png') no-repeat center;
          background-size: 103px 38px;
          margin-top: 19px;
        }
        .pay-wechat {
          background: url('/public/imgs/pay/icon-wechat.png') no-repeat center;
          background-size: 103px 38px;
        }
      }
    }
  }
}
</style>
