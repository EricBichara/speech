<script lang="ts">
    import {onMount} from "svelte";

    const keys = ['A', 'A#', 'B', 'C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#']
    const types = ['Major', 'Minor'];
    const inversions = ['Root Inversion - 1 3 5', '1st Inversion - 3 5 1', '2nd Inversion - 5 1 3'];
    let key = keys[0];
    let type = types[0];
    let inversion = inversions[0];
    let intervalId;

    let recognition;
    let transcript;
    let isRecording = false;
    let instructions = 'Not Listening';
    onMount(() => {
        try {
            let SpeechRecognition =
                window.SpeechRecognition || window.webkitSpeechRecognition;
            recognition = new SpeechRecognition();
            recognition.continuous = true;

            recognition.onresult = function (event) {
                let current = event.resultIndex;

                transcript = event.results[current][0].transcript;

                if (transcript.trim() === 'stop') {
                    console.log('stopping');
                    recognition.stop();
                    isRecording = false;
                }

                if (transcript.trim() === 'next') {
                    doAtInterval();
                }
            };

            recognition.onstart = function () {
                isRecording = true;
                instructions = 'Recording';
            }

            recognition.onspeechend = function () {
                instructions = 'Stopped';
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
        intervalId = setInterval(doAtInterval, 10000);
    }

    function stopInterval() {
        if (intervalId) {
            clearInterval(intervalId);
        }
    }

    function doAtInterval() {
        key = keys[Math.floor(Math.random() * keys.length)];
        type = types[Math.floor(Math.random() * types.length)];
        inversion = inversions[Math.floor(Math.random() * inversions.length)];
    }
</script>
<div class="place-content-center h-screen grid">
    <button class="btn"
            class:btn-error={!isRecording}
            class:btn-primary={isRecording}
            on:click={toggle}>{!isRecording ? 'Start Listening' : 'Stop Listening'}</button>

    <div class="flex flex-col items-center my-10">
        <div class="text-4xl mb-2 font-bold">{key}</div>
        <div class="text-4xl mb-2 font-bold">{type}</div>
        <div class="text-4xl font-bold">{inversion}</div>
    </div>

    <div class="flex flex-row items-center justify-around">
        <button class="btn btn-primary" on:click={startInterval}>Start</button>
        <button class="btn btn-accent" on:click={stopInterval}>Stop</button>
        <button class="btn btn-active" on:click={doAtInterval}>Next</button>
    </div>
</div>


