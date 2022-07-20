<template>
  <div>
    <v-data-table
      :headers="headers"
      :items="person"
      sort-by="id"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Person CRUD</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                New Item
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.nome"
                        label="Nome"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        v-model="editedItem.dtNascimento"
                        label="Data de nascimento"
                        type="date"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-alert dismissible v-if="isError" type="error">
                {{ isError }}
              </v-alert>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">
                  Cancel
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  text
                  v-if="formTitle === 'New Item'"
                  @click="save"
                >
                  Save
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  text
                  v-if="formTitle === 'Edit Item'"
                  @click="update"
                >
                  Update
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5"
                >Are you sure you want to delete this item?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete"
                  >Cancel</v-btn
                >
                <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                  >OK</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
        <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
      </template>
    </v-data-table>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'IndexPage',
  data: () => ({
    dialog: false,
    dialogDelete: false,
    headers: [
      { text: 'ID', value: 'id' },
      { text: 'Name', value: 'nome' },
      { text: 'age', value: 'idade' },
      { text: 'Actions', value: 'actions', sortable: false },
    ],
    person: [],
    editedIndex: -1,

    editedItem: {
      nome: '',
      dtNascimento: '',
    },
    defaultItem: {
      nome: '',
      dtNascimento: '',
    },
    updateItem: '',
    delete: '',
    isError: false,
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    },
  },

  watch: {
    dialog(val) {
      val || this.close()
    },
    dialogDelete(val) {
      val || this.closeDelete()
    },
  },

  created() {
    this.initialize()
  },

  methods: {
    initialize() {
      axios.get('http://localhost:4000/person').then((res) => {
        this.person = res.data
      })
    },

    editItem(item) {
      this.editedIndex = this.person.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.updateItem = item
      this.dialog = true
    },

    update() {
      if (this.editedItem.nome === '' || this.editedItem.dtNascimento === '') {
        return (this.isError = 'Both fields are mandatory')
      }

      const data = {
        id: this.updateItem.id,
        nome: this.editedItem.nome,
        dtNascimento: new Date(this.editedItem.dtNascimento).toISOString(),
      }

      axios
        .put('http://localhost:4000/personid', data, {
          headers: {
            Accept: 'application/json',
          },
        })
        .then(() => {
          this.initialize()
          this.close()
        })
        .catch((e) => {
          this.isError = e.data ? false : 'Something went wrong'
        })
    },

    deleteItem(item) {
      this.delete = item
      this.dialogDelete = true
    },

    deleteItemConfirm() {
      axios
        .delete('http://localhost:4000/personid/' + this.delete.id)
        .then(() => {
          this.initialize()
          this.closeDelete()
        })
        .catch((e) => {
          this.isError = e.data ? false : 'Something went wrong'
        })
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    save() {
      if (this.editedItem.nome === '' || this.editedItem.dtNascimento === '') {
        this.isError = 'Both fields are mandatory'
        return setTimeout(() => {
          this.isError = false
        }, 5000)
      }
      let data = {
        nome: this.editedItem.nome,
        dtNascimento: new Date(this.editedItem.dtNascimento).toISOString(),
      }

      axios
        .post('http://localhost:4000/person', data, {
          headers: {
            Accept: 'application/json',
          },
        })
        .then(() => {
          this.initialize()
          this.close()
        })
        .catch((e) => {
          this.isError = e.data ? false : 'Something went wrong'
        })
    },
  },
}
</script>
