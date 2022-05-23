<template>
  <div class="hello">
    <Content>
      <a-row>
        <!--========Guardar========-->
        <a-button @click="showModal" class="editable-add-btn">
          Guardar
        </a-button>

        <!--========Table========-->
        <a-col :span="24">
          <a-table :columns="columns" :dataSource="dataSource" :loading="loading" rowKey="id" bordered>
            <!--=Estado=-->
            <template #estado="{ record }">
              <a-tag color="#87d068" v-if="record.estado == 'Activado'">{{
                  record.estado
              }}</a-tag>
              <a-tag color="#f50" v-else>{{ record.estado }}</a-tag>
            </template>
            <!--=Eliminar=-->
            <template #acciones="{ record }">
              <a-button :loading="record.loading" @click="eliminar(record.id), (record.loading = true)">Eliminar
              </a-button>
            </template>
          </a-table>
        </a-col>
      </a-row>

      <!--========Modal========-->
      <a-modal v-model:visible="visible" title="Guardar">
        <!--=Formulario=-->
        <a-form id="Form" layout="vertical" autocomplete="off" :model="formState" @finish="guardar">
          <a-form-item label="Name" name="name" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input v-model:value="formState.name" />
          </a-form-item>
          <a-form-item label="Email" name="email" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input v-model:value="formState.email" />
          </a-form-item>
          <a-form-item label="Estado" name="estado" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input v-model:value="formState.estado" />
          </a-form-item>
        </a-form>
        <!--=Footer=-->
        <template #footer>
          <a-button type="primary" form="Form" key="submit" htmlType="submit">Enviar</a-button>
        </template>
      </a-modal>
    </Content>
  </div>
</template>

<!--========Script========-->
<script>
import axios from "axios";
import { ref } from "vue";
import { reactive } from "vue";
import { Form } from "ant-design-vue";
import { message } from "ant-design-vue";

const useForm = Form.useForm;

const columns = [
  {
    title: "Id",
    dataIndex: "id",
    key: "id",
  },
  {
    title: "Role",
    dataIndex: "role.descripcion",
    key: "role.descripcion",
  },
  {
    title: "Name",
    dataIndex: "name",
    key: "name",
  },
  {
    title: "Email",
    dataIndex: "email",
    key: "email",
  },
  {
    title: "Estado",
    dataIndex: "estado",
    slots: { customRender: "estado" },
  },
  {
    title: "Acciónes",
    dataIndex: "acciones",
    slots: { customRender: "acciones" },
  },
];

export default {
  data() {
    return {
      //Get
      dataSource: null,
      columns,
    };
  },

  setup() {
    //Modal
    const visible = ref(false);

    const showModal = () => {
      visible.value = true;
    };
    const notModal = () => {
      visible.value = false;
      resetFields();
    };

    //Post
    const formState = reactive({
      name: null,
      email: null,
      estado: null,
    });

    const { resetFields } = useForm(formState, reactive({}));

    return {
      //modal
      visible,
      showModal,
      notModal,

      //Post
      formState,
      resetFields,
    };
  },

  mounted() {
    this.get();
  },

  methods: {
    //Get
    get() {
      axios
        .get("http://api.test/public/api/user")
        .then((response) => {
          console.log(response)
          this.dataSource = response.data;
        })
        .catch((e) => console.log(e));
    },

    //post
    guardar(values) {
      axios
        .post("http://api.test/public/api/user", values)
        .then((response) => {
          this.get();
          this.notModal();
          message.success("Guardado");
          return response;
        })
        .catch((e) => console.log(e));
    },

    //Delete
    eliminar(id) {
      axios
        .delete("http://api.test/public/api/user/" + id)
        .then((response) => {
          this.get();
          message.success("Eliminado");
          return response;
        })
        .catch((e) => console.log(e));
    },
  },
};
</script>

<!--========Css========-->
<style scoped>
content {
  display: block;
  margin: 0 50px;
}

.editable-add-btn {
  margin: 0 0 20px 0;
}
</style>
