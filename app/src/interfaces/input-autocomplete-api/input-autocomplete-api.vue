<template>
	<v-notice type="warning" v-if="!url || !resultsPath || !valuePath">
		{{ $t('one_or_more_options_are_missing') }}
	</v-notice>
	<div v-else>
		<v-menu attached :disabled="disabled">
			<template #activator="{ activate, deactivate }">
				<v-input
					:placeholder="placeholder"
					:disabled="disabled"
					:class="font"
					:value="value"
					@input="onInput"
					@focus="activate"
					@blur="deactivate"
				>
					<template v-if="iconLeft" #prepend><v-icon :name="iconLeft" /></template>
					<template v-if="iconRight" #append><v-icon :name="iconRight" /></template>
				</v-input>
			</template>

			<v-list v-if="results.length > 0">
				<v-list-item v-for="result of results" :key="result" @click="() => emitValue(result)">
					<v-list-item-content>{{ result }}</v-list-item-content>
				</v-list-item>
			</v-list>
		</v-menu>
	</div>
</template>

<script lang="ts">
import { defineComponent, ref, PropType } from '@vue/composition-api';
import axios from 'axios';
import { throttle, get, debounce } from 'lodash';
import { render } from 'micromustache';

export default defineComponent({
	props: {
		value: {
			type: [String, Number],
			default: null,
		},
		url: {
			type: String,
			default: null,
		},
		resultsPath: {
			type: String,
			default: null,
		},
		valuePath: {
			type: String,
			default: null,
		},
		trigger: {
			type: String as PropType<'debounce' | 'throttle'>,
			default: 'throttle',
		},
		rate: {
			type: [Number, String],
			default: 500,
		},
		placeholder: {
			type: String,
			default: null,
		},
		iconLeft: {
			type: String,
			default: null,
		},
		iconRight: {
			type: String,
			default: null,
		},
		font: {
			type: String as PropType<'sans-serif' | 'serif' | 'monospace'>,
			default: 'sans-serif',
		},
		disabled: {
			type: Boolean,
			default: false,
		},
	},
	setup(props, { emit }) {
		const results = ref<string[]>([]);

		const fetchResultsRaw = async (value: string | null) => {
			if (!value) {
				results.value = [];
				return;
			}

			const url = render(props.url, { value });

			try {
				const result = await axios.get(url);
				const resultsArray = get(result.data, props.resultsPath);

				if (Array.isArray(resultsArray) === false) {
					console.warn(`Expected results type of array, "${typeof resultsArray}" recieved`);
					return;
				} else {
					results.value = resultsArray
						.map((result: Record<string, unknown>) => get(result, props.valuePath))
						.filter((val: unknown) => val);
				}
			} catch (err) {
				console.warn(err);
			}
		};

		const fetchResults =
			props.trigger === 'debounce'
				? debounce(fetchResultsRaw, Number(props.rate))
				: throttle(fetchResultsRaw, Number(props.rate));

		return { results, onInput, emitValue };

		function onInput(value: string) {
			emitValue(value);
			fetchResults(value);
		}

		function emitValue(value: string) {
			emit('input', value);
		}
	},
});
</script>

<style lang="scss" scoped>
.v-input {
	&.monospace {
		--v-input-font-family: var(--family-monospace);
	}

	&.serif {
		--v-input-font-family: var(--family-serif);
	}

	&.sans-serif {
		--v-input-font-family: var(--family-sans-serif);
	}
}
</style>
