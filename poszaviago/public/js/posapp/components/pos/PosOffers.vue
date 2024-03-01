<template>
  <div>
    <v-card
      class="selection mx-auto elevation-0"
      style="max-height: 80vh; height: 80vh"
    >
      <v-card-title class="ml-n1 pt-6 pb-5 px-5 mr-2" :style="{ borderRight:'1px solid #F4F4F4',borderBottom:'1px solid #DFDFDF' }">
        <span class="text-h5 font-weight-bold">โปรโมชัน</span>
      </v-card-title>
      <div class="ma-0 ml-n1 px-5 pt-6 overflow-y-auto" :style="{ height: '80vh', maxHeight:'80vh',backgroundColor:'#DFDFDF', scrollbarWidth:'none', width:'calc(100% - 4px)',paddingBottom:'100px' }">
        <template @mouseover="style = 'cursor: pointer'">
          <v-data-table
            :headers="items_headers"
            :items="pos_offers"
            :single-expand="singleExpand"
            :expanded.sync="expanded"
            show-expand
            item-key="row_id"
            class="elevation-0 table-lists"
            :items-per-page="itemsPerPage"
            hide-default-footer
            :style="{ borderRadius:'10px' }"
          >
            <template v-slot:item.offer_applied="{ item }">
              <v-simple-checkbox
                @click="forceUpdateItem"
                v-model="item.offer_applied"
                :disabled="
                  (item.offer == 'Give Product' &&
                    !item.give_item &&
                    (!offer.replace_cheapest_item || !offer.replace_item)) ||
                  (item.offer == 'Grand Total' &&
                    discount_percentage_offer_name &&
                    discount_percentage_offer_name != item.name)
                "
              ></v-simple-checkbox>
            </template>
            <template v-slot:expanded-item="{ headers, item }">
              <td :colspan="headers.length">
                <v-row class="my-0">
                  <v-col v-if="item.description">
                    <div
                      class="primary--text"
                      v-html="handleNewLine(item.description)"
                    ></div>
                  </v-col>
                  <v-col v-if="item.offer == 'Give Product'">
                    <v-autocomplete
                      v-model="item.give_item"
                      :items="get_give_items(item)"
                      item-text="item_code"
                      outlined
                      dense
                      color="primary"
                      :label="frappe._('Give Item')"
                      :disabled="
                        item.apply_type != 'Item Group' ||
                        item.replace_item ||
                        item.replace_cheapest_item
                      "
                    ></v-autocomplete>
                  </v-col>
                </v-row>
              </td>
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
import format from '../../format';
export default {
  mixins: [format],
  data: () => ({
    loading: false,
    pos_profile: '',
    pos_offers: [],
    allItems: [],
    discount_percentage_offer_name: null,
    itemsPerPage: 1000,
    expanded: [],
    singleExpand: true,
    items_headers: [
      { text: "ชื่อโปรโมชัน", value: 'name', align: 'start' },
      { text: "ใช้งานกับ", value: 'apply_on', align: 'start' },
      { text: "ข้อเสนอ", value: 'offer', align: 'start' },
      { text: "ใช้งาน", value: 'offer_applied', align: 'start' },
    ],
  }),

  computed: {
    offersCount() {
      return this.pos_offers.length;
    },
    appliedOffersCount() {
      return this.pos_offers.filter((el) => !!el.offer_applied).length;
    },
  },

  methods: {
    back_to_invoice() {
      evntBus.$emit('show_offers', 'false');
    },
    forceUpdateItem() {
      let list_offers = [];
      list_offers = [...this.pos_offers];
      this.pos_offers = list_offers;
    },
    makeid(length) {
      let result = '';
      const characters = 'abcdefghijklmnopqrstuvwxyz0123456789';
      const charactersLength = characters.length;
      for (var i = 0; i < length; i++) {
        result += characters.charAt(
          Math.floor(Math.random() * charactersLength)
        );
      }
      return result;
    },
    updatePosOffers(offers) {
      const toRemove = [];
      this.pos_offers.forEach((pos_offer) => {
        const offer = offers.find((offer) => offer.name === pos_offer.name);
        if (!offer) {
          toRemove.push(pos_offer.row_id);
        }
      });
      this.removeOffers(toRemove);
      offers.forEach((offer) => {
        const pos_offer = this.pos_offers.find(
          (pos_offer) => offer.name === pos_offer.name
        );
        if (pos_offer) {
          pos_offer.items = offer.items;
          if (
            pos_offer.offer === 'Grand Total' &&
            !this.discount_percentage_offer_name
          ) {
            pos_offer.offer_applied = !!pos_offer.auto;
          }
          if (
            offer.apply_on == 'Item Group' &&
            offer.apply_type == 'Item Group' &&
            offer.replace_cheapest_item
          ) {
            pos_offer.give_item = offer.give_item;
            pos_offer.apply_item_code = offer.apply_item_code;
          }
        } else {
          const newOffer = { ...offer };
          if (!offer.row_id) {
            newOffer.row_id = this.makeid(20);
          }
          if (offer.apply_type == 'Item Code') {
            newOffer.give_item = offer.apply_item_code || 'Nothing';
          }
          if (offer.offer_applied) {
            newOffer.offer_applied == !!offer.offer_applied;
          } else {
            if (
              offer.apply_type == 'Item Group' &&
              offer.offer == 'Give Product' &&
              !offer.replace_cheapest_item &&
              !offer.replace_item
            ) {
              newOffer.offer_applied = false;
            } else if (
              offer.offer === 'Grand Total' &&
              this.discount_percentage_offer_name
            ) {
              newOffer.offer_applied = false;
            } else {
              newOffer.offer_applied = !!offer.auto;
            }
          }
          if (newOffer.offer == 'Give Product' && !newOffer.give_item) {
            newOffer.give_item = this.get_give_items(newOffer)[0].item_code;
          }
          this.pos_offers.push(newOffer);
          evntBus.$emit('show_mesage', {
            text: __('New Offer Available'),
            color: 'warning',
          });
        }
      });
    },
    removeOffers(offers_id_list) {
      this.pos_offers = this.pos_offers.filter(
        (offer) => !offers_id_list.includes(offer.row_id)
      );
    },
    handelOffers() {
      const applyedOffers = this.pos_offers.filter(
        (offer) => offer.offer_applied
      );
      evntBus.$emit('update_invoice_offers', applyedOffers);
    },
    handleNewLine(str) {
      if (str) {
        return str.replace(/(?:\r\n|\r|\n)/g, '<br />');
      } else {
        return '';
      }
    },
    get_give_items(offer) {
      if (offer.apply_type == 'Item Code') {
        return [offer.apply_item_code];
      } else if (offer.apply_type == 'Item Group') {
        const items = this.allItems;
        let filterd_items = [];
        const filterd_items_1 = items.filter(
          (item) => item.item_group == offer.apply_item_group
        );
        if (offer.less_then > 0) {
          filterd_items = filterd_items_1.filter(
            (item) => item.rate < offer.less_then
          );
        } else {
          filterd_items = filterd_items_1;
        }
        return filterd_items;
      } else {
        return [];
      }
    },
    updateCounters() {
      evntBus.$emit('update_offers_counters', {
        offersCount: this.offersCount,
        appliedOffersCount: this.appliedOffersCount,
      });
    },
    updatePosCoupuns() {
      const applyedOffers = this.pos_offers.filter(
        (offer) => offer.offer_applied && offer.coupon_based
      );
      evntBus.$emit('update_pos_coupons', applyedOffers);
    },
  },

  watch: {
    pos_offers: {
      deep: true,
      handler(pos_offers) {
        this.handelOffers();
        this.updateCounters();
        this.updatePosCoupuns();
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
        this.offers = [];
      }
    });
    evntBus.$on('update_pos_offers', (data) => {
      this.updatePosOffers(data);
    });
    evntBus.$on('update_discount_percentage_offer_name', (data) => {
      this.discount_percentage_offer_name = data.value;
    });
    evntBus.$on('set_all_items', (data) => {
      this.allItems = data;
    });
  },
};
</script>

<style scoped>

.table-lists .v-data-table__wrapper table {
  width:120% !important;
}

</style>