<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from 'vue'
import ChatBubble from './HertaChatBubble.vue'
import hertaAudio1 from '../assets/audio/hertaAudio1.mp3'
import hertaAudio2 from '../assets/audio/hertaAudio2.mp3'
import hertaAudio3 from '../assets/audio/hertaAudio3.mp3'
import hertaAudio4 from '../assets/audio/hertaAudio4.mp3'
import silverwolfAudio1 from '../assets/audio/silverwolfAudio1.wav'
import silverwolfAudio2 from '../assets/audio/silverwolfAudio2.wav'
import herta from '../assets/img/herta.webp'
import herta1 from '../assets/img/herta1.webp'
import herta2 from '../assets/img/herta2.webp'
import silverwolf from '../assets/img/silverwolf.webp'

const title = '黑塔转圈圈'

// 计数器
const count = ref<number>(0)
onMounted(() => {
	count.value = localStorage.getItem('count') ? Number(localStorage.getItem('count')) : 0
})

const clickTimes: number[] = [] // 存储点击时间戳的数组
const silverwolfMsg = ref('哼，就这速度？太慢了！')
const silverwolfMsgArr = ['哼，就这速度？太慢了！', '嗯，手速还行']
const silverwolfAudio = [silverwolfAudio1, silverwolfAudio2]
let silverwolfAudioId = 0
const hertaAudio = [hertaAudio1, hertaAudio2, hertaAudio3, hertaAudio4]
const hertaMsg = ref('转圈圈~')
const hertaMsgArr = ['咕噜噜~', '要坏掉了！', '转圈圈', '转圈圈咯~']
let hertaAudioId = 2

// 处理用户点击
function handleClick() {
	const now = Date.now()
	clickTimes.push(now)
	// 只保留最近5次点击的时间戳
	if (clickTimes.length > 5)
		clickTimes.shift()

	// 计算点击速度，如果平均速度小于500毫秒，则显示提示消息
	const averageSpeed = calculateAverageSpeed()
	silverwolfAudioId = averageSpeed < 500 ? 1 : 0
	silverwolfMsg.value = silverwolfMsgArr[silverwolfAudioId]

	// 播放声音
	hertaAudioId = Math.floor(Math.random() * 4)
	const audio = new Audio(hertaAudio[hertaAudioId])

	// 更新文本
	const text = hertaMsgArr[hertaAudioId]
	hertaMsg.value = text
	document.title = text
	audio.play()

	// 播放动画
	startAnimation()

	// 计数
	count.value++
	localStorage.setItem('count', count.value.toString())
}

// 计算平均速度
function calculateAverageSpeed() {
	if (clickTimes.length < 2)
		return Number.POSITIVE_INFINITY

	const totalDuration = clickTimes[clickTimes.length - 1] - clickTimes[0]
	const averageSpeed = totalDuration / (clickTimes.length - 1)
	return averageSpeed
}

let isPlaying = false // 表示音乐是否正在播放
function handlePlayAudio(src: string) {
	if (isPlaying)
		return

	const audio = new Audio(src)
	audio.addEventListener('ended', () => {
		isPlaying = false
	})
	audio.play()
	isPlaying = true
}

function handleClickSilverwolf() {
	return handlePlayAudio(silverwolfAudio[silverwolfAudioId])
}

function handleClickHerta() {
	return handlePlayAudio(hertaAudio[hertaAudioId])
}

// 动画
const animations = ref<{ image: ImageMetadata }[]>([])
const images = [herta1, herta2]

function startAnimation() {
	const randomIndex = Math.floor(Math.random() * 2)
	const randomImage = images[randomIndex]
	animations.value.push({ image: randomImage })
}

function destroyAnimation() {
	animations.value.shift()
}

// 添加案件控制
onMounted(() => {
	window.addEventListener('keydown', handleKeyDown)
})

onBeforeUnmount(() => {
	window.removeEventListener('keydown', handleKeyDown)
})

function handleKeyDown(event: KeyboardEvent) {
	switch (event.code) {
		case 'Space': // Space键
			handleClick()
			break
		default:
			break
	}
}
</script>

<template>
	<main class="flex min-h-screen flex-col items-center justify-between p-10">
		<div class="max-w-sm w-full items-center justify-between flex">
			<div class="stats shadow">
				<div class="stat">
					<div class="stat-title">
						黑塔总计转了
					</div>
					<div class="stat-value">
						{{ count }}
					</div>
					<div class="stat-desc">
						次圈圈，黑塔{{ hertaMsg }}
					</div>
				</div>
			</div>
			<div class="avatar">
				<div class="w-20 rounded-full ring ring-neutral ring-offset-base-200 ring-offset-2 opacity-70">
					<img :src="herta.src" :alt="title">
				</div>
			</div>
		</div>

		<div class="max-w-sm w-full relative">
			<ChatBubble name="黑塔" :avatar="herta.src" msg="欢迎来到黑塔空间站" />
			<ChatBubble end name="银狼" :avatar="silverwolf.src" msg="哟，让我看看" />
			<ChatBubble name="黑塔" :avatar="herta.src" :msg="hertaMsg" @click="handleClickHerta" />
			<ChatBubble end name="银狼" :avatar="silverwolf.src" :msg="silverwolfMsg" @click="handleClickSilverwolf" />
			<TransitionGroup name="fade">
				<template v-for="animation in animations" :key="animation">
					<img class="z-10 w-56 rounded-full absolute bottom-1/2 right-1/4 moving-image"
						:src="animation.image.src" :alt="hertaMsg" @transitionend="destroyAnimation">
				</template>
			</TransitionGroup>
			<div class="pt-16">
				<div class="lg:tooltip btn-block" data-tip="SPACE">
					<button class="btn btn-neutral btn-lg btn-block" @click="handleClick">
						{{ hertaMsg }}
					</button>
				</div>
			</div>
		</div>

		<div class="footer footer-center p-4 text-base-content opacity-40">
			<aside>
				<p><a href="https://silverwolf.cn">All systems nominal</a></p>
				<p>黑塔GIF动图作者Seseren_kr 黑塔和银狼IP形象版权所属miHoYo</p>
			</aside>
		</div>
	</main>
</template>

<style scoped>
.moving-image {
	transform: translateX(50%);
	transition:
		transform 3s linear,
		opacity 4s;
}

.fade-enter-active,
.fade-leave-active {
	transition:
		transform 3s linear,
		opacity 4s;
	opacity: 1;
}

.fade-enter-to,
.fade-leave-to {
	transform: translateX(-50%);
	opacity: 0;
}
</style>