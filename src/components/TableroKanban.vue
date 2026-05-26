<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue';
import { colummnasIniciales } from './tareasData.js';

const tareasGuardadas = localStorage.getItem('mis_columnas_kanban');
const columnas = ref(tareasGuardadas ? JSON.parse(tareasGuardadas) :colummnasIniciales);
const nuevaTareaTexto = ref('');

let tareaArrastrada = null;
let columnaOrigenId = null;

// Variable reactiva para guardar la hora actual del sistema (se actualiza cada minuto)
const horaActual = ref('');

const actualizarHoraActual = () => {
  const ahora = new Date();
  // Formatea la hora en formato "HH:MM" (ej: "14:35") para compararla fácilmente
  const horas = String(ahora.getHours()).padStart(2, '0');
  const minutos = String(ahora.getMinutes()).padStart(2, '0');
  horaActual.value = `${horas}:${minutos}`;
};

// Reloj en segundo plano
let intervaloReloj = null;
onMounted(() => {
  actualizarHoraActual();
  // Cada 30 segundos revisamos la hora para asegurar precisión
  intervaloReloj = setInterval(actualizarHoraActual, 30000);
});

onUnmounted(() => {
  clearInterval(intervaloReloj);
});

// Función de apoyo para saber si una tarea está vencida
const estaVencida = (tarea, columnaId) => {
  // Si no tiene hora asignada o ya está en la columna "Completadas", no está vencida
  if (!tarea.horaAlarma || columnaId === 'completadas') return false;
  
  // Compara el texto de las horas (ej: "14:30" < "15:00")
  return tarea.horaAlarma < horaActual.value;
};

watch(columnas, (nuevasColumnas) => {
  localStorage.setItem('mis_columnas_kanban', JSON.stringify(nuevasColumnas));
}, { deep: true });

const iniciarArrastre = (evento, tarea, columnaId) => {
  tareaArrastrada = tarea;
  columnaOrigenId = columnaId;
  evento.dataTransfer.effectAllowed = 'move';
};

const alSoltar = (columnaDestinoId) => {
  if (!tareaArrastrada || columnaOrigenId === columnaDestinoId) return;

  const columnaOrigen = columnas.value.find(col => col.id === columnaOrigenId);
  const columnaDestino = columnas.value.find(col => col.id === columnaDestinoId);

  if (columnaOrigen && columnaDestino) {
    columnaOrigen.tareas = columnaOrigen.tareas.filter(t => t.id !== tareaArrastrada.id);
    columnaDestino.tareas.push(tareaArrastrada);
  }

  tareaArrastrada = null;
  columnaOrigenId = null;
};

const agregarTarea = () => {
  if (!nuevaTareaTexto.value.trim()) return;
  const columnaPendientes = columnas.value.find(col => col.id === 'pendientes');
  if (columnaPendientes) {
    columnaPendientes.tareas.push({
      id: 'tarea-' + Date.now(),
      texto: nuevaTareaTexto.value,
      horaAlarma: '' // Nace vacío listo para que el usuario elija hora
    });
    nuevaTareaTexto.value = '';
  }
};

const eliminarTarea = (columnaId, tareaId) => {
  const columna = columnas.value.find(col => col.id === columnaId);
  if (columna) {
    columna.tareas = columna.tareas.filter(t => t.id !== tareaId);
  }
};

</script>

 
    <template>
  <div class="kanban-container">
    <header class="tablero-header">
      <h2>Mi Tablero de Objetivos Diarios</h2>
      <p>Hora actual del sistema: <strong class="reloj-digital">{{ horaActual }}</strong></p>
    </header>

    <div class="constructor-tarea">
      <input 
        v-model="nuevaTareaTexto" 
        placeholder="Escribe un nuevo objetivo diario..." 
        @keyup.enter="agregarTarea"
      />
      <button @click="agregarTarea">
        <span>+</span> Añadir Tarea
      </button>
    </div>

    <div class="tablero">
      <div 
        v-for="columna in columnas" 
        :key="columna.id" 
        class="columna"
        @dragover.prevent 
        @drop="alSoltar(columna.id)"
      >
        <div class="columna-header">
          <h3>{{ columna.titulo }}</h3>
          <span class="contador-tareas">{{ columna.tareas.length }}</span>
        </div>
        
        <div class="lista-tareas">
          <TransitionGroup name="lista-animada">
            <div 
              v-for="tarea in columna.tareas" 
              :key="tarea.id" 
              class="tarjeta-tarea"
              :class="{ 'tarjeta-vencida': estaVencida(tarea, columna.id) }"
              draggable="true"
              @dragstart="iniciarArrastre($event, tarea, columna.id)"
            >
              <div class="tarea-contenido">
                <span class="tarea-texto">{{ tarea.texto }}</span>
                
                <div class="alarma-contenedor">
                  <span class="icono-reloj">⏰</span>
                  <input 
                    type="time" 
                    v-model="tarea.horaAlarma" 
                    class="selector-hora"
                  />
                </div>
              </div>

              <button class="btn-eliminar" @click.stop="eliminarTarea(columna.id, tarea.id)">
                ✕
              </button>
            </div>
          </TransitionGroup>
          
          <p v-if="columna.tareas.length === 0" class="vacio">
            No hay tareas aquí. ¡Buen trabajo!
          </p>
        </div>
      </div>
    </div>
  </div>
</template>
 <style scoped>
 /* Contenedor Principal */
.kanban-container {
  width: 100%;
  max-width: 1100px;
  font-family: 'Segoe UI', system-ui, sans-serif;
  color: #f3f4f6;
  padding: 20px;
}

.tablero-header {
  margin-bottom: 30px;
}
.tablero-header h2 {
  font-size: 2rem;
  font-weight: 700;
  background: linear-gradient(135deg, #42b883 0%, #35495e 100%);
  background-clip: text; 
  -webkit-text-fill-color: transparent;
  margin: 0 0 5px 0;
}
.tablero-header p {
  color: #9ca3af;
  margin: 0;
  font-size: 0.95rem;
}

/* Formulario de Entrada */
.constructor-tarea {
  margin-bottom: 30px;
  display: flex;
  gap: 12px;
}
.constructor-tarea input {
  flex-grow: 1;
  padding: 14px 20px;
  border-radius: 10px;
  border: 1px solid #374151;
  background-color: #1f2937;
  color: #f3f4f6;
  font-size: 1rem;
  outline: none;
  transition: all 0.2s;
  box-shadow: inset 0 2px 4px rgba(0,0,0,0.2);
}
.constructor-tarea input:focus {
  border-color: #42b883;
  box-shadow: 0 0 0 3px rgba(66, 184, 131, 0.2);
}
.constructor-tarea button {
  padding: 14px 28px;
  background: linear-gradient(135deg, #42b883 0%, #34d399 100%);
  border: none;
  border-radius: 10px;
  color: #111827;
  font-weight: bold;
  font-size: 0.95rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: transform 0.1s, filter 0.2s;
}
.constructor-tarea button:hover {
  filter: brightness(1.1);
}
.constructor-tarea button:active {
  transform: scale(0.98);
}

/* Estructura del Tablero */
.tablero {
  display: flex;
  gap: 24px;
  align-items: flex-start;
}
.columna {
  background-color: #111827;
  flex: 1;
  border-radius: 14px;
  padding: 20px;
  min-height: 450px;
  border: 1px solid #1f2937;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.3);
}
.columna-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #1f2937;
  padding-bottom: 12px;
  margin-bottom: 15px;
}
.columna-header h3 {
  margin: 0;
  font-size: 1.05rem;
  font-weight: 600;
  color: #e5e7eb;
}
.contador-tareas {
  background-color: #1f2937;
  color: #9ca3af;
  padding: 2px 8px;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: bold;
}

/* Tarjetas de Tareas */
.lista-tareas {
  display: flex;
  flex-direction: column;
  gap: 12px;
  min-height: 350px;
}
.tarjeta-tarea {
  background-color: #1f2937;
  padding: 16px;
  border-radius: 10px;
  border: 1px solid #374151;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  cursor: grab;
  position: relative;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-right: 40px;
  transition: border-color 0.2s, background-color 0.2s;
}
.tarjeta-tarea:hover {
  border-color: #4b5563;
  background-color: #253246;
}
.tarjeta-tarea:active {
  cursor: grabbing;
}
.tarea-texto {
  font-size: 0.95rem;
  line-height: 1.4;
  color: #f3f4f6;
  word-break: break-word;
}
.btn-eliminar {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  color: #6b7280;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
}
.btn-eliminar:hover {
  background-color: rgba(239, 68, 68, 0.1);
  color: #f87171;
}
.vacio {
  color: #4b5563;
  text-align: center;
  font-size: 0.85rem;
  margin-top: 40px;
  font-style: italic;
}

/* ==========================================================================
   ✨ MAGIA DE ANIMACIONES VUE: Clases de transiciones para TransitionGroup
   ========================================================================== */
.lista-animada-enter-from,
.lista-animada-leave-to {
  opacity: 0;
  transform: translateY(15px) scale(0.95);
}

.lista-animada-enter-active,
.lista-animada-leave-active {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

/* Evita que los elementos salten rústicamente cuando uno del medio es eliminado */
.lista-animada-leave-active {
  position: absolute;
  width: calc(100% - 40px); /* Ajusta el ancho al colapsar */
}

.lista-animada-move {
  transition: transform 0.3s ease;
}
/* Estilos para el Reloj Digital del Header */
.reloj-digital {
  background-color: #1f2937;
  padding: 4px 10px;
  border-radius: 6px;
  color: #42b883;
  border: 1px solid #374151;
  font-family: monospace;
  font-size: 1rem;
}

/* Estructura interna de la tarjeta */
.tarea-contenido {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
}

/* Selector de Alarma */
.alarma-contenedor {
  display: flex;
  align-items: center;
  gap: 6px;
}
.icono-reloj {
  font-size: 0.85rem;
}
.selector-hora {
  background: #111827;
  border: 1px solid #374151;
  color: #9ca3af;
  border-radius: 4px;
  padding: 2px 6px;
  font-size: 0.8rem;
  font-family: monospace;
  outline: none;
  cursor: pointer;
}
.selector-hora:focus {
  border-color: #42b883;
  color: #f3f4f6;
}

/* ==========================================================================
   🚨 ESTILO DE ALERTA: Si la tarea se pasa de la hora, se inyecta esta clase
   ========================================================================== */
.tarjeta-tarea.tarjeta-vencida {
  border-color: #ef4444;
  background: linear-gradient(135deg, #1f2937 0%, rgba(239, 68, 68, 0.05) 100%);
  box-shadow: 0 0 10px rgba(239, 68, 68, 0.15);
}
.tarjeta-vencida .selector-hora {
  border-color: #f87171;
  color: #f87171;
}
.tarjeta-vencida .tarea-texto {
  color: #fca5a5;
  font-weight: 500;
}
</style>