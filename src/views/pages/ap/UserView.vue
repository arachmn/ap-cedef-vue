<template>
  <Pages :title="title">
    <CCard class="mb-4">
      <CCardHeader class="d-flex justify-content-between align-items-center">
        <strong>User</strong>
        <CButton v-c-tooltip="{ content: 'Tambah User', placement: 'top' }" @click="openmodalNewUser" color="success">
          <CIcon icon="cil-note-add" width="24" height="24" />
        </CButton>
      </CCardHeader>
      <CCardBody>
        <CAlert
          class="d-flex align-items-center"
          :color="alert.color"
          :visible="alert.visibleOnMain"
          dismissible
          @close="
            () => {
              alert.visibleOnMain = false;
            }
          "
        >
          <CIcon v-if="alert.color == 'warning'" icon="cil-warning" class="flex-shrink-0 me-2" width="24" height="24" />
          <CIcon v-if="alert.color == 'success'" icon="cil-check-circle" class="flex-shrink-0 me-2" width="24" height="24" />
          <CIcon v-if="alert.color == 'danger'" icon="cil-x-circle" class="flex-shrink-0 me-2" width="24" height="24" />
          <div>
            <strong>{{ alert.title }}</strong> {{ alert.message }}
          </div>
        </CAlert>
        <CRow>
          <CCol md="12" class="mb-3">
            <CInputGroup>
              <CInputGroupText>Data</CInputGroupText>
              <CFormSelect v-model="filter.perPage">
                <option value="25">25</option>
                <option value="50">50</option>
                <option value="75">75</option>
                <option value="100">100</option>
              </CFormSelect>
              <CButton v-c-tooltip="{ content: 'Filter data', placement: 'top' }" type="button" color="secondary" variant="outline" @click="getData">
                <CIcon icon="cil-filter" width="24" height="24" />
              </CButton>
            </CInputGroup>
          </CCol>
        </CRow>
        <table class="table table-hover table-bordered table-striped table-sm">
          <thead class="align-middle text-center">
            <tr class="table-secondary">
              <th>#</th>
              <th>Departemen</th>
              <th>Username</th>
              <th>Nama</th>
              <th>Role</th>
              <th>Status</th>
              <th>Action</th>
            </tr>
          </thead>

          <tbody class="align-middle text-start" v-if="mUser && mUser.data && mUser.data.length > 0">
            <tr v-for="(sj, idx) in mUser.data">
              <td>{{ idx + 1 }}</td>
              <td>{{ sj.departements.dep_name }}</td>
              <td>{{ sj.username }}</td>
              <td>{{ sj.name }}</td>
              <td>{{ sj.roles.role_name }}</td>
              <td>{{ sj.status }}</td>
              <td class="text-center">
                <CButtonGroup>
                  <CButton v-c-tooltip="{ content: 'Edit', placement: 'top' }" color="primary" @click="getEdit(sj.id)">
                    <CIcon icon="cil-pen" width="20" height="20" />
                  </CButton>
                  <CButton v-c-tooltip="{ content: 'Cancel', placement: 'top' }" color="danger" @click="getDisable(sj.id)">
                    <CIcon icon="cil-x-circle" width="20" height="20" />
                  </CButton>
                </CButtonGroup>
              </td>
            </tr>
          </tbody>
          <tbody v-else>
            <tr>
              <td colspan="9" class="text-center">Tidak ada data.</td>
            </tr>
          </tbody>
        </table>
        <nav v-if="mUser && mUser.data && mUser.data.length > 0" class="mt-2">
          <CButtonGroup role="group">
            <CButton type="button" color="secondary" variant="outline" :disabled="mUser.current_page === 1" @click="getData(mUser.current_page - 1)">&#60;</CButton>
            <template v-for="page in calculatePages()">
              <CButton type="button" color="secondary" variant="outline" @click="getData(page)" :class="{ active: page === mUser.current_page }">{{ page }}</CButton>
            </template>
            <CButton type="button" color="secondary" variant="outline" :disabled="mUser.current_page === mUser.last_page" @click="getData(mUser.current_page + 1)">&#62;</CButton>
          </CButtonGroup>
        </nav>
      </CCardBody>
    </CCard>
  </Pages>
  <CModal
    size="xl"
    alignment="center"
    backdrop="static"
    :visible="modal.user"
    @close="
      () => {
        modal.user = false;
        setDefault();
      }
    "
  >
    <CModalHeader>
      <CModalTitle>Form Input User</CModalTitle>
    </CModalHeader>
    <CModalBody>
      <CAlert
        class="d-flex align-items-center"
        :color="alert.color"
        :visible="alert.visibleOnModal"
        dismissible
        @close="
          () => {
            alert.visibleOnModal = false;
          }
        "
      >
        <CIcon v-if="alert.color == 'warning'" icon="cil-warning" class="flex-shrink-0 me-2" width="24" height="24" />
        <CIcon v-if="alert.color == 'success'" icon="cil-check-circle" class="flex-shrink-0 me-2" width="24" height="24" />
        <CIcon v-if="alert.color == 'danger'" icon="cil-x-circle" class="flex-shrink-0 me-2" width="24" height="24" />
        <div>
          <strong>{{ alert.title }}</strong> {{ alert.message }}
        </div>
      </CAlert>
      <CRow>
        <CCol md="12" class="mb-3">
          <CFormInput v-model="user.name" type="text" label="Nama User" placeholder="Nama User" />
        </CCol>
      </CRow>
      <CRow>
        <CCol md="6" class="mb-3">
          <CFormSelect v-model="user.role" label="Role">
            <option v-for="role in roles" :key="role.id" :value="parseString(role.id)">
              {{ role.role_name }}
            </option>
          </CFormSelect>
        </CCol>
        <CCol md="6" class="mb-3">
          <CFormSelect v-model="user.dep" label="Departement">
            <option v-for="dep in departements" :key="dep.id" :value="dep.dep_code">
              {{ dep.dep_name }}
            </option>
          </CFormSelect>
        </CCol>
      </CRow>
      <CRow>
        <CCol md="6" class="mb-3">
          <CFormInput v-model="user.username" type="text" label="Username" placeholder="Username" />
        </CCol>
        <CCol md="6" class="mb-3">
          <CFormInput v-model="user.password" type="text" label="Password" placeholder="Password" />
        </CCol>
      </CRow>
    </CModalBody>
    <CModalFooter>
      <CButton v-c-tooltip="{ content: 'Simpan', placement: 'top' }" @click="validUser" color="success">
        <CIcon icon="cil-send" width="24" height="24" />
      </CButton>
    </CModalFooter>
  </CModal>
  <CModal
    :visible="modal.cancel"
    alignment="center"
    backdrop="static"
    @close="
      () => {
        modal.cancel = false;
      }
    "
  >
    <CModalHeader>
      <CModalTitle>Konfirmasi</CModalTitle>
    </CModalHeader>
    <CModalBody>Apakah anda yakin akan menonaktifkan User?</CModalBody>
    <CModalFooter>
      <CButton v-c-tooltip="{ content: 'Confirm', placement: 'top' }" @click="cancelUser" color="success">
        <CIcon icon="cil-send" width="24" height="24" />
      </CButton>
    </CModalFooter>
  </CModal>
</template>

<style>
.table td {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-left: 0.75rem !important;
  padding-right: 0.75rem !important;
}

.table th {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

#mySecondary {
  background-color: rgb(226, 227, 229);
  color: rgb(0, 0, 0);
  box-shadow: rgba(0, 0, 0, 0) 0px 0px 0px 9999px inset;
}
</style>

<script>
import Pages from "@/components/template/Pages.vue";
import ApiService from "@/services/api/api.js";
import vSelect from "vue-select";
import "vue-select/dist/vue-select.css";

export default {
  components: {
    Pages,
    vSelect,
  },
  data() {
    return {
      title: "A/P GENERAL",
      modal: {
        user: false,
        cancel: false,
        userId: null,
        action: 1,
      },
      mUser: [],
      roles: [],
      departements: [],
      // user: JSON.parse(localStorage.getItem("user")),
      alert: {
        visibleOnModal: false,
        visibleOnMain: false,
        title: null,
        message: null,
        color: "danger",
      },
      user: {
        selectedId: null,
        name: null,
        username: null,
        password: null,
        role: "2",
        dep: null,
      },
      filter: {
        perPage: "50",
      },
    };
  },
  methods: {
    async getData(page = 1) {
      try {
        this.$root.loader = true;
        const response = await ApiService.getUser(page);
        this.mUser = response.data;
        this.getAlert("success", "Success!", `Success with ${this.mUser.total} data.`, "main");
      } catch (error) {
        console.error("Error fetching data:", error);
        this.getAlert("danger", "Error", "Terjadi kesalahan saat mengambil data.", "main");
      } finally {
        this.$root.loader = false;
      }
    },
    async cancelUser() {
      try {
        this.$root.loader = true;
        const response = await ApiService.cancelUser(this.user.selectedId);
        this.getData();
      } catch (error) {
        console.error("Error fetching data:", error);
        this.getAlert("danger", "Error", "Terjadi kesalahan saat membatalkan User.", "main");
      } finally {
        this.$root.loader = false;
        this.modal.cancel = false;
      }
    },
    async openmodalNewUser() {
      try {
        this.$root.loader = true;
        const responseDep = await ApiService.getDep();
        const responseRole = await ApiService.getRole();

        this.roles = responseRole.data;
        this.departements = responseDep.data;

        this.modal.user = true;
        this.modal.action = 1;
      } catch (error) {
        console.error("Error fetching data:", error);
        this.getAlert("danger", "Error", "Terjadi kesalahan saat membatalkan User.", "main");
      } finally {
        this.$root.loader = false;
        this.modal.cancel = false;
      }
    },
    getCancelUser(id) {
      this.user.selectedId = id;
      this.modal.cancel = true;
    },
    async saveUser() {
      try {
        this.$root.loader = true;
        const data = {
          name: this.user.name,
          username: this.user.username,
          password: this.user.password,
          role_id: this.user.role,
          dep_code: this.user.dep,
          status: 1,
        };
        if (this.modal.action == 1) {
          const response = await ApiService.saveUser(data);
        } else if (this.modal.action == 2) {
          const response = await ApiService.editUser(data, this.modal.userId);
        }
        this.getData();
        this.getAlert("success", "Success", "Berhasil menyimpan User.", "main");
      } catch (error) {
        console.error("Error fetching data:", error);
        this.getAlert("danger", "Error", "Terjadi kesalahan saat menyimpan User.", "main");
      } finally {
        this.$root.loader = false;
        this.modal.confirmFirst = false;
        this.modal.confirmSecond = false;
      }
    },
    validUser() {
      if (!this.user.name || !this.user.password || !this.user.username || !this.user.role || !this.user.dep) {
        this.getAlert("danger", "Error", "Semua data harus diisi dengan lengkap.", "modal");
      } else {
        this.modal.user = false;
        this.saveUser();
      }
    },
    parseString(value) {
      return value.toString();
    },
    getAlert(color, title, message, target) {
      this.alert.title = title;
      this.alert.color = color;
      this.alert.message = message;
      if (target == "modal") {
        this.alert.visibleOnModal = true;
      } else if (target == "main") {
        this.alert.visibleOnMain = true;
      }
    },
    calculatePages() {
      const startPage = Math.max(1, this.mUser.current_page - 4);
      const endPage = Math.min(startPage + 8, this.mUser.last_page);

      return Array.from(
        {
          length: endPage - startPage + 1,
        },
        (_, i) => startPage + i
      );
    },
  },
};
</script>
