<template>
	<view class="market-container plr40">
		<view class="tabs flex-between alcenter mb20">
<!-- 			<view class="tab-item flex-one flex alcenter">
				<view class="first flex-between alcenter plr20">
					<view>BTC</view>
					<image src="../../static/image/triangle.png" mode="" class="triangle-img"></image>
				</view>
			</view> -->
			<view class="tab-item flex-one flex-center">
				<view>资产</view>
<!-- 				<image class="ml10" src="../../static/image/shangxia@2x.png" mode="" @click="handleSort('price')"></image>
 -->			</view>
			<view class="tab-item flex-one flex-center">
				<view>最新价</view>
				<image class="ml10" src="../../static/image/shangxia@2x.png" mode="" @click="handleSort('price')"></image>
			</view>
			<view class="tab-item flex-one flex-center">
				<view>涨幅度</view>
				<image class="ml10" src="../../static/image/shangxia@2x.png" mode="" @click="handleSort('trend')"></image>
			</view>
		</view>
		<view class="list">
			<view class="list-item h80 mb40 flex-between alcenter" v-for="item in gds_lst" :key="item.id">
				<view class="data-item flex-one flex alcenter">
					<image src="http://193.203.215.185:8080/media/wallet/2022/10/03/BTC2x.png" mode="" class="mr40"></image>
					<view>
						<view class="ft36">{{ item.symbol }}</view>
						<view class="c_9397AF ft24">{{ item.chain }}</view>
					</view>
				</view>
				<view class="data-item price flex-one flex-column flex alcenter">
					<view class="ft36">{{ item.usd_price }}</view>
					<view class="c_9397AF ft24">¥{{ item.cny_price }}</view>
				</view>
				<view class="data-item flex-one flex-center rate">
					<view class="rate flex-center c-white ft24 alcenter" :class="{'up': +item.rate >= 0 }">{{ `${(+item.margin >= 0) ? '+' : ''}${ (+item.margin) }` }}%</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import { getMarketPrice } from '@/api/market.js'
	import config from '@/config.js'
  import {getUUID} from "../../common/uuid";
	export default {
		data() {
			return {
				config,
				newRriceSort: false,
				trendSort: false,
				gds_lst: [],
				pageParams: {
					device_id: 0,
					exchange_id: 1,
					page: 1,
					pageSize: 1000,
					total: null
				},
				timer: null,
        uuid: null,
			};
		},
		onShow () {
			this.getMarketPrice()
		},
		onHide () {
			// clearInterval(this.timer)
			// this.timer = null
		},
    created(){
      this.initSocket()
      // window.addEventListener('beforeunload', this.destroySocket)
    },
    beforeDestroy(){
      this.destroySocket()
    },
		methods: {
      destroySocket() {
        let _this = this;
        if (_this.socket) {
          _this.socket.send(JSON.stringify({
            method: 'un_register',
            uuid: _this.uuid,
          }));
        }
      },
      initSocket () {
        this.uuid = getUUID()
        function decode(msg) {
          if (msg && msg.data) {
            return JSON.parse(msg.data)
          }
          return null
        }

        const _this = this
        const socketUrl = this.config.socketBaseUrl + '/ws/market/';
        console.log('start connect socket')
        try {
          this.socket = new WebSocket(socketUrl);
        } catch (e) {
          console.log(e)
          return
        }
        this.socket.onopen = function() {
          console.log('socket init success')
          _this.socket.send(JSON.stringify({
            method: 'register',
            uuid: _this.uuid,
            exchange_id: 1,
            device_id: 0
          }));
        };
        this.socket.onmessage = function(msg) {
          const result = decode(msg)
          if (result.code === 0 ) {
            console.log('data: ' + JSON.stringify(result.data))
            _this.gds_lst = result.data
          }
        };
        this.socket.onerror = function(err) {
          console.log('socket onerror' + JSON.stringify(err));
        };
        this.socket.onclose = function() {
          console.log('socket close');
        }
      },
			async getMarketPrice () {
				try {
					const res = await getMarketPrice(this.pageParams)
					console.log(res)
					if (+res.code === 200) {
						this.gds_lst = res.result || []
						this.pageParams.total = +res.data.total
					}
				} catch (e) {
					console.log(e)
				}
			},
			handleSort (type) {
				switch (type) {
					case 'price': // 按最新价排序
						this.newRriceSort = !this.newRriceSort
						this.gds_lst.sort((a, b) => {
							if (this.newRriceSort) return b.cny_price - a.cny_price // 降序
							return a.cny_price - b.cny_price
						})
						break
					case 'trend': // 按涨跌幅排序
						this.trendSort = !this.trendSort
						this.gds_lst.sort((a, b) => {
							if (this.trendSort) return b.rate - a.rate // 降序
							return a.rate - b.rate
						})
						break
					default:
						console.log('type错误：', type)
						break
				}
			}
		}
	}
</script>

<style lang="scss">
	.market-container{
		.tabs{
			position: sticky;
			top: 0;
			z-index: 10;
			background-color: #ffffff;
			padding: 20rpx 0;
			.tab-item{
				font-size: 32rpx;
				.first{
					width: 196rpx;
					height: 52rpx;
					color: #4C6EF5;
					background: #F1F3F5;
					border-radius: 30rpx;
					box-sizing: border-box;
				}
				image{
					width: 24rpx;
					height: 24rpx;
				}
				.triangle-img{
					width: 44rpx;
					height: 44rpx;
				}
			}
		}
		.list{
			.list-item{
				.data-item{
					image{
						width: 80rpx;
						height: 80rpx;
						border-radius: 20rpx;
					}
					.rate{
						width: 130rpx;
						height: 50rpx;
						background-image: url(../../static/image/hong@2x.png);
						background-size: 100% 100%;
						background-repeat: no-repeat;
						&.up{
							background-image: url(../../static/image/lv@2x.png);
						}
					}
				}
			}
		}
	}
</style>
