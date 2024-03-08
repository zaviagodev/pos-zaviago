<template>
  <v-row justify="center">
    <v-dialog v-model="addressDialog" max-width="600px">
      <v-card class="px-6 py-8" :style="{ borderRadius:'10px' }">
        <v-card-title class="pa-0 pb-8">
          <span class="modal-title">เพิ่มที่อยู่ใหม่</span>
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container>
            <v-row>
              <v-col cols="12" class="px-0">
                <v-text-field
                  dense
                  outlined
                  color="primary"
                  label="ชื่อที่อยู่"
                  background-color="white"
                  hide-details
                  v-model="address.name"
                ></v-text-field>
              </v-col>
              <v-col cols="12" class="px-0">
                <v-text-field
                  dense
                  outlined
                  color="primary"
                  label="ที่อยู่บรรทัดที่ 1"
                  background-color="white"
                  hide-details
                  v-model="address.address_line1"
                ></v-text-field>
              </v-col>
              <v-col cols="12" class="px-0">
                <v-text-field
                  dense
                  outlined
                  color="primary"
                  label="แขวง/ตำบล"
                  background-color="white"
                  hide-details
                  v-model="address.address_line2"
                ></v-text-field>
              </v-col>
              <v-col cols="6" class="pl-0">
                <v-text-field
                  label="เขต/อำเภอ"
                  dense
                  outlined
                  color="primary"
                  background-color="white"
                  hide-details
                  v-model="address.city"
                ></v-text-field>
              </v-col>
              <v-col cols="6" class="pr-0">
                <v-text-field
                  label="จังหวัด"
                  dense
                  outlined
                  background-color="white"
                  hide-details
                  v-model="address.state"
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
    addressDialog: false,
    address: {},
    customer: '',
  }),

  methods: {
    close_dialog() {
      this.addressDialog = false;
    },

    submit_dialog() {
      const vm = this;
      this.address.customer = this.customer;
      this.address.doctype = 'Customer';
      frappe.call({
        method: 'poszaviago.poszaviago.api.posapp.make_address',
        args: {
          args: this.address,
        },
        callback: (r) => {
          if (!r.exc) {
            evntBus.$emit('add_the_new_address', r.message);
            evntBus.$emit('show_mesage', {
              text: 'สร้างที่อยู่ใหม่สำเร็จ',
              color: 'success',
            });
            vm.addressDialog = false;
            vm.customer = '';
            vm.address = {};
          }
        },
      });
    },
  },
  created: function () {
    evntBus.$on('open_new_address', (data) => {
      this.addressDialog = true;
      this.customer = data;
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