<script>
	import { bundle, get_full_filename, handle_change, module_editor, selected } from '$lib/state';
	import CodeMirror from '../CodeMirror.svelte';
	import Message from '../Message.svelte';

	/** @type {import('$lib/types').StartOrEnd | null} */
	export let errorLoc = null;
	// export let theme;

	export function focus() {
		$module_editor?.focus();
	}

	/** @type {import('$lib/types').Error | null | undefined} */
	let error = null;

	/** @type {import('$lib/types').Warning[]} */
	let warnings = [];

	$: filename = $selected?.name + '.' + $selected?.type;

	let error_file = '';

	$: if ($bundle) {
		error = $bundle?.error;
		warnings = $bundle?.warnings ?? [];

		if (error || warnings.length > 1) {
			error_file = error?.filename ?? warnings[0].filename;
		}
	}

	$: diagnostics = /** @type {import('@codemirror/lint').Diagnostics[]} */ (
		$selected && error_file === get_full_filename($selected)
			? [
					...(error
						? [
								{
									from: error.start.character,
									to: error.end.character,
									severity: 'error',
									message: error.message
								}
						  ]
						: []),
					...warnings.map((warning) => ({
						from: warning.start.character,
						to: warning.end.character,
						severity: 'warning',
						message: warning.message
					}))
			  ]
			: []
	);
</script>

<div class="editor-wrapper">
	<div class="editor notranslate" translate="no">
		<CodeMirror
			bind:this={$module_editor}
			{errorLoc}
			on:change
			on:change={handle_change}
			{diagnostics}
		/>
	</div>

	<div class="info">
		{#if error}
			<Message kind="error" details={error} {filename} />
		{:else if warnings.length > 0}
			{#each warnings as warning}
				<Message kind="warning" details={warning} {filename} />
			{/each}
		{/if}
	</div>
</div>

<style>
	.editor-wrapper {
		z-index: 5;
		background: var(--sk-back-3);
		display: flex;
		flex-direction: column;
	}

	.editor {
		height: 0;
		flex: 1 1 auto;
	}

	.info {
		background-color: var(--sk-theme-2);
		max-height: 50%;
		overflow: auto;
	}

	:global(.columns) .editor-wrapper {
		/* make it easier to interact with scrollbar */
		padding-right: 8px;
		height: auto;
		height: 100%;
	}
</style>
