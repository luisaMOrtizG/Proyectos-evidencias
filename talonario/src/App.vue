<template>
  <div class="app">
    <header class="topbar" @click="irAPaginaPrincipal" style="cursor: pointer;">
      TALONARIO
    </header>

    <!-- Alerta -->
    <div v-if="alerta" class="alert alert-success rounded-3 w-75 mx-auto mt-3">
      {{ alerta }}
      <button type="button" class="btn-close float-end" @click="alerta=''" />
    </div>

    <div class="container-xl my-4">
      <div class="row g-4">
        <!-- Col izquierda -->
        <div class="col-lg-3">
          <Formulario
            v-if="!mostrandoFormulario && configuracionOK"
            :datos="configuracion"
            @editar="editar"
          />
          <div v-else class="placeholder-panel panel"></div>
        </div>

        <!-- Col centro -->
        <div class="col-lg-6">
          <!-- FORM -->
          <div v-if="mostrandoFormulario" class="card form-card shadow-sm">
            <div class="form-card-header">
              CONFIGURA TU TALONARIO
              <button class="btn-close btn-close-white float-end" @click="cancelarForm" />
            </div>
            <div class="card-body">
              <div v-if="error" class="alert alert-danger py-2">{{ error }}</div>

              <input v-model.trim="configuracion.premio" class="form-control mb-3" placeholder="Premio" />
              <input v-model.number="configuracion.valor" type="number" min="1" class="form-control mb-3" placeholder="Valor de la boleta" />
              <select v-model="configuracion.loteria" class="form-select mb-3">
                <option value="">Seleccione la lotería</option>
                <option>Cruz Roja</option>
                <option>La Perla</option>
                <option>Santander</option>
                <option>Baloto</option>
              </select>
              <select v-model="configuracion.rango" class="form-select mb-3">
                <option value="">Seleccione un rango</option>
                <option value="100">0 - 99 (100 boletas)</option>
                <option value="1000">0 - 999 (1000 boletas)</option>
              </select>
              <input v-model="configuracion.fecha" type="date" class="form-control mb-3" :min="fechaMinima" />
              <button class="btn btn-primary w-100 py-2" @click="guardarConfiguracion">Guardar</button>
            </div>
          </div>

          <!-- GRID -->
          <div v-else>
            <InfoTalonario
              :boletas="boletas"
              @actualizar-boleta="onActualizarBoleta"
            />
          </div>
        </div>

        <!-- Col derecha -->
        <div class="col-lg-3">
          <div class="panel actions-panel text-center">
            <h4 class="panel-title">Acciones</h4>
            <div class="panel-body d-grid gap-3">
              <button class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#modalListado">🧾 LISTAR BOLETAS</button>
              <button class="btn btn-outline-primary" @click="personalizar">⚙️ PERSONALIZAR</button>
              <button class="btn btn-outline-danger mt-2" @click="reiniciar">♻️ REINICIAR</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- MODAL LISTADO -->
    <div class="modal fade" id="modalListado" tabindex="-1" aria-hidden="true">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">

          <!-- Encabezado -->
          <div class="modal-header" style="background-color: #0056b3; color: white;">
            <h5 class="modal-title">Listado de Boletas</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
          </div>

          <!-- Cuerpo -->
          <div class="modal-body">
            <table class="table table-bordered text-center">
              <thead>
                <tr style="background-color: #d4af37; color: white;">
                  <th>N° Boleta</th>
                  <th>Nombre Comprador</th>
                  <th>Dirección</th>
                  <th>Teléfono</th>
                  <th>Fecha Compra</th>
                  <th>Estado</th>
                  
                </tr>
              </thead>
              <tbody>
                <tr v-for="b in boletas" :key="b.numero">
                  <td>{{ b.numero }}</td>
                  <td>{{ b.nombre }}</td>
                  <td>{{ b.direccion }}</td>
                  <td>{{ b.telefono }}</td>
                  <td>{{ b.fecha || '' }}</td>
                  <td>{{ b.estado }}</td>
                  
                </tr>
              </tbody>
            </table>

            <!-- Totales -->
            <table class="table table-bordered">
              <tbody>
                <tr>
                  <td>Dinero recaudado: $ {{ totalRecaudado.toLocaleString() }}</td>
                </tr>
                <tr>
                  <td>Deuda total: $ {{ deudaTotal.toLocaleString() }}</td>
                </tr>
              </tbody>
            </table>

            <!-- Botón generar -->
            <div class="text-center">
              <button class="btn btn-primary" @click="generarArchivo">
                <i class="bi bi-download"></i> GENERAR ARCHIVO
              </button>
            </div>
          </div>

        </div>
      </div>
    </div>

    <footer class="footer">Copyright ©2024. Todos los derechos reservados.</footer>
  </div>
</template>

<script>
import Formulario from './components/Formulario.vue'
import InfoTalonario from './components/infotalo.vue'
import jsPDF from "jspdf"
import "jspdf-autotable"

const LS_KEY = 'talonario.v1'

export default {
  name: 'App',
  components: { Formulario, InfoTalonario },
  data() {
    const hoy = new Date(); 
    hoy.setDate(hoy.getDate() + 1)
    return {
      mostrandoFormulario: true,
      configuracion: { premio:'', valor:null, loteria:'', rango:'', fecha:'' },
      boletas: [],
      alerta: '',
      error: '',
      fechaMinima: hoy.toISOString().split('T')[0],
    }
  },
  computed: {
    configuracionOK() {
      const c = this.configuracion
      return c.premio && c.valor && c.loteria && c.rango && c.fecha
    },
    totalRecaudado() {
      return this.boletas
        .filter(b => b.estado === "Pagado")
        .reduce((acc, b) => acc + (this.configuracion.valor || 0), 0)
    },
    deudaTotal() {
      return this.boletas
        .filter(b => b.estado !== "Pagado")
        .reduce((acc, b) => acc + (this.configuracion.valor || 0), 0)
    }
  },
  mounted() {
    const raw = localStorage.getItem(LS_KEY)
    if (raw) {
      const { configuracion, boletas } = JSON.parse(raw)
      this.configuracion = configuracion
      this.boletas = boletas || []
      this.mostrandoFormulario = !this.configuracionOK
    }
  },
  methods: {
    irAPaginaPrincipal() {
      this.mostrandoFormulario = true
    },
    guardarConfiguracion() {
      const c = this.configuracion
      if (!c.premio) return this.error = 'Ingresa el premio'
      if (!c.valor || c.valor <= 0) return this.error = 'Valor de boleta inválido'
      if (!c.loteria) return this.error = 'Selecciona lotería'
      if (!c.rango) return this.error = 'Selecciona rango'
      if (!c.fecha) return this.error = 'Selecciona fecha'

      this.error = ''
      const n = parseInt(c.rango, 10)
      this.boletas = Array.from({ length: n }, (_, i) => ({
        numero: i,
        nombre: '',
        direccion: '',
        telefono: '',
        fecha: '',
        estado: 'Disponible'
      }))

      this.persistir()
      this.alerta = 'Registro exitoso'
      this.mostrandoFormulario = false
    },
    editar() { this.mostrandoFormulario = true },
    cancelarForm() { this.mostrandoFormulario = false },
    onActualizarBoleta(bActualizada) {
      const idx = this.boletas.findIndex(b => b.numero === bActualizada.numero)
      if (idx !== -1) this.boletas.splice(idx, 1, bActualizada)
      this.persistir()
    },
   generarArchivo() {
  if (!this.boletas.length) {
    alert("No hay boletas para exportar.");
    return;
  }

  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();

  doc.setFontSize(16);
  doc.text("Listado de Boletas", 14, 15);

  doc.autoTable({
    head: [["N° Boleta", "Nombre Comprador", "Dirección", "Teléfono", "Fecha Compra", "Estado"]],
    body: this.boletas.map(b => [
      b.numero,
      b.nombre || "",
      b.direccion || "",
      b.telefono || "",
      b.fecha || "",
      b.estado
    ]),
    startY: 20
  });

  const totalY = doc.lastAutoTable.finalY + 10;
  doc.text(`Dinero recaudado: $${this.totalRecaudado.toLocaleString()}`, 14, totalY);
  doc.text(`Deuda total: $${this.deudaTotal.toLocaleString()}`, 14, totalY + 8);

  doc.save("boletas.pdf");
},


    personalizar() {},
    reiniciar() {
      localStorage.removeItem(LS_KEY)
      this.mostrandoFormulario = true
      this.configuracion = { premio:'', valor:null, loteria:'', rango:'', fecha:'' }
      this.boletas = []
      this.alerta = ''
    },
    persistir() {
      localStorage.setItem(LS_KEY, JSON.stringify({
        configuracion: this.configuracion,
        boletas: this.boletas
      }))
    }
  }
}
</script>




<style>
:root{
  --primary: #6f2dbd;
  --primary-700:#5a22a0;
  --accent:  #ff7eb6;
}

.app { 
  background:#ffffff; 
  min-height:100vh; 
  display:flex; 
  flex-direction:column; 
}

.topbar{
  background: linear-gradient(90deg, var(--primary), var(--accent));
  color:#fff; font-weight:800; letter-spacing:.06em; font-size:2.2rem;
  text-align:center; padding:12px 0;
}

.footer{
  margin-top:auto; background: linear-gradient(90deg, var(--primary), var(--accent));
  color:white; text-align:center; padding:12px; font-size:.95rem;
}

.panel { background:#eeeef2; border-radius:18px; padding:18px; }
.panel-title { font-weight:800; color:#2e2a31; margin:0 0 12px 0; }
.panel-body { background:#ffffff; border-radius:16px; padding:22px; }

.form-card { border-radius:18px; overflow:hidden; }
.form-card-header{
  background: var(--primary);
  color:white; font-weight:800; text-align:center; padding:12px 16px;
}

.btn-primary{ background: var(--primary); border:none; }
.btn-primary:hover{ background: var(--primary-700); }
.btn-outline-primary{ color:var(--primary); border-color:var(--primary); }
.btn-outline-primary:hover{ background:var(--primary); color:white; }

/* ===== RESPONSIVIDAD ===== */
@media (max-width: 992px) {
  .col-lg-3, .col-lg-6 {
    flex: 0 0 100%;
    max-width: 100%;
  }
}

@media (max-width: 768px) {
  .topbar {
    font-size: 1.5rem;
    padding: 8px 0;
  }

  .panel {
    padding: 14px;
  }

  .form-card-header {
    font-size: 1rem;
  }

  .actions-panel .btn {
    width: 100%;
  }

  /* Hacer tabla de boletas scrollable */
  .table-responsive {
    overflow-x: auto;
  }
}
</style>

