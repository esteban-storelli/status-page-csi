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
                    <div class="actions" v-if="hasToken">
                        <div class="actions" v-if="!enableEditMode">
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
                        <div v-else>
                        <button class="btn btn-primary btn-add-group me-2" @click="createIncident">
                            <font-awesome-icon icon="bullhorn" />
                            {{ $t("Create Incident") }}
                        </button>
                    </div>
                    </div>
                </div>
            </div>
            <div class="subtitle">
                Stato dei servizi
            </div>

            <!-- Admin functions -->
            <!-- <div v-if="hasToken" class="mb-4">
                <div v-if="!enableEditMode">
                    <button class="btn btn-info me-2" @click="edit">
                        <font-awesome-icon icon="edit" />
                        {{ $t("Edit Status Page") }}
                    </button>

                    <a href="/manage-status-page" class="btn btn-info">
                        <font-awesome-icon icon="tachometer-alt" />
                        {{ $t("Go to Dashboard") }}
                    </a>
                </div> -->

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
            <div class="list overall-status">
                <div v-if="Object.keys($root.publicMonitorList).length === 0 && loadedData">
                    <font-awesome-icon icon="question-circle" class="ok" />
                    {{ $t("No Services") }}
                </div>

                <template v-else>
                    <div v-if="allUp">
                    <div class="status-box">
                            <div class="ok-icon status-icon-box">
                            <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 25" fill="none">
                                <path d="M10.6 17.1L17.65 10.05L16.25 8.65L10.6 14.3L7.75 11.45L6.35 12.85L10.6 17.1ZM12 22.5C10.6167 22.5 9.31667 22.2375 8.1 21.7125C6.88333 21.1875 5.825 20.475 4.925 19.575C4.025 18.675 3.3125 17.6167 2.7875 16.4C2.2625 15.1833 2 13.8833 2 12.5C2 11.1167 2.2625 9.81667 2.7875 8.6C3.3125 7.38333 4.025 6.325 4.925 5.425C5.825 4.525 6.88333 3.8125 8.1 3.2875C9.31667 2.7625 10.6167 2.5 12 2.5C13.3833 2.5 14.6833 2.7625 15.9 3.2875C17.1167 3.8125 18.175 4.525 19.075 5.425C19.975 6.325 20.6875 7.38333 21.2125 8.6C21.7375 9.81667 22 11.1167 22 12.5C22 13.8833 21.7375 15.1833 21.2125 16.4C20.6875 17.6167 19.975 18.675 19.075 19.575C18.175 20.475 17.1167 21.1875 15.9 21.7125C14.6833 22.2375 13.3833 22.5 12 22.5Z" fill="white"/>
                            </svg>
                            </div>
                            <div class="status-title-box">
                            {{ $t("All Systems Operational") }}
                            </div>
                        </div>
                    </div>

                    <div v-else-if="partialDown">
                        <!-- <font-awesome-icon icon="exclamation-circle" class="warning" /> -->
                        <div class="status-box">
                            <div class="warning-icon status-icon-box">
                            <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48" fill="none">
                                <path d="M2 42L24 4L46 42H2ZM8.9 38H39.1L24 12L8.9 38ZM24 36C24.5667 36 25.0417 35.8083 25.425 35.425C25.8083 35.0417 26 34.5667 26 34C26 33.4333 25.8083 32.9583 25.425 32.575C25.0417 32.1917 24.5667 32 24 32C23.4333 32 22.9583 32.1917 22.575 32.575C22.1917 32.9583 22 33.4333 22 34C22 34.5667 22.1917 35.0417 22.575 35.425C22.9583 35.8083 23.4333 36 24 36ZM22 30H26V20H22V30Z" fill="white"/>
                            </svg>
                            </div>
                            <div class="status-title-box">
                            {{ $t("Partially Degraded Service") }}
                            </div>
                        </div>
                    </div>

                    <div v-else-if="allDown">
                        <div class="status-box">
                            <div class="danger-icon status-icon-box">
                            <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 25" fill="none">
                                <path d="M12 17.5C12.2833 17.5 12.5208 17.4042 12.7125 17.2125C12.9042 17.0208 13 16.7833 13 16.5C13 16.2167 12.9042 15.9792 12.7125 15.7875C12.5208 15.5958 12.2833 15.5 12 15.5C11.7167 15.5 11.4792 15.5958 11.2875 15.7875C11.0958 15.9792 11 16.2167 11 16.5C11 16.7833 11.0958 17.0208 11.2875 17.2125C11.4792 17.4042 11.7167 17.5 12 17.5ZM11 13.5H13V7.5H11V13.5ZM12 22.5C10.6167 22.5 9.31667 22.2375 8.1 21.7125C6.88333 21.1875 5.825 20.475 4.925 19.575C4.025 18.675 3.3125 17.6167 2.7875 16.4C2.2625 15.1833 2 13.8833 2 12.5C2 11.1167 2.2625 9.81667 2.7875 8.6C3.3125 7.38333 4.025 6.325 4.925 5.425C5.825 4.525 6.88333 3.8125 8.1 3.2875C9.31667 2.7625 10.6167 2.5 12 2.5C13.3833 2.5 14.6833 2.7625 15.9 3.2875C17.1167 3.8125 18.175 4.525 19.075 5.425C19.975 6.325 20.6875 7.38333 21.2125 8.6C21.7375 9.81667 22 11.1167 22 12.5C22 13.8833 21.7375 15.1833 21.2125 16.4C20.6875 17.6167 19.975 18.675 19.075 19.575C18.175 20.475 17.1167 21.1875 15.9 21.7125C14.6833 22.2375 13.3833 22.5 12 22.5ZM12 20.5C14.2333 20.5 16.125 19.725 17.675 18.175C19.225 16.625 20 14.7333 20 12.5C20 10.2667 19.225 8.375 17.675 6.825C16.125 5.275 14.2333 4.5 12 4.5C9.76667 4.5 7.875 5.275 6.325 6.825C4.775 8.375 4 10.2667 4 12.5C4 14.7333 4.775 16.625 6.325 18.175C7.875 19.725 9.76667 20.5 12 20.5Z" fill="white"/>
                            </svg>
                            </div>
                            <div class="status-title-box">
                             {{ $t("Degraded Service") }}
                            </div>
                        </div>
                    </div>

                    <div v-else-if="isMaintenance">
                        <div class="status-box">
                            <div class="maintenance-icon status-icon-box">
                            <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48" fill="none">
                                <path d="M18.4999 44L17.6999 37.6C17.2666 37.4333 16.8582 37.2333 16.4749 37C16.0916 36.7667 15.7166 36.5167 15.3499 36.25L9.3999 38.75L3.8999 29.25L9.0499 25.35C9.01657 25.1167 8.9999 24.8917 8.9999 24.675V23.325C8.9999 23.1083 9.01657 22.8833 9.0499 22.65L3.8999 18.75L9.3999 9.25L15.3499 11.75C15.7166 11.4833 16.0999 11.2333 16.4999 11C16.8999 10.7667 17.2999 10.5667 17.6999 10.4L18.4999 4H29.4999L30.2999 10.4C30.7332 10.5667 31.1416 10.7667 31.5249 11C31.9082 11.2333 32.2832 11.4833 32.6499 11.75L38.5999 9.25L44.0999 18.75L38.9499 22.65C38.9832 22.8833 38.9999 23.1083 38.9999 23.325V24.675C38.9999 24.8917 38.9666 25.1167 38.8999 25.35L44.0499 29.25L38.5499 38.75L32.6499 36.25C32.2832 36.5167 31.8999 36.7667 31.4999 37C31.0999 37.2333 30.6999 37.4333 30.2999 37.6L29.4999 44H18.4999ZM24.0999 31C26.0332 31 27.6832 30.3167 29.0499 28.95C30.4166 27.5833 31.0999 25.9333 31.0999 24C31.0999 22.0667 30.4166 20.4167 29.0499 19.05C27.6832 17.6833 26.0332 17 24.0999 17C22.1332 17 20.4749 17.6833 19.1249 19.05C17.7749 20.4167 17.0999 22.0667 17.0999 24C17.0999 25.9333 17.7749 27.5833 19.1249 28.95C20.4749 30.3167 22.1332 31 24.0999 31ZM24.0999 27C23.2666 27 22.5582 26.7083 21.9749 26.125C21.3916 25.5417 21.0999 24.8333 21.0999 24C21.0999 23.1667 21.3916 22.4583 21.9749 21.875C22.5582 21.2917 23.2666 21 24.0999 21C24.9332 21 25.6416 21.2917 26.2249 21.875C26.8082 22.4583 27.0999 23.1667 27.0999 24C27.0999 24.8333 26.8082 25.5417 26.2249 26.125C25.6416 26.7083 24.9332 27 24.0999 27ZM21.9999 40H25.9499L26.6499 34.7C27.6832 34.4333 28.6416 34.0417 29.5249 33.525C30.4082 33.0083 31.2166 32.3833 31.9499 31.65L36.8999 33.7L38.8499 30.3L34.5499 27.05C34.7166 26.5833 34.8332 26.0917 34.8999 25.575C34.9666 25.0583 34.9999 24.5333 34.9999 24C34.9999 23.4667 34.9666 22.9417 34.8999 22.425C34.8332 21.9083 34.7166 21.4167 34.5499 20.95L38.8499 17.7L36.8999 14.3L31.9499 16.4C31.2166 15.6333 30.4082 14.9917 29.5249 14.475C28.6416 13.9583 27.6832 13.5667 26.6499 13.3L25.9999 8H22.0499L21.3499 13.3C20.3166 13.5667 19.3582 13.9583 18.4749 14.475C17.5916 14.9917 16.7832 15.6167 16.0499 16.35L11.0999 14.3L9.1499 17.7L13.4499 20.9C13.2832 21.4 13.1666 21.9 13.0999 22.4C13.0332 22.9 12.9999 23.4333 12.9999 24C12.9999 24.5333 13.0332 25.05 13.0999 25.55C13.1666 26.05 13.2832 26.55 13.4499 27.05L9.1499 30.3L11.0999 33.7L16.0499 31.6C16.7832 32.3667 17.5916 33.0083 18.4749 33.525C19.3582 34.0417 20.3166 34.4333 21.3499 34.7L21.9999 40Z" fill="white"/>
                            </svg>
                            </div>
                            <div class="status-title-box">
                                {{ $t("maintenanceStatus-under-maintenance") }}
                            </div>
                        </div>
                    </div>

                    <div v-else>
                        <div class="status-box">
                            <div class="unknown-icon status-icon-box">
                            <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48" fill="none">
                                <path d="M21.2 32C21.2 29.3 21.4417 27.3583 21.925 26.175C22.4083 24.9917 23.4333 23.7 25 22.3C26.3667 21.1 27.4083 20.0583 28.125 19.175C28.8417 18.2917 29.2 17.2833 29.2 16.15C29.2 14.7833 28.7417 13.65 27.825 12.75C26.9083 11.85 25.6333 11.4 24 11.4C22.3 11.4 21.0083 11.9167 20.125 12.95C19.2417 13.9833 18.6167 15.0333 18.25 16.1L13.1 13.9C13.8 11.7667 15.0833 9.91667 16.95 8.35C18.8167 6.78333 21.1667 6 24 6C27.5 6 30.1917 6.975 32.075 8.925C33.9583 10.875 34.9 13.2167 34.9 15.95C34.9 17.6167 34.5417 19.0417 33.825 20.225C33.1083 21.4083 31.9833 22.75 30.45 24.25C28.8167 25.8167 27.825 27.0083 27.475 27.825C27.125 28.6417 26.95 30.0333 26.95 32H21.2ZM24 44C22.9 44 21.9583 43.6083 21.175 42.825C20.3917 42.0417 20 41.1 20 40C20 38.9 20.3917 37.9583 21.175 37.175C21.9583 36.3917 22.9 36 24 36C25.1 36 26.0417 36.3917 26.825 37.175C27.6083 37.9583 28 38.9 28 40C28 41.1 27.6083 42.0417 26.825 42.825C26.0417 43.6083 25.1 44 24 44Z" fill="white"/>
                            </svg>
                            </div>
                            <div class="status-title-box">
                                Stato sconosciuto
                            </div>
                        </div>
                    </div>
                    <div class="refresh-info">
                        <div>{{ $t("Last Updated") }}:  {{ lastUpdateTimeDisplay }}</div>
                        <div>{{ $tc("statusPageRefreshIn", [ updateCountdownText]) }}</div>
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
            <div v-if="! enableEditMode" class="alert-heading" v-html="descriptionHTML"></div>

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
            <div class="custom-footer-text text-start">
                <strong v-if="enableEditMode">{{ $t("Custom Footer") }}:</strong>
            </div>
            <Editable v-if="enableEditMode" v-model="config.footerText" tag="div" :contenteditable="enableEditMode" :noNL="false" class="alert-heading p-2" />
            <!-- eslint-disable-next-line vue/no-v-html-->
            <div v-if="! enableEditMode" class="alert-heading p-2" v-html="footerHTML"></div>
            <footer class="footer">
                <div class="footer-top-container">
                    <img src="/frame_dark.svg">
                    <div class="footer-button-container">
                        <button @click="scrollToTop" class="unset footer-top-button">Torna su</button>
                        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 20 20" fill="none">
                            <path d="M9.16667 15.0001V7.33341L6.16667 10.3334L5 9.16675L10 4.16675L15 9.16675L13.8333 10.3334L10.8333 7.33341V15.0001H9.16667Z" fill="white"/>
                        </svg>
                    </div>
                </div>
                <div class="footer-bottom-container">
                    <!-- <p v-if="config.showPoweredBy">
                    {{ $t("Powered by") }} <a target="_blank" rel="noopener noreferrer" href="https://github.com/louislam/uptime-kuma">{{ $t("Uptime Kuma" ) }}</a>
                    </p> -->

                    <!-- <div class="refresh-info mb-2">
                        <div>{{ $t("Last Updated") }}:  {{ lastUpdateTimeDisplay }}</div>
                        <div>{{ $tc("statusPageRefreshIn", [ updateCountdownText]) }}</div>
                    </div> -->
                    <div class="social-container">
                        <a href="https://www.instagram.com/cantone_ticino" class="unset footer-link">
                            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none">
                                <path d="M11.993 7.35399C9.439 7.35399 7.37 9.42399 7.37 11.977C7.37 14.531 9.439 16.601 11.993 16.601C14.545 16.601 16.616 14.531 16.616 11.977C16.616 9.42399 14.545 7.35399 11.993 7.35399ZM11.993 14.981C10.334 14.981 8.989 13.636 8.989 11.978C8.989 10.319 10.334 8.97499 11.993 8.97499C13.652 8.97499 14.995 10.319 14.995 11.978C14.995 13.636 13.652 14.981 11.993 14.981Z" fill="white"/>
                                <path d="M16.8 8.26199C17.3954 8.26199 17.878 7.77936 17.878 7.18399C17.878 6.58863 17.3954 6.10599 16.8 6.10599C16.2046 6.10599 15.722 6.58863 15.722 7.18399C15.722 7.77936 16.2046 8.26199 16.8 8.26199Z" fill="white"/>
                                <path d="M20.527 6.08799C20.058 4.87899 19.103 3.92299 17.894 3.45599C17.195 3.19299 16.456 3.05199 15.708 3.03599C14.745 2.99399 14.44 2.98199 11.998 2.98199C9.556 2.98199 9.243 2.98199 8.288 3.03599C7.542 3.05099 6.803 3.19199 6.104 3.45599C4.894 3.92299 3.939 4.87899 3.471 6.08799C3.208 6.78799 3.067 7.52599 3.052 8.27399C3.009 9.23599 2.996 9.54099 2.996 11.984C2.996 14.426 2.996 14.737 3.052 15.694C3.067 16.442 3.208 17.18 3.471 17.881C3.94 19.089 4.895 20.045 6.105 20.513C6.801 20.785 7.54 20.939 8.29 20.963C9.253 21.005 9.558 21.018 12 21.018C14.442 21.018 14.755 21.018 15.71 20.963C16.457 20.948 17.196 20.806 17.896 20.544C19.105 20.075 20.06 19.12 20.529 17.911C20.792 17.211 20.933 16.473 20.948 15.725C20.991 14.763 21.004 14.458 21.004 12.015C21.004 9.57199 21.004 9.26199 20.948 8.30499C20.935 7.54699 20.795 6.79599 20.527 6.08799ZM19.309 15.62C19.302 16.196 19.198 16.767 18.998 17.308C18.693 18.095 18.072 18.717 17.286 19.019C16.751 19.218 16.187 19.322 15.616 19.33C14.666 19.374 14.398 19.385 11.962 19.385C9.524 19.385 9.275 19.385 8.307 19.33C7.738 19.323 7.172 19.218 6.638 19.019C5.849 18.718 5.224 18.096 4.919 17.308C4.723 16.774 4.617 16.209 4.608 15.639C4.565 14.689 4.555 14.421 4.555 11.985C4.555 9.54799 4.555 9.29899 4.608 8.32999C4.615 7.75399 4.719 7.18399 4.919 6.64299C5.224 5.85399 5.849 5.23299 6.638 4.93099C7.172 4.73299 7.738 4.62799 8.307 4.61999C9.258 4.57699 9.525 4.56499 11.962 4.56499C14.399 4.56499 14.649 4.56499 15.616 4.61999C16.187 4.62699 16.751 4.73199 17.286 4.93099C18.072 5.23399 18.693 5.85599 18.998 6.64299C19.194 7.17699 19.3 7.74199 19.309 8.31199C19.352 9.26299 19.363 9.52999 19.363 11.967C19.363 14.403 19.363 14.665 19.32 15.621H19.309V15.62Z" fill="white"/>
                            </svg>
                        </a>
                        <a href="https://www.linkedin.com/company/repubblica-e-cantone-ticino/" class="unset footer-link">
                            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 20 20" fill="none">
                                <path d="M3.08549 5.28902C4.29389 5.28902 5.27349 4.30942 5.27349 3.10102C5.27349 1.89263 4.29389 0.913025 3.08549 0.913025C1.87709 0.913025 0.897491 1.89263 0.897491 3.10102C0.897491 4.30942 1.87709 5.28902 3.08549 5.28902Z" fill="white"/>
                                <path d="M7.33949 6.94702V19.086H11.1085V13.083C11.1085 11.499 11.4065 9.96502 13.3705 9.96502C15.3075 9.96502 15.3315 11.776 15.3315 13.183V19.087H19.1025V12.43C19.1025 9.16002 18.3985 6.64703 14.5765 6.64703C12.7415 6.64703 11.5115 7.65403 11.0085 8.60703H10.9575V6.94702H7.33949ZM1.19749 6.94702H4.97249V19.086H1.19749V6.94702Z" fill="white"/>
                            </svg>
                        </a>
                        <a href="https://x.com/Cantone_Ticino" class="unset footer-link">
                            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none">
                                <path d="M13.7145 10.6226L20.4161 3H18.8282L13.0066 9.61719L8.36025 3H3L10.0278 13.0073L3 21H4.58786L10.7319 14.0104L15.6398 21H21L13.7145 10.6226ZM11.539 13.0951L10.8258 12.098L5.16048 4.17132H7.59976L12.1732 10.5708L12.8834 11.5679L18.8275 19.8857H16.3882L11.539 13.0951Z" fill="white"/>
                            </svg>
                        </a>
                        <a href="https://www.youtube.com/@CantoneTI" class="unset footer-link">
                            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 21 15" fill="none">
                                <path d="M19.607 2.20301C19.377 1.34501 18.702 0.668007 17.845 0.437007C16.279 0.00700739 10.014 7.59651e-06 10.014 7.59651e-06C10.014 7.59651e-06 3.75 -0.00699261 2.183 0.404007C1.343 0.633007 0.649002 1.32501 0.417002 2.18201C0.00400165 3.74801 1.46514e-06 6.99601 1.46514e-06 6.99601C1.46514e-06 6.99601 -0.00399852 10.26 0.406001 11.81C0.636001 12.667 1.311 13.344 2.169 13.575C3.751 14.005 9.999 14.012 9.999 14.012C9.999 14.012 16.264 14.019 17.83 13.609C18.686 13.379 19.364 12.703 19.597 11.846C20.011 10.281 20.014 7.03401 20.014 7.03401C20.014 7.03401 20.034 3.76901 19.607 2.20301ZM8.01 10.005L8.015 4.00501L13.222 7.01001L8.01 10.005Z" fill="white"/>
                            </svg>
                        </a>
                        <a href="https://www.whatsapp.com/channel/0029Vb5q0cHElagqL7xrkD32" class="unset footer-link">
                            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 18 18" fill="none">
                                <path d="M15.304 2.61562C13.6205 0.928125 11.3786 0 8.99598 0C4.07812 0 0.0763393 4.00179 0.0763393 8.91964C0.0763393 10.4906 0.486161 12.0254 1.26562 13.3795L0 18L4.72902 16.7585C6.0308 17.4696 7.49732 17.8433 8.99196 17.8433H8.99598C13.9098 17.8433 18 13.8415 18 8.92366C18 6.54107 16.9875 4.30312 15.304 2.61562ZM8.99598 16.3406C7.66205 16.3406 6.35625 15.983 5.2192 15.308L4.95 15.1473L2.14554 15.8826L2.89286 13.1464L2.71607 12.8652C1.97277 11.6839 1.58304 10.3219 1.58304 8.91964C1.58304 4.83348 4.90982 1.5067 9 1.5067C10.9808 1.5067 12.8411 2.27812 14.2393 3.68036C15.6375 5.08259 16.4973 6.94286 16.4933 8.92366C16.4933 13.0138 13.0821 16.3406 8.99598 16.3406ZM13.0621 10.7879C12.8411 10.6754 11.7442 10.1371 11.5393 10.0647C11.3344 9.98839 11.1857 9.95223 11.0371 10.1772C10.8884 10.4022 10.4625 10.9004 10.3299 11.0531C10.2013 11.2018 10.0688 11.2219 9.84777 11.1094C8.53795 10.4545 7.67813 9.94018 6.81429 8.45759C6.58527 8.06384 7.0433 8.09196 7.4692 7.24018C7.54152 7.09152 7.50536 6.96295 7.44911 6.85045C7.39286 6.73795 6.94688 5.64107 6.76205 5.19509C6.58125 4.76116 6.39643 4.82143 6.25982 4.81339C6.13125 4.80536 5.98259 4.80536 5.83393 4.80536C5.68527 4.80536 5.4442 4.86161 5.23929 5.08259C5.03437 5.30759 4.45982 5.84598 4.45982 6.94286C4.45982 8.03973 5.25938 9.10045 5.36786 9.24911C5.48036 9.39777 6.93884 11.6478 9.17679 12.6161C10.5911 13.2268 11.1455 13.279 11.8527 13.1746C12.2826 13.1103 13.1705 12.6362 13.3554 12.1138C13.5402 11.5915 13.5402 11.1455 13.4839 11.0531C13.4317 10.9527 13.283 10.8964 13.0621 10.7879Z" fill="white"/>
                            </svg>
                        </a>
                        <a href="https://threema.id/*BCEPJFW" class="unset footer-link">
                            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 18 20" fill="none">
                                <path fill-rule="evenodd" clip-rule="evenodd" d="M10.4456 18.5353C10.4456 19.3447 9.79863 19.9994 8.99878 19.9994C8.19892 19.9994 7.55317 19.3441 7.55317 18.5353C7.55317 17.7266 8.20014 17.0713 8.99878 17.0713C9.79741 17.0713 10.4456 17.7272 10.4456 18.5353ZM5.2282 18.5353C5.2282 19.3447 4.58122 19.9994 3.78137 19.9994C2.98151 19.9994 2.33576 19.3441 2.33576 18.5353C2.33576 17.7266 2.98273 17.0713 3.78137 17.0713C4.58 17.0713 5.2282 17.7272 5.2282 18.5353ZM15.663 18.5353C15.663 19.3447 15.016 19.9994 14.2162 19.9994C13.4163 19.9994 12.7706 19.3441 12.7706 18.5353C12.7706 17.7266 13.4176 17.0713 14.2162 17.0713C15.0148 17.0713 15.663 17.7272 15.663 18.5353ZM5.27529 14.3733L1.01673 15.4506L1.92665 11.767C1.02529 10.5659 0.5 9.12162 0.5 7.56845C0.5 3.38851 4.30543 0 9 0C13.6946 0 17.5 3.38851 17.5 7.56845C17.5 11.7484 13.6946 15.1369 9 15.1369C7.66385 15.1369 6.39986 14.8622 5.27529 14.3733ZM6.58881 6.94038H6.49403C6.28 6.94038 6.10633 7.11612 6.10633 7.33269V10.7713C6.10633 10.9879 6.28 11.1636 6.49403 11.1636H11.506C11.72 11.1636 11.8937 10.9879 11.8937 10.7713V7.33269C11.8937 7.11612 11.72 6.94038 11.506 6.94038H11.4112V5.96578C11.4112 4.62114 10.3325 3.52959 8.99878 3.52959C7.66507 3.52959 6.58881 4.62176 6.58881 5.96578V6.94038ZM10.4468 6.94038H7.55317V5.96578C7.55317 5.15888 8.20076 4.50357 8.99939 4.50357C9.79802 4.50357 10.4468 5.15888 10.4468 5.96578V6.94038Z" fill="white"/>
                            </svg>
                        </a>
                        <a href="https://www4.ti.ch/index.php?id=73333" class="unset footer-link">Vedi tutti</a>
                    </div>
                    <div class="link-container">
                        <a href="https://www4.ti.ch/tich/informazioni-legali/informazioni-legali" class="unset footer-link">Informazioni legali</a>
                    </div>
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
        scrollToTop() {
            window.scrollTo({
                top: 0,
                behavior: 'smooth'
            });
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

// footer {
//     text-align: center;
//     font-size: 14px;
// }

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

// .maintenance-icon {
//     font-size: 35px;
//     vertical-align: middle;
// }

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
    display: flex;
    padding: 12px 0 48px 48px;
    flex-direction: column;
    align-items: flex-start;
    color: #0D0F13 !important;
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
    box-shadow: 0 15px 70px rgba(0, 0, 0, 0.1);
    margin: 0 auto;
}

.header {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    border-bottom: 1px solid #DBDFE6;
}

.logo-and-actions {
    display: flex;
    width: 1440px;
    height: 64px;
    padding: 0px 48px;
    justify-content: space-between;
    align-items: center;
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

.subtitle {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    padding: 20px 40px;
    gap: 4px;
    font-size: 18px;
    border-bottom: 1px solid #DBDFE6;
}

.status-box {
    display: flex;
    padding: 64px 48px;
    align-items: center;
    align-self: stretch;
    border-bottom: 1px solid #DBDFE6;
}

.warning-icon {
    background-color: $warning;
}

.ok-icon {
    background-color: $primary;
}

.danger-icon {
    background-color: $danger;
}

.maintenance-icon {
    background-color: $maintenance;
}

.unknown-icon {
    background-color: #56657A;
}

.status-icon-box {
    display: flex;
    width: 96px;
    padding: 24px 8px;
    justify-content: center;
    align-items: flex-start;
    gap: 8px;
    align-self: stretch;
    border-radius: 4px 0px 0px 4px;
}

.status-title-box {
    display: flex;
    padding: 24px;
    align-items: flex-start;
    gap: 24px;
    flex: 1 0 0;
    align-self: stretch;
    border-radius: 0px 4px 4px 0px;
    background: #F0F2F4;
}

.footer {
    display: flex;
    width: 1440px;
    padding: 0px 48px;
    flex-direction: column;
    align-items: flex-start;
    gap: 40px;
    background: #00163C;
    color: #FFF;
}

.footer-top-container {
    display: flex;
    padding-top: 48px;
    justify-content: space-between;
    align-items: center;
    align-self: stretch;
}

.footer-button-container {
    display: flex;
    padding: 10px 10px 10px 16px;
    justify-content: center;
    gap: 4px;
    border-radius: 4px;
}

.footer-top-button, .footer-link, .refresh-info {
    color: #FFF;
    font-size: 14px;
    font-style: normal;
    line-height: 20px;
    letter-spacing: 0.07px;
}

.footer-bottom-container {
    display: flex;
    padding-bottom: 48px;
    align-self: stretch;
    justify-content: space-between;
    padding-top: 20px;
}

.social-container {
    display: flex;
    justify-content: flex-start;
    gap: 24px;
}

.link-container {
    display: flex;
    flex-direction: row;
    gap: 24px;
}

</style>
