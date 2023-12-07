<script lang="ts">
    import { browser } from "$app/environment";
    import logo from "$lib/images/logo.webp";
    import type { KeyboardEventHandler } from "svelte/elements";
    import type { ISourceOptions, Engine } from "@tsparticles/engine";
    import { onMount, type SvelteComponent } from "svelte";
    import { loadFull } from "tsparticles";
    import { loadPolygonPath } from "@tsparticles/path-polygon";

    let ParticlesComponent: typeof SvelteComponent;

    onMount(async () => {
        const module = await import("svelte-particles");
        ParticlesComponent = module.default as typeof SvelteComponent;
    });

    const particlesConfig: ISourceOptions = {
        particles: {
            color: {
                value: "#ff8080",
                animation: {
                    enable: true,
                    speed: 100
                }
            },
            move: {
                direction: "none",
                enable: true,
                outModes: {
                    default: "destroy"
                },
                path: {
                    clamp: false,
                    enable: true,
                    delay: {
                        value: 0
                    },
                    generator: "polygonPathGenerator",
                    options: {
                        sides: 6,
                        turnSteps: 50,
                        angle: 0
                    }
                },
                random: false,
                speed: 8,
                straight: false,
                trail: {
                    fill: {
                        color: "#000"
                    },
                    length: 40,
                    enable: true
                }
            },
            number: {
                value: 0
            },
            opacity: {
                value: 1
            },
            shape: {
                type: "circle"
            },
            size: {
                value: 3
            }
        },
        background: {
            color: "#000"
        },
        fullScreen: {
            zIndex: -1
        },
        emitters: {
            direction: "none",
            rate: {
                quantity: 1,
                delay: 0.25
            },
            size: {
                width: 0,
                height: 0
            },
            position: {
                x: 50,
                y: 50
            }
        }
    };


    const particlesInit = async (engine: Engine) => {
        loadPolygonPath(engine);
        // @ts-ignore
        await loadFull(engine);
    };

    let time = 0;
    let timeStartTime = 0;
    let selectedSound = "game-sound";

    let sound: HTMLAudioElement;
    $: if (browser && selectedSound) {
        sound?.pause();
        sound = new Audio(`/audio/${selectedSound}.wav`);
        sound.load();
    }

    let testRunning = false;
    let soundPlaying = false;
    let failed = false;
    let timeout: number;

    const handleSpaceBar: KeyboardEventHandler<Window> = (event) => {
        if (event.key !== " ") return;

        if (testRunning) {
            clearTimeout(timeout);
            testRunning = false;
            if (!soundPlaying) {
                failed = true;
            } else {
                time = Date.now() - timeStartTime;
                failed = false;
                soundPlaying = false;
                sound.pause();
                sound.currentTime = 0;
            }
        } else {
            time = -1;
            testRunning = true;
            timeout = setTimeout(() => {
                soundPlaying = true;
                sound.play();
                timeStartTime = Date.now();
            }, randomInt(minTime, maxTime));
        }
    };

    let bgEnabled = true;
    let minTime = 500;
    let maxTime = 15000;
    const randomInt = (min: number, max: number) => Math.floor(Math.random() * (max - min + 1)) + min;
</script>
<svelte:window on:keydown={handleSpaceBar}/>
{#if bgEnabled}
    <svelte:component
            options={particlesConfig}
            {particlesInit}
            this={ParticlesComponent}
    />
{/if}
<div class="container">
    {#if testRunning}
        <h1 class="title">Press Space When You Hear The Sound</h1>
    {:else}
        {#if failed}
            <h1 class="title">You Pressed Space Too Early!</h1>
        {/if}
        {#if time !== -1}
            <h1 class="title">Your Reaction Time Was {time}ms</h1>
        {/if}
        <div class="columns">
            <img src={logo} alt="MCNC Logo" width="100">
            <h1 class="title">Press Space To Start</h1>
            <img src={logo} alt="MCNC Logo" width="100">
        </div>
        <form>
            <div class="rows">
                <select bind:value={selectedSound}>
                    <option value="game-sound">Game Sound</option>
                    <option value="beep">Beep</option>
                    <option value="alarm">Alarm</option>
                    <option value="explosion">Explosion</option>
                    <option value="roar">Animal Roar</option>
                    <option value="surprise">Surprised Sound Effect</option>
                    <option value="thunder">Thunder</option>
                    <option value="piano">Piano Music</option>
                </select>
                <label>
                    <span>Minimum Time (ms)</span>
                    <input type="number" bind:value={minTime}>
                </label>
                <label>
                    <span>Maximum Time (ms)</span>
                    <input type="number" min={0} max={maxTime} bind:value={maxTime}>
                </label>
                <label>
                    <input type="checkbox" min={minTime} bind:checked={bgEnabled}>
                    <span>Enable Background</span>
                </label>
            </div>
        </form>
    {/if}
</div>

<style>
    :global(body) {
        background-color: black;
    }

    * {
        color: #fff;
        font-family: "Fira Code", monospace;
        border-radius: 8px;
    }

    .container {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100vh;
        gap: 2rem;
    }

    .title {
        font-size: 5rem;
        text-align: center;
        margin-bottom: 1rem;
    }

    select {
        background-color: black;
        border-color: #ffffff;
        border-style: solid;
        padding: 1rem;
        font-size: 1.5rem;
        width: 40vw;
        appearance: none;
    }

    input {
        background: black;
        border: solid white 1px;
        padding: 0.5rem;
        font-size: 1rem;
    }

    .columns {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;
        gap: 2rem;
    }

    .rows {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 2rem;
    }

    span {
        font-size: 1.5rem;
    }
</style>
