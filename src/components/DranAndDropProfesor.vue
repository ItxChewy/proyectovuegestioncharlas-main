<template>
  <div class="container">
    <i class="arrow-icon fa fa-arrow-left ml-3 mt-3" @click="navigateToStudentList"></i>
    <h1 class="text-center text-3xl font-weight-bold">
      Ronda <span class="font-weight-bold">{{ idRonda }}</span>
    </h1>
    <p class="text-center mb-0">Curso {{ charlasPropuestas[0]?.idCurso }}</p>

    <div class="row mt-4">
      <!-- Charlas Propuestas -->
      <div class="col-md-6">
        <div class="card p-4 shadow-sm border">
          <div class="d-flex justify-content-between mb-4">
            <h2 class="h5">Charlas Propuestas</h2>
            <!-- <div
              class="d-flex align-items-center bg-info text-white px-4 py-2 rounded-pill"
            >
              <span class="font-weight-bold">Votos Totales</span>
              <span class="font-weight-semibold h4"
                >{{ votosPropuestos }} / {{ totalVotos }}</span
              >
            </div> -->
          </div>
          <div class="scrollable-content" @dragover.prevent @drop="onDrop($event, 'propuestas')">
            <div v-for="charla in charlasPropuestas" :key="charla.idCharla"
              class="draggable-item d-flex align-items-center p-4 bg-light shadow-sm rounded mb-3" draggable="true"
              @dragstart="onDragStart($event, charla)">
              <img :src="charla.imagenCharla" alt="Imagen Charla" class="w-16 h-16 object-cover rounded-circle mr-3"
                @error="onImageError($event)" />
              <div class="w-100">
                <h3 class="h6 font-weight-bold">{{ charla.titulo }}</h3>
                <div class="d-flex align-items-center">
                  <span :class="{
                    'bg-secondary text-white': charla.idEstadoCharla === 1,
                    'bg-success text-white': charla.idEstadoCharla === 2,
                  }" class="px-2 py-1 text-xs rounded-pill">
                    {{ charla.idEstadoCharla === 1 ? "Propuesta" : "Aceptada" }}
                  </span>
                  <span class="px-2 py-1 text-xs bg-info text-white rounded-pill ml-2">
                    Votos: {{ charla.votos }}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Charlas Aceptadas -->
      <div class="col-md-6">
        <div class="card p-4 shadow-sm border">
          <h2 class="h5 mb-4">Charlas Aceptadas</h2>
          <div class="scrollable-content" @dragover.prevent @drop="onDrop($event, 'aceptadas')">
            <div
              class="draggable-item text-center p-5 bg-light shadow-sm rounded mb-4 border-dashed border-2 border-info"
              @click="acceptCharla">
              <i class="fas fa-plus text-muted" style="font-size: 2rem"></i>
              <p class="text-muted">Arrastra aquí una charla para aceptarla</p>
            </div>
            <div v-for="charla in charlasAceptadas" :key="charla.idCharla"
              class="relative draggable-item d-flex align-items-center p-4 bg-light shadow-sm rounded mb-3"
              draggable="true" @dragstart="onDragStart($event, charla)">
              <div v-if="charla.idEstadoCharla === 2" class="position-absolute top-0 right-0 mt-2 mr-2 text-warning">

              </div>
              <img :src="charla.imagenCharla" alt="Imagen Charla" class="w-16 h-16 object-cover rounded-circle mr-3"
                @error="onImageError($event)" />
              <div class="w-100">
                <h3 class="h6 font-weight-bold">{{ charla.titulo }}</h3>
                <div class="d-flex align-items-center">
                  <span :class="{
                    'bg-secondary text-white': charla.idEstadoCharla === 1,
                    'bg-success text-white': charla.idEstadoCharla === 2,
                  }" class="px-2 py-1 text-xs rounded-pill">
                    {{ charla.idEstadoCharla === 1 ? "Propuesta" : "Aceptada" }}
                  </span>
                  <span class="px-2 py-1 text-xs bg-info text-white rounded-pill ml-2">
                    Votos: {{ charla.votos }}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Botones de acción -->
    <div class="d-flex justify-content-between mt-4">
      <button class="btn btn-warning p-3" @click="abrirConfirmacionCancelar">
        Cancelar cambios
      </button>
      <button class="btn btn-primary p-3" @click="abrirConfirmacionGuardar">
        Guardar cambios
      </button>
    </div>

    <!-- Popup de confirmación -->
    <div v-if="mostrarPopup"
      class="fixed inset-0 bg-dark bg-opacity-75 d-flex align-items-center justify-content-center z-50">
      <div class="bg-white p-5 rounded-lg shadow-lg max-w-sm w-100">
        <h3 class="h5 mb-4">{{ mensajePopup }}</h3>
        <div class="d-flex justify-content-between">
          <button class="btn btn-success" @click="confirmarAccion">
            Confirmar
          </button>
          <button class="btn btn-danger" @click="cerrarPopup">Cancelar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import CharlasService from '@/services/CharlasService';
export default {
  data() {
    return {
      idRonda: 0,
      idCurso: 0,
      charlasPropuestas: [],
      charlasAceptadas: [],
      charlasTotal: [],
      draggedCharla: null,
      mostrarPopup: false,
      mensajePopup: "",
      votosPropuestos: 0,
      totalVotos: 0,
      charlasService: new CharlasService()
    };
  },
  methods: {
    onImageError(event) {
      event.target.src =
        "https://cdn-icons-png.freepik.com/512/3415/3415488.png";
    },
    navigateToStudentList() {
      this.$router.push(`/studentslist/${this.idCurso}`);
    },
    onDragStart(event, charla) {
      this.draggedCharla = charla;
    },
    onDrop(event, targetList) {
      if (this.draggedCharla) {
        if (targetList === "propuestas") {
          const charlaExistente = this.charlasPropuestas.some(
            (c) => c.idCharla === this.draggedCharla.idCharla
          );
          if (!charlaExistente) {
            this.charlasAceptadas = this.charlasAceptadas.filter(
              (c) => c.idCharla !== this.draggedCharla.idCharla
            );
            this.charlasPropuestas.push(this.draggedCharla);
            this.draggedCharla.idEstadoCharla = 1;
          }
        } else {
          const charlaExistente = this.charlasAceptadas.some(
            (c) => c.idCharla === this.draggedCharla.idCharla
          );
          if (!charlaExistente) {
            this.charlasPropuestas = this.charlasPropuestas.filter(
              (c) => c.idCharla !== this.draggedCharla.idCharla
            );
            this.charlasAceptadas.push(this.draggedCharla);
            this.draggedCharla.idEstadoCharla = 2;
          }
        }
        this.draggedCharla = null;
      }
    },
    abrirConfirmacionGuardar() {
      this.mostrarPopup = true;
      this.mensajePopup = "¿Está seguro de que desea guardar los cambios?";
    },
    abrirConfirmacionCancelar() {
      this.mostrarPopup = true;
      this.mensajePopup = "¿Está seguro de que desea cancelar los cambios?";
    },
    confirmarAccion() {
      if (
        this.mensajePopup === "¿Está seguro de que desea guardar los cambios?"

      ) {
        this.guardarCambios();
      } else if (
        this.mensajePopup === "¿Está seguro de que desea cancelar los cambios?"
      ) {
        this.cancelarCambios();
      }
      this.cerrarPopup();
    },
    cerrarPopup() {
      this.mostrarPopup = false;
    },
    guardarCambios() {
      console.log("ADFSDFEFDGAFDGHEFGFDAHFDHGHFD");
      // Implementar lógica para guardar cambios en las charlas
      const updatedCharlas = this.charlasAceptadas.map((charla) => ({
        idCharla: charla.idCharla,
        titulo: charla.titulo,
        descripcion: charla.descripcion,
        tiempo: charla.tiempo,
        fechaPropuesta: charla.fechaPropuesta,
        idUsuario: charla.idUsuario,
        idEstadoCharla: charla.idEstadoCharla,
        idRonda: charla.idRonda,
        imagenCharla: charla.imagenCharla,
      }));

      //console.log(updatedCharlas);
      updatedCharlas.forEach((charla) => {
        console.log(charla)
        this.charlasService.updateEstadoCharla(charla.idCharla, charla.idEstadoCharla).then(response => {
          console.log(response);
        });
      });
    },
    cancelarCambios() {
      window.location.reload();
      
    },
    estadocharlas() {
      this.charlasTotal.forEach((charla, i) => {
        console.log(i)
        if (charla.estadoCharla == "ACEPTADA") {
          this.charlasAceptadas.push(charla);

        } else {
          this.charlasPropuestas.push(charla)
          console.log(charla)
        }
      })
    }
  },
  mounted() {
    this.idRonda = this.$route.params.id;
    this.idCurso = this.$route.params.idCurso;
    // Aquí debes cargar las charlas y alumnos de la misma forma que en el componente Angular
    this.charlasService.getCharlasRonda(this.idRonda).then(r => {
      this.charlasTotal = r
      this.estadocharlas();
    })

  },
};
</script>

<style scoped>
.scrollable-content {
  max-height: 400px;
  /* Ajusta según sea necesario */
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.draggable-item {
  display: flex;
  align-items: center;
  padding: 8px 12px;
  background: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  font-size: 14px;
  margin-bottom: 6px;
}

.draggable-item img {
  width: 40px;
  height: 40px;
  object-fit: cover;
  border-radius: 50%;
  margin-right: 8px;
}

.draggable-item h3 {
  font-size: 14px;
  margin: 0;
  font-weight: 600;
}

.draggable-item span {
  font-size: 12px;
  padding: 4px 6px;
}

@keyframes float {
  0% {
    transform: translateY(0) translateX(0);
    opacity: 1;
  }

  50% {
    transform: translateY(-100px) translateX(50px);
    opacity: 0.7;
  }

  100% {
    transform: translateY(-200px) translateX(-50px);
    opacity: 0;
  }
}
</style>