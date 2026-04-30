<template>
  <div class="card">
    <div class="card-img-wrap">
      <img :src="imagenUrl" :alt="mueble.nombre" class="card-img" @error="onImgError">
      <span v-if="stockBajo" class="badge-stock">Últimas {{ mueble.stock }}</span>
      <span v-else-if="agotado" class="badge-agotado">Agotado</span>
      <template v-if="esAdmin">
        <button class="card-del" @click.stop="$emit('pedir-eliminar', mueble)">🗑️</button>
        <button class="card-edit" @click.stop="$emit('editar', mueble)">✏️</button>
      </template>
    </div>
    <div class="card-body">
      <div class="card-cat">{{ mueble.categoria }}</div>
      <div class="card-name">{{ mueble.nombre }}</div>
      <div class="card-desc">{{ mueble.descripcion }}</div>
      <div class="card-footer">
        <span class="card-price"><sup>$</sup>{{ mueble.precio.toLocaleString() }}</span>
        <button class="card-ver" @click="$emit('ver-detalle', mueble)">Ver →</button>
      </div>
    </div>
  </div>
</template>

<script>
const IMGS = {
  'Sala': 'https://images.unsplash.com/photo-1555041469-a586c61ea9bc?w=600&q=80',
  'Recámara': 'https://images.unsplash.com/photo-1540518614846-7eded433c457?w=600&q=80',
  'Comedor': 'https://images.unsplash.com/photo-1617806118233-18e1de247200?w=600&q=80',
  'Oficina': 'https://images.unsplash.com/photo-1593642632559-0c6d3fc62b89?w=600&q=80',
}

export default {
  name: 'ProductCard',
  props: {
    mueble: { type: Object, required: true },
    esAdmin: { type: Boolean, default: false }
  },
  emits: ['ver-detalle', 'agregar-carrito', 'editar', 'pedir-eliminar'],
  computed: {
    agotado() { return this.mueble.stock === 0 },
    stockBajo() { return this.mueble.stock <= 3 && this.mueble.stock > 0 },
    imagenUrl() { return this.mueble.imagen || IMGS[this.mueble.categoria] || IMGS['Sala'] }
  },
  methods: {
    onImgError(e) { e.target.src = IMGS['Sala'] }
  }
}
</script>
