<template>
    <div>
        <b-jumbotron>
            <h1>🍒 DAO Osaka 🌸</h1>
            <div class="row" v-if="daoContract && daoStatics">
                <div class="col-12 col-sm-6">
                    <div>
                        <span class="text-muted">DAO contract:</span> <code>{{ daoContract.address }}</code>
                    </div>
                    <div>
                        <span class="text-muted">Guild Bank:</span> <code>{{ daoStatics.guildBank }}</code><br/>
                    </div>
                    <div>
                        <span class="text-muted">Approved Token:</span> <code>{{ daoStatics.approvedToken }}</code>
                    </div>
                    <div>
                        <span class="text-muted">Total Shares:</span> <span class="badge badge-success ml-2 pl-2 pr-2">{{ daoStatics.totalShares }}</span>
                    </div>
                    <div v-if="proposalsQueueLength">
                        <span class="text-muted">Queue Length:</span> <span class="badge badge-warning ml-2 pl-2 pr-2">{{ proposalsQueueLength }}</span>
                    </div>
                    <div>
                        <span class="text-muted">Current Period:</span> <strong class="ml-2">{{ daoStatics.currentPeriod }}</strong>
                    </div>
                </div>
                <div class="col-12 col-sm-6">
                    <div>
                        <span class="text-muted">Summoning Time:</span> {{ daoStatics.summoningTime | moment('from') }}
                    </div>
                    <div>
                        <span class="text-muted">Period Duration:</span> {{ daoStatics.periodDuration }} secs ({{ daoStatics.periodDuration / 60 / 60 }} hours)
                    </div>
                    <div>
                        <span class="text-muted">Voting Period Length:</span> {{ daoStatics.votingPeriodLength }} ({{ daoStatics.periodDuration * daoStatics.votingPeriodLength / 60 / 60 }} hours)
                    </div>
                    <div>
                        <span class="text-muted">Grace Period Length:</span> {{ daoStatics.gracePeriodLength }} ({{ daoStatics.periodDuration * daoStatics.gracePeriodLength / 60 / 60 }} hours)
                    </div>
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
                <div class="card mb-4">
                    <div class="card-body">
                        <div class="card-header" :class="{'bg-minty': proposal[4] > 0, 'bg-yellowy': !proposal[6]  }">
                            <div class="row">
                                <div class="col text-dark">#{{ proposal[12] }}</div>
                                <div class="col text-right">{{ proposal[10] }}</div>
                            </div>
                        </div>

                        <div class="m-2">
                            <p class="card-title small">Applicant: <code>{{ proposal[1] }}</code></p>

                            <div class="row mb-4 mt-4">
                                <div class="col">{{ proposal[9] | toUnit }} {{ unit }}</div>
                                <div class="col text-right">{{ proposal[2] }} Shares</div>
                            </div>

                            <div class="row text-center mb-4">
                                <div class="col">
                                    <span class="badge" :class="{'badge-success': proposal[4] > 0}">YES: <span>{{ proposal[4] }}</span></span>
                                </div>
                                <div class="col">
                                    <span class="badge" :class="{'badge-danger': proposal[5] > 0}">NO: <span>{{ proposal[5] }}</span></span>
                                </div>
                                <div class="col" v-if="proposal[6] > 0">
                                    <span class="badge badge-success">PROCESSED</span>
                                </div>
                                <div class="col" v-else>
                                    <span class="badge badge-warning">UNPROCESSED</span>
                                </div>
                                <div class="col" v-if="proposal[7]">
                                    <span class="badge badge-info">PASS</span>
                                </div>
                                <div class="col" v-if="proposal[8]">
                                    <span class="badge badge-danger">ABORTED</span>
                                </div>
                            </div>

                            <div class="row mb-2 small">
                                <div class="col">Start: <code>{{ proposal[3] }}</code></div>
                                <div class="col text-center">Vote End: <code>{{ parseInt(proposal[3]) + parseInt(daoStatics.votingPeriodLength) }}</code></div>
                                <div class="col text-right">Grace End: <code>{{ parseInt(proposal[3]) + parseInt(daoStatics.votingPeriodLength) + parseInt(daoStatics.gracePeriodLength) }}</code></div>
                            </div>

                            <div class="row mb-4 small" v-if="!proposal[6]">
                                <div class="col">Periods Left: <code>{{ parseInt(proposal[3]) + parseInt(daoStatics.votingPeriodLength) + parseInt(daoStatics.gracePeriodLength) - daoStatics.currentPeriod }}</code></div>
                                <div class="col text-center"><span class="badge badge-danger" v-if="proposal[13]">Voting closed</span></div>
                                <div class="col text-right">Approx Hours Left: <code>{{ (parseInt(proposal[3]) + parseInt(daoStatics.votingPeriodLength) + parseInt(daoStatics.gracePeriodLength) - daoStatics.currentPeriod) * daoStatics.periodDuration / 60 / 60 }}</code></div>
                            </div>

                            <div class="small">
                                <div>Proposer: <code>{{ proposal[0] }}</code></div>
                                <div>Total @ YES: <code>{{ proposal[11] }}</code></div>
                            </div>
                        </div>
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
    import Spinner from '../../../blockcities-admin/src/components/Spinner';

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
            ]),
        },
        created: async function () {
            if (this.provider) {
                this.blocknumber = await this.provider.getBlockNumber();
            }

            if (this.daoContract) {
                this.getContractDetails(this.daoContract);
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
                    this.getContractDetails(daoContract);
                }
            },
        },
        methods: {
            async getContractDetails(doaContract) {
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
        },
        filters: {
            toUnit: function (value) {
                if (!value) return '';
                return utils.formatEther(utils.bigNumberify(value.toString()));
            }
        },
    };
</script>


