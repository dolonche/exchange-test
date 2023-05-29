<template>
  <main class="page-main">
    <form action="" class="exchange">
      <fieldset class="exchange__step">
        <div class="exchange__step-header">
          <b class="exchange__step-title">Отдаете</b>
          <div class="exchange__step-search">
            <input type="search"
                   name="search-in"
                   id="search-in"
                   class="exchange__step-search-input"
                   placeholder="Поиск валюты"
                   v-model="searchIn"
            >
          </div>
        </div>
        <div class="exchange__options-list">
          <RadioCurrency
              v-for="currencyInItem in searchCurrency(currencyIn, searchIn)"
              :idNumber='currencyInItem.ids[0]'
              :id="`currency-in-${currencyInItem.ids[0]}`"
              :nameAttr='"currency-in"'
              :labelMainText='currencyInItem.name'
              :labelSubText=currencyInItem.currency[0]
              :image='currencyInItem.logo.withBackground'
              :key="currencyInItem.ids[0]"
              @radioActive=getCurrencyOut
          ></RadioCurrency>
        </div>
      </fieldset>
      <fieldset class="exchange__step">
        <div class="exchange__step-header">
          <b class="exchange__step-title">Получаете</b>
          <div class="exchange__step-search">
            <input type="search"
                   name="search-out"
                   id="search-out"
                   class="exchange__step-search-input"
                   placeholder="Поиск валюты"
                   v-model="searchOut"
            >
          </div>
        </div>
        <div class="exchange__options-list">
          <span class="exchange__options-before"
                v-if="activeCurrencyInId === 0">{{ getCurrencyOutMessage }}</span>
          <RadioCurrency
              v-else
              v-for="currencyOutItem in searchCurrency(currencyOut, searchOut)"
              :idNumber='currencyOutItem.ids[0]'
              :id="`currency-out-${currencyOutItem.ids[0]}`"
              :nameAttr='"currency-out"'
              :labelMainText='currencyOutItem.name'
              :labelSubText=currencyOutItem.currency[0]
              :image='currencyOutItem.logo.withBackground'
              :key="currencyOutItem.ids[0]"
              @radioActive=setActiveCurrencyOut
          ></RadioCurrency>
        </div>
      </fieldset>
      <fieldset class="exchange__step exchange__step--final">
        <div class="exchange__final-currency">
          <b class="exchange__final-currency-title">Отдаете</b>
          <div class="exchange__final-currency-value">
            <div class="exchange__final-currency-info">
              <img v-if="activeCurrencyIn.logo"
                   :src="activeCurrencyIn.logo.withBackground"
                   :alt="activeCurrencyIn.name"
                   class="exchange__final-currency-img">
              <span class="exchange__final-currency-name">{{ activeCurrencyIn.name }}</span>
            </div>
            <input
                type="number"
                class="exchange__final-currency-value-input"
                :min="currencyPair !== null ? currencyPair.from.min : 0"
                :max="currencyPair !== null ? currencyPair.from.max : 0"
                :id="currencyPair !== null ? currencyPair.structure.from.currency[0].id : ''"
                :disabled="inputDisabled === 1"
                v-model="inCurrencyValue"
                @input="paymentInputCalc('in')"
            >
          </div>
          <div class="exchange__final-currency-sub">
            <span class="exchange__final-currency-sub-min">Мин.: <span
                v-if="currencyPair !== null">{{ currencyPair.from.min }} {{ currencyPair.from.currency }}</span></span>
            <span class="exchange__final-currency-sub-max">Макс.: <span
                v-if="currencyPair !== null">{{ currencyPair.from.max }} {{ currencyPair.from.currency }}</span></span>
          </div>
        </div>
        <div class="exchange__final-currency">
          <b class="exchange__final-currency-title">Получаете</b>
          <div class="exchange__final-currency-value">
            <div class="exchange__final-currency-info">
              <img v-if="activeCurrencyOut.logo"
                   :src="activeCurrencyOut.logo.withBackground"
                   :alt="activeCurrencyOut.name"
                   class="exchange__final-currency-img">
              <span class="exchange__final-currency-name">{{ activeCurrencyOut.name }}</span>
            </div>
            <input
                type="number"
                class="exchange__final-currency-value-input"
                :min="currencyPair !== null ? currencyPair.to.min : 0"
                :max="currencyPair !== null ? currencyPair.to.max : 0"
                :id="currencyPair !== null ? currencyPair.structure.to.currency[0].id : ''"
                :disabled="inputDisabled === 1"
                v-model="outCurrencyValue"
                @input="paymentInputCalc('out')"
            >
          </div>
          <div class="exchange__final-currency-sub">
            <span class="exchange__final-currency-sub-min">Мин.: <span
                v-if="currencyPair !== null">{{ currencyPair.to.min }} {{ currencyPair.to.currency }}</span></span>
            <span class="exchange__final-currency-sub-max">Макс.: <span
                v-if="currencyPair !== null">{{ currencyPair.to.max }} {{ currencyPair.to.currency }}</span></span>
          </div>
        </div>
        <div class="exchange__payment-details" v-if="currencyPair !== null">
          <b class="exchange__payment-details-title">Ваши реквизиты</b>
          <div class="exchange__payment-details-bunch" v-for="input in calcPaymentInput">
            <div class="exchange__payment-details-item">
              <img src="@/assets/img/icons/avatar.svg" alt="" class="exchange__payment-details-item-img">
              <input :type="input.type"
                     class="exchange__payment-details-item-input"
                     :placeholder="input.send.placeholder"
                     :name="input.name"
                     :id="`${input.name}-send`"
              >
            </div>
          </div>
          <div class="exchange__payment-details-check">
            <input type="checkbox" name="agree" id="agree" class="exchange__payment-details-check-input">
            <label for="agree" class="exchange__payment-details-check-label">
              Я согласен с <a href="#">обработкой персональных данных</a>
              и принимаю <a href="#">правила обмена</a>
            </label>
          </div>
          <button type="submit" class="exchange__submit">Перейти к оплате</button>
        </div>
      </fieldset>
    </form>
  </main>
</template>

<script lang="ts">
import {RadioCurrency} from "#components";

type CurrencyItem = {
  ids: number[];
  name: string;
  logo: string[];
  currency: string;
  filter: string[];
};

type CurrencyPair = {
  course: number;
  from: object;
  to: object;
  structure: object;
}

let currencyIdIncludeCalc = (allCurrencyArr: CurrencyItem[], compareCurrencyArr: number[]) => {
  let outArr: [] = [];
  allCurrencyArr.forEach(currencyItem => {
    compareCurrencyArr.forEach((currencyInIdItem, index) => {
      if (currencyItem.ids[0] === currencyInIdItem) {
        outArr.push(currencyItem);
        compareCurrencyArr.slice(index, 1);
        return;
      }
    })
  })
  return outArr;
}
export default defineComponent({
  components: {RadioCurrency},

  async setup() {
    let [{data: currency}, {data: currencyInId}] = await Promise.all([
      useFetch<Currency[]>('https://dev7d8d3h4.sova.gg/api/v1/calculator/'),
      useFetch<object>('https://dev7d8d3h4.sova.gg/api/v1/calculator/from/')
    ]);
    if (!currency.value) {
      throw createError({statusCode: 404, message: 'Произошла ошибка при загрузке данных'})
    }
    let currencyValue: Currency = currency.value;
    let currencyInIdValue: number[] = currencyInId.value;
    let currencyIn: [] = currencyIdIncludeCalc(currencyValue, currencyInIdValue)
    return {currency, currencyInId, currencyIn}
  },
  data() {
    return {
      currency: [],
      currencyInId: [],
      currencyIn: [],
      currencyOutId: [],
      currencyOut: [],
      searchIn: '',
      searchOut: '',
      activeCurrencyInId: 0,
      activeCurrencyOutId: 0,
      activeCurrencyIn: {},
      activeCurrencyOut: {},
      currencyPair: null,
      paymentInput: null,
      outCurrencyValue: 0,
      inCurrencyValue: 0,
      inputDisabled: 1,
      getCurrencyOutMessage: 'Сначала выберите валюту для обмена',
      errorMessage: 'Произошла ошибка при загрузке данных',
    }
  },
  computed: {
    calcPaymentInput() {
      if (this.currencyPair !== null) {
        let inputArr: object[] = [];
        let fromInputs: [] = this.currencyPair.structure.from.input;
        let outInputs: [] = this.currencyPair.structure.to.input;
        inputArr = fromInputs.concat(outInputs);
        return inputArr
      }
    }
  },
  methods: {
    async getCurrencyOut(id: number) {
      this.currencyOutId = await useFetch<object>(`https://dev7d8d3h4.sova.gg/api/v1/calculator/from/${id}/`);
      if (this.currencyOutId.data === null) {
        this.getCurrencyOutMessage = this.errorMessage;
        throw createError({statusCode: 404, message: this.errorMessage});
      }
      this.resetCurrencyOut();
      this.currencyOut = currencyIdIncludeCalc(this.currency, this.currencyOutId.data);
      this.activeCurrencyInId = id;
      this.activeCurrencyIn = this.currency[this.searchActiveCurrency(this.currency, id)];
      if (document.querySelector('.radio-currency__input[name="currency-out"]:checked')) {
        document.querySelector('.radio-currency__input[name="currency-out"]:checked').checked = false;
      }
      this.inputDisabled = 1;
      this.setDefaultPaymentValue();
    },
    async setActiveCurrencyOut(id: number) {
      this.activeCurrencyOutId = id;
      this.activeCurrencyOut = this.currency[this.searchActiveCurrency(this.currency, id)];
      this.currencyPair = await useFetch<CurrencyPair[]>(`https://dev7d8d3h4.sova.gg/api/v1/calculator/pair/${this.activeCurrencyInId}/${this.activeCurrencyOutId}/`).data;
      this.inputDisabled = 0;
      this.setDefaultPaymentValue();
    },
    searchCurrency(currencyList: CurrencyItem[], value: string) {
      return currencyList.filter((currencyListItem) =>
          currencyListItem.name.toLowerCase().includes(value.toLowerCase()) || currencyListItem.currency[0].toLowerCase().includes(value.toLowerCase())
      );
    },
    searchActiveCurrency(currencyArr: CurrencyItem[], id: Number) {
      let currencyIdArr: Number[] = [];
      currencyArr.forEach(currencyItem => currencyIdArr.push(currencyItem.ids[0]));
      return currencyIdArr.indexOf(id)
    },
    paymentInputCalc(type: String) {
      let mantissa: any;
      switch (type) {
        case 'in':
          this.outCurrencyValue = (parseFloat(this.inCurrencyValue * this.currencyPair.course)).toFixed(this.currencyPair.to.round);
          mantissa = (this.inCurrencyValue.toString().includes('.')) ? (this.inCurrencyValue.toString().split('.').pop().length) : (0);
          if (mantissa >= this.currencyPair.from.round) {
            parseFloat(this.inCurrencyValue).toFixed(this.currencyPair.from.round);
          }
          break;
        case 'out':
          this.inCurrencyValue = (parseFloat(this.outCurrencyValue * (1 / this.currencyPair.course))).toFixed(this.currencyPair.from.round);
          mantissa = (this.outCurrencyValue.toString().includes('.')) ? (this.outCurrencyValue.toString().split('.').pop().length) : (0);
          if (mantissa >= this.currencyPair.to.round) {
            parseFloat(this.outCurrencyValue).toFixed(this.currencyPair.to.round);
          }
          break;
      }
    },
    setDefaultPaymentValue() {
      this.outCurrencyValue = 0;
      this.inCurrencyValue = 0;
    },
    resetCurrencyOut() {
      this.activeCurrencyOutId = 0;
      this.currencyPair = null;
      this.activeCurrencyOut = {};
      this.searchOut = '';
    }
  }
})
</script>
