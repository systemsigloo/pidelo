<template>
  <div style="border: 1px solid #ccc; padding: 10px; margin-bottom: 10px;">
    <img :src="product.image" alt="" style="width: 100px; float: left; margin-right: 10px;" />
    <h4>{{ product.name }}</h4>
    <p>{{ product.description }}</p>
    <p><strong>Precio: Bs {{ product.price }}</strong></p>

    <div>
      <label for="quantity">Cantidad:</label>
      <input type="number" v-model.number="quantity" min="1" style="width: 60px;" />
      <button @click="addToCart">Agregar al Pedido</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const props = defineProps({
  product: Object,
});

const emit = defineEmits(['addToCart']);

const quantity = ref(1);

const addToCart = () => {
  emit('addToCart', props.product, quantity.value);
  quantity.value = 1;
};
</script>
