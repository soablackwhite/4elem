<script lang="ts">
    import {NoirLight} from '@skeletonlabs/skeleton';
    export let index = 0;
    let images = ["earth.png", "water.png", "fire.png", "wind.png"]
    let ringedClass = ["variant-ringed-success", "variant-ringed-tertiary", "variant-ringed-error", "variant-ringed-primary"]
</script>

<NoirLight />

<div class="ui">
    <slot> </slot>
    <div class="icons">
        {#each images as img, i}
            <div class="img-container">
                <img alt="monster" src="media/{img}" class="{ringedClass[i]} icon {(index === i) ? "selected":""}" style="filter: url({'#NoirLight'})">
            </div>
        {/each}
    </div>

</div>

<style>
    .icons{
        justify-content: right;
        display: flex;
        flex-flow: row;
        border: 1px rgba(255, 255, 255, 0.12) solid;
        border-radius: 10px;
    }
    img{
        height: 75px;
        width: auto;
        border-radius: 10px;
    }
    .ui{
        position: fixed;
        bottom: 0;
        right: 0;
        display: flex;
        flex-flow: column;
        justify-content: center;
        text-align: center;
    }
    .img-container{
        position: relative;
        margin: 10px;
        z-index: 1000 !important;
    }
    .img-container:hover{
        box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.52);
    }
    .selected {
        filter: none !important;
    }
    
</style>