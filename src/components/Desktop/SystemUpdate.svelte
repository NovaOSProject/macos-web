<script lang="ts">
	import { untrack } from 'svelte';
	import { useRegisterSW } from 'virtual:pwa-register/svelte';
	import { system_needs_update } from '🍎/state/system.svelte';
	import SystemDialog from '../SystemUI/SystemDialog.svelte';

	let system_update_dialog = $state<SystemDialog>();

	// replaced dynamically
	const build_date = '__DATE__';

	// Will store the update event, so we can use this value on AppStore to show the badge.
	// If the user click on Later instead Restart, the dialog is closed but the update is still there.
	// We don't need to store it on localStorage since the new sw is on skip waiting state, and so
	// a refresh or reopening the browser will prompt again the dialog to restart.
	// Once updateServiceWorker is called, there is a full reload, so the app will be loaded again.

	const { needRefresh, updateServiceWorker } = useRegisterSW({
		onRegistered(swr) {
			console.log(`SW registered: ${swr}`);
		},
		onRegisterError(error) {
			console.log('SW registration error', error);
		},
	});

	$effect(() => {
		if ($needRefresh) {
			system_update_dialog?.open();
		}
	});

	$effect(() => {
		$needRefresh;

		untrack(() => (system_needs_update.value = $needRefresh));
	});

	function close() {
		system_update_dialog.close();
		needRefresh.set(false);
	}

	async function handle_update_app() {
		updateServiceWorker();
	}
</script>

<SystemDialog bind:this={system_update_dialog}>
	<section class="system-update-section">
		<img
			width="128"
			height="128"
			src="/app-icons/system-preferences/256.webp"
			alt="AppStore app"
			draggable="false"
		/>

		<h3>Updates Available</h3>
		<p>Do you want to restart to install these updates now?</p>

		<div class="buttons">
			<button onclick={close}>Later</button>
			<button class="confirm" onclick={handle_update_app}> Update </button>
		</div>
	</section>
</SystemDialog>

<div class="pwa-date">{build_date}</div>

<style>
	.pwa-date {
		visibility: hidden;

		height: 0;
		width: 0;
	}

	.system-update-section {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;

		padding: 1rem 0 0;

		width: 20rem;

		color: var(--system-color-dark);

		h3,
		p {
			text-align: center;
		}

		h3 {
			font-size: 1.2rem;
			font-weight: 500;
		}

		p {
			font-size: 0.9rem;
		}

		.buttons {
			display: grid;
			grid-template-columns: 1fr 1fr;
			gap: 0.5rem;

			width: 100%;

			button {
				width: 100%;
				height: 2rem;

				font-weight: 500;

				border-radius: 0.5rem;

				background-color: hsla(var(--system-color-dark-hsl), 0.2);

				&:hover {
					background-color: hsla(var(--system-color-dark-hsl), 0.3);
				}

				&.confirm {
					background-color: var(--system-color-primary);

					color: var(--system-color-primary-contrast);

					&:hover {
						background-color: hsla(var(--system-color-primary-hsl), 0.8);
					}
				}
			}
		}
	}
</style>
