<template>
  <div>
      

    <!-- Botón flotante a la izquierda -->
    <button class="floating-menu-btn-left" @click="toggleMenu">
      &#9776; <!-- Icono ☰ -->
    </button>

    <!-- Drawer lateral izquierdo -->
    <div :class="['drawer-left', { 'drawer-open': isOpen }]">
      <button class="close-menu-btn" @click="toggleMenu">X</button>
      <h3>Categorías</h3>
      <ul>
        <li v-for="categoria in categorias"
        :key="categoria.id"
        class="categoria-chip"
        >
          <button class="categoria-btn" @click="seleccionarCategoria(categoria)">
            {{ categoria.nombre }}
          </button>
        </li>
      </ul>
    </div>

    <!-- Overlay oscuro -->
    <div v-if="isOpen" class="overlay" @click="toggleMenu"></div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'CategoriaList',

  data() {
    return {
        isOpen: false,
      categorias: [],
      loading: false,
    };
  },

  methods: {
    toggleMenu() {
      this.isOpen = !this.isOpen;
    },
    async obtenerCategorias() {
      this.loading = true;
      try {
        const apiUrl = import.meta.env.VITE_API_URL;
        const response = await axios.get(apiUrl+'/categorias');
        this.categorias = response.data;
      } catch (error) {
        console.error('Error al obtener categorías:', error);
      } finally {
        this.loading = false;
      }
    },

    seleccionarCategoria(categoria) {
      console.log('Categoría seleccionada:', categoria);
      // Aquí puedes emitir un evento si quieres filtrar productos, por ejemplo:
      this.$emit('categoriaSeleccionada', categoria);
      this.isOpen = !this.isOpen;
    },
  },

  mounted() {
    this.obtenerCategorias();
  },
};
</script>

<style scoped>
/* Botón flotante a la izquierda */
.floating-menu-btn-left {
  position: fixed;
  bottom: 20px;
  left: 20px;
  width: 56px;
  height: 56px;
  background-color: #352c2c;
  color: white;
  font-size: 28px;
  border: none;
  border-radius: 50%;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  cursor: pointer;
  z-index: 1001;
}

/* Drawer lateral izquierdo */
.drawer-left {
  position: fixed;
  top: 0;
  left: -320px;
  width: 280px;
  height: 100%;
  background-color: #420000;
  box-shadow: 2px 0 8px rgba(0, 0, 0, 0.3);
  padding: 20px;
  transition: left 0.3s ease;
  z-index: 1002;
}

.drawer-open {
  left: 0;
}

.close-menu-btn {
  background: transparent;
  border: none;
  font-size: 22px;
  float: right;
  cursor: pointer;
  margin-bottom: 20px;
}

.categoria-btn {
  display: block;
  width: 100%;
  padding: 10px;
  background-color: #4d0000;
  color: #9c7f17;
  border: none;
  text-align: left;
  margin-bottom: 8px;
  border-radius: 4px;
  cursor: pointer;
  font-size: large;
  transition: background-color 0.2s;
}

.categoria-btn:hover {
  background-color: #e0e0e0;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.4);
  z-index: 1000;
}
</style>
