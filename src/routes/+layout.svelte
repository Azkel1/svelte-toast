<script lang="ts">
	import type { Snippet } from 'svelte';
	import ToastDrawer, { createToast, type ToastDrawerPosition } from '$lib/ToastDrawer.svelte';

	let { children }: { children: Snippet } = $props();
	let currentPosition = $state<ToastDrawerPosition>('bottom right');
	let currentText = $state<string>();

	const success = () =>
		createToast({ type: 'success', message: currentText || 'Operation successful' });
	const warning = () =>
		createToast({
			type: 'warning',
			timeout: false,
			message: currentText || 'Be patient, this action may take a while'
		});
	const error = () =>
		createToast({ type: 'error', message: currentText || 'Error processing information' });
	const info = () =>
		createToast({
			type: 'info',
			timeout: false,
			message:
				currentText ||
				'Message with a very long text including a message, some data for the user, logs, or any other text you want to display'
		});
	const question = () =>
		createToast({
			type: 'warning',
			message: currentText || 'This action is irreversible, are you sure?',
			timeout: false,
			isClosable: true,
			buttons: [
				{ text: 'yes', type: 'danger', callback: success },
				{ text: 'no', callback: error }
			]
		});
</script>

{@render children()}

<label>
	Type a message and press one of the buttons below:
	<input bind:value={currentText} />
</label>

<section>
	<button onclick={success}>Display success toast</button>
	<button onclick={warning}>Display warning toast</button>
	<button onclick={error}>Display error toast</button>
	<button onclick={info}>Display info toast</button>
	<button onclick={question}>Display question toast</button>
</section>

<fieldset>
	<label>
		Top left
		<input
			type="radio"
			name="toast-drawer-position"
			value="top left"
			bind:group={currentPosition}
		/>
	</label>
	<label>
		Top center
		<input
			type="radio"
			name="toast-drawer-position"
			value="top center"
			bind:group={currentPosition}
		/>
	</label>
	<label>
		Top right
		<input
			type="radio"
			name="toast-drawer-position"
			value="top right"
			bind:group={currentPosition}
		/>
	</label>
	<label>
		Bottom left
		<input
			type="radio"
			name="toast-drawer-position"
			value="bottom left"
			bind:group={currentPosition}
		/>
	</label>
	<label>
		Bottom center
		<input
			type="radio"
			name="toast-drawer-position"
			value="bottom center"
			bind:group={currentPosition}
		/>
	</label>
	<label>
		Bottom right
		<input
			type="radio"
			name="toast-drawer-position"
			value="bottom right"
			bind:group={currentPosition}
		/>
	</label>
</fieldset>

<ToastDrawer position={currentPosition} />

<style>
	:global(html) {
		font-family: monospace;
		font-size: 0.9rem;
	}

	:global(body) {
		margin: 0;
		min-height: 100dvh;
		display: grid;
		gap: 1em;
		place-content: center;
		place-items: center;
		text-align: center;
	}

	:global(*) {
		box-sizing: border-box;
	}

	:global(h1) {
		margin: 0;
	}

	fieldset {
		display: grid;
	}
</style>
