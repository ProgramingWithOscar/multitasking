<template>
  <div class="container py-5">
    <div class="text-center mb-5 accion">
      <h1 class="fw-bold text-primary">Bienvenido al Control de Finanzas Personales</h1>
      <p class="text-muted fs-5">Administra tus ingresos y gastos de forma sencilla y eficiente.</p>
    </div>

    <div v-if="!salarioConfirmado" class="alert alert-info d-flex align-items-center gap-3" role="alert">
      <i class="bi bi-cash-coin fs-3 text-primary"></i>
      <div>
        <strong>¡Hola!</strong> Ingresa tu salario base para comenzar.
      </div>
    </div>

    <div v-if="!salarioConfirmado" class="d-flex flex-column flex-md-row mb-4 gap-3 justify-content-center align-items-center">
      <div class="d-flex align-items-center gap-2">
        <label class="form-label m-0 fw-bold">Salario base:</label>
        <input v-model.number="nuevo.salarioBase" class="form-control" type="number" placeholder="Ej. 1500000" />
      </div>
      <button @click="agregarSalarioBase" class="btn btn-primary mt-3 mt-md-0">Confirmar</button>
    </div>

    <div v-if="movimientos.length > 0" class="text-center mb-4">
      <button @click="imprimirResumen" class="btn btn-outline-primary">
        <i class="bi bi-printer-fill"></i> Imprimir resumen
      </button>
    </div>

    <form v-if="salarioConfirmado" @submit.prevent="modoEdicion ? actualizarMovimiento() : agregarMovimiento()" class="row g-3 mb-4">
      <div class="col-12 col-md-4">
        <input type="text" class="form-control" placeholder="Descripción" v-model="nuevo.descripcion" required />
      </div>
      <div class="col-12 col-md-3">
        <input type="number" class="form-control" placeholder="Monto" v-model.number="nuevo.monto" required />
      </div>
      <div class="col-12 col-md-3">
        <select class="form-select" v-model="nuevo.tipo" required>
          <option disabled value="">Seleccionar tipo</option>
          <option value="ingreso">Ingreso</option>
          <option value="gasto">Gasto</option>
        </select>
      </div>
      <div class="col-12 col-md-2">
        <button type="submit" :class="modoEdicion ? 'btn-info' : 'btn-success'" class="btn w-100">
          {{ modoEdicion ? 'Actualizar' : 'Agregar' }}
        </button>
      </div>
    </form>

    <div class="table-responsive mb-4">
      <table v-if="movimientos.length > 0" class="table table-hover align-middle">
        <thead class="table-dark">
          <tr>
            <th class="text-start">Concepto</th>
            <th class="text-end">Monto (COP)</th>
            <th class="text-center">Categoría</th>
            <th class="text-center">Estado</th>
            <th class="accion text-end">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(mov, index) in movimientos" :key="index" :class="{ 'gasto-pagado': mov.tipo === 'gasto' && mov.pagado }">
            <td class="text-start">{{ mov.descripcion }}</td>
            <td class="text-end" :class="mov.tipo === 'gasto' ? 'text-danger' : 'text-success'">
              ${{ mov.monto != null ? mov.monto.toLocaleString() : '0.00' }}
            </td>
            <td class="text-center">
              <span :class="mov.tipo === 'ingreso' ? 'badge bg-success' : 'badge bg-danger'">
                {{ mov.tipo.toUpperCase() }}
              </span>
            </td>
            <td class="text-center">
              <span v-if="mov.tipo === 'gasto'" :class="mov.pagado ? 'badge bg-success' : 'badge bg-warning'">
                {{ mov.pagado ? 'PAGADO' : 'PENDIENTE' }}
              </span>
              <span v-else class="badge bg-secondary">N/A</span>
            </td>
            <td class="accion text-end">
              <div class="d-flex flex-column flex-md-row justify-content-end gap-2">
                <button class="btn btn-sm btn-outline-info" @click="editarMovimiento(index)" v-if="mov.tipo === 'gasto'">
                  <i class="bi bi-pencil-fill"></i> Editar
                </button>
                <button class="btn btn-sm" :class="mov.pagado ? 'btn-outline-warning' : 'btn-outline-success'" @click="alternarPagado(index)" v-if="mov.tipo === 'gasto'">
                  <i :class="mov.pagado ? 'bi-x-circle-fill' : 'bi-check-circle-fill'"></i> {{ mov.pagado ? 'No Pagado' : 'Pagado' }}
                </button>
                <button class="btn btn-sm btn-outline-danger" @click="eliminarMovimiento(index)">
                  <i class="bi bi-trash-fill"></i> Eliminar
                </button>
              </div>
            </td>
          </tr>
          <tr class="table-active fw-bold border-top border-2">
            <td class="text-start">TOTAL</td>
            <td class="text-end" :class="total >= 0 ? 'text-success' : 'text-danger'">
              ${{ total.toLocaleString() }}
            </td>
            <td colspan="3"></td>
          </tr>
        </tbody>
      </table>

      <div v-else class="text-center text-muted p-4 rounded border bg-light">
        <h4>
          Aún no hay registros. Ingresa tu <span class="fw-bold text-primary">salario base</span> para empezar.
        </h4>
        <img class="mt-4" style="width: 300px;" src="https://static-00.iconduck.com/assets.00/9-404-error-illustration-1024x454-1e9ol1ls.png" alt="Sin datos" />
      </div>
    </div>

    <div v-if="movimientos.length > 0" class="row text-center g-3 mb-4">
      <div class="col-12 col-md-4">
        <div class="card shadow-sm border-success">
          <div class="card-body">
            <h5 class="card-title text-success">Ingresos</h5>
            <p class="card-text fs-4 fw-bold">${{ ingresos.toLocaleString() }}</p>
          </div>
        </div>
      </div>
      <div class="col-12 col-md-4">
        <div class="card shadow-sm border-danger">
          <div class="card-body">
            <h5 class="card-title text-danger">Egresos</h5>
            <p class="card-text fs-4 fw-bold">${{ egresos.toLocaleString() }}</p>
          </div>
        </div>
      </div>
      <div class="col-12 col-md-4">
        <div :class="total >= 0 ? 'border-primary' : 'border-danger'" class="card shadow-sm">
          <div class="card-body">
            <h5 class="card-title">Total Disponible</h5>
            <p :class="total >= 0 ? 'text-primary' : 'text-danger'" class="fs-4 fw-bold">
              ${{ total.toLocaleString() }}
            </p>
          </div>
        </div>
      </div>
    </div>

    <div v-if="total <= 0 && movimientos.length > 0" class="alert alert-danger mt-5">
      <h4 class="alert-heading">⚠ Atención</h4>
      <p>Tu balance actual está en rojo o es insuficiente. Revisa tus gastos y ajusta tu presupuesto.</p>
    </div>

    <footer class="text-center mt-5 pt-4 border-top">
      <p class="text-muted">Desarrollado 💻 por <strong>Oscar Poveda</strong></p>
    </footer>

    <div class="print-only">
      <div class="print-header">
        <div class="company-header">
          <h1>REPORTE FINANCIERO</h1>
          <div class="report-info">
            <p class="report-date">Fecha de generación: {{ fechaActual }}</p>
            <p class="report-period">Período: {{ periodoReporte }}</p>
          </div>
        </div>
      </div>

      <div class="executive-summary">
        <h2>RESUMEN EJECUTIVO</h2>
        <div class="summary-grid">
          <div class="summary-item">
            <span class="summary-label">Total Ingresos</span>
            <span class="summary-value positive">${{ ingresos.toLocaleString() }}</span>
          </div>
          <div class="summary-item">
            <span class="summary-label">Total Egresos</span>
            <span class="summary-value negative">${{ egresos.toLocaleString() }}</span>
          </div>
          <div class="summary-item">
            <span class="summary-label">Balance Neto</span>
            <span class="summary-value" :class="total >= 0 ? 'positive' : 'negative'">${{ total.toLocaleString() }}</span>
          </div>
        </div>
      </div>

      <div class="transactions-detail">
        <h2>DETALLE DE MOVIMIENTOS</h2>
        <table class="report-table">
          <thead>
            <tr>
              <th class="text-start">CONCEPTO</th>
              <th class="text-end">MONTO (COP)</th>
              <th class="text-center">CATEGORÍA</th>
              <th class="text-center">ESTADO</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="mov in movimientos" :key="mov.descripcion">
              <td class="text-start">{{ mov.descripcion }}</td>
              <td class="text-end" :class="mov.tipo === 'gasto' ? 'negative' : 'positive'">
                ${{ mov.monto.toLocaleString() }}
              </td>
              <td class="text-center">
                <span class="category-badge" :class="mov.tipo === 'ingreso' ? 'badge-success' : 'badge-danger'">
                  {{ mov.tipo.toUpperCase() }}
                </span>
              </td>
              <td class="text-center">
                <span v-if="mov.tipo === 'gasto'" :class="mov.pagado ? 'badge-success' : 'badge-warning'">
                  {{ mov.pagado ? 'PAGADO' : 'PENDIENTE' }}
                </span>
                <span v-else class="category-badge badge-secondary">N/A</span>
              </td>
            </tr>
          </tbody>
          <tfoot>
            <tr>
              <td class="text-start"><strong>BALANCE TOTAL</strong></td>
              <td class="text-end" :class="total >= 0 ? 'positive' : 'negative'">
                <strong>${{ total.toLocaleString() }}</strong>
              </td>
              <td colspan="2"></td>
            </tr>
          </tfoot>
        </table>
      </div>

      <div class="report-footer">
        <p>Documento generado automáticamente por el Sistema de Control Financiero</p>
        <p class="page-number">Página 1 de 1</p>
      </div>
    </div>

    <div v-if="movimientos.length > 0" class="d-flex justify-content-end mb-4">
      <div class="print-button-container">
        <button @click="imprimirResumen" class="btn btn-primary d-flex align-items-center gap-2">
          <i class="bi bi-file-earmark-text-fill"></i>
          <span>Generar Reporte PDF</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      nuevo: {
        descripcion: '',
        monto: null,
        tipo: '',
        salarioBase: null,
        pagado: false // Nuevo campo para el estado de pago
      },
      salarioConfirmado: false,
      movimientos: [],
      modoEdicion: false, // Nuevo estado para el modo de edición
      indiceEdicion: null // Índice del movimiento que se está editando
    }
  },
  computed: {
    total() {
      // Ajusta el cálculo del total para incluir solo los gastos pagados si es lo que deseas.
      // Si quieres que todos los gastos (pagados y pendientes) se resten del total,
      // entonces simplemente usa `this.egresosTotal` que calcula todos los gastos.
      // Para este caso, el `total` seguirá considerando `egresos` que solo cuenta los pagados.
      return this.ingresos - this.egresos;
    },
    ingresos() {
      return this.movimientos
        .filter(m => m.tipo === 'ingreso')
        .reduce((acc, mov) => acc + mov.monto, 0);
    },
    egresos() {
      // Los egresos que afectan el balance disponible (total) son solo los pagados.
      return this.movimientos
        .filter(m => m.tipo === 'gasto' && m.pagado)
        .reduce((acc, mov) => acc + mov.monto, 0);
    },
    egresosTotal() {
      // Este computed es útil si quieres mostrar el total de *todos* los gastos,
      // independientemente de si están pagados o no, por ejemplo, en un resumen.
      return this.movimientos
        .filter(m => m.tipo === 'gasto')
        .reduce((acc, mov) => acc + mov.monto, 0);
    },
    fechaActual() {
      return new Date().toLocaleDateString('es-CO', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    },
    periodoReporte() {
      const meses = ['Enero', 'Febrero', 'Marzo', 'Abril', 'Mayo', 'Junio', 'Julio', 'Agosto', 'Septiembre', 'Octubre', 'Noviembre', 'Diciembre'];
      const fecha = new Date();
      return `${meses[fecha.getMonth()]} ${fecha.getFullYear()}`;
    }
  },
  methods: {
    agregarMovimiento() {
      if (!this.nuevo.descripcion || this.nuevo.monto == null || !this.nuevo.tipo) return;
      // Por defecto, un nuevo gasto no está pagado
      this.movimientos.push({ ...this.nuevo, pagado: this.nuevo.tipo === 'gasto' ? false : true });
      this.guardarEnLocalStorage();
      this.resetearFormulario();
    },
    agregarSalarioBase() {
      const salarioMinimo = 1300000; // Salario mínimo legal vigente 2024
      if (this.nuevo.salarioBase != null) {
        if (this.nuevo.salarioBase < salarioMinimo) {
          alert(`El salario base no puede ser menor al salario mínimo legal vigente ($${salarioMinimo.toLocaleString()})`);
          return;
        }
        this.movimientos.push({
          descripcion: 'Salario base',
          monto: this.nuevo.salarioBase,
          tipo: 'ingreso',
          pagado: true // El salario base se considera pagado
        });
        this.salarioConfirmado = true;
        this.nuevo.salarioBase = null;
        this.guardarEnLocalStorage();
      }
    },
    eliminarMovimiento(index) {
      this.movimientos.splice(index, 1);
      this.guardarEnLocalStorage();
      this.verificarSalarioConfirmado();
    },
    editarMovimiento(index) {
      this.modoEdicion = true;
      this.indiceEdicion = index;
      const movimientoAEditar = this.movimientos[index];
      this.nuevo.descripcion = movimientoAEditar.descripcion;
      this.nuevo.monto = movimientoAEditar.monto;
      this.nuevo.tipo = movimientoAEditar.tipo;
    },
    actualizarMovimiento() {
      if (!this.nuevo.descripcion || this.nuevo.monto == null || !this.nuevo.tipo) return;

      const movimientoActualizado = { ...this.nuevo };
      // Mantener el estado de 'pagado' si se está editando un gasto
      if (this.movimientos[this.indiceEdicion].tipo === 'gasto') {
        movimientoActualizado.pagado = this.movimientos[this.indiceEdicion].pagado;
      } else {
        // Asegurarse de que los ingresos siempre estén "pagados" (o completados)
        movimientoActualizado.pagado = true;
      }

      this.$set(this.movimientos, this.indiceEdicion, movimientoActualizado);
      this.guardarEnLocalStorage();
      this.resetearFormulario();
      this.modoEdicion = false;
      this.indiceEdicion = null;
    },
    alternarPagado(index) {
      if (this.movimientos[index].tipo === 'gasto') {
        this.movimientos[index].pagado = !this.movimientos[index].pagado;
        this.guardarEnLocalStorage();
      }
    },
    resetearFormulario() {
      this.nuevo.descripcion = '';
      this.nuevo.monto = null;
      this.nuevo.tipo = '';
    },
    guardarEnLocalStorage() {
      localStorage.setItem('movimientos', JSON.stringify(this.movimientos));
    },
    cargarDesdeLocalStorage() {
      const data = localStorage.getItem('movimientos');
      if (data) {
        this.movimientos = JSON.parse(data);
        // Asegurarse de que el campo 'pagado' exista para movimientos antiguos
        this.movimientos.forEach(mov => {
          if (typeof mov.pagado === 'undefined') {
            mov.pagado = mov.tipo === 'ingreso' ? true : false; // Por defecto, ingresos pagados, gastos pendientes
          }
        });
        this.verificarSalarioConfirmado();
      }
    },
    verificarSalarioConfirmado() {
      this.salarioConfirmado = this.movimientos.some(item => item.descripcion === 'Salario base' && item.tipo === 'ingreso');
    },
    imprimirResumen() {
      window.print();
    }
  },
  mounted() {
    this.cargarDesdeLocalStorage();
  }
}
</script>

<style scoped>
body {
  background-color: #f4f6f9;
}
.card {
  border-width: 2px;
}
.print-only {
  display: none;
}

.print-button-container {
  background: #f8f9fa;
  padding: 15px 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  border: 1px solid #e9ecef;
}

.print-button-container .btn-primary {
  background-color: #2c3e50;
  border: none;
  padding: 10px 20px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.print-button-container .btn-primary:hover {
  background-color: #1a252f;
  transform: translateY(-1px);
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}

.print-button-container .bi {
  font-size: 1.2rem;
}

/* Estilo para gastos pagados */
.gasto-pagado {
  background-color: #e6ffe6; /* Un tono verde claro para indicar pagado */
  opacity: 0.8;
}

/* Estilos para el modo de impresión */
@media print {
  body {
    background-color: #fff;
    margin: 0;
    padding: 0;
  }
  .container {
    width: 100%;
    max-width: none;
    padding: 0;
  }
  /* Ocultar elementos no deseados en la impresión */
  .accion,
  .alert,
  .d-flex.mb-4.gap-3,
  .row.g-3.mb-4,
  .table-responsive.mb-4 + .row,
  .alert.mt-5,
  footer,
  .print-button-container,
  .accion.text-end /* Ocultar la columna de acciones en la tabla impresa */
  {
    display: none !important;
  }

  /* Mostrar solo el contenido de impresión */
  .print-only {
    display: block !important;
    font-family: 'Arial', sans-serif;
    color: #333;
    padding: 20px; /* Un poco de padding para el contenido impreso */
  }

  .print-header {
    text-align: center;
    margin-bottom: 30px;
    padding-bottom: 20px;
    border-bottom: 2px solid #eee;
  }
  .print-header h1 {
    color: #2c3e50;
    font-size: 2.5em;
    margin-bottom: 10px;
  }
  .report-info {
    font-size: 0.9em;
    color: #666;
  }
  .executive-summary {
    margin-bottom: 30px;
  }
  .executive-summary h2 {
    color: #2c3e50;
    font-size: 1.8em;
    border-bottom: 1px solid #eee;
    padding-bottom: 10px;
    margin-bottom: 20px;
  }
  .summary-grid {
    display: flex;
    justify-content: space-around;
    gap: 20px;
    flex-wrap: wrap; /* Asegura que los ítems se envuelvan en pantallas pequeñas al imprimir */
  }
  .summary-item {
    flex: 1;
    min-width: 180px; /* Ajuste para evitar que se aplasten demasiado */
    text-align: center;
    padding: 15px;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: #f9f9f9;
  }
  .summary-label {
    display: block;
    font-size: 1em;
    color: #555;
    margin-bottom: 5px;
  }
  .summary-value {
    font-size: 1.5em;
    font-weight: bold;
  }
  .summary-value.positive {
    color: #28a745;
  }
  .summary-value.negative {
    color: #dc3545;
  }

  .transactions-detail {
    margin-bottom: 30px;
  }
  .transactions-detail h2 {
    color: #2c3e50;
    font-size: 1.8em;
    border-bottom: 1px solid #eee;
    padding-bottom: 10px;
    margin-bottom: 20px;
  }
  .report-table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 20px;
  }
  .report-table th, .report-table td {
    border: 1px solid #eee;
    padding: 10px;
    vertical-align: top;
  }
  .report-table thead th {
    background-color: #f2f2f2;
    color: #333;
    font-weight: bold;
  }
  .report-table .text-start { text-align: left; }
  .report-table .text-end { text-align: right; }
  .report-table .text-center { text-align: center; }

  .report-table .positive { color: #28a745; }
  .report-table .negative { color: #dc3545; }
  .category-badge {
    padding: 5px 10px;
    border-radius: 5px;
    color: white;
    font-size: 0.85em;
    font-weight: bold;
  }
  .category-badge.badge-success { background-color: #28a745; }
  .category-badge.badge-danger { background-color: #dc3545; }
  .category-badge.badge-warning { background-color: #ffc107; }
  .category-badge.badge-secondary { background-color: #6c757d; }

  .report-table tfoot strong {
    font-size: 1.1em;
  }

  .report-footer {
    text-align: center;
    font-size: 0.8em;
    color: #999;
    border-top: 1px solid #eee;
    padding-top: 20px;
    margin-top: 30px;
  }
  .page-number {
    margin-top: 10px;
  }
}
</style>