<template>
  <div>
    <img :src="bgImg" id="background-img" :style="style.backgroundImg"/>
    <div
        :class="this.class.title"
        :style="{ color: $vuetify.theme.themes.light.primary }"
    >
      总览
    </div>
    <v-hover v-slot="{ hover }">
      <v-card
          class="mt-9 transition-swing"
          :class="[isMobile ? 'mx-auto' : 'ml-9', `elevation-${hover ? 24 : 6}`]"
          :style="{ width: isMobile ? '90%' : '50%' }"
      >
        <v-row no-gutters align="center">
          <v-col class="pl-6">
            <v-row class="pa-6" align="center">
              {{ currentMonth }}月总支出：
              <span class="text-h6">
                ¥&nbsp;{{ monthStatistics.out.sum }}
              </span>
            </v-row>

            <v-row class="pa-6" align="center">
              {{ currentMonth }}月总收入：
              <span class="text-h6"> ¥&nbsp;{{ monthStatistics.in.sum }} </span>
            </v-row>
          </v-col>
          <v-col>
            <v-row class="my-5" justify="center">
              <div style="width: 150px; height: 150px" ref="pieChart"></div>
            </v-row>
          </v-col>
        </v-row>
      </v-card>
    </v-hover>
    <v-hover v-slot="{ hover }">
      <v-card
          class="mt-9 transition-swing"
          :class="[isMobile ? 'mx-auto' : 'ml-9', `elevation-${hover ? 24 : 6}`]"
          :style="{ width: isMobile ? '90%' : '50%' }"
          v-if="billList != null"
      >
        <v-list two-line>
          <v-list-item-group>
            <v-list-item
                v-for="(item, i) in billList"
                :key="i"
                @click.stop="selectBill(item)"
            >
              <v-list-item-icon class="mr-5" v-if="!isMobile">
                <v-icon x-large color="primary" v-if="item.flow === 'in'">
                  mdi-cash-plus
                </v-icon>
                <v-icon x-large color="error" v-if="item.flow === 'out'">
                  mdi-cash-minus
                </v-icon>
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title>
                  <v-row align="center">
                    <v-col cols="7" class="text-truncate">
                      <span class="text-no-wrap">
                        {{ item.note }}
                      </span>
                    </v-col>
                    <v-col>
                      <v-row
                          align="center"
                          justify="end"
                          no-gutters
                          class="mr-4 text-h6"
                      >
                        <span class="float-right" v-if="item.flow === 'in'">
                          +
                        </span>
                        <span class="float-right" v-if="item.flow === 'out'">
                          -
                        </span>
                        <span class="text-sublime2 float-right">
                          {{ item.amount }}
                        </span>
                      </v-row>
                    </v-col>
                  </v-row>
                </v-list-item-title>
                <v-list-item-subtitle class="mt-2">
                  <v-row>
                    <v-col cols="7">
                      {{ item.type }}
                      <v-chip
                          v-if="item.remark != null && item.remark !== ''"
                          x-small
                          class="mx-2"
                          style="max-width: 100px"
                      >{{ item.remark }}
                      </v-chip
                      >
                    </v-col>
                    <v-col class="text-right pr-6">
                      {{ item.billDate }}
                    </v-col>
                  </v-row>
                </v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </v-card>
    </v-hover>
    <v-dialog v-model="showDialog" max-width="400px">
      <v-card>
        <v-form ref="form" v-model="formValid">
          <v-container>
            <v-row justify="center" no-gutters class="mt-6">
              <v-col cols="12" class="">
                <v-row no-gutters justify="space-around">
                  <v-col cols="5">
                    <v-text-field
                        label="金额"
                        prefix="¥"
                        :rules="[rules.amountRequired, rules.isNumber]"
                        v-model="selectedBill.amount"
                        outlined
                        clearable
                        required
                    ></v-text-field>
                  </v-col>
                  <v-col cols="5">
                    <v-combobox
                        :items="
                        selectedBill.flow === 'in'
                          ? incomeTypeList
                          : outcomeTypeList
                      "
                        v-model="selectedBill.type"
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
                        v-model="selectedBill.remark"
                        outlined
                        clearable
                    ></v-text-field
                    >
                  </v-col>
                  <v-col cols="5">
                    <v-dialog
                        ref="dialog"
                        v-model="showDatePicker"
                        :return-value.sync="selectedBill.billDate"
                        width="290px"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                            v-model="selectedBill.billDate"
                            label="日期"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                            outlined
                        ></v-text-field>
                      </template>
                      <v-date-picker
                          v-model="selectedBill.billDate"
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
                            @click="$refs.dialog.save(selectedBill.billDate)"
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
                        v-model="selectedBill.note"
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
                    <v-btn x-large color="primary" @click="updateBill()"
                    >更新
                    </v-btn>
                  </v-col>
                  <v-col cols="6" class="text-center">
                    <v-btn x-large color="error" @click="deleteBill()"
                    >删除
                    </v-btn>
                  </v-col>
                </v-row>
              </v-col>
            </v-row>
          </v-container>
        </v-form>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  name: "Home",
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
      bgImg: this.GLOBAL.images.ideas,
      billList: null,
      formValid: false,
      showDialog: false,
      showDatePicker: false,
      outcomeTypeList: [
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
      incomeTypeList: [
        "工资",
        "奖金补贴",
        "生活费",
        "人情社交",
        "兼职",
        "投资理财",
        "报销",
        "中奖",
        "其他",
      ],
      selectedBill: {},
      rules: {
        amountRequired: (value) => !!value || "请输入金额",
        typeRequired: (value) => !!value || "请选择类型",
        min: (v) => v.length >= 6 || "长度最少为6位",
        isNumber: (v) =>
            /^\d+$/.test(v) || /^\d+[.]?\d+$/.test(v) || "只能输入数字",
      },
      monthStatistics: {
        in: {
          sum: 0,
        },
        out: {
          sum: 0,
        },
      },
      currentMonth: new Date().getMonth() + 1,
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
    selectBill(item) {
      this.showDialog = true;
      this.selectedBill = item;
    },
    deleteBill() {
      this.axios
          .delete(this.GLOBAL.apiBase + "/bill/deleteBill/" + this.selectedBill.id)
          .then(() => {
            this.showDialog = false;
            this.$notify({
              title: "删除成功",
              message: "",
              type: "success",
              duration: 2000,
            });
            this.loadCountBanner();
            this.axios.get(this.GLOBAL.apiBase + "/bill/listBill").then((response) => {
              this.billList = response.data.data;
            });
          });
    },
    updateBill() {
      if (this.$refs.form.validate()) {
        if (
            this.selectedBill.billDate == null ||
            this.selectedBill.billDate === ""
        ) {
          this.selectedBill.billDate = new Date().Format("yyyy-MM-dd");
        }
        this.axios
            .put(this.GLOBAL.apiBase + "/bill/updateBill", JSON.stringify(this.selectedBill), {
              headers: {
                "Content-Type": "application/json;charset=UTF-8",
              },
            })
            .then(() => {
              this.showDialog = false;
              this.loadCountBanner();
              this.$notify({
                title: "更新成功",
                message: "",
                type: "success",
                duration: 2000,
              });
            });
      }
    },
    initCharts() {
      let pieChart = this.$echarts.init(this.$refs.pieChart);
      const _this = this;
      pieChart.setOption({
        tooltip: {
          trigger: "item",
        },
        series: [
          {
            name: "收支统计",
            type: "pie",
            radius: ["45%", "90%"],
            data: [
              {value: _this.monthStatistics.in.sum, name: "收入"},
              {value: _this.monthStatistics.out.sum, name: "支出"},
            ],
            color: ["#9ECA7F", "#5A6FC0"],

            itemStyle: {
              normal: {
                label: {
                  show: true,
                  position: "", //标签的位置
                  textStyle: {color: "#3c4858"},
                },
                labelLine: {
                  show: true,
                  lineStyle: {color: "#3c4858"},
                },
              },
              emphasis: {
                itemStyle: {
                  shadowBlur: 10,
                  shadowOffsetX: 0,
                  shadowColor: "rgba(0, 0, 0, 0.5)",
                },
              },
            },
          },
        ],
      });
    },
    loadCountBanner() {
      this.axios
          .get(
              this.GLOBAL.apiBase + "/bill/statisticsMonthBill?countMonth=" +
              new Date().Format("yyyy-MM")
          )
          .then((response) => {
            let data = response.data.data;
            if (data != null) {
              if (data.in != null) {
                this.monthStatistics.in = data.in;
              }
              if (data.out != null) {
                this.monthStatistics.out = data.out;
              }
            }
            this.initCharts();
          });
    },
  },
  mounted() {
    this.axios.get(this.GLOBAL.apiBase + "/bill/listBill").then((response) => {
      if (response.data.data != null && response.data.data.length > 0) {
        this.billList = response.data.data;
      }
    });
    this.loadCountBanner();
  },
};
</script>

<style lang="scss" scoped>
#background-img {
  position: fixed;
}
</style>