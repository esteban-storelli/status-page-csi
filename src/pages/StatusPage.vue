<template>
    <div v-if="loadedTheme" class="main-container mt-3">
        <!-- Sidebar for edit mode -->
        <div v-if="enableEditMode" class="sidebar">
            <div class="sidebar-body">
                <div class="my-3">
                    <label for="slug" class="form-label">{{ $t("Slug") }}</label>
                    <div class="input-group">
                        <span id="basic-addon3" class="input-group-text">/status/</span>
                        <input id="slug" v-model="config.slug" type="text" class="form-control">
                    </div>
                </div>

                <div class="my-3">
                    <label for="title" class="form-label">{{ $t("Title") }}</label>
                    <input id="title" v-model="config.title" type="text" class="form-control">
                </div>

                <!-- Description -->
                <div class="my-3">
                    <label for="description" class="form-label">{{ $t("Description") }}</label>
                    <textarea id="description" v-model="config.description" class="form-control"></textarea>
                    <div class="form-text">
                        {{ $t("markdownSupported") }}
                    </div>
                </div>

                <!-- Footer Text -->
                <div class="my-3">
                    <label for="footer-text" class="form-label">{{ $t("Footer Text") }}</label>
                    <textarea id="footer-text" v-model="config.footerText" class="form-control"></textarea>
                    <div class="form-text">
                        {{ $t("markdownSupported") }}
                    </div>
                </div>

                <div class="my-3">
                    <label for="switch-theme" class="form-label">{{ $t("Theme") }}</label>
                    <select id="switch-theme" v-model="config.theme" class="form-select">
                        <option value="auto">{{ $t("Auto") }}</option>
                        <option value="light">{{ $t("Light") }}</option>
                        <option value="dark">{{ $t("Dark") }}</option>
                    </select>
                </div>

                <div class="my-3 form-check form-switch">
                    <input id="showTags" v-model="config.showTags" class="form-check-input" type="checkbox">
                    <label class="form-check-label" for="showTags">{{ $t("Show Tags") }}</label>
                </div>

                <!-- Show Powered By -->
                <div class="my-3 form-check form-switch">
                    <input id="show-powered-by" v-model="config.showPoweredBy" class="form-check-input" type="checkbox">
                    <label class="form-check-label" for="show-powered-by">{{ $t("Show Powered By") }}</label>
                </div>

                <!-- Show certificate expiry -->
                <div class="my-3 form-check form-switch">
                    <input id="show-certificate-expiry" v-model="config.showCertificateExpiry" class="form-check-input" type="checkbox">
                    <label class="form-check-label" for="show-certificate-expiry">{{ $t("showCertificateExpiry") }}</label>
                </div>

                <div v-if="false" class="my-3">
                    <label for="password" class="form-label">{{ $t("Password") }} <sup>{{ $t("Coming Soon") }}</sup></label>
                    <input id="password" v-model="config.password" disabled type="password" autocomplete="new-password" class="form-control">
                </div>

                <!-- Domain Name List -->
                <div class="my-3">
                    <label class="form-label">
                        {{ $t("Domain Names") }}
                        <button class="p-0 bg-transparent border-0" :aria-label="$t('Add a domain')" @click="addDomainField">
                            <font-awesome-icon icon="plus-circle" class="action text-primary" />
                        </button>
                    </label>

                    <ul class="list-group domain-name-list">
                        <li v-for="(domain, index) in config.domainNameList" :key="index" class="list-group-item">
                            <input v-model="config.domainNameList[index]" type="text" class="no-bg domain-input" placeholder="example.com" />
                            <button class="p-0 bg-transparent border-0" :aria-label="$t('Remove domain', [ domain ])" @click="removeDomain(index)">
                                <font-awesome-icon icon="times" class="action remove ms-2 me-3 text-danger" />
                            </button>
                        </li>
                    </ul>
                </div>

                <!-- Google Analytics -->
                <div class="my-3">
                    <label for="googleAnalyticsTag" class="form-label">{{ $t("Google Analytics ID") }}</label>
                    <input id="googleAnalyticsTag" v-model="config.googleAnalyticsId" type="text" class="form-control">
                </div>

                <!-- Custom CSS -->
                <div class="my-3">
                    <div class="mb-1">{{ $t("Custom CSS") }}</div>
                    <prism-editor v-model="config.customCSS" class="css-editor" :highlight="highlighter" line-numbers></prism-editor>
                </div>

                <div class="danger-zone">
                    <button class="btn btn-danger me-2" @click="deleteDialog">
                        <font-awesome-icon icon="trash" />
                        {{ $t("Delete") }}
                    </button>
                </div>
            </div>

            <!-- Sidebar Footer -->
            <div class="sidebar-footer">
                <button class="btn btn-success me-2" :disabled="loading" @click="save">
                    <font-awesome-icon icon="save" />
                    {{ $t("Save") }}
                </button>

                <button class="btn btn-danger me-2" @click="discard">
                    <font-awesome-icon icon="undo" />
                    {{ $t("Discard") }}
                </button>
            </div>
        </div>

        <div :class="{ edit: enableEditMode}" class="main">
        <!-- Main Status Page
            <!.-- Logo & Title --.>
            <h1 class="mb-4 title-flex">
                <!-.- Logo --.>
                <span class="logo-wrapper" @click="showImageCropUploadMethod">
                    <img :src="logoURL" alt class=" mb-2" :class="logoClass" />
                    <font-awesome-icon v-if="enableEditMode" class="icon-upload" icon="upload" />
                </span>

                <!.-- Uploader --.>
                <!.--    url="/api/status-page/upload-logo" --.>
                <ImageCropUpload
                    v-model="showImageCropUpload"
                    field="img"
                    :width="128"
                    :height="128"
                    :langType="$i18n.locale"
                    img-format="png"
                    :noCircle="true"
                    :noSquare="false"
                    @crop-success="cropSuccess"
                />

                <!.-- Title --.>
                <Editable v-model="config.title" tag="span" :contenteditable="editMode" :noNL="true" />
            </h1> -->

            <div class="header">
                <div class="logo-and-actions">
                    <div class="logo">
                        <img src="/frame.svg">
                    </div>
                    <div class="actions">
                        <div class="modify">
                            <button class="unset" @click="edit">{{ $t("Edit Status Page") }}</button>
                            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none">
                                <path d="M9.24995 22L8.84995 18.8C8.63328 18.7167 8.42912 18.6167 8.23745 18.5C8.04578 18.3833 7.85828 18.2583 7.67495 18.125L4.69995 19.375L1.94995 14.625L4.52495 12.675C4.50828 12.5583 4.49995 12.4458 4.49995 12.3375V11.6625C4.49995 11.5542 4.50828 11.4417 4.52495 11.325L1.94995 9.375L4.69995 4.625L7.67495 5.875C7.85828 5.74167 8.04995 5.61667 8.24995 5.5C8.44995 5.38333 8.64995 5.28333 8.84995 5.2L9.24995 2H14.75L15.15 5.2C15.3666 5.28333 15.5708 5.38333 15.7625 5.5C15.9541 5.61667 16.1416 5.74167 16.325 5.875L19.3 4.625L22.05 9.375L19.475 11.325C19.4916 11.4417 19.5 11.5542 19.5 11.6625V12.3375C19.5 12.4458 19.4833 12.5583 19.45 12.675L22.025 14.625L19.275 19.375L16.325 18.125C16.1416 18.2583 15.95 18.3833 15.75 18.5C15.55 18.6167 15.35 18.7167 15.15 18.8L14.75 22H9.24995ZM12.05 15.5C13.0166 15.5 13.8416 15.1583 14.525 14.475C15.2083 13.7917 15.55 12.9667 15.55 12C15.55 11.0333 15.2083 10.2083 14.525 9.525C13.8416 8.84167 13.0166 8.5 12.05 8.5C11.0666 8.5 10.2375 8.84167 9.56245 9.525C8.88745 10.2083 8.54995 11.0333 8.54995 12C8.54995 12.9667 8.88745 13.7917 9.56245 14.475C10.2375 15.1583 11.0666 15.5 12.05 15.5ZM12.05 13.5C11.6333 13.5 11.2791 13.3542 10.9875 13.0625C10.6958 12.7708 10.55 12.4167 10.55 12C10.55 11.5833 10.6958 11.2292 10.9875 10.9375C11.2791 10.6458 11.6333 10.5 12.05 10.5C12.4666 10.5 12.8208 10.6458 13.1125 10.9375C13.4041 11.2292 13.55 11.5833 13.55 12C13.55 12.4167 13.4041 12.7708 13.1125 13.0625C12.8208 13.3542 12.4666 13.5 12.05 13.5ZM11 20H12.975L13.325 17.35C13.8416 17.2167 14.3208 17.0208 14.7625 16.7625C15.2041 16.5042 15.6083 16.1917 15.975 15.825L18.45 16.85L19.425 15.15L17.275 13.525C17.3583 13.2917 17.4166 13.0458 17.45 12.7875C17.4833 12.5292 17.5 12.2667 17.5 12C17.5 11.7333 17.4833 11.4708 17.45 11.2125C17.4166 10.9542 17.3583 10.7083 17.275 10.475L19.425 8.85L18.45 7.15L15.975 8.2C15.6083 7.81667 15.2041 7.49583 14.7625 7.2375C14.3208 6.97917 13.8416 6.78333 13.325 6.65L13 4H11.025L10.675 6.65C10.1583 6.78333 9.67912 6.97917 9.23745 7.2375C8.79578 7.49583 8.39162 7.80833 8.02495 8.175L5.54995 7.15L4.57495 8.85L6.72495 10.45C6.64162 10.7 6.58328 10.95 6.54995 11.2C6.51662 11.45 6.49995 11.7167 6.49995 12C6.49995 12.2667 6.51662 12.525 6.54995 12.775C6.58328 13.025 6.64162 13.275 6.72495 13.525L4.57495 15.15L5.54995 16.85L8.02495 15.8C8.39162 16.1833 8.79578 16.5042 9.23745 16.7625C9.67912 17.0208 10.1583 17.2167 10.675 17.35L11 20Z" fill="#0D0F13"/>
                            </svg>
                        </div>
                        <div class="dashboard">
                             <a class="unset" href="/manage-status-page">{{ $t("Go to Dashboard") }}</a>
                            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none">
                                <path d="M6 19H9V13H15V19H18V10L12 5.5L6 10V19ZM4 21V9L12 3L20 9V21H13V15H11V21H4Z" fill="#0D0F13"/>
                            </svg>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Admin functions -->
            <div v-if="hasToken" class="mb-4">
                <div v-if="!enableEditMode">
                    <button class="btn btn-info me-2" @click="edit">
                        <font-awesome-icon icon="edit" />
                        {{ $t("Edit Status Page") }}
                    </button>

                    <a href="/manage-status-page" class="btn btn-info">
                        <font-awesome-icon icon="tachometer-alt" />
                        {{ $t("Go to Dashboard") }}
                    </a>
                </div>

                <div v-else>
                    <button class="btn btn-primary btn-add-group me-2" @click="createIncident">
                        <font-awesome-icon icon="bullhorn" />
                        {{ $t("Create Incident") }}
                    </button>
                </div>
            </div>

            <!-- Incident -->
            <div v-if="incident !== null" class="shadow-box alert mb-4 p-4 incident" role="alert" :class="incidentClass">
                <strong v-if="editIncidentMode">{{ $t("Title") }}:</strong>
                <Editable v-model="incident.title" tag="h4" :contenteditable="editIncidentMode" :noNL="true" class="alert-heading" />

                <strong v-if="editIncidentMode">{{ $t("Content") }}:</strong>
                <Editable v-if="editIncidentMode" v-model="incident.content" tag="div" :contenteditable="editIncidentMode" class="content" />
                <div v-if="editIncidentMode" class="form-text">
                    {{ $t("markdownSupported") }}
                </div>
                <!-- eslint-disable-next-line vue/no-v-html-->
                <div v-if="! editIncidentMode" class="content" v-html="incidentHTML"></div>

                <!-- Incident Date -->
                <div class="date mt-3">
                    {{ $t("Date Created") }}: {{ $root.datetime(incident.createdDate) }} ({{ dateFromNow(incident.createdDate) }})<br />
                    <span v-if="incident.lastUpdatedDate">
                        {{ $t("Last Updated") }}: {{ $root.datetime(incident.lastUpdatedDate) }} ({{ dateFromNow(incident.lastUpdatedDate) }})
                    </span>
                </div>

                <div v-if="editMode" class="mt-3">
                    <button v-if="editIncidentMode" class="btn btn-light me-2" @click="postIncident">
                        <font-awesome-icon icon="bullhorn" />
                        {{ $t("Post") }}
                    </button>

                    <button v-if="!editIncidentMode && incident.id" class="btn btn-light me-2" @click="editIncident">
                        <font-awesome-icon icon="edit" />
                        {{ $t("Edit") }}
                    </button>

                    <button v-if="editIncidentMode" class="btn btn-light me-2" @click="cancelIncident">
                        <font-awesome-icon icon="times" />
                        {{ $t("Cancel") }}
                    </button>

                    <div v-if="editIncidentMode" class="dropdown d-inline-block me-2">
                        <button id="dropdownMenuButton1" class="btn btn-secondary dropdown-toggle" type="button" data-bs-toggle="dropdown" aria-expanded="false">
                            {{ $t("Style") }}: {{ $t(incident.style) }}
                        </button>
                        <ul class="dropdown-menu" aria-labelledby="dropdownMenuButton1">
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'info'">{{ $t("info") }}</a></li>
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'warning'">{{ $t("warning") }}</a></li>
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'danger'">{{ $t("danger") }}</a></li>
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'primary'">{{ $t("primary") }}</a></li>
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'light'">{{ $t("light") }}</a></li>
                            <li><a class="dropdown-item" href="#" @click="incident.style = 'dark'">{{ $t("dark") }}</a></li>
                        </ul>
                    </div>

                    <button v-if="!editIncidentMode && incident.id" class="btn btn-light me-2" @click="unpinIncident">
                        <font-awesome-icon icon="unlink" />
                        {{ $t("Unpin") }}
                    </button>
                </div>
            </div>

            <!-- Overall Status -->
            <div class="shadow-box list  p-4 overall-status mb-4">
                <div v-if="Object.keys($root.publicMonitorList).length === 0 && loadedData">
                    <font-awesome-icon icon="question-circle" class="ok" />
                    {{ $t("No Services") }}
                </div>

                <template v-else>
                    <div v-if="allUp">
                        <font-awesome-icon icon="check-circle" class="ok" />
                        {{ $t("All Systems Operational") }}
                    </div>

                    <div v-else-if="partialDown">
                        <font-awesome-icon icon="exclamation-circle" class="warning" />
                        {{ $t("Partially Degraded Service") }}
                    </div>

                    <div v-else-if="allDown">
                        <font-awesome-icon icon="times-circle" class="danger" />
                        {{ $t("Degraded Service") }}
                    </div>

                    <div v-else-if="isMaintenance">
                        <font-awesome-icon icon="wrench" class="status-maintenance" />
                        {{ $t("maintenanceStatus-under-maintenance") }}
                    </div>

                    <div v-else>
                        <font-awesome-icon icon="question-circle" style="color: #efefef;" />
                    </div>
                </template>
            </div>

            <!-- Maintenance -->
            <template v-if="maintenanceList.length > 0">
                <div
                    v-for="maintenance in maintenanceList" :key="maintenance.id"
                    class="shadow-box alert mb-4 p-3 bg-maintenance mt-4 position-relative" role="alert"
                >
                    <h4 class="alert-heading">{{ maintenance.title }}</h4>
                    <!-- eslint-disable-next-line vue/no-v-html-->
                    <div class="content" v-html="maintenanceHTML(maintenance.description)"></div>
                    <MaintenanceTime :maintenance="maintenance" />
                </div>
            </template>

            <!-- Description -->
            <strong v-if="editMode">{{ $t("Description") }}:</strong>
            <Editable v-if="enableEditMode" v-model="config.description" :contenteditable="editMode" tag="div" class="mb-4 description" />
            <!-- eslint-disable-next-line vue/no-v-html-->
            <div v-if="! enableEditMode" class="alert-heading p-2" v-html="descriptionHTML"></div>

            <div v-if="editMode" class="mb-4">
                <div>
                    <button class="btn btn-primary btn-add-group me-2" @click="addGroup">
                        <font-awesome-icon icon="plus" />
                        {{ $t("Add Group") }}
                    </button>
                </div>

                <div class="mt-3">
                    <div v-if="sortedMonitorList.length > 0 && loadedData">
                        <label>{{ $t("Add a monitor") }}:</label>
                        <VueMultiselect
                            v-model="selectedMonitor"
                            :options="sortedMonitorList"
                            :multiple="false"
                            :searchable="true"
                            :placeholder="$t('Add a monitor')"
                            label="name"
                            trackBy="name"
                            class="mt-3"
                        >
                            <template #option="{ option }">
                                <div class="d-inline-flex">
                                    <span>{{ option.pathName }} <Tag v-for="tag in option.tags" :key="tag" :item="tag" :size="'sm'" /></span>
                                </div>
                            </template>
                        </VueMultiselect>
                    </div>
                    <div v-else class="text-center">
                        {{ $t("No monitors available.") }}  <router-link to="/add">{{ $t("Add one") }}</router-link>
                    </div>
                </div>
            </div>

            <div class="mb-4">
                <div v-if="$root.publicGroupList.length === 0 && loadedData" class="text-center">
                    <!-- 👀 Nothing here, please add a group or a monitor. -->
                    👀 {{ $t("statusPageNothing") }}
                </div>

                <PublicGroupList :edit-mode="enableEditMode" :show-tags="config.showTags" :show-certificate-expiry="config.showCertificateExpiry" />
            </div>

            <footer class="mt-5 mb-4">
                <div class="custom-footer-text text-start">
                    <strong v-if="enableEditMode">{{ $t("Custom Footer") }}:</strong>
                </div>
                <Editable v-if="enableEditMode" v-model="config.footerText" tag="div" :contenteditable="enableEditMode" :noNL="false" class="alert-heading p-2" />
                <!-- eslint-disable-next-line vue/no-v-html-->
                <div v-if="! enableEditMode" class="alert-heading p-2" v-html="footerHTML"></div>

                <p v-if="config.showPoweredBy">
                    {{ $t("Powered by") }} <a target="_blank" rel="noopener noreferrer" href="https://github.com/louislam/uptime-kuma">{{ $t("Uptime Kuma" ) }}</a>
                </p>

                <div class="refresh-info mb-2">
                    <div>{{ $t("Last Updated") }}:  {{ lastUpdateTimeDisplay }}</div>
                    <div>{{ $tc("statusPageRefreshIn", [ updateCountdownText]) }}</div>
                </div>
            </footer>
        </div>

        <Confirm ref="confirmDelete" btn-style="btn-danger" :yes-text="$t('Yes')" :no-text="$t('No')" @yes="deleteStatusPage">
            {{ $t("deleteStatusPageMsg") }}
        </Confirm>

        <component is="style" v-if="config.customCSS" type="text/css">
            {{ config.customCSS }}
        </component>
    </div>
</template>

<script>
import axios from "axios";
import dayjs from "dayjs";
import duration from "dayjs/plugin/duration";
import Favico from "favico.js";
// import highlighting library (you can use any library you want just return html string)
import { highlight, languages } from "prismjs/components/prism-core";
import "prismjs/components/prism-css";
import "prismjs/themes/prism-tomorrow.css"; // import syntax highlighting styles
import ImageCropUpload from "vue-image-crop-upload";
// import Prism Editor
import { PrismEditor } from "vue-prism-editor";
import "vue-prism-editor/dist/prismeditor.min.css"; // import the styles somewhere
import { useToast } from "vue-toastification";
import { marked } from "marked";
import DOMPurify from "dompurify";
import Confirm from "../components/Confirm.vue";
import PublicGroupList from "../components/PublicGroupList.vue";
import MaintenanceTime from "../components/MaintenanceTime.vue";
import { getResBaseURL } from "../util-frontend";
import { STATUS_PAGE_ALL_DOWN, STATUS_PAGE_ALL_UP, STATUS_PAGE_MAINTENANCE, STATUS_PAGE_PARTIAL_DOWN, UP, MAINTENANCE } from "../util.ts";
import Tag from "../components/Tag.vue";
import VueMultiselect from "vue-multiselect";

const toast = useToast();
dayjs.extend(duration);

const leavePageMsg = "Do you really want to leave? you have unsaved changes!";

// eslint-disable-next-line no-unused-vars
let feedInterval;

const favicon = new Favico({
    animation: "none"
});

export default {

    components: {
        PublicGroupList,
        ImageCropUpload,
        Confirm,
        PrismEditor,
        MaintenanceTime,
        Tag,
        VueMultiselect
    },

    // Leave Page for vue route change
    beforeRouteLeave(to, from, next) {
        if (this.editMode) {
            const answer = window.confirm(leavePageMsg);
            if (answer) {
                next();
            } else {
                next(false);
            }
        }
        next();
    },

    props: {
        /** Override for the status page slug */
        overrideSlug: {
            type: String,
            required: false,
            default: null,
        },
    },

    data() {
        return {
            slug: null,
            enableEditMode: false,
            enableEditIncidentMode: false,
            hasToken: false,
            config: {},
            selectedMonitor: null,
            incident: null,
            previousIncident: null,
            showImageCropUpload: false,
            imgDataUrl: "/icon.svg",
            loadedTheme: false,
            loadedData: false,
            baseURL: "",
            clickedEditButton: false,
            maintenanceList: [],
            autoRefreshInterval: 5,
            lastUpdateTime: dayjs(),
            updateCountdown: null,
            updateCountdownText: null,
            loading: true,
        };
    },
    computed: {

        logoURL() {
            if (this.imgDataUrl.startsWith("data:")) {
                return this.imgDataUrl;
            } else {
                return this.baseURL + this.imgDataUrl;
            }
        },

        /**
         * If the monitor is added to public list, which will not be in this list.
         */
        sortedMonitorList() {
            let result = [];

            for (let id in this.$root.monitorList) {
                if (this.$root.monitorList[id] && ! (id in this.$root.publicMonitorList)) {
                    let monitor = this.$root.monitorList[id];
                    result.push(monitor);
                }
            }

            result.sort((m1, m2) => {

                if (m1.active !== m2.active) {
                    if (m1.active === 0) {
                        return 1;
                    }

                    if (m2.active === 0) {
                        return -1;
                    }
                }

                if (m1.weight !== m2.weight) {
                    if (m1.weight > m2.weight) {
                        return -1;
                    }

                    if (m1.weight < m2.weight) {
                        return 1;
                    }
                }

                return m1.pathName.localeCompare(m2.pathName);
            });

            return result;
        },

        editMode() {
            return this.enableEditMode && this.$root.socket.connected;
        },

        editIncidentMode() {
            return this.enableEditIncidentMode;
        },

        isPublished() {
            return this.config.published;
        },

        logoClass() {
            if (this.editMode) {
                return {
                    "edit-mode": true,
                };
            }
            return {};
        },

        incidentClass() {
            return "bg-" + this.incident.style;
        },

        maintenanceClass() {
            return "bg-maintenance";
        },

        overallStatus() {

            if (Object.keys(this.$root.publicLastHeartbeatList).length === 0) {
                return -1;
            }

            let status = STATUS_PAGE_ALL_UP;
            let hasUp = false;

            for (let id in this.$root.publicLastHeartbeatList) {
                let beat = this.$root.publicLastHeartbeatList[id];

                if (beat.status === MAINTENANCE) {
                    return STATUS_PAGE_MAINTENANCE;
                } else if (beat.status === UP) {
                    hasUp = true;
                } else {
                    status = STATUS_PAGE_PARTIAL_DOWN;
                }
            }

            if (! hasUp) {
                status = STATUS_PAGE_ALL_DOWN;
            }

            return status;
        },

        allUp() {
            return this.overallStatus === STATUS_PAGE_ALL_UP;
        },

        partialDown() {
            return this.overallStatus === STATUS_PAGE_PARTIAL_DOWN;
        },

        allDown() {
            return this.overallStatus === STATUS_PAGE_ALL_DOWN;
        },

        isMaintenance() {
            return this.overallStatus === STATUS_PAGE_MAINTENANCE;
        },

        incidentHTML() {
            if (this.incident.content != null) {
                return DOMPurify.sanitize(marked(this.incident.content));
            } else {
                return "";
            }
        },

        descriptionHTML() {
            if (this.config.description != null) {
                return DOMPurify.sanitize(marked(this.config.description));
            } else {
                return "";
            }
        },

        footerHTML() {
            if (this.config.footerText != null) {
                return DOMPurify.sanitize(marked(this.config.footerText));
            } else {
                return "";
            }
        },

        lastUpdateTimeDisplay() {
            return this.$root.datetime(this.lastUpdateTime);
        }
    },
    watch: {

        /**
         * If connected to the socket and logged in, request private data of this statusPage
         * @param connected
         */
        "$root.loggedIn"(loggedIn) {
            if (loggedIn) {
                this.$root.getSocket().emit("getStatusPage", this.slug, (res) => {
                    if (res.ok) {
                        this.config = res.config;

                        if (!this.config.customCSS) {
                            this.config.customCSS = "body {\n" +
                                "  \n" +
                                "}\n";
                        }

                    } else {
                        toast.error(res.msg);
                    }
                });
            }
        },

        /**
         * Selected a monitor and add to the list.
         */
        selectedMonitor(monitor) {
            if (monitor) {
                if (this.$root.publicGroupList.length === 0) {
                    this.addGroup();
                }

                const firstGroup = this.$root.publicGroupList[0];

                firstGroup.monitorList.push(monitor);
                this.selectedMonitor = null;
            }
        },

        // Set Theme
        "config.theme"() {
            this.$root.statusPageTheme = this.config.theme;
            this.loadedTheme = true;
        },

        "config.title"(title) {
            document.title = title;
        },

        "$root.monitorList"() {
            let count = Object.keys(this.$root.monitorList).length;

            // Since publicGroupList is getting from public rest api, monitors' tags may not present if showTags = false
            if (count > 0) {
                for (let group of this.$root.publicGroupList) {
                    for (let monitor of group.monitorList) {
                        if (monitor.tags === undefined && this.$root.monitorList[monitor.id]) {
                            monitor.tags = this.$root.monitorList[monitor.id].tags;
                        }
                    }
                }
            }
        }

    },
    async created() {
        this.hasToken = ("token" in this.$root.storage());

        // Browser change page
        // https://stackoverflow.com/questions/7317273/warn-user-before-leaving-web-page-with-unsaved-changes
        window.addEventListener("beforeunload", (e) => {
            if (this.editMode) {
                (e || window.event).returnValue = leavePageMsg;
                return leavePageMsg;
            } else {
                return null;
            }
        });

        // Special handle for dev
        this.baseURL = getResBaseURL();
    },
    async mounted() {
        this.slug = this.overrideSlug || this.$route.params.slug;

        if (!this.slug) {
            this.slug = "default";
        }

        this.getData().then((res) => {
            this.config = res.data.config;

            if (!this.config.domainNameList) {
                this.config.domainNameList = [];
            }

            if (this.config.icon) {
                this.imgDataUrl = this.config.icon;
            }

            this.incident = res.data.incident;
            this.maintenanceList = res.data.maintenanceList;
            this.$root.publicGroupList = res.data.publicGroupList;

            this.loading = false;
        }).catch( function (error) {
            if (error.response.status === 404) {
                location.href = "/page-not-found";
            }
            console.log(error);
        });

        // Configure auto-refresh loop
        this.updateHeartbeatList();
        feedInterval = setInterval(() => {
            this.updateHeartbeatList();
        }, (this.autoRefreshInterval * 60 + 10) * 1000);

        this.updateUpdateTimer();

        // Go to edit page if ?edit present
        // null means ?edit present, but no value
        if (this.$route.query.edit || this.$route.query.edit === null) {
            this.edit();
        }
    },
    methods: {

        /**
         * Get status page data
         * It should be preloaded in window.preloadData
         * @returns {Promise<any>}
         */
        getData: function () {
            if (window.preloadData) {
                return new Promise(resolve => resolve({
                    data: window.preloadData
                }));
            } else {
                return axios.get("/api/status-page/" + this.slug);
            }
        },

        /**
         * Provide syntax highlighting for CSS
         * @param {string} code Text to highlight
         * @returns {string}
         */
        highlighter(code) {
            return highlight(code, languages.css);
        },

        /** Update the heartbeat list and update favicon if neccessary */
        updateHeartbeatList() {
            // If editMode, it will use the data from websocket.
            if (! this.editMode) {
                axios.get("/api/status-page/heartbeat/" + this.slug).then((res) => {
                    const { heartbeatList, uptimeList } = res.data;

                    this.$root.heartbeatList = heartbeatList;
                    this.$root.uptimeList = uptimeList;

                    const heartbeatIds = Object.keys(heartbeatList);
                    const downMonitors = heartbeatIds.reduce((downMonitorsAmount, currentId) => {
                        const monitorHeartbeats = heartbeatList[currentId];
                        const lastHeartbeat = monitorHeartbeats.at(-1);

                        if (lastHeartbeat) {
                            return lastHeartbeat.status === 0 ? downMonitorsAmount + 1 : downMonitorsAmount;
                        } else {
                            return downMonitorsAmount;
                        }
                    }, 0);

                    favicon.badge(downMonitors);

                    this.loadedData = true;
                    this.lastUpdateTime = dayjs();
                    this.updateUpdateTimer();
                });
            }
        },

        /**
         * Setup timer to display countdown to refresh
         * @returns {void}
         */
        updateUpdateTimer() {
            clearInterval(this.updateCountdown);

            this.updateCountdown = setInterval(() => {
                const countdown = dayjs.duration(this.lastUpdateTime.add(this.autoRefreshInterval, "minutes").add(10, "seconds").diff(dayjs()));
                if (countdown.as("seconds") < 0) {
                    clearInterval(this.updateCountdown);
                } else {
                    this.updateCountdownText = countdown.format("mm:ss");
                }
            }, 1000);
        },

        /** Enable editing mode */
        edit() {
            if (this.hasToken) {
                this.$root.initSocketIO(true);
                this.enableEditMode = true;
                this.clickedEditButton = true;

                // Try to fix #1658
                this.loadedData = true;
            }
        },

        /** Save the status page */
        save() {
            this.loading = true;
            let startTime = new Date();
            this.config.slug = this.config.slug.trim().toLowerCase();

            this.$root.getSocket().emit("saveStatusPage", this.slug, this.config, this.imgDataUrl, this.$root.publicGroupList, (res) => {
                if (res.ok) {
                    this.enableEditMode = false;
                    this.$root.publicGroupList = res.publicGroupList;

                    // Add some delay, so that the side menu animation would be better
                    let endTime = new Date();
                    let time = 100 - (endTime - startTime) / 1000;

                    if (time < 0) {
                        time = 0;
                    }

                    setTimeout(() => {
                        this.loading = false;
                        location.href = "/status/" + this.config.slug;
                    }, time);

                } else {
                    this.loading = false;
                    toast.error(res.msg);
                }
            });
        },

        /** Show dialog confirming deletion */
        deleteDialog() {
            this.$refs.confirmDelete.show();
        },

        /** Request deletion of this status page */
        deleteStatusPage() {
            this.$root.getSocket().emit("deleteStatusPage", this.slug, (res) => {
                if (res.ok) {
                    this.enableEditMode = false;
                    location.href = "/manage-status-page";
                } else {
                    toast.error(res.msg);
                }
            });
        },

        /**
         * Returns label for a specifed monitor
         * @param {Object} monitor Object representing monitor
         * @returns {string}
         */
        monitorSelectorLabel(monitor) {
            return `${monitor.name}`;
        },

        /** Add a group to the status page */
        addGroup() {
            let groupName = this.$t("Untitled Group");

            if (this.$root.publicGroupList.length === 0) {
                groupName = this.$t("Services");
            }

            this.$root.publicGroupList.unshift({
                name: groupName,
                monitorList: [],
            });
        },

        /** Add a domain to the status page */
        addDomainField() {
            this.config.domainNameList.push("");
        },

        /** Discard changes to status page */
        discard() {
            location.href = "/status/" + this.slug;
        },

        /**
         * Set URL of new image after successful crop operation
         * @param {string} imgDataUrl URL of image in data:// format
         */
        cropSuccess(imgDataUrl) {
            this.imgDataUrl = imgDataUrl;
        },

        /** Show image crop dialog if in edit mode */
        showImageCropUploadMethod() {
            if (this.editMode) {
                this.showImageCropUpload = true;
            }
        },

        /** Create an incident for this status page */
        createIncident() {
            this.enableEditIncidentMode = true;

            if (this.incident) {
                this.previousIncident = this.incident;
            }

            this.incident = {
                title: "",
                content: "",
                style: "primary",
            };
        },

        /** Post the incident to the status page */
        postIncident() {
            if (this.incident.title === "" || this.incident.content === "") {
                toast.error(this.$t("Please input title and content"));
                return;
            }

            this.$root.getSocket().emit("postIncident", this.slug, this.incident, (res) => {

                if (res.ok) {
                    this.enableEditIncidentMode = false;
                    this.incident = res.incident;
                } else {
                    toast.error(res.msg);
                }

            });

        },

        /** Click Edit Button */
        editIncident() {
            this.enableEditIncidentMode = true;
            this.previousIncident = Object.assign({}, this.incident);
        },

        /** Cancel creation or editing of incident */
        cancelIncident() {
            this.enableEditIncidentMode = false;

            if (this.previousIncident) {
                this.incident = this.previousIncident;
                this.previousIncident = null;
            }
        },

        /** Unpin the incident */
        unpinIncident() {
            this.$root.getSocket().emit("unpinIncident", this.slug, () => {
                this.incident = null;
            });
        },

        /**
         * Get the relative time difference of a date from now
         * @returns {string}
         */
        dateFromNow(date) {
            return dayjs.utc(date).fromNow();
        },

        /**
         * Remove a domain from the status page
         * @param {number} index Index of domain to remove
         */
        removeDomain(index) {
            this.config.domainNameList.splice(index, 1);
        },

        /**
         * Generate sanitized HTML from maintenance description
         * @param {string} description
         * @returns {string} Sanitized HTML
         */
        maintenanceHTML(description) {
            if (description) {
                return DOMPurify.sanitize(marked(description));
            } else {
                return "";
            }
        },

    }
};
</script>

<style lang="scss" scoped>
@import "../assets/vars.scss";

.overall-status {
    font-weight: bold;
    font-size: 25px;

    .ok {
        color: $primary;
    }

    .warning {
        color: $warning;
    }

    .danger {
        color: $danger;
    }
}

h1 {
    font-size: 30px;

    img {
        vertical-align: middle;
        height: 60px;
        width: 60px;
    }
}

.sidebar {
    position: fixed;
    left: 0;
    top: 0;
    width: 300px;
    height: 100vh;

    border-right: 1px solid #ededed;

    .danger-zone {
        border-top: 1px solid #ededed;
        padding-top: 15px;
    }

    .sidebar-body {
        padding: 0 10px 10px 10px;
        overflow-x: hidden;
        overflow-y: auto;
        height: calc(100% - 70px);
    }

    .sidebar-footer {
        border-top: 1px solid #ededed;
        border-right: 1px solid #ededed;
        padding: 10px;
        width: 300px;
        height: 70px;
        position: fixed;
        left: 0;
        bottom: 0;
        background-color: white;
        display: flex;
        align-items: center;
    }
}

footer {
    text-align: center;
    font-size: 14px;
}

.description span {
    min-width: 50px;
}

.title-flex {
    display: flex;
    align-items: center;
    gap: 10px;
}

.logo-wrapper {
    display: inline-block;
    position: relative;

    &:hover {
        .icon-upload {
            transform: scale(1.2);
        }
    }

    .icon-upload {
        transition: all $easing-in 0.2s;
        position: absolute;
        bottom: 6px;
        font-size: 20px;
        left: -14px;
        background-color: white;
        padding: 5px;
        border-radius: 10px;
        cursor: pointer;
        box-shadow: 0 15px 70px rgba(0, 0, 0, 0.9);
    }
}

.logo {
    transition: all $easing-in 0.2s;

    &.edit-mode {
        cursor: pointer;

        &:hover {
            transform: scale(1.2);
        }
    }
}

.incident {
    .content {
        &[contenteditable="true"] {
            min-height: 60px;
        }
    }

    .date {
        font-size: 12px;
    }
}

.maintenance-bg-info {
    color: $maintenance;
}

.maintenance-icon {
    font-size: 35px;
    vertical-align: middle;
}

.dark .shadow-box {
    background-color: #0d1117;
}

.status-maintenance {
    color: $maintenance;
    margin-right: 5px;
}

.mobile {
    h1 {
        font-size: 22px;
    }

    .overall-status {
        font-size: 20px;
    }
}

.dark {
    .sidebar {
        background-color: $dark-header-bg;
        border-right-color: $dark-border-color;

        .danger-zone {
            border-top-color: $dark-border-color;
        }

        .sidebar-footer {
            border-right-color: $dark-border-color;
            border-top-color: $dark-border-color;
            background-color: $dark-header-bg;
        }
    }
}

.domain-name-list {
    li {
        display: flex;
        align-items: center;
        padding: 10px 0 10px 10px;

        .domain-input {
            flex-grow: 1;
            background-color: transparent;
            border: none;
            color: $dark-font-color;
            outline: none;

            &::placeholder {
                color: #1d2634;
            }
        }
    }
}

/* required class */
.css-editor {
    /* we dont use `language-` classes anymore so thats why we need to add background and text color manually */

    border-radius: 1rem;
    padding: 10px 5px;
    border: 1px solid #ced4da;

    .dark & {
        background: $dark-bg;
        border: 1px solid $dark-border-color;
    }
}

.bg-maintenance {
    .alert-heading {
        font-weight: bold;
    }
}

.refresh-info {
    opacity: 0.7;
}

.main {
    transition: all ease-in-out 0.1s;

    &.edit {
        margin-left: 300px;
    }
}

.main-container {
    display: flex;
    width: 1440px;
    flex-direction: column;
    align-items: flex-end;

    background: #FFF;

    margin: 0 auto;
}

.header {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
}

.logo-and-actions {
    display: flex;
    width: 1440px;
    height: 64px;
    padding: 0px 48px;
    justify-content: space-between;
    align-items: center;

    border-bottom: 1px solid #DBDFE6;
    background: #FFF;
}

.logo {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
    align-self: stretch;
}

.actions {
    display: flex;
    align-items: center;
    gap: 4px;

    color: #0D0F13;
}

.modify, .dashboard {
    display: flex;
    padding: 12px 12px 12px 16px;
    justify-content: center;
    align-items: center;
    gap: 4px;
}

.unset {
    text-decoration: none;
    border: none;
    background: none;
    font-weight: inherit;
}

</style>
