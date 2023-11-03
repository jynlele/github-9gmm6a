<template>
  <Layout>
    <div>
      <div style="margin: 1rem 0">
        <PiniaLogo />
      </div>

      <h2 v-if="isShowHello">Hello {{ user.name }}</h2>

      <form v-on:submit.prevent="addItemToCart" data-testid="add-items">
        <input type="text" v-model="itemName" />
        <button>Add</button>
      </form>

      <button @click="onChangeToShow">showHello</button>

      <div>{{ itemName }}</div>

      <form @submit.prevent="buy">
        <ul data-testid="items">
          <li v-for="item in cart.items" :key="item.name">
            {{ item.name }} ({{ item.amount }})
            <button @click="cart.removeItem(item.name)" type="button">X</button>
          </li>
        </ul>

        <button :disabled="!user.name || cart.items.length === 0">Buy</button>
        <button
          :disabled="!cart.items.length"
          @click="clearCart"
          type="button"
          data-testid="clear"
        >
          Clear the cart
        </button>
      </form>
    </div>
  </Layout>
</template>

<script lang="ts">
import Layout from './layouts/default.vue';
import PiniaLogo from './components/PiniaLogo.vue';

import { defineComponent, ref } from 'vue';
import { useUserStore } from './stores/user';
import { useCartStore } from './stores/cart';

export default defineComponent({
  components: { Layout, PiniaLogo },

  props: {
    book: String,
    isShowHello: Boolean,
  },
  emits: [{e:'click', isShowHello: Boolean}],
  setup(props, { emit }) {
    const user = useUserStore();
    const cart = useCartStore();

    const itemName = ref('');

    // const isShowHello = ref(false);

    function onChangeToShow() {
      emit('change');

      // console.log('isShow hello', props.isShowHello);
      // isShowHello.value = !isShowHello.value;
    }

    function addItemToCart() {
      console.log('itemName', itemName.value);
      if (!itemName.value) return;
      cart.addItem(itemName.value);
      itemName.value = '';
    }

    function clearCart() {
      if (window.confirm('Are you sure you want to clear the cart?')) {
        cart.rawItems = [];
      }
    }

    async function buy() {
      const n = await cart.purchaseItems();

      console.log(`Bought ${n} items`);

      cart.rawItems = [];
    }

    // @ts-ignore
    window.stores = { user, cart };

    return {
      itemName,
      addItemToCart,
      cart,
      onChangeToShow,

      user,
      buy,
      clearCart,
    };
  },
  mounted() {
    const user = useUserStore();
    console.log(user.name);
    user.name = 'Tester';
    const isShowHello = ref(true);
  },
});
</script>

<style scoped>
img {
  width: 200px;
}

button,
input {
  margin-right: 0.5rem;
  margin-bottom: 0.5rem;
}
</style>
