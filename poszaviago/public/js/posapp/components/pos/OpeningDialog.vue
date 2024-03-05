<template>
  <v-row justify="center">
    <v-dialog v-model="isOpen" persistent max-width="600px">
      <!-- <template v-slot:activator="{ on, attrs }">
        <v-btn color="primary" dark v-bind="attrs" v-on="on">Open Dialog</v-btn>
      </template>-->
      <v-card class="px-6 py-8" :style="{ borderRadius:'10px' }">
        <v-card-title class="pa-0 pb-8">
          <span class="modal-title">เปิดการขาย</span>
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container>
            <v-row>
              <v-col cols="12" class="px-0">
                <v-autocomplete
                  :items="companies"
                  label="ชื่อธุรกิจ"
                  v-model="company"
                  required
                  outlined
                  hide-details
                ></v-autocomplete>
              </v-col>
              <v-col cols="12" class="px-0">
                <v-autocomplete
                  :items="pos_profiles"
                  label="สาขา"
                  v-model="pos_profile"
                  required
                  outlined
                  hide-details
                ></v-autocomplete>
              </v-col>
              <v-col cols="12" class="px-0">
                <template>
                  <v-data-table
                    :headers="payments_methods_headers"
                    :items="payments_methods"
                    item-key="mode_of_payment"
                    class="elevation-0"
                    :items-per-page="itemsPerPage"
                    hide-default-footer
                  >
                    <template v-slot:item.amount="props">
                      <v-edit-dialog :return-value.sync="props.item.amount">
                        <v-icon>mdi-pencil-outline</v-icon>
                        {{ currencySymbol(props.item.currency) }}
                        {{ formtCurrency(props.item.amount) }}
                        <template v-slot:input>
                          <v-text-field
                            v-model="props.item.amount"
                            :rules="[max25chars]"
                            :label="frappe._('Edit')"
                            single-line
                            counter
                            type="number"
                          ></v-text-field>
                        </template>
                      </v-edit-dialog>
                    </template>
                  </v-data-table>
                </template>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions class="pa-0 pt-8">
          <v-spacer></v-spacer>
          <v-btn class="elevation-0 px-5" color="gray01" style="color:black;height:50px;border-radius:10px" dark @click="go_desk">ยกเลิก</v-btn>
          <v-btn
            class="elevation-0 px-5" color="black" style="height:50px;border-radius:10px"
            :disabled="is_loading"
            dark
            @click="submit_dialog"
            >เปิดการขาย</v-btn
          >
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
  props: ['dialog'],
  data() {
    return {
      isOpen: this.dialog ? this.dialog : false,
      dialog_data: {},
      is_loading: false,
      companies: [],
      company: '',
      pos_profiles_data: [],
      pos_profiles: [],
      pos_profile: '',
      payments_method_data: [],
      payments_methods: [],
      payments_methods_headers: [
        {
          text: "รูปแบบการชำระเงิน",
          align: 'start',
          sortable: false,
          value: 'mode_of_payment',
        },
        {
          text: "จำนวนเงินในลิ้นชัก",
          value: 'amount',
          align: 'center',
          sortable: false,
        },
      ],
      itemsPerPage: 100,
      max25chars: (v) => v.length <= 12 || 'Input too long!', // TODO : should validate as number
      pagination: {},
      snack: false, // TODO : need to remove
      snackColor: '', // TODO : need to remove
      snackText: '', // TODO : need to remove
    };
  },
  watch: {
    company(val) {
      this.pos_profiles = [];
      this.pos_profiles_data.forEach((element) => {
        if (element.company === val) {
          this.pos_profiles.push(element.name);
        }
        if (this.pos_profiles.length) {
          this.pos_profile = this.pos_profiles[0];
        } else {
          this.pos_profile = '';
        }
      });
    },
    pos_profile(val) {
      this.payments_methods = [];
      this.payments_method_data.forEach((element) => {
        if (element.parent === val) {
          this.payments_methods.push({
            mode_of_payment: element.mode_of_payment,
            amount: 0,
            currency: element.currency,
          });
        }
      });
    },
  },
  methods: {
    close_opening_dialog() {
      evntBus.$emit('close_opening_dialog');
    },
    get_opening_dialog_data() {
      const vm = this;
      frappe.call({
        method: 'poszaviago.poszaviago.api.posapp.get_opening_dialog_data',
        args: {},
        callback: function (r) {
          if (r.message) {
            r.message.companies.forEach((element) => {
              vm.companies.push(element.name);
            });
            vm.company = vm.companies[0];
            vm.pos_profiles_data = r.message.pos_profiles_data;
            vm.payments_method_data = r.message.payments_method;
          }
        },
      });
    },
    submit_dialog() {
      if (!this.payments_methods.length || !this.company || !this.pos_profile) {
        return;
      }
      this.is_loading = true;
      const vm = this;
      return frappe
        .call('poszaviago.poszaviago.api.posapp.create_opening_voucher', {
          pos_profile: this.pos_profile,
          company: this.company,
          balance_details: this.payments_methods,
        })
        .then((r) => {
          if (r.message) {
            evntBus.$emit('register_pos_data', r.message);
            evntBus.$emit('set_company', r.message.company);
            vm.close_opening_dialog();
            is_loading = false;
          }
        });
    },
    go_desk() {
      frappe.set_route('/');
      location.reload();
    },
  },
  created: function () {
    this.$nextTick(function () {
      this.get_opening_dialog_data();
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