<script setup lang="ts">
import defaultAvatar from '../assets/img/silverwolf.webp'

const props = withDefaults(defineProps<{
	end?: boolean
	name: string
	avatar?: string
	msg: string
}>(), {
	end: false,
	avatar: defaultAvatar.src,
})

function currentTime() {
	const now = new Date()
	const hours = now.getHours().toString().padStart(2, '0')
	const minutes = now.getMinutes().toString().padStart(2, '0')
	return `${hours}:${minutes}`
}

const isChatEnd = props.end ? 'chat chat-end' : 'chat chat-start'
</script>

<template>
	<div :class="isChatEnd">
		<div class="chat-image avatar">
			<div class="w-10 rounded-full">
				<img :src="avatar" :alt="name">
			</div>
		</div>
		<div class="chat-header">
			{{ name }}
			<time class="text-xs opacity-50">{{ currentTime() }}</time>
		</div>
		<div class="chat-bubble">{{ msg }}</div>
	</div>
</template>