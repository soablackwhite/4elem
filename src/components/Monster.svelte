<script lang="ts">
    import { onMount } from 'svelte';
    import { fight } from '../stores.js'
    export let state = 0;
    export let match = "";
    export let index = 0;
    export let indexPlayer = 0;

    // Animation stuff
    const spriteSheet = "media/animation/barrierAni.png";
    const spriteWidth = 192;
    const spriteHeight = 192;
    const totalFrames = 20;
    const frameRate = 20;
    let animationInterval = 0;
    let frameShield = 0;
    let isAnimatingMonster = false;

    // SFX
    const effectSheet = ["earthAni.png", "waterAni.png", "fireAni.png", "windAni.png"];
    const effectWidth = 320;
    const effectHeight = 320;
    let totalFramesEffect = [16, 13, 10, 10];
    let finalFrame = [15, 0, 9, 9];
    let effectInterval = 0;
    let isAnimatingEffect = false;
    let frameEffect = 0;

    //hp stuff
    export let monsterHP = 0;
    let maxHP = 5;
    let hearts = [];
    function updateHearts() {
        hearts = Array.from({ length: monsterHP }, (_, i) => i < maxHP);
    }

    function stopMonster() {
        clearInterval(animationInterval);
        isAnimatingMonster = false;
        fight.set(false);
    }

    function animateMonster() {
        if (isAnimatingMonster) {
            stopMonster();
        }
        frameShield = 0;
        isAnimatingMonster = true;
        animationInterval = setInterval(() => {
            frameShield = (frameShield + 1) % totalFrames;
            if (frameShield === 0) {
                stopMonster();
            }
        }, 1000 / frameRate);
    }

    function stopEffect() {
        clearInterval(effectInterval);
        isAnimatingEffect = false;
        fight.set(false);
    }

    function animateEffect() {
        if (isAnimatingEffect) {
            stopEffect();
        }
        frameEffect  = 0;
        isAnimatingEffect = true;
        const effectIndex = (match === "win") ? indexPlayer : index;
        effectInterval = setInterval(() => {
            frameEffect  = (frameEffect  + 1) % totalFramesEffect[effectIndex];
            if (frameEffect  === finalFrame[effectIndex]) {
                stopEffect();
            }
        }, 1000 / frameRate);
    }

    $: if ($fight) {
        switch (match) {
            case "draw":
                animateMonster();
                break;
            case "win":
                animateEffect();
                break;
            case "loss":
                animateEffect();
                break;
        }
        updateHearts();
    }

    onMount(() => {
        updateHearts();
        return () => stopMonster();
    });
</script>

<!------------- LISTENER -------------->

<!---------- MONSTER ---------->
<div class="monster-card card p-4"> 
    <header class="card-header"><h2 class="h2">SKULLY</h2></header>
    <div class="row">
    <img alt="cute monster image" id="skully" src="/media/monster.png">
        <div class="hearts">
            {#each hearts as heart}
                <img alt="life icon" class="heart" src="/media/heartbw.png" />
            {/each}
        </div>
    </div>
</div>

  
<!------------ SHIELD EFFECT ON DRAW ---------->
    <div
        class="sprite-container effect { (match==="draw") ? "activeEffect" : "" }"
        style="
            width: {spriteWidth}px;
            height: {spriteHeight}px;
            background-image: url('{spriteSheet}');
            background-position: -{frameShield * spriteWidth}px 0;"
    ></div>
<!---------- SPECIAL EFFECTS ON WIN ---------->
    <div
        class="sprite-container effect { (match==="win") ? "activeEffect" : "" }"
        style="
            width: {effectWidth}px;
            height: {effectHeight}px;
            background-image: url('/media/animation/{effectSheet[indexPlayer]}');
            background-position: -{frameEffect * effectWidth}px 0;"
        >
    </div>
<!---------- SPECIAL EFFECTS ON LOSS ---------->
<div
    class="camEffect-container effect { (match==="loss") ? "activeEffect" : "" }"
    style="
        width: {effectWidth}px;
        height: {effectHeight}px;
        background-image: url('/media/animation/{effectSheet[index]}');
        background-position: -{frameEffect * effectWidth}px 0;"
    >
</div>


    <style>
        /* MONSTER CARD */
        #skully{
            margin-left: 10px;
        }
        .card-header{
            background-color: rgba(70, 9, 92, 0.163);
            border-radius: 10px;
            border: rgba(0, 0, 0, 0.101) 1px solid;
            padding: 5px !important;
        }
        .monster-card{
            position: absolute;
            left: 50%;
            top: 50%;
            text-align: center;
            transform: translate(-50%, calc(-50% - 30px));
            width: 250px;
            background-color: rgba(255, 255, 255, 0.716);
            box-shadow: -20px 20px 15px rgba(0, 0, 0, 0.168);
        }
        .hearts{
            display: flex;
            flex-flow: row wrap;
            justify-content: center;
        }
        /* MONSTER CARD */
        .sprite-container {
            background-repeat: no-repeat;
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%, calc(-50% - 25px));
        }
        .camEffect-container {
            background-repeat: no-repeat;
            position: absolute;
            left: 0%;
            top: 0%;
            transform: translate(-80px, -20%);
        }
        .effect{
            display: none;
        }
        .activeEffect{
            display: block;
        }
        *{
            font-family: 'Pixelify Sans', sans-serif;
            color: rgb(38, 36, 47);
        }
        @media(max-width:400px){
            .monster-card{
                top: 55% !important;
            }
            .sprite-container {
                top: 55% !important;
            }
        }
    </style>