<template>
    <div>
        <span class="badge badge-warning">{{ chain.name }} [{{chain.chainId}}]</span>
    </div>
</template>

<script>
    export default {
        name: 'network-badge',
        props: ['chain']
    };
</script>

<style lang="scss" scoped>
    .badge {
        font-size: 1.5rem;
    }
</style>
