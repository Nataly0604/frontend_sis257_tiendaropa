<script setup lang="ts">
import type { Empleado } from '@/models/empleado'
import type { Usuario } from '@/models/usuario'
import http from '@/plugins/axios'
import Button from 'primevue/button'
import Dialog from 'primevue/dialog'
import { onMounted, ref } from 'vue'
import PaginacionTabla from '../PaginacionTabla.vue'
import { usePaginacion } from '@/composables/usePaginacion'

const ENDPOINT = 'empleados'
const empleados = ref<Empleado[]>([])
const usuarios = ref<Usuario[]>([])
const emit = defineEmits(['edit'])
const empleadoDelete = ref<Empleado | null>(null)
const mostrarConfirmDialog = ref<boolean>(false)

// Implementar paginación
const {
  datosPaginados: empleadosPaginados,
  totalRegistros,
  onCambioPagina,
  onCambioRecordsPorPagina,
} = usePaginacion(empleados, { recordsPorPaginaInicial: 5 })

async function obtenerLista() {
  const [empleadosRes, usuariosRes] = await Promise.all([http.get(ENDPOINT), http.get('usuarios')])
  console.log('Empleados:', empleadosRes.data)
  console.log('Usuarios:', usuariosRes.data)
  usuarios.value = usuariosRes.data
  empleados.value = empleadosRes.data.map((e: any) => ({
    ...e,
    usuario: e.usuario // Si ya viene embebido
      ? e.usuario
      : usuarios.value.find((u) => u.idUsuario === e.idUsuario || u.idUsuario === e.usuarioId) || {
          Nombreusuario: 'Sin usuario',
        },
  }))
}

function emitirEdicion(empleado: Empleado) {
  emit('edit', empleado)
}

function mostrarEliminarConfirm(empleado: Empleado) {
  empleadoDelete.value = empleado
  mostrarConfirmDialog.value = true
}

async function eliminar() {
  await http.delete(`${ENDPOINT}/${empleadoDelete.value?.id}`)
  await obtenerLista()
  mostrarConfirmDialog.value = false
}

onMounted(() => {
  obtenerLista()
})
defineExpose({ obtenerLista })
</script>

<template>
  <div class="table-container">
    <div class="table-wrapper">
      <table class="admin-table">
        <thead>
          <tr>
            <th>Nombre</th>
            <th>Apellido</th>
            <th>Teléfono</th>
            <th>Dirección</th>
            <th>Cargo</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="empleado in empleadosPaginados" :key="empleado.id">
            <td class="employee-name">{{ empleado.nombre }}</td>
            <td>{{ empleado.apellido }}</td>
            <td class="phone-cell">{{ empleado.telefono }}</td>
            <td class="address-cell">{{ empleado.direccion }}</td>
            <td class="position-cell">{{ empleado.cargo }}</td>
            <td class="actions-cell">
              <Button
                icon="pi pi-pencil"
                aria-label="Editar"
                text
                severity="info"
                @click="emitirEdicion(empleado)"
              />
              <Button
                icon="pi pi-trash"
                aria-label="Eliminar"
                text
                severity="danger"
                @click="mostrarEliminarConfirm(empleado)"
              />
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Componente de paginación -->
    <PaginacionTabla
      :total-registros="totalRegistros"
      :records-por-pagina-inicial="5"
      @cambio-pagina="onCambioPagina"
      @cambio-records-por-pagina="onCambioRecordsPorPagina"
    />

    <Dialog
      v-model:visible="mostrarConfirmDialog"
      header="Confirmar Eliminación"
      :style="{ width: '25rem' }"
    >
      <p>¿Estás seguro de que deseas eliminar este registro?</p>
      <div class="flex justify-end gap-2">
        <Button
          type="button"
          label="Cancelar"
          severity="secondary"
          @click="mostrarConfirmDialog = false"
        />
        <Button type="button" label="Eliminar" @click="eliminar" />
      </div>
    </Dialog>
  </div>
</template>

<style scoped>
.table-container {
  padding: 1.5rem;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  margin: 1rem 0;
}

.table-wrapper {
  overflow-x: auto;
  border-radius: 8px;
  border: 1px solid #e5e7eb;
}

.admin-table {
  width: 100%;
  border-collapse: collapse;
  font-family: 'Poppins-Regular', Arial, sans-serif;
  background: #fff;
}

.admin-table thead {
  background: linear-gradient(135deg, #717fe0 0%, #6c7ae0 100%);
}

.admin-table thead th {
  padding: 1rem 0.75rem;
  text-align: left;
  font-weight: 600;
  font-size: 0.875rem;
  color: #fff;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  border: none;
  position: relative;
}

.admin-table thead th:not(:last-child)::after {
  content: '';
  position: absolute;
  right: 0;
  top: 25%;
  height: 50%;
  width: 1px;
  background: rgba(255, 255, 255, 0.2);
}

.admin-table tbody tr {
  transition: all 0.3s ease;
}

.admin-table tbody tr:nth-child(even) {
  background-color: #f8fafc;
}

.admin-table tbody tr:hover {
  background-color: #e0e7ff;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(113, 127, 224, 0.15);
}

.admin-table tbody td {
  padding: 1rem 0.75rem;
  border-bottom: 1px solid #e5e7eb;
  font-size: 0.875rem;
  color: #374151;
  vertical-align: middle;
}

.employee-name {
  font-weight: 600;
  color: #6c7ae0;
}

.phone-cell {
  font-weight: 500;
  color: #059669;
}

.address-cell {
  max-width: 200px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.position-cell {
  font-weight: 600;
  color: #d97706;
  background: #fef3c7;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  text-align: center;
  font-size: 0.8rem;
}

.actions-cell {
  display: flex;
  gap: 0.5rem;
  justify-content: center;
  align-items: center;
}

.actions-cell .p-button {
  padding: 0.5rem;
  border-radius: 6px;
  transition: all 0.3s ease;
}

.actions-cell .p-button:hover {
  transform: scale(1.1);
}

/* Responsive design */
@media (max-width: 768px) {
  .table-container {
    padding: 1rem;
    margin: 0.5rem 0;
  }

  .admin-table thead th,
  .admin-table tbody td {
    padding: 0.75rem 0.5rem;
    font-size: 0.8rem;
  }

  .address-cell {
    max-width: 150px;
  }

  .actions-cell {
    flex-direction: column;
    gap: 0.25rem;
  }
}

@media (max-width: 640px) {
  .admin-table {
    font-size: 0.75rem;
  }

  .admin-table thead th,
  .admin-table tbody td {
    padding: 0.5rem 0.25rem;
  }

  .address-cell {
    max-width: 100px;
  }
}
</style>
