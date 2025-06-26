<script setup lang="ts">
import { ref, onMounted } from 'vue'
import http from '@/plugins/axios'
import type { Empleado } from '@/models/empleado'
import type { Usuario } from '@/models/usuario'

const props = defineProps<{
  ENDPOINT_API: string
}>()

const ENDPOINT = props.ENDPOINT_API ?? ''

const empleado = ref<Empleado & { usuario?: Usuario }>({
  id: 0,
  nombre: '',
  apellido: '',
  telefono: '',
  direccion: '',
  cargo: '',
  usuario: undefined,
})

const usuarios = ref<Usuario[]>([])

const emit = defineEmits(['saved'])

async function crearEmpleado() {
  try {
    await http.post(ENDPOINT, {
      idUsuario: empleado.value.usuario?.idUsuario,
      nombre: empleado.value.nombre,
      apellido: empleado.value.apellido,
      telefono: empleado.value.telefono,
      direccion: empleado.value.direccion,
      cargo: empleado.value.cargo,
    })
    emit('saved')
  } catch (error) {
    console.error('Error al crear el empleado:', error)
  }
}

async function getUsuarios() {
  try {
    const response = await http.get('usuarios')
    usuarios.value = response.data
  } catch (error) {
    console.error('Error al obtener los usuarios:', error)
  }
}

onMounted(getUsuarios)
</script>

<template>
  <div class="container" v-if="empleado">
    <div class="row">
      <h2>Crear Nuevo Empleado</h2>
    </div>
    <div class="row">
      <form @submit.prevent="crearEmpleado">
        <div class="form-floating mb-3">
          <select class="form-select" v-model="empleado.usuario" required>
            <option value="" disabled>Seleccione un usuario</option>
            <option v-for="usuario in usuarios" :key="usuario.idUsuario" :value="usuario">
              {{ usuario.Nombreusuario }}
            </option>
          </select>
          <label for="usuario">Usuario</label>
        </div>
        <div class="form-floating mb-3">
          <input
            type="text"
            class="form-control"
            v-model="empleado.nombre"
            placeholder="Nombre"
            required
          />
          <label for="nombre">Nombre</label>
        </div>
        <div class="form-floating mb-2">
          <input
            type="text"
            class="form-control"
            v-model="empleado.apellido"
            placeholder="Apellido"
            required
          />
          <label for="apellido">Apellido</label>
        </div>
        <div class="form-floating mb-2">
          <input
            type="text"
            class="form-control"
            v-model="empleado.telefono"
            placeholder="Teléfono"
            required
          />
          <label for="telefono">Teléfono</label>
        </div>
        <div class="form-floating mb-2">
          <input
            type="text"
            class="form-control"
            v-model="empleado.direccion"
            placeholder="Dirección"
            required
          />
          <label for="direccion">Dirección</label>
        </div>
        <div class="form-floating mb-2">
          <input
            type="text"
            class="form-control"
            v-model="empleado.cargo"
            placeholder="Cargo"
            required
          />
          <label for="cargo">Cargo</label>
        </div>
        <div class="text-center mt-3">
          <button type="submit" class="btn btn-primary btn-lg">Guardar</button>
        </div>
      </form>
    </div>
  </div>
</template>

<style scoped>
.container {
  padding: 20px;
}
button {
  margin-top: 10px;
}
</style>
