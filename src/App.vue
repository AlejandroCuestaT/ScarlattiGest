<template>
  <div id="app">
    <header class="barra-superior">
      <h1>Escarlatti-Gest</h1>
      <div v-if="sesion" class="info-usuario">
        <span><strong>{{ sesion.login }}</strong> ({{ nombreRol }})</span>
        <button @click="cerrarSesion" class="btn-salir">Cerrar sesión</button>
      </div>
    </header>

    <Login v-if="!sesion" @logueado="iniciarSesion" />

    <div v-else class="contenedor-principal">
      <nav class="menu-lateral">
        <button @click="vista = 'inicio'" :class="{ activo: vista === 'inicio' }">Inicio</button>
        
        <div class="seccion-menu">ACADÉMICO</div>
        <button v-if="['1', '2', '4'].includes(sesion.rol_id.toString())" 
                @click="vista = 'alumnos'" 
                :class="{ activo: vista === 'alumnos' }">Alumnos</button>
        <button v-if="['1', '2', '4'].includes(sesion.rol_id.toString())" 
                @click="vista = 'cursos'" 
                :class="{ activo: vista === 'cursos' }">Cursos</button>
        
        <div v-if="['1', '4'].includes(sesion.rol_id.toString())" class="seccion-menu">CONFIGURACIÓN</div>
        <template v-if="['1', '4'].includes(sesion.rol_id.toString())">
          <button @click="vista = 'profesores'" :class="{ activo: vista === 'profesores' }">Profesores</button>
          <button @click="vista = 'departamentos'" :class="{ activo: vista === 'departamentos' }">Departamentos</button>
          <button @click="vista = 'espacios'" :class="{ activo: vista === 'espacios' }">Espacios</button>
          <button @click="vista = 'etapas'" :class="{ activo: vista === 'etapas' }">Etapas</button>
          <button @click="vista = 'roles'" :class="{ activo: vista === 'roles' }">Roles</button>
        </template>

        <div v-if="sesion.rol_id == '1'" class="seccion-menu">SISTEMA</div>
        <button v-if="sesion.rol_id == '1'" 
                @click="vista = 'usuarios'" 
                :class="{ activo: vista === 'usuarios' }">Usuarios</button>
      </nav>

      <main class="contenido">
        <div v-if="vista === 'inicio'" class="bienvenida">
          <h2>Panel de Gestión Central</h2>
          <p>Bienvenido, {{ sesion.login }}. Utiliza el menú lateral para gestionar los recursos del centro.</p>
          <div class="status-box">
            Sesión iniciada como: <strong>{{ nombreRol }}</strong>
          </div>
        </div>
        
        <Alumnos v-if="vista === 'alumnos'" />
        <Profesores v-if="vista === 'profesores'" />
        <Cursos v-if="vista === 'cursos'" />
        <Departamentos v-if="vista === 'departamentos'" />
        <Espacios v-if="vista === 'espacios'" />
        <Etapas v-if="vista === 'etapas'" />
        <Roles v-if="vista === 'roles'" />
        <Usuarios v-if="vista === 'usuarios'" />
      </main>
    </div>
  </div>
</template>

<script>
import Login from './components/LoginComponent.vue'
import Alumnos from './components/AlumnosComponent.vue'
import Profesores from './components/ProfesoresComponent.vue'
import Cursos from './components/CursosComponent.vue'
import Departamentos from './components/DepartamentosComponent.vue'
import Espacios from './components/EspaciosComponent.vue'
import Etapas from './components/EtapasComponent.vue'
import Roles from './components/RolesComponent.vue'
import Usuarios from './components/UsuariosComponent.vue'

export default {
  name: 'App',
  components: { 
    Login, Alumnos, Profesores, Cursos, 
    Departamentos, Espacios, Etapas, Roles, Usuarios 
  },
  data() {
    return {
      sesion: null,
      vista: 'inicio'
    }
  },
  computed: {
    nombreRol() {
      if (!this.sesion) return '';
      const roles = { '1': 'Administrador', '2': 'Profesor', '3': 'Alumno', '4': 'TIC' };
      return roles[this.sesion.rol_id.toString()] || 'Usuario';
    }
  },
  methods: {
    iniciarSesion(usuario) {
      this.sesion = usuario;
      this.vista = 'inicio';
    },
    cerrarSesion() {
      this.sesion = null;
      this.vista = 'inicio';
    }
  }
}
</script>

<style>
body {
  margin: 0;
  font-family: 'Segoe UI', Arial, sans-serif;
  background-color: #f7f7f7;
  color: #222;
}

.barra-superior {
  background-color: #000;
  color: #fff;
  padding: 10px 30px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.btn-salir {
  background: #333;
  color: #fff;
  border: 1px solid #555;
  padding: 5px 12px;
  cursor: pointer;
  margin-left: 15px;
}

.btn-salir:hover { background: #ff4444; border-color: #ff4444; }

.contenedor-principal {
  display: flex;
  height: calc(100vh - 50px);
}

.menu-lateral {
  width: 220px;
  background-color: #fff;
  border-right: 1px solid #ddd;
  overflow-y: auto;
}

.seccion-menu {
  padding: 15px 20px 5px;
  font-size: 0.65rem;
  font-weight: 800;
  color: #999;
  letter-spacing: 1px;
}

.menu-lateral button {
  display: block;
  width: 100%;
  background: none;
  border: none;
  padding: 12px 25px;
  text-align: left;
  cursor: pointer;
  font-size: 0.9rem;
  color: #444;
}

.menu-lateral button:hover { background-color: #f5f5f5; }
.menu-lateral button.activo {
  background-color: #eee;
  font-weight: bold;
  border-left: 5px solid #000;
  color: #000;
}

.contenido {
  flex: 1;
  padding: 30px;
  background-color: #fff;
  overflow-y: auto;
}

.bienvenida {
  text-align: center;
  margin-top: 60px;
}

.status-box {
  display: inline-block;
  margin-top: 20px;
  padding: 15px 25px;
  background: #f9f9f9;
  border: 1px solid #eee;
  border-radius: 4px;
  color: #666;
}
</style>