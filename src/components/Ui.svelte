<script lang="ts">
    import {NoirLight} from '@skeletonlabs/skeleton';
    export let index = 0;
    let images = ["earth.png", "water.png", "fire.png", "wind.png"]
</script>

<NoirLight />

<div class="ui">
    <slot> </slot>
    <div class="icons">
        {#each images as img, i}
            <div class="img-container">
                <img alt="monster" src="media/{img}" class="icon {(index === i) ? "selected":""}" style="filter: url({'#NoirLight'})">
            </div>
        {/each}
    </div>

</div>

<style>
    .icons{
        justify-content: right;
        display: flex;
        flex-flow: row;
    }
    img{
        height: 75px;
        width: auto;
        box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.12);
    }
    .ui{
        position: fixed;
        bottom: 0;
        right: 0;
        display: flex;
        flex-flow: column;
        justify-content: center;
        text-align: center;
        border-radius: 15px;
        /* border: 2px solid white; */
    }
    .img-container{
        margin: 10px;
    }
    .selected {
        filter: none !important;
    }
    
</style>