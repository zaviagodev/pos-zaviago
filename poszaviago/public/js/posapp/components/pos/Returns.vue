<template>
  <v-row justify="center">
    <v-dialog v-model="invoicesDialog" max-width="800px" min-width="800px">
      <v-card class="px-6 py-8">
        <v-card-title class="pa-0 pb-8">
          <span class="modal-title">เลือกบิลที่ต้องการคืนสินค้า</span>
        </v-card-title>
        <v-container class="pa-0">
          <v-row class="mb-4 mr-0">
            <v-text-field
              color="primary"
              label="หมายเลขบิล"
              background-color="white"
              hide-details
              v-model="invoice_name"
              dense
              outlined
              clearable
              class="mx-4"
            ></v-text-field>
            <v-btn
              text
              class="ml-2 elevation-0"
              color="primary"
              :style="{ height:'40px',borderRadius:'10px',backgroundColor:'#EBF8FF' }"
              @click="search_invoices"
              >ค้นหา</v-btn
            >
          </v-row>
          <v-row>
            <v-col cols="12" class="pa-1" v-if="dialog_data">
              <template>
                <v-data-table
                  :headers="headers"
                  :items="dialog_data"
                  item-key="name"
                  class="elevation-0"
                  :single-select="singleSelect"
                  show-select
                  v-model="selected"
                  no-data-text="ยังไม่มีข้อมูล"
                  rows-per-page-text="จำนวนแถวต่อหน้า"
                  :footer-props="{
                    'items-per-page-text':'จำนวนแถวต่อหน้า',        
                    'page-text':'{0}-{1} จาก {2}'
                  }"
                >
                  <template v-slot:item.grand_total="{ item }">
                    {{ currencySymbol(item.currency) }}
                    {{ formtCurrency(item.grand_total) }}</template
                  >
                </v-data-table>
              </template>
            </v-col>
          </v-row>
        </v-container>
        <v-card-actions class="mt-4 pa-0">
          <v-spacer></v-spacer>
          <v-btn class="elevation-0 px-5" color="gray01" style="color:black;height:50px;border-radius:10px" dark @click="close_dialog">ยกเลิก</v-btn>
          <v-btn
            v-if="selected.length"
            class="elevation-0 px-5" color="black" style="height:50px;border-radius:10px"
            dark
            @click="submit_dialog"
          >
            <v-icon class='mr-2'>mdi-plus-circle-outline</v-icon>
            ยืนยัน
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import { evntBus } from '../../bus';
import format from '../../format';
export default {
  mixins: [format],
  data: () => ({
    invoicesDialog: false,
    singleSelect: true,
    selected: [],
    dialog_data: '',
    company: '',
    invoice_name: '',
    headers: [
      {
        text: "ลูกค้า",
        value: 'customer',
        align: 'start',
        sortable: true,
      },
      {
        text:"วันที่",
        align: 'start',
        sortable: true,
        value: 'posting_date',
      },
      {
        text: "หมายเลขบิล",
        value: 'name',
        align: 'start',
        sortable: true,
      },
      {
        text: "ยอดรวม",
        value: 'grand_total',
        align: 'end',
        sortable: false,
      },
    ],
  }),
  watch: {},
  methods: {
    close_dialog() {
      this.invoicesDialog = false;
    },
    search_invoices_by_enter(e) {
      if (e.keyCode === 13) {
        this.search_invoices();
      }
    },
    search_invoices() {
      const vm = this;
      frappe.call({
        method: 'poszaviago.poszaviago.api.posapp.search_invoices_for_return',
        args: {
          invoice_name: vm.invoice_name,
          company: vm.company,
        },
        async: false,
        callback: function (r) {
          if (r.message) {
            vm.dialog_data = r.message;
          }
        },
      });
    },
    submit_dialog() {
      if (this.selected.length > 0) {
        const return_doc = this.selected[0];
        const invoice_doc = {};
        const items = [];
        return_doc.items.forEach((item) => {
          const new_item = { ...item };
          new_item.qty = item.qty * -1;
          new_item.stock_qty = item.stock_qty * -1;
          new_item.amount = item.amount * -1;
          items.push(new_item);
        });
        invoice_doc.items = items;
        invoice_doc.is_return = 1;
        invoice_doc.return_against = return_doc.name;
        invoice_doc.customer = return_doc.customer;
        const data = { invoice_doc, return_doc };
        evntBus.$emit('load_return_invoice', data);
        this.invoicesDialog = false;
      }
    },
  },
  created: function () {
    evntBus.$on('open_returns', (data) => {
      this.invoicesDialog = true;
      this.company = data;
      this.invoice_name = '';
      this.dialog_data = '';
      this.selected = [];
    });
  },
};
</script>

<style scoped>

.modal-title {
  font-weight:600 !important;
  font-size:24px;
}

</style>