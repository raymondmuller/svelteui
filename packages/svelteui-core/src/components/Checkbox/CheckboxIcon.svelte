<script lang="ts">
	import type { CheckboxProps as $$CheckboxProps } from './Checkbox.styles';

	interface $$Props extends $$CheckboxProps {}

	/** Used for custom classes to be applied to the checkbox icon e.g. Tailwind classes */
	export let className: $$Props['className'] = '';
	/** Sets the checkbox to indetermined state, overrides the checked state */
	export let indeterminate: $$Props['indeterminate'] = false;
	export { className as class };
</script>

{#if indeterminate}
	<svg class={className} xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 32 6">
		<rect width="32" height="6" fill="currentColor" rx="3" />
	</svg>
{:else}
	<svg class={className} viewBox="0 0 10 7" fill="none" xmlns="http://www.w3.org/2000/svg">
		<path
			d="M4 4.586L1.707 2.293A1 1 0 1 0 .293 3.707l3 3a.997.997 0 0 0 1.414 0l5-5A1 1 0 1 0 8.293.293L4 4.586z"
			fill="currentColor"
			fill-rule="evenodd"
			clip-rule="evenodd"
		/>
	</svg>
{/if}
