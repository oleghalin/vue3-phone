<script setup>
import {computed, nextTick, onBeforeMount, ref, toRef, watch} from "vue";
import {default as parsePhoneWithLib} from 'libphonenumber-js'
import {onClickOutside} from '@vueuse/core'

const props = defineProps({
  modelValue: {
    required: true
  },
  modelObject: {
    required: false,
    type: Object
  },
  preselectFirst: {
    default: true
  },
  prependCode: {
    default: true,
    type: Boolean
  },
  autoFormat: {
    default: true,
    type: Boolean
  },
})

const emit = defineEmits(["update:modelValue", "update:modelObject"])

onBeforeMount(() => {
  if (props.preselectFirst) {
    selectedCountry.value = countries[0]
  }

  if (props.modelValue === "" && props.prependCode) {
    emitChange(`+${selectedCountry.value.code}`)
  }
})

const emitChange = (value) => {
  emit("update:modelValue", value)
}

const showDropdown = ref(false)

const countries = [
  {
    label: "Ukraine",
    iso: "ua",
    code: "380"
  },
  {
    label: "Spain",
    iso: "es",
    code: "34"
  },
]

const searchQuery = ref("")

const filteredCountries = computed(() => searchQuery.value === "" ? countries : countries.filter((country) => {
  return country.label.toLowerCase().includes(searchQuery.value.toLowerCase())
}));

const selectedCountry = ref(null)

const onCountrySelect = (country) => {
  selectedCountry.value = country

  showDropdown.value = false
}

const inputRef = ref(null)
const dropdownRef = ref(null)
const dropdownButtonRef = ref(null)

onClickOutside(dropdownRef, () => {
  if (showDropdown.value) {
    showDropdown.value = false
  }
}, {
  ignore: [dropdownButtonRef]
})


const parsePhone = (value) => {
  let phone = value

  if (phone[0] !== "+") {
    phone = `+${phone}`
  }

  return parsePhoneWithLib(phone)
}

const isAllowedKey = (value) => /^[()\-+\d\s]*$/.test(value)

const allowedKeys = [
    8, //Backspace
    46, // Delete
]

const onKeyDown = (event) => {
  const char = String.fromCharCode(event.which);

  if (event.metaKey || event.ctrlKey || allowedKeys.includes(event.which)) {
    return;
  }

  if (!isAllowedKey(char)) {
    event.preventDefault()
  }
}

const onInput = (event) => {
  let value = event.target.value

  const parsedPhone = parsePhone(value);

  if (props.autoFormat && parsedPhone) {
    value = parsedPhone.formatInternational()
  }

  if (props.modelObject) {
    emit("update:modelObject", parsedPhone)
  }

  emitChange(value)
}

const phone = toRef(props, "modelValue")
</script>

<template>
  <div class="wrapper">
    <div class="wrapper__input-dropdown" v-show="showDropdown" ref="dropdownRef">
      <input class="wrapper__search-input" type="text" v-model="searchQuery" placeholder="Type to search..">
      <ul class="wrapper__list">
        <li class="wrapper__list-item"
            :class="{selected: country.iso === selectedCountry.iso}"
            v-for="(country, index) in filteredCountries"
            :key="index"
            @click="onCountrySelect(country)">
          <span>{{ country.iso }}</span>
          <span>{{ country.label }}</span>
          <span class="wrapper__list-item-code">+{{ country.code }}</span>
        </li>
      </ul>
    </div>
    <div class="wrapper__button">
      <button @click="showDropdown = true" ref="dropdownButtonRef">
        {{ selectedCountry?.iso }}
      </button>
    </div>
    <input type="text" class="wrapper__input" :value="phone" @input="onInput" @keydown="onKeyDown" ref="inputRef">
  </div>
</template>

<style lang="postcss">
.wrapper {
  display: flex;
  position: relative;
  align-items: center;
  height: theme(height.8);
  background-color: theme('colors.gray.500');
  padding: theme(padding.2);
}

.wrapper__input-dropdown {
  left: 0;
  top: theme(height.9);
  background-color: theme('colors.red.500');
  position: absolute;
  min-height: theme(height.48);
  width: 100%;
  border-radius: theme('borderRadius.xl');
}

.wrapper__list {
  color: theme('colors.white');
}

.wrapper__search-input {
  width: 100%;
}

.wrapper__list-item {
  display: flex;
  padding: theme(padding.2);

  &:first-child:hover {
    border-radius: theme('borderRadius.xl') theme('borderRadius.xl') 0 0;
  }

  &:hover {
    cursor: pointer;
    background-color: theme('colors.red.400');
  }

  &.selected {
    background-color: theme('colors.red.200');
  }

  &.selected:hover {
    background-color: theme('colors.red.100');
  }
}

.wrapper__list-item-code {
  margin-left: auto;
}
</style>