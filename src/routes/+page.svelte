<script lang="ts">
    import {onMount} from "svelte";
    import isMobile from "ismobilejs";

    const keys = ['A', 'A#', 'B', 'C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#']
    const inversions = ['Root Inversion', '1st Inversion', '2nd Inversion'];
    let types = [];
    let key = keys[0];

    let inversion = inversions[0];
    let intervalId;

    $: {
        const chords = [];
        if (includeMajor) {
            chords.push('Major');
        }
        if (includeMinor) {
            chords.push('Minor');
        }
        if (includeSus2) {
            chords.push('Sus2');

        }
        if (includeSus4) {
            chords.push('Sus4');
        }
        types = chords;
    }
    let type = types[0];

    let includeMajor = true;
    let includeMinor = true;
    let includeSus2 = false;
    let includeSus4 = false;
    let countValue = '10';

    let recognition;
    let transcript;
    let isRecording = false;

    let enableListening = false;

    $: count = parseInt(countValue);

    onMount(() => {
        enableListening = !isMobile().any;

        type = types[0];
        try {
            let SpeechRecognition =
                window.SpeechRecognition || window.webkitSpeechRecognition;
            recognition = new SpeechRecognition();
            recognition.continuous = true;

            recognition.onresult = function (event) {
                let current = event.resultIndex;

                transcript = event.results[current][0].transcript;

                if (transcript.trim() === 'stop') {
                    recognition.stop();
                    isRecording = false;
                }

                if (transcript.trim() === 'next') {
                    doAtInterval();
                }
            };

            recognition.onstart = function () {
                isRecording = true;
            }
        } catch (e) {
            console.error(e);
        }
    });

    function toggle() {
        if (isRecording === true) {
            recognition.stop();
            isRecording = false;
        } else {
            recognition.start();
            isRecording = true;
        }
    }

    function startInterval() {
        stopInterval();
        intervalId = setInterval(doAtInterval, 1000);
    }

    function stopInterval() {
        if (intervalId) {
            clearInterval(intervalId);
            intervalId = null;
        }
        count = parseInt(countValue);
    }

    function doAtInterval() {
        if (intervalId) {
            if (count > 1) {
                count--;
                return;
            } else {
                count = parseInt(countValue);
            }
        }

        key = keys[Math.floor(Math.random() * keys.length)];
        type = types[Math.floor(Math.random() * types.length)];
        inversion = inversions[Math.floor(Math.random() * inversions.length)];
    }
</script>
<div class="mx-auto max-w-lg pt-2 px-4 md:px-0">

    <div class="flex flex-col items-center my-10">
        <div class="text-4xl mb-2 font-bold">{key}</div>
        <div class="text-4xl mb-2 font-bold">{type}</div>
        <div class="text-4xl font-bold">{inversion}</div>
    </div>

    <button class="w-full btn btn-active flex-1 drop-shadow-lg border-1" disabled={intervalId} on:click={doAtInterval}>
        Next
    </button>

    <div class="flex flex-row items-center justify-center gap-3 mt-4">
        <button class="btn btn-primary flex-1 drop-shadow-lg border-1" disabled={intervalId != null}
                on:click={startInterval}>Start
        </button>
        <div class="flex-1 flex border-2 bg-white drop-shadow-lg rounded-full h-12 justify-center items-center">
            <div class="font-bold">{count}</div>
        </div>
        <button class="btn btn-accent flex-1 drop-shadow-lg border-1" disabled={isRecording || intervalId == null}
                on:click={stopInterval}>Stop
        </button>
    </div>

    {#if enableListening}
        <div class="flex mt-4 tooltip"
             data-tip="When listening say 'Next' to go to the next chord or 'Stop' to stop listening">
            <button class="btn flex-1 drop-shadow-lg border-1"
                    disabled={intervalId}
                    class:btn-error={!isRecording}
                    class:btn-primary={isRecording}
                    on:click={toggle}>{!isRecording ? 'Start Listening' : 'Stop Listening'}</button>
        </div>
    {/if}

    <div class="mt-4 font-bold">Settings</div>
    <div class="flex flex-col">
        <div class="form-control">
            <label class="label cursor-pointer">
                <span class="label-text">Major Chords</span>
                <input type="checkbox" bind:checked={includeMajor} class="checkbox"/>
            </label>
        </div>
        <div class="form-control">
            <label class="label cursor-pointer">
                <span class="label-text">Minor Chords</span>
                <input type="checkbox" bind:checked={includeMinor} class="checkbox"/>
            </label>
        </div>
        <div class="form-control">
            <label class="label cursor-pointer">
                <span class="label-text">Sus2 Chords</span>
                <input type="checkbox" bind:checked={includeSus2} class="checkbox"/>
            </label>
        </div>
        <div class="form-control">
            <label class="label cursor-pointer">
                <span class="label-text">Sus4 Chords</span>
                <input type="checkbox" bind:checked={includeSus4} class="checkbox"/>
            </label>
        </div>
        <div class="form-control">
            <label class="label cursor-pointer">
                <span class="label-text">Count down (in seconds)</span>
                <input type="text" bind:value={countValue} class="input input-bordered w-14 bg-white h-10 text-end"
                       disabled={intervalId}/>
            </label>
        </div>
    </div>
</div>


