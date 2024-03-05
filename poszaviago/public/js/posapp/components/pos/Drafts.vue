<template>
  <v-row justify="center">
    <v-dialog v-model="draftsDialog" max-width="900px">
      <!-- <template v-slot:activator="{ on, attrs }">
        <v-btn color="primary" dark v-bind="attrs" v-on="on">Open Dialog</v-btn>
      </template>-->
      <v-card class="px-6 py-8" style="border-radius:10px">
        <v-card-title class="pa-0 pb-8" :style="{ borderRadius:'10px' }">
          <span class="modal-title">เรียกบิลที่พักไว้</span>
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container class="pa-0">
            <v-row no-gutters>
              <v-col cols="12" class="pa-0">
                <template>
                  <v-data-table
                    :headers="headers"
                    :items="dialog_data"
                    item-key="name"
                    class="elevation-0"
                    :single-select="singleSelect"
                    show-select
                    v-model="selected"
                  >
                    <template v-slot:item.posting_time="{ item }">
                      {{ item.posting_time.split('.')[0] }}
                    </template>
                    <template v-slot:item.grand_total="{ item }">
                      {{ currencySymbol(item.currency) }}
                      {{ formtCurrency(item.grand_total) }}
                    </template>
                  </v-data-table>
                </template>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions class="pa-0 pt-8">
          <v-spacer></v-spacer>
          <v-btn class="elevation-0 px-5" color="gray01" style="color:black;height:50px;border-radius:10px" dark @click="close_dialog">ยกเลิก</v-btn>
          <v-btn class="elevation-0 px-5" color="black" style="height:50px;border-radius:10px" dark @click="submit_dialog">
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
  // props: ["draftsDialog"],
  mixins: [format],
  data: () => ({
    draftsDialog: false,
    singleSelect: true,
    selected: [],
    dialog_data: {},
    headers: [
      {
        text: "ลูกค้า",
        value: 'customer_name',
        align: 'start',
        sortable: true,
      },
      {
        text: "วันที่",
        align: 'start',
        sortable: true,
        value: 'posting_date',
      },
      {
        text: "เวลา",
        align: 'start',
        sortable: true,
        value: 'posting_time',
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
      this.draftsDialog = false;
    },

    submit_dialog() {
      if (this.selected.length > 0) {
        evntBus.$emit('load_invoice', this.selected[0]);
        this.draftsDialog = false;
      }
    },
  },
  created: function () {
    evntBus.$on('open_drafts', (data) => {
      this.draftsDialog = true;
      this.dialog_data = data;
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