<template>
  <div class="app-wrapper p-10 w-full min-h-screen bg-gray-900 text-white">
    <div class="flex justify-between">
      <div class="mr-5">Vending machine</div>
      <div class="flex flex-col justify-end items-end">
        <div>
          Duit
          <span class="font-bold">{{ balance.toLocaleString() }}</span>
        </div>
        <div>
          Pecahan duidmu
          <span class="font-bold">{{ fractionSelected.toLocaleString() }}</span>
        </div>
        <select class="text-black" v-model="fractionSelected">
          <option
            v-for="(item, index) in fraction"
            :key="index"
            :value="item"
            @change="validateFractionSelected($evt)"
            >{{ item }}</option
          >
        </select>
      </div>
    </div>
    <div class="vending-machine flex bg-gray-800 mt-5" style="width: 600;">
      <div class="col w-4/6 grid grid-cols-3 gap-2 bg-gray-700 p-5">
        <div
          class="col-span-1 bg-blue-500 p-5 flex flex-col"
          v-for="(item, i) in items"
          :key="i"
        >
          <div>{{ item.label }}</div>
          <div>stock {{ item.stock }}</div>
          <div>code {{ item.code }}</div>
          <div>price {{ item.price.toLocaleString() }}</div>
        </div>
      </div>
      <div class="col w-2/6 px-5 pb-6">
        <div class="mt-5 mb-3">kode barang</div>
        <div class="grid grid-cols-5 gap-2">
          <div class="btn col-span-1" v-for="(item, i) in items" :key="i">
            <div
              class="bg-gray-900 text-white flex items-center justify-center cursor-pointer"
              @click="updateUserChoice('code', item.code)"
              :style="btnCodestyleActive('code', item.code)"
            >
              {{ item.code }}
            </div>
          </div>
        </div>
        <div class="mt-5 mb-2">pecahan</div>
        <div class="grid grid-cols-2 gap-2">
          <div
            class="col-span-1 bg-gray-900 px-2 cursor-pointer"
            v-for="(item, i) in fraction"
            :key="i"
            @click="updateUserChoice('fraction', item)"
            :style="btnCodestyleActive('fraction', item)"
          >
            {{ item.toLocaleString() }}
          </div>
        </div>
        <div class="mt-5">
          {{ msg }}
        </div>
        <button
          class="w-full py-3 bg-green-500 mt-5 cursor-pointer text-center"
          @click="buy"
          :disabled="successBuy"
        >
          buy
        </button>
        <div class="output-items py-16 px-5 bg-gray-900 mt-5">
          <div
            class="p-5 bg-teal-500 cursor-pointer"
            v-if="successBuy"
            @click="reset"
          >
            {{ userChoice.item.label }}
          </div>
        </div>
      </div>
    </div>
    <div>your items</div>
    <div>{{ itemChoice }}</div>
  </div>
</template>

<script>
import { mock } from "../utils/mock";
export default {
  data: () => ({
    balance: 50000,
    fractionSelected: 0,
    items: mock,
    successBuy: false,
    fraction: [2000, 5000, 10000, 20000, 50000],
    itemChoice: [],
    msg: "",
    userChoice: {
      code: null,
      fraction: null,
      item: {},
    },
  }),
  watch: {
    fractionSelected(val) {
      if (val > this.balance) {
        this.fractionSelected = 0;
      }
    },
    msg(val) {
      if (val) {
        setTimeout(() => {
          this.msg = "";
        }, 2000);
      }
    },
    successBuy(val) {
      if (val) {
        setTimeout(() => {
          this.reset();
        }, 2000);
      }
    },
  },
  methods: {
    updateUserChoice(type, value) {
      this.userChoice[type] = value;
      if (type == "code") {
        this.getItem(value);
      }
    },
    getItem(value) {
      let res = this.items.filter((item) => item.code == value)[0];
      this.userChoice.item = res;
    },
    btnCodestyleActive(type, val) {
      return {
        backgroundColor: val == this.userChoice[type] ? "deeppink" : "",
      };
    },
    buy() {
      if (
        this.userChoice.item == null ||
        this.userChoice.fraction == null ||
        this.userChoice.code == null ||
        this.fractionSelected == 0
      ) {
        // validate error
      } else {
        let checkFraction = this.fractionSelected == this.userChoice.fraction;
        if (checkFraction) {
          let isMoneyEnoughWithItem =
            this.fractionSelected >= this.userChoice.item.price;
          if (isMoneyEnoughWithItem) {
            let indexItem = this.items.findIndex(
              (obj) => obj == this.userChoice.item
            );
            this.items[indexItem].stock -= 1;
            this.balance = this.balance - this.userChoice.item.price;
            this.itemChoice.push({
              label: this.userChoice.item.label,
              price: this.userChoice.item.price,
            });

            this.successHandler();
          } else {
            this.msg = "duidmu tidak cukup untuk membeli barang ini!";
          }
        } else {
          this.msg = "Pecahan duidmu tidak sama";
        }
      }
    },
    successHandler() {
      this.successBuy = true;
      this.msg = "ambil barangmu";
    },
    reset() {
      this.successBuy = false;
      this.msg = "";
      this.userChoice = {
        fraction: null,
        code: null,
        item: null,
      };
    },
  },
};
</script>
