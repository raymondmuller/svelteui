<script lang="ts">
	import { cssvariable } from '$lib/actions/use-css-variable/use-css-variable';

	export let cssVariables = { display: 'block', 'background-color': 'yellow' };
</script>

<div id="css-variable" use:cssvariable={cssVariables} />
