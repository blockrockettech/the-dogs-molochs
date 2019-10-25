<template>
    <div>
        <b-jumbotron>
            <h1 class="text-center">{{ doaName }}</h1>
            <div class="row" v-if="daoContract && daoStatics">
            <div class="col-12 col-sm-6">
                <div>
                    <span class="text-muted">Total Shares:</span><span class="badge badge-primary ml-2">{{ daoStatics.totalShares }}</span>
                </div>
                <div>
                    <span class="text-muted">Share price:</span><span class="badge badge-info ml-2">{{ (parseFloat(daoStatics.guildBankApprovedTokenBalance)  /  parseFloat(daoStatics.totalShares)) | toUnit }} {{ unit }}</span>
                </div>
                <div>
                    <span class="text-muted">DAO:</span> <code>{{ daoContract.address }}</code>
                </div>
                <div>
                    <span class="text-muted">DAO Balance:</span> <span class="badge badge-info ml-2">{{ daoStatics.daoBalance | toUnit }} {{ unit }}</span><br/>
                </div>
                <div>
                    <span class="text-muted">Approved Token:</span> <code>{{ daoStatics.approvedToken }}</code>
                </div>
                <hr/>
                <div>
                    <span class="text-muted">Guild Bank:</span> <code>{{ daoStatics.guildBank }}</code><br/>
                </div>
                <div>
                    <span class="text-muted">Guild Bank Balance:</span> <span class="badge badge-info ml-2">{{ daoStatics.guildBankApprovedTokenBalance | toUnit }} {{ unit }}</span><br/>
                </div>
                <hr/>
                <div v-if="proposalsQueueLength">
                    <span class="text-muted">Queue Length:</span> <span class="badge badge-secondary ml-2">{{ proposalsQueueLength }}</span>
                </div>
            </div>
            <div class="col-12 col-sm-6">
                <!--<div>-->
                    <!--<span class="text-muted">Summoning Time:</span> {{ daoStatics.summoningTime | moment('from') }}-->
                <!--</div>-->
                <div>
                    <span class="text-muted">Current Period:</span> {{ daoStatics.currentPeriod }}
                </div>
                <div>
                    <span class="text-muted">Period Duration:</span> {{ daoStatics.periodDuration }} <small>secs</small> ({{ 1 | toHrs(daoStatics.periodDuration) }} <small>hrs</small>)
                </div>
                <div>
                    <span class="text-muted">Voting Period Length:</span> {{ daoStatics.votingPeriodLength }} ({{  daoStatics.votingPeriodLength | toHrs(daoStatics.periodDuration) }} <small>hrs</small>)
                </div>
                <div>
                    <span class="text-muted">Grace Period Length:</span> {{ daoStatics.gracePeriodLength }} ({{ daoStatics.gracePeriodLength | toHrs(daoStatics.periodDuration) }} <small>hrs</small>)
                </div>
                <hr/>
                <div>
                    <span class="text-muted">Proposal Deposit:</span> {{ daoStatics.proposalDeposit | toUnit }} {{ unit }}
                </div>
                <div>
                    <span class="text-muted">Processing Reward:</span> {{ daoStatics.processingReward | toUnit }} {{ unit }}
                </div>

            </div>
        </div>
            <div v-else>
                <spinner></spinner>
            </div>
        </b-jumbotron>
        <div class="row" v-if="proposals && daoContract && daoStatics">
            <div class="col-sm-4 col-12" v-for="proposal in proposals">
                <div class="shadow card mb-4">
                    <div class="card-body">
                        <div class="card-header" :class="{'bg-minty': proposal[4] > 0, 'bg-yellowy': !proposal[6], 'bg-tomato': proposal[8] }">
                            <div class="row">
                                <div class="col text-dark"><span class="badge badge-dark">#{{ proposal[12] }}</span></div>
                                <div class="col text-right details"><span class="text-muted small">{{ proposal[10] }}</span></div>
                            </div>
                        </div>

                        <ul class="list-group list-group-flush">
                            <li class="list-group-item bg-light">
                                <p class="card-title small">Applicant: <code>{{ proposal[1] }}</code></p>
                            </li>
                            <li class="list-group-item">
                                <div class="row">
                                    <div class="col"><span class="badge badge-info">{{ proposal[9] | toUnit }} {{ unit }}</span></div>
                                    <div class="col text-right"><span class="badge badge-dark">{{ proposal[2] }} Shares</span></div>
                                </div>
                            </li>
                            <li class="list-group-item bg-light">
                                <div class="row">
                                    <div class="col">
                                        <span class="badge" :class="{'badge-success': proposal[4] > 0}">YES: <span>{{ proposal[4] }}</span></span>
                                    </div>
                                    <div class="col">
                                        <span class="badge" :class="{'badge-danger': proposal[5] > 0}">NO: <span>{{ proposal[5] }}</span></span>
                                    </div>
                                    <div class="col" v-if="proposal[6] > 0">
                                        <span class="badge badge-secondary">PROCESSED</span>
                                    </div>
                                    <div class="col" v-else>
                                        <span class="badge badge-warning">UNPROCESSED</span>
                                    </div>
                                    <div class="col" v-if="proposal[7]">
                                        <span class="badge badge-success">PASS</span>
                                    </div>
                                    <div class="col" v-if="proposal[8]">
                                        <span class="badge badge-danger">ABORTED</span>
                                    </div>
                                </div>
                            </li>
                            <li class="list-group-item">
                                <div class="row mb-2 small">
                                    <div class="col">Start: {{ proposal[3] }}</div>
                                    <div class="col text-center">End: {{ parseInt(proposal[3]) + parseInt(daoStatics.votingPeriodLength) }}</div>
                                    <div class="col text-right">Grace: {{ parseInt(proposal[3]) + parseInt(daoStatics.votingPeriodLength) + parseInt(daoStatics.gracePeriodLength) }}</div>
                                </div>
                                <div class="row mb-4 small" v-if="!proposal[6]">
                                    <div class="col">
                                        <span class="badge badge-danger" v-if="proposal[13]">Voting closed</span>
                                        <span v-else>
                                           Voting End: {{ (parseInt(proposal[3]) + parseInt(daoStatics.votingPeriodLength) - daoStatics.currentPeriod) | toHrs(daoStatics.periodDuration) }} <small>hrs</small>
                                        </span>
                                    </div>
                                    <div class="col text-right">
                                        Grace End: {{ (parseInt(proposal[3]) + parseInt(daoStatics.votingPeriodLength) + parseInt(daoStatics.gracePeriodLength) - daoStatics.currentPeriod) | toHrs(daoStatics.periodDuration) }} <small>hrs</small>
                                    </div>
                                </div>
                            </li>
                            <li class="list-group-item bg-light">
                                <div class="small">
                                    <div>Proposer: <code>{{ proposal[0] }}</code></div>
                                    <!--<div>Total @ YES: <code>{{ proposal[11] }}</code></div>-->
                                </div>
                            </li>
                            <li class="list-group-item small">
                                <code>{{ proposal[10] }}</code>
                            </li>
                            <li class="list-group-item small bg-dark" v-if="!proposal[6] && (parseInt(proposal[3]) + parseInt(daoStatics.votingPeriodLength) - daoStatics.currentPeriod) > 0">
                                <div class="row">
                                    <div class="col">
                                        <b-button variant="success" size="sm" v-on:click="vote(proposal[12], 1)">YES</b-button>
                                    </div>
                                    <div class="col text-center">
                                        <b-button variant="danger" size="sm" v-on:click="vote(proposal[12], 2)">NO</b-button>
                                    </div>
                                    <div class="col text-right">
                                        <b-button variant="warning" size="sm" v-on:click="vote(proposal[12], 0)">ABSTAIN</b-button>
                                    </div>
                                </div>
                            </li>
                            <li class="list-group-item small bg-dark" v-if="!proposal[6] && (parseInt(proposal[3]) + parseInt(daoStatics.votingPeriodLength) + parseInt(daoStatics.gracePeriodLength) - daoStatics.currentPeriod) <= 0">
                                <div class="row">
                                    <div class="col text-center">
                                        <b-button variant="light" size="sm" v-on:click="processProposal(proposal[12])">PROCESS PROPOSAL</b-button>
                                    </div>
                                </div>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
        <div v-else>
            <spinner></spinner>
        </div>
    </div>
</template>

<script>
    import { ethers, utils } from 'ethers';
    import NetworkBadge from '../components/NetworkBadge';
    import { mapGetters } from 'vuex';
    import Spinner from '../components/Spinner';

    export default {
        name: 'home',
        components: {Spinner, NetworkBadge},
        data() {
            return {
                blocknumber: null,
                lastSaleBlock: null,
                proposalsQueueLength: null,
                proposals: [],
                tx: null,
                form: {
                    recipient: null,
                    selectedSpecial: null,
                    specials: []
                }
            };
        },
        computed: {
            ...mapGetters([
                'rootApi',
                'daoContract',
                'daoStatics',
                'provider',
                'chain',
                'unit',
                'doaName',
                'accBalance',
            ]),
        },
        created: async function () {
            if (this.provider) {
                this.blocknumber = await this.provider.getBlockNumber();
            }

            if (this.daoContract) {
                this.buildProposals(this.daoContract);
            }
        },
        watch: {
            async provider(provider) {
                if (provider) {
                    this.blocknumber = await provider.getBlockNumber();
                }
            },
            async daoContract(daoContract) {
                if (daoContract) {
                    this.proposals = [];
                    this.buildProposals(daoContract);
                }
            },
        },
        methods: {
            async buildProposals(doaContract) {
                this.proposalsQueueLength = (await doaContract.getProposalQueueLength()).toString();

                const qLen = parseInt(this.proposalsQueueLength);
                if (qLen > 0) {
                    for (let i = 0; i < qLen; i++) {
                        let proposal = await doaContract.proposalQueue(i);
                        proposal.push(i); // add index

                        if (!proposal[6]) {
                            const proposalVotingExpired = await doaContract.hasVotingPeriodExpired(proposal[3]);
                            proposal.push(proposalVotingExpired);
                        }

                        this.proposals.push(proposal);
                    }
                }
            },

            async vote(index, voteVal) {
                console.log(`voting proposal ${index} with vote of ${voteVal}`);

                const tx = await this.daoContract.submitVote(index, voteVal);
                alert(`Transaction: ${JSON.stringify(tx)}`);

                const receipt = await tx.wait(1);
                alert(`Receipt: ${JSON.stringify(receipt)}`);
            },

            async processProposal(index) {
                console.log(`process proposal ${index}`);

                const tx = await this.daoContract.processProposal(index);
                alert(`Transaction: ${JSON.stringify(tx)}`);

                const receipt = await tx.wait(1);
                alert(`Receipt: ${JSON.stringify(receipt)}`);
            }
        },
        filters: {
            toUnit: function (value) {
                if (!value) return '';
                return utils.formatEther(utils.bigNumberify(value.toString()));
            },
            toHrs: function (value, periodInSecs) {
                if (!value) return '';
                return (parseFloat(value) * periodInSecs) / 60 / 60;
            },
        },
    };
</script>
<style lang="scss" scoped>
    .badge-lg {
        font-size: 1.5rem;
    }

    .details {
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }

    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s;
    }
    .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
        opacity: 0;
    }
</style>


