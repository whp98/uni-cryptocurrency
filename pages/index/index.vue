<template>
	<view class="content">
		<view>实时行情数据(美元$)</view>
		
		<view v-for="(value, key, index) in prices" :key="key" v-bind:class="value.isup ? 'isup' : 'isdown'">
		   <view class="price_box">
			   {{ key }} : {{ value.p }}
		   </view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				prices:{
					"bitcoin":{"p":"","isup":true},
					"ethereum":{"p":"","isup":true},
					"tether":{"p":"","isup":true},
					"ripple":{"p":"","isup":true},
					"bitcoin-cash":{"p":"","isup":true},
					"bitcoin-sv":{"p":"","isup":true},
					"litecoin":{"p":"","isup":true},
					"binance-coin":{"p":"","isup":true},
					"eos":{"p":"","isup":true},
					"cardano":{"p":"","isup":true}
				},
				showall:false
			}
		},
		onLoad() {
			var top10_url = "wss://ws.coincap.io/prices?assets=bitcoin,ethereum,tether,ripple,bitcoin-cash,bitcoin-sv,,litecoin,binance-coin,eos,cardano";
			var all_url = "wss://ws.coincap.io/prices?assets=ALL";
			uni.connectSocket({
				url:this.showall?all_url:top10_url
			})
			var _this =this;
			uni.onSocketMessage(function (res) {
				var obj = JSON.parse(res.data)
			  for(let key in obj){
				  if(_this.prices[key]==undefined){
					  var new_one = new Object();
					  new_one["p"]=obj[key];
					  new_one["isup"]=true;
					  _this.prices[key]=new_one;
				  }else{
					 _this.prices[key]["isup"]=obj[key]>_this.prices[key]["p"] ? true : false;
					 _this.prices[key]["p"]=obj[key];
				  }
			  } 
			});
		},
		methods: {

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
	
	.isup{
		width: 100%;
		background: #ff6e6e;
	}
	.isdown{
		width: 100%;
		background: #59fd72;
	}
	
	.price_box{
		margin: 10px;
	}
</style>
