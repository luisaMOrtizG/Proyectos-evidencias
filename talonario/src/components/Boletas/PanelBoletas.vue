<template>
  <div class="text-center">
    <h5 class="fw-bold mb-3">Listado de Boletas (0 - {{ cantidad - 1 }})</h5>
    <div class="d-flex flex-wrap justify-content-center gap-2">
      <button
        v-for="numero in cantidad"
        :key="numero"
        class="btn btn-outline-primary rounded-circle"
        style="width: 50px; height: 50px"
      >
        {{ numero - 1 }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    cantidad: {
      type: Number,
      required: true
    }
  }
}
</script>
