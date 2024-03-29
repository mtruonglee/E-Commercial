<template>
  <div
    class="bg-gradient-to-r from-slate-200 from-10% via-slate-300 via-30% to-slate-200 to-90% px-28"
  >
    <h4 class="text-black font-medium pt-3">Pay</h4>
    <div class="grid grid-cols-6 gap-4 py-2">
      <div class="col-span-3 bg-white p-3 rounded flex items-center">
        <div class="">
          <div
            class="flex flex-col rounded-lg sm:flex-row items-center"
            v-for="cart in carts"
            :key="cart._id"
          >
            <img
              class="m-2 h-24 w-28 rounded-md border object-cover object-center"
              :src="cart.product.image"
              alt=""
            />
            <div class="flex w-full flex-col px-4 py-4">
              <span class="font-semibold">{{ cart.product.title }}</span>
              <span class="float-right text-gray-400"
                >Quantity: {{ cart.quantity }}</span
              >
              <p class="text-lg font-bold text-red-600">
                {{ (cart.price * cart.quantity).toLocaleString() }}
                <u class="no-underline">VND</u>
              </p>
            </div>
          </div>
        </div>
        <!--
          <div class="pt-5">
            <h4 class="font-medium">Select a payment method</h4>
            <div class="bg-gray-100 rounded py-2 mt-1 px-4">
              <div class="py-2">
                <input type="checkbox" id="" name="" value="" />
                <label for=""> Payment on delivery</label>
              </div>
              <div class="py-2">
                <input type="checkbox" id="" name="" value="" />
                <label for=""> Payment by paypal</label>
              </div>
            </div>
          </div>
        -->
      </div>
      <div class="col-span-3 bg-slate-300 p-3 rounded">
        <div v-if="user">
          <div class="flex">
            <strong class="pr-3 font-semibold">Recipient's name:</strong>
            <span class="capitalize text-black font-extrabold pr-6">
              {{ user.fullName }}
            </span>
            <button class="" @click="handleEditInfo('fullName')">
              <p class="text-blue-800">Change</p>
            </button>
          </div>

          <div class="flex">
            <strong class="pr-3 font-semibold">Phone number:</strong>
            <span class="capitalize text-black font-extrabold pr-6">
              {{ user.phone }}
            </span>
            <button class="" @click="handleEditInfo('phone')">
              <p class="text-blue-800">Change</p>
            </button>
          </div>

          <div class="flex">
            <strong class="pr-3 font-semibold">Delivery address:</strong>
            <span class="capitalize text-black font-extrabold pr-6">
              {{ user.address }}
            </span>
            <button class="" @click="handleEditInfo('address')">
              <p class="text-blue-800">Change</p>
            </button>
          </div>
        </div>
        <div class="border-b-2 border-black"></div>
        <div class="pt-3">
          <div class="flex items-center justify-between">
            <p class="text-sm font-medium text-gray-900">Provisional:</p>
            <p class="font-semibold text-red-600">
              {{
                carts
                  .reduce((acc, cur) => acc + cur.price * cur.quantity, 0)
                  .toLocaleString()
              }}
              <u class="no-underline">VND</u>
            </p>
          </div>
          <div class="flex items-center justify-between">
            <p class="text-sm font-medium text-gray-900">Transport fee:</p>
            <p class="font-semibold text-red-600">
              30,000 <u class="no-underline">VND</u>
            </p>
          </div>
          <div class="mt-6 flex items-center justify-between">
            <p class="text-sm font-medium text-gray-900">Total price:</p>
            <p class="text-2xl font-semibold text-red-600">
              {{
                carts
                  .reduce((acc, cur) => acc + cur.price * cur.quantity, 30000)
                  .toLocaleString()
              }}
              <u class="no-underline">VND</u>
            </p>
          </div>
          <button
            class="mt-4 mb-8 w-full rounded-md bg-gray-500 px-6 py-3 font-medium text-white hover:bg-gray-900"
            :disabled="isPendingSubmiting"
            @click="handleOrder"
          >
            {{ !isPendingSubmiting ? "Order" : "Loading..." }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { authApi } from "@/api/authApi";
import { computed } from "@vue/reactivity";
import { ACCESS_TOKEN_WEB_BLINKBLINK } from "@/utils/settingSystem";
import { useStore } from "vuex";
import { ref } from "vue";
import { cartApi } from "@/api/cartApi";
import { receiptApi } from "@/api/receiptApi";
import { useRouter } from "vue-router";

export default {
  setup() {
    // Hooks
    const store = useStore();
    const router = useRouter();
    //cua tui
    const user = computed(() => store.state.user.user);

    const handleEditInfo = async (field) => {
      let newValue = prompt("Enter new information!");
      if (!newValue) return;
      try {
        // Call api

        // Cần phải cập nhật lại cái accesstoken mới vì thông tin người dùng sau khi cập nhật sẽ được lưu trong access token mới
        const { user, accessToken } = await authApi.updateProfile({
          [field]: newValue,
        });

        localStorage.setItem(ACCESS_TOKEN_WEB_BLINKBLINK, accessToken);

        // Get again data
        store.commit("user/setUser", user);
      } catch (err) {
        console.log(err);
      }
    };
    //ket cua tui
    // Handle get cart for user in component Header

    // Ref
    const loading = ref(false);
    const isPendingSubmiting = ref(false);

    const carts = computed(() => store.getters["carts/cartOrdering"]);
    console.log(carts);
    //const user = computed(() => store.state.user.user);
    //total
    // const subPrice = ref(0);
    // carts.value.map((product) => {
    //   subPrice += product.price * product.quantity;
    // });
    // const total = ref(subPrice.value + 30000);

    // Function global
    const handleQuantityChange = async (data) => {
      // data = {idCart:"",quantity:""}
      if (data.quantity === 0) return;
      // Submit data
      loading.value = true;
      try {
        // Call api
        await cartApi.editCart(data);
        // Update data in vuex store
        store.dispatch("carts/getCarts");
      } catch (err) {
        console.log(err);
      }
      loading.value = false;
    };

    const handleDeleteCart = async (idCart) => {
      if (window.confirm("Remove this product from the cart?")) {
        loading.value = true;
        try {
          // Call api
          await cartApi.deleteCart(idCart);
          // Update data in vuex store
          store.dispatch("carts/getCarts");
        } catch (err) {
          console.log(err);
        }
        loading.value = false;
      }
    };

    const handleOrder = async () => {
      if (window.confirm("Order confirmation?")) {
        isPendingSubmiting.value = true;
        try {
          // Call api order
          await cartApi.order();

          // Call api add receipt
          await receiptApi.addReceipt({ carts: carts.value });

          // Update data in vuex store
          store.dispatch("carts/getCarts");
          //push home
          router.push({ name: "Home" });
        } catch (err) {
          console.log(err);
        }
        isPendingSubmiting.value = false;
      }
    };

    return {
      carts,
      // user,
      //
      user,
      handleEditInfo,
      //
      // subPrice,
      // total,
      loading,
      isPendingSubmiting,
      handleQuantityChange,
      handleDeleteCart,
      handleOrder,

      //

      //
    };
  },
};
</script>

<style></style>
