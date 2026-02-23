<template>
  <div id="app">
    <header v-if="usuarioActivo" class="navbar-pro">
      <div class="logo">Escarlatti-Gestión</div>
      <div class="user-info">
        <span class="badge">{{ usuarioActivo.rol_id }}</span>
        <span class="user-name">Hola, <strong>{{ usuarioActivo.login }}</strong></span>
        <button @click="cerrarSesion" class="btn-salir">Cerrar Sesión</button>
      </div>
    </header>

    <LoginComponent v-if="!usuarioActivo" @logueado="iniciarSesion" />

    <div v-else class="main-layout">
      <aside class="sidebar">
        <p class="menu-label">ACADÉMICO</p>
        <button @click="pagina = 'alumnos'" :class="{active: pagina === 'alumnos'}">Alumnos</button>
        <button @click="pagina = 'profesores'" :class="{active: pagina === 'profesores'}">Profesores</button>
        <button @click="pagina = 'cursos'" :class="{active: pagina === 'cursos'}">Cursos</button>

        <p class="menu-label">ORGANIZACIÓN</p>
        <button @click="pagina = 'departamentos'" :class="{active: pagina === 'departamentos'}">Departamentos</button>
        <button @click="pagina = 'espacios'" :class="{active: pagina === 'espacios'}">Espacios</button>
        <button @click="pagina = 'etapas'" :class="{active: pagina === 'etapas'}">Etapas</button>
        <button @click="pagina = 'turnos'" :class="{active: pagina === 'turnos'}">Turnos</button>

        <p class="menu-label">SEGURIDAD</p>
        <button @click="pagina = 'usuarios'" :class="{active: pagina === 'usuarios'}">Usuarios</button>
        <button @click="pagina = 'roles'" :class="{active: pagina === 'roles'}">Roles</button>
      </aside>

      <main class="content-area">
        <div v-if="pagina === 'inicio'" class="welcome-card">
          <h1>Panel de Administración</h1>
          <p>Bienvenido al sistema de gestión. Selecciona una opción del menú para administrar la base de datos.</p>
        </div>

        <AlumnosComponent v-if="pagina === 'alumnos'" />
        <ProfesoresComponent v-if="pagina === 'profesores'" />
        <CursosComponent v-if="pagina === 'cursos'" />
        <DepartamentosComponent v-if="pagina === 'departamentos'" />
        <EspaciosComponent v-if="pagina === 'espacios'" />
        <EtapasComponent v-if="pagina === 'etapas'" />
        <TurnosComponent v-if="pagina === 'turnos'" />
        <UsuariosComponent v-if="pagina === 'usuarios'" />
        <RolesComponent v-if="pagina === 'roles'" />
      </main>
    </div>
  </div>
</template>

<script>
// 1. IMPORTACIÓN DE TODOS LOS COMPONENTES
import LoginComponent from './components/LoginComponent.vue';
import AlumnosComponent from './components/AlumnosComponent.vue';
import ProfesoresComponent from './components/ProfesoresComponent.vue';
import CursosComponent from './components/CursosComponent.vue';
import DepartamentosComponent from './components/DepartamentosComponent.vue';
import EspaciosComponent from './components/EspaciosComponent.vue';
import EtapasComponent from './components/EtapasComponent.vue';
import TurnosComponent from './components/TurnosComponent.vue';
import UsuariosComponent from './components/UsuariosComponent.vue';
import RolesComponent from './components/RolesComponent.vue';

export default {
  // 2. REGISTRO DE COMPONENTES
  components: { 
    LoginComponent, AlumnosComponent, ProfesoresComponent, 
    CursosComponent, DepartamentosComponent, EspaciosComponent,
    EtapasComponent, TurnosComponent, UsuariosComponent, RolesComponent
  },
  data() {
    return {
      usuarioActivo: null, // Guardamos la info del usuario logueado
      pagina: 'inicio'     // Controlamos qué componente se muestra
    }
  },
  methods: {
    iniciarSesion(u) {
      this.usuarioActivo = u;
      this.pagina = 'inicio';
    },
    cerrarSesion() {
      this.usuarioActivo = null;
      this.pagina = 'inicio';
    }
  }
}
</script>

<style>
/* --- CSS GLOBAL Y ESTRUCTURA --- */
body { margin: 0; font-family: 'Segoe UI', sans-serif; background: #f4f7f6; color: #333; }

/* Barra Superior */
.navbar-pro { background: #1a1a1a; color: white; padding: 15px 40px; display: flex; justify-content: space-between; align-items: center; box-shadow: 0 2px 5px rgba(0,0,0,0.2); }
.logo { font-size: 1.4rem; font-weight: bold; letter-spacing: 1px; }
.user-info { display: flex; align-items: center; gap: 15px; }
.badge { background: #007bff; color: white; padding: 4px 12px; border-radius: 20px; font-size: 0.75rem; font-weight: bold; text-transform: uppercase; }

/* Layout Principal */
.main-layout { display: flex; height: calc(100vh - 65px); }

/* Sidebar (Menú Lateral) */
.sidebar { width: 250px; background: white; border-right: 1px solid #ddd; padding: 20px; overflow-y: auto; }
.menu-label { font-size: 0.7rem; font-weight: bold; color: #999; margin: 25px 0 10px 5px; letter-spacing: 1px; }
.sidebar button { width: 100%; text-align: left; padding: 12px 15px; border: none; background: none; border-radius: 8px; cursor: pointer; margin-bottom: 4px; transition: all 0.2s; font-size: 0.9rem; color: #555; }
.sidebar button:hover:not(.active) { background: #f0f4f8; color: #007bff; }
.sidebar button.active { background: #007bff; color: white; font-weight: bold; box-shadow: 0 4px 10px rgba(0, 123, 255, 0.3); }

/* Contenido Principal */
.content-area { flex: 1; padding: 40px; overflow-y: auto; }
.welcome-card { background: white; padding: 50px; border-radius: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); text-align: center; max-width: 700px; margin: 0 auto; }
.welcome-card h1 { color: #1a1a1a; margin-bottom: 15px; }

/* Botón Salir */
.btn-salir { background: #ff4d4f; color: white; border: none; padding: 8px 16px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 0.85rem; }
.btn-salir:hover { background: #d9363e; }
</style>