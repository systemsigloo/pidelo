<template>
  <div>
    <ProductItem
      v-for="product in productos"
      :key="product.id"
      :product="product"
      @addToCart="handleAddToCart"
    />

    <div v-if="cart.length > 0" style="margin-top: 20px;">
      <h3>Pedido</h3>
      <ul>
        <li v-for="item in cart" :key="item.id">
          {{ item.name }} x {{ item.quantity }} = Bs {{ item.quantity * item.price }}
        </li>
      </ul>
      <p><strong>Total:</strong> Bs {{ total }}</p>
      <button @click="submitOrder">Enviar Pedido</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import axios from 'axios';
import ProductItem from './ProductItem.vue';
export default {
  name: 'CatalogoProductos',
  data() {
    return {
      productos: [],
      loading: false,
    };
  },
  mounted() {
    this.fetchProductos();
  },
  methods: {
    async fetchProductos() {
      this.loading = true;
      try {
        const response = await axios.get('http://localhost/pedidos-app/public/productos');
        // Asignamos la respuesta a "productos", agregando campo cantidad por defecto
        this.productos = response.data.map(p => ({
          ...p,
          cantidad: 0
        }));
      } catch (error) {
        console.error('Error al obtener productos:', error);
      } finally {
        this.loading = false;
      }
    }
  }
};
const products = ref([
  { id: 1, name: 'Producto 1', description: 'Descripción producto 1', price: 10, image: 'https://via.placeholder.com/150' },
  { id: 2, name: 'Producto 2', description: 'Descripción producto 2', price: 20, image: 'https://via.placeholder.com/150' },
  { id: 3, name: 'Producto 3', description: 'Descripción producto 3', price: 30, image: 'https://via.placeholder.com/150' },
]);

const cart = ref([]);

const handleAddToCart = (product, quantity) => {
  const existing = cart.value.find(item => item.id === product.id);
  if (existing) {
    existing.quantity += quantity;
  } else {
    cart.value.push({
      ...product,
      quantity,
    });
  }
};

const total = computed(() => {
  return cart.value.reduce((sum, item) => sum + item.price * item.quantity, 0);
});

const submitOrder = () => {
  const orderDetails = cart.value.map(item => `${item.name} x${item.quantity}`).join(', ');
  console.log('Pedido enviado:', orderDetails);

  // Aquí luego se enviará al backend (Laravel)
  alert(`Pedido enviado: ${orderDetails}`);

  // Vaciar carrito después de enviar
  cart.value = [];
};
</script>
