<template>
  <div class="hello">
    <Content>
      <a-row>
        <!--========Guardar========-->
        <a-button @click="add = true" class="editable-add-btn">
          Guardar
        </a-button>

        <!--========Table========-->
        <a-col :span="24">
          <a-table :columns="columns" :dataSource="dataSource" :loading="loading" rowKey="id" bordered>
            <!--Estado-->
            <template #estado="{ record }">
              <a-tag color="#87d068" v-if="record.estado == 'Activado'">{{
                  record.estado
              }}</a-tag>
              <a-tag color="#f50" v-else>{{ record.estado }}</a-tag>
            </template>
            <!--Acciones-->
            <template #acciones="{ record }">
              <a-row>
                <a-col :span="12">
                  <a-button @click="destroy(record.id)">
                    <DeleteOutlined />
                  </a-button>
                </a-col>
                <a-col :span="12">
                  <a-button @click="patch(record.id)">
                    <EditOutlined />
                  </a-button>
                </a-col>
              </a-row>
            </template>
          </a-table>
        </a-col>
      </a-row>

      <!--========Add========-->
      <a-modal v-model:visible="add" title="Guardar">
        <!--Formulario-->
        <a-form id="Form" layout="vertical" autocomplete="off" :model="formState" @finish="post">
          <a-form-item label="Name" name="name" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input type="text" v-model:value="formState.name" placeholder="..." />
          </a-form-item>
          <a-form-item label="Email" name="email" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input type="email" v-model:value="formState.email" placeholder="..." />
          </a-form-item>
          <a-form-item label="Estado" name="estado" :rules="[{ required: true, message: 'Requerido' }]">
            <a-textarea v-model:value="formState.estado" placeholder="..." :rows="4" showCount :maxlength="100" />
          </a-form-item>
        </a-form>
        <!--Footer-->
        <template #footer>
          <a-button type="primary" form="Form" key="submit" htmlType="submit">Submit</a-button>
        </template>
      </a-modal>

      <!--========Update========-->
      <a-modal v-model:visible="update" title="Editar">
        <!--Formulario-->
        <a-form id="Form" layout="vertical" autocomplete="off" :model="formState" @finish="put">
          <a-form-item label="Name" name="name" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input type="text" v-model:value="formState.name" placeholder="..." />
          </a-form-item>
          <a-form-item label="Email" name="email" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input type="email" v-model:value="formState.email" placeholder="..." />
          </a-form-item>
          <a-form-item label="Estado" name="estado" :rules="[{ required: true, message: 'Requerido' }]">
            <a-textarea v-model:value="formState.estado" placeholder="..." :rows="4" showCount :maxlength="100" />
          </a-form-item>
        </a-form>
        <!--Footer-->
        <template #footer>
          <a-button type="primary" form="Form" key="submit" htmlType="submit" danger>Submit</a-button>
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
import { DeleteOutlined, EditOutlined } from "@ant-design/icons-vue";

const useForm = Form.useForm;

//Table
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
  }
];

export default {
  data() {
    return {
      //Get
      dataSource: null,
      columns
    };
  },

  setup() {
    //Modal
    const add = ref(false);
    const update = ref(false);

    //Formulario
    const formState = reactive({
      id: null,
      name: null,
      email: null,
      estado: null
    });

    const { resetFields } = useForm(formState, reactive({}));

    return {
      //Modal
      add,
      update,

      //Post
      formState,
      resetFields,
    };
  },

  components: {
    //Icons
    DeleteOutlined,
    EditOutlined,
  },

  mounted() {
    this.get();
  },

  methods: {
    //Get
    get() {
      axios
        .get("http://api.test/api/user")
        .then((response) => {
          this.dataSource = response.data;
        })
        .catch((e) => console.log(e));
    },

    //post
    post(values) {
      axios
        .post("http://api.test/api/user", values)
        .then((response) => {
          this.get();
          this.resetFields();
          this.add = false;
          message.success("Guardado");
        })
        .catch((e) => console.log(e));
    },

    //Patch
    patch(id) {
      axios
        .get("http://api.test/api/user/" + id)
        .then((response) => {
          this.formState.id = response.data.id;
          this.formState.name = response.data.name;
          this.formState.email = response.data.email;
          this.formState.estado = response.data.estado;
          this.update = true;
        })
        .catch((e) => console.log(e));
    },

    //Put
    put(values) {
      axios
        .put("http://api.test/api/user/" + this.formState.id, values)
        .then((response) => {
          this.get();
          this.resetFields();
          this.update = false;
          message.warning("Editado");
        })
        .catch((e) => console.log(e));
    },

    //Destroy
    destroy(id) {
      axios
        .delete("http://api.test/api/user/" + id)
        .then((response) => {
          this.get();
          message.error("Eliminado");
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