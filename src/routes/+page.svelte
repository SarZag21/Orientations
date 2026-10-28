<script>
	import { onMount } from 'svelte';

	let alpha = $state(null);
	let beta = $state(null);
	let gamma = $state(null);
	let message = $state('Tap the button to start.');
	let supported = $state(true);
	let enabled = $state(false);

	function handleOrientation(event) {
		alpha = event.alpha;
		beta = event.beta;
		gamma = event.gamma;
	}

	async function start() {
		try {
			const sensor = window.DeviceOrientationEvent;

			if (typeof sensor?.requestPermission === 'function') {
				const permission = await sensor.requestPermission();
				if (permission !== 'granted') {
					message = 'Motion permission was not granted.';
					return;
				}
			}

			window.addEventListener('deviceorientation', handleOrientation);
			enabled = true;
			message = 'Move your device to see the readings.';
		} catch {
			message = 'Could not access motion sensors. Try allowing access in your browser settings.';
		}
	}

	onMount(() => {
		supported = 'DeviceOrientationEvent' in window;
		if (!supported) message = 'This browser does not support device orientation.';

		return () => window.removeEventListener('deviceorientation', handleOrientation);
	});
</script>



<h1>Device Orientation</h1>
<p>{message}</p>
<button onclick={start} disabled={!supported || enabled}>
	{enabled ? 'Started' : 'Start'}
</button>
<p>Alpha: {alpha ?? '—'}</p>
<p>Beta: {beta ?? '—'}</p>
<p>Gamma: {gamma ?? '—'}</p>
<p>On phones, open this page over HTTPS and tap Start.</p>
