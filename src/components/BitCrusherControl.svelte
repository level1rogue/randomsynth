<script>
	import { createEventDispatcher } from "svelte"

	export let bitCrusherConfig

	const dispatch = createEventDispatcher()

	$: {
		console.log(
			"bitCrusher config changed in BitCrusherControl:",
			bitCrusherConfig
		)
		dispatch("change")
	}
</script>

<div class="bitcrusher-controls">
	<h3>BitCrusher</h3>
	<div class="bitcrusher-settings">
		<div class="control-row">
			<label>
				Bits: {bitCrusherConfig.bits}
				<input
					type="range"
					min="1"
					max="16"
					step="1"
					bind:value={bitCrusherConfig.bits}
				/>
			</label>
		</div>
		<div class="control-row">
			<label>
				Wet: {(bitCrusherConfig.wet * 100).toFixed(0)}%
				<input
					type="range"
					min="0"
					max="1"
					step="0.01"
					bind:value={bitCrusherConfig.wet}
				/>
			</label>
		</div>
	</div>
</div>

<style>
	.bitcrusher-controls {
		padding: var(--gap-md);
		background: var(--bg-secondary);
		border: 1px solid var(--border-primary);
		border-radius: var(--radius-md);
		min-width: 200px;
	}

	h3 {
		font-size: 1rem;
		margin: 0 0 var(--gap-md) 0;
		color: var(--text-primary);
		text-align: center;
	}

	.bitcrusher-settings {
		display: flex;
		flex-direction: column;
		gap: var(--gap-md);
	}

	.control-row {
		display: flex;
		flex-direction: column;
		gap: var(--gap-sm);
	}

	label {
		display: flex;
		flex-direction: column;
		gap: var(--gap-sm);
		font-size: 0.9rem;
	}

	input[type="range"] {
		width: 100%;
	}
</style>
