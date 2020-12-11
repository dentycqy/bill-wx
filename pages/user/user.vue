<template>
	<view>
		<cu-custom bgColor="bg-gradual-pink"><block slot="content">w(ﾟДﾟ)w</block></cu-custom>
		<view class="qiun-columns">
			<!-- <view class="qiun-bg-white qiun-title-bar qiun-common-mt"><view class="qiun-title-dot-light">环形图</view></view> -->
			<view class="qiun-charts"><canvas canvas-id="canvasRing" id="canvasRing" class="charts" @touchstart="touchRing"></canvas></view>
		</view>
	</view>
</template>

<script>
import uCharts from '@/components/u-charts/u-charts.js';
var myJs = require('../../static/js/my.js');
var _self;
var canvaRing = null;

export default {
	data() {
		return {
			cWidth: '',
			cHeight: '',
			pixelRatio: 1,
			serverData: ''
		};
	},
	onLoad() {
		_self = this;
		this.cWidth = uni.upx2px(750);
		this.cHeight = uni.upx2px(500);
		this.getServerData();
	},
	methods: {
		getServerData() {
			var time1 = myJs.getWeekStartTime().Format();
			var time2 = myJs.getWeekEndTime().Format();
			uniCloud
				.callFunction({
					name: 'getStatMsg',
					data: {
						userId: uni.getStorageSync('userId'),
						time1: time1,
						time2: time2
					}
				})
				.then(res => {
					console.log(JSON.stringify(res.result.data));
					let responeData = res.result.data;
					let dataList = [];
					let total = 0; // 总金额
					responeData.forEach(function(item) {
						if (item._id) {
							let one = { name: item._id, data: item.sum / 10000 };
							total += one.data;
							dataList.push(one);
						}
					});
					console.log(dataList);
					let Ring = { series: [] };
					Ring.series = dataList;
					_self.showRing('canvasRing', Ring, total);
				})
				.catch(err => {
					console.log(err);
				});
		},
		showRing(canvasId, chartData, total) {
			canvaRing = new uCharts({
				$this: _self,
				canvasId: canvasId,
				type: 'ring',
				fontSize: 11,
				legend: true,
				title: {
					name: total,
					color: '#7cb5ec',
					fontSize: 25 * _self.pixelRatio,
					offsetY: 0 * _self.pixelRatio
				},
				subtitle: {
					name: '合计',
					color: '#666666',
					fontSize: 15 * _self.pixelRatio,
					offsetY: 1 * _self.pixelRatio
				},
				extra: {
					pie: {
						offsetAngle: -45,
						ringWidth: 40 * _self.pixelRatio,
						labelWidth: 15
					}
				},
				background: '#FFFFFF',
				pixelRatio: _self.pixelRatio,
				series: chartData.series,
				animation: true,
				width: _self.cWidth * _self.pixelRatio,
				height: _self.cHeight * _self.pixelRatio,
				disablePieStroke: true,
				dataLabel: true
			});
		},
		touchRing(e) {
			canvaRing.showToolTip(e, {
				format: function(item) {
					return item.name + ':' + item.data;
				}
			});
		}
	}
};
</script>

<style>
page {
	background: #f2f2f2;
	width: 750upx;
	overflow-x: hidden;
}
.qiun-padding {
	padding: 2%;
	width: 96%;
}
.qiun-wrap {
	display: flex;
	flex-wrap: wrap;
}
.qiun-rows {
	display: flex;
	flex-direction: row !important;
}
.qiun-columns {
	display: flex;
	flex-direction: column !important;
}
.qiun-common-mt {
	margin-top: 10upx;
}
.qiun-bg-white {
	background: #ffffff;
}
.qiun-title-bar {
	width: 96%;
	padding: 10upx 2%;
	flex-wrap: nowrap;
}
.qiun-title-dot-light {
	border-left: 10upx solid #0ea391;
	padding-left: 10upx;
	font-size: 32upx;
	color: #000000;
}
.qiun-charts {
	width: 750upx;
	height: 500upx;
	background-color: #ffffff;
}
.charts {
	width: 750upx;
	height: 500upx;
	background-color: #ffffff;
}
</style>
