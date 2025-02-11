<template>
	<view class="scratch-ticket-page">
		<image class="scratch-ticket-bg" :src="bgImg" mode="widthFix"></image>
		<canvas class="canvas" type="2d" id="canvas-number"></canvas>
		<canvas class="canvas" type="2d" id="canvas-layer" disable-scroll @touchstart="touchstart" @touchmove="touchmove"></canvas>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				containerHeight: 0,
				bgImg: '/static/wukong-bg.jpg', // 背景图
				layerImg: `/static/wukong-layer.png`, // 刮刮乐图层
				ctx: null,
				points: []
			}
		},
		mounted() {
			this.drawNumberInit();
		},
		methods: {
			// 绘制图层
			async drawLayerInit() {
				let that = this;

				// #ifdef H5
				const canvasNumber = document.querySelector('#canvas-layer canvas')
				that.ctx = canvasNumber.getContext("2d");
				const { width, height } = canvasNumber.getBoundingClientRect();
				const dpr = window.devicePixelRatio;
				
				// 设置物理像素
				canvasNumber.width = width * dpr;
				canvasNumber.height = height * dpr;
				canvasNumber.style.width = `${width}px`;
				canvasNumber.style.height = `${height}px`;
				
				const systemInfo = uni.getSystemInfoSync();
				let wdthDpr = systemInfo.windowWidth / 375;
				// 图层
				let layerImg = new Image;
				layerImg.src = that.layerImg;
				layerImg.onload = (res) => {
					that.ctx.drawImage(layerImg, 0, 0, wdthDpr * 323, wdthDpr *
						244);
					that.ctx.lineWidth = 40;
					that.ctx.lineCap = "round";
					that.ctx.lineJoin = "round";
					that.ctx.globalCompositeOperation = 'destination-out';
				}	
				// #endif
				// #ifndef H5
				const query = uni.createSelectorQuery().in(this);
				query
					.select("#canvas-layer")
					.fields({
						node: true,
						size: true
					})
					.exec((res) => {
						const canvas = res[0].node;
						that.ctx = canvas.getContext("2d");
						const systemInfo = uni.getSystemInfoSync();
						const dpr = systemInfo.pixelRatio;
						canvas.width = res[0].width * dpr;
						canvas.height = res[0].height * dpr;
						let wdthDpr = systemInfo.windowWidth / 375;
						that.ctx.scale(dpr, dpr);
						// 图层
						let layerImg = canvas.createImage();
						layerImg.src = that.layerImg;
						layerImg.onload = (res) => {
							that.ctx.drawImage(layerImg, 0, 0, wdthDpr * 323, wdthDpr *
								244);
							that.ctx.lineWidth = 40;
							that.ctx.lineCap = "round";
							that.ctx.lineJoin = "round";
							that.ctx.globalCompositeOperation = 'destination-out';
						}
					})
				// #endif
			},
			// 绘制刮刮乐号码
			drawNumberInit() {
				let that = this;
				// #ifdef H5
				const canvasNumber = document.querySelector('#canvas-number canvas')
				const { width, height } = canvasNumber.getBoundingClientRect();
				const dpr = window.devicePixelRatio;
				
				// 设置物理像素
				canvasNumber.width = width * dpr;
				canvasNumber.height = height * dpr;
				canvasNumber.style.width = `${width}px`;
				canvasNumber.style.height = `${height}px`;
				
				const ctx = canvasNumber.getContext('2d');
				ctx.scale(dpr, dpr);
				ctx.font = "normal bold 10px sans-serif";
				ctx.fillStyle = "#333333";
				const systemInfo = uni.getSystemInfoSync();
				let wdthDpr = systemInfo.windowWidth / 375 / dpr;
				// let wdthDpr = window.devicePixelRatio;
				// 中奖号码
				ctx.fillText(that.randNumber(), wdthDpr * 120, wdthDpr * 40);
				ctx.fillText(that.randNumber(), wdthDpr * 180, wdthDpr * 40);
				// 刮奖号码
				// 生成一个4x5数组
				const numberArr = new Array(4).fill('').map(item => new Array(5).fill(1));
				numberArr.forEach((data, dataIndex) => {
					data.forEach((item, index) => {
						ctx.fillText(that.randNumber(), wdthDpr * (20 + 65 * index), wdthDpr *
							(85 + 45 * dataIndex));
					})
				})
				ctx.font = "normal bold 4px sans-serif";
				ctx.fillStyle = "#999999";
				numberArr.forEach((data, dataIndex) => {
					data.forEach((item, index) => {
						ctx.fillText(`￥${that.randNumber()}.00`, wdthDpr * (5 + 65 * index),
							wdthDpr * (100 + 45 * dataIndex));
					})
				})
				ctx.restore();
				that.drawLayerInit();				
				// #endif
				// #ifndef H5
				const query = uni.createSelectorQuery().in(this);
				query
					.select("#canvas-number")
					.fields({
						node: true,
						size: true
					})
					.exec((res) => {
						const canvas = res[0].node;
						const ctx = canvas.getContext("2d");
						const systemInfo = uni.getSystemInfoSync();
						const dpr = systemInfo.pixelRatio;
						canvas.width = res[0].width * dpr;
						canvas.height = res[0].height * dpr;
						let wdthDpr = systemInfo.windowWidth / 375;
						ctx.scale(dpr, dpr);
						ctx.font = "normal bold 18px sans-serif";
						ctx.fillStyle = "#333333";
						// 中奖号码
						ctx.fillText(that.randNumber(), wdthDpr * 120, wdthDpr * 40);
						ctx.fillText(that.randNumber(), wdthDpr * 180, wdthDpr * 40);
						// 刮奖号码
						// 生成一个4x5数组
						const numberArr = new Array(4).fill('').map(item => new Array(5).fill(1));
						numberArr.forEach((data, dataIndex) => {
							data.forEach((item, index) => {
								ctx.fillText(that.randNumber(), wdthDpr * (20 + 65 * index), wdthDpr *
									(85 + 45 * dataIndex));
							})
						})
						ctx.font = "normal bold 12px sans-serif";
						ctx.fillStyle = "#999999";
						numberArr.forEach((data, dataIndex) => {
							data.forEach((item, index) => {
								ctx.fillText(`￥${that.randNumber()}.00`, wdthDpr * (5 + 65 * index),
									wdthDpr * (100 + 45 * dataIndex));
							})
						})
						ctx.restore();
						that.drawLayerInit();
					})
				// #endif
			},
			touchstart(e) {
				e.preventDefault()
				let startX = e.changedTouches[0].x;
				let startY = e.changedTouches[0].y;
				let startPoint = {
					x: startX,
					y: startY
				};
				this.points.push(startPoint);
				this.ctx.beginPath();
			},
			touchmove(e) {
				e.preventDefault()
				let moveX = e.changedTouches[0].x;
				let moveY = e.changedTouches[0].y;
				let movePoint = {
					x: moveX,
					y: moveY,
				}
				this.points.push(movePoint);
				if (this.points.length >= 2) {
					this.drawLine();
				}
			},
			drawLine() {
				let points1 = this.points[0];
				let points2 = this.points[1];
				this.points.shift();
				this.ctx.moveTo(points1.x, points1.y);
				this.ctx.lineTo(points2.x, points2.y);
				this.ctx.stroke();
			},
			randNumber() {
				let number = Math.floor(Math.random() * (100 - 1)) + 1;
				return number < 10 ? `0${number}` : number;
			},
		}
	}
</script>

<style lang="scss" scoped>
	.scratch-ticket-page {
		position: relative;
		width: 100%;
		// height: 100vh;

		.scratch-ticket-bg {
			width: 100%;
			vertical-align: middle;
			// height: 100vh;
		}

		.canvas {
			position: absolute;
			left: 52rpx;
			top: 784rpx;
			width: 650rpx;
			height: 490rpx;
		}
	}
</style>