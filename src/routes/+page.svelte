
<script>
    let alpha = $state(null);
    let beta = $state(null);
    let gamma = $state(null);

    async function start() {
        // Ask for permission on iPhones
        if (typeof DeviceOrientationEvent.requestPermission === 'function') {
            const permission = await DeviceOrientationEvent.requestPermission();

            if (permission !== 'granted') return;
        }

        // Read phone rotation
        window.addEventListener('deviceorientation', (event) => {
            alpha = event.alpha;
            beta = event.beta;
            gamma = event.gamma;
        });
    }
</script>

<h1>Device Orientation</h1>

<button onclick={start}>Start</button>

<p>Alpha: {alpha}</p>
<p>Beta: {beta}</p>
<p>Gamma: {gamma}</p>

