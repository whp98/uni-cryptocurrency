<template>
	<view class="qiun-columns">
		<view class="qiun-charts">
			<h4>{{coin}}/usdt</h4>
			<!--#ifdef MP-ALIPAY -->
			<canvas canvas-id="canvasCandle" id="canvasCandle" class="charts" :width="cWidth*pixelRatio" :height="cHeight*pixelRatio"
			 :style="{'width':cWidth+'px','height':cHeight+'px'}" disable-scroll=true @touchstart="touchCandle" @touchmove="moveCandle"
			 @touchend="touchEndCandle"></canvas>
			<!--#endif-->
			<!--#ifndef MP-ALIPAY -->
			<canvas canvas-id="canvasCandle" id="canvasCandle" class="charts" disable-scroll=true @touchstart="touchCandle"
			 @touchmove="moveCandle" @touchend="touchEndCandle"></canvas>
			<!--#endif-->
		</view>
		<view class="qiun-bg-white qiun-title-bar qiun-common-mt qiun-rows">
			<view style="flex: 1;qiun-rows;text-align: right;">
				<button type="default" size="mini" @tap="tapButton('in')">放大</button>
				<button type="default" size="mini" style="margin-left: 20upx;" @tap="tapButton('out')">缩小</button>
			</view>
		</view>
		<view class="qiun-padding qiun-bg-white ">
			<slider :value="itemCount" min="5" :max="sliderMax" block-color="#f8f8f8" block-size="18" @changing="sliderMove"
			 @change="sliderMove" />
		</view>
		<!--#ifdef H5 -->
		<view style="display: flex;width: 100%;"><view class="name">K线</view>
			<picker mode="selector" class="pick" :range="selector_array" :value="selector_index" @change="PickK">
				<view style="border: solid;" class="uni-input">{{this.selector_array[this.selector_index]}}</view>
			</picker>
		</view>
		<view style="display: flex;width: 100%;"><view class="name">
			开始时间
		</view>  <picker  mode="date" class="pick" :value="starttime" @change="bindDateStartChange">
				<view style="border: solid;" class="uni-input">{{starttime}}</view>
			</picker>
		</view>
		<view style="display: flex;width: 100%;"><view class="name">结束时间</view><picker mode="date" class="pick" :value="endtime" @change="bindDateEndChange">
				<view style="border: solid;" class="uni-input">{{endtime}}</view>
			</picker>
		</view>
		<button class="qiun-button" type="primary" @tap="getServerData()">更新图表</button>
		<!--#endif-->
	</view>
</template>

<script>
	import uCharts from '@/components/u-charts/u-charts.js';
	var _self;
	var canvaCandle = null;

	export default {
		data() {
			return {
				cWidth: '',
				cHeight: '',
				pixelRatio: 1,
				itemCount: 20, //x轴单屏数据密度
				sliderMax: 50,
				starttime: "2020-01-01",
				endtime: "2020-01-10",
				selector_array: ["m1", "m5", "m15", "m30", "h1", "h2", "h4", "h8", "h12", "d1", "w1"],
				selector_index: 4,
				coin:"eos"
			}
		},
		onLoad(e) {
			console.debug(e);
			if(e.coin){
				this.coin=e.coin;
			}
			_self = this;
			//#ifdef MP-ALIPAY
			uni.getSystemInfo({
				success: function(res) {
					if (res.pixelRatio > 1) {
						//正常这里给2就行，如果pixelRatio=3性能会降低一点
						//_self.pixelRatio =res.pixelRatio;
						_self.pixelRatio = 2;
					}
				}
			});
			//#endif
			this.cWidth = uni.upx2px(750);
			this.cHeight = uni.upx2px(500);
			this.getServerData();
		},
		methods: {
			PickK(e) {
				this.selector_index = e.target.value
			},
			getTimeStamp(str) {
				var time = new Date(str);
				console.debug(time.getTime());
				return time.getTime();
			},
			bindDateStartChange: function(e) {
				this.starttime = e.target.value
			},
			bindDateEndChange: function(e) {
				this.endtime = e.target.value
				console.debug(e)
			},
			getServerData() {
				uni.request({
					url: 'https://api.coincap.io/v2/candles?start=' + this.getTimeStamp(this.starttime) + '&end=' + this.getTimeStamp(
							this.endtime) +
						'&exchange=poloniex&interval=' + this.selector_array[this.selector_index] + '&baseId='+this.coin+'&quoteId=tether',
					data: {},
					success: function(res) {
						//console.log(res.data.data)
						let Candle = {
							categories: [],
							series: []
						};
						var obj = new Object();
						obj["name"] = "交易数据"
						obj["data"] = []
						res.data.data.forEach(function(item) {
							var array = [];
							// Pie.series.push({"name":item.id,"data":item.volumeUsd24Hr});
							Candle.categories.push(_self.formatetime(item.period));
							array.push(parseFloat(item.open))
							array.push(parseFloat(item.close))
							array.push(parseFloat(item.low))
							array.push(parseFloat(item.high))
							obj["data"].push(array)
							//console.debug(Candle);
						});
						//这里我后台返回的是数组，所以用等于，如果您后台返回的是单条数据，需要push进去
						//console.debug(Candle);

						Candle.series.push(obj);
						_self.showCandle("canvasCandle", Candle);
					},
					fail: () => {
						_self.tips = "网络错误，小程序端请检查合法域名";
					},
				});
			},
			formatetime(timestamp) {
				if (timestamp) {
					var time = new Date(timestamp);
					var y = time.getFullYear();
					var M = time.getMonth() + 1;
					var d = time.getDate();
					var h = time.getHours();
					var m = time.getMinutes();
					var s = time.getSeconds();
					return y + '-' + _self.addZero(M) + '-' + _self.addZero(d);
				} else {
					return '';
				}
			},
			addZero(m) {
				return m < 10 ? '0' + m : m;
			},
			showCandle(canvasId, chartData) {
				canvaCandle = new uCharts({
					$this: _self,
					canvasId: canvasId,
					type: 'candle',
					fontSize: 11,
					padding: [15, 15, 0, 15],
					legend: {
						show: true,
						padding: 5,
						lineHeight: 11,
						margin: 8,
					},
					background: '#FFFFFF',
					pixelRatio: _self.pixelRatio,
					enableMarkLine: true,
					/***需要开启标记线***/
					categories: chartData.categories,
					series: chartData.series,
					animation: true,
					enableScroll: true, //开启图表拖拽功能
					xAxis: {
						disableGrid: true, //不绘制X轴网格线
						labelCount: 4, //X轴文案数量
						//type:'grid',
						//gridType:'dash',
						itemCount: _self.itemCount, //可不填写，配合enableScroll图表拖拽功能使用，x轴单屏显示数据的数量，默认为5个
						scrollShow: true, //新增是否显示滚动条，默认false
						scrollAlign: 'right',
						//scrollBackgroundColor:'#F7F7FF',//可不填写，配合enableScroll图表拖拽功能使用，X轴滚动条背景颜色,默认为 #EFEBEF
						//scrollColor:'#DEE7F7',//可不填写，配合enableScroll图表拖拽功能使用，X轴滚动条颜色,默认为 #A6A6A6
					},
					yAxis: {
						//disabled:true
						gridType: 'dash',
						splitNumber: 5,
						format: (val) => {
							return val.toFixed(0)
						}
					},
					width: _self.cWidth * _self.pixelRatio,
					height: _self.cHeight * _self.pixelRatio,
					dataLabel: false,
					dataPointShape: true,
					extra: {
						candle: {
							color: {
								upLine: '#f04864',
								upFill: '#f04864',
								downLine: '#2fc25b',
								downFill: '#2fc25b'
							},
							average: {
								show: true,
								name: ['MA5', 'MA10', 'MA30'],
								day: [5, 10, 20],
								color: ['#1890ff', '#2fc25b', '#facc14']
							}
						},
						tooltip: {
							bgColor: '#000000',
							bgOpacity: 0.7,
							gridType: 'dash',
							dashLength: 5,
							gridColor: '#1890ff',
							fontColor: '#FFFFFF',
							horizentalLine: true,
							xAxisLabel: true,
							yAxisLabel: true,
							labelBgColor: '#DFE8FF',
							labelBgOpacity: 0.95,
							labelAlign: 'left',
							labelFontColor: '#666666'
						},
						markLine: {
							type: 'dash',
							dashLength: 5,
							data: [{
								value: 2150,
								lineColor: '#f04864',
								showLabel: true
							}, {
								value: 2350,
								lineColor: '#f04864',
								showLabel: true
							}]
						}
					}
				});

			},
			touchCandle(e) {
				canvaCandle.scrollStart(e);
			},
			moveCandle(e) {
				canvaCandle.scroll(e);
			},
			touchEndCandle(e) {
				canvaCandle.scrollEnd(e);
				//下面是toolTip事件，如果滚动后不需要显示，可不填写
				canvaCandle.showToolTip(e, {
					format: function(item, category) {
						return category + ' ' + item.name + ':' + item.data
					}
				});
				//这里演示了获取点击序列的方法，如需要将数据显示到canvas外面，可用此方法。
				var xx = canvaCandle.getCurrentDataIndex(e);
				//console.log(canvaCandle.opts.series[0].data[xx]);
				//下面是计算好的MA均线集合，想要点击序列中的当前数据，需要自己遍历seriesMA
				//console.log(canvaCandle.opts.seriesMA);
			},
			tapButton(type) {
				let step = 5;
				if (type == 'in') {
					_self.itemCount -= step;
					if (_self.itemCount <= 5) {
						_self.itemCount = 5;
					}
				} else {
					_self.itemCount += step;
					if (_self.itemCount >= _self.sliderMax) {
						_self.itemCount = _self.sliderMax;
					}
				}
				_self.zoomCandle(_self.itemCount);
			},
			sliderMove(e) {
				_self.itemCount = e.detail.value;
				_self.zoomCandle(e.detail.value);
			},
			zoomCandle(val) {
				canvaCandle.zoom({
					itemCount: val
				});
			},
			changeData() {
				if (isJSON(_self.textarea)) {
					let newdata = JSON.parse(_self.textarea);
					canvaCandle.updateData({
						series: newdata.series,
						categories: newdata.categories
					});
				} else {
					uni.showToast({
						title: '数据格式错误',
						image: '../../../static/images/alert-warning.png'
					})
				}
			}
		}
	}
</script>

<style>
	/*样式的width和height一定要与定义的cWidth和cHeight相对应*/
	.qiun-charts {
		width: 750upx;
		height: 500upx;
		background-color: #FFFFFF;
	}

	.charts {
		width: 750upx;
		height: 500upx;
		background-color: #FFFFFF;
	}
	
	.pick{
		width: 40%;
	}
	
	.name{
		width: 60%;
	}
</style>
