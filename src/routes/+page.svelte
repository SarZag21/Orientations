
<script>
    import { onMount } from 'svelte';

    let dice = $state('-');
    let status = $state('Press Start');
    let started = $state(false);

    let gravity = { x: null, y: null, z: null };
    let rolling = false;
    let timer;

    // Start the phone sensors
    async function start() {
        if (!window.isSecureContext) {
            status = 'HTTPS required';
            return;
        }

        if (!window.DeviceMotionEvent) {
            status = 'Device Motion not supported';
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

            window.addEventListener('devicemotion', motion);

            started = true;
            status = 'Shake your phone!';
        } catch (error) {
            status = error.message;
        }
    }

    // Detect shaking and read phone position
    function motion(event) {
        const g = event.accelerationIncludingGravity;

        if (g && g.x !== null && g.y !== null && g.z !== null) {
            gravity = { x: g.x, y: g.y, z: g.z };
        }

        const a = event.acceleration;

        if (!a || a.x === null || a.y === null || a.z === null) {
            return;
        }

        const force = Math.sqrt(
            a.x ** 2 + a.y ** 2 + a.z ** 2
        );

        // Detect a shake
        if (force > 15 && !rolling) {
            rolling = true;
            status = 'Rolling...';

            // Wait one second before showing the result
            timer = setTimeout(() => {
                calculateDice();
                rolling = false;
                status = 'Shake again!';
            }, 1000);
        }
    }

    // Fixed numbers based on phone position
    function calculateDice() {
        const { x, y, z } = gravity;

        if (x === null || y === null || z === null) {
            status = 'No sensor data';
            return;
        }

        // Screen up = 1, screen down = 6
        if (Math.abs(z) >= Math.abs(x) && Math.abs(z) >= Math.abs(y)) {
            dice = z > 0 ? 1 : 6;
        }

        // Right edge up = 3, left edge up = 4
        else if (Math.abs(x) >= Math.abs(y)) {
            dice = x > 0 ? 4 : 3;
        }

        // Top edge up = 2, bottom edge up = 5
        else {
            dice = y > 0 ? 2 : 5;
        }
    }

    // Remove sensor listener when leaving the page
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
    {started ? 'Started' : 'Start'}
</button>

<p>{status}</p>
