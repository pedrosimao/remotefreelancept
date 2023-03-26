<template>
  <div class="grid grid-cols-1 md:grid-cols-2 md:space-x-10 mt-5 md:mt-5">
    <div>
      <!-- left side -->
      <div
        class="flex items-baseline justify-center md:justify-start flex-wrap"
      >
        <p class="text-sm text-neutral-600 mr-2 w-12/12">Show income per</p>
        <div class="flex justify-center items-center">
          <button
            v-for="frequencyChoice in Object.keys(FrequencyChoices)"
            :key="frequencyChoice"
            class="
              w-fit
              text-gray-700
              block
              px-4
              mx-1
              py-2
              text-sm
              hover:bg-neutral-200
              uppercase
              transition
              delay-5
              duration-100
              ease-in-out
            "
            :class="{
              'bg-secondary hover:bg-secondary cursor-default':
                displayFrequency === FrequencyChoices[frequencyChoice],
            }"
            @click="setFrequency(frequencyChoice)"
          >
            {{ frequencyChoice }}
          </button>
        </div>
      </div>
      <div class="flex ml-3 md:ml-0 justify-start items-center mt-6 space-x-4">
        <p class="text-sm w-fit">Nr. of months to simulate your earnings</p>
        <AdjustCounter v-model:value="nrMonthsDisplay" :min="1" unit="months" />
        <InfoButton>
          <p class="text-sm w-64 text-center">
            In a portugues company, you can get payed 2 extra months per year
            (for holidays and christmas). If you want to compare your remote
            salary with some local company, you should change this field to 14
            months.
          </p>
        </InfoButton>
      </div>

      <div class="flex ml-3 md:ml-0 justify-start items-center mt-6 space-x-4">
        <p class="text-sm w-fit">Adjust your contribution to social security</p>
        <AdjustCounter
          v-model:value="ssDiscountPosition"
          :min="0"
          :max="ssDiscountChoices.length - 1"
        >
          {{ ssDiscountDisplay }}
        </AdjustCounter>
        <InfoButton
          link="https://www.seg-social.pt/documents/10152/15974914/1009%20Trabalhador%20independente%20-%20novo%20regime/87b6e00c-523d-4718-8a88-942ea804c18a"
        >
          <p class="text-sm w-64 text-center">
            You can adjust your income for social security tax calculations with
            a minimum of -25% and a maximum of +25%. This will probably afect
            your retirement pension. Click to see more.
          </p>
        </InfoButton>
      </div>
      <div class="flex ml-3 md:ml-0 justify-start items-center mt-6 sm:space-x-2 md:space-x-4">
        <SwitchButton
          label="Are you within the first 12 months of starting your activity?"
          v-model:value="firstYear"
        />
        <InfoButton
          link="https://www.montepio.org/ei/pessoal/emprego-e-formacao/seguranca-social-guia-com-as-regras-para-os-trabalhadores-independentes#"
        >
          <p class="text-sm w-64 text-center">
            You are exempt from paying social security in your first 12 months
            as a freelancer in Portugal. Click to see more.
          </p>
        </InfoButton>
      </div>
      <div
        v-if="expensesNeeded > 0"
        class="flex ml-3 md:ml-0 justify-start items-center mt-6 space-x-4"
      >
        <div>
          <p class="text-sm w-fit">
            How much can you justify as professional related expenses?
          </p>
          <span class="text-xs text-neutral-500"
            >the maximum required is
            <span class="font-semibold">{{ asCurrency(expensesNeeded) }}</span>
            <span class="text-xs text-neutral-500">/year</span>
          </span>
        </div>
        <AdjustCounter
          v-model:value="expenses"
          :min="0"
          :max="grossIncome.year"
          :step="100"
          unit="€"
          :width="14"
        >
        </AdjustCounter>
        <InfoButton
          link="https://www.montepio.org/ei/pessoal/impostos/regime-simplificado-como-funciona-a-justificacao-de-despesas/"
        />
      </div>
      <div class="flex justify-center my-12">
        <Chart />
      </div>
    </div>
    <div>
      <div class="flex gap-x-2">
        <!-- right side -->
        <div
          class="
            px-5
            py-5
            border-2
            rounded-lg
            border-dashed border-green-500
            w-6/12
            text-center
          "
        >
          <span class="text-xl"> {{ netIncomeDisplay }}</span>
          <small class="text-xs text-neutral-400">
            / {{ displayFrequency }}*</small
          >

          <div class="uppercase text-xs mt-2 tracking-widest text-green-700">
            Net income
          </div>
        </div>
        <div
          class="
            px-5
            pt-5
            pb-2
            border-2
            rounded-lg
            border-dashed border-red-500
            w-6/12
            text-center
          "
        >
          {{ taxesDisplay
          }}<small class="text-xs text-neutral-400">
            / {{ displayFrequency }}*</small
          >
          <div class="uppercase text-xs mt-2 tracking-widest text-red-700">
            Taxes
          </div>
          <div
            class="
              mt-2
              flex
              justify-between
              text-neutral-500
              uppercase
              tracking-wide
            "
          >
            <small
              >{{ irsDisplay }}<span class="text-red-400"> IRS</span></small
            >
            <small>{{ ssDisplay }}<span class="text-blue-400"> SS</span></small>
          </div>
        </div>
      </div>
      <div class="mt-10">
        <Table></Table>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref, watch, computed } from "vue";
import { storeToRefs } from "pinia";

import { asCurrency } from "@/utils.js";
import { FrequencyChoices } from "@/typings";
import { useTaxesStore } from "@/stores";

import Chart from "@/components/Chart.vue";
import AdjustCounter from "@/components/AdjustCounter.vue";
import Table from "@/components/Table.vue";
import InfoButton from "@/components/InfoButton.vue";
import SwitchButton from "@/components/SwitchButton.vue";

// store
const {
  displayFrequency,
  netIncomeDisplay,
  taxesDisplay,
  ssDisplay,
  nrMonthsDisplay,
  irsDisplay,
  expensesNeeded,
  grossIncome,
  expenses,
  firstYear,
} = storeToRefs(useTaxesStore());

const store = useTaxesStore();

// frequency
const setFrequency = (frequencyChoice: string) => {
  store.setDisplayFrequency(FrequencyChoices[frequencyChoice]);
};

// ssDiscount
const ssDiscountChoices = [
  -0.25, -0.2, -0.15, -0.1, -0.05, 0, +0.05, +0.1, +0.15, +0.2, +0.25,
];
const ssDiscountPosition = ref(5);
watch(
  () => ssDiscountPosition.value,
  (newPosition) => {
    store.setSsDiscount(ssDiscountChoices[newPosition]);
  }
);
const ssDiscountDisplay = computed(() => {
  return `${store.ssDiscount > 0 ? "+" : ""}${store.ssDiscount * 100}%`;
});
</script>