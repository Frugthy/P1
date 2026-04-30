<template>
  <nav class="navbar">
    <div class="nav-brand">
      <div class="brand-logo-sm">AC</div>
      <div>
        <div class="brand-name">Arboleda Casa</div>
        <div class="brand-sub">MUEBLERÍA PREMIUM</div>
      </div>
    </div>
    <div class="nav-cats">
      <button
        v-for="cat in categorias"
        :key="cat"
        :class="['nav-cat', filtroActivo(cat) ? 'active' : '']"
        @click="$emit('set-categoria', cat)"
      >{{ cat }}</button>
    </div>
    <div class="nav-right">
      <div class="nav-search">
        <span>🔍</span>
        <input
          :value="busqueda"
          @input="$emit('update:busqueda', $event.target.value)"
          placeholder="Buscar mueble..."
        >
      </div>
      <button class="btn-cart" @click="$emit('abrir-carrito')">
        🛒 <span v-if="totalCarrito > 0" class="cart-badge">{{ totalCarrito }}</span>
      </button>
      <button v-if="esAdmin" class="btn-nuevo" @click="$emit('nueva-pieza')">+ Nuevo</button>
      <div class="nav-user" @click.stop="menuUsuario = !menuUsuario">
        <div class="user-avatar">{{ usuario.nombre[0] }}</div>
        <div class="user-info">
          <div class="user-name">{{ nombreCorto }}</div>
          <div :class="['user-rol', usuario.rol]">{{ usuario.rol }}</div>
        </div>
        <span>▾</span>
        <div v-if="menuUsuario" class="user-menu" @click.stop>
          <div class="user-menu-email">{{ usuario.email }}</div>
          <div class="user-menu-item" @click="handleVerPedidos">📦 Mis pedidos</div>
          <div class="user-menu-sep"></div>
          <div class="user-menu-item danger" @click="$emit('cerrar-sesion')">🚪 Cerrar sesión</div>
        </div>
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  name: 'NavBar',
  props: {
    usuario: { type: Object, required: true },
    categorias: { type: Array, required: true },
    filtroCategoria: { type: String, default: '' },
    totalCarrito: { type: Number, default: 0 },
    esAdmin: { type: Boolean, default: false },
    busqueda: { type: String, default: '' }
  },
  emits: ['abrir-carrito', 'set-categoria', 'nueva-pieza', 'ver-pedidos', 'cerrar-sesion', 'update:busqueda'],
  data() {
    return { menuUsuario: false }
  },
  computed: {
    nombreCorto() { return this.usuario.nombre.split(' ')[0] }
  },
  mounted() {
    this._cerrarMenu = () => { this.menuUsuario = false }
    document.addEventListener('click', this._cerrarMenu)
  },
  beforeUnmount() {
    document.removeEventListener('click', this._cerrarMenu)
  },
  methods: {
    filtroActivo(cat) {
      return this.filtroCategoria === (cat === 'Todas' ? '' : cat)
    },
    handleVerPedidos() {
      this.menuUsuario = false
      this.$emit('ver-pedidos')
    }
  }
}
</script>
