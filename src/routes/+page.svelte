<script>
    import { onMount } from 'svelte';

    let dice = $state(1);
    let status = $state('Press Start');
    let started = $state(false);

    let gravity = { x: null, y: null, z: null };
    let rolling = false;
    let timer;

    // Start sensors
    async function start() {
        if (!window.isSecureContext) {
            status = 'HTTPS required';
            return;
        }

        if (!window.DeviceMotionEvent || !window.DeviceOrientationEvent) {
            status = 'Sensors not supported';
            return;
        }

        try {
            // Ask permission on iPhones
            if (typeof DeviceMotionEvent.requestPermission === 'function') {
                const permission = await DeviceMotionEvent.requestPermission();
                if (permission !== 'granted') {
                    status = 'Permission denied';
                    return;
                }
            }

            if (typeof DeviceOrientationEvent.requestPermission === 'function') {
                const permission = await DeviceOrientationEvent.requestPermission();
                if (permission !== 'granted') {
                    status = 'Permission denied';
                    return;
                }
            }

            window.removeEventListener('devicemotion', motion);
            window.addEventListener('devicemotion', motion);

            started = true;
            status = 'Shake your phone!';
        } catch (error) {
            status = error.message;
        }
    }

    // Detect shaking and read gravity
    function motion(event) {
        const g = event.accelerationIncludingGravity;

        if (g && g.x !== null && g.y !== null && g.z !== null) {
            gravity = { x: g.x, y: g.y, z: g.z };
        }

        const a = event.acceleration;
        if (!a || a.x === null || a.y === null || a.z === null) return;

        const force = Math.sqrt(
            a.x ** 2 + a.y ** 2 + a.z ** 2
        );

        // Shake detected
        if (force > 15 && !rolling) {
            rolling = true;
            status = 'Rolling...';

            // Wait until phone stops moving
            timer = setTimeout(() => {
                calculateDice();
                rolling = false;
                status = 'Shake again!';
            }, 1000);
        }
    }

    // Calculate the number from phone position
    function calculateDice() {
        const { x, y, z } = gravity;

        if (x === null || y === null || z === null) {
            status = 'No gravity data';
            return;
        }

        // Screen up or down
        if (Math.abs(z) >= Math.abs(x) && Math.abs(z) >= Math.abs(y)) {
            dice = z >= 0 ? 1 : 6;
        }
        // Right or left
        else if (Math.abs(x) >= Math.abs(y)) {
            dice = x >= 0 ? 3 : 4;
        }
        // Top or bottom
        else {
            dice = y >= 0 ? 2 : 5;
        }
    }

    // Clean up sensors
    onMount(() => {
        return () => {
            clearTimeout(timer);
            window.removeEventListener('devicemotion', motion);
        };
    });
</script>

<h1>Phone Dice 🎲</h1>

<h1 style="font-size: 100px;">{dice}</h1>

<button onclick={start} disabled={started}>
    {started ? 'Sensors started' : 'Start'}
</button>

<p>{status}</p>
