<template>
  <div id="app">
    <header v-if="usuarioActivo" class="navbar-pro">
      <div class="logo">ESCARLATTI-GEST</div>
      <div class="user-info">
        <span :class="['badge', usuarioActivo.rol_id === 'Admin_a' ? 'badge-admin' : 'badge-prof']">
          {{ usuarioActivo.rol_id === 'Admin_a' ? 'ADMINISTRADOR' : 'PROFESOR' }}
        </span>
        <span class="user-name">Hola, <strong>{{ usuarioActivo.usuario }}</strong></span>
        <button @click="cerrarSesion" class="btn-salir">Cerrar Sesión</button>
      </div>
    </header>

    <LoginComponent v-if="!usuarioActivo" @logueado="iniciarSesion" />

    <div v-else class="main-layout">
      <aside class="sidebar">
        
        <div v-if="usuarioActivo.rol_id === 'Admin_a'">
          <p class="menu-label">DATOS MAESTROS</p>
          <button @click="pagina = 'alumnos'" :class="{active: pagina === 'alumnos'}">Alumnos</button>
          <button @click="pagina = 'profesores'" :class="{active: pagina === 'profesores'}">Profesores</button>
          <button @click="pagina = 'cursos'" :class="{active: pagina === 'cursos'}">Cursos</button>
          
          <p class="menu-label">SISTEMA</p>
          <button @click="pagina = 'turnos'" :class="{active: pagina === 'turnos'}">Turnos</button>
          <button @click="pagina = 'usuarios'" :class="{active: pagina === 'usuarios'}">Usuarios</button>
        </div>

        <p class="menu-label">RESERVAS</p>
        <button @click="pagina = 'espacios'" :class="{active: pagina === 'espacios'}">Reservar Aula</button>
        <button @click="pagina = 'horarios'" :class="{active: pagina === 'horarios'}">Cronograma General</button>
        
      </aside>

      <main class="content-area">
        <EspaciosComponent 
          v-if="pagina === 'espacios'" 
          :usuarioActivo="usuarioActivo" 
        />

        <HorariosComponent v-if="pagina === 'horarios'" />
        
        <template v-if="usuarioActivo.rol_id === 'Admin_a'">
          <AlumnosComponent v-if="pagina === 'alumnos'" />
          <ProfesoresComponent v-if="pagina === 'profesores'" />
          <CursosComponent v-if="pagina === 'cursos'" />
          <TurnosComponent v-if="pagina === 'turnos'" />
          <UsuariosComponent v-if="pagina === 'usuarios'" />
        </template>

        <div v-if="pagina === 'inicio'" class="welcome-card">
          <h1>Panel de Control</h1>
          <p>Bienvenido, <strong>{{ usuarioActivo.usuario }}</strong>.</p>
          <hr>
          <p v-if="usuarioActivo.rol_id === 'Prof_a'">
            Usa el botón <b>"Reservar Aula"</b> para ver la disponibilidad de los espacios 11111 y 22222.
          </p>
          <p v-else>Tienes acceso total a la configuración del sistema.</p>
        </div>
      </main>
    </div>
  </div>
</template>

<script>
import LoginComponent from './components/LoginComponent.vue';
import AlumnosComponent from './components/AlumnosComponent.vue';
import ProfesoresComponent from './components/ProfesoresComponent.vue';
import CursosComponent from './components/CursosComponent.vue';
import TurnosComponent from './components/TurnosComponent.vue';
import HorariosComponent from './components/HorariosComponent.vue';
import EspaciosComponent from './components/EspaciosComponent.vue';
import UsuariosComponent from './components/UsuariosComponent.vue';


export default {
  name: 'App',
  components: {
    LoginComponent, AlumnosComponent, ProfesoresComponent,
    CursosComponent, TurnosComponent, HorariosComponent, 
    EspaciosComponent, UsuariosComponent
  },
  data() {
    return {
      usuarioActivo: null,
      pagina: 'inicio'
    }
  },
  methods: {
    iniciarSesion(usuario) {
      console.log("Usuario recibido:", usuario); // Revisa la consola para ver cómo se llama el campo del nombre
      this.usuarioActivo = usuario;
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
/* Estilos generales idénticos a los anteriores para mantener coherencia */
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

.content-area { flex: 1; padding: 30px; overflow-y: auto; }
.welcome-card { background: white; padding: 40px; border-radius: 12px; text-align: center; box-shadow: 0 4px 10px rgba(0,0,0,0.1); max-width: 500px; margin: 40px auto; }
.btn-salir { background: #ff4d4f; color: white; border: none; padding: 8px 15px; border-radius: 5px; cursor: pointer; }
</style>