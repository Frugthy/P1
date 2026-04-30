<template>
  <div class="overlay" @click.self="$emit('cancelar')">
    <div class="modal-admin">
      <div class="modal-header">
        <h2>{{ editandoId ? '✏️ Editar mueble' : '➕ Nuevo mueble' }}</h2>
        <button class="modal-close" @click="$emit('cancelar')">✕</button>
      </div>
      <div class="admin-form">
        <div class="fgroup">
          <label>Nombre</label>
          <input v-model="form.nombre" placeholder="Ej: Sofá moderno">
        </div>
        <div class="fgroup">
          <label>Categoría</label>
          <select v-model="form.categoria">
            <option v-for="c in categorias.slice(1)" :key="c">{{ c }}</option>
          </select>
        </div>
        <div class="fgroup">
          <label>Precio ($)</label>
          <input v-model.number="form.precio" type="number">
        </div>
        <div class="fgroup">
          <label>Stock</label>
          <input v-model.number="form.stock" type="number">
        </div>
        <div class="fgroup full">
          <label>Descripción</label>
          <textarea v-model="form.descripcion"></textarea>
        </div>
        <div class="fgroup full">
          <label>URL de imagen (opcional)</label>
          <input v-model="form.imagen" placeholder="https://...">
        </div>
      </div>
      <div class="modal-actions">
        <button class="btn-cerrar" @click="$emit('cancelar')">Cancelar</button>
        <button class="btn-agregar" @click="handleGuardar">💾 Guardar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AdminForm',
  props: {
    editandoId: { type: Number, default: null },
    categorias: { type: Array, required: true },
    formInicial: { type: Object, required: true }
  },
  emits: ['guardar', 'cancelar'],
  data() {
    return { form: { ...this.formInicial } }
  },
  watch: {
    formInicial(val) { this.form = { ...val } }
  },
  methods: {
    handleGuardar() {
      this.$emit('guardar', { ...this.form })
    }
  }
}
</script>
