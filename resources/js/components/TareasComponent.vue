<template>
  <div>
    <hr />
    <form @submit.prevent="editarTareas(tarea)" v-if="editarActivo">
      <h3>Editar Tareas</h3>
      <input type="text" placeholder="Nombre" v-model="tarea.nombre" class="form-control mb-2" />
      <input
        type="text"
        placeholder="Descripcion"
        v-model="tarea.descripcion"
        class="form-control mb-2"
      />
      <button class="btn btn-success" type="submit">Guardar</button>
      <button class="btn btn-danger" type="submit" @click="cancelarEdicion()">Cancelar Edicion</button>
    </form>

    <form @submit.prevent="agregar" v-else>
      <h3>Agregar Tareas</h3>
      <input type="text" placeholder="Nombre" v-model="tarea.nombre" class="form-control mb-2" />
      <input
        type="text"
        placeholder="Descripcion"
        v-model="tarea.descripcion"
        class="form-control mb-2"
      />
      <button class="btn btn-primary" type="submit">Agregar</button>
    </form>

    <hr class="m-3" />
    <h3>Listado de notas</h3>

    <ul class="list-group my-2">
      <li class="list-group-item" v-for="(item, index) in tareas" :key="index">
        <span class="badge badge-primary float-right">{{item.updated_at}}</span>
        <p>{{item.nombre}}</p>
        <p>{{item.descripcion}}</p>
        <button class="btn btn-warning btn-sm" @click="editarTarea(item)">Editar</button>
        <button class="btn btn-danger btn-sm" @click="eliminarTarea(item,index)">Eliminar</button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tareas: [],
      tarea: { nombre: "", descripcion: "" },
      editarActivo: false,
    };
  },
  created() {
    axios.get("/notas").then((res) => {
      this.tareas = res.data;
    });
  },
  methods: {
    agregar() {
      if (
        this.tarea.nombre.trim() === "" ||
        this.tarea.descripcion.trim() === ""
      ) {
        alert("Debes completar todos los campos anteriores antes de guardar");
        return;
      }

      //console.log(this.tarea.nombre, this.tarea.descripcion);
      const params = {
        nombre: this.tarea.nombre,
        descripcion: this.tarea.descripcion,
      };
      this.tarea.nombre = "";
      this.tarea.descripcion = "";
      axios.post("/notas", params).then((res) => {
        this.tareas.push(res.data);
      });
    },
    eliminarTarea(tarea, index) {
      axios.delete(`/notas/${tarea.id}`).then(() => {
        this.tareas.splice(index, 1);
      });
    },
    editarTarea(item) {
      this.editarActivo = true;
      this.tarea.nombre = item.nombre;
      this.tarea.descripcion = item.descripcion;
      this.tarea.id = item.id;
    },
    editarTareas(item) {
      const params = { nombre: item.nombre, descripcion: item.descripcion };
      axios.put(`/notas/${item.id}`, params).then((res) => {
        this.editarActivo = false;
        const index = this.tareas.findIndex((item) => item.id === res.data.id);
        this.tareas[index] = res.data;
        this.tarea = { nombre: "", descripcion: "" };
        axios.get("/notas").then((res) => {
          this.tareas = res.data;
        });
      });
    },
    cancelarEdicion() {
      this.editarActivo = false;
      this.tarea = { nombre: "", descripcion: "" };
    },
  },
};
</script>