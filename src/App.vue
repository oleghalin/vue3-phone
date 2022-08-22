<script setup>
import { PhoneInput } from "./components/index.js";
import {ref} from "vue";

const phone = ref("")
const object = ref({})
</script>

<template>
  {{ phone }} <br>
  {{ object }}
  <phone-input v-model="phone" v-model:model-object="object"></phone-input>
</template>