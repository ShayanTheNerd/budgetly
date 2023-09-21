<script setup>
	import { computed, toRefs } from 'vue';
	import { useTransition } from '@vueuse/core';
	import { useBudgetStore } from '../budgetStore.js';
	import { useFormatNumber } from '@/composables/useFormatNumber.mjs';

	const props = defineProps({
		type: {
			required: true,
			type: String,
		},
		value: {
			required: true,
			type: Number,
		},
		percentage: {
			type: Number,
			default: 0,
		},
	});

	const { formatNumber } = useFormatNumber();
	const { formatNumber: formatNumberPercent } = useFormatNumber({ style: 'percent' });

	const { value, percentage } = toRefs(props);
	const valueAnimated = useTransition(value);
	const percentageAnimated = useTransition(percentage);
	const budgetStore = useBudgetStore();
	const percentagePrefix = computed(() => {
		const totalExpenses = budgetStore.getTotal('expenses');
		return percentageAnimated.value > 100 ? '+' : totalExpenses === 0 ? '' : '~';
	});
	const percentageNumber = computed(() => {
		const percentageValue = Math.floor(percentageAnimated.value || (budgetStore.balance < 0 ? 100 : 0));
		return formatNumberPercent(Math.min(100, percentageValue));
	});
</script>

<template>
	<div
		:class="{ 'bg-emerald-500': type === 'incomes', 'bg-red-500': type === 'expenses' }"
		class="scrollbar-hidden flex h-12 items-center justify-between gap-3.5 overflow-x-auto rounded-md px-2.5 text-lg sm:w-1/2 lg:text-xl">
		<p class="w-20 shrink-0 capitalize tracking-wide">{{ type }}</p>

		<p class="text-[21px] font-medium tracking-wide">
			{{ formatNumber(valueAnimated) }}
		</p>

		<small :class="type === 'incomes' && 'invisible'" class="w-14 rounded-md bg-slate-300/40">
			{{ `${percentagePrefix}${percentageNumber}` }}
		</small>
	</div>
</template>