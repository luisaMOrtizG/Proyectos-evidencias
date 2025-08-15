<template>
  <div class="panel info-panel">
    <h4 class="panel-title">Información del Talonario</h4>

    <div class="panel-body">
      <p class="item"><span>🏆</span> <strong>{{ datos.premio }}</strong></p>
      <p class="item"><span>💵</span> $ {{ Number(datos.valor).toLocaleString() }}</p>
      <p class="item"><span>🏛️</span> {{ datos.loteria }}</p>
      <p class="item"><span>📅</span> {{ datos.fecha }}</p>

      <button class="btn btn-primary w-100 mt-3" @click="$emit('editar')">
        Editar ✏️
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Formulario',
  props: {
    datos: { type: Object, required: true }
  }
}
</script>

<style scoped>
.panel {
  background: #f5f5f7;
  border-radius: 18px;
  padding: 18px;
  box-shadow: 0 1px 0 rgba(0, 0, 0, .06) inset;
}

.panel-title {
  font-weight: 800;
  color: #2e2a31;
  margin-bottom: 12px;
}

.panel-body {
  background: #ffffff;
  border-radius: 16px;
  padding: 16px;
}

.item {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 1.05rem;
  margin: 14px 0;
}

.btn-primary {
  background: var(--primary);
  border: none;
}

.btn-primary:hover {
  background: var(--primary-700);
}

/* ===== RESPONSIVIDAD ===== */
@media (max-width: 768px) {
  .panel {
    padding: 14px;
  }

  .panel-title {
    font-size: 1.1rem;
  }

  .item {
    font-size: 0.95rem;
    margin: 10px 0;
  }
}
</style>

