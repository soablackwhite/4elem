<script lang="ts">
    import { onMount } from 'svelte';
    import Monster from '../components/Monster.svelte';
    import Ui from '../components/Ui.svelte';
    import { fight } from '../stores';
    import { ProgressRadial } from '@skeletonlabs/skeleton';

    // ML stuff
    let URL: string = "https://teachablemachine.withgoogle.com/models/XOsAJst__/";
    let model: tmImage.CustomMobileNet | undefined;
    let webcam: tmImage.Webcam | undefined;
    let labelContainer: HTMLElement | null;
    let maxPredictions: number | undefined;
    let finalPrediction = 0;
    let finalPredictionName = "";
    const camWidth = 170;
    const camHeight = 140;

    // Game parameters
    const maxHP = 7;
    let winnerPredict = 0;
    let playerName = "";
    let idxMonster = 0;
    let hp = 4;
    let monsterHP = 9;
    const card_number = hp + monsterHP;
    let monsterChoice = "";
    let gameState = 0; // 0 is for menu before camera starts, 1 is for during the game, 2 is for result
    let result = "";
    let history = [];
    let historyContainer: HTMLElement;

    // Animation stuff
    let loading = false;

    async function init(): Promise<void> {
        gameState = 1;
        loading = true;
        try {
            // TensorFlow preload
            await tf.ready();
            console.log("TensorFlow.js ready");

            // Set backend (webgl or webgpu || test on different devices)
            await tf.setBackend('webgl');
            console.log("Webgl ready");

            const modelURL: string = URL + "model.json";
            const metadataURL: string = URL + "metadata.json";

            // Loading model
            model = await tmImage.load(modelURL, metadataURL);
            console.log("Model loaded");
            maxPredictions = model.getTotalClasses();

            // Webcam setup
            const flip = true; // whether to flip the webcam
            webcam = new tmImage.Webcam(camWidth, camHeight, flip); // width, height, flip
            await webcam.setup(); // request access
            await webcam.play();
            window.requestAnimationFrame(loop);

            // DOM stuff -> webcam
            labelContainer = document.getElementById("label-container");
            const webcamContainer = document.getElementById("webcam-container");
            if (webcamContainer && webcam.canvas) {
                webcamContainer.appendChild(webcam.canvas);
            }
            if (labelContainer) {
                for (let i = 0; i < maxPredictions; i++) { // and class labels
                    labelContainer.appendChild(document.createElement("div"));
                }
            }
        } catch (error) {
            console.error("Error initializing the model or webcam:", error);
        } finally {
            loading = false;
        }
    }

    async function loop(): Promise<void> {
        if (webcam) {
            webcam.update(); // update the webcam frame
            await predict();
            window.requestAnimationFrame(loop);
        }
    }

    async function predict(): Promise<void> {
        if (model && webcam && labelContainer) {
            const prediction = await model.predict(webcam.canvas);
            let maxProb = -1;
            let maxIndex = -1;
            for (let i = 0; i < maxPredictions!; i++) {
                const classPrediction = prediction[i].className + ": " + prediction[i].probability.toFixed(2);
                labelContainer.childNodes[i].innerHTML = classPrediction;
                if (prediction[i].probability > maxProb) {
                    maxProb = prediction[i].probability;
                    finalPredictionName = prediction[i].className;
                    maxIndex = i;
                }
            }
            if (maxIndex !== -1) {
                finalPrediction = maxIndex;
            }
        }
    }

    //to scroll to bottom log
    const scrollToBottom = () => {
        if (historyContainer) {
            historyContainer.scrollTop = historyContainer.scrollHeight;
        }
    };

    function attack() {
        //make random choice for monster
        idxMonster = Math.round(Math.random() * 3);
        let elements = ['Earth', 'Water', 'Fire', 'Air']
        monsterChoice = elements[idxMonster];
        // matchups
        const beats = {
            Water: 'Fire',
            Earth: 'Water',
            Fire: 'Air',
            Air: 'Earth'
        };
        //update history
        let newLog = document.createElement('div');
        let resultSpan = document.createElement('span');
        //updates based on result
        if (beats[finalPredictionName] === monsterChoice) {
            result = "win";
            monsterHP -= 1;
            resultSpan.className = "text-success-500";

        }
        else if ( Math.abs(idxMonster - finalPrediction)%2 === 0) {
            result = "tie";
            resultSpan.className = "text-warning-500";

        } else {
            result = "loss";
            hp -= 1;
            resultSpan.className = "text-error-500";
        }
        resultSpan.textContent = result;
        newLog.textContent = `${finalPredictionName} vs ${monsterChoice}: `;
        newLog.className = finalPredictionName;
        newLog.appendChild(resultSpan);
        historyContainer.appendChild(newLog);
        scrollToBottom();
        //trigger animation in Monster component
        fight.set(true);
        winnerPredict = finalPrediction;
    }
</script>

<!------------------------------------ UI BANNER CARD ---------------------------------------->
<div class="banner">
    <div class="block card card-hover player-container variant-ghost-tertiary"> 
        <div class="player-banner inline-flex flex-row">
            <div class="p-2" id="playerName">  <h4 class="h5">{(playerName.length > 0) ? playerName : "player"}</h4> </div>
            <div class="p-2 hp"> <h4 class="h5">{hp}/{maxHP}</h4> </div>
        </div>
        <div id="webcam-container" class="avatar-image"></div>
    </div>
    <div class="history p-2" bind:this={historyContainer}> <h5 class="h5"> Log </h5> </div>
    <div id="label-container"></div>
</div>

{#if gameState === 0}
<!-------------------------------- PLAYER NAME | WELCOME SCREEN ------------------------------>
<div class="textbox">
    <h1 class="h1 my-8">4 E L E M</h1>
    <form on:submit={init}>
        <label class="label">
            <span> type in your name </span>
            <input class="input" type="text" placeholder="player" maxlength="10" bind:value={playerName} />
        </label>
        <button type="submit" class="btn variant-filled m-4"> submit </button>
    </form>
</div>
<!--------------------------------------- GAME LOOP ------------------------------------------->
{:else if  gameState === 1 && !loading}
    <Monster state={gameState} match={result} index={idxMonster} indexPlayer={winnerPredict}/>
    <Ui index={finalPrediction}>
        <h2 class="h2"> {finalPredictionName}</h2>
    </Ui>
    <div class="textbox">
        <button id="attack" on:click={attack} class="btn variant-filled m-4"> ATTACK </button>
    </div>
<!------------------------------------- RESULT SCREEN ----------------------------------------->
{:else}
<!------------------------------------- ERROR SCREEN ------------------------------------------>
{/if}

<!---------------------------------------- LOADER --------------------------------------------->
{#if loading}
    <div class="loader"> 
        <ProgressRadial width="w-32" />
    </div>
{/if}

<style>
    .banner{
        display: flex;
        /* flex-flow: row ; */
        flex-direction: row;
    }
    .player-container{
        border-radius: 10px;
        overflow: hidden
    }
    .player-banner{
        @apply text-success-500;
        align-items: center;
        align-content: center;
        justify-content: center;
    }
    .hp{
        @apply text-error-500;
        width: 100%;
        text-align: right;
    }
    .loader{
        position: fixed;
        top:0;
        left:0;
        display: flex;
        width: 100vw;
        height: 100vh;
        align-items: center;
        justify-content: center;
        align-content: center;
    }
    .textbox{
        position: absolute;
        left: 15%;
        top: 0;
        width: 70%;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        text-align: center;
        align-items: center;
        align-content: center;
    }
    .instructions{
        text-align: left;
    }
    * {
        font-family: 'Pixelify Sans', sans-serif;
        font-weight: 400;
    }
    .label{
        width: 300px;
    }
    input{
        padding: 10px !important;
    }
    .textbox h1{
        text-shadow: 2px 2px 2px rgba(255, 255, 255, 0.357);
        font-size: 4em;
        /* gradient stuff */
        @apply bg-clip-text text-transparent box-decoration-clone;
        @apply bg-gradient-to-br;
        @apply from-primary-500 via-tertiary-500 to-secondary-500;
    }
    .history{
        right: 0px;
        width: 170px;
        max-height: 180px;
        overflow-y: scroll;
        background-color: rgba(255, 255, 255, 0.07);
        font-size: small;
        border-radius: 10px;
    }
    #attack{
        margin-top: 300px;
    }
    #label-container{
        display: none;
    }
</style>
