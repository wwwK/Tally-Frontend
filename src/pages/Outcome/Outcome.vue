<template>
  <div>
    <img :src="bgImg" id="background-img" :style="style.backgroundImg"/>
    <div
        :class="this.class.title"
        :style="{ color: $vuetify.theme.themes.light.primary }"
    >
      支出
    </div>

    <v-hover v-slot="{ hover }">
      <v-card
          class="mt-9 transition-swing"
          :class="[
          isMobile ? 'mx-auto' : 'ml-9',
          `elevation-${hover ? 24 : 6}`,
        ]"
          :style="{ width: isMobile ? '90%' : '50%' }"
      >
        <v-form ref="form" v-model="valid">
          <v-container>
            <v-row justify="center" no-gutters class="mt-6">
              <v-col cols="12" class="">
                <v-row no-gutters justify="space-around">
                  <v-col cols="5">
                    <v-text-field
                        label="金额"
                        prefix="¥"
                        :rules="[rules.amountRequired, rules.isNumber]"
                        v-model="bill.amount"
                        outlined
                        clearable
                        required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="5">
                    <v-combobox
                        :items="typeList"
                        v-model="bill.type"
                        :rules="[rules.typeRequired]"
                        label="类型"
                        clearable
                        outlined
                        required
                    ></v-combobox>
                  </v-col>
                </v-row>
                <v-row justify="space-around" no-gutters>
                  <v-col cols="5">
                    <v-text-field
                        label="标签"
                        v-model="bill.remark"
                        outlined
                        clearable
                    ></v-text-field
                    >
                  </v-col>
                  <v-col cols="5">
                    <v-dialog
                        ref="dialog"
                        v-model="showDatePicker"
                        :return-value.sync="bill.billDate"
                        width="290px"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                            v-model="bill.billDate"
                            label="日期"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                            outlined
                        ></v-text-field>
                      </template>
                      <v-date-picker
                          v-model="bill.billDate"
                          scrollable
                          locale="zh-cn"
                      >
                        <v-spacer></v-spacer>
                        <v-btn
                            text
                            color="error"
                            @click="showDatePicker = false"
                        >
                          取消
                        </v-btn>
                        <v-btn
                            text
                            color="primary"
                            @click="$refs.dialog.save(bill.billDate)"
                        >
                          确认
                        </v-btn>
                      </v-date-picker>
                    </v-dialog>
                  </v-col>
                </v-row>
                <v-row no-gutters justify="center">
                  <v-col cols="11">
                    <v-textarea
                        v-model="bill.note"
                        auto-grow
                        rows="1"
                        outlined
                        clearable
                        label="备注"
                    ></v-textarea
                    >
                  </v-col>
                </v-row>
                <v-row no-gutters justify="space-around" class="mb-6">
                  <v-col cols="6" class="text-center">
                    <v-btn x-large color="primary" @click="submitForm()"
                    >提交
                    </v-btn>
                  </v-col>
                  <v-col cols="6" class="text-center">
                    <v-btn x-large color="error" @click="resetForm()"
                    >清空
                    </v-btn>
                  </v-col>
                </v-row>
              </v-col>
            </v-row>
          </v-container>
        </v-form>
      </v-card>
    </v-hover>
  </div>
</template>

<script>
export default {
  name: "Outcome",
  components: {},
  computed: {
    isMobile: function () {
      return this.$vuetify.breakpoint.mobile;
    },
    isDark: function () {
      return this.$vuetify.theme.dark;
    },
  },
  data: function () {
    return {
      bgImg: this.GLOBAL.images.coding,
      valid: false,
      typeList: [
        "餐饮",
        "水果零食",
        "购物",
        "住房",
        "生活服务",
        "通讯",
        "生活日用",
        "人情社交",
        "穿搭美容",
        "医疗保健",
        "休闲娱乐",
        "家居家电",
        "网络虚拟",
        "其他",
      ],
      bill: {
        flow: "out",
        amount: null,
        type: null,
        remark: null,
        note: null,
        billDate: null,
      },
      showDatePicker: false,
      rules: {
        amountRequired: (value) => !!value || "请输入金额",
        typeRequired: (value) => !!value || "请选择类型",
        min: (v) => v.length >= 6 || "长度最少为6位",
        isNumber: (v) =>
            /^\d+$/.test(v) || /^\d+[.]?\d+$/.test(v) || "只能输入数字",
      },
      style: {
        backgroundImg: {
          width: this.$vuetify.breakpoint.mobile ? "60vw" : "20vw",
          bottom: this.$vuetify.breakpoint.mobile ? "20vw" : "3vw",
          right: this.$vuetify.breakpoint.mobile ? "3vw" : "3vw",
        },
      },
      class: {
        title: {
          "text-h2": !this.$vuetify.breakpoint.mobile,
          "text-h3": this.$vuetify.breakpoint.mobile,
          "ml-3": this.$vuetify.breakpoint.mobile,
          "ml-9": !this.$vuetify.breakpoint.mobile,
          "mt-3": this.$vuetify.breakpoint.mobile,
          "mt-9": !this.$vuetify.breakpoint.mobile,
        },
      },
    };
  },
  methods: {
    submitForm() {
      if (this.$refs.form.validate()) {
        if (this.bill.billDate == null || this.bill.billDate === "") {
          this.bill.billDate = new Date().Format("yyyy-MM-dd");
        }
        this.axios
            .post(
                this.GLOBAL.apiBase + "/bill/insertBill",
                JSON.stringify(this.bill),
                {
                  headers: {
                    "Content-Type": "application/json;charset=UTF-8",
                  },
                }
            )
            .then(() => {
              this.$notify({
                title: "提交成功",
                message: "",
                type: "success",
                duration: 2000,
              });
            });
      }
    },
    resetForm() {
      this.$refs.form.reset();
    },
  },
  mounted() {
    this.bill.billDate = new Date().Format("yyyy-MM-dd");
  },
};
</script>

<style lang="scss" scoped>
#background-img {
  position: fixed;
}
</style>