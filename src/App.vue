<script setup>
import { ref, onMounted } from 'vue';
import api from './services/api';

const pestañaActiva = ref('reparaciones');
const clientes = ref([]);
const vehiculos = ref([]);
const reparaciones = ref([]);
const mensaje = ref(null);

const nuevoCliente = ref({ nombre: '', telefono: '', direccion: '', email: '' });
const nuevoVehiculo = ref({ placa: '', marca: '', modelo: '', anio: '', color: '', cliente_id: '' });
const nuevaReparacion = ref({ vehiculo_id: '', descripcion_problema: '', costo_total: '' });

const cargarDatos = async () => {
  try {
    const [resClientes, resVehiculos, resReparaciones] = await Promise.all([
      api.get('/clientes'),
      api.get('/vehiculos'),
      api.get('/reparaciones')
    ]);
    clientes.value = resClientes.data;
    vehiculos.value = resVehiculos.data;
    reparaciones.value = resReparaciones.data;
  } catch (error) {
    console.error('Error al cargar datos:', error);
  }
};

const registrarCliente = async () => {
  try {
    await api.post('/clientes', nuevoCliente.value);
    mostrarMensaje('¡Cliente registrado con éxito!');
    nuevoCliente.value = { nombre: '', telefono: '', direccion: '', email: '' };
    cargarDatos();
  } catch (error) {
    mostrarMensaje('Error al registrar cliente.', true);
  }
};

const registrarVehiculo = async () => {
  try {
    await api.post('/vehiculos', nuevoVehiculo.value);
    mostrarMensaje('¡Vehículo registrado con éxito!');
    nuevoVehiculo.value = { placa: '', marca: '', modelo: '', anio: '', color: '', cliente_id: '' };
    cargarDatos();
  } catch (error) {
    mostrarMensaje('Error al registrar vehículo.', true);
  }
};

const registrarReparacion = async () => {
  try {
    await api.post('/reparaciones', nuevaReparacion.value);
    mostrarMensaje('¡Orden de reparación creada con éxito!');
    nuevaReparacion.value = { vehiculo_id: '', descripcion_problema: '', costo_total: '' };
    cargarDatos();
  } catch (error) {
    mostrarMensaje('Error al crear reparación.', true);
  }
};

const actualizarEstado = async (id, nuevoEstado) => {
  try {
    await api.put(`/reparaciones/${id}`, { estado: nuevoEstado });
    mostrarMensaje('¡Estado actualizado correctamente!');
    cargarDatos();
  } catch (error) {
    mostrarMensaje('Error al actualizar estado.', true);
  }
};

const mostrarMensaje = (texto, esError = false) => {
  mensaje.value = { texto, esError };
  setTimeout(() => mensaje.value = null, 3500);
};

onMounted(() => {
  cargarDatos();
});
</script>

<template>
  <div class="app-container">
    <!-- Navbar Superior -->
    <header class="header">
      <div class="logo-area">
        <span style="font-size: 1.75rem;">🚗</span>
        <div>
          <h1>Taller Mecánico Pro</h1>
          <p>Sistema Integral de Gestión de Taller</p>
        </div>
      </div>

      <nav class="nav-tabs">
        <button @click="pestañaActiva = 'reparaciones'" :class="{ active: pestañaActiva === 'reparaciones' }">
          🔧 Reparaciones
        </button>
        <button @click="pestañaActiva = 'vehiculos'" :class="{ active: pestañaActiva === 'vehiculos' }">
          🚙 Vehículos
        </button>
        <button @click="pestañaActiva = 'clientes'" :class="{ active: pestañaActiva === 'clientes' }">
          👥 Clientes
        </button>
      </nav>
    </header>

    <!-- Notificación Flotante -->
    <div v-if="mensaje" :class="['alert', mensaje.esError ? 'alert-error' : 'alert-success']">
      {{ mensaje.texto }}
    </div>

    <!-- Contenido Principal -->
    <main class="main-content">

      <!-- ================= REPARACIONES ================= -->
      <section v-if="pestañaActiva === 'reparaciones'" class="grid-layout">
        <div class="card">
          <h2>📝 Nueva Orden</h2>
          <form @submit.prevent="registrarReparacion">
            <div class="form-group">
              <label>Vehículo</label>
              <select v-model="nuevaReparacion.vehiculo_id" required class="form-control">
                <option disabled value="">-- Elige un vehículo --</option>
                <option v-for="v in vehiculos" :key="v.id" :value="v.id">
                  {{ v.placa }} - {{ v.marca }} {{ v.modelo }}
                </option>
              </select>
            </div>
            <div class="form-group">
              <label>Problema</label>
              <textarea v-model="nuevaReparacion.descripcion_problema" rows="3" required class="form-control" placeholder="Falla en frenos..."></textarea>
            </div>
            <div class="form-group">
              <label>Costo ($)</label>
              <input v-model="nuevaReparacion.costo_total" type="number" step="0.01" class="form-control" placeholder="0.00" />
            </div>
            <button type="submit" class="btn-primary">Crear Orden</button>
          </form>
        </div>

        <div class="card">
          <h2>⚙️ Seguimiento de Taller</h2>
          <div v-if="reparaciones.length === 0" class="empty-state">No hay órdenes registradas.</div>
          <div v-else class="items-list">
            <div v-for="rep in reparaciones" :key="rep.id" class="item-card">
              <div class="item-header">
                <div>
                  <span class="badge">Placa: {{ rep.placa }}</span>
                  <strong style="margin-left: 0.5rem; font-size: 0.9rem;">{{ rep.marca }} {{ rep.modelo }}</strong>
                </div>
                <select :value="rep.estado" @change="e => actualizarEstado(rep.id, e.target.value)" class="form-control" style="width: auto; padding: 0.3rem 0.5rem; font-size: 0.75rem;">
                  <option value="Pendiente">Pendiente</option>
                  <option value="En Proceso">En Proceso</option>
                  <option value="Finalizado">Finalizado</option>
                </select>
              </div>
              <p style="font-size: 0.85rem; color: #cbd5e1; margin: 0.25rem 0;">
                <strong>Problema:</strong> {{ rep.descripcion_problema }}
              </p>
              <div style="display: flex; justify-content: space-between; font-size: 0.75rem; color: #94a3b8; border-top: 1px solid rgba(51, 65, 85, 0.4); padding-top: 0.5rem;">
                <span>Dueño: {{ rep.dueno_nombre }}</span>
                <span style="color: #34d399; font-weight: bold;">Costo: ${{ Number(rep.costo_total).toLocaleString() }}</span>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- ================= VEHÍCULOS ================= -->
      <section v-if="pestañaActiva === 'vehiculos'" class="grid-layout">
        <div class="card">
          <h2>🚙 Registrar Vehículo</h2>
          <form @submit.prevent="registrarVehiculo">
            <div class="form-group">
              <label>Placa</label>
              <input v-model="nuevoVehiculo.placa" type="text" required class="form-control" placeholder="XYZ-123" />
            </div>
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 0.5rem;">
              <div class="form-group"><label>Marca</label><input v-model="nuevoVehiculo.marca" type="text" required class="form-control" placeholder="Renault" /></div>
              <div class="form-group"><label>Modelo</label><input v-model="nuevoVehiculo.modelo" type="text" required class="form-control" placeholder="Logan" /></div>
            </div>
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 0.5rem;">
              <div class="form-group"><label>Año</label><input v-model="nuevoVehiculo.anio" type="number" class="form-control" placeholder="2020" /></div>
              <div class="form-group"><label>Color</label><input v-model="nuevoVehiculo.color" type="text" class="form-control" placeholder="Gris" /></div>
            </div>
            <div class="form-group">
              <label>Propietario</label>
              <select v-model="nuevoVehiculo.cliente_id" required class="form-control">
                <option disabled value="">-- Selecciona un dueño --</option>
                <option v-for="c in clientes" :key="c.id" :value="c.id">{{ c.nombre }}</option>
              </select>
            </div>
            <button type="submit" class="btn-primary">Guardar Vehículo</button>
          </form>
        </div>

        <div class="card">
          <h2>📋 Lista de Vehículos</h2>
          <div v-if="vehiculos.length === 0" class="empty-state">No hay vehículos registrados.</div>
          <div v-else class="items-list">
            <div v-for="v in vehiculos" :key="v.id" class="item-card">
              <div class="item-header">
                <span class="badge">{{ v.placa }}</span>
                <span style="font-size: 0.75rem; color: #94a3b8;">{{ v.anio || 'N/A' }} - {{ v.color || 'Sin color' }}</span>
              </div>
              <strong style="font-size: 0.95rem;">{{ v.marca }} {{ v.modelo }}</strong>
            </div>
          </div>
        </div>
      </section>

      <!-- ================= CLIENTES ================= -->
      <section v-if="pestañaActiva === 'clientes'" class="grid-layout">
        <div class="card">
          <h2>👤 Nuevo Cliente</h2>
          <form @submit.prevent="registrarCliente">
            <div class="form-group"><label>Nombre</label><input v-model="nuevoCliente.nombre" type="text" required class="form-control" placeholder="Carlos Pérez" /></div>
            <div class="form-group"><label>Teléfono</label><input v-model="nuevoCliente.telefono" type="text" required class="form-control" placeholder="3101234567" /></div>
            <div class="form-group"><label>Dirección</label><input v-model="nuevoCliente.direccion" type="text" class="form-control" placeholder="Calle 10 # 20-30" /></div>
            <div class="form-group"><label>Correo</label><input v-model="nuevoCliente.email" type="email" class="form-control" placeholder="carlos@mail.com" /></div>
            <button type="submit" class="btn-primary">Guardar Cliente</button>
          </form>
        </div>

        <div class="card">
          <h2>📋 Lista de Clientes</h2>
          <div v-if="clientes.length === 0" class="empty-state">No hay clientes registrados.</div>
          <div v-else class="items-list">
            <div v-for="c in clientes" :key="c.id" class="item-card">
              <strong style="font-size: 0.95rem;">{{ c.nombre }}</strong>
              <span style="font-size: 0.8rem; color: #cbd5e1;">📞 {{ c.telefono }}</span>
              <span style="font-size: 0.8rem; color: #cbd5e1;">📍 {{ c.direccion || 'Sin dirección' }}</span>
              <span style="font-size: 0.75rem; color: #94a3b8;">✉️ {{ c.email || 'Sin correo' }}</span>
            </div>
          </div>
        </div>
      </section>

    </main>
  </div>
</template>