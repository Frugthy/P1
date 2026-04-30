<template>
  <div class="overlay" @click.self="$emit('cerrar')">
    <div class="modal-carrito">
      <div class="modal-header">
        <h2>🛒 Mi carrito</h2>
        <button class="modal-close" @click="$emit('cerrar')">✕</button>
      </div>
      <div v-if="!vacio">
        <div class="carrito-lista">
          <div class="carrito-item" v-for="item in carrito" :key="item.id">
            <img :src="getImg(item)" class="ci-img" @error="onImgError">
            <div class="ci-info">
              <div class="ci-name">{{ item.nombre }}</div>
              <div class="ci-price">${{ item.precio.toLocaleString() }} c/u</div>
            </div>
            <div class="qty-ctrl">
              <button @click="$emit('cambiar-qty', { item, delta: -1 })">−</button>
              <span>{{ item.qty }}</span>
              <button @click="$emit('cambiar-qty', { item, delta: 1 })">+</button>
            </div>
            <div class="ci-sub">${{ (item.precio * item.qty).toLocaleString() }}</div>
            <button class="ci-del" @click="$emit('quitar', item.id)">✕</button>
          </div>
        </div>
        <div class="carrito-footer">
          <div class="carrito-total">
            <span>Total</span>
            <span class="total-num">${{ totalPrecio.toLocaleString() }}</span>
          </div>
          <button class="btn-comprar" @click="$emit('iniciar-compra')">Proceder al pago →</button>
        </div>
      </div>
      <div v-else class="empty" style="padding:40px">Tu carrito está vacío</div>
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
  name: 'CartModal',
  props: {
    carrito: { type: Array, required: true },
    totalPrecio: { type: Number, required: true }
  },
  emits: ['cerrar', 'cambiar-qty', 'quitar', 'iniciar-compra'],
  computed: {
    vacio() { return this.carrito.length === 0 }
  },
  methods: {
    getImg(m) { return m.imagen || IMGS[m.categoria] || IMGS['Sala'] },
    onImgError(e) { e.target.src = IMGS['Sala'] }
  }
}
</script>
