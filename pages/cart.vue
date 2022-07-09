<template>
    <div v-if="!pending" class="flex flex-col md:flex-row justify-center items-start mx-20 space-x-20 mt-10">
        <div class="w-[500px] h-full">
            <h1 class="text-2xl font-bold border-b pb-4 border-gray-300 mb-6">Shopping Cart</h1>
            <div
                v-for="cart in carts"
                class="mb-6 h-40 flex justify-start items-start space-x-6 border-b pb-6 border-gray-300"
            >
                <div class="h-full w-1/3" :style="`background-color: ${cart.colors.name}`">.</div>
                <div class="flex flex-col flex-1">
                    <p class="text-lg font-semibold">{{ cart.colors.name }}</p>
                    <p class="text-lg font-semibold">${{ cart.colors.price }}.00</p>
                    <div class="flex items-center justify-between mt-5 w-full">
                        <div class="flex items-center space-x-3">
                            <button
                                @click="minCount(cart)"
                                class="w-6 h-6 rounded-lg flex justify-center items-center bg-gray-200"
                            >
                                -
                            </button>
                            <p>{{ cart.count }}</p>
                            <button
                                @click="addCount(cart)"
                                class="w-6 h-6 rounded-lg flex justify-center items-center bg-gray-200"
                            >
                                +
                            </button>
                        </div>
                        <p class="text-lg font-semibold">${{ cart.count * cart.colors.price }}.00</p>
                    </div>
                </div>
            </div>
        </div>
        <div class="w-[300px] h-full p-5 bg-gray-100 rounded-md mt-10">
            <h1 class="text-xl font-semibold mb-4">Order Summary</h1>
            <div class="flex justify-between items-center mb-5">
                <p>Subtotal</p>
                <p>${{ subtotal }}.00</p>
            </div>
            <div class="flex justify-between items-center mb-5">
                <p>Taxes</p>
                <p>${{ subtotal / 10 }}.00</p>
            </div>
            <hr />
            <div class="flex justify-between items-center mb-5">
                <p>Total</p>
                <p>${{ subtotal + subtotal / 10 }}.00</p>
            </div>
            <button class="px-4 py-2 rounded-md bg-black text-white w-full">Checkout</button>
        </div>
    </div>
</template>

<script setup lang="ts">
import { Cart } from "~~/types/types";

definePageMeta({
    middleware: ["auth"],
});

const client = useSupabaseClient();
const user = useSupabaseUser();

const pending = ref<boolean>(false);
const carts = ref<Cart[]>([]);

pending.value = true;
const { data } = await client.from<Cart>("carts").select(`*, colors(*)`).match({ user_id: user.value.id });
carts.value = [];
carts.value.push(...data);
pending.value = false;

// const { data: carts, pending } = await useAsyncData("carts", async () => {
//     const { data } = await client.from<Cart>("carts").select(`*, colors(*)`);
//     return data;
// });
// console.log(carts.value);

const subtotal = computed(() => {
    let total = 0;
    carts.value.forEach((c) => {
        total += c.count * c.colors.price;
    });
    return total;
});

const updateCart = async (cart: Cart) => {
    await client
        .from<Cart>("carts")
        .update({
            count: cart.count,
        })
        .eq("id", cart.id);
};

const addCount = async (cart: Cart) => {
    cart.count++;
    await updateCart(cart);
};

const minCount = async (cart: Cart) => {
    cart.count--;
    if (cart.count == 0) {
        await client.from<Cart>("carts").delete().eq("id", cart.id);
        carts.value = carts.value.filter((c) => c.id != cart.id);
    } else {
        await updateCart(cart);
    }
};
</script>