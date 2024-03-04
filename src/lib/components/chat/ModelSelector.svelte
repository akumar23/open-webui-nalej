<script lang="ts">
	import { setDefaultModels } from '$lib/apis/configs';
	import { models, settings, user } from '$lib/stores';
	import { onMount, tick } from 'svelte';
	import { toast } from 'svelte-sonner';

	export let selectedModels = [''];
	export let disabled = false;

	const saveDefaultModel = async () => {
		const hasEmptyModel = selectedModels.filter((it) => it === '');
		if (hasEmptyModel.length) {
			toast.error('Choose a model before saving...');
			return;
		}
		settings.set({ ...$settings, models: selectedModels });
		localStorage.setItem('settings', JSON.stringify($settings));

		if ($user.role === 'admin') {
			console.log('setting default models globally');
			await setDefaultModels(localStorage.token, selectedModels.join(','));
		}
		toast.success('Default model updated');
	};

	$: if (selectedModels.length > 0 && $models.length > 0) {
		selectedModels = selectedModels.map((model) =>
			$models.map((m) => m.id).includes(model) ? model : ''
		);
	}
</script>

<div class="flex flex-col my-2">
	{#each selectedModels as selectedModel, selectedModelIdx}
		<div class="flex">
			<select
				id="models"
				class="outline-none bg-transparent text-lg font-semibold rounded-lg block w-full placeholder-gray-400"
				bind:value={selectedModel}
				{disabled}
			>
				<option class=" text-gray-700" value="" selected disabled>Select a model</option>

				{#each $models as model}
					{#if model.name === 'hr'}
						<hr />
					{:else}
						<option value={model.id} class="text-gray-700 text-lg"
							>{model.name +
								`${model.size ? ` (${(model.size / 1024 ** 3).toFixed(1)}GB)` : ''}`}</option
						>
					{/if}
				{/each}
			</select>

			{#if selectedModelIdx === 0}
				<button
					class="  self-center {selectedModelIdx === 0
						? 'mr-3'
						: 'mr-7'} disabled:text-gray-600 disabled:hover:text-gray-600"
					{disabled}
					on:click={() => {
						selectedModels = [...selectedModels, ''];
					}}
				>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						fill="none"
						viewBox="0 0 24 24"
						stroke-width="1.5"
						stroke="currentColor"
						class="w-4 h-4"
					>
						<path stroke-linecap="round" stroke-linejoin="round" d="M12 6v12m6-6H6" />
					</svg>
				</button>
			{:else}
				<button
					class="  self-center disabled:text-gray-600 disabled:hover:text-gray-600 {selectedModelIdx ===
					0
						? 'mr-3'
						: 'mr-7'}"
					{disabled}
					on:click={() => {
						selectedModels.splice(selectedModelIdx, 1);
						selectedModels = selectedModels;
					}}
				>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						fill="none"
						viewBox="0 0 24 24"
						stroke-width="1.5"
						stroke="currentColor"
						class="w-4 h-4"
					>
						<path stroke-linecap="round" stroke-linejoin="round" d="M19.5 12h-15" />
					</svg>
				</button>
			{/if}
		</div>
	{/each}
</div>

<div class="text-left mt-1.5 text-xs text-gray-500">
	<button on:click={saveDefaultModel}> Set as default</button>
</div>
