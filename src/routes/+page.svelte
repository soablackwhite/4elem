<script lang="ts">
    import { onMount } from 'svelte';
    import Monster from '../components/Monster.svelte';
    import Ui from '../components/Ui.svelte';
    import { fight } from '../stores';
    import { ProgressRadial} from '@skeletonlabs/skeleton';

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
    let elements = ['Earth', 'Water', 'Fire', 'Air'];
    let colors = ['text-success-500', 'text-tertiary-500', 'text-error-500', 'text-primary-400'];
    let jokerColor:string;
    let winnerPredict = 0;
    let playerName = "";
    let idxMonster = 0;
    let gameState = 0; // 0 is for menu before camera starts, 1 is for during the game, 2 is for result
    let result = "";
    let historyContainer: HTMLElement;
    //hearts
    let monsterHP = 5;
    const maxHP = 3;
    let hp = 3;
    let hearts = [];
    function updateHearts() {
        hearts = Array.from({ length: hp }, (_, i) => i < maxHP);
    }
    //monster hand
    let hand = [];
    let monsterChoice = "";
    const card_number = Math.round(( hp + monsterHP ) * 2); //this should be 12 lol
    let cardCount = 0 + card_number;
    let joker: string;
    let jokerCount = 0;
   
    //animation stuff
    let loading = false;

    onMount(() => {
        hand = distributeCards(card_number);
        updateHearts();
    })

    function distributeCards(totalcards:number){
        //pick the joker
        let pick = Math.floor(Math.random() * 4); //draw the joker
        joker = elements[pick];
        jokerColor = colors[pick];
        //distribute cards
        let res: Array<string>;
        res = [];
        for (let i = 0; i < totalcards; i++){
            let cardpick = elements[Math.floor(Math.random() * 4)];
            jokerCount = (cardpick === joker) ? jokerCount + 1 : jokerCount; //count jokers
            res.push(cardpick);
        }
        return(res)
    }

    // game reset function
    function reset(){
        //remove all logs
        while (historyContainer.firstChild) {
            historyContainer.removeChild(historyContainer.firstChild);
        }
        //reset variables
        monsterHP = 5;
        hp = 3;
        gameState = 1;
        //reset hand
        cardCount = 0 + card_number;
        jokerCount = 0;
        hand = distributeCards(card_number);
        //reset animations and UI
        updateHearts();
    }

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
        //card drawing
        monsterChoice = hand.pop(); //draw a card
        idxMonster = elements.indexOf(monsterChoice); //update numerical index
        cardCount -= 1; //update count
        jokerCount = (monsterChoice === joker) ? jokerCount - 1 : jokerCount; //update joker count

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
            updateHearts();

        }
        else if ( Math.abs(idxMonster - finalPrediction)%2 === 0) {
            result = "draw";
            resultSpan.className = "text-warning-500";
        } else {
            result = "loss";
            hp -= 1;
            resultSpan.className = "text-error-500";
            updateHearts();
        }
        //update history
        resultSpan.textContent = result;
        newLog.textContent = `${finalPredictionName} vs ${monsterChoice}: `;
        newLog.className = finalPredictionName;
        newLog.appendChild(resultSpan);
        historyContainer.appendChild(newLog);
        scrollToBottom();
        //trigger animation in Monster component
        fight.set(true);
        winnerPredict = finalPrediction;

        //game state
        if ( hp < 1 ){ //lose
            gameState = 2;
        } else if (monsterHP < 1){ //win
            gameState = 3;
        } else if (hand.length < 1){ //second win condition
            gameState = 3;
        }
    }
</script>

<!-- INSTRUCTIONS -->

<!------------------------------------ UI BANNER CARD ---------------------------------------->
<div class="banner">
    <div class="block card card-hover player-container variant-ghost-black"> 
        <div class="player-banner inline-flex flex-row">
            <div class="p-2"> <h4 class="h5"> <strong> {(playerName.length > 0) ? playerName : "player"} </strong></h4> </div>
            <div class="hearts">
                {#each hearts as heart}
                    <img alt="life icon" class="heart" src="/media/heart.png" />
                {/each}
            </div>
        </div>
        <div id="webcam-container" class="avatar-image"></div>
    </div>
    <div class="history p-2" bind:this={historyContainer}> <h5 class="h5"> Log </h5> </div>
    <div id="label-container"></div>
</div>

{#if gameState === 0}
<!---------------------------- PLAYER NAME | WELCOME SCREEN ----------------------------->
<div class="textbox" >
    <h1 class="h1 my-8 title">4 E L E</h1>
    <form on:submit={init}>
        <label class="label">
            <span> type in your name </span>
            <input class="input" type="text" placeholder="player" maxlength="10" bind:value={playerName} />
        </label>
        <button type="submit" class="btn variant-filled m-4"> submit </button>
    </form>
</div>
<!------------------------------------- GAME LOOP ------------------------------------------->
{:else if  gameState === 1 && !loading}
    <!---------------------------- MONSTER OVERLAY, JOKERINFO ------------------------------->
    <Monster state={gameState} match={result} index={idxMonster} indexPlayer={winnerPredict} monsterHP = {monsterHP}/>
    <div class="jokerInfo">
        <div class= "jokerLabel {jokerColor}">
            {jokerCount} x {joker}
        </div>
        <div>
            {cardCount - jokerCount} other
        </div>
    </div>
    <Ui index={finalPrediction}>
        <h2 class="h2"> {finalPredictionName}</h2>
    </Ui>
    <div class="textbox">
        <button id="attack" on:click={attack} class="btn variant-filled m-4"> ATTACK </button>
    </div>
<!------------------------------------- WIN SCREEN ----------------------------------------->
{:else if gameState === 2}
    <div class="textbox">
        <h1 class="h1 defeat"> D E F E A T </h1>
        <button on:click={reset} class="btn variant-filled my-12"> RESTART </button>
    </div>
<!------------------------------------- LOSE SCREEN ----------------------------------------->
{:else if gameState === 3}
    <div class="textbox" >
        <h1 class="h1 victory"> V I C T O R Y </h1>
        <button on:click={reset} class="btn variant-filled my-12"> RESTART </button>
    </div>
{/if}

<!---------------------------------------- LOADER ------------------------------------------->
{#if loading}
    <div class="loader" > 
        <ProgressRadial width="w-32" />
    </div>
{/if}


<style>
    .jokerInfo{
        position: absolute;
        top: 50%;
        left: calc(50% + 140px);
        display: flex;
        flex-flow: column wrap;
        text-align: right;
        width: 120px;
        transform: translate(-50%, -50%);
        color: white;
    }
    .jokerInfo div{
        /* border: rgba(255, 255, 255, 0.202) solid 2px; */
        margin-bottom: 10px;
        background-color: rgba(0, 0, 0, 0.304);
        padding: 4px;
        border-radius: 20px;
        transition: all 0.2s;
    }
    .jokerInfo div:hover{
        transform: translate(10px, -3px);
        background-color: rgba(255, 255, 255, 0.304);
    }
    .jokerLabel{
        text-shadow: -3px 1px 3px rgba(0, 0, 0, 0.238);
    }
    .hearts{
        margin-top: 2px;
        display: flex;
        flex-flow: row wrap;
        justify-content: right;
        padding: 4px;
        width: 100%;
    }
    .hearts img{
        width: 24px;
    }
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
        width: 100%;
        align-items: center;
        align-content: center;
        justify-content: center;
        background-color: rgba(255, 255, 255, 0.836);
        color: black;
    }
    .card{
        border: 1px solid rgba(255, 255, 255, 0.333);
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
        z-index: -10;
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
    .title{
        text-shadow: 2px 2px 2px rgba(255, 255, 255, 0.357);
        font-size: 6em;
    }
    .defeat{
        text-shadow: 2px 10px 12px rgba(0, 0, 0, 0.557);
        font-size: 5em;
        @apply bg-clip-text text-transparent box-decoration-clone;
        @apply bg-gradient-to-br;
        @apply from-violet-500 via-error-500 to-secondary-800;
    }
    .victory{
        text-shadow: 2px 10px 15px rgba(235, 235, 235, 0.157);
        font-size: 5em;
        @apply bg-clip-text text-transparent box-decoration-clone;
        @apply bg-gradient-to-br;
        @apply from-warning-500 via-success-500 to-warning-500;
    }
    .history{
        right: 0px;
        width: 170px;
        max-height: 180px;
        overflow-y: scroll;
        background-color: rgba(255, 255, 255, 0.07);
        font-size: small;
        border-radius: 10px;
        z-index: 1;
    }
    #attack{
        margin-top: 340px;
    }
    #label-container{
        display: none;
    }
    @media(max-width:600px){
        .title{
            font-size: 5em;
        }
        .victory, .defeat{
            font-size: xxx-large;
        }

    }
    @media(max-width:400px){
        .monster-card{
            top: 55% !important;
        }
        #attack{
            margin-top: 380px;
        }
        .victory, .defeat{
            font-size: 2.7em;
        }
    }
</style>
