<template>
    <div class="flex-item current-project-view-panel">
        <div v-if="selectedProject" class="project-view-contents layout-full-size layout-flex-rows-container">

            <div class="layout-flex-row">
                <div class="screen-header">
                    <div class="project-status" v-bind:class="{ online: status == 'online' }"></div>
                    <h3>{{ selectedProject.name }}</h3>
                    <h4>{{ selectedProject.client }}</h4>

                    <a class="header-icon settings-link">{{ $t('projects.settings_link') }}</a>
                    <a class="header-icon star-indicator" v-on:click="toggleStar()" v-bind:class="{ starred: selectedProject.starred }"></a>
                </div>

                <div class="control-toolbar clearfix">
                    <a
                        class="btn btn-default"
                        btn-icon="start"
                        v-bind:disabled="isStartButtonDisabled"
                        @click.stop="startServer"
                        href="#"
                    >{{ $t('projects.start') }}</a>

                    <a
                        class="btn btn-default"
                        btn-icon="stop"
                        v-bind:disabled="isStopButtonDisabled"
                        @click.stop="stopServer"
                        href="#"
                    >{{ $t('projects.stop') }}</a>

                    <a
                        class="btn btn-primary pull-right"
                        btn-icon="deploy"
                        href="#"
                        @click.stop="deploy"
                    >{{ $t('projects.deploy') }}</a>
                </div>

                <div class="standard-panel-paddings styled-text-document" data-open-links-in-browser v-html="description"></div>

                <div class="standard-panel-paddings standard-padding-bottom">
                    <table class="table attribute-table" data-open-links-in-browser>
                        <tr>
                            <th>{{ $t('projects.server') }}</th>
                            <td>{{ $t(environmentType) }}</td>
                        </tr>
                        <tr>
                            <th>{{ $t('projects.location') }}</th>
                            <td><a href="#" @click.prevent="openProjectFolder">{{ selectedProject.location }}</a></td>
                        </tr>
                        <tr>
                            <th>{{ $t('projects.local') }}</th>
                            <td>
                                <a v-bind:href="localUrl" v-bind:class="{ 'text-muted': status != 'online' }">{{ localUrl }}</a>,
                                <a v-bind:href="localBackendUrl" v-bind:class="{ 'text-muted': status != 'online' }">{{ localBackendUrl }}</a>
                            </td>
                        </tr>
                        <tr>
                            <th>{{ $t('projects.production') }}</th>
                            <td><a v-bind:href="productionUrl">{{ productionUrl }}</a></td>
                        </tr>
                    </table>
                </div>
            </div>

            <div class="layout-flex-row layout-stretch layout-flex-rows-container min-height-200">
                <div class="material-tabs layout-flex-row layout-stretch layout-flex-rows-container">
                    <ul class="tabs layout-flex-row">
                        <li class="active"><a href="#tab-server-log" data-toggle="tab">{{ $t('projects.server_log') }}</a></li>
                        <li><a href="#tab-application-log" data-toggle="tab">{{ $t('projects.application_log') }}</a></li>
                        <li><a href="#tab-php-error-log" data-toggle="tab">{{ $t('projects.php_error_log') }} <!-- <span class="badge badge-error">4</span> --></a></li>
                    </ul>

                    <div class="tab-content layout-flex-row layout-stretch layout-flex-rows-container layout-relative">
                        <div role="tabpanel" class="tab-pane active layout-full-size" id="tab-server-log">
                            <log-panel v-bind:log="selectedProject.runtime.serverLog"></log-panel>
                        </div>
                        <div role="tabpanel" class="tab-pane layout-full-size" id="tab-application-log">
                            <log-panel v-bind:log="selectedProject.runtime.applicationLog"></log-panel>
                        </div>
                        <div role="tabpanel" class="tab-pane layout-full-size" id="tab-php-error-log">
                            <log-panel v-bind:log="selectedProject.runtime.phpErrorLog"></log-panel>
                        </div>
                    </div>
                </div>
            </div>

        </div>
        <div v-else class="no-project-selected">
            <div>
                <h3>{{ $t('projects.no_project_selected') }}</h3>
                <p>
                    {{ $t('projects.select_project_or') }} 
                    <router-link to="/new-project">{{ $t('projects.create_new_project') }}</router-link>
                </p>
            </div>
        </div>
    </div>
</template>

<script>
import LogPanel from './log-panel.vue'

import marked from 'marked'
import environments from '../environments'
import environmentStatus from '../environments/status'

import DeployTest from '../remote/deployment/deploy-test'

export default {
    computed: {
        selectedProject () {
            return this.$store.state.projects.selectedProject
        },
        description () {
            if (!this.selectedProject) {
                return ''
            }

            return this.selectedProject.description === undefined ? '' : marked(this.selectedProject.description)
        },
        environmentType () {
            if (!this.selectedProject) {
                return ''
            }

            return environments.get(this.selectedProject).getEnvironmentTypeName()
        },
        localUrl () {
            return environments.get(this.selectedProject).getLocalUrl()
        },
        localBackendUrl () {
            return this.localUrl + '/backend/'
        },
        productionUrl () {
            return 'http://landing.acme.com'
        },
        status () {
            return this.selectedProject.runtime.status
        },
        isStopButtonDisabled () {
            return this.status == environmentStatus.OFFLINE || this.status == environmentStatus.STARTING || this.status == environmentStatus.STOPPING
        },
        isStartButtonDisabled () {
            return this.status == environmentStatus.ONLINE || this.status == environmentStatus.STARTING || this.status == environmentStatus.STOPPING
        }
    },
    methods: {
        toggleStar () {
            this.$store.dispatch('toggleProjectStar', this.selectedProject)
        },
        startServer () {
            if (this.status != environmentStatus.OFFLINE) {
                return
            }

            this.$store.dispatch('startServer', this.selectedProject)
        },
        stopServer () {
            if (this.status != environmentStatus.ONLINE) {
                return
            }

            this.$store.dispatch('stopServer', this.selectedProject)
        },
        openProjectFolder () {
            const {shell} = require('electron')
            shell.showItemInFolder(this.selectedProject.location)
        },
        deploy () {
            // This is temporary

            const deployTest = new DeployTest()
            deployTest.deploy()
        }
    },
    components: {
        LogPanel
    }
}
</script>
