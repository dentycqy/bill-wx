<template>
	<view>
		<view :style="bgCss">
			<cu-custom><block slot="content" style="color: #ffffff;">用钱</block></cu-custom>
			<form>
				<view class="cu-form-group margin-top translucent">
					<view class="title font-round" style="color: #ffffff;">金额</view>
					<input placeholder-style="color:#FFFFFF" placeholder="0.00" type="digit" v-model="bill.money" style="color: #FFFFFF;" focus="true" />
				</view>
				<view class="cu-form-group margin-top translucent">
					<view class="title font-round" style="color: #ffffff;">备注</view>
					<input placeholder-style="color:#FFFFFF" placeholder="备注" v-model="bill.memo" style="color: #FFFFFF;" />
				</view>
			</form>
		</view>
		<view class="grid col-4 translucent">
			<view
				v-for="(item, index) in listType"
				class="margin-tb-sm text-center font-round"
				:style="[{ animation: 'show ' + ((index + 1) * 0.2 + 1) + 's 1' }]"
				:key="item.name"
				@tap="chooseType(item)"
			>
				<button class="cu-btn shadow" :style="'color:#ffffff; background-image: linear-gradient(45deg, ' + item.color + ',' + item.color1 + ');'">
					<span :class="'rmb rmb-' + item.icon"></span>
					{{ '&nbsp;' + item.name }}
				</button>
			</view>
		</view>
		<view class="padding flex flex-direction">
			<button class="cu-btn lg font-round" :style="bgCss" @tap="saveRecord">钱财乃是身外之物</button>
			<button open-type="getUserInfo" />
		</view>
		<view class="cu-modal" :class="modal.isShow ? 'show' : ''">
			<view class="cu-dialog">
				<view class="cu-bar bg-white justify-end">
					<view class="content">{{ modal.title }}</view>
					<view class="action" @tap="hideModal"><text class="cuIcon-close text-red"></text></view>
				</view>
				<view class="padding-xl">{{ modal.content }}</view>
			</view>
		</view>
	</view>
</template>

<script>
var app = getApp();
var vm = null;
export default {
	data() {
		return {
			listType: [],
			currType: {},
			bill: { money: null, memo: '' },
			modal: {
				isShow: false,
				title: '',
				content: ''
			}
		};
	},
	onLoad: function() {
		vm = this;
		// 读取类型json文件
		let listType = require('@/static/data/dict-type.json');
		this.listType = listType;
		// 设置上一次使用的类型,如第一次使用,设置为默认类型
		let defaultType = this.listType[0];
		if (uni.getStorageSync('type') != null && uni.getStorageSync('type').trim() != '') {
			this.currType = JSON.parse(uni.getStorageSync('type'));
			this.bill.type = this.currType.name;
		} else {
			this.currType = defaultType;
			this.bill.type = defaultType.name;
		}
	},
	computed: {
		bgCss() {
			return 'color:#ffffff;background-image: linear-gradient(45deg, ' + this.currType.color + ',' + this.currType.color1 + ');';
		}
	},
	methods: {
		chooseType: function(item) {
			this.currType = item;
			this.bill.type = item.name;
			uni.setStorage({
				key: 'type',
				data: JSON.stringify(this.currType)
			});
		},
		saveRecord: function() {
			if (!this.bill.money || this.bill.money <= 0) {
				this.modal.isShow = true;
				this.modal.title = '提示';
				this.modal.content = '0元白嫖?哪有,带带我!';
				return;
			}
			// 金额乘以10000，以 1/10000元 为单位保存，避免使用double计算
			let sendData = {
				money: vm.bill.money * 10000,
				type: vm.bill.type,
				dateTime: new Date().Format(),
				userId: uni.getStorageSync('userId')
			};
			uni.showLoading({
				title: '保存中...'
			});
		
			uniCloud
				.callFunction({
					name: 'add',
					data: sendData
				})
				.then(res => {
					// 置空金额和备注
					vm.bill.money = null;
					vm.bill.memo = '';

					uni.hideLoading();
					uni.showModal({
						content: `保存成功`,
						showCancel: false
					});
				})
				.catch(err => {
					uni.hideLoading();
					uni.showModal({
						content: `添加数据失败，错误信息为：${err.message}`,
						showCancel: false
					});
					console.log(err);
				});
		},
		hideModal: function() {
			this.modal.isShow = false;
		}
	}
};
</script>

<style>
.translucent {
	border: none !important;
	background-color: rgba(0, 0, 0, 0);
}

.uni-input-placeholder {
	color: #ffffff;
}

uni-input,
.uni-input-input {
	color: #ffffff;
}
</style>
