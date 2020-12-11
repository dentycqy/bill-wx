<template name="index">
	<view>
		<!-- <image src="../../static/BasicsBg.png" mode="widthFix" class="response"></image> -->
		<cu-custom bgColor="bg-gradual-pink"><block slot="content">$$$$</block></cu-custom>
		<view class="card radius">
			<view class="flex">
				<view class="flex-sub bg-cyan padding-sm  margin-xs radius">
					本周:
					<text>{{ '&emsp;' + stat.weekTotal / 10000 }} ¥</text>
				</view>
				<view class="flex-sub bg-cyan padding-sm margin-xs radius">
					本月:
					<text>{{ '&emsp;' + stat.monthTotal / 10000 }} ¥</text>
				</view>
			</view>
		</view>
		<view v-for="(item, index) in listBill" :key="index">
			<view class="card radius padding-sm">
				<view class="text-xs radius bg-gradual-orange cu-tag">{{ item.date }}</view>
				<text class="text-xs radius cu-tag bg-gradual-green">合计：2000 ¥</text>
			</view>
			<view class="cu-timeline">
				<view
					v-for="(item1, index1) in item.dataList"
					class="cu-item text-red animation-slide-left rmb"
					:class="'rmb-' + item1.icon"
					:style="[{ animationDelay: (index1 + 1) * 0.1 + 's' }]"
					style="position: relative;"
				>
					<view class="content shadow-blur" :style="'background-color:' + item1.color">
						<text class="text-white">『{{ item1.type }}』</text>
						<text class="text-white">『{{ item1.money / 10000 + '&nbsp;' }}¥』</text>
						<text class="text-white">{{ item1.memo }}</text>
						<view class="action" style="position: absolute;right: 15rpx;top: 30rpx;">
							<view class="text-xs radius bg-white cu-tag">{{ item1.dateTime.substr(10) }}</view>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
var myJs = require('../../static/js/my.js');
let app = getApp();
let vm = null;
export default {
	name: 'index',
	data() {
		return {
			listType: [],
			listBill: [],
			stat: {
				weekTotal: 0,
				monthTotal: 0
			}
		};
	},
	onShow: function() {},
	onLoad: function() {
		vm = this;
		this.listType = require('@/static/data/dict-type.json');
		uni.startPullDownRefresh();
	},
	onPullDownRefresh() {
		vm.queryDataList();
		setTimeout(function() {
			uni.stopPullDownRefresh();
		}, 1000);
	},
	methods: {
		queryDataList() {
			var time1 = myJs.getWeekStartTime().Format();
			var time2 = myJs.getWeekEndTime().Format();

			var monthTime1 = myJs.getMonthStartTime().Format();
			var monthTime2 = myJs.getMonthEndTime().Format();

			uniCloud
				.callFunction({
					name: 'getSumByDateArea',
					data: {
						userId: uni.getStorageSync('userId'),
						time1: time1,
						time2: time2
					}
				})
				.then(res => {
					vm.stat.weekTotal = res.result.data[0].sum;
				})
				.catch(err => {});
			uniCloud
				.callFunction({
					name: 'getSumByDateArea',
					data: {
						userId: uni.getStorageSync('userId'),
						time1: monthTime1,
						time2: monthTime2
					}
				})
				.then(res => {
					vm.stat.monthTotal = res.result.data[0].sum;
				})
				.catch(err => {});

			uni.showLoading({
				title: '查询中...'
			});
			uniCloud
				.callFunction({
					name: 'get',
					data: {
						userId: uni.getStorageSync('userId'),
						time1: time1,
						time2: time2
					}
				})
				.then(res => {
					uni.hideLoading();
					vm.showData(res.result.data);
				})
				.catch(err => {});
		},
		showData(dataList) {
			dataList.forEach(d => {
				vm.listType.some(item => {
					if (d.type === item.name) {
						d.icon = item.icon;
						d.color = item.color;
						d.color1 = item.color1;
						return true;
					}
				});
			});
			dataList = this.formatDateList(dataList);
			// return;
			this.listBill = dataList;
		},
		formatDateList: function(dataList) {
			// 赋值日期
			// 将不同的日期生成数组
			let dateList = [];
			dataList.forEach(d => {
				d.date = d.dateTime.substr(0, 10);
				if (dateList.indexOf(d.date) === -1) {
					dateList.push(d.date);
				}
			});
			let resultList = [];
			// 生成日期和对应日期数据数组的 数组  [{date:"2020-12-12",dataList:[{},{}]}]
			dateList.forEach(function(item) {
				let one = { date: item }; // resultList的一条数据
				let oneDataList = []; // 一条数据的账单明细数组
				dataList.forEach(d => {
					if (d.date === item) {
						oneDataList.push(d);
					}
				});
				one.dataList = oneDataList;
				resultList.push(one);
			});
			return resultList;
		}
	}
};
</script>

<style>
@import '../../components/colorui/animation.css';

.card {
	box-shadow: 1rpx 1rpx 1rpx rgba(0, 0, 0, 0.1);
	margin: 10rpx;
	padding: 5rpx;
	background-color: #ffffff;
}
.rmb::before {
	font-family: rmb !important;
	font-size: 20px;
}
</style>
