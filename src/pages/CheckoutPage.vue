<template>
<div class="bg-[#f3f3f7]">
<!-- main checkout page container -->
<div class="px-4 py-4 mx-auto max-w-screen-lg">
	<div class="text-2xl font-medium">
		Оформление заказа
		</div>

	<!-- select delivery method -->
	<div class="mt-4 text-lg font-medium">
		Способ доставки
	</div>
	<div class="mt-2 max-w-[500px]">
		<input-select-main
			:selectValues="delivery_methods"
			:activeValue="checkout_info.delivery_method"
			@select-value="updateDeliveryMethod"
		/>
	</div>
	<!-- eof select delivery method -->
	
    <!-- guest user name -->
    <div
		class="py-1 pl-4 mt-6 bg-white rounded-lg max-w-[600px]"
    >
        <input
            v-model="checkout_info.guest_name"
            placeholder="Ваше имя"
            class="w-full px-3 py-3 rounded-lg outline-none"
        />
    </div>
    <!-- eof guest user name -->

	<!-- set user phone, if use if guest -->
	<div 
		class="py-1 pl-4 mt-2 bg-white rounded-lg max-w-[600px]"
	>
		<input
			@maska="updateGuestPhone"
			v-maska="'+7 ###-###-##-##'"
			v-model="checkout_info.guest_phone_display"
			placeholder="Ваш номер телефона"
			class="w-full px-3 py-3 rounded-lg outline-none"
		/>
	</div>
	<!-- eof set guest user phone -->

	<!-- select address and time -->
	<div 
	class="py-1 pl-4 mt-2 bg-white rounded-lg max-w-[600px]"
	>
		<div v-if="checkout_info.delivery_method == 'delivery'">
		<!-- select delivery address -->
			<select-checkout-main
				v-if="user_info && user_module_enabled"
				@click="openCheckoutModal('choose_delivery_address_modal_open',true)"
				:dataNew="checkout_info.delivery_address == undefined"
				:showIconSelected="true"
				:titleNew="'Выберите адрес доставки'"
				:title="getCheckoutDeliveryAddressDisplay"
			/>

		<!-- eof select delivery address -->


		<div v-else>
			<input
				:value="checkout_info.guest_delivery_address"
				@input="updateGuestDeliveryAddress"
				id="address_input"
				placeholder="Укажите адрес доставки"
				class="w-full px-3 py-3 rounded-lg outline-none"
			/>
		</div>

		</div>
<!-- select pickup address -->
			<select-checkout-main
				v-if="checkout_info.delivery_method == 'pickup'"
				@click="openCheckoutModal('choose_pickup_address_modal_open',true)"
				:dataNew="checkout_info.pickup_address == undefined"
				:showIconSelected="true"
				:titleNew="'Выберите пункт выдачи'"
				:title="getCheckoutPickupAddressDisplay"
			/>
		<!-- eof select pickup address -->
	</div>
	<!-- eof select address and time -->

	<!-- choose payment methods -->
	<div class="mt-4 text-lg font-medium">
		Способ оплаты
	</div>
	<div class="pl-4 py-1 mt-3 bg-white max-w-[600px] rounded-lg">
		<!-- select payment method -->
		<!--
				@click="openCheckoutChoosePaymentMethod"
		-->
			<select-checkout-main
				@click="openCheckoutModal('choose_payment_method_modal_open', true)"
				:dataNew="checkout_info.payment_method == undefined"
				:showIconSelected="true"
				:titleNew="'Выберите способ оплаты'"
				:title="getCheckoutPaymentMethodDisplay"
			/>
		<!-- eof select payment method -->
	</div>
	<!-- eof choose payment method -->

	<!-- make order button -->
	<div class="mt-8">
		<Button
			@button-click="makeOrder"
			:title="'Оформить заказ'"
			rounded="full"
			class="flex items-center justify-center px-5 min-h-[45px] text-white bg-default max-w-[400px]"
		/>
	</div>
	<!-- eof make order button -->

</div>
	<!-- eof main checkout page container -->
</div>


<!-- modals -->
	<!-- choose delivery address modal -->
	<checkout-choose-delivery-address-modal
		v-if="checkout_modals.choose_delivery_address_modal_open"
		:addressList="user_delivery_addresses"
		:activeAddress="checkout_info.delivery_address"
		@delivery-address="updateDeliveryAddress"
		@delete-delivery-address="deleteDeliveryAddress"
		@close-modal="openCheckoutModal('choose_delivery_address_modal_open',false)"
		@open-create-delivery="openCheckoutModal('create_delivery_address_modal_open',true)"
	/>
	<create-delivery-address-modal
		v-if="checkout_modals.create_delivery_address_modal_open"
		@close-modal="openCheckoutModal('create_delivery_address_modal_open',false)"
		@create-delivery-address="createDeliveryAddress"
	/>
	<!-- eof choose delivery address modal -->
	<!-- choose pickup address modal -->
	<checkout-choose-pickup-address-modal
		v-if="checkout_modals.choose_pickup_address_modal_open"
		:addressList="pickup_addresses"
		:activeAddress="checkout_info.pickup_address"
		@close-modal="openCheckoutModal('choose_pickup_address_modal_open',false)"
		@pickup-address="updatePickupAddress"	
	/>
	<!-- eof choose pickup address modal -->
	<!-- choose payment method modal -->
	<checkout-choose-payment-method-modal
		v-if="checkout_modals.choose_payment_method_modal_open"
		:paymentMethods="payment_methods"
		:activePayment="checkout_info.payment_method"
		@close-modal="openCheckoutModal('choose_payment_method_modal_open',false)"
		@payment-method="updatePaymentMethod"
	/>
	<!-- eof choose payment methods modal -->
<!-- modals -->
</template>

<script lang="ts">
import { computed, defineComponent, onBeforeMount, ref} from 'vue';
import { useStore } from 'vuex';
import { useRouter } from 'vue-router';

import { createToast } from 'mosha-vue-toastify';
import { maska } from 'maska';
// local components
import Button from '@/components/buttons/Button.vue';
import InputSelectMain from '@/components/input/InputSelectMain.vue';
import SelectCheckoutMain from '@/components/input/SelectCheckoutMain.vue';

import CheckoutChooseDeliveryAddressModal from '@/components/checkout/CheckoutChooseDeliveryAddressModal.vue';
import CheckoutChoosePickupAddressModal from '@/components/checkout/CheckoutChoosePickupAddressModal.vue';
import CreateDeliveryAddressModal from '@/components/checkout/CreateDeliveryAddressModal.vue';
import CheckoutChoosePaymentMethodModal from '@/components/checkout/CheckoutChoosePaymentMethodModal.vue';


// validators 
import { validateCreateDeliveryAddress } from '@/validators/checkout';

export default defineComponent({
	name: "CheckoutPage",
	components: {
		Button,
		InputSelectMain,
		SelectCheckoutMain,
		CheckoutChooseDeliveryAddressModal,
		CheckoutChoosePickupAddressModal,
		CheckoutChoosePaymentMethodModal,
		CreateDeliveryAddressModal,
	},
	directives: {
		maska,
	},
	setup () {
	onBeforeMount(async () => {
		if (!delivery_methods.value || !pickup_addresses.value || !payment_methods.value) {
			await store.dispatch("cart/getCheckoutCommonInfoAPI")
		}
		if (!user_delivery_addresses.value) {
			await store.dispatch("getUserDeliveryAddressAPI")
		}
	});
	// toast 
	const inputErrorToast = (title: string) => {
		createToast(
			title, {
				'type': 'danger',
				'showIcon': true,
				'hideProgressBar': true,	
			}
		);
	};
	const inputSuccessToast = (title: string) => {
		createToast(
			title, {
				'type': 'success',
				'showIcon': true,
				'hideProgressBar': true,
			}
		);
	};
	var errorToast = (title: string) => inputErrorToast(title)
	var successToast =  (title: string) => inputSuccessToast(title)


	const store = useStore()
	const router = useRouter()
	// check if user module enabled
	const user_module_enabled = computed(() => store.state.user.module_enabled)
	//  use_authorized
	const user_authorized = computed(() => store.state.user.user_authorized);
	// computed
	const user_delivery_addresses = computed(() => store.state.user.delivery_addresses)
	// get current user info
	const user_info = computed(() => store.state.user.user)
	// available delivery methods
	const delivery_methods = computed(() => store.state.checkout.delivery_methods)
	// available pick up addresses
	const pickup_addresses = computed(() => store.state.checkout.pickup_addresses)
	// available payment methods
	const payment_methods = computed(() => store.state.checkout.payment_methods)
	// common checkout_info, that user choosed
	const checkout_info = computed(() => store.state.checkout.checkout_info)
	// checkout modals open state
	const checkout_modals = computed(() => store.state.checkout.modals)
	// functions
	const updateDeliveryMethod = (new_delivery_method:string) => store.commit('checkout/setCheckoutInfoDeliveryMethod', new_delivery_method)


	// set checkout delivery address to checkout_info
	const updateDeliveryAddress = (address: Record<string,any>) => { 
		store.commit("checkout/setCheckoutInfoDeliveryAddress", address)
		// close the modal
		// openCheckoutModal('choose_delivery_address_modal_open',false)
	}
	const updateGuestDeliveryAddress = (event: Record<string,any>) => { 
		const address = event.target.value
		store.commit("checkout/setCheckoutInfoGuestDeliveryAddress", address)
	}
	const updateGuestPhone = (event: Record<string,any>) => { 
        var phone_raw_value = event.target.getAttribute('data-mask-raw-value')
        if (phone_raw_value.trim().length > 0) {
            checkout_info.value['guest_phone'] = '7' + phone_raw_value
        } else {
            checkout_info.value['guest_phone'] = null
        }
	}

	const deleteDeliveryAddress = async (address_id: string) => { 
		const is_deleted = await store.dispatch("deleteUserDeliveryAddressAPI", address_id)
		if (is_deleted) {
			return null
		} else {
			return errorToast("Возникла ошибка во время удаления адреса")
		}
	}
	// set checkout pickup address to checkout_info
	const updatePickupAddress = (address: Record<string,any>) => { 
		store.commit("checkout/setCheckoutInfoPickupAddress", address)
		// close the modal
		openCheckoutModal('choose_pickup_address_modal_open',false)
	}
	// set new payment_method to checkout_info
	const updatePaymentMethod = (payment_method: Record<string,any>) => { 
		store.commit("checkout/setCheckoutInfoPaymentMethod", payment_method)
		// close the modal
		openCheckoutModal('choose_payment_method_modal_open',false)
	}
	// make API call and create delivery address
	const createDeliveryAddress = async (new_delivery_address: Record<string,any>) => {
		// need to validate form
		const validate_info = validateCreateDeliveryAddress(new_delivery_address)
		if (!validate_info.is_valid) {
			errorToast(validate_info.v_msg)
		} else {
			const is_created = await store.dispatch('createUserDeliveryAddressAPI', new_delivery_address)
			if (is_created) {
				successToast('Адрес успешно добавлен')
				openCheckoutModal('create_delivery_address_modal_open',false)
			} else {
				errorToast("Возникла ошибка при добавлении адреса")
			}
		}
	}

	// helper to get checkout delivery address display
	var getCheckoutDeliveryAddressDisplay = computed(() => {
		if (checkout_info.value.delivery_address != null) {
			return checkout_info.value.delivery_address.address_display
		} else {
			return ''
		}
	});
	// helper to get checkout pickup address display
	var getCheckoutPickupAddressDisplay = computed(() => {
		if (checkout_info.value.pickup_address != null) {
			return checkout_info.value.pickup_address.name
		} else {
			return ''
		}
	});
	// helper to get checkout pickup address display
	var getCheckoutPaymentMethodDisplay = computed(() => {
		if (checkout_info.value.payment_method != null) {
			return checkout_info.value.payment_method.name
		} else {
			return ''
		}
	});
	// set open state to specified checkout modal
	var openCheckoutModal = (modal_name: string, is_open: boolean) => {
		store.commit('checkout/setCheckoutModalOpen', { modal_name, is_open })
	}
	// make order function 
	const makeOrder = async () => {
		// need to validate, that order could be make
        const valid_res = await store.dispatch('checkout/validateCheckout')
        if (!valid_res?.can_make_order) {
            return errorToast(valid_res?.msg)
        }
		// 1. Check, if user authorized, if not - show authorization popup
		// 2. Check, if user selected delivery method, delivery/pickup address, show error Toast msg if not
		// 3. Check, if user has enough cart items, to make purchase for delivery (enough emount)
		// 4. Submit order, if all validation passed
		if (user_info.value && user_module_enabled.value) {
			const is_created = await store.dispatch('cart/createOrderAPI')
			if (is_created) { 
				await store.dispatch('getUserOrdersAPI')
				successToast("Заказ успешно создан!");
				// load user orders
				// route user to the profile page
				return router.push("/profile/orders");
			}  
			return errorToast("Возникла ошибка во время создания заказа")
		} else {
			// need to create guest order
			const is_created = await store.dispatch('cart/createGuestOrderAPI')
			if (is_created) { 
				successToast("Заказ успешно создан!");
				return router.push("/");
			}  
			return errorToast("Возникла ошибка во время создания заказа")
		}
	}
		return {
			// computed
			user_authorized,
			user_delivery_addresses,
			user_info,
			delivery_methods,
			checkout_info,
			checkout_modals,

			pickup_addresses,
			payment_methods,
			// functions
			deleteDeliveryAddress,
			updateDeliveryMethod,
			updateGuestDeliveryAddress,
            updateGuestPhone,
			updateDeliveryAddress,
			updatePickupAddress,
			updatePaymentMethod,

			createDeliveryAddress,
			
			openCheckoutModal,

			getCheckoutDeliveryAddressDisplay,
			getCheckoutPickupAddressDisplay,
			getCheckoutPaymentMethodDisplay,

			makeOrder,
		}
	}
});
</script>

