<template>
	<view>
		<view v-for="item in data" v-bind:key="item.name" class="price_box">
			<view style="display: flex;">
				<view class="c1">{{item.rank}} </view>
				<view class="c2">{{item.name}} </view>
				<view class="c3">{{toFixed(item.percentTotalVolume)}}</view>
				<button class="c4" type="primary" @click="copy(item.exchangeUrl)">复制网址</button>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				data: {},
			}
		},
		methods: {
			toFixed(e){
				if(e===undefined){
					return ""
				}
				return parseFloat(e).toFixed(2);
			},
			copy(e){
				console.debug(e);
				uni.setClipboardData({
				    data: e,
				    success: function () {
				        uni.showToast({
				        	title: "复制完成",
				        	duration: 500
				        })
				    }
				});
			}
		},
		onLoad() {
			var _this = this;
			uni.request({
				url:'https://api.coincap.io/v2/exchanges',
				data: {},
				success: function(res) {
					console.debug(res);
					_this.data=res.data.data;
				},
				fail: () => {
					_self.tips = "网络错误，小程序端请检查合法域名";
				},
			});
		}
	}
</script>

<style>
	.c1{
		width: 10%;
	}
	.c2{
		width: 30%;
	}
	.c3{
		width: 20%;
	}
	.c4{
		width: 30%;
	}
</style>
