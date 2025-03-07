<template>
<div class="px-4 mx-auto my-5">
	<div class="text-2xl font-semibold">
		Профиль
	</div>
	
	<div class="max-w-[500px]">
	<!-- user name -->
		<div class="mt-4">
			<user-profile-input
				:canChange="true"
				:title="'Имя'"
				:inputValue="user.name"
				@save-value="updateUserName"	
			/>
		</div>
	<!-- eof user name -->
	<!-- user phone -->
		<div class="mt-4">
			<user-profile-input
				:canChange="false"
				:title="'Номер телефона'"
				:inputValue="user.username"
			/>
		</div>
	<!-- eof user phone -->
	<!-- user email -->
		<div class="mt-4">
			<user-profile-input
				:canChange="true"
				:title="'Электронная почта'"
				:inputValue="user.email"
				@save-value="updateUserEmail"
			/>
		</div>
	<!-- eof user email -->

	<!-- user change password button -->
	<div 
	@click="setUserChangePasswordModal(true)"
	class="flex items-center px-2 py-2 mt-6 cursor-pointer select-none rounded-xl text-defaultText bg-defaultDim">
		<Icon
			icon="fluent:fingerprint-24-filled"
			width="35"
		/>
		<div class="ml-1">
			Сменить пароль
		</div>
	</div>
	<!-- eof user change password button -->
	
	<!-- user logout button -->
	<div 
	@click="logoutUserClicked"
	class="flex items-center px-2 py-2 mt-5 cursor-pointer select-none rounded-xl text-defaultText bg-defaultDim">
		<Icon
			icon="fe:logout"
			width="35"
		/>
		<div class="ml-1">
			Выйти с аккаунта
		</div>
	</div>
	<!-- eof user logout button -->
	</div>

</div>

	<user-change-password-modal
		v-if="user_change_password_open"
		:userLoginInfo="user_login_info"
		@close-modal="setUserChangePasswordModal(false)"
		@change-password="changeUserPassword"
	/>
	
</template>

<script lang="ts">
import { defineComponent, computed } from 'vue';
import { useRouter } from 'vue-router';
import { useStore } from 'vuex';
import { Icon } from '@iconify/vue';
import { createToast } from 'mosha-vue-toastify';
// local components
import UserProfileInput from '@/components/input/UserProfileInput.vue';
import UserChangePasswordModal from '@/components/modals/UserChangePasswordModal.vue';

import { maska } from 'maska';

export default defineComponent({
	name: "UserProfilePagePersonalData",
	components: {
		UserProfileInput,
		Icon,
		UserChangePasswordModal,
	},
	setup(props) {
		// toasts
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
		// eof toasts
		const store = useStore()
		const router = useRouter()
		// computed
		// user

		const user = computed(() => store.state.user.user);


		const user_change_password_open = computed(() => store.state.modals.user_change_password_open)
		const user_login_info = computed(() => store.state.user.user_login_info)

		// functions
		// call update user login info
		const logoutUserClicked = async () => {
			await store.dispatch("logoutUser")
			router.push('/')
		}

		
		// call update user name dispatch
		const updateUserName = async (new_user_name: string) => {
			const is_updated = await store.dispatch('updateUserInfoAPI', {"name": new_user_name})
			if (is_updated) {
				successToast("Данные успешно изменены")
			} else {
				errorToast("Возникла ошибка во время изменения данных")
			}
		}
		// call update user email dispatch
		const updateUserEmail = async (new_user_email: string) => {
			const is_updated = await store.dispatch('updateUserInfoAPI', {"email": new_user_email})
			if (is_updated) {
				successToast("Данные успешно изменены")
			} else {
				errorToast("Возникла ошибка во время изменения данных")
			}
		}
		// call open restore user password modal
		const setUserChangePasswordModal = (is_open: boolean) => {
			// commit to open user restore password modal
			if (!is_open) {
				store.commit('resetUserLoginInfo')
			}
			store.commit('modals/setUserChangePasswordOpen', is_open)
		}
		// change user password function 
		const changeUserPassword = async () => {
			// validate, if passwords inputs are correct		
			const validate_info: Record<string,any> = await store.dispatch('validateCheckRegister')
			if (!validate_info.is_valid) {
				return errorToast(validate_info.v_msg)
			} else {
				// can dispatch action on serve to change password
				const password_changed = await store.dispatch("changeUserPasswordAPI")
				if (password_changed) {
					setUserChangePasswordModal(false)
					successToast("Ваш пароль успешно изменен 🔥")
				} else {
					errorToast("Возникла ошибка при смене пароля")
				}
			}
		}

		return {
			// computed
			user,

			user_change_password_open,
			user_login_info,
			// functions
			logoutUserClicked,
			updateUserName,
			updateUserEmail,
			setUserChangePasswordModal,
			changeUserPassword,
		}
	}
});
</script>
