<template>
    <div>
        <b-navbar toggleable="lg" type="dark" variant="info">
            <b-navbar-brand to="/">🍒</b-navbar-brand>



            <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

            <b-collapse id="nav-collapse" is-nav>

                <!-- Right aligned nav items -->
                <b-navbar-nav class="ml-auto">
                    <network-badge :chain="chain" v-if="chain"></network-badge>
                </b-navbar-nav>
            </b-collapse>
        </b-navbar>
        <div class="container-fluid mt-4">
            <router-view/>
        </div>
    </div>
</template>

<script>
    import { ethers } from 'ethers';
    import NetworkBadge from './components/NetworkBadge';
    import store from './store';
    import { mapGetters } from 'vuex';

    export default {
        components: {NetworkBadge},
        computed: {
            ...mapGetters([
                'chain',
            ]),
        },
        created: async function () {
            try {
                await window.ethereum.enable();
                const provider = new ethers.providers.Web3Provider(web3.currentProvider);
                const signer = provider.getSigner();

                store.dispatch('provider', provider);
            } catch (e) {
                console.error(e);
            }
        },
    };
</script>

<style lang="scss">
    @import '../node_modules/bootstrap/scss/bootstrap';

    h1 {
        margin-bottom: 2rem;
    }

    .card-body {
        margin: 0;
        padding: 0;
    }

    .bg-minty {
        background-color: #C5E3BF;
    }

    .bg-yellowy {
        background-color: #FFF3D8;
    }
</style>
