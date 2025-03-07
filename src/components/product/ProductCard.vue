<template>
<!-- 
	product card display as vertical card component,
	but at tylewindcss "md" default screen responsive breakpoint 
	wraps to horizontail view
-->
<div class="relative z-0 flex flex-1 md:flex-col md:items-center md:justify-between md:max-w-[400px] border-b border-gray-100 pt-2 pb-5
	md:border-none py-0 overflow-x-hidden">

	<!-- image -->
	<div 
	@click="goToProductPage"
	class="w-5/12 flex-shrink-0 max-w-[180px]  md:max-w-[250px] md:h-[220px] max-h-[220px] min-w-[50px] relative rounded md:w-full">
		<img
			v-lazy="product?.imgsrc[0]"
			class="object-contain w-full md:h-[220px] md:max-h-full h-full rounded cursor-pointer"
		/>
	</div>
	<!-- eof image -->
	<!-- details -->
	<div class="flex flex-col justify-center md:justify-between w-10/12 ml-2 md:w-full h-full">
		<div>
			<!-- name -->
			<div
			@click="goToProductPage"
			>
				<span
					class="text-lg font-semibold leading-none md:text-xl"
				>
					{{ product.name }} 
				</span>
			</div>
			<!-- eof name -->
			<!-- description -->
			<div
			v-if="product.description"
			class="mt-1 leading-4 md:leading-5"
			>
				<span class="text-[0.8rem] text-[#5c6370] md:text-[0.9rem]">
					{{ product.description }}
				</span>
			</div>
			<!-- eof description -->
		</div>

		<div>
		<!-- addcart -->
			<!-- mobile add cart -->
			<div class="mt-3 md:hidden">
				<Button
					@button-click="addCartClick"
					v-if="!isInCart"
					:title="getProductPrice(product) + ' &#8381;'"
					rounded="full"
					class="py-[0.4rem] font-medium px-8 text-defaultText bg-default bg-opacity-10 text-[14px]"
				/>

				<add-quantity-input
					v-if="isInCart"
					@add="addQuantityClick"
					@remove="removeQuantityClick"
					:quantity="cartItem ? cartItem.quantity : null"
				/>

			</div>	
			<!-- eof mobile add cart -->
			<!-- desktop add cart -->
			<div class="items-center justify-between hidden mt-4 md:flex">
				<div class="text-xl font-semibold">
					 {{ getProductPrice(product) }} &#8381;
				</div>
				<Button
				@button-click="addCartClick"
				v-if="!isInCart"
				title="В корзину"
				rounded="full"
				size="large"
				class="font-medium bg-default bg-opacity-10 text-defaultText"
				/>

				<add-quantity-input 
					v-if="isInCart"
					@add="addQuantityClick"
					@remove="removeQuantityClick"
					:quantity="cartItem ? cartItem.quantity : null"
				/>

			</div>
			<!-- eof desktop add cart -->
		<!-- eof addcart -->
		</div>

	</div>
	<!-- eof details -->
	<Icon
	/>

</div>
</template>

<script lang="ts">

import { defineComponent, PropType, computed } from 'vue';
import { Icon } from '@iconify/vue';
import ProductInterface from '@/interfaces/ProductInterface';
import Button from '@/components/buttons/Button.vue';
//import LazyImage from '@/components/image/LazyImage.vue';
import AddQuantityInput from '@/components/input/AddQuantityInput.vue';

// delete below code
import { useRouter } from 'vue-router';

export default defineComponent({
	name: "ProductCard",
	components: {
		Icon,
		Button,
		AddQuantityInput,
//		LazyImage,
	},
	props: {
		product: {
			type: Object as PropType<ProductInterface>, // change to product interface Object type?
			default: null,
		},
		/*
		isInCart: {
			type: Boolean,
			default: false, 
		},
		*/
		cartItem: {
			type: Object,
			default: null,
		},
	},
	emits: ['add-to-cart', 'remove-from-cart', 'add-item-quantity', 'remove-item-quantity', 'open-product-modal'],

	setup (props, { emit }) {	
		const router = useRouter()

		// computed
		const isInCart = computed(() => props.cartItem == null ? false : true);
		// functions
		var goToProductPage  = () => router.push('/product/' + props.product.id)
		// var goToProductPage  = () => emit('open-product-modal', props.product.id)
		var removeFromCartClick = () => emit('remove-from-cart', props.cartItem)
		var addCartClick = () => emit('add-to-cart', props.product)
		var addQuantityClick = () => emit('add-item-quantity', props.cartItem)
		var removeQuantityClick = () => emit('remove-item-quantity', props.cartItem)
		const getProductPrice = (product: Record<string,any>) => {
			if (product.sale_price && product.sale_price > 0) {
				return product.sale_price
			}
			return product.price
		}
		return {
			// computed
			isInCart,
			// functions
			getProductPrice,
			addCartClick,
			removeFromCartClick,
			addQuantityClick,
			removeQuantityClick,
			goToProductPage,
		}
	}
});
</script>
