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
            <template #userId="{ record }">
              <a-tag color="#87d068" v-if="record.userId == 1">Nuevo</a-tag>
              <a-tag color="#f50" v-else>Antiguo</a-tag>
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
          <a-form-item label="UserId" name="userId" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input type="number" v-model:value="formState.userId" placeholder="Id" />
          </a-form-item>
          <a-form-item label="Titulo" name="title" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input type="text" v-model:value="formState.title" placeholder="titulo" />
          </a-form-item>
          <a-form-item label="Body" name="body" :rules="[{ required: true, message: 'Requerido' }]">
            <a-textarea v-model:value="formState.body" placeholder="..." :rows="4" showCount :maxlength="100" />
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
          <a-form-item label="UserId" name="userId" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input type="number" v-model:value="formState.userId" placeholder="Id" />
          </a-form-item>
          <a-form-item label="Titulo" name="title" :rules="[{ required: true, message: 'Requerido' }]">
            <a-input type="text" v-model:value="formState.title" placeholder="titulo" />
          </a-form-item>
          <a-form-item label="Body" name="body" :rules="[{ required: true, message: 'Requerido' }]">
            <a-textarea v-model:value="formState.body" placeholder="..." :rows="4" showCount :maxlength="100" />
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
    title: "Usuario",
    dataIndex: "userId",
    slots: { customRender: "userId" },
  },
  {
    title: "Titulo",
    dataIndex: "title",
    key: "title",
  },
  {
    title: "Body",
    dataIndex: "body",
    key: "body",
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
      userId: null,
      title: null,
      body: null
    });

    const { resetFields } = useForm(formState, reactive({}));

    return {
      //modal
      add,
      update,

      //Post
      formState,
      resetFields
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
        .get("https://jsonplaceholder.typicode.com/posts")
        .then((response) => {
          this.dataSource = response.data;
        })
        .catch((e) => console.log(e));
    },

    //Post
    post(values) {
      axios
        .post("https://jsonplaceholder.typicode.com/posts", values)
        .then((response) => {
          this.get();
          this.resetFields();
          this.add = false;
          message.success("Guardado");
          return response;
        })
        .catch((e) => console.log(e));
    },

    //Patch
    patch(id) {
      axios
        .get("https://jsonplaceholder.typicode.com/posts/" + id)
        .then((response) => {
          this.formState.id = response.data.id;
          this.formState.userId = response.data.userId;
          this.formState.title = response.data.title;
          this.formState.body = response.data.body;
          this.update = true;
        })
        .catch((e) => console.log(e));
    },

    //Put
    put(values) {
      axios
        .put("https://jsonplaceholder.typicode.com/posts/" + this.formState.id, values)
        .then((response) => {
          this.get();
          this.resetFields();
          this.update = false;
          message.warning("Editado", 2.5);
          return response;
        })
        .catch((e) => console.log(e));
    },

    //Delete
    destroy(id) {
      axios
        .delete("https://jsonplaceholder.typicode.com/posts/" + id)
        .then((response) => {
          this.get();
          message.error("Eliminado", 2.5);
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
