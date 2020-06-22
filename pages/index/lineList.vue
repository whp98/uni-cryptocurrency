<template>
	<view>
		<view style="display: flex;width: 100%;justify-content: center;">
			点击查看实时价格折线图
		</view>
		<view v-for="(value, key, index) in list" :key="key" class="price_box">
			<view style="display: flex;">
				<view @click="showcandle(key)">
					{{ key }}
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				list:{
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
				}
			}
		},
		methods: {
			showcandle(e) {
				console.debug(e);
				uni.navigateTo({
					url: '../charts/showCurrentLine?coin=' + e
				});
			}
		}
	}
</script>

<style>
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
