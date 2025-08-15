<template>
    <div>
        <div class="boletas-grid">
            <button v-for="b in boletas" :key="b.numero" class="boleta" :style="estiloBoleta(b.estado)"
                @click="abrirModal(b)">
                {{ b.numero }}
            </button>
        </div>

        <!-- Modal Datos Boleta -->
        <div class="modal fade" id="modalBoleta" tabindex="-1" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header border-0">
                        <h5 class="modal-title">Datos de Boleta {{ seleccion?.numero }}</h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                    </div>
                    <div class="modal-body pt-0">
                        <!-- Alerta de error -->
                        <div v-if="errorBoleta" class="alert alert-danger py-2">
                            {{ errorBoleta }}
                        </div>

                        <input v-model.trim="form.nombre" class="form-control mb-2"
                            placeholder="Ingrese nombre del comprador" />

                        <input v-model.trim="form.direccion" class="form-control mb-2"
                            placeholder="Ingrese dirección del comprador" />

                        <input v-model.trim="form.telefono" class="form-control mb-3"
                            type="text"
                            placeholder="Ingrese número telefónico"
                            @input="form.telefono = form.telefono.replace(/[^0-9]/g, '')" />

                        <select v-model="form.estado" class="form-select">
                            <option disabled value="">Seleccione estado</option>
                            <option value="Disponible">Disponible</option>
                            <option value="Apartado">Apartado</option>
                            <option value="Pagado">Pagado</option>
                        </select>
                    </div>
                    <div class="modal-footer border-0">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">
                            Cancelar
                        </button>
                        <button type="button" class="btn btn-primary" @click="guardarBoleta">
                            Guardar
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'InfoTalonario',
    props: {
        boletas: { type: Array, required: true }
    },
    data() {
        return {
            seleccion: null,
            form: { nombre: '', direccion: '', telefono: '', estado: '' },
            errorBoleta: ''
        }
    },
    methods: {
        estiloBoleta(estado) {
            const map = {
                Pagado: { bg: '#9D00FF80', border: '#9D00FF80', fg: '#4a224f' },
                Apartado: { bg: '#FFA6009D', border: '#e68a00', fg: '#4a2a00' },
                Disponible: { bg: '#e5e7eb', border: '#c7cbd1', fg: '#333333' }
            }
            const c = map[estado] || map.Disponible
            return { background: c.bg, borderColor: c.border, color: c.fg }
        },
        abrirModal(boleta) {
            this.seleccion = boleta
            this.form = { ...boleta }
            this.errorBoleta = ''
            const m = new bootstrap.Modal(document.getElementById('modalBoleta'), { backdrop: 'static' })
            m.show()
        },
        guardarBoleta() {
            if (!this.form.nombre.trim()) {
                this.errorBoleta = 'El nombre es obligatorio';
                return;
            }
            if (!this.form.direccion.trim()) {
                this.errorBoleta = 'La dirección es obligatoria';
                return;
            }
            if (!this.form.telefono.trim()) {
                this.errorBoleta = 'El número telefónico es obligatorio';
                return;
            }
            if (!this.form.estado.trim()) {
                this.errorBoleta = 'Debe seleccionar un estado';
                return;
            }
            if (this.form.estado === 'Disponible') {
                this.errorBoleta = 'Debe cambiar el estado a "Apartado" o "Pagado" para guardar';
                return;
            }

            this.errorBoleta = '';
            const actualizado = { ...this.seleccion, ...this.form }
            this.$emit('actualizar-boleta', actualizado)
            const modal = bootstrap.Modal.getInstance(document.getElementById("modalBoleta"));
            modal.hide();
        }
    }
}
</script>

<style scoped>
.boletas-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(45px, 1fr));
    gap: 10px;
    justify-content: center;
}

.boleta {
    aspect-ratio: 1/1;
    min-width: 45px;
    border-radius: 50%;
    border: 3px solid #b3b3b3;
    font-weight: 700;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 2px 0 rgba(0, 0, 0, .12);
    transition: transform .08s ease;
    font-size: 0.9rem;
}

.boleta:hover {
    transform: translateY(-2px);
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
  .boletas-grid {
    gap: 8px;
  }

  .boleta {
    min-width: 40px;
    font-size: 0.85rem;
  }
}

@media (max-height: 600px) {
  .modal-content {
    max-height: 90vh;
    overflow-y: auto;
  }
}
</style>

