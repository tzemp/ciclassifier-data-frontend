<template>
    <b-container class="mt-5">
        <div class="progress-wrapper mb-5" v-if="this.$store.state.counter < this.amountOfClassifications">
            <div class="text-center">
                <p v-if="false" class="small mb-1">Evaluation {{ this.$store.state.counter }} out of {{
                    this.amountOfClassifications
                    }}.</p>
            </div>
            <b-progress :value="this.$store.state.counter" :max="this.amountOfClassifications" show-value
                        animated></b-progress>
        </div>

        <div class="loading-container" v-if="loading">
            <font-awesome-icon class="loading-icon" spin :icon="['fas', 'spinner']"/>
        </div>
        <div v-else>
            <b-row>
                <b-col cols="8">
                    <h1 class="mb-5">{{ project.name }}</h1>
                    <table class="table table-striped">
                        <thead/>
                        <tbody>
                        <tr>
                            <th>Description</th>
                            <td>{{ project.description }}</td>
                        </tr>
                        <tr>
                            <th>Language</th>
                            <td>{{ project.language }}</td>
                        </tr>
                        <tr>
                            <th>Commits</th>
                            <td>{{project.commits}}</td>
                        </tr>
                        <tr>
                            <th>Forks</th>
                            <td>{{project.forks}}</td>
                        </tr>
                        <tr>
                            <th>Stars</th>
                            <td>{{project.stars}}</td>
                        </tr>
                        <tr>
                            <th>Last Change</th>
                            <td>{{ project.lastChangeAt | moment }}</td>
                        </tr>
                        <tr>
                            <th>GitHub</th>
                            <td>
                                <a :href="gitHubUrl" target="_blank" class="view-file mr-5">
                                    <font-awesome-icon class="mr-1" :icon="['fas', 'share']"/>
                                    GitHub Repository
                                </a></td>
                        </tr>
                        </tbody>
                    </table>
                </b-col>
                <b-col cols="3" offset="1">
                    <div class="help-box">
                        <div class="box-header d-flex">
                            <font-awesome-icon class="mr-2" :icon="['fas', 'info-circle']"/>
                            <h6>What to do?</h6>
                        </div>
                        <div class="box-body">
                            <p>We would like to know what you think about the linked project and the adopted CI
                                practices. Please check the repository, the TravisCI configuration, as well as the
                                TravisCI page and then answer the questions below."</p>
                        </div>
                    </div>
                </b-col>
            </b-row>

            <div class="travis-ci-part mt-5">
                <h3 class="mb-3">Travis CI</h3>
                <div class="travis-status-link mb-3">
                    <a target="_Blank" :href="travisCiUrl">
                        <img style="margin-top: -3px;" class="travis-status mr-2" :src="travisCiStatus"
                             alt="Travis-CI Build Status">
                        <span><font-awesome-icon class="mr-1" :icon="['fas', 'share']"/>Project Site on Travis-CI</span>
                    </a>
                </div>

                <div class="configuration mb-3">
                    <div class="configuration-header">
                        <font-awesome-icon class="mr-1" :icon="['fas', 'cogs']"/>
                        <strong>Build Configuration</strong>
                        <a :href="configurationUrl" target="_blank" class="view-file">
                            <font-awesome-icon class="mr-1" :icon="['fas', 'share']"/>
                            Raw Configuration File
                        </a>
                    </div>
                    <div class="configuration-body">
                        <pre>{{ project.configuration }}</pre>
                    </div>
                </div>
            </div>
            <hr class="mb-5">
            <div class="mb-5">
                <h2 class="mb-3">Evaluation</h2>
                <div :key="slug" v-for="(question, slug) in this.questions" class="question">
                    <div class="question-wrapper">
                        <div class="text-left question-title">
                            <h4>{{ question }}</h4>
                        </div>
                    </div>
                    <div class="radio-wrapper">
                        <div :class="{active: evaluation[slug] === true}" class="radio-tile yes"
                             v-on:click="toggle(slug, true)">
                            <font-awesome-icon :icon="['fas', 'check']"/>
                        </div>
                    </div>
                    <div class="radio-wrapper">
                        <div :class="{active: evaluation[slug] === false}" class="radio-tile no"
                             v-on:click="toggle(slug, false)">
                            <font-awesome-icon :icon="['fas', 'times']"/>
                        </div>
                    </div>
                </div>
                <div class="text-right mt-5">
                    <button :disabled="!isReady" v-on:click="classify" class="btn btn-lg btn-primary">Next Page
                    </button>
                </div>
            </div>
        </div>
    </b-container>
</template>

<script>
  import axios from 'axios';
  import moment from 'moment';

  export default {
    layout: 'default',
    data() {
      return {
        amountOfClassifications: 10,
        apiUrl: process.env.apiUrl,
        loading: true,
        project: {},
        classification: {},
        questions: {
          serious: 'Do you think the linked repository contains a serious software project (in contrast to, for example, study projects or examples)?',
          integrated: 'Do you get the impression that TravisCI is an important part of the project’s development process?',
          tailored: 'Does the TravisCI configuration look customized to the project\'s needs, i.e., is it more than a template or a stub?',
          interesting: 'Is this TravisCI configuration interesting for you or could it be interesting for others, for example, to learn something new?',
          include: 'Would you say that this TravisCI configuration is a good example of a genuine CI pipeline?',
        },
        evaluation: {
          serious: null,
          tailored: null,
          integrated: null,
          interesting: null,
          include: null,
        },
      };
    },
    async fetch() {
      this.loadProject();
    },
    methods: {
      reset: function() {
        this.classification = {};
        this.project = {};
        this.evaluation = {
          serious: null,
          tailored: null,
          integrated: null,
          interesting: null,
          include: null,
        };
      },
      loadProject: function() {
        let that = this;
        axios.get(that.apiUrl + '/api/project_classification/next', {
          headers: {
            'Content-Type': 'application/json+ld',
            'Authorization': that.$store.state.user.accessKey,
          },
        }).then(function(response) {
          // we are finished :-D
          if (that.$store.state.counter === that.amountOfClassifications && !that.$store.state.hasSeenFinishScreen) {
            that.$router.push('/thanks');
            return;
          }
          if (response.data === null) {
            that.$router.push('/finish');
            return;
          }
          that.classification = response.data;
          if (that.classification.project) {
            axios.get(that.apiUrl + that.classification.project, {
              headers: {
                'Content-Type': 'application/json+ld',
                'Authorization': that.$store.state.user.accessKey,
              },
            }).then(function(response) {
              that.project = response.data;
              that.loading = false;
            });
          }
        });
      },
      toggle: function(slug, state) {
        this.evaluation[slug] = state;
      },
      classify: function() {
        this.loading = true;
        const data = {...{'status': 'done'}, ...this.evaluation};
        let that = this;
        axios.patch(that.apiUrl + '/api/project_classifications/' + that.classification.id, data, {
          headers: {
            'Content-Type': 'application/merge-patch+json',
            'Authorization': that.$store.state.user.accessKey,
          },
        }).then(function(response) {
          that.reset();
          that.$store.commit('INCREMENT_COUNTER');
          that.loadProject();
        });
      },
    },
    computed: {
      isReady: function() {
        return this.evaluation.serious !== null
            && this.evaluation.tailored !== null
            && this.evaluation.integrated !== null
            && this.evaluation.interesting !== null
            && this.evaluation.include !== null;
      },
      gitHubUrl: function() {
        return 'https://github.com/' + this.project.name;
      },
      configurationUrl: function() {
        return 'https://github.com/' + this.project.name + '/blob/' + this.project.defaultBranch + '/.travis.yml';
      },
      travisCiUrl: function() {
        return 'https://travis-ci.' + this.project.travisDestination + '/' + this.project.name;
      },
      travisCiStatus: function() {
        return 'https://api.travis-ci.' + this.project.travisDestination + '/' + this.project.name +
            '.svg?branch=' + this.project.defaultBranch;
      },
    },
    filters: {
      moment: function(date) {
        return moment(date).format('MMMM Do YYYY, h:mm:ss a');
      },
    },
  };
</script>

<style lang="scss">
    .question {
        margin-bottom: 1rem;
        display: flex;
        justify-content: space-around;
        align-items: stretch;
    }

    .question-wrapper {
        flex-grow: 1;
    }

    .radio-wrapper {
        width: 4rem;
        flex: 0 0 4rem;
    }

    .question-title {
        display: flex;
        align-items: center;
        height: 100%;

        h4 {
            font-size: 1.25rem;
            margin-bottom: 0;
        }
    }

    .travis-status {
        height: 25px;
    }

    .radio-tile {
        float: right;
        width: 3rem;
        height: 3rem;
        display: flex;
        align-items: center;
        justify-content: center;
        opacity: 0.5;
        cursor: pointer;

        svg {
            color: white;
            font-size: 1rem;

        }

        &.active {
            opacity: 1;
        }
    }

    .radio-tile.yes {
        background-color: #27ae60;
    }

    .radio-tile.no {
        background-color: #c0392b;
    }

    .loading-container {
        width: 100%;
        height: 50vh;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .loading-icon {
        font-size: 100px;
    }

    .card {
        flex-direction: row;
        align-items: center;

        &:hover {
            background: lightgray;
        }

        img {
            width: 50px;
            padding: 10px;
            margin-right: 10px;
        }

        span {
            font-size: 18px;
            font-weight: 700;
            color: black;
            text-decoration: none;
        }
    }

    .button-wrapper {
        width: 250px;
        margin: 0 auto;

        .button-group {
            display: flex;

            .btn {
                width: 100%;
            }
        }
    }

    a:hover {
        text-decoration: none;
    }

    .configuration-header {
        padding: 0.75rem 1rem;
        border: 1px solid #34495e;
        border-bottom: 0;
        background-color: #34495e;
        color: white;

        svg {
            color: white;
        }

        .view-file {
            float: right;
            color: white;
        }
    }

    .configuration-body {
        height: 20rem;
        overflow-y: scroll;
        border: 1px solid #34495e;
        padding: 1rem;
        background: #ecf0f1;
    }

    .help-box {
        background-color: #e67e22;
        padding: 15px;

        * {
            color: white;
        }
    }
</style>
