<template>
  <v-row justify="center">
    <v-dialog
      v-model="customerDialog"
      max-width="600px"
      @click:outside="clear_customer"
    >
      <v-card class="px-6 py-8">
        <v-card-title class="pa-0 pb-8">
          <span v-if="customer_id" class="modal-title">อัปเดตลูกค้า</span>
          <span v-else class="modal-title">สร้างลูกค้าใหม่</span> 
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container class="pa-0">
            <v-row>
              <v-col cols="12">
                <v-text-field
                  outlined
                  dense
                  color="primary"
                  label="ชื่อลูกค้า *"
                  background-color="white"
                  hide-details
                  v-model="customer_name"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-text-field
                  outlined
                  dense
                  color="primary"
                  label="เลขประจำตัวผู้เสียภาษี"
                  background-color="white"
                  hide-details
                  v-model="tax_id"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-text-field
                  outlined
                  dense
                  color="primary"
                  label="เบอร์โทร"
                  background-color="white"
                  hide-details
                  v-model="mobile_no"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-text-field
                  outlined
                  dense
                  color="primary"
                  label="อีเมล"
                  background-color="white"
                  hide-details
                  v-model="email_id"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-select
                  outlined
                  dense
                  hide-details
                  label="เพศ"
                  :items="genders"
                  v-model="gender"
                ></v-select>
              </v-col>
              <v-col cols="6">
                <v-text-field
                  outlined
                  dense
                  color="primary"
                  label="รหัสแนะนำ"
                  background-color="white"
                  hide-details
                  v-model="referral_code"
                ></v-text-field>
              </v-col>
              <v-col cols="6">
                <v-menu
                  ref="birthday_menu"
                  v-model="birthday_menu"
                  :close-on-content-click="false"
                  transition="scale-transition"
                  dense
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="birthday"
                      label="วันเกิด"
                      readonly
                      outlined
                      dense
                      clearable
                      hide-details
                      v-bind="attrs"
                      v-on="on"
                      color="primary"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    v-model="birthday"
                    color="primary"
                    no-title
                    outlined
                    dense
                    scrollable
                    :max="frappe.datetime.now_date()"
                    @input="birthday_menu = false"
                  >
                  </v-date-picker>
                </v-menu>
              </v-col>
              <v-col cols="6">
                <v-autocomplete
                  clearable
                  outlined
                  dense
                  auto-select-first
                  color="primary"
                  label="กลุ่มลูกค้า *"
                  v-model="group"
                  :items="groups"
                  background-color="white"
                  :no-data-text="__('Group not found')"
                  hide-details
                  required
                >
                </v-autocomplete>
              </v-col>
              <v-col cols="6">
                <v-autocomplete
                  clearable
                  outlined
                  dense
                  auto-select-first
                  color="primary"
                  label="ภูมิภาค *"
                  v-model="territory"
                  :items="territorys"
                  background-color="white"
                  :no-data-text="__('Territory not found')"
                  hide-details
                  required
                >
                </v-autocomplete>
              </v-col>
              <v-col cols="6" v-if="loyalty_program">
                <v-text-field
                  v-model="loyalty_program"
                  :label="frappe._('Loyalty Program')"
                  outlined
                  dense
                  readonly
                  hide-details
                ></v-text-field>
              </v-col>
              <v-col cols="6" v-if="loyalty_points">
                <v-text-field
                  v-model="loyalty_points"
                  :label="frappe._('Loyalty Points')"
                  outlined
                  dense
                  readonly
                  hide-details
                ></v-text-field>
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
export default {
  data: () => ({
    customerDialog: false,
    pos_profile: '',
    customer_id: '',
    customer_name: '',
    tax_id: '',
    mobile_no: '',
    email_id: '',
    referral_code: '',
    birthday: null,
    birthday_menu: false,
    group: '',
    groups: [],
    territory: '',
    territorys: [],
    genders: [],
    customer_type: 'Individual',
    gender: '',
    loyalty_points: null,
    loyalty_program: null,
  }),
  watch: {},
  methods: {
    close_dialog() {
      this.customerDialog = false;
      this.clear_customer();
    },
    clear_customer() {
      this.customer_name = '';
      this.tax_id = '';
      this.mobile_no = '';
      this.email_id = '';
      this.referral_code = '';
      this.birthday = '';
      this.group = frappe.defaults.get_user_default('Customer Group');
      this.territory = frappe.defaults.get_user_default('Territory');
      this.customer_id = '';
      this.customer_type = 'Individual';
      this.gender = '';
      this.loyalty_points = null;
      this.loyalty_program = null;
    },
    getCustomerGroups() {
      if (this.groups.length > 0) return;
      const vm = this;
      frappe.db
        .get_list('Customer Group', {
          fields: ['name'],
          filters: { is_group: 0 },
          limit: 1000,
          order_by: 'name',
        })
        .then((data) => {
          if (data.length > 0) {
            data.forEach((el) => {
              vm.groups.push(el.name);
            });
          }
        });
    },
    getCustomerTerritorys() {
      if (this.territorys.length > 0) return;
      const vm = this;
      frappe.db
        .get_list('Territory', {
          fields: ['name'],
          filters: { is_group: 0 },
          limit: 5000,
          order_by: 'name',
        })
        .then((data) => {
          if (data.length > 0) {
            data.forEach((el) => {
              vm.territorys.push(el.name);
            });
          }
        });
    },
    getGenders() {
      const vm = this;
      frappe.db
        .get_list('Gender', {
          fields: ['name'],
          page_length: 10,
        })
        .then((data) => {
          if (data.length > 0) {
            data.forEach((el) => {
              vm.genders.push(el.name);
            });
          }
        });
    },
    submit_dialog() {
      // validate if all required fields are filled
      if (!this.customer_name) {
        evntBus.$emit('show_mesage', {
          text: "จำเป็นต้องระบุชื่อลูกค้า",
          color: 'error',
        });
        return;
      }
      if (!this.group) {
        evntBus.$emit('show_mesage', {
          text: "จำเป็นต้องระบุกลุ่มลูกค้า",
          color: 'error',
        });
        return;
      }
      if (!this.territory) {
        evntBus.$emit('show_mesage', {
          text: "จำเป็นต้องระบุภูมิภาค",
          color: 'error',
        });
        return;
      }
      if (this.customer_name) {
        const vm = this;
        const args = {
          customer_id: this.customer_id,
          customer_name: this.customer_name,
          company: this.pos_profile.company,
          tax_id: this.tax_id,
          mobile_no: this.mobile_no,
          email_id: this.email_id,
          referral_code: this.referral_code,
          birthday: this.birthday,
          customer_group: this.group,
          territory: this.territory,
          customer_type: this.customer_type,
          gender: this.gender,
          method: this.customer_id ? 'update' : 'create',
          pos_profile_doc: this.pos_profile,
        };
        frappe.call({
          method: 'poszaviago.poszaviago.api.posapp.create_customer',
          args: args,
          callback: (r) => {
            if (!r.exc && r.message.name) {
              let text = "เพิ่มลูกค้าใหม่สำเร็จ";
              if (vm.customer_id) {
                text = __('แก้ไขข้อมูลลูกค้าสำเร็จ');
              }
              evntBus.$emit('show_mesage', {
                text: text,
                color: 'success',
              });
              args.name = r.message.name;
              frappe.utils.play_sound('submit');
              evntBus.$emit('add_customer_to_list', args);
              evntBus.$emit('set_customer', r.message.name);
              evntBus.$emit('fetch_customer_details');
              this.close_dialog();
            } else {
              frappe.utils.play_sound('error');
              evntBus.$emit('show_mesage', {
                text: __('เพิ่มลูกค้าใหม่ไม่สำเร็จ'),
                color: 'error',
              });
            }
          },
        });
        this.customerDialog = false;
      }
    },
  },
  created: function () {
    evntBus.$on('open_update_customer', (data) => {
      this.customerDialog = true;
      if (data) {
        this.customer_name = data.customer_name;
        this.customer_id = data.name;
        this.tax_id = data.tax_id;
        this.mobile_no = data.mobile_no;
        this.email_id = data.email_id;
        this.referral_code = data.referral_code;
        this.birthday = data.birthday;
        this.group = data.customer_group;
        this.territory = data.territory;
        this.loyalty_points = data.loyalty_points;
        this.loyalty_program = data.loyalty_program;
        this.gender = data.gender;
      }
    });
    evntBus.$on('register_pos_profile', (data) => {
      this.pos_profile = data.pos_profile;
    });
    evntBus.$on('payments_register_pos_profile', (data) => {
      this.pos_profile = data.pos_profile;
    });
    this.getCustomerGroups();
    this.getCustomerTerritorys();
    this.getGenders();
    // set default values for customer group and territory from user defaults
    this.group = frappe.defaults.get_user_default('Customer Group');
    this.territory = frappe.defaults.get_user_default('Territory');
  },
};
</script>

<style scoped>

.modal-title {
  font-weight:600 !important;
  font-size:24px;
}

</style>