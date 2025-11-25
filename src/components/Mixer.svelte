<script>
	import ModuleContainer from "./ModuleContainer.svelte"
	import { getDestination } from "tone"

	export let synthChannels = []
	export let drumChannels = []

	let mainVolume = 0 // dB

	function handleMixerChange(channel) {
		if (!channel || !channel.channel) return
		// Update volume and pan
		if (channel.volume !== undefined) {
			channel.channel.volume.value = channel.volume
		}
		if (channel.pan !== undefined) {
			channel.channel.pan.value = channel.pan
		}
		// Apply solo state to Tone.Channel (Tone will auto-manage muting of non-solo channels)
		if (channel.isSoloed !== undefined) {
			try {
				channel.channel.solo = channel.isSoloed
				// Optional: if soloing a muted channel, unmute it for clarity
				if (channel.isSoloed && channel.isMuted) {
					channel.isMuted = false
					channel.channel.mute = false
				}
			} catch {}
		}
		// Mute/unmute - handle channel and send effects
		if (channel.isMuted !== undefined) {
			channel.channel.mute = channel.isMuted
			// When muted, silence the send effects; when unmuted, restore them
			if (channel.sendGain && channel.reverbSend !== undefined) {
				channel.sendGain.gain.value = channel.isMuted ? 0 : channel.reverbSend
			}
			if (channel.delayGain && channel.delaySend !== undefined) {
				channel.delayGain.gain.rampTo(
					channel.isMuted ? 0 : channel.delaySend,
					0.01
				)
			}
			if (channel.bitCrusherGain && channel.bitCrusherSend !== undefined) {
				channel.bitCrusherGain.gain.value = channel.isMuted
					? 0
					: channel.bitCrusherSend
			}
		} else {
			// Normal operation when not handling mute state
			if (channel.sendGain && channel.reverbSend !== undefined) {
				channel.sendGain.gain.value = channel.reverbSend
			}
			if (channel.delayGain && channel.delaySend !== undefined) {
				channel.delayGain.gain.rampTo(channel.delaySend, 0.01)
			}
			if (channel.bitCrusherGain && channel.bitCrusherSend !== undefined) {
				channel.bitCrusherGain.gain.value = channel.bitCrusherSend
			}
		}

		// Sync reverbSend back to config
		if (channel.configRef && channel.reverbSend !== undefined) {
			channel.configRef.reverbSend = channel.reverbSend
		}
		if (channel.drumRef && channel.reverbSend !== undefined) {
			channel.drumRef.reverbSend = channel.reverbSend
		}
		// Sync delaySend back to config
		if (channel.configRef && channel.delaySend !== undefined) {
			channel.configRef.delaySend = channel.delaySend
		}
		if (channel.drumRef && channel.delaySend !== undefined) {
			channel.drumRef.delaySend = channel.delaySend
		}
		// Sync bitCrusherSend back to config
		if (channel.configRef && channel.bitCrusherSend !== undefined) {
			channel.configRef.bitCrusherSend = channel.bitCrusherSend
		}
		if (channel.drumRef && channel.bitCrusherSend !== undefined) {
			channel.drumRef.bitCrusherSend = channel.bitCrusherSend
		}
		// Sync solo state back to config
		if (channel.configRef && channel.isSoloed !== undefined) {
			channel.configRef.isSoloed = channel.isSoloed
		}
		if (channel.drumRef && channel.isSoloed !== undefined) {
			channel.drumRef.isSoloed = channel.isSoloed
		}
	}

	// Reactive updates for all channels
	$: synthChannels.forEach((ch) => handleMixerChange(ch))
	$: drumChannels.forEach((ch) => handleMixerChange(ch))

	// Update main output volume
	$: try {
		getDestination().volume.value = mainVolume
	} catch (e) {
		// Destination not ready yet
	}
</script>

<div class="mixer-grid">
	<!-- Drum Channels -->
	{#each drumChannels as channel}
		<ModuleContainer
			type="drum"
			isMuted={channel.isMuted}
			isSoloed={channel.isSoloed}
		>
			<div class="mixer-channel">
				<h3>{channel.name}</h3>

				<label>
					Rev <br />
					{Math.round((channel.reverbSend || 0) * 100)}%
					<input
						type="range"
						min="0"
						max="1"
						step="0.01"
						bind:value={channel.reverbSend}
					/>
				</label>

				<label>
					Delay <br />
					{Math.round((channel.delaySend || 0) * 100)}%
					<input
						type="range"
						min="0"
						max="1"
						step="0.01"
						bind:value={channel.delaySend}
					/>
				</label>
				<label>
					Pan: <br />
					{channel.pan.toFixed(2)}
					<input
						type="range"
						min="-1"
						max="1"
						step="0.01"
						bind:value={channel.pan}
					/>
				</label>

				<label>
					CrushSend <br />
					{Math.round((channel.bitCrusherSend || 0) * 100)}%
					<input
						type="range"
						min="0"
						max="1"
						step="0.01"
						bind:value={channel.bitCrusherSend}
					/>
				</label>
				<label>
					{channel.volume.toFixed(1)}dB
					<input
						type="range"
						orient="vertical"
						min="-48"
						max="6"
						step="0.5"
						bind:value={channel.volume}
					/>
				</label>
				<div class="mute-solo-controls">
					<button
						class="solo-button"
						class:active={channel.isSoloed}
						on:click={() => (channel.isSoloed = !channel.isSoloed)}
					>
						S
					</button>
					<button
						class="mute-button"
						class:active={channel.isMuted}
						on:click={() => (channel.isMuted = !channel.isMuted)}
					>
						M
					</button>
				</div>
			</div>
		</ModuleContainer>
	{/each}
	<!-- Synth Channels -->
	{#each synthChannels as channel}
		<ModuleContainer isMuted={channel.isMuted} isSoloed={channel.isSoloed}>
			<div class="mixer-channel" class:muted={channel.isMuted}>
				<h3>{channel.name}</h3>

				<!-- <label>
					<input type="checkbox" bind:checked={channel.isMuted} />
					Mute
				</label> -->
				<label>
					Rev <br />
					{Math.round((channel.reverbSend || 0) * 100)}%
					<input
						type="range"
						min="0"
						max="1"
						step="0.01"
						bind:value={channel.reverbSend}
					/>
				</label>
				<label>
					Delay <br />
					{Math.round((channel.delaySend || 0) * 100)}%
					<input
						type="range"
						min="0"
						max="1"
						step="0.01"
						bind:value={channel.delaySend}
					/>
				</label>
				<label>
					Pan <br />
					{channel.pan.toFixed(2)}
					<input
						type="range"
						min="-1"
						max="1"
						step="0.01"
						bind:value={channel.pan}
					/>
				</label>
				<label>
					CrushSend <br />
					{Math.round((channel.bitCrusherSend || 0) * 100)}%
					<input
						type="range"
						min="0"
						max="1"
						step="0.01"
						bind:value={channel.bitCrusherSend}
					/>
				</label>
				<label>
					{channel.volume.toFixed(1)}dB
					<input
						type="range"
						orient="vertical"
						min="-48"
						max="6"
						step="0.5"
						bind:value={channel.volume}
					/>
				</label>
				<div class="mute-solo-controls">
					<button
						class="solo-button"
						class:active={channel.isSoloed}
						on:click={() => (channel.isSoloed = !channel.isSoloed)}
					>
						S
					</button>
					<button
						class="mute-button"
						class:active={channel.isMuted}
						on:click={() => (channel.isMuted = !channel.isMuted)}
					>
						M
					</button>
				</div>
			</div>
		</ModuleContainer>
	{/each}
	<!-- main out -->
	<ModuleContainer type="main-out">
		<div class="mixer-channel main-out">
			<h3>MAIN</h3>

			<label>
				{mainVolume.toFixed(1)}dB
				<input
					type="range"
					orient="vertical"
					min="-48"
					max="6"
					step="0.5"
					bind:value={mainVolume}
				/>
			</label>
		</div>
	</ModuleContainer>
</div>

<style>
	.mixer-grid {
		flex: 1;
		display: flex;
		gap: var(--gap-sm);
		flex-wrap: wrap;
		margin: var(--gap-md) auto 0;
		justify-content: space-around;
		max-width: fit-content;
	}

	.mixer-channel {
		/* min-width: 120px; */
		width: 56px;
		display: flex;
		flex-direction: column;
		gap: var(--gap-md);
	}

	.mixer-channel.muted {
		opacity: 0.7;
	}

	/* .mixer-channel.main-out removed (empty ruleset) */

	.mixer-channel h3 {
		font-size: 1rem;
		text-align: center;
		padding-bottom: var(--gap-sm);
		border-bottom: 1px solid var(--border-secondary);
	}

	.mute-solo-controls {
		display: flex;
		gap: var(--gap-sm);
	}

	.mute-button,
	.solo-button {
		/* width: 100%; */
		padding: 0.5rem;
		font-size: 1rem;
		background: var(--bg-secondary);
		border: 1px solid var(--border-secondary);
		cursor: pointer;
		transition: all 0.1s;
	}

	.mute-button:hover,
	.solo-button:hover {
		border-color: var(--accent-synth);
	}

	.mute-button.active {
		background: var(--bg-primary);
		border-color: var(--border-muted);
		color: var(--border-muted);
		/* opacity: 0.6; */
	}
	.solo-button.active {
		background: var(--border-soloed);
		border-color: var(--border-soloed);
		color: var(--bg-primary);
	}

	.mixer-channel label {
		display: flex;
		flex-direction: column;
		gap: var(--gap-sm);
	}

	.mixer-channel input[type="range"] {
		width: 100%;
	}

	.mixer-channel input[type="range"][orient="vertical"] {
		writing-mode: bt-lr;
		-webkit-appearance: slider-vertical;
		appearance: slider-vertical;
		height: 100px;
		width: 20px;
		align-self: center;
		background: var(--bg-secondary);
		border: 1px solid var(--border-secondary);
	}

	.mixer-channel.main-out input[type="range"][orient="vertical"] {
		height: 250px;
	}

	.mixer-channel input[type="range"][orient="vertical"]::-webkit-slider-thumb {
		width: 28px;
		height: 10px;
	}

	.mixer-channel input[type="range"][orient="vertical"]::-moz-range-thumb {
		width: 28px;
		height: 10px;
	}
</style>
