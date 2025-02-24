<script lang="ts">
	import useStyles from './NumberInput.styles';
	import { createEventDispatcher } from 'svelte';
	import { TextInput } from '../TextInput';
	import { defaultFormatter, defaultParser } from './NumberInput.styles';
	import type { NumberInputProps as $$NumberInputProps } from './NumberInput.styles';

	interface $$Props extends $$NumberInputProps {}

	export let use: $$Props['use'] = [],
		element: $$Props['element'] = undefined,
		className: $$Props['className'] = '',
		override: $$Props['override'] = {},
		overrideControls: $$Props['override'] = {},
		root: $$Props['root'] = 'input',
		placeholder: $$Props['placeholder'] = undefined,
		icon: $$Props['icon'] = null,
		iconWidth: $$Props['iconWidth'] = 36,
		iconProps: $$Props['iconProps'] = { size: 20, color: 'currentColor' },
		wrapperProps: $$Props['wrapperProps'] = {},
		required: $$Props['required'] = false,
		radius: $$Props['radius'] = 'sm',
		variant: $$Props['variant'] = 'default',
		disabled: $$Props['disabled'] = false,
		size: $$Props['size'] = 'sm',
		value: $$Props['value'] = undefined,
		defaultValue: $$Props['defaultValue'] = undefined,
		decimalSeparator: $$Props['decimalSeparator'] = '.',
		min: $$Props['min'] = -Infinity,
		max: $$Props['max'] = Infinity,
		step: $$Props['step'] = 1,
		stepHoldDelay: $$Props['stepHoldDelay'] = 250,
		stepHoldInterval: $$Props['stepHoldInterval'] = 150,
		hideControls: $$Props['hideControls'] = false,
		precision: $$Props['precision'] = 0,
		noClampOnBlur: $$Props['noClampOnBlur'] = false,
		formatter: $$Props['formatter'] = defaultFormatter,
		parser: $$Props['parser'] = defaultParser;
	export { className as class };

	/** Function that allows incrementing the value outside the component, useful for external controls */
	export function increment() {
		onStep(true, false);
	}
	/** Function that allows decrementing the value outside the component, useful for external controls */
	export function decrement() {
		onStep(false, false);
	}

	const dispatch = createEventDispatcher();

	let isKeyDown = false;
	let stepCount = 0;
	let holdTimeout = null;
	let holdDelayTimeout = null;

	function formatNumber(val: string | number = ''): string {
		let parsedStr = typeof val === 'number' ? String(val) : val;

		// replaces all periods '.' for the given decimal separator
		if (decimalSeparator) {
			parsedStr = parsedStr.replace(/\./g, decimalSeparator);
		}

		return formatter(parsedStr);
	}

	function parseNumber(val: string): string {
		let number = val;

		// replaces all decimal separators for a period '.'
		if (decimalSeparator) {
			number = number.replace(new RegExp(`\\${decimalSeparator}`, 'g'), '.');
		}

		return parser(number);
	}

	function onInput() {
		if (this.value === '' || this.value === '-') {
			value = undefined;
		} else {
			const parsedNumber = parseNumber(this.value);
			if (parsedNumber === undefined || Number.isNaN(parseNumber)) return;
			value = parseFloat(parsedNumber);
		}
		dispatch('change', value);
	}

	function stepInterval(up) {
		const interval =
			typeof stepHoldInterval === 'number' ? stepHoldInterval : stepHoldInterval(stepCount);

		holdTimeout = setTimeout(() => {
			onStep(up, true, false);
		}, interval);
	}

	function onStep(up, hold = true, first = true) {
		const _value = value === undefined ? 0 : value;
		const tmpValue = up ? _value + step : _value - step;

		const clamped = _clamp(tmpValue);
		value = parseFloat(clamped.toFixed(precision));
		stepCount += 1;

		// dispatches change events so that listeners can get
		// the original value, not formatted
		dispatch('change', value);

		if (!hold) return;

		if (first) {
			holdDelayTimeout = setTimeout(() => stepInterval(up), stepHoldDelay);
		} else {
			stepInterval(up);
		}
	}

	function onStepDone() {
		if (holdDelayTimeout) clearTimeout(holdDelayTimeout);
		if (holdTimeout) clearTimeout(holdTimeout);
		holdDelayTimeout = null;
		holdTimeout = null;
		stepCount = 0;
	}

	function onKeyDown(event) {
		if (event.key !== 'ArrowUp' && event.key !== 'ArrowDown') return;
		if (isKeyDown) return;

		isKeyDown = true;
		onStep(event.key === 'ArrowUp');
	}

	function onKeyUp(event) {
		if (event.key !== 'ArrowUp' && event.key !== 'ArrowDown') return;
		isKeyDown = false;
		onStepDone();
	}

	function onBlur() {
		if (noClampOnBlur) return;

		const clamped = _clamp(value);
		value = parseFloat(clamped.toFixed(precision));
		dispatch('change', value);

		(element as HTMLInputElement).value = formatNumber(value);
	}

	function _clamp(value) {
		return Math.min(Math.max(value, min), max);
	}

	function _valueC(val: number): number | undefined {
		if (val === undefined && typeof defaultValue === 'number') {
			return defaultValue;
		}
		if (typeof val !== 'number') {
			return undefined;
		}
		return val;
	}

	$: value = _valueC(value);
	$: showControls = !hideControls && variant !== 'unstyled' && !disabled;

	$: ({ cx, classes, getStyles } = useStyles({ radius, size }));
</script>

<!--
@component
**UNSTABLE**: new API, yet to be vetted.

Number input component that allows inputting numbers and incremeting/decrementing them, as well as set steps, minimum and maximum
values and add custom parsers and formatters.

@see https://svelteui.org/core/number-input
@example
    ```svelte
    <NumberInput defaultValue={2} />
	<NumberInput max={10} min={0} step={0.5} precision={1} />
	<NumberInput defaultValue={0} step={0.2} precision={2} decimalSeparator="," />
	<NumberInput label='Your age' required defaultValue={0} />
    ```
-->

<TextInput
	{use}
	{root}
	{icon}
	{iconWidth}
	{iconProps}
	{wrapperProps}
	{required}
	{size}
	{radius}
	{variant}
	{disabled}
	{placeholder}
	class={className}
	override={{ '& .rightSection': { width: 'auto' }, ...override }}
	value={formatNumber(value)}
	showRightSection={showControls}
	{...$$restProps}
	bind:element
	on:input={onInput}
	on:keyup={onKeyUp}
	on:keydown={onKeyDown}
	on:blur={onBlur}
>
	<div
		slot="rightSection"
		class={cx(className, classes.controls, getStyles({ css: overrideControls }))}
	>
		{#if showControls}
			<button
				class="control control-up"
				type="button"
				tabIndex={-1}
				aria-hidden
				disabled={value >= max}
				on:mousedown={() => onStep(true)}
				on:mouseup={onStepDone}
				on:mouseleave={onStepDone}
			/>
			<button
				class=" control control-down"
				type="button"
				tabIndex={-1}
				aria-hidden
				disabled={value <= min}
				on:mousedown={() => onStep(false)}
				on:mouseup={onStepDone}
				on:mouseleave={onStepDone}
			/>
		{/if}
	</div>
</TextInput>
