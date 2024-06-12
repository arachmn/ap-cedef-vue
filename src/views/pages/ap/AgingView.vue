<template>
  <Pages :title="title">
    <CCard class="mb-4">
      <CCardHeader>
        <strong>Aging</strong>
      </CCardHeader>
      <CCardBody>
        <CAlert
          class="d-flex align-items-center"
          :color="alert.color"
          :visible="alert.visible"
          dismissible
          @close="
            () => {
              alert.visible = false;
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

        <CRow class="mb-3">
          <CCol>
            <CInputGroup>
              <CInputGroupText id="mySecondary">Type</CInputGroupText>
              <CFormSelect v-model="filter.type">
                <option value="summary">Summary</option>
                <option value="detail">Detail</option>
              </CFormSelect>
              <CButton v-c-tooltip="{ content: 'Filter data', placement: 'top' }" type="button" color="secondary" variant="outline" @click="getData()">
                <CIcon icon="cil-filter" width="24" height="24" />
              </CButton>
            </CInputGroup>
          </CCol>
        </CRow>

        <table v-if="filter.type == 'summary' && summary.total_weekly && summary.vendors" class="table table-hover table-bordered table-striped table-sm align-middle">
          <thead class="align-middle text-center">
            <tr class="table-secondary">
              <th>#</th>
              <th>Vendor</th>
              <th v-for="week in weeks" :key="week">{{ week }}</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(sm, idx) in summary.vendors">
              <td>{{ idx + 1 }}</td>
              <td>{{ `${sm.vend_code} | ${sm.vend_name}` }}</td>
              <td>{{ formatAmount(sm.weekly_invoice_data.week_1) }}</td>
              <td>{{ formatAmount(sm.weekly_invoice_data.week_2) }}</td>
              <td>{{ formatAmount(sm.weekly_invoice_data.week_3) }}</td>
              <td>{{ formatAmount(sm.weekly_invoice_data.week_4) }}</td>
              <td>{{ formatAmount(sm.weekly_invoice_data.week_5) }}</td>
              <td class="text-center">
                <CButtonGroup>
                  <CButton @click="getInvoices(sm.vend_code)" v-c-tooltip="{ content: 'Detail', placement: 'top' }" color="success">
                    <CIcon icon="cil-description" width="20" height="20" />
                  </CButton>
                  <CButton v-c-tooltip="{ content: 'Export', placement: 'top' }" color="primary">
                    <CIcon icon="cil-save" width="20" height="20" />
                  </CButton>
                </CButtonGroup>
              </td>
            </tr>
          </tbody>
          <tfoot>
            <tr>
              <td colspan="2" class="text-center"><strong>Total</strong></td>
              <td>
                <strong>{{ formatAmount(summary.total_weekly.total_week_1) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(summary.total_weekly.total_week_2) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(summary.total_weekly.total_week_3) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(summary.total_weekly.total_week_4) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(summary.total_weekly.total_week_5) }}</strong>
              </td>
            </tr>
          </tfoot>
        </table>
        <table v-if="filter.type == 'detail' && detail.vendors && detail.total_weekly" class="table table-hover table-striped table-bordered table-sm text-start">
          <tbody class="align-middle" v-for="(mnt, idx) in detail.vendors">
            <tr class="text-center table-danger">
              <th><strong>Vendor</strong></th>
              <th><strong>Doc. Num.</strong></th>
              <th><strong>Doc. Date</strong></th>
              <th><strong>Due. Date</strong></th>
              <th v-for="week in weeks" :key="week">
                <strong>{{ week }}</strong>
              </th>
            </tr>
            <tr v-for="inv in mnt.weekly.week_1.invoices">
              <td>{{ `${mnt.vendor.vend_code} | ${mnt.vendor.vend_name}` }}</td>
              <td>{{ inv.inv_number }}</td>
              <td>{{ formatDate(inv.inv_doc_date) }}</td>
              <td>{{ formatDate(inv.inv_due_date) }}</td>
              <td>{{ formatAmount(inv.inv_not_payed) }}</td>
              <td>0.00</td>
              <td>0.00</td>
              <td>0.00</td>
              <td>0.00</td>
            </tr>
            <tr v-for="inv in mnt.weekly.week_2.invoices">
              <td>{{ `${mnt.vendor.vend_code} | ${mnt.vendor.vend_name}` }}</td>
              <td>{{ inv.inv_number }}</td>
              <td>{{ formatDate(inv.inv_doc_date) }}</td>
              <td>{{ formatDate(inv.inv_due_date) }}</td>
              <td>0.00</td>
              <td>{{ formatAmount(inv.inv_not_payed) }}</td>
              <td>0.00</td>
              <td>0.00</td>
              <td>0.00</td>
            </tr>
            <tr v-for="inv in mnt.weekly.week_3.invoices">
              <td>{{ `${mnt.vendor.vend_code} | ${mnt.vendor.vend_name}` }}</td>
              <td>{{ inv.inv_number }}</td>
              <td>{{ formatDate(inv.inv_doc_date) }}</td>
              <td>{{ formatDate(inv.inv_due_date) }}</td>
              <td>0.00</td>
              <td>0.00</td>
              <td>{{ formatAmount(inv.inv_not_payed) }}</td>
              <td>0.00</td>
              <td>0.00</td>
            </tr>
            <tr v-for="inv in mnt.weekly.week_4.invoices">
              <td>{{ `${mnt.vendor.vend_code} | ${mnt.vendor.vend_name}` }}</td>
              <td>{{ inv.inv_number }}</td>
              <td>{{ formatDate(inv.inv_doc_date) }}</td>
              <td>{{ formatDate(inv.inv_due_date) }}</td>
              <td>0.00</td>
              <td>0.00</td>
              <td>0.00</td>
              <td>{{ formatAmount(inv.inv_not_payed) }}</td>
              <td>0.00</td>
            </tr>
            <tr v-for="inv in mnt.weekly.week_5.invoices">
              <td>{{ `${mnt.vendor.vend_code} | ${mnt.vendor.vend_name}` }}</td>
              <td>{{ inv.inv_number }}</td>
              <td>{{ formatDate(inv.inv_doc_date) }}</td>
              <td>{{ formatDate(inv.inv_due_date) }}</td>
              <td>0.00</td>
              <td>0.00</td>
              <td>0.00</td>
              <td>0.00</td>
              <td>{{ formatAmount(inv.inv_not_payed) }}</td>
            </tr>
            <tr>
              <td class="text-center" colspan="4"><strong>Sub. Total</strong></td>
              <td>
                <strong>{{ formatAmount(mnt.weekly.week_1.total) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(mnt.weekly.week_2.total) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(mnt.weekly.week_3.total) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(mnt.weekly.week_4.total) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(mnt.weekly.week_5.total) }}</strong>
              </td>
            </tr>
          </tbody>
          <tfoot class="align-middle">
            <tr>
              <td class="text-center" colspan="4"><strong>Total</strong></td>
              <td>
                <strong>{{ formatAmount(detail.total_weekly.total_week_1) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(detail.total_weekly.total_week_2) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(detail.total_weekly.total_week_3) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(detail.total_weekly.total_week_4) }}</strong>
              </td>
              <td>
                <strong>{{ formatAmount(detail.total_weekly.total_week_5) }}</strong>
              </td>
            </tr>
          </tfoot>
        </table>
      </CCardBody>
    </CCard>
  </Pages>
  <CModal
    fullscreen
    alignment="center"
    backdrop="static"
    :visible="modal.detail"
    @close="
      () => {
        modal.detail = false;
      }
    "
  >
    <CModalHeader>
      <CModalTitle>{{ invoice?.vendor.vend_code }} | {{ invoice?.vendor.vend_name }} </CModalTitle>
    </CModalHeader>
    <CModalBody>
      <table class="table table-hover table-striped table-bordered table-sm text-start">
        <tbody class="align-middle">
          <tr class="text-center table-danger">
            <th><strong>Doc. Num.</strong></th>
            <th><strong>Doc. Date</strong></th>
            <th><strong>Due. Date</strong></th>
            <th v-for="week in weeks" :key="week">
              <strong>{{ week }}</strong>
            </th>
          </tr>
          <tr v-for="inv in invoice?.weekly.week_1.invoices">
            <td>{{ inv.inv_number }}</td>
            <td>{{ formatDate(inv.inv_doc_date) }}</td>
            <td>{{ formatDate(inv.inv_due_date) }}</td>
            <td>{{ formatAmount(inv.inv_not_payed) }}</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
          </tr>
          <tr v-for="inv in invoice?.weekly.week_2.invoices">
            <td>{{ inv.inv_number }}</td>
            <td>{{ formatDate(inv.inv_doc_date) }}</td>
            <td>{{ formatDate(inv.inv_due_date) }}</td>
            <td>0.00</td>
            <td>{{ formatAmount(inv.inv_not_payed) }}</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
          </tr>
          <tr v-for="inv in invoice?.weekly.week_3.invoices">
            <td>{{ inv.inv_number }}</td>
            <td>{{ formatDate(inv.inv_doc_date) }}</td>
            <td>{{ formatDate(inv.inv_due_date) }}</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>{{ formatAmount(inv.inv_not_payed) }}</td>
            <td>0.00</td>
            <td>0.00</td>
          </tr>
          <tr v-for="inv in invoice?.weekly.week_4.invoices">
            <td>{{ inv.inv_number }}</td>
            <td>{{ formatDate(inv.inv_doc_date) }}</td>
            <td>{{ formatDate(inv.inv_due_date) }}</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>{{ formatAmount(inv.inv_not_payed) }}</td>
            <td>0.00</td>
          </tr>
          <tr v-for="inv in invoice?.weekly.week_5.invoices">
            <td>{{ inv.inv_number }}</td>
            <td>{{ formatDate(inv.inv_doc_date) }}</td>
            <td>{{ formatDate(inv.inv_due_date) }}</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>0.00</td>
            <td>{{ formatAmount(inv.inv_not_payed) }}</td>
          </tr>
          <tr>
            <td class="text-center" colspan="3"><strong>Sub. Total</strong></td>
            <td>
              <strong>{{ formatAmount(invoice?.weekly.week_1.total) }}</strong>
            </td>
            <td>
              <strong>{{ formatAmount(invoice?.weekly.week_2.total) }}</strong>
            </td>
            <td>
              <strong>{{ formatAmount(invoice?.weekly.week_3.total) }}</strong>
            </td>
            <td>
              <strong>{{ formatAmount(invoice?.weekly.week_4.total) }}</strong>
            </td>
            <td>
              <strong>{{ formatAmount(invoice?.weekly.week_5.total) }}</strong>
            </td>
          </tr>
        </tbody>
      </table>
    </CModalBody>
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

export default {
  components: {
    Pages,
  },
  mounted() {
    this.calculateWednesdays();
  },
  data() {
    return {
      title: "A/P GENERAL",
      user: JSON.parse(localStorage.getItem("user")),
      modal: {
        detail: false,
      },
      weeks: [],
      alert: {
        visible: false,
        title: null,
        message: null,
        color: "danger",
      },
      filter: {
        type: "summary",
      },
      invoice: [],
      summary: [],
      detail: [],
    };
  },
  methods: {
    calculateWednesdays() {
      let currentDate = new Date();
      let countWednesdays = 0;
      while (countWednesdays < 5) {
        currentDate.setDate(currentDate.getDate() + 1);
        if (currentDate.getDay() === 3) {
          let formattedDate = this.formatDateHeader(currentDate);
          this.weeks.push(formattedDate);
          countWednesdays++;
        }
      }
    },
    formatDateHeader(date) {
      let dd = String(date.getDate()).padStart(2, "0");
      let mm = String(date.getMonth() + 1).padStart(2, "0");
      let yyyy = date.getFullYear();
      return dd + "/" + mm + "/" + yyyy;
    },
    formatDate(data) {
      return format(new Date(data), "dd-MM-yyyy");
    },
    formatAmount(number) {
      return numeral(number).format("0,0.00");
    },
    async getInvoices(vendorNumber) {
      try {
        this.alert.visible = false;
        this.$root.loader = true;
        const response = await ApiService.getInvoices(vendorNumber);
        this.invoice = response.data;
        this.modal.detail = true;
        console.log(this.invoice);
      } catch (error) {
        this.getAlert("danger", "Error!", error.message);
        console.error("Error fetching data:", error);
      } finally {
        this.$root.loader = false;
      }
    },
    async getData() {
      try {
        this.alert.visible = false;
        this.$root.loader = true;
        const response = await ApiService.getAging(this.filter);
        if (this.filter.type == "summary") {
          this.summary = response.data;
        } else if (this.filter.type == "detail") {
          this.detail = response.data;
        }
      } catch (error) {
        this.getAlert("danger", "Error!", error.message);
        console.error("Error fetching data:", error);
      } finally {
        this.$root.loader = false;
      }
    },
    getAlert(color, title, message) {
      this.alert.title = title;
      this.alert.color = color;
      this.alert.message = message;
      this.alert.visible = true;
    },
  },
};
</script>
