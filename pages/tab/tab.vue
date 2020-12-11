<template>
	<view>
		<index v-if="PageCur=='index'"></index>
		<add v-if="PageCur=='add'"></add>
		<user v-if="PageCur=='user'"></user>
		<view class="cu-bar tabbar bg-white shadow foot">
			<view class="action" @click="NavChange" data-cur="index" id="index">
				<view class='cuIcon-cu-image'>
					<image :src="'/static/tabbar/basics' + [PageCur=='index'?'_cur':''] + '.png'"></image>
				</view>
				<view :class="PageCur=='index'?'text-green':'text-gray'">记录</view>
			</view>
			<view class="action" @click="NavChange" data-cur="add">
				<view class='cuIcon-cu-image'>
					<image :src="'/static/tabbar/component' + [PageCur == 'add'?'_cur':''] + '.png'"></image>
				</view>
				<view :class="PageCur=='add'?'text-green':'text-gray'">扔钱</view>
			</view>
			<view class="action" @click="NavChange" data-cur="user">
				<view class='cuIcon-cu-image'>
					<image :src="'/static/tabbar/plugin' + [PageCur == 'user'?'_cur':''] + '.png'"></image>
				</view>
				<view :class="PageCur=='user'?'text-green':'text-gray'">个人</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
		return {
				PageCur: 'index'
			}
		},
		methods: {
			NavChange: function(e) {
				this.PageCur = e.currentTarget.dataset.cur
			}
		}
	}
	// import index from '../../pages/index/index.vue';
	uni.$on('turn2Index', function(data) {
		SelectorQuery.select("#index").click();
		// index.PageCur = 'index';
		console.log('监听到事件来自 turn2Index ，携带参数 msg 为：' + data.msg);
	});
</script>

<style>

</style>
