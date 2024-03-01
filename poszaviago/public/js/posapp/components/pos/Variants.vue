<template>
  <v-row justify="center">
    <v-dialog v-model="varaintsDialog" max-width="800px" :style="{ borderRadius:'10px' }">
      <v-card min-height="500px" class="px-6 py-8">
        <v-card-title class="pa-0">
          <span class="headline">เลือกสินค้า</span>
          <v-spacer></v-spacer>
          <v-btn class="elevation-0 px-5" color="gray01" style="color:black;height:40px;border-radius:10px" dark @click="close_dialog">ยกเลิก</v-btn>
        </v-card-title>
        <v-card-text class="pa-0">
          <v-container v-if="parentItem" class="pa-0 pt-5">
            <div v-for="attr in parentItem.attributes" :key="attr.attribute">
              <v-chip-group
                v-model="filters[attr.attribute]"
                active-class="primary--text text--accent-4"
                class="py-3"
                column
              >
                <v-chip
                  v-for="value in attr.values"
                  :key="value.abbr"
                  :value="value.attribute_value"
                  outlined
                  label
                  @click="updateFiltredItems"
                >
                  {{ value.attribute_value }}
                </v-chip>
              </v-chip-group>
              <v-divider class="p-0 m-0"></v-divider>
            </div>
            <div>
              <v-row dense class="overflow-y-auto mt-4" style="max-height: 500px">
                <v-col
                  v-for="(item, idx) in filterdItems"
                  :key="idx"
                  lg="3"
                  md="4"
                  sm="4"
                  cols="4"
                  min-height="50"
                >
                  <v-card hover="hover" @click="add_item(item)" class="elevation-0 pa-3" :style="{ borderRadius:'10px',border:'1px solid #F0F0F0' }">
                    <v-img
                      :src="
                        item.image ||
                        '/assets/poszaviago/js/posapp/components/pos/placeholder-image.png'
                      "
                      class="align-end rounded-lg"
                      height="120px"
                    >
                    </v-img>
                    <v-card-text
                        v-text="item.item_name"
                        class="text-subtitle-2 pa-0 py-3"
                      ></v-card-text>
                    <v-card-text class="pa-0">
                      <div class="text-subtitle-2 font-weight-regular">
                        {{ item.rate || 0 }} {{ item.currency || '' }}
                      </div>
                    </v-card-text>
                  </v-card>
                </v-col>
              </v-row>
            </div>
          </v-container>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import { evntBus } from '../../bus';
export default {
  data: () => ({
    varaintsDialog: false,
    parentItem: null,
    items: null,
    filters: {},
    filterdItems: [],
  }),

  computed: {
    variantsItems() {
      if (!this.parentItem) {
        return [];
      } else {
        return this.items.filter(
          (item) => item.variant_of == this.parentItem.item_code
        );
      }
    },
  },

  methods: {
    close_dialog() {
      this.varaintsDialog = false;
    },
    formtCurrency(value) {
      value = parseFloat(value);
      return value.toFixed(2).replace(/\d(?=(\d{3})+\.)/g, '$&,');
    },
    updateFiltredItems() {
      this.$nextTick(function () {
        const values = [];
        Object.entries(this.filters).forEach(([key, value]) => {
          if (value) {
            values.push(value);
          }
        });

        if (!values.length) {
          this.filterdItems = this.variantsItems;
        } else {
          const itemsList = [];
          this.filterdItems = [];
          this.variantsItems.forEach((item) => {
            let apply = true;
            item.item_attributes.forEach((attr) => {
              if (
                this.filters[attr.attribute] &&
                this.filters[attr.attribute] != attr.attribute_value
              ) {
                apply = false;
              }
            });
            if (apply && !itemsList.includes(item.item_code)) {
              this.filterdItems.push(item);
              itemsList.push(item.item_code);
            }
          });
        }
      });
    },
    add_item(item) {
      evntBus.$emit('add_item', item);
      this.close_dialog();
    },
  },

  created: function () {
    evntBus.$on('open_variants_model', (item, items) => {
      this.varaintsDialog = true;
      this.parentItem = item || null;
      this.items = items;
      this.filters = {};
      this.$nextTick(function () {
        this.filterdItems = this.variantsItems;
      });
    });
  },
};
</script>
