<!-- ⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯ JS/TS ⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯ -->
<script lang="ts" module>
	import { flip } from 'svelte/animate';
	import { quadIn, quintOut } from 'svelte/easing';
	import { fly } from 'svelte/transition';

	// Types
	export type ToastDrawerPosition =
		| 'top left'
		| 'top center'
		| 'top right'
		| 'bottom left'
		| 'bottom center'
		| 'bottom right';
	export type ToastType = 'success' | 'warning' | 'error' | 'info';
	export type ToastButton = {
		text: string;
		type?: 'default' | 'caution' | 'danger';
		callback: () => void;
	};
	export type ToastOptions = {
		type: ToastType;
		message: string;
		timeout?: number | false;
		isClosable?: boolean;
		buttons?: Array<ToastButton>;
	};
	export type Toast = ToastOptions & {
		id: symbol;
	};

	// State management
	let toasts = $state<Array<Toast>>([]);
	let isToastDrawerInTop = false;

	export function createToast(options: ToastOptions) {
		const toast: Toast = {
			id: Symbol(),
			...options
		};

		if (isToastDrawerInTop) toasts.push(toast);
		else toasts.unshift(toast);

		if (options.timeout !== false) {
			setTimeout(() => closeToast(toast.id), options.timeout ?? 3000);
		}
	}

	function closeToast(toastID: symbol) {
		toasts.splice(
			toasts.findIndex((t) => t.id === toastID),
			1
		);
	}

	function wrapButtonCallback(toast: Toast, callback: ToastButton['callback']) {
		return () => {
			callback();
			closeToast(toast.id);
		};
	}
</script>

<script lang="ts">
	interface Props {
		position?: ToastDrawerPosition;
	}

	let { position = 'bottom right' }: Props = $props();
	let toastFlyOffset = $derived.by(() => {
		isToastDrawerInTop = position.startsWith('top');
		return isToastDrawerInTop ? -200 : 200;
	});
</script>

<!-- ⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯ HTML ⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯ -->
{#snippet toast(t: Toast)}
	{t.message}

	{#if t.isClosable !== false}
		<button class="close" onclick={() => closeToast(t.id)}>&#10006;</button>
	{/if}

	{#if t.buttons?.length}
		<footer>
			{#each t.buttons ?? [] as b}
				<button class={b.type} onclick={wrapButtonCallback(t, b.callback)}>{b.text}</button>
			{/each}
		</footer>
	{/if}
{/snippet}

<div class="toast-drawer" data-position={position}>
	{#each toasts as t (t.id)}
		<output
			in:fly={{ y: toastFlyOffset, duration: 300, easing: quintOut }}
			out:fly={{ y: toastFlyOffset, duration: 600, easing: quadIn }}
			animate:flip={{ duration: 600, easing: quintOut }}
			class="toast {t.type}"
		>
			{@render toast(t)}
		</output>
	{/each}
</div>

<!-- ⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯ CSS ⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯ -->
<style>
	.toast-drawer {
		--_drawer-margin: 1rem;
		--_drawer-toast-gap: 0.5rem;
		--_drawer-max-width: 30vw;

		display: grid;
		gap: var(--_drawer-toast-gap);

		position: fixed;
		margin: var(--_drawer-margin);
		max-width: var(--_drawer-max-width);

		&[data-position^='top'] {
			top: 0;
		}

		&[data-position^='bottom'] {
			bottom: 0;
		}

		&[data-position$='left'] {
			justify-items: start;
			left: 0;
		}

		&[data-position$='center'] {
			justify-items: center;
			left: 50%;
			transform: translateX(-50%);
		}

		&[data-position$='right'] {
			justify-items: end;
			right: 0;

			> .toast {
				text-align: end;
			}
		}

		> .toast {
			--_toast-color-success: #59d469;
			--_toast-color-info: #31aff8;
			--_toast-color-warning: #ffc13c;
			--_toast-color-error: #e73d3d;
			--_toast-border-radius: 7px;
			--_toast-padding: 0.5rem;
			--_toast-gap: 0.4rem;

			background-color: color-mix(in srgb, var(--_toast-color) 50%, white);
			border: 1px solid var(--_toast-color);
			border-radius: var(--_toast-border-radius);
			text-align: start;

			display: grid;
			grid-template-columns: 1fr auto;
			gap: var(--_toast-gap);
			align-items: center;
			justify-items: inherit;

			padding: var(--_toast-padding);

			&.success {
				--_toast-color: var(--_toast-color-success);
			}

			&.warning {
				--_toast-color: var(--_toast-color-warning);
			}

			&.error {
				--_toast-color: var(--_toast-color-error);
			}

			&.info {
				--_toast-color: var(--_toast-color-info);
			}

			> button.close {
				background-color: transparent;
				border: none;
				cursor: pointer;
				font-size: 1.5ch;

				align-self: self-start;

				height: 1em;
				width: 1em;
				line-height: 0;
				transition: transform 300ms;
				will-change: transform;
				padding: 0;

				&:hover {
					transform: rotate(90deg);
				}
			}

			> footer {
				display: flex;
				justify-content: inherit;
				gap: 0.4rem;
				grid-column: span 2;

				> button {
					--_toast-button-color: lightgray;

					background-color: color-mix(in srgb, var(--_toast-button-color) 40%, white);
					border: 1px solid var(--_toast-button-color);
					border-radius: 5px;
					cursor: pointer;

					padding: 0.2rem 0.5rem;

					&.caution {
						--_toast-button-color: var(--_toast-color-warning);
					}

					&.danger {
						--_toast-button-color: var(--_toast-color-error);
					}
				}
			}
		}
	}
</style>
