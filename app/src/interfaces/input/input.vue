<template>
	<v-input
		:autofocus="autofocus"
		:value="value"
		:nullable="!clear"
		:placeholder="placeholder"
		:disabled="disabled"
		:trim="trim"
		:type="inputType"
		:class="font"
		:db-safe="dbSafe"
		:slug="slug"
		:min="min"
		:max="max"
		:step="step"
		@input="$listeners.input"
	>
		<template v-if="iconLeft" #prepend><v-icon :name="iconLeft" /></template>
		<template #append>
			<span
				v-if="percentageRemaining && percentageRemaining <= 20"
				class="remaining"
				:class="{
					warning: percentageRemaining < 10,
					danger: percentageRemaining < 5,
				}"
			>
				{{ charsRemaining }}
			</span>
			<v-icon
				:class="{ hide: percentageRemaining !== false && percentageRemaining <= 20 }"
				v-if="iconRight"
				:name="iconRight"
			/>
		</template>
	</v-input>
</template>

<script lang="ts">
import { defineComponent, PropType, computed } from '@vue/composition-api';

export default defineComponent({
	props: {
		value: {
			type: [String, Number],
			default: null,
		},
		type: {
			type: String,
			default: null,
		},
		clear: {
			type: Boolean,
			default: false,
		},
		disabled: {
			type: Boolean,
			default: false,
		},
		placeholder: {
			type: String,
			default: null,
		},
		masked: {
			type: Boolean,
			default: false,
		},
		iconLeft: {
			type: String,
			default: null,
		},
		iconRight: {
			type: String,
			default: null,
		},
		trim: {
			type: Boolean,
			default: true,
		},
		font: {
			type: String as PropType<'sans-serif' | 'serif' | 'monospace'>,
			default: 'sans-serif',
		},
		length: {
			type: Number,
			default: null,
		},
		dbSafe: {
			type: Boolean,
			default: false,
		},
		autofocus: {
			type: Boolean,
			default: false,
		},

		slug: {
			type: Boolean,
			default: false,
		},
		min: {
			type: Number,
			default: null,
		},
		max: {
			type: Number,
			default: null,
		},
		step: {
			type: Number,
			default: 1,
		},
	},
	setup(props) {
		const charsRemaining = computed(() => {
			if (typeof props.value === 'number') return null;

			if (!props.length) return null;
			if (!props.value) return null;
			return +props.length - props.value.length;
		});

		const percentageRemaining = computed(() => {
			if (typeof props.value === 'number') return null;

			if (!props.length) return false;
			if (!props.value) return false;
			return 100 - (props.value.length / +props.length) * 100;
		});

		const inputType = computed(() => {
			if (props.masked) return 'new-password';
			if (['bigInteger', 'integer', 'float', 'decimal'].includes(props.type)) return 'number';
			return 'text';
		});

		return { inputType, charsRemaining, percentageRemaining };
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

.remaining {
	display: none;
	width: 24px;
	color: var(--foreground-subdued);
	font-weight: 600;
	text-align: right;
	vertical-align: middle;
	font-feature-settings: 'tnum';
}

.v-input:focus-within .remaining {
	display: block;
}

.v-input:focus-within .hide {
	display: none;
}

.warning {
	color: var(--warning);
}

.danger {
	color: var(--danger);
}
</style>
