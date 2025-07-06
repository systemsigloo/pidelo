<script setup>

import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import MenuCategorias from '@/components/categoria.vue';
const apiUrl = import.meta.env.VITE_API_URL;
// Lista de productos
const productos = ref([]);

// Carrito
const carrito = ref([]);
const mostrarCarrito = ref(false);
// Cargar productos desde API
const obtenerProductos = async () => {
  try {
    const response = await axios.get(apiUrl+'/productos');
    productos.value = response.data;
  } catch (error) {
    console.error('Error al obtener productos:', error);
  }
};
const categoriaSeleccionada = ref(null);
const productosFiltrados = computed(() => {
  if (categoriaSeleccionada.value === null) {
    return productos.value;
  } else {
    console.log("computed: " + categoriaSeleccionada.value);
    return productos.value.filter(producto => producto.categoria_id === categoriaSeleccionada.value);
  }
});

// Agregar producto al carrito
const agregarAlCarrito = (producto) => {
  const item = carrito.value.find(p => p.id === producto.id);
  if (item) {
    item.cantidad++;
  } else {
    carrito.value.push({ ...producto, cantidad: 1 });
  }
};

// Quitar producto del carrito
const quitarDelCarrito = (producto) => {
  carrito.value = carrito.value.filter(p => p.id !== producto.id);
};
const totalItemsCarrito = () => {
  return carrito.value.reduce((total, item) => {
    return total + item.cantidad;
  }, 0);
};
// Obtener total
const totalCarrito = () => {
  return carrito.value.reduce((total, item) => {
    return total + item.precio * item.cantidad;
  }, 0);
};

onMounted(() => {
  obtenerProductos();
});

const submitOrder = () => {
  abrirModal();
  const orderDetails = carrito.value.map(item => `${item.nombre} x${item.cantidad}`).join(', ');
  console.log('Pedido enviado:', orderDetails);
};

const filtrarPorCategoria = (categoria) => {
  console.log('Filtrando por:', categoria.id);
  categoriaSeleccionada.value = (categoria.id);
  console.log(categoriaSeleccionada.value);
};
const mostrarModal = ref(false);

const nombre = ref('');
const prefijos = ['0414', '0424', '0416', '0426', '0412', '0422'];
const prefijo = ref('');
const numero = ref('');
const comentarios = ref('');
// Errores
const nombreError = ref(false);
const prefijoError = ref(false);
const numeroError = ref(false);
const abrirModal = () => {
  mostrarModal.value = true;
};

const cerrarModal = () => {
  mostrarModal.value = false;
};

const finalizarPedido = async () => {
    if (!validarCampos()) {
    return;
  }
  const telefonoCompleto = `${prefijo.value}-${numero.value}`;
  console.log('Nombre:', nombre.value);
  console.log('Teléfono:', telefonoCompleto);
  console.log('Comentarios:', comentarios.value);
  console.log('Comprobante:', comprobante.value ? comprobante.value.name : 'Ninguno');
  
  const formData = new FormData();
  formData.append('nombre', nombre.value);
  formData.append('telefono', telefonoCompleto);
  formData.append('comentarios', comentarios.value);
  formData.append('productos', JSON.stringify(carrito.value.map(item => ({
    producto_id: item.id,
    nombre: item.nombre,
    cantidad: item.cantidad,
    precio_unitario: item.precio,
    subtotal: item.cantidad * item.precio
  }))));
  formData.append('total', '0');
  if (comprobante.value) {
    formData.append('comprobante', comprobante.value);
  }

  try {
    const response = await axios.post(apiUrl+'/pedidos', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    });
    if (response.status === 200) {
      alert('Pedido enviado con éxito');
      carrito.value = [];
    }
  } catch (error) {
    console.error('Error al enviar el pedido:', error);
    alert('Hubo un error al enviar el pedido');
  }
 
  cerrarModal();
  nombre.value = '';
  prefijo.value = '';
  numero.value = '';
  comentarios.value = '';
  comprobante.value = null;
  carrito.value = [];
  mostrarCarrito.value = false;
};

defineExpose({
  abrirModal,
});

const validarCampos = () => {
  nombreError.value = nombre.value.trim() === '';
  prefijoError.value = prefijo.value === '';
  numeroError.value = numero.value.trim().length !== 7;
  comprobanteError.value = !comprobante.value;
  return !nombreError.value && !prefijoError.value && !numeroError.value && !comprobanteError.value;
};

// Modal de detalle de producto
const mostrarModalDetalle = ref(false);
const productoSeleccionado = ref(null);

const abrirModalDetalle = (producto) => {
  productoSeleccionado.value = producto;
  mostrarModalDetalle.value = true;
};

const cerrarModalDetalle = () => {
  mostrarModalDetalle.value = false;
  productoSeleccionado.value = null;
};

// Nuevas variables para comprobante
const comprobante = ref(null);
const comprobanteError = ref(false);

const handleComprobanteChange = (event) => {
  comprobante.value = event.target.files[0];
  comprobanteError.value = false;
};
</script>

<template>
  <div class="contenedor-principal">
    <MenuCategorias @categoria-seleccionada="filtrarPorCategoria" />

    <div class="catalogo">
      <div v-if="productos.length === 0" class="loading">Cargando productos...</div>
      <div v-else class="product-grid">
        <div class="card-producto" v-for="producto in productosFiltrados" :key="producto.id">
          <div class="producto-imagen" @click="abrirModalDetalle(producto)">
            <img :src="'https://restgoodfriends.com/back-end/public' + producto.imagen" :alt="producto.nombre" />
          </div>
          <div class="producto-info">
            <h3>{{ producto.nombre }}</h3>
            <div class="precio-boton">
              <p class="precio">$ {{ producto.precio }}</p>
              <button class="boton-agregar" @click="agregarAlCarrito(producto)">+</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="carrito-panel" :class="{ abierto: mostrarCarrito }">
      <div class="carrito-header">
        <h2><button @click="mostrarCarrito = false">X</button> Mi Carrito </h2>
      </div>
      <div class="carrito-contenido">
        <div v-if="carrito.length === 0">Tu carrito está vacío</div>
        <div v-else>
          <div v-for="item in carrito" :key="item.id" class="carrito-item">
            <h4>{{ item.nombre }} (x{{ item.cantidad }})</h4>
            <p>Precio: $ {{ item.precio }}</p>
            <button @click="quitarDelCarrito(item)">Quitar</button>
          </div>
        </div>
      </div>
      <div class="carrito-footer">
        <div class="total">
          <h3>Total: $ {{ totalCarrito() }}</h3>
        </div>
        <button class="btn-enviar" @click="submitOrder">Confirmar</button>
      </div>
    </div>

    <button class="boton-carrito" @click="mostrarCarrito = true">
      🛒 {{ totalItemsCarrito() }} <br> $ {{ totalCarrito() }}
    </button>

    <div v-if="mostrarModal" class="modal-overlay">
      <div class="modal-contenido">
        <h2>Finalizar Pedido</h2>
        <form @submit.prevent="finalizarPedido">
          <div class="form-group">
            <label for="nombre">Nombre:</label>
            <input
              id="nombre"
              v-model="nombre"
              type="text"
              required
              :class="{ 'input-error': nombreError }"
            />
          </div>
          <div class="form-group">
            <label for="telefono">Teléfono:</label>
            <div class="telefono-input">
              <select
                v-model="prefijo"
                required
                :class="{ 'input-error': prefijoError }"
              >
                <option value="" disabled>Prefijo</option>
                <option v-for="pre in prefijos" :key="pre" :value="pre">{{ pre }}</option>
              </select>
              <input
                type="tel"
                maxlength="7"
                v-model="numero"
                pattern="[0-9]{7}"
                placeholder="XXXXXXX"
                required
                :class="{ 'input-error': numeroError }"
              />
            </div>
          </div>
          <div class="form-group">
            <label for="comentarios">Comentarios:</label>
            <textarea id="comentarios" v-model="comentarios" rows="3"></textarea>
          </div>
          <div class="form-group">
            <label>Método de Pago:</label>
            <div class="metodo-pago">
              <p><strong>Banco:</strong> Provincial</p>
              <p><strong>Teléfono:</strong> 0414 9845031</p>
              <p><strong>CI:</strong> 14606203</p>
            </div>
          </div>
          <div class="form-group">
            <label for="comprobante">Comprobante de Pago:</label>
            <input
              id="comprobante"
              type="file"
              accept="image/*"
              @change="handleComprobanteChange"
              :class="{ 'input-error': comprobanteError }"
            />
          </div>
          <div class="modal-footer">
            <button type="button" class="btn-cancelar" @click="cerrarModal">Cancelar</button>
            <button type="submit" class="btn-finalizar">Finalizar Pedido</button>
          </div>
        </form>
      </div>
    </div>

    <div v-if="mostrarModalDetalle" class="modal-detalle-overlay">
      <div class="modal-detalle-contenido">
        <h2>{{ productoSeleccionado?.nombre }}</h2>
        <div class="detalle-imagen">
          <img :src="'https://restgoodfriends.com/back-end/public' + productoSeleccionado?.imagen" :alt="productoSeleccionado?.nombre" />
        </div>
        <p class="detalle-precio">$ {{ productoSeleccionado?.precio }}</p>
        <p class="detalle-descripcion">{{ productoSeleccionado?.descripcion }}</p>
        <div class="modal-detalle-footer">
          <button class="btn-cancelar" @click="cerrarModalDetalle">Cerrar</button>
          <button class="boton-agregar2" @click="agregarAlCarrito(productoSeleccionado); cerrarModalDetalle()">+ Agregar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600&display=swap');

.contenedor-principal {
  padding: 0.5rem;
  background-color: #ffffff;
  font-family: 'Poppins', sans-serif;
  color: #000000;
}

.loading {
  text-align: center;
  padding: 1rem;
  font-size: 1.1rem;
}

.catalogo {
  padding: 0.5rem;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.5rem;
}

.card-producto {
  background-color: #ffffff;
  border-radius: 8px;
  padding: 0.5rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.producto-imagen {
  width: 140px;
  height: 140px;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #ddd;
  margin-bottom: 0.5rem;
  cursor: pointer;
}

.producto-imagen img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.producto-info {
  text-align: center;
  width: 100%;
}

.producto-info h3 {
  font-size: 1rem;
  font-weight: 600;
  margin: 0.3rem 0;
}

.precio-boton {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.3rem;
}

.precio {
  font-size: 0.9rem;
  font-weight: 500;
}

.boton-agregar {
  width: 32px;
  height: 32px;
  border: none;
  background-color: #28a745;
  color: white;
  font-size: 1.2rem;
  border-radius: 50%;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.boton-agregar2 {
  background-color: #28a745;
  color: white;
  font-size: 1.2rem;
  transition: background-color 0.2s ease;
  flex: 1;
  padding: 0.5rem;
  border: none;
  border-radius: 5px;
  font-weight: 500;
  cursor: pointer;
}

.boton-agregar:hover,
.boton-agregar2:hover {
  background-color: #218838;
}

.boton-carrito {
  position: fixed;
  top: 0.5rem;
  right: 0.5rem;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 50px;
  padding: 0.5rem 0.75rem;
  font-size: 1rem;
  cursor: pointer;
  z-index: 1000;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.carrito-panel {
  position: fixed;
  top: 0;
  Peugeot: -100%;
  width: 80%;
  max-width: 320px;
  height: 100%;
  background-color: #ffffff;
  border-left: 1px solid #ccc;
  box-shadow: -3px 0 6px rgba(0, 0, 0, 0.2);
  display: flex;
  flex-direction: column;
  transition: right 0.3s ease;
  z-index: 999;
  font-family: 'Poppins', sans-serif;
}

.carrito-panel.abierto {
  right: 0;
}

.carrito-header {
  padding: 0.5rem;
  border-bottom: 1px solid #ddd;
  background: #f8f9fa;
}

.carrito-header h2 {
  margin: 0;
  font-size: 1.25rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.carrito-contenido {
  flex: 1;
  overflow-y: auto;
  padding: 0.5rem;
}

.carrito-item {
  border-bottom: 1px solid #eee;
  padding: 0.3rem 0;
  margin-bottom: 0.3rem;
}

.carrito-item h4 {
  font-size: 1rem;
  margin: 0 0 0.3rem;
}

.carrito-item p {
  font-size: 0.9rem;
  margin: 0 0 0.3rem;
}

.carrito-item button {
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 0.3rem;
  border-radius: 4px;
  cursor: pointer;
}

.carrito-footer {
  padding: 0.5rem;
  border-top: 1px solid #ddd;
  background: #f8f9fa;
}

.total {
  margin-bottom: 0.3rem;
  font-weight: 600;
}

.btn-enviar {
  width: 100%;
  padding: 0.5rem;
  background-color: #ffc107;
  color: #000000;
  border: none;
  border-radius: 5px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s;
}

.btn-enviar:hover {
  background-color: #e0a800;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-contenido {
  background: #ffffff;
  padding: 1rem;
  width: 90%;
  max-width: 400px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  font-family: 'Poppins', sans-serif;
}

.modal-contenido h2 {
  margin-bottom: 0.5rem;
  text-align: center;
  font-size: 1.25rem;
}

.form-group {
  margin-bottom: 0.5rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.3rem;
  font-weight: 500;
}

.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 0.3rem;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-family: 'Poppins', sans-serif;
}

.form-group input[type="file"] {
  padding: 0.2rem;
}

.input-error {
  border-color: #dc3545;
}

.metodo-pago {
  background-color: #f8f9fa;
  padding: 0.5rem;
  border-radius: 5px;
  border: 1px solid #ddd;
}

.metodo-pago p {
  margin: 0.2rem 0;
  font-size: 0.9rem;
}

.telefono-input {
  display: flex;
  gap: 0.3rem;
}

.telefono-input select {
  width: 40%;
}

.telefono-input input {
  width: 60%;
}

.modal-footer {
  display: flex;
  justify-content: space-between;
  gap: 0.5rem;
  margin-top: 0.5rem;
}

.btn-cancelar {
  flex: 1;
  padding: 0.5rem;
  background: #6c757d;
  color: white;
  border: none;
  border-radius: 5px;
  font-weight: 500;
  cursor: pointer;
}

.btn-finalizar {
  flex: 1;
  padding: 0.5rem;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 5px;
  font-weight: 500;
  cursor: pointer;
}

.btn-finalizar:hover {
  background: #218838;
}

.modal-detalle-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-detalle-contenido {
  background: #ffffff;
  padding: 1rem;
  width: 90%;
  max-width: 400px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  font-family: 'Poppins', sans-serif;
  text-align: center;
}

.modal-detalle-contenido h2 {
  margin-bottom: 0.5rem;
  font-size: 1.25rem;
}

.detalle-imagen {
  width: 200px;
  height: 200px;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #ddd;
  margin: 0 auto 0.5rem;
}

.detalle-imagen img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.detalle-precio {
  font-size: 1rem;
  font-weight: 500;
  margin: 0.5rem 0;
}

.detalle-descripcion {
  font-size: 0.9rem;
  color: #333333;
  margin: 0.5rem 0;
}

.modal-detalle-footer {
  display: flex;
  justify-content: space-between;
  gap: 0.5rem;
  margin-top: 0.5rem;
}

.modal-detalle-footer .boton-agregar {
  flex: 1;
  width: auto;
  padding: 0.5rem;
  font-size: 1rem;
}
</style>