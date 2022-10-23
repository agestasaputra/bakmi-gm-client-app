<template>
  <b-container class="mx-auto vh-100">
    <div v-if="table.isLoading" class="d-flex align-items-center justify-content-center h-100">
      <b-spinner variant="primary" label="Spinning"></b-spinner>
    </div>
    <template v-else>
      <div class="text-right">
        <b-button variant="primary" class="my-3" @click="onButtonCreateClick">
          Create
        </b-button>
      </div>
      <b-table hover :items="table.items" :fields="table.fields">
        <template #cell(actions)="row">
          <b-button variant="outline-primary" class="mx-1" @click="onButtonEditClick(row.item)">
            Edit
          </b-button>
          <b-button variant="danger" @click="onButtonDeleteClick(row.item)">
            Delete
          </b-button>
        </template>
      </b-table>
    </template>

    <!-- Modal Create -->
    <b-modal 
      v-if="modal.create.isShow"
      v-model="modal.create.isShow"
      hide-header
      hide-footer
      no-close-on-backdrop
    >
      <template v-slot:default>
        <b-form @submit.prevent="onCreateSubmit">
          <b-form-group label="Code:" label-for="input-code">
            <b-form-input
              id="input-code"
              v-model="form.code"
              placeholder="Ex: P0001"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group label="Name:" label-for="input-name">
            <b-form-input
              id="input-name"
              v-model="form.name"
              placeholder="Ex: Bakso Malang"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group label="Description:" label-for="input-description">
            <b-form-textarea
              id="input-description"
              v-model="form.description"
              placeholder="Ex: Jln. A, Jakarta Barat"
              rows="3"
              max-rows="6"
            ></b-form-textarea>
          </b-form-group>

          <b-form-group label="Category:" label-for="input-category">
            <b-form-select
              id="input-category"
              v-model="form.category"
              :options="list.categories"
              required
            ></b-form-select>
          </b-form-group>

          <b-form-group label="Price:" label-for="input-price" description="*Number only">
            <b-form-input
              id="input-price"
              v-model="form.price"
              type="text"
              placeholder="Ex: 30000"
              pattern="[0-9]+"
              required
            ></b-form-input>
          </b-form-group>

          <div class="text-right">
            <b-button variant="outline-danger" class="mx-1" @click="onModalClose('create')">
              Cancel
            </b-button>
            <b-button type="submit" :disable="true" variant="primary">
              Create
            </b-button>
          </div>
        </b-form>
      </template>
    </b-modal>

    <!-- Modal Edit -->
    <b-modal 
      v-if="modal.edit.isShow"
      v-model="modal.edit.isShow"
      hide-header
      hide-footer
      no-close-on-backdrop
    >
      <template v-slot:default>
        <b-form @submit.prevent="onEditSubmit">
          <b-form-group label="Code:" label-for="input-code">
            <b-form-input
              id="input-code"
              v-model="form.code"
              placeholder="Ex: P0001"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group label="Name:" label-for="input-name">
            <b-form-input
              id="input-name"
              v-model="form.name"
              placeholder="Ex: Bakso Malang"
              required
            ></b-form-input>
          </b-form-group>

          <b-form-group label="Description:" label-for="input-description">
            <b-form-textarea
              id="input-description"
              v-model="form.description"
              placeholder="Ex: Jln. A, Jakarta Barat"
              rows="3"
              max-rows="6"
            ></b-form-textarea>
          </b-form-group>

          <b-form-group label="Category:" label-for="input-category">
            <b-form-select
              id="input-category"
              v-model="form.category"
              :options="list.categories"
              required
            ></b-form-select>
          </b-form-group>

          <b-form-group label="Price:" label-for="input-price" description="*Number only">
            <b-form-input
              id="input-price"
              v-model="form.price"
              type="text"
              placeholder="Ex: 30000"
              pattern="[0-9]+"
              required
            ></b-form-input>
          </b-form-group>

          <div class="text-right">
            <b-button variant="outline-danger" class="mx-1" @click="onModalClose('edit')">
              Cancel
            </b-button>
            <b-button type="submit" :disable="true" variant="primary">
              Save
            </b-button>
          </div>
        </b-form>
      </template>
    </b-modal>

    <!-- Modal Delete -->
    <b-modal 
      v-if="modal.delete.isShow"
      v-model="modal.delete.isShow"
      hide-header
      no-close-on-backdrop
    >
      <template v-slot:default>
        Are you sure want to delete <strong>{{ form.name }}</strong>?
      </template>
      <template v-slot:modal-footer>
        <b-button variant="outline-success" class="mx-1" @click="onModalClose('delete')">
          Cancel
        </b-button>
        <b-button variant="danger" @click="onDeleteSubmit">
          Delete
        </b-button>
      </template>
    </b-modal>
  </b-container>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data() {
    return {
      list: {
        categories: [
          { value: null, text: 'Ex: Makanan', disabled: true },
          { value: 'Makanan', text: 'Makanan' },
          { value: 'Minuman', text: 'Minuman' },
          { value: 'Lain-Lain', text: 'Lain-Lain' },
        ]
      },
      form: {
        code: '',
        name: '',
        description: '',
        category: null,
        price: null,
      },
      selectedData: null,
      modal: {
        create: {
          isShow: false,
        },
        edit: {
          isShow: false,
        },
        delete: {
          isShow: false,
        }
      },
      table: {
        isLoading: false,
        // Note `isActive` is left out and will not appear in the rendered table
        fields: [
          {
            key: 'code',
            label: 'Code',
            sortable: true
          },
          {
            key: 'name',
            label: 'Name',
            sortable: true
          },
          // {
          //   key: 'description',
          //   label: 'Description',
          //   sortable: true,
          // },
          {
            key: 'category',
            label: 'Category',
            sortable: true,
          },
          {
            key: 'price',
            label: 'Price',
            sortable: true,
            // Variant applies to the whole column, including the header and footer
            // variant: 'danger'
          },
          {
            key: 'actions',
            label: 'Actions',
            sortable: false,
          }
        ],
        items: []
      }
    }
  },
  mounted() {
    this.onFetchProducts()
  },
  methods: {
    async onFetchProducts() {
      this.table.isLoading = true
      try {
        const { data: { datas } } = await axios.get('http://localhost:8080/api/products')
        this.table.items = datas
      } catch (error) {
        this.$bvToast.toast(error.message, {
          title: `Error`,
          variant: "danger",
          solid: true
        })
      }

      setTimeout(() => {
        this.table.isLoading = false
      }, 1000);
    },
    onModalOpen(type) {
      this.modal[type].isShow = true
    },
    onModalClose(type) {
      this.form = {
        code: '',
        name: '',
        description: '',
        category: null,
        price: null,
      }
      this.modal[type].isShow = false
    },
    onButtonCreateClick() {
      this.modal.create.isShow = true
      this.onModalOpen('create')
    },
    onButtonEditClick(data) {
      this.form = {
        code: data.code,
        name: data.name,
        description: data.description,
        category: data.category,
        price: data.price
      }
      this.onModalOpen('edit')
    },
    onButtonDeleteClick(data) {
      this.form = {
        code: data.code,
        name: data.name,
        description: data.description,
        category: data.category,
        price: data.price
      }
      this.onModalOpen('delete')
    },
    async onCreateSubmit() {
      try {
        const payload = { ...this.form }
        const { data } = await axios.post('http://localhost:8080/api/products', payload)
        this.onModalClose('create')
        this.onFetchProducts()
        this.$bvToast.toast(data.message, {
          title: `Success`,
          variant: "success",
          solid: true
        })
      } catch(error) {
        this.$bvToast.toast(error.message, {
          title: `Error`,
          variant: "danger",
          solid: true
        })
      }
    },
    async onEditSubmit() {
      try {
        const payload = { ...this.form }
        const { data } = await axios.put(`http://localhost:8080/api/products/${this.form.code}`, payload)
        this.onModalClose('edit')
        this.onFetchProducts()
        this.$bvToast.toast(data.message, {
          title: `Success`,
          variant: "success",
          solid: true
        })
      } catch(error) {
        this.$bvToast.toast(error.message, {
          title: `Error`,
          variant: "danger",
          solid: true
        })
      }
    },
    async onDeleteSubmit() {
      try {
        const { data } = await axios.delete(`http://localhost:8080/api/products/${this.form.code}`)
        this.$bvToast.toast(data.message, {
          title: `Success`,
          variant: "success",
          solid: true
        })
        this.onModalClose('delete')
        this.onFetchProducts()
      } catch(error) {
        this.$bvToast.toast(error.message, {
          title: `Error`,
          variant: "danger",
          solid: true
        })
      }
    },
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
