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
    const frameRate = 10;
    let animationInterval = 0;
    let frame = 0;

    // SFX
    const effectSheet = ["earthAni.png", "waterAni.png", "fireAni.png", "windAni.png"];
    const effectWidth = 320;
    const effectHeight = 320;
    let totalFramesEffect = [16, 13, 10, 9];
    let effectInterval = 0;

    function stopMonster() {
        clearInterval(animationInterval);
        fight.set(false);
    }

    function animateMonster() {
        animationInterval = setInterval(() => {
            frame = (frame + 1) % totalFrames;
            if (frame === 0) { stopMonster() }
        }, 1000 / frameRate);
    }

    function stopEffect() {
        clearInterval(effectInterval);
        fight.set(false);
    }

    function animateEffect() {
        effectInterval = setInterval(() => {
            frame = (frame + 1) % totalFramesEffect[indexPlayer];
            if (frame === 0) { stopEffect() }
        }, 1000 / frameRate);
    }

    $: if($fight){
        switch(match){
            case "tie":
                stopMonster();
                stopEffect();
                break
            case "win":
                animateEffect();
                break
            case "loss":
                animateMonster();
                break
        }
    }

    onMount(() => {
        return () => stopMonster();
    });
</script>

<style>
    .sprite-container {
        background-repeat: no-repeat;
        position: fixed;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
    }
</style>

<!------------- LISTENER -------------->


<!---------- MONSTER ---------->
<div
    class="sprite-container"
    style="
        width: {spriteWidth}px;
        height: {spriteHeight}px;"
> <img alt="cute monster image" src="/media/monster.png"> </div>
  
<!------------ SHIELD EFFECT ON TIE ---------->
<!-- if you lose -->
 {#if match === "loss"}
    <div
        class="sprite-container"
        style="
            width: {spriteWidth}px;
            height: {spriteHeight}px;
            background-image: url('{spriteSheet}');
            background-position: -{frame * spriteWidth}px 0;"
    ></div>
{:else if match === "win"}
    <!---------- SPECIAL EFFECTS ON WIN ---------->
    <div
        class="sprite-container"
        style="
            width: {effectWidth}px;
            height: {effectHeight}px;
            background-image: url('/media/animation/{effectSheet[indexPlayer]}');
            background-position: -{frame * effectWidth}px 0;"
        >
    </div>
    <!---------- SPECIAL EFFECTS ON LOSS ---------->
{/if}



