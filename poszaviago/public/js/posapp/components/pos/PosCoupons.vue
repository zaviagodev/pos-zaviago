<template>
  <div>
    <v-card
      class="selection mx-auto elevation-0"
      style="max-height: 80vh; height: 80vh;"
    >
      <v-card-title class="ml-n1 pt-5 pb-4 px-5 mr-2" :style="{ borderRight:'1px solid #F4F4F4',borderBottom:'1px solid #DFDFDF' }">
        <v-row no-gutters align="center" justify="center">
          <v-col cols="6">
            <span class="table-list-title">คูปอง</span>
          </v-col>
          <v-col cols="6">
            <div class="d-flex" :style="{ gap:'24px' }">
              <v-text-field
                dense
                outlined
                color="primary"
                label="คูปองโค้ด"
                background-color="white"
                hide-details
                v-model="new_coupon"
              ></v-text-field>
              <v-btn
                class='d-flex align-center text-white elevation-0' :style="{ height:'40px' }"
                color="black"
                dark
                @click="add_coupon(new_coupon)"
              >
                <v-icon class='mr-2'>mdi-plus-circle-outline</v-icon>
                เพิ่มคูปอง
              </v-btn>
            </div>
          </v-col>
        </v-row>
      </v-card-title>
      <div class="ma-0 ml-n1 px-5 pt-6 overflow-y-auto" :style="{ height: '80vh', maxHeight:'80vh' ,backgroundColor:'#DFDFDF', scrollbarWidth:'none', width:'calc(100% - 4px)',paddingBottom:'100px' }">
        <template @mouseover="style = 'cursor: pointer'">
          <v-data-table
            :headers="items_headers"
            :items="posa_coupons"
            :single-expand="singleExpand"
            :expanded.sync="expanded"
            item-key="coupon"
            class="elevation-0 table-lists"
            :items-per-page="itemsPerPage"
            hide-default-footer
            no-data-text="ยังไม่มีข้อมูล"
            :style="{ borderRadius:'10px' }"
            :footer-props="{
              'items-per-page-text':'จำนวนแถวต่อหน้า',        
              'page-text':'{0}-{1} จาก {2}'
            }"
          >
            <template v-slot:item.applied="{ item }">
              <v-simple-checkbox
                v-model="item.applied"
                disabled
              ></v-simple-checkbox>
            </template>
          </v-data-table>
        </template>
      </div>
    </v-card>

    <div class="ma-0 py-4 px-5 bg-white" :style="{ position:'fixed',bottom:0,left:0,boxShadow: '0px 4px 20px 0px #2323233D',width:'calc(58.3333333% - 8px)' }">
      <v-row align="start" no-gutters>
        <v-col cols="12">
          <v-btn
            block
            class="pa-1 elevation-0"
            large
            :style="{ backgroundColor:'#383838',height:'60px',borderRadius:'10px' }"
            dark
            @click="back_to_invoice"
            >
              <span class="d-flex justify-center align-center" style="gap:6px">
                <v-img src="/assets/poszaviago/js/posapp/components/icons/Reply.svg" max-width="25"></v-img>
                ย้อนกลับ
              </span>
            </v-btn
          >
        </v-col>
      </v-row>
    </div>
  </div>
</template>

<script>
import { evntBus } from '../../bus';
export default {
  data: () => ({
    loading: false,
    pos_profile: '',
    customer: '',
    posa_coupons: [],
    new_coupon: null,
    itemsPerPage: 1000,
    singleExpand: true,
    items_headers: [
      { text: "คูปอง", value: 'coupon_code', align: 'start' },
      { text: "ประเภท", value: 'type', align: 'start' },
      { text: "รายละเอียด", value: 'pos_offer', align: 'start' },
      { text: "ใช้งาน?", value: 'applied', align: 'start' },
    ],
  }),

  computed: {
    couponsCount() {
      return this.posa_coupons.length;
    },
    appliedCouponsCount() {
      return this.posa_coupons.filter((el) => !!el.applied).length;
    },
  },

  methods: {
    back_to_invoice() {
      evntBus.$emit('show_coupons', 'false');
    },
    add_coupon(new_coupon) {
      if (!this.customer || !new_coupon) return;
      const exist = this.posa_coupons.find(
        (el) => el.coupon_code == new_coupon
      );
      if (exist) {
        evntBus.$emit('show_mesage', {
          text: 'คูปองถูกใช้แล้ว!',
          color: 'error',
        });
        return;
      }
      const vm = this;
      frappe.call({
        method: 'poszaviago.poszaviago.api.posapp.get_pos_coupon',
        args: {
          coupon: new_coupon,
          customer: vm.customer,
          company: vm.pos_profile.company,
        },
        callback: function (r) {
          if (r.message) {
            const res = r.message;
            if (res.msg != 'Apply' || !res.coupon) {
              evntBus.$emit('show_mesage', {
                text: "ไม่พบคูปองนี้",
                color: 'error',
              });
            } else {
              vm.new_coupon = null;
              const coupon = res.coupon;
              vm.posa_coupons.push({
                coupon: coupon.name,
                coupon_code: coupon.coupon_code,
                type: coupon.coupon_type,
                applied: 0,
                pos_offer: coupon.pos_offer,
                customer: coupon.customer || vm.customer,
              });
            }
          }
        },
      });
    },
    setActiveGiftCoupons() {
      if (!this.customer) return;
      const vm = this;
      frappe.call({
        method: 'poszaviago.poszaviago.api.posapp.get_active_gift_coupons',
        args: {
          customer: vm.customer,
          company: vm.pos_profile.company,
        },
        callback: function (r) {
          if (r.message) {
            const coupons = r.message;
            coupons.forEach((coupon_code) => {
              vm.add_coupon(coupon_code);
            });
          }
        },
      });
    },

    updatePosCoupons(offers) {
      this.posa_coupons.forEach((coupon) => {
        const offer = offers.find(
          (el) => el.offer_applied && el.coupon == coupon.coupon
        );
        if (offer) {
          coupon.applied = 1;
        } else {
          coupon.applied = 0;
        }
      });
    },

    removeCoupon(reomove_list) {
      this.posa_coupons = this.posa_coupons.filter(
        (coupon) => !reomove_list.includes(coupon.coupon)
      );
    },
    updateInvoice() {
      evntBus.$emit('update_invoice_coupons', this.posa_coupons);
    },
    updateCounters() {
      evntBus.$emit('update_coupons_counters', {
        couponsCount: this.couponsCount,
        appliedCouponsCount: this.appliedCouponsCount,
      });
    },
  },

  watch: {
    posa_coupons: {
      deep: true,
      handler() {
        this.updateInvoice();
        this.updateCounters();
      },
    },
  },

  created: function () {
    this.$nextTick(function () {
      evntBus.$on('register_pos_profile', (data) => {
        this.pos_profile = data.pos_profile;
      });
    });
    evntBus.$on('update_customer', (customer) => {
      if (this.customer != customer) {
        const to_remove = [];
        this.posa_coupons.forEach((el) => {
          if (el.type == 'Promotional') {
            el.customer = customer;
          } else {
            to_remove.push(el.coupon);
          }
        });
        this.customer = customer;
        if (to_remove.length) {
          this.removeCoupon(to_remove);
        }
      }
      this.setActiveGiftCoupons();
    });
    evntBus.$on('update_pos_coupons', (data) => {
      this.updatePosCoupons(data);
    });
    evntBus.$on('set_pos_coupons', (data) => {
      this.posa_coupons = data;
    });
  },
};
</script>

<style scoped>

.table-lists .v-data-table__wrapper table {
  width:120% !important;
}

.table-list-title {
  font-weight:600 !important;
  font-size:28px;
}

</style>
