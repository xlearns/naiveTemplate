<template>
	<div class="bd-captcha">
		<div class="drag-verify-container">
			<div :style="dragVerifyImgStyle">
				<img
					ref="checkImg"
					crossOrigin="anonymous"
					:src="imgsrc"
					@load="checkimgLoaded"
					style="width: 100%"
					alt=""
				/>
				<canvas ref="maincanvas" class="main-canvas"></canvas>
				<canvas
					ref="movecanvas"
					:class="{ goFirst: isOk, goKeep: isKeep }"
					class="move-canvas"
				></canvas>
				<div class="refresh" v-if="showRefresh && !modelValue">
					<i :class="refreshIcon" @click="refreshimg"></i>
				</div>
				<div class="tips success" v-if="showTips && modelValue">
					{{ successTip }}
				</div>
				<div class="tips danger" v-if="showTips && !modelValue && showErrorTip">
					{{ failTip }}
				</div>
			</div>
			<div
				ref="dragVerify"
				class="drag_verify"
				:style="dragVerifyStyle"
				@mousemove="dragMoving"
				@mouseup="dragFinish"
				@mouseleave="dragFinish"
				@touchmove="dragMoving"
				@touchend="dragFinish"
			>
				<div
					class="dv_progress_bar"
					:class="{ goFirst2: isOk }"
					ref="progressBar"
					:style="progressBarStyle"
				>
					{{ successMessage }}
				</div>
				<div class="dv_text" :style="textStyle" ref="messageDom">
					{{ message }}
				</div>

				<div
					class="dv_handler dv_handler_bg"
					:class="{ goFirst: isOk }"
					@mousedown="dragStart"
					@touchstart="dragStart"
					ref="handler"
					:style="handlerStyle"
				>
					<i :class="handlerIcon"></i>
				</div>
			</div>
			<div class="triangle"></div>
		</div>
	</div>
</template>
<script lang="ts">
import {
	defineComponent,
	onMounted,
	reactive,
	toRefs,
	ref,
	computed,
} from "vue";

export default defineComponent({
	name: "bdCaptcha",
	props: {
		logo: {
			type: String,
			default: () => {
				return;
			},
		},
		plain: {
			type: Boolean,
			default: () => {
				return false;
			},
		},
		modelValue: {
			type: Boolean,
			default: () => {
				return false;
			},
		},
		width: {
			type: Number,
			default: () => {
				return 250;
			},
		},
		height: {
			type: Number,
			default: () => {
				return 40;
			},
		},
		text: {
			type: String,
			default: () => {
				return "请按住滑块拖动";
			},
		},
		successText: {
			type: String,
			default: () => {
				return "验证通过";
			},
		},
		background: {
			type: String,
			default: () => {
				return "#eee";
			},
		},
		progressBarBg: {
			type: String,
			default: () => {
				return "#76c61d";
			},
		},
		completedBg: {
			type: String,
			default: () => {
				return "#76c61d";
			},
		},
		circle: {
			type: Boolean,
			default: () => {
				return false;
			},
		},
		radius: {
			type: String,
			default: () => {
				return "4px";
			},
		},
		handlerIcon: {
			type: String,
			default: () => {
				return "iconfont icon-right";
			},
		},
		successIcon: {
			type: String,
			default: () => {
				return "iconfont icon-chenggong";
			},
		},
		handlerBg: {
			type: String,
			default: () => {
				return "#fff";
			},
		},
		textSize: {
			type: String,
			default: () => {
				return "14px";
			},
		},
		textColor: {
			type: String,
			default: () => {
				return "#333";
			},
		},
		imgsrc: {
			type: String,
			default: () => {
				return "data:image/png;base64,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";
			},
		},
		barWidth: {
			type: Number,
			default: () => {
				return 40;
			},
		},
		barRadius: {
			type: Number,
			default: () => {
				return 8;
			},
		},
		showRefresh: {
			type: Boolean,
			default: () => {
				return false;
			},
		},
		refreshIcon: {
			type: String,
			default: () => {
				return "iconfont icon-shuaxin icon_fff";
			},
		},
		showTips: {
			type: Boolean,
			default: true,
		},
		successTip: {
			type: String,
			default: "验证通过，超过80%用户",
		},
		failTip: {
			type: String,
			default: "验证未通过，拖动滑块将悬浮图像正确合并",
		},
		diffWidth: {
			type: Number,
			default: 20,
		},
	},
	setup(props, { emit }) {
		const dragVerify = ref();
		const checkImg = ref();
		const maincanvas = ref();
		const movecanvas = ref();
		const handler = ref();
		const messageDom = ref();
		const progressBar = ref();
		const state = reactive({
			isMoving: false,
			x: 0,
			isOk: false,
			isKeep: false,
			clipBarx: 0,
			showErrorTip: false,
		});
		const init = function () {
			const dragEl = dragVerify.value;
			dragEl.style.setProperty("--textColor", props.textColor);
			dragEl.style.setProperty("--width", Math.floor(props.width / 2) + "px");
			dragEl.style.setProperty("--pwidth", -Math.floor(props.width / 2) + "px");
		};
		const methods = {
			draw(ctx: any, x: number, y: number, operation: any) {
				const l = props.barWidth;
				const r = props.barRadius;
				const PI = Math.PI;
				ctx.beginPath();
				ctx.moveTo(x, y);
				ctx.arc(x + l / 2, y - r + 2, r, 0.72 * PI, 2.26 * PI);
				ctx.lineTo(x + l, y);
				ctx.arc(x + l + r - 2, y + l / 2, r, 1.21 * PI, 2.78 * PI);
				ctx.lineTo(x + l, y + l);
				ctx.lineTo(x, y + l);
				ctx.arc(x + r - 2, y + l / 2, r + 0.4, 2.76 * PI, 1.24 * PI, true);
				ctx.lineTo(x, y);
				ctx.lineWidth = 2;
				ctx.fillStyle = "rgba(255, 255, 255, 0.8)";
				ctx.strokeStyle = "rgba(255, 255, 255, 0.8)";
				ctx.stroke();
				ctx[operation]();
				ctx.globalCompositeOperation = "destination-over";
			},
			checkimgLoaded() {
				const barWidth = props.barWidth;
				const imgHeight = checkImg.value.height;
				const imgWidth = checkImg.value.width;

				const halfWidth = Math.floor(props.width / 2);
				const refreshHeigth = 25;
				const tipHeight = 20;
				const x =
					halfWidth +
					Math.ceil(
						Math.random() * (halfWidth - barWidth - props.barRadius - 5)
					);
				let y =
					refreshHeigth +
					Math.floor(
						Math.random() * (imgHeight - barWidth - refreshHeigth - tipHeight)
					);
				const __maincanvas = maincanvas.value;
				__maincanvas.setAttribute("width", imgWidth);
				__maincanvas.setAttribute("height", imgHeight);
				__maincanvas.style.display = "block";
				const canvasCtx = maincanvas.value.getContext("2d");
				methods.draw(canvasCtx, x, y, "fill");
				state.clipBarx = x;
				const __moveCanvas = movecanvas.value;
				__moveCanvas.setAttribute("width", imgWidth);
				__moveCanvas.setAttribute("height", imgHeight);
				__moveCanvas.style.display = "block";
				const L = barWidth + props.barRadius * 2 + 3; //实际宽度
				const moveCtx = __moveCanvas.getContext("2d");
				moveCtx.clearRect(0, 0, imgWidth, imgHeight);
				methods.draw(moveCtx, x, y, "clip");
				moveCtx.drawImage(checkImg.value, 0, 0, imgWidth, imgHeight);
				y = y - props.barRadius * 2 - 1;
				const ImageData = moveCtx.getImageData(x, y, L, L);
				__moveCanvas.setAttribute("width", L);
				__moveCanvas.setAttribute("height", imgHeight);
				moveCtx.putImageData(ImageData, 0, y);
			},
			dragStart(e: any) {
				if (!props.modelValue) {
					state.isMoving = true;
					state.x = e.pageX || e.touches[0].pageX;
				}
				state.showErrorTip = false;
				emit("handlerMove");
			},
			dragMoving(e: any) {
				if (state.isMoving && !props.modelValue) {
					const _x = (e.pageX || e.touches[0].pageX) - state.x;
					const __handler = handler.value;
					const __progressBar = progressBar.value;
					const __movecanvas = movecanvas.value;
					__handler.style.left = _x + "px";
					__progressBar.style.width = _x + props.height / 2 + "px";
					__movecanvas.style.left = _x + "px";
				}
			},
			dragFinish(e: any) {
				if (state.isMoving && !props.modelValue) {
					const _x = (e.pageX || e.changedTouches[0].pageX) - state.x;
					if (Math.abs(_x - state.clipBarx) > props.diffWidth) {
						state.isOk = true;
						const __handler = handler.value;
						const __progressBar = progressBar.value;
						const __movecanvas = movecanvas.value;
						setTimeout(function () {
							__handler.style.left = "0";
							__progressBar.style.width = "0";
							__movecanvas.style.left = "0";
							state.isOk = false;
						}, 500);
						emit("passfail");
						state.showErrorTip = true;
					} else {
						methods.passVerify();
					}
					state.isMoving = false;
				}
			},
			passVerify() {
				emit("update:modelValue", true);
				state.isMoving = false;
				const __handler = handler.value;
				const __progressBar = progressBar.value;
				const __message = messageDom.value;
				const __movecanvas = movecanvas.value;
				const __maincanvas = maincanvas.value;
				__handler.children[0].className = props.successIcon;
				__progressBar.style.background = props.completedBg;
				__message.style["-webkit-text-fill-color"] = "unset";
				__message.style.animation = "slidetounlock2 3s infinite";
				__progressBar.style.color = "#fff";
				__progressBar.style.fontSize = props.textSize;
				state.isKeep = true;
				setTimeout(() => {
					__movecanvas.style.left = state.clipBarx + "px";
					setTimeout(() => {
						state.isKeep = false;
						__maincanvas.style.display = "none";
						__movecanvas.style.display = "none";
					}, 200);
				}, 100);
				emit("passcallback");
			},
			reset() {
				methods.reImg();
				methods.checkimgLoaded();
			},
			reImg() {
				emit("update:modelValue", false);
				//重置
				state.isMoving = false;
				state.x = 0;
				state.isOk = false;
				state.isKeep = false;
				state.clipBarx = 0;
				state.showErrorTip = false;

				const __handler = handler.value;
				const __message: any = messageDom.value;
				const __progressBar = progressBar.value;
				const __movecanvas = movecanvas.value;
				__handler.style.left = "0";
				__progressBar.style.width = "0";
				__handler.children[0].className = props.handlerIcon;
				__message.style["-webkit-text-fill-color"] = "transparent";
				__message.style.animation = "slidetounlock 3s infinite";
				__message.style.color = props.background;
				__movecanvas.style.left = "0px";
			},
			refreshimg() {
				emit("refresh");
			},
		};
		const handlerStyle = computed(() => {
			return {
				width: props.height + "px",
				height: props.height + "px",
				background: props.handlerBg,
			};
		});
		const message = computed(() => {
			return props.modelValue ? "" : props.text;
		});
		const successMessage = computed(() => {
			return props.modelValue ? props.successText : "";
		});
		const dragVerifyStyle = computed(() => {
			return {
				width: props.width + "px",
				height: props.height + "px",
				lineHeight: props.height + "px",
				background: props.background,
				borderRadius: props.circle ? props.height / 2 + "px" : props.radius,
			};
		});
		const dragVerifyImgStyle = computed(() => {
			return {
				width: props.width + "px",
				position: "relative",
				overflow: "hidden",
			};
		});
		const progressBarStyle = computed(() => {
			return {
				background: props.progressBarBg,
				height: props.height + "px",
				borderRadius: props.circle
					? props.height / 2 + "px 0 0 " + props.height / 2 + "px"
					: props.radius,
			};
		});
		const textStyle = computed(() => {
			return {
				height: props.height + "px",
				width: props.width + "px",
				fontSize: props.textSize,
			};
		});

		onMounted(() => {
			init();
		});
		return {
			...methods,
			...toRefs(state),
			dragVerify,
			handlerStyle,
			progressBarStyle,
			textStyle,
			dragVerifyImgStyle,
			dragVerifyStyle,
			successMessage,
			message,
			checkImg,
			maincanvas,
			movecanvas,
			handler,
			progressBar,
			messageDom,
		};
	},
});
</script>
