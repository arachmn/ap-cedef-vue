<template>
  <Pages :title="title">
    <CCard class="mb-4">
      <CCardHeader class="d-flex justify-content-between align-items-center">
        <strong>Payment Vouchers</strong>
        <CButton v-c-tooltip="{ content: 'Tambah Payment Vouchers', placement: 'top' }" @click="openModalNewPV" color="success">
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
              <CInputGroupText>Status</CInputGroupText>
              <CFormSelect v-model="filter.status">
                <option value="7">All</option>
                <option value="1">Draft</option>
                <option value="2">Complete</option>
                <option value="3">Canceled</option>
                <option value="4">Approved</option>
                <option value="5">Rejected</option>
                <option value="6">Closed</option>
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
              <td>{{ formatStatus(pv.pv_status) }}</td>
              <td>{{ pv.users.name }}</td>
              <td class="text-center">
                <CButtonGroup>
                  <CButton v-c-tooltip="{ content: 'Export Docx', placement: 'top' }" color="secondary" @click="getDocx(pv.id)">
                    <CIcon icon="cil-arrow-thick-from-top" width="20" height="20" />
                  </CButton>
                  <CButton v-c-tooltip="{ content: 'Detail', placement: 'top' }" color="success" @click="getDetail(pv.id)">
                    <CIcon icon="cil-description" width="20" height="20" />
                  </CButton>
                  <CButton :disabled="pv.pv_status != 1" v-c-tooltip="{ content: 'Edit', placement: 'top' }" :color="pv.pv_status != 1 ? 'danger' : 'primary'" @click="getEdit(pv.id)">
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
    :visible="modal.pv"
    @close="
      () => {
        modal.pv = false;
      }
    "
  >
    <CModalHeader>
      <CModalTitle>Form Input Payment Voucher</CModalTitle>
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
        <CCol md="5">
          <label class="mb-2">Vendor</label>
          <v-select v-model="vendors.selectedVendor" @option:selected="setData()" :options="vendors.data" :filterable="true" label="id_vendname" @search="searchVendorsGeneral" :loading="vendors.isLoading"></v-select>
        </CCol>
        <CCol md="5">
          <label class="mb-2">Bank</label>
          <v-select :disabled="!vendors.selectedVendor" v-model="bank.selectedBank" :options="bank.data" :filterable="true" label="bank_n_code" @search="searchBank" :loading="bank.isLoading"></v-select>
        </CCol>
        <CCol md="2" class="mb-3">
          <CFormInput type="date" v-model="payment.dueDate" label="Tanggal Jatuh Tempo" placeholder="Tanggal Jatuh Tempo" />
        </CCol>
      </CRow>
      <CRow>
        <CCol md="4" class="mb-3">
          <CFormInput type="text" disabled v-model="payment.accountDppDesc" label="Account DPP" placeholder="Account DPP" />
        </CCol>
        <CCol md="4" class="mb-3">
          <CFormInput type="text" disabled v-model="payment.accountPpnDesc" label="Account PPN" placeholder="Account PPN" />
        </CCol>
        <CCol md="4" class="mb-3">
          <CFormInput type="text" disabled v-model="payment.accountPphDesc" label="Account PPH" placeholder="Account PPH" />
        </CCol>
      </CRow>
      <CRow>
        <CCol md="3" class="mb-3">
          <CFormInput type="text" disabled v-model="payment.dpp" label="Nilai DPP" placeholder="Nilai DPP" />
        </CCol>
        <CCol md="3" class="mb-3">
          <CFormInput type="text" disabled v-model="payment.ppn" label="Nilai PPN" placeholder="Nilai PPN" />
        </CCol>
        <CCol md="3" class="mb-3">
          <CFormInput type="text" disabled v-model="payment.pph" label="Nilai PPH" placeholder="Nilai PPH" />
        </CCol>
        <CCol md="3" class="mb-3">
          <CFormInput type="text" disabled v-model="payment.total" label="Nilai Total" placeholder="Nilai Total" />
        </CCol>
      </CRow>
      <CRow>
        <CCol md="12" class="mb-3">
          <CFormTextarea :v-model="payment.note" type="text" label="Note" placeholder="Note"></CFormTextarea>
        </CCol>
      </CRow>
    </CModalBody>
    <CModalFooter>
      <CButton
        :disabled="!vendors.selectedVendor"
        v-c-tooltip="{ content: 'Tambah Approval', placement: 'top' }"
        @click="
          () => {
            modal.approval = true;
            modal.pv = false;
          }
        "
        color="primary"
      >
        <CIcon icon="cil-pen-alt" width="24" height="24" />
      </CButton>
      <CButton
        :disabled="!vendors.selectedVendor"
        v-c-tooltip="{ content: 'Tambah Invoice', placement: 'top' }"
        @click="
          () => {
            modal.invoice = true;
            modal.pv = false;
          }
        "
        color="secondary"
      >
        <CIcon icon="cil-inbox" width="24" height="24" />
      </CButton>
      <CButton @click="validPV" v-c-tooltip="{ content: 'Simpan', placement: 'top' }" color="success">
        <CIcon icon="cil-send" width="24" height="24" />
      </CButton>
    </CModalFooter>
  </CModal>
  <CModal
    fullscreen
    alignment="center"
    backdrop="static"
    :visible="modal.invoice"
    @close="
      () => {
        modal.invoice = false;
        modal.pv = true;
      }
    "
  >
    <CModalHeader>
      <CModalTitle>Form Input Invoice</CModalTitle>
    </CModalHeader>
    <CModalBody>
      <CRow>
        <CCol md="4">
          <label class="mb-2">Invoice</label>
          <v-select multiple v-model="invoice.selectedInvoice" :options="invoice.data" :filterable="true" label="inv_number" @search="searchInvoice" :loading="invoice.isLoading"></v-select>
        </CCol>
        <CCol md="2" class="mb-3">
          <CFormInput type="number" disabled v-model="payment.dpp" label="Nilai DPP" placeholder="Nilai DPP" />
        </CCol>
        <CCol md="2" class="mb-3">
          <CFormInput type="number" disabled v-model="payment.ppn" label="Nilai PPN" placeholder="Nilai PPN" />
        </CCol>
        <CCol md="2" class="mb-3">
          <CFormInput type="number" disabled v-model="payment.pph" label="Nilai PPH" placeholder="Nilai PPH" />
        </CCol>
        <CCol md="2" class="mb-3">
          <CFormInput type="number" disabled v-model="payment.total" label="Nilai Total" placeholder="Nilai Total" />
        </CCol>
      </CRow>
      <CRow>
        <table class="table table-hover table-bordered table-striped table-sm align-middle">
          <thead class="text-center">
            <tr class="table-secondary">
              <th>#</th>
              <th>Inv. Number</th>
              <th>DPP</th>
              <th>PPN</th>
              <th>PPH</th>
              <th>DPP</th>
              <th>PPN</th>
              <th>PPH</th>
              <th>Sub. Total</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody v-if="invoice.selectedInvoice.length > 0">
            <tr v-for="(inv, idx) in invoice.selectedInvoice">
              <td>{{ idx + 1 }}</td>
              <td>{{ inv.inv_number }}</td>
              <td>{{ inv.dpp_amount }}</td>
              <td>{{ inv.ppn_amount }}</td>
              <td>{{ inv.pph_amount }}</td>
              <td>
                <CFormInput type="number" v-model="inv.dpp" floatingLabel="Nilai DPP" placeholder="Nilai DPP" />
              </td>
              <td>
                <CFormInput type="number" disabled v-model="inv.ppn" floatingLabel="Nilai PPN" placeholder="Nilai PPN" />
              </td>
              <td>
                <CFormInput type="number" disabled v-model="inv.pph" floatingLabel="Nilai PPH" placeholder="Nilai PPH" />
              </td>
              <td>
                <CFormInput type="number" disabled v-model="inv.total" floatingLabel="Sub Total" placeholder="Sub Total" />
              </td>
              <td class="text-center">
                <CButton v-c-tooltip="{ content: 'Cancel', placement: 'top' }" color="danger" @click="removeSelectedInvoice(index)">
                  <CIcon icon="cil-x-circle" width="20" height="20" />
                </CButton>
              </td>
            </tr>
          </tbody>
          <tbody v-else>
            <tr>
              <td class="text-center" colspan="10">Tidak ada data yang dipilih.</td>
            </tr>
          </tbody>
        </table>
      </CRow>
    </CModalBody>
  </CModal>
  <CModal
    size="lg"
    alignment="center"
    backdrop="static"
    :visible="modal.approval"
    @close="
      () => {
        modal.approval = false;
        modal.pv = true;
      }
    "
  >
    <CModalHeader>
      <CModalTitle>Form Tambah Approval</CModalTitle>
    </CModalHeader>
    <CModalBody>
      <CRow>
        <CCol md="12" class="mb-3">
          <label class="mb-2">Approval</label>
          <v-select v-model="payment.approval.selectedApproval" :options="payment.approval.data" :filterable="true" label="apvh_name"></v-select>
        </CCol>
      </CRow>
      <CRow v-for="(item, index) in payment.approval.selectedApproval.approval_users">
        <CCol md="6" class="mb-3">
          <CFormInput disabled type="text" :value="item.users.name" :label="`Tingkat ${item.apvu_level}`" :placeholder="`Tingkat ${item.apvu_level}`" />
        </CCol>
        <CCol md="6" class="mb-3">
          <CFormInput disabled type="text" :value="item.apvu_description" label="Deskripsi" placeholder="Deskripsi" />
        </CCol>
      </CRow>
    </CModalBody>
  </CModal>
  <CModal
    :visible="modal.confirmFirst"
    alignment="center"
    backdrop="static"
    @close="
      () => {
        (modal.confirmFirst = false), (modal.pv = true);
      }
    "
  >
    <CModalHeader>
      <CModalTitle>Konfirmasi</CModalTitle>
    </CModalHeader>
    <CModalBody>Simpan payment voucher?</CModalBody>
    <CModalFooter>
      <CButton
        v-c-tooltip="{ content: 'Complete', placement: 'top' }"
        color="success"
        @click="
          () => {
            (modal.confirmFirst = false), (modal.confirmSecond = true), (payment.status = 2), (modal.secondMessage = 'Apakah anda yakin akan menyelesaikan payment voucher?');
          }
        "
      >
        <CIcon icon="cil-arrow-thick-to-top" width="24" height="24" />
      </CButton>
      <CButton v-c-tooltip="{ content: 'Draft', placement: 'top' }" @click="saveAsDraft" color="secondary">
        <CIcon icon="cil-paperclip" width="24" height="24" />
      </CButton>
      <CButton
        v-c-tooltip="{ content: 'Cancel', placement: 'top' }"
        color="danger"
        @click="
          () => {
            (modal.confirmFirst = false), (modal.confirmSecond = true), (payment.status = 3), (modal.secondMessage = 'Apakah anda yakin akan membatalkan payment voucher?');
          }
        "
      >
        <CIcon icon="cil-x-circle" width="24" height="24" />
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
      <CButton v-c-tooltip="{ content: 'Confirm', placement: 'top' }" @click="savePV()" color="success">
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
      alert: {
        visibleOnModal: false,
        visibleOnMain: false,
        title: null,
        message: null,
        color: "danger",
      },
      mPayment: [],
      payment: {
        status: 1,
        dueDate: null,
        accountDpp: null,
        accountPpn: null,
        accountPph: null,
        accountDppDesc: null,
        accountPpnDesc: null,
        accountPphDesc: null,
        dpp: 0,
        ppn: 0,
        pph: 0,
        total: 0,
        note: null,
        approval: {
          selectedApproval: [],
          data: [],
        },
      },
      keyword: "",
      filter: {
        perPage: "50",
        status: "7",
        dateStart: null,
        dateEnd: null,
      },
      vendors: {
        data: [],
        selectedVendor: null,
        isLoading: false,
      },
      bank: {
        data: [],
        selectedBank: null,
        isLoading: false,
      },
      invoice: {
        data: [],
        selectedInvoice: [],
        isLoading: false,
      },
    };
  },
  watch: {
    "invoice.selectedInvoice": {
      handler(newVal) {
        let ppnAccountSet = false;
        let pphAccountSet = false;
        newVal.forEach((inv) => {
          if (inv.ppn_type == 1 && !ppnAccountSet) {
            this.payment.accountPpn = inv.ppn_account;
            this.payment.accountPpnDesc = `${inv.ppn_account} | ${inv.ppn_account_desc}`;
            ppnAccountSet = true;
          }
          if (inv.pph_type == 1 && !pphAccountSet) {
            this.payment.accountPph = inv.pph_account;
            this.payment.accountPphDesc = `${inv.pph_account} | ${inv.pph_account_desc}`;
            pphAccountSet = true;
          }
          if (inv.ppn_type == 1) {
            inv.ppn = Math.round(parseInt(inv.dpp) * 0.11);
          }
          if (inv.pph_type == 1) {
            inv.pph = Math.round(parseInt(inv.dpp) * 0.02);
          }

          inv.total = Math.round(parseInt(inv.dpp) + parseInt(inv.ppn) + parseInt(inv.pph));
        });
        this.calculatePaymentTotals();
      },
      deep: true,
    },
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
    async openModalNewPV() {
      try {
        this.$root.loader = true;
        const responseApprov = await ApiService.getApproval();
        this.payment.approval.data = responseApprov.data.map((apvh) => ({
          apvh_code: apvh.apvh_code,
          apvh_name: apvh.apvh_name,
          apvh_description: apvh.apvh_description,
          approval_users: apvh.approval_users,
        }));
        this.modal.pv = true;
        this.modal.action = 1;
      } catch (error) {
        this.getAlert("danger", "Error", "Terjadi error saat mengambil data.", "main");
      } finally {
        this.$root.loader = false;
      }
    },
    formatStatus(status) {
      const statusMap = {
        1: "Draft",
        2: "Complete",
        3: "Canceled",
        4: "Approved",
        5: "Rejected",
        6: "Closed",
      };
      return statusMap[status] || "Unknown";
    },
    async getDocx(id) {
      try {
        this.$root.loader = true;
        const response = await ApiService.getPVDocx(id);
        const base64Data = response.data;

        const byteArray = Uint8Array.from(atob(base64Data), (c) => c.charCodeAt(0));
        const blob = new Blob([byteArray], { type: "application/vnd.openxmlformats-officedocument.wordprocessingml.document" });

        const blobUrl = URL.createObjectURL(blob);

        const link = document.createElement("a");
        link.href = blobUrl;
        link.download = `${Math.random().toString(36).substring(2, 15)}.docx`;
        link.rel = "noopener";

        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);

        URL.revokeObjectURL(blobUrl);
      } catch (error) {
        console.error("Error fetching data:", error);
        this.getAlert("danger", "Error", "Terjadi kesalahan saat mengambil data.", "main");
      } finally {
        this.$root.loader = false;
      }
    },
    async savePV() {
      try {
        this.$root.loader = true;
        const data = {
          vend_code: this.vendors.selectedVendor.vend_code,
          dpp_account: this.payment.accountDpp,
          ppn_account: this.payment.accountPpn,
          pph_account: this.payment.accountPph,
          approval: this.payment.approval.selectedApproval.apvh_code,
          pv_status: this.payment.status,
          pv_due_date: this.payment.dueDate,
          bank_id: this.bank.selectedBank.id,
          ppn_amount: this.payment.ppn,
          pph_amount: this.payment.pph,
          dpp_amount: this.payment.dpp,
          pv_amount: this.payment.total,
          pv_note: this.payment.note,
          inv_data: this.invoice.selectedInvoice,
          user_id: this.user.id,
        };
        if (this.modal.action == 1) {
          const response = await ApiService.savePV(data, this.modal.pvId);
        } else if (this.modal.action == 2) {
          const response = await ApiService.editPV(data, this.modal.pvId);
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
    async searchVendorsGeneral(keyword) {
      try {
        this.vendors.data = [];
        if (this.abortController) {
          this.abortController.abort();
        }

        this.abortController = new AbortController();
        const signal = this.abortController.signal;
        this.vendors.isLoading = true;
        const response = await ApiService.searchVendorsGeneral(keyword, signal);
        this.vendors.data = response.data.map((vendor) => ({
          vend_code: vendor.vend_code,
          id_vendname: vendor.id_vendname,
          vend_account: vendor.account,
          vend_account_desc: vendor.vendor_accounts?.accdesc,
        }));
      } catch (error) {
        if (error.message !== "canceled") {
          console.error("Error fetching data:", error);
        }
      } finally {
        this.vendors.isLoading = false;
      }
    },
    setData() {
      this.payment.accountDppDesc = `${this.vendors.selectedVendor.vend_account} | ${this.vendors.selectedVendor.vend_account_desc}`;
      this.payment.accountDpp = this.vendors.selectedVendor.vend_account;
    },
    calculatePaymentTotals() {
      this.payment.dpp = this.invoice.selectedInvoice.reduce((sum, inv) => sum + parseInt(inv.dpp), 0);
      this.payment.ppn = this.invoice.selectedInvoice.reduce((sum, inv) => sum + parseInt(inv.ppn), 0);
      this.payment.pph = this.invoice.selectedInvoice.reduce((sum, inv) => sum + parseInt(inv.pph), 0);
      this.payment.total = parseInt(this.payment.dpp) + parseInt(this.payment.dpp) + parseInt(this.payment.dpp);
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
    async getEdit(id) {
      try {
        this.$root.loader = true;

        const [response, responseApprov] = await Promise.all([ApiService.getPVDetail(id), ApiService.getApproval()]);

        let { pvDetail } = this;
        pvDetail = response.data;

        const { approval_data_headers, vendor_accounts, tax_accounts, pph_accounts, bank_accounts, vendors, payment_voucher_invoices } = response.data;

        const formattedDate = this.formatDateOther(pvDetail.pv_due_date);

        const approvalData = responseApprov.data.map((apvh) => ({
          apvh_code: apvh.apvh_code,
          apvh_name: apvh.apvh_name,
          apvh_description: apvh.apvh_description,
          approval_users: apvh.approval_users,
        }));

        this.payment = {
          status: pvDetail.pv_status,
          dueDate: formattedDate,
          accountDpp: pvDetail.dpp_account,
          accountPpn: pvDetail.ppn_account,
          accountPph: pvDetail.pph_account,
          accountDppDesc: `${pvDetail.dpp_account} | ${vendor_accounts.accdesc}`,
          accountPpnDesc: `${pvDetail.ppn_account} | ${tax_accounts.dscription}`,
          accountPphDesc: `${pvDetail.pph_account} | ${pph_accounts.accdesc}`,
          dpp: pvDetail.dpp_amount,
          ppn: pvDetail.ppn_amount,
          pph: pvDetail.pph_amount,
          total: pvDetail.pv_amount,
          note: pvDetail.pv_note,
          approval: {
            data: approvalData,
            selectedApproval: {
              apvh_code: approval_data_headers.approval_headers ? approval_data_headers.approval_headers.apvh_code : "",
              apvh_name: approval_data_headers.approval_headers ? approval_data_headers.approval_headers.apvh_name : "",
              apvh_description: approval_data_headers.approval_headers ? approval_data_headers.approval_headers.apvh_description : "",
              approval_users: approval_data_headers.approvals ? approval_data_headers.approvals : "",
            },
          },
        };

        this.bank.selectedBank = {
          bank_n_code: `${bank_accounts.bankcode} | ${bank_accounts.bankname}`,
          id: bank_accounts.id,
          bankname: bank_accounts.bankname,
          bankcode: bank_accounts.bankcode,
        };

        this.vendors.selectedVendor = {
          id_vendname: `${vendors.vend_code} | ${vendors.vend_name}`,
          id: vendors.vend_code,
        };

        this.invoice.selectedInvoice = payment_voucher_invoices.map((inv) => ({
          inv_number: inv.inv_number,
          ppn_type: inv.invoices.ppn_type,
          ppn_account: inv.ppn_account,
          ppn_account_desc: inv.invoices.tax_accounts.dscription,
          pph_type: inv.invoices.pph_type,
          pph_account: inv.invoices.pph_account,
          pph_account_desc: inv.invoices.pph_accounts?.accdesc,
          dpp_amount: parseInt(inv.invoices.dpp_amount),
          ppn_amount: parseInt(inv.invoices.ppn_amount),
          pph_amount: parseInt(inv.invoices.pph_amount),
          inv_amount: parseInt(inv.invoices.inv_amount),
          dpp: parseInt(inv.dpp_amount),
          ppn: parseInt(inv.ppn_amount),
          pph: parseInt(inv.pph_amount),
          inv: parseInt(inv.inv_amount),
        }));

        this.invoice.selectedInvoice = payment_voucher_invoices;

        this.modal.action = 2;
        this.modal.pvId = id;
        this.modal.pv = true;
      } catch (error) {
        console.error("Error fetching data:", error);
        this.getAlert("danger", "Error", "Terjadi kesalahan saat mengambil data.", "main");
      } finally {
        this.$root.loader = false;
      }
    },
    async searchInvoice(keyword) {
      try {
        this.invoice.data = [];
        if (this.abortController) {
          this.abortController.abort();
        }

        this.abortController = new AbortController();
        const signal = this.abortController.signal;
        this.invoice.isLoading = true;
        const response = await ApiService.searchInvoice(keyword, this.vendors.selectedVendor.vend_code, signal);
        this.invoice.data = response.data.map((inv) => ({
          inv_id: inv.id,
          inv_number: inv.inv_number,
          ppn_type: inv.ppn_type,
          ppn_account: inv.ppn_account,
          ppn_account_desc: inv.tax_accounts.dscription,
          pph_type: inv.pph_type,
          pph_account: inv.pph_account,
          pph_account_desc: inv.pph_accounts?.accdesc,
          dpp_amount: parseInt(inv.dpp_amount),
          ppn_amount: parseInt(inv.ppn_amount),
          pph_amount: parseInt(inv.pph_amount),
          inv_amount: parseInt(inv.inv_amount),
          dpp: parseInt(inv.dpp_amount),
          ppn: parseInt(inv.ppn_amount),
          pph: parseInt(inv.pph_amount),
          inv: parseInt(inv.inv_amount),
        }));
      } catch (error) {
        if (error.message !== "canceled") {
          console.error("Error fetching data:", error);
        }
      } finally {
        this.invoice.isLoading = false;
      }
    },
    async searchBank(keyword) {
      try {
        this.bank.data = [];
        if (this.abortController) {
          this.abortController.abort();
        }

        this.abortController = new AbortController();
        const signal = this.abortController.signal;
        this.bank.isLoading = true;
        const response = await ApiService.searchBank(keyword, signal);
        this.bank.data = response.data.map((bk) => ({
          bank_n_code: bk.bank_n_code,
          id: bk.id,
          bankname: bk.bankname,
          bankcode: bk.bankcode,
        }));
      } catch (error) {
        if (error.message !== "canceled") {
          console.error("Error fetching data:", error);
        }
      } finally {
        this.bank.isLoading = false;
      }
    },
    validPV() {
      this.alert.visibleOnModal = false;
      const isFieldEmpty = (field) => field === undefined || field === null || field === "";

      if (
        isFieldEmpty(this.vendors.selectedVendor.vend_code) ||
        isFieldEmpty(this.payment.accountDpp) ||
        isFieldEmpty(this.payment.accountPpn) ||
        isFieldEmpty(this.payment.accountPph) ||
        isFieldEmpty(this.payment.approval.selectedApproval.apvh_code) ||
        isFieldEmpty(this.payment.status) ||
        isFieldEmpty(this.payment.dueDate) ||
        isFieldEmpty(this.bank.selectedBank.id) ||
        isFieldEmpty(this.payment.ppn) ||
        isFieldEmpty(this.payment.pph) ||
        isFieldEmpty(this.payment.dpp) ||
        isFieldEmpty(this.payment.total) ||
        isFieldEmpty(this.invoice.selectedInvoice)
      ) {
        this.getAlert("danger", "Error", "Semua data harus diisi dengan lengkap.", "modal");
      } else {
        this.modal.confirmFirst = true;
        this.modal.pv = false;
      }
    },
    async getData(page = 1) {
      try {
        this.$root.loader = true;
        const response = await ApiService.getPV(this.filter, page);
        this.mPayment = response.data;
        this.getAlert("success", "Success!", `Success with ${this.mPayment.total} data.`, "main");
      } catch (error) {
        console.error("Error fetching data:", error);
        this.getAlert("danger", "Error", "Terjadi kesalahan saat mengambil data.", "main");
      } finally {
        this.$root.loader = false;
      }
    },
    saveAsDraft() {
      this.payment.status = 1;
      this.savePV();
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
