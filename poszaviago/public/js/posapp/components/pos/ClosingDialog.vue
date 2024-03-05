<template>
  <v-row justify="center">
    <v-dialog v-model="closingDialog" max-width="900px">
      <v-card class="px-6 py-8" :style="{ borderRadius:'10px' }">
        <v-card-title class="pa-0 pb-8">
          <span class="modal-title">ปิดการขาย</span>
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container>
            <v-row>
              <v-col cols="12" class="pa-0">
                <template>
                  <v-data-table
                    :headers="headers"
                    :items="dialog_data.payment_reconciliation"
                    item-key="mode_of_payment"
                    class="elevation-0 px-0"
                    :items-per-page="itemsPerPage"
                    hide-default-footer
                  >
                    <template v-slot:item.closing_amount="props">
                      <v-edit-dialog
                        :return-value.sync="props.item.closing_amount"
                      >
                        <v-icon>mdi-pencil-outline</v-icon>
                        {{ currencySymbol(pos_profile.currency) }}
                        {{ formtCurrency(props.item.closing_amount) }}
                        <template v-slot:input>
                          <v-text-field
                            v-model="props.item.closing_amount"
                            :rules="[max25chars]"
                            :label="frappe._('Edit')"
                            single-line
                            counter
                            type="number"
                          ></v-text-field>
                        </template>
                      </v-edit-dialog>
                    </template>
                    <template v-slot:item.difference="{ item }">
                      {{ currencySymbol(pos_profile.currency) }}
                      {{
                        (item.difference = formtCurrency(
                          item.expected_amount - item.closing_amount
                        ))
                      }}</template
                    >
                    <template v-slot:item.opening_amount="{ item }">
                      {{ currencySymbol(pos_profile.currency) }}
                      {{ formtCurrency(item.opening_amount) }}</template
                    >
                    <template v-slot:item.expected_amount="{ item }">
                      {{ currencySymbol(pos_profile.currency) }}
                      {{ formtCurrency(item.expected_amount) }}</template
                    >
                  </v-data-table>
                </template>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions class="pa-0 pt-8">
          <v-spacer></v-spacer>
          <v-btn class="elevation-0 px-5" color="gray01" style="color:black;height:50px;border-radius:10px" dark @click="close_dialog">ยกเลิก</v-btn>
          <v-btn class="elevation-0 px-5" color="black" style="height:50px;border-radius:10px" dark @click="submit_dialog">ยืนยันการทำรายการ</v-btn>
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
    closingDialog: false,
    itemsPerPage: 20,
    dialog_data: {},
    pos_profile: '',
    headers: [
      {
        text: "รูปแบบการชำระเงิน",
        value: 'mode_of_payment',
        align: 'start',
        sortable: true,
      },
      {
        text: "จำนวนเงินในลิ้นชัก",
        align: 'end',
        sortable: true,
        value: 'opening_amount',
      },
      {
        text: "คงเหลือ(ตามจริง)",
        value: 'closing_amount',
        align: 'end',
        sortable: true,
      },
    ],
    max25chars: (v) => v.length <= 20 || 'Input too long!', // TODO : should validate as number
    pagination: {},
  }),
  watch: {},

  methods: {
    close_dialog() {
      this.closingDialog = false;
    },
    submit_dialog() {
      evntBus.$emit('submit_closing_pos', this.dialog_data);
      this.closingDialog = false;
    },
  },

  created: function () {
    evntBus.$on('open_ClosingDialog', (data) => {
      this.closingDialog = true;
      this.dialog_data = data;
    });
    evntBus.$on('register_pos_profile', (data) => {
      this.pos_profile = data.pos_profile;
      if (!this.pos_profile.hide_expected_amount) {
        this.headers.push({
          text: "คงเหลือ(ที่คาดหวัง)",
          value: 'expected_amount',
          align: 'end',
          sortable: false,
        });
        this.headers.push({
          text: "ส่วนต่าง",
          value: 'difference',
          align: 'end',
          sortable: false,
        });
      }
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