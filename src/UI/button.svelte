<script lang="ts">
	import Icon from './icon.svelte';

	export let loading = false;
	export let disabled = false;
</script>

<button class="btn block relative" on:click disabled={disabled || loading}>
	{#if loading}
		<div class="spinner-wrapper">
			<Icon name="spinner" />
		</div>
	{/if}
	<span class:opacity-0={loading}>
		<slot />
	</span>
</button>

<style lang="postcss">
	.spinner-wrapper {
		display: grid;
		place-items: center;
		position: absolute;
		top: 50%;
		left: 50%;
		translate: -50% -50%;
	}
</style>
