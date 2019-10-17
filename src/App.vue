<template>
    <div>
        <b-navbar toggleable="lg" type="dark" variant="info">
            <b-navbar-brand to="/">🍒 <code>the-dogs-molochs</code></b-navbar-brand>

            <network-badge :chain="chain" v-if="chain"></network-badge>

            <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

            <b-collapse id="nav-collapse" is-nav>

                <!-- Right aligned nav items -->
                <b-navbar-nav class="ml-auto">
                    <b-nav-item-dropdown text="DAOs" right>
                        <b-dropdown-item v-for="dao in daos"  v-if="daos" @click="dispatchDao(dao.key)" v-bind:key="dao.key">
                            {{ dao.name }}
                        </b-dropdown-item>
                    </b-nav-item-dropdown>
                </b-navbar-nav>
            </b-collapse>
        </b-navbar>
        <div class="container-fluid mt-4">
            <router-view/>
        </div>

        <footer class="m-5">
            <div class="row">
                <div class="col ">
                    <a href="https://www.urbandictionary.com/define.php?term=Dog%27s%20Bollocks" target="_blank" class="small">Why is this called 'the-dogs-molochs'?</a>
                </div>
                <div class="col text-right">
                    <a href="https://blockrocket.tech" target="_blank">Built by <img src="./assets/BR.png" style="max-height: 50px"/></a>
                </div>
            </div>
        </footer>
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
                'daos',
            ]),
        },
        methods: {
            dispatchDao(daoName) {
                console.log(`Switching to ${daoName}`);

                store.dispatch('daoContract', daoName);
            },
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
    @import url('https://fonts.googleapis.com/css?family=Roboto+Mono&display=swap');

    $enable-rounded: false;

    @import '../node_modules/bootstrap/scss/bootstrap';


    body {
        font-family: 'Roboto Mono', monospace;
    }

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

    .bg-tomato {
        background-color: #FF6347;
    }
</style>
