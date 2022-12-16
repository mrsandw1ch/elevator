<template lang="">
    <div
        class="shaft">
        <h2>Шахта №{{ id + 1 }}</h2>
        <div
            class="cabin"
            :style="{transition: 'bottom ' + travelTime + 'ms ease-in-out',
                    backgroundColor: status === 'idle' ? 'green' : status === 'rest' ? 'orange' : 'red'}">
            <p>{{ status }}</p>
            <p>{{ currentFloor }}</p>
        </div>
    </div>
</template>

<script>

export default {
    name: 'Shaft',
    props: {
        id: Number,
        status: String,
        currentFloor: Number,
        travelTime: Number
    }
}
</script>

<style lang="scss">
.shaft {
    border: 1px solid;
    height: 100%;
    width: $shaft-width;

    h2 {
        opacity: 0.3;
        position: relative;
        top: -100px;
    }

    .cabin {
        position: absolute;
        width: $shaft-width;
        height: $floor-height;
        bottom: calc((v-bind(currentFloor) - 1) * #{$floor-height});
        border: 1px solid;

        p {
            color: white;
        }
    }
}
</style>