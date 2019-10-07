<template>
    <div class="spinner-square mx-auto mt-5 mb-5">
        <div class="square-1 square"></div>
        <div class="square-2 square"></div>
        <div class="square-3 square"></div>
    </div>
</template>

<script>
    export default {
        name: 'spinner',
    };
</script>

<style scoped lang="scss">
    @import '../../node_modules/bootstrap/scss/bootstrap';

    .spinner-square {
        display: flex;
        flex-direction: row;
        width: 90px;
        height: 120px;
    }

    .spinner-square > .square {
        width: 17px;
        height: 80px;
        margin: auto auto;
        border-radius: 4px;
    }

    .square-1 {
        animation: square-anim 1200ms cubic-bezier(0.445, 0.05, 0.55, 0.95) 0s infinite;
    }

    .square-2 {
        animation: square-anim 1200ms cubic-bezier(0.445, 0.05, 0.55, 0.95) 200ms infinite;
    }

    .square-3 {
        animation: square-anim 1200ms cubic-bezier(0.445, 0.05, 0.55, 0.95) 400ms infinite;
    }

    @keyframes square-anim {
        0% {
            height: 80px;
            background-color: $pink;
        }
        20% {
            height: 80px;
        }
        40% {
            height: 120px;
            background-color: $red;
        }
        80% {
            height: 80px;
        }
        100% {
            height: 80px;
            background-color: $orange;
        }
    }
</style>

