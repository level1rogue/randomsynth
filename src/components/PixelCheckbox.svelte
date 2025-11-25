<script>
	import { createEventDispatcher } from "svelte"
	export let checked = false
	export let label = ""
	export let disabled = false
	export let size = 9 // box size in px (compact)
	const dispatch = createEventDispatcher()
	let innerSize = 3
	$: innerSize = Math.max(4, Math.round(size / 2))

	function onChange(e) {
		checked = e.target.checked
		dispatch("change", { checked })
	}
</script>

<label
	class="pixel-checkbox"
	data-checked={checked}
	data-disabled={disabled}
	style={`--box-size:${size}px; --inner-size:${innerSize}px;`}
>
	<input
		type="checkbox"
		bind:checked
		{disabled}
		aria-label={label}
		on:change={onChange}
	/>
	<span class="box" aria-hidden="true">
		<span class="inner"></span>
	</span>
	{#if label}<span class="lbl">{label}</span>{/if}
</label>

<style>
	.pixel-checkbox {
		display: inline-flex;
		align-items: center;
		gap: 6px;
		font-size: 0.7rem;
		font-family: inherit;
		cursor: pointer;
		user-select: none;
		position: relative;
		margin-top: 4px;
	}
	.pixel-checkbox[data-disabled="true"] {
		opacity: 0.4;
		cursor: not-allowed;
	}
	.pixel-checkbox input {
		position: absolute;
		opacity: 0;
		width: 0;
		height: 0;
		margin: 0;
		padding: 0;
	}
	.pixel-checkbox .box {
		width: var(--box-size);
		height: var(--box-size);
		background: var(--bg-secondary);
		border: 1px solid var(--border-secondary);
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
		image-rendering: pixelated;
	}
	.pixel-checkbox[data-checked="true"] .box {
		border-color: var(--accent-synth);
	}
	.pixel-checkbox .inner {
		display: none;
		width: var(--inner-size);
		height: var(--inner-size);
		background: var(--accent-synth);
	}
	.pixel-checkbox[data-checked="true"] .inner {
		display: block;
	}
	.pixel-checkbox .lbl {
		color: var(--text-secondary);
		letter-spacing: 0.05em;
		text-transform: uppercase;
	}
</style>
