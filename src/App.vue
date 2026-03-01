<template>
  <div id="app">
    <header v-if="usuarioActivo" class="navbar-pro">
      <div class="logo">ESCARLATTI-GEST</div>
      <div class="user-info">
        <span :class="['badge', getBadgeClass(usuarioActivo.rol)]">
          {{ usuarioActivo.rol.toUpperCase() }}
        </span>
        <span class="user-name">Hola, <strong>{{ usuarioActivo.nombre }} {{ usuarioActivo.apellidos }}</strong></span>
        <button @click="cerrarSesion" class="btn-salir">Cerrar Sesión</button>
      </div>
    </header>

    <LoginComponent v-if="!usuarioActivo" @logueado="iniciarSesion" />

    <div v-else class="main-layout">
      <aside class="sidebar">
        
        <div v-if="usuarioActivo.rol === 'Administrador'">
          <p class="menu-label">DATOS MAESTROS</p>
          <button @click="pagina = 'alumnos'" :class="{active: pagina === 'alumnos'}">Alumnos</button>
          <button @click="pagina = 'profesores'" :class="{active: pagina === 'profesores'}">Profesores</button>
          <button @click="pagina = 'cursos'" :class="{active: pagina === 'cursos'}">Cursos</button>
          <button @click="pagina = 'departamentos'" :class="{active: pagina === 'departamentos'}">Departamentos</button>
          <button @click="pagina = 'etapas'" :class="{active: pagina === 'etapas'}">Etapas</button>
          
          <p class="menu-label">SISTEMA</p>
          <button @click="pagina = 'usuarios'" :class="{active: pagina === 'usuarios'}">Usuarios</button>
          <button @click="pagina = 'roles'" :class="{active: pagina === 'roles'}">Roles</button>
          <button @click="pagina = 'turnos'" :class="{active: pagina === 'turnos'}">Turnos</button>
        </div>

        <div v-if="usuarioActivo.rol === 'Administrador' || usuarioActivo.rol === 'Profesor' || usuarioActivo.rol === 'TIC'">
          <p class="menu-label">RESERVAS</p>
          <button @click="pagina = 'espacios'" :class="{active: pagina === 'espacios'}">
            {{ usuarioActivo.rol === 'Administrador' ? 'Gestionar Aulas' : 'Reservar Aula' }}
          </button>
          <button @click="pagina = 'horarios'" :class="{active: pagina === 'horarios'}">Horarios Generales</button>
        </div>

        <p class="menu-label">SOPORTE</p>
        <button @click="pagina = 'incidencias'" :class="{active: pagina === 'incidencias'}">
          {{ (usuarioActivo.rol === 'Administrador' || usuarioActivo.rol === 'TIC') ? 'Panel Incidencias' : 'Mis Incidencias' }}
        </button>
      </aside>

      <main class="content-area">
        <div v-if="pagina === 'inicio'" class="welcome-card">
          <h1>Panel de Control</h1>
          <p>Bienvenido, <strong>{{ usuarioActivo.nombre }}</strong>.</p>
          <hr>
          <p>Has accedido con el rol: <b>{{ usuarioActivo.rol }}</b></p>
        </div>

        <template v-if="usuarioActivo.rol === 'Administrador'">
          <AlumnosComponent v-if="pagina === 'alumnos'" />
          <ProfesoresComponent v-if="pagina === 'profesores'" />
          <CursosComponent v-if="pagina === 'cursos'" />
          <DepartamentosComponent v-if="pagina === 'departamentos'" />
          <EtapasComponent v-if="pagina === 'etapas'" />
          <UsuariosComponent v-if="pagina === 'usuarios'" />
          <RolesComponent v-if="pagina === 'roles'" />
          <TurnosComponent v-if="pagina === 'turnos'" />
          <AdminEspacios v-if="pagina === 'espacios'" />
          <HorariosComponent v-if="pagina === 'horarios'" />
          <IncidenciasTIC v-if="pagina === 'incidencias'" />
        </template>

        <template v-if="usuarioActivo.rol === 'Profesor'">
          <EspaciosComponent v-if="pagina === 'espacios'" :usuarioActivo="usuarioActivo" />
          <HorariosComponent v-if="pagina === 'horarios'" />
          <IncidenciasComponent v-if="pagina === 'incidencias'" :usuarioActivo="usuarioActivo" />
        </template>

        <template v-if="usuarioActivo.rol === 'TIC'">
          <EspaciosComponent v-if="pagina === 'espacios'" :usuarioActivo="usuarioActivo" />
          <HorariosComponent v-if="pagina === 'horarios'" />
          <IncidenciasTIC v-if="pagina === 'incidencias'" />
        </template>

        <template v-if="usuarioActivo.rol === 'Alumno'">
          <IncidenciasComponent v-if="pagina === 'incidencias'" :usuarioActivo="usuarioActivo" />
        </template>
      </main>
    </div>
  </div>
</template>

<script>

import LoginComponent from './components/LoginComponent.vue';
import AlumnosComponent from './components/AlumnosComponent.vue';
import ProfesoresComponent from './components/ProfesoresComponent.vue';
import UsuariosComponent from './components/UsuariosComponent.vue';
import EspaciosComponent from './components/EspaciosComponent.vue';
import AdminEspacios from './components/AdminEspacios.vue';
import HorariosComponent from './components/HorariosComponent.vue';
import IncidenciasComponent from './components/IncidenciasComponent.vue';
import IncidenciasTIC from './components/IncidenciasTIC.vue';
import CursosComponent from './components/CursosComponent.vue';
import DepartamentosComponent from './components/DepartamentosComponent.vue';
import EtapasComponent from './components/EtapasComponent.vue';
import RolesComponent from './components/RolesComponent.vue';
import TurnosComponent from './components/TurnosComponent.vue';


export default {
  components: {
    LoginComponent, AlumnosComponent, ProfesoresComponent, UsuariosComponent,
    EspaciosComponent, AdminEspacios, HorariosComponent, IncidenciasComponent, IncidenciasTIC, CursosComponent, DepartamentosComponent, EtapasComponent,
    RolesComponent, TurnosComponent
  },
  data() {
    return {
      usuarioActivo: null,
      pagina: 'inicio'
    }
  },
  methods: {
    iniciarSesion(data) {
      this.usuarioActivo = Array.isArray(data) ? data[0] : data;
      this.pagina = 'inicio';
    },
    cerrarSesion() {
      this.usuarioActivo = null;
      this.pagina = 'inicio';
    },
    getBadgeClass(rol) {
      if (rol === 'Administrador') return 'badge-admin';
      if (rol === 'TIC') return 'badge-tic';
      if (rol === 'Profesor') return 'badge-prof';
      return 'badge-alum';
    }
  }
}
</script>

<style>
/* Estilos para que se vea como en tus capturas */
body { margin: 0; font-family: 'Segoe UI', sans-serif; background: #f4f7f6; }
.navbar-pro { background: #1a1a1a; color: white; padding: 12px 30px; display: flex; justify-content: space-between; align-items: center; }
.main-layout { display: flex; height: calc(100vh - 65px); }
.sidebar { width: 220px; background: white; border-right: 1px solid #ddd; padding: 20px; }
.menu-label { font-size: 11px; color: #999; margin-top: 25px; font-weight: bold; text-transform: uppercase; }
.sidebar button { width: 100%; text-align: left; padding: 12px; margin-top: 5px; border: none; background: none; cursor: pointer; border-radius: 6px; }
.sidebar button.active { background: #007bff; color: white; font-weight: bold; }
.badge { padding: 4px 12px; border-radius: 15px; font-size: 11px; font-weight: bold; margin-right: 15px; }
.badge-admin { background: #e67e22; color: white; }
.badge-prof { background: #2ecc71; color: white; }
.content-area { flex: 1; padding: 30px; }
.welcome-card { background: white; padding: 40px; border-radius: 12px; text-align: center; box-shadow: 0 4px 10px rgba(0,0,0,0.1); max-width: 600px; margin: 0 auto; }
.btn-salir { background: #ff4d4f; color: white; border: none; padding: 8px 15px; border-radius: 5px; cursor: pointer; }
.badge-admin { background: #e67e22; color: white; }
.badge-prof { background: #2ecc71; color: white; }
.badge-tic { background: #8e44ad; color: white; }
.badge-alum { background: #3498db; color: white; }
</style>