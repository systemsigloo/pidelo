<template>
  <div>
  <ProductItem
      v-for="product in products"
      :key="product.id"
      :product="product"
      @addToCart="handleAddToCart"
    />
    <div v-if="loading">Cargando productos...</div>
    <div v-else class="productos-grid">
      <div v-for="producto in productos" :key="producto.id" class="producto-card">
        <img :src="producto.foto" alt="Foto del producto" class="producto-img" />
        <h2>{{ producto.nombre }}</h2>
        <p>{{ producto.descripcion }}</p>
        <p><strong>Precio: </strong> Bs {{ producto.precio }}</p>
        <input type="number" min="0" v-model.number="producto.cantidad" placeholder="Cantidad" />
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

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
</script>

<style scoped>
.productos-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}
.producto-card {
  border: 1px solid #ddd;
  padding: 10px;
  border-radius: 8px;
  width: 200px;
  text-align: center;
}
.producto-img {
  width: 100%;
  height: auto;
  margin-bottom: 10px;
}
</style>
