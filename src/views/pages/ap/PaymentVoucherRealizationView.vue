<template>
  <Pages :title="title">
    <CCard class="mb-4">
      <CCardHeader>
        <strong>Realisasi Payment Vouchers</strong>
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
              <CInputGroupText>Date Start</CInputGroupText>
              <CFormInput v-model="filter.dateStart" type="date" />
              <CInputGroupText>Date End</CInputGroupText>
              <CFormInput v-model="filter.dateEnd" type="date" />
              <CButton v-c-tooltip="{ content: 'Filter data', placement: 'top' }" type="button" color="secondary" variant="outline" @click="getData">
                <CIcon icon="cil-filter" width="24" height="24" />
              </CButton>
              <CButton v-c-tooltip="{ content: 'Export to excel', placement: 'top' }" type="button" color="secondary" variant="outline">
                <CIcon icon="cil-save" width="24" height="24" />
              </CButton>
            </CInputGroup>
          </CCol>
        </CRow>
        <CRow>
          <CCol md="12" class="mb-3">
            <CInputGroup>
              <CFormInput v-model="keyword" placeholder="Search..." @keydown.enter="searchData" />
              <CButton v-c-tooltip="{ content: 'Search data', placement: 'top' }" type="button" color="secondary" variant="outline" @click="searchData">
                <CIcon icon="cil-search" width="24" height="24" />
              </CButton>
            </CInputGroup>
          </CCol>
        </CRow>
        <table class="table table-hover table-bordered table-striped table-sm">
          <thead class="align-middle text-center">
            <tr class="table-secondary">
              <th>#</th>
              <th>Nomer</th>
              <th>Tanggal</th>
              <th>Tanggal Tempo</th>
              <th>Vendor</th>
              <th>Status</th>
              <th>User</th>
              <th>User</th>
              <th>Action</th>
            </tr>
          </thead>

          <tbody class="align-middle text-start" v-if="mPayment && mPayment.data && mPayment.data.length > 0">
            <tr v-for="(pv, idx) in filteredData">
              <td>{{ idx + 1 }}</td>
              <td>{{ pv.pv_number }}</td>
              <td class="text-center">{{ formatDate(pv.pv_doc_date) }}</td>
              <td class="text-center">{{ formatDate(pv.pv_due_date) }}</td>
              <td>{{ `${pv.vendors.vend_code} | ${pv.vendors.vend_name}` }}</td>
              <td>{{ formatStatus(pv.payment_voucher_realizations?.pvr_status) }}</td>
              <td>{{ pv.users.name }}</td>
              <td>{{ pv.payment_voucher_realizations?.users.name }}</td>
              <td class="text-center">
                <CButtonGroup>
                  <CButton v-c-tooltip="{ content: 'Detail', placement: 'top' }" color="success" @click="getDetail(pv.id)">
                    <CIcon icon="cil-description" width="20" height="20" />
                  </CButton>
                  <CButton :disabled="pv.pv_status != 4" v-c-tooltip="{ content: 'Edit', placement: 'top' }" :color="pv.pv_status != 4 ? 'danger' : 'primary'" @click="setPVR(pv.id)">
                    <CIcon icon="cil-pen" width="20" height="20" />
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
        <nav v-if="mPayment && mPayment.data && mPayment.data.length > 0" class="mt-2">
          <CButtonGroup role="group">
            <CButton type="button" color="secondary" variant="outline" :disabled="mPayment.current_page === 1" @click="getData(mPayment.current_page - 1)">&#60;</CButton>
            <template v-for="page in calculatePages()">
              <CButton type="button" color="secondary" variant="outline" @click="getData(page)" :class="{ active: page === mPayment.current_page }">{{ page }}</CButton>
            </template>
            <CButton type="button" color="secondary" variant="outline" :disabled="mPayment.current_page === mPayment.last_page" @click="getData(mPayment.current_page + 1)">&#62;</CButton>
          </CButtonGroup>
        </nav>
      </CCardBody>
    </CCard>
  </Pages>
  <CModal
    size="xl"
    alignment="center"
    backdrop="static"
    :visible="modal.pvr"
    @close="
      () => {
        modal.pvr = false;
      }
    "
  >
    <CModalHeader>
      <CModalTitle>Form Input Realisasi Payment Voucher</CModalTitle>
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
        <CCol md="6" class="mb-3">
          <CFormInput type="text" v-model="pvr.number" label="Nomer Voucher" placeholder="Nomer Voucher" />
        </CCol>
        <CCol md="3" class="mb-3">
          <CFormInput type="date" v-model="pvr.relDate" label="Tanggal Realisasi" placeholder="Tanggal Realisasi" />
        </CCol>
        <CCol md="3" class="mb-3">
          <CFormInput type="date" v-model="pvr.dueDate" label="Tanggal Jatuh Tempo" placeholder="Tanggal Jatuh Tempo" />
        </CCol>
      </CRow>
      <CRow>
        <CCol md="12" class="mb-3">
          <CFormTextarea :v-model="pvr.note" type="text" label="Note" placeholder="Note"></CFormTextarea>
        </CCol>
      </CRow>
    </CModalBody>
    <CModalFooter>
      <CButton @click="validPVR" v-c-tooltip="{ content: 'Simpan', placement: 'top' }" color="success">
        <CIcon icon="cil-send" width="24" height="24" />
      </CButton>
    </CModalFooter>
  </CModal>
  <CModal
    :visible="modal.confirmFirst"
    alignment="center"
    backdrop="static"
    @close="
      () => {
        (modal.confirmFirst = false), (modal.pvr = true);
      }
    "
  >
    <CModalHeader>
      <CModalTitle>Konfirmasi</CModalTitle>
    </CModalHeader>
    <CModalBody>Simpan realisasi payment voucher?</CModalBody>
    <CModalFooter>
      <CButton
        v-c-tooltip="{ content: 'Complete', placement: 'top' }"
        color="success"
        @click="
          () => {
            (modal.confirmFirst = false), (modal.confirmSecond = true), (pvr.status = 1), (modal.secondMessage = 'Apakah anda yakin akan menyelesaikan realisasi payment voucher?');
          }
        "
      >
        <CIcon icon="cil-arrow-thick-to-top" width="24" height="24" />
      </CButton>
    </CModalFooter>
  </CModal>
  <CModal
    :visible="modal.confirmSecond"
    alignment="center"
    backdrop="static"
    @close="
      () => {
        (modal.confirmSecond = false), (modal.confirmFirst = true);
      }
    "
  >
    <CModalHeader>
      <CModalTitle>Konfirmasi</CModalTitle>
    </CModalHeader>
    <CModalBody>{{ modal.secondMessage }}</CModalBody>
    <CModalFooter>
      <CButton v-c-tooltip="{ content: 'Confirm', placement: 'top' }" @click="savePVR()" color="success">
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
import { format } from "date-fns";
import numeral from "numeral";
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
      user: JSON.parse(localStorage.getItem("user")),
      modal: {
        action: 1,
        pvId: null,
        pv: false,
        invoice: false,
        approval: false,
        detail: false,
        confirmFirst: false,
        confirmSecond: false,
        secondMessage: "",
      },
      pvr: {
        status: 1,
        number: null,
        relDate: null,
        dueDate: null,
        note: null,
      },
      alert: {
        visibleOnModal: false,
        visibleOnMain: false,
        title: null,
        message: null,
        color: "danger",
      },
      mPayment: [],
      keyword: "",
      filter: {
        perPage: "50",
        status: "7",
        dateStart: null,
        dateEnd: null,
      },
    };
  },
  computed: {
    filteredData() {
      if (this.keyword.trim() === "") {
        return this.mPayment.data;
      }

      const filtered = this.mPayment.data.filter((item) => {
        if (item && item.pv_number) {
          return item.pv_number.toLowerCase().includes(this.keyword.toLowerCase());
        }
        return false;
      });

      if (filtered.length === 0) {
        this.getAlert("warning", "Not Found!", "Data tidak ditemukan pada halaman ini atau keyword salah.", "main");
      }

      return filtered;
    },
  },
  methods: {
    formatDateOther(data) {
      return format(new Date(data), "yyyy-MM-dd");
    },
    formatDate(data) {
      return format(new Date(data), "dd-MM-yyyy");
    },
    formatAmount(number) {
      return numeral(number).format("0,0.00");
    },
    removeSelectedInvoice(index) {
      this.invoice.selectedInvoice.splice(index, 1);
      this.calculatePaymentTotals();
    },
    formatStatus(status) {
      const statusMap = {
        1: "Complete",
        2: "Canceled",
      };
      return statusMap[status] || "Pending";
    },
    setPVR(id) {
      this.modal.pvId = id;
      this.modal.pvr = true;
      console.log(id);
    },
    async searchData() {
      try {
        this.alert.visible = false;
        if (!this.keyword) {
          this.getAlert("danger", "Error!", "Tolong isi keyword pencarian.", "main");
          return;
        } else {
          this.$root.loader = true;
          const data = await ApiService.searchData(this.filter.perPage, this.keyword);
          this.mPayment = data;

          if (this.mPayment.total > 0) {
            this.getAlert("success", "Success!", `Success with ${this.mPayment.total} data.`, "main");
          } else {
            this.getAlert("warning", "Failed!", `Data tidak ditemukan atau keyword salah.`, "main");
          }
        }
      } catch (error) {
        this.getAlert("warning", "Failed!", `Terjadi kesalaahan saat mengambil data.`, "main");
        console.error("Error fetching data:", error);
      } finally {
        this.$root.loader = false;
      }
    },
    async savePVR() {
      try {
        this.$root.loader = true;
        const data = {
          pvr_number: this.pvr.number,
          pvr_status: this.pvr.status,
          pvr_rel_date: this.pvr.relDate,
          pvr_due_date: this.pvr.dueDate,
          pvr_note: this.pvr.note,
          user_id: this.user.id,
        };
        if (this.modal.action == 1) {
          const response = await ApiService.savePVR(data, this.modal.pvId);
        } else if (this.modal.action == 2) {
          const userId = { user: this.user.id };
          const response = await ApiService.cancelPVR(userId, this.modal.pvId);
        }
        // this.setDefault();
        this.getData();
        this.getAlert("success", "Success", "Berhasil menyimpan payment voucher.", "main");
      } catch (error) {
        console.error("Error fetching data:", error);
        this.getAlert("danger", "Error", "Terjadi kesalahan saat menyimpan payment voucher.", "main");
      } finally {
        this.$root.loader = false;
        this.modal.confirmFirst = false;
        this.modal.confirmSecond = false;
      }
    },
    validPVR() {
      this.alert.visibleOnModal = false;
      const isFieldEmpty = (field) => field === undefined || field === null || field === "";

      if (isFieldEmpty(this.pvr.number) || isFieldEmpty(this.pvr.status) || isFieldEmpty(this.pvr.relDate)) {
        this.getAlert("danger", "Error", "Semua data harus diisi dengan lengkap.", "modal");
      } else {
        this.modal.confirmFirst = true;
        this.modal.pvr = false;
      }
    },
    async getData(page = 1) {
      try {
        this.$root.loader = true;
        const response = await ApiService.getPVR(this.filter, page);
        this.mPayment = response.data;
        this.getAlert("success", "Success!", `Success with ${this.mPayment.total} data.`, "main");
      } catch (error) {
        console.error("Error fetching data:", error);
        this.getAlert("danger", "Error", "Terjadi kesalahan saat mengambil data.", "main");
      } finally {
        this.$root.loader = false;
      }
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
      const startPage = Math.max(1, this.mPayment.current_page - 4);
      const endPage = Math.min(startPage + 8, this.mPayment.last_page);

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
