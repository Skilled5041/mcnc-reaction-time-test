<script lang="ts">
    import { browser } from "$app/environment";

    let time = 0;
    let stopwatchStartTime = 0;
    let interval: number | undefined;
    const startStopwatch = () => {
        stopwatchStartTime = Date.now();
        interval = setInterval(() => {
            time = Date.now() - stopwatchStartTime;
        }, 1);
    };

    const stopStopwatch = () => {
        clearInterval(interval);
    };

    let sound: HTMLAudioElement;
    if (browser) {
        sound = new Audio("");
    }
</script>

<h1>{time}ms</h1>
<button on:click={startStopwatch}>Start</button>
<button on:click={stopStopwatch}>Stop</button>
<button on:click={() => sound?.play()}>Play Sound</button>

<style>

</style>