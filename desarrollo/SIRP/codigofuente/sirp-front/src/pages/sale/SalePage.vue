<template>
  <div id="social">
    <vue-perfect-scrollbar
      class="drawer-menu--scroll"
      :settings="scrollSettings"
    >
      <v-container fluid>
        <v-row>
          <v-col cols="12">
            <h4>Mis ventas</h4>
            <v-spacer></v-spacer>
            <v-btn
              small
              color="primary"
              @click="$router.push({ name: 'SaleRegisterPage' })"
            >
              Agregar venta
            </v-btn>
          </v-col>
        </v-row>
        <v-row>
          <template v-if="!loading">
            <div class="text-xs-center">
              <v-progress-circular
                :size="50"
                color="primary"
                indeterminate
              ></v-progress-circular>
            </div>
          </template>
          <template v-else>
            <v-col cols="12">
              <v-card>
                <v-card-title>
                  <v-spacer></v-spacer>
                  <v-text-field
                    v-model="searchInvitation"
                    append-icon="search"
                    label="Buscar por Nombre"
                    single-line
                    hide-details
                    class="mt-0 pt-0"
                  ></v-text-field>
                </v-card-title>
                <v-data-table
                  :headers="headers"
                  :items="sales"
                  :search="searchInvitation"
                  item-key="id"
                  hide-default-footer
                  class="elevation-2"
                >
                  <template v-slot:headers="{ props: { headers } }">
                    <thead>
                      <tr>
                        <th v-for="item in headers" :key="item.value">
                          <span :key="`span_${item.value}`">{{
                            item.text
                          }}</span>
                        </th>
                      </tr>
                    </thead>
                  </template>
                  <template v-slot:item.action="{ item }">
                    <v-menu top offset-y>
                      <template v-slot:activator="{ on }">
                        <v-btn icon v-on="on">
                          <v-icon>more_vert</v-icon>
                        </v-btn>
                      </template>

                      <v-list>
                        <v-list-item @click="onDialogUpdate(item, 'edit')">
                          <v-list-item-title>Editar</v-list-item-title>
                        </v-list-item>
                        <v-list-item @click="openDialogDelete(item)">
                          <v-list-item-title>Eliminar</v-list-item-title>
                        </v-list-item>
                      </v-list>
                    </v-menu>
                  </template>
                </v-data-table>
                <v-pagination
                  v-model="currentPage"
                  class="mt-3"
                  :length="totalPages"
                  :total-visible="10"
                  @input="onChangePagination"
                ></v-pagination>
              </v-card>
            </v-col>
          </template>
        </v-row>
      </v-container>
    </vue-perfect-scrollbar>

    <v-dialog v-model="dialogAdd" persistent max-width="600px">
      <product-form
        :type="typeDialog"
        :product="selectedItem"
        @close="dialogAdd = false"
        @submit="dialogAdd = false"
      />
    </v-dialog>

    <v-dialog v-model="dialogDelete" persistent max-width="450px">
      <v-card v-if="dialogDelete">
        <v-card-title>
          <span class="subheading">Eliminar Invitado</span>
        </v-card-title>
        <v-card-text>
          <div>
            ¿Estás seguro que deseas eliminar a {{ selectedItem.name }}?
          </div>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn flat @click="dialogDelete = false">No</v-btn>
          <v-btn
            color="blue darken-1"
            :loading="loadDelete"
            flat
            @click="onDelete()"
            >Sí</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
import VuePerfectScrollbar from "vue-perfect-scrollbar";
import { mapState, mapActions } from "vuex";
import ProductForm from "@/components/product/ProductForm";

export default {
  name: "SalePage",
  components: {
    ProductForm,
    VuePerfectScrollbar
  },
  data() {
    return {
      scrollSettings: {
        maxScrollbarLength: 160
      },
      searchInvitation: "",
      dialogDelete: false,
      dialogDetail: false,
      dialogAdd: false,
      loadDelete: false,
      selectedItem: {},
      typeDialog: "new",
      currentPage: 1,
      headers: [
        { text: "Serie", value: "serie" },
        { text: "Total", value: "total" },
        { text: "Fecha", value: "sale_date" },
        { text: "Acción", value: "action" }
      ]
    };
  },
  computed: {
    ...mapState({
      loading: state => state.sales.loading,
      sales: state => state.sales.items,
      totalPages: state => state.sales.totalPages,
      currentUser: state => state.session.currentUser
    })
  },
  async mounted() {
    await this.fetchSales(this.currentPage);
  },
  methods: {
    ...mapActions(["fetchSales"]),
    async onChangePagination(val) {
      await this.fetchSales(val);
    },
    onDialogUpdate(item = null, type) {
      if (item) {
        this.typeDialog = type;
      }
      this.selectedItem = item;
      this.dialogAdd = true;
    },
    openDialogDelete(item) {
      this.selectedItem = item;
      this.dialogDelete = true;
    },
    onDelete() {
      this.loadDelete = true;
      this.$store
        .dispatch("deleteInvitation", this.selectedItem.id)
        .then(() => {
          this.loadDelete = false;
          this.dialogDelete = false;
          this.selectedItem = {};
        });
    },
    openDialogDetail(item) {
      this.selectedItem = item;
      this.dialogDetail = true;
    }
  }
};
</script>
