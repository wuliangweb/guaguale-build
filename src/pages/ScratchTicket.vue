<template>
	<view class="">
		<view class="reset" @click="handleReset">
			重新开始
		</view>
		<zou-scratch-ticket ref="ticket"></zou-scratch-ticket>
	</view>
</template>

<script>
	import zouScratchTicket from "../component/zou-scratch-ticket"
	export default {
		components: {
			zouScratchTicket
		},
		methods: {
			handleReset () {
				console.log(this.$refs.ticket.drawNumberInit())
			}
		}
	}
</script>

<style scoped>
	.reset {
		z-index: 1;
		position: absolute;
		top: 32rpx;
		left: 370rpx;
		color: white;
		font-family: emoji;
	}
</style>