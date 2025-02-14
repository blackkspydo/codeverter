<script lang="ts">
	import Copy from '$components/copy.svelte';
	import Modal from '$components/modal.svelte';
	import Output from '$components/output.svelte';
	import { objectKeys } from '$helpers/object';
	import { fetchStream } from '$helpers/stream';
	import { queryOptions } from '$lib/query';
	import { key } from '$stores/key';
	import Button from '$UI/button.svelte';
	import Select from '$UI/select.svelte';

	let selected = objectKeys(queryOptions)[0];
	$: lang = queryOptions[selected].lang;

	enum ErrorCode {
		NoKey = 'Set your API key',
		Generic = 'Failed to contact OpenAI :('
	}
	let error: ErrorCode | null = null;
	let loading = false;

	let input = '';
	let output = '';
	let settingsOpen = false;

	async function search() {
		if (loading || !input) return;
		output = '';
		error = null;
		loading = true;

		try {
			const response = await fetch('/api/generate', {
				method: 'POST',
				body: JSON.stringify({ input, type: selected, key: $key }),
				headers: {
					'content-type': 'application/json'
				}
			});

			if (!response.ok) throw new Error('Network response was not ok');

			await fetchStream(response, (chunk) => {
				output += chunk;
			});
		} catch (err) {
			error = $key ? ErrorCode.Generic : ErrorCode.NoKey;
		}

		loading = false;
	}

	$: if ($key) error = null;
</script>

<svelte:head>
	<title>Codeverter</title>
	<meta name="description" content="Convert code to your programming language of choice" />
</svelte:head>

<main class="relative flex min-h-screen flex-col justify-between gap-4 overflow-hidden pb-8">
	<div class="bg" />
	<div class="mx-auto w-full max-w-7xl px-2 lg:px-4">
		<h1 class="mx-auto mt-16 max-w-5xl text-center text-3xl font-bold lg:text-5xl lg:leading-tight">
			Convert <span class="gradient-text">code</span> to your programming
			<span class="gradient-text">language</span> of choice
		</h1>

		<div class="mt-8 grid w-full gap-4 lg:grid-cols-2">
			<div class="flex h-[20rem] flex-col lg:h-[40rem]">
				<label for="input" class="font-semibold">Input</label>
				<textarea
					bind:value={input}
					name="input"
					class="textarea mt-2 w-full grow overflow-auto"
					placeholder="Type here..."
				/>
			</div>
			<div class="flex h-[20rem] flex-col lg:h-[40rem]">
				<div class="flex justify-between items-center">
					<label for="output" class="font-semibold">Output</label>
					{#if output}
						<Copy value={output} />
					{/if}
				</div>

				<Output value={output} {lang} />
			</div>
		</div>

		<div class="mt-8 flex items-center justify-center gap-4">
			<Button {loading} disabled={!input.trim()} on:click={search}>Convert</Button>
			<span>to</span>
			<Select bind:value={selected} icon={queryOptions[selected].icon}>
				{#each objectKeys(queryOptions) as key}
					<option value={key}>{queryOptions[key].label}</option>
				{/each}
			</Select>
		</div>

		{#if error === ErrorCode.Generic}
			<p class="mt-4 text-center text-red-500">{error}</p>
		{/if}
		{#if error === ErrorCode.NoKey}
			<button
				class="mt-4 mx-auto block text-red-500 underline hover:text-red-400"
				on:click={() => (settingsOpen = true)}
			>
				Set your API key
			</button>
		{/if}
	</div>

	<footer class="text-center">
		<p class="mt-8">
			Made by <a
				class="text-orange-300 underline hover:text-orange-200"
				href="https://www.thomasglopes.com/"
				target="_blank"
			>
				Thomas G. Lopes
			</a>
		</p>
		<p class="mt-1 text-sm text-gray-500">Warning: Code conversions may not be accurate.</p>
		<p class="text-sm text-gray-500">
			<button class="underline hover:text-gray-400" on:click={() => (settingsOpen = true)}>
				Manage API key
			</button>
			-
			<a
				href="https://github.com/TGlide/codeverter"
				target="_blank"
				class="underline hover:text-gray-400"
			>
				Source
			</a>
		</p>
	</footer>
</main>

<Modal bind:open={settingsOpen} title="Settings">
	<div class="flex flex-col gap-2">
		<label class="font-semibold" for="api-key">API key: </label>
		<input class="input px-2 py-2" type="password" id="api-key" bind:value={$key} />
	</div>

	<p class="mt-4 text-sm text-gray-300">
		Get your free API key <a
			class="underline hover:opacity-75"
			href="https://platform.openai.com/account/api-keys"
			target="_blank">here</a
		>
	</p>
</Modal>

<style lang="postcss">
	.gradient-text {
		background: linear-gradient(90deg, theme('colors.orange.400'), theme('colors.red.400'));
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
	}

	.bg {
		background: radial-gradient(
			farthest-side,
			theme('colors.orange.400/0.25') 0%,
			theme('colors.red.400/0') 100%
		);
		position: absolute;
		width: 1200px;
		height: 800px;

		left: 50%;
		translate: -50% -50px;
		z-index: -1;
	}

	:global(.shiki) {
		background-color: transparent !important;
		white-space: pre-wrap;
	}
</style>
