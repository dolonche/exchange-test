<template>
  <div class="radio-currency">
    <input type="radio"
           :name="nameAttr"
           :id="id"
           class="radio-currency__input"
    >
    <label :for="id"
           class="radio-currency__label"
           @click="$emit('radioActive', idNumber)">
      <span class="radio-currency__media">
        <img :src="image"
             alt=""
             class="radio-currency__media-img">
      </span>
      <span class="radio-currency__text">
        <span class="radio-currency__name">{{ labelMainText }}</span>
        <span class="radio-currency__subname"
              v-if="labelSubText && labelSubText[0] !== 'rub'">
                      {{ labelSubText }}
        </span>
      </span>
    </label>
  </div>
</template>

<script lang="ts">
export default {
  name: "RadioCurrency",
  props: {
    nameAttr: {
      type: String,
      required: true,
    },
    id: {
      type: String,
      required: true,
    },
    idNumber: {
      type: Number,
      required: true,
    },
    labelMainText: {
      type: String,
      required: true,
    },
    labelSubText: {
      type: String,
      required: false
    },
    image: {
      type: String,
      required: true,
    },
  }
}
</script>

<style scoped>

</style>