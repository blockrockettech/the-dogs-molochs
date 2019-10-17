<template>
    <div>
        <span class="badge badge-secondary small">{{ chain.name }} [{{chain.chainId}}]</span>
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
        font-size: 1rem;
    }
</style>
