<template>	
<!-- add favorite block -->
<div class="cursor-pointer"
>
	<Icon icon="carbon:trash-can"
	class="text-gray-500 hover:text-red-500 transition"
	width="23"
	/>
</div>
<!-- eof add favorite block -->
</template>

<script>
import { Icon } from '@iconify/vue';
// import { ref } from 'vue';
export default {
	name: "CartProductItemAddFavorite",
	components: {
		Icon,
	},
	setup () {
		return {
		}
	}
}
</script>
