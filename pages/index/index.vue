<template>
	<view class="content">
		<view>实时行情数据(美元$)</view>
		<view>全部展示<switch @change="showAll"></switch>
		</view>
		<view>红跌绿涨<switch @change="switchColor"></switch>
		</view>
		<view v-for="(value, key, index) in prices" :key="key" class="price_box">
			<view style="display: flex;">
				<view v-bind:class="value.isup24h ? 'isup1' : 'isdown1'">
					{{value.rate}}%
				</view>
				<view v-bind:class="value.isup ? 'isup' : 'isdown'">
					{{ key }} : {{ value.p }}
				</view>
				<view class="button_k">
					<button @click="showcandle(key)">k线</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				prices: {
					"bitcoin": {
						"p": "",
						"isup": true,
						"isup24h": true,
						"rate": ""
					},
					"ethereum": {
						"p": "",
						"isup": true,
						"isup24h": true,
						"rate": ""
					},
					"tether": {
						"p": "",
						"isup": true,
						"isup24h": true,
						"rate": ""
					},
					"ripple": {
						"p": "",
						"isup": true,
						"isup24h": true,
						"rate": ""
					},
					"bitcoin-cash": {
						"p": "",
						"isup": true,
						"isup24h": true,
						"rate": ""
					},
					"bitcoin-sv": {
						"p": "",
						"isup": true,
						"isup24h": true,
						"rate": ""
					},
					"litecoin": {
						"p": "",
						"isup": true,
						"isup24h": true,
						"rate": ""
					},
					"binance-coin": {
						"p": "",
						"isup": true,
						"isup24h": true,
						"rate": ""
					},
					"eos": {
						"p": "",
						"isup": true,
						"isup24h": true,
						"rate": ""
					},
					"cardano": {
						"p": "",
						"isup": true,
						"isup24h": true,
						"rate": ""
					}
				},
				pricein24h: {},
				showall: false,
				hongDown: false
			}
		},
		onLoad() {
			this.load24hData()
			this.submsg();

		},
		onShow(){
			this.submsg()
		},
		onHide() {
			uni.closeSocket({});
		},
		methods: {
			showcandle(e) {
				console.debug(e);
				uni.navigateTo({
					url: '../charts/showCandle?coin=' + e
				});
			},
			switchColor: function(e) {
				// console.log('switch1 发生 change 事件，携带值为', e.target.value)
				this.hongDown = e.target.value;
				this.submsg();
				console.log(this.showall)
			},
			showAll: function(e) {
				// console.log('switch1 发生 change 事件，携带值为', e.target.value)
				this.showall = e.target.value;
				this.submsg();
				this.load24hData();
				console.log(this.showall)
			},
			submsg: function() {
				var top10_url =
					"wss://ws.coincap.io/prices?assets=bitcoin,ethereum,tether,ripple,bitcoin-cash,bitcoin-sv,,litecoin,binance-coin,eos,cardano";
				var all_url = "wss://ws.coincap.io/prices?assets=ALL";
				uni.closeSocket({});
				uni.connectSocket({
					url: this.showall ? all_url : top10_url
				});
				var _this = this;
				uni.onSocketMessage(function(res) {
					var obj = JSON.parse(res.data)
					for (let key in obj) {
						if (_this.hongDown) {
							if (_this.prices[key] == undefined) {
								var new_one = new Object();
								new_one["p"] = obj[key];
								new_one["isup"] = false;
								new_one["isup24h"] = obj[key] < _this.pricein24h[key];
								new_one["rate"] = (((obj[key] - _this.pricein24h[key]) / _this.pricein24h[key]) * 100).toFixed(2);
								_this.prices[key] = new_one;
							} else {
								_this.prices[key]["isup"] = obj[key] < _this.prices[key]["p"] ? false : true;
								_this.prices[key]["p"] = obj[key];
								_this.prices[key]["isup24h"] = obj[key] < _this.pricein24h[key];
								_this.prices[key]["rate"] = (((obj[key] - _this.pricein24h[key]) / _this.pricein24h[key]) * 100).toFixed(2);
							}
						} else {
							if (_this.prices[key] == undefined) {
								var new_one = new Object();
								new_one["p"] = obj[key];
								new_one["isup"] = true;
								new_one["isup24h"] = obj[key] > _this.pricein24h[key];
								new_one["rate"] = (((obj[key] - _this.pricein24h[key]) / _this.pricein24h[key]) * 100).toFixed(2);
								_this.prices[key] = new_one;
							} else {
								_this.prices[key]["isup"] = obj[key] > _this.prices[key]["p"] ? true : false;
								_this.prices[key]["p"] = obj[key];
								_this.prices[key]["isup24h"] = obj[key] > _this.pricein24h[key];
								_this.prices[key]["rate"] = (((obj[key] - _this.pricein24h[key]) / _this.pricein24h[key]) * 100).toFixed(2);
							}
						}

					}
				});
			},
			load24hData: function() {
				//加载最近24小时的价格数据
				var all_url = "https://api.coincap.io/v2/assets";
				var top10_url =
					'https://api.coincap.io/v2/assets?ids=bitcoin,ethereum,tether,ripple,bitcoin-cash,bitcoin-sv,litecoin,binance-coin,eos,cardano';
				var _this = this;
				uni.request({
					url: this.showall ? all_url : top10_url,
					success: (res) => {
						res.data.data.forEach(function(item) {
							_this.pricein24h[item.id] = item.vwap24Hr;
						})
						console.debug(_this.pricein24h)
					}
				});
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-top: 200rpx;
		margin-left: auto;
		margin-right: auto;
		margin-bottom: 50rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}

	.isup {
		width: 65%;
		background: #ff6e6e;
	}

	.isdown {
		width: 65%;
		background: #59fd72;
	}

	.isup1 {
		width: 15%;
		border: solid;
		background: #ca0000;
	}

	.isdown1 {
		width: 15%;
		border: solid;
		background: #007f00;
	}

	.price_box {
		margin: 0;
		width: 100%;
	}

	.button_k {
		width: 20%;
		font-size: small;
	}
</style>
