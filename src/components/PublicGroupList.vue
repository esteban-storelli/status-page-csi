<template>
    <!-- Group List -->
    <Draggable
        v-model="$root.publicGroupList"
        :disabled="!editMode"
        item-key="id"
        :animation="100"
    >
        <template #item="group">
            <div>
                <div class="flex-div">
                    <!-- Group Title -->
                    <h2 class="group-title">
                        <font-awesome-icon v-if="editMode && showGroupDrag" icon="arrows-alt-v" class="action drag me-3" />
                        <font-awesome-icon v-if="editMode" icon="times" class="action remove me-3" @click="removeGroup(group.index)" />
                        <Editable v-model="group.element.name" :contenteditable="editMode" tag="span" />
                    </h2>
                    <div class="refresh-container">
                        <button class="unset refresh-button" @click="refreshPage">
                            Aggiorna
                        </button>
                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none">
                            <path d="M5.1 16.05C4.73333 15.4167 4.45833 14.7667 4.275 14.1C4.09167 13.4333 4 12.75 4 12.05C4 9.81667 4.775 7.91667 6.325 6.35C7.875 4.78333 9.76667 4 12 4H12.175L10.575 2.4L11.975 1L15.975 5L11.975 9L10.575 7.6L12.175 6H12C10.3333 6 8.91667 6.5875 7.75 7.7625C6.58333 8.9375 6 10.3667 6 12.05C6 12.4833 6.05 12.9083 6.15 13.325C6.25 13.7417 6.4 14.15 6.6 14.55L5.1 16.05ZM12.025 23L8.025 19L12.025 15L13.425 16.4L11.825 18H12C13.6667 18 15.0833 17.4125 16.25 16.2375C17.4167 15.0625 18 13.6333 18 11.95C18 11.5167 17.95 11.0917 17.85 10.675C17.75 10.2583 17.6 9.85 17.4 9.45L18.9 7.95C19.2667 8.58333 19.5417 9.23333 19.725 9.9C19.9083 10.5667 20 11.25 20 11.95C20 14.1833 19.225 16.0833 17.675 17.65C16.125 19.2167 14.2333 20 12 20H11.825L13.425 21.6L12.025 23Z" fill="#0058EF"/>
                        </svg>
                    </div>
                </div>
                <div class="search-container">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none">
                        <path d="M19.6 21L13.3 14.7C12.8 15.1 12.225 15.4167 11.575 15.65C10.925 15.8833 10.2333 16 9.5 16C7.68333 16 6.14583 15.3708 4.8875 14.1125C3.62917 12.8542 3 11.3167 3 9.5C3 7.68333 3.62917 6.14583 4.8875 4.8875C6.14583 3.62917 7.68333 3 9.5 3C11.3167 3 12.8542 3.62917 14.1125 4.8875C15.3708 6.14583 16 7.68333 16 9.5C16 10.2333 15.8833 10.925 15.65 11.575C15.4167 12.225 15.1 12.8 14.7 13.3L21 19.6L19.6 21ZM9.5 14C10.75 14 11.8125 13.5625 12.6875 12.6875C13.5625 11.8125 14 10.75 14 9.5C14 8.25 13.5625 7.1875 12.6875 6.3125C11.8125 5.4375 10.75 5 9.5 5C8.25 5 7.1875 5.4375 6.3125 6.3125C5.4375 7.1875 5 8.25 5 9.5C5 10.75 5.4375 11.8125 6.3125 12.6875C7.1875 13.5625 8.25 14 9.5 14Z" fill="#0D0F13"/>
                    </svg>
                    <input
                        type="text"
                        v-model="searchText"
                        placeholder="Cerca contenuto"
                        class="unset search-box"
                    />
                </div>
                <div class="monitor-list mt-4 position-relative">
                    <div v-if="group.element.monitorList.length === 0" class="text-center no-monitor-msg">
                        {{ $t("No Monitors") }}
                    </div>

                    <!-- Monitor List -->
                    <!-- animation is not working, no idea why -->
                    <Draggable
                        :list="filteredMonitorList(group)"
                        class="monitor-list"
                        group="same-group"
                        :disabled="!editMode"
                        :animation="100"
                        item-key="id"
                    >
                        <template #item="monitor">
                            <div class="item status-page-item-list">
                                <div class="column-flex">
                                    <div class="col-9 col-md-8 small-padding">
                                        <div class="info column-flex">
                                            <font-awesome-icon v-if="editMode" icon="arrows-alt-v" class="action drag me-3" />
                                            <font-awesome-icon v-if="editMode" icon="times" class="action remove me-3" @click="removeMonitor(group.index, monitor.index)" />

                                            <Uptime :monitor="monitor.element" type="24" :pill="true" />
                                            <a
                                                v-if="showLink(monitor)"
                                                :href="monitor.element.url"
                                                class="item-name"
                                                target="_blank"
                                                rel="noopener noreferrer"
                                            >
                                                {{ monitor.element.name }}
                                            </a>
                                            <p v-else class="item-name"> {{ monitor.element.name }} </p>

                                            <span
                                                title="Setting"
                                            >
                                                <font-awesome-icon
                                                    v-if="editMode"
                                                    :class="{'link-active': true, 'btn-link': true}"
                                                    icon="cog" class="action me-3"
                                                    @click="$refs.monitorSettingDialog.show(group, monitor)"
                                                />
                                            </span>
                                        </div>
                                        <div class="extra-info">
                                            <div v-if="showCertificateExpiry && monitor.element.certExpiryDaysRemaining">
                                                <Tag :item="{name: $t('Cert Exp.'), value: formattedCertExpiryMessage(monitor), color: certExpiryColor(monitor)}" :size="'sm'" />
                                            </div>
                                            <div v-if="showTags">
                                                <Tag v-for="tag in monitor.element.tags" :key="tag" :item="tag" :size="'sm'" />
                                            </div>
                                        </div>
                                    </div>
                                    <div :key="$root.userHeartbeatBar" class="col-3 col-md-4 big-row">
                                        <HeartbeatBar size="large" :monitor-id="monitor.element.id" />
                                    </div>
                                </div>
                            </div>
                        </template>
                    </Draggable>
                </div>
            </div>
        </template>
    </Draggable>
    <MonitorSettingDialog ref="monitorSettingDialog" />
</template>

<script>
import MonitorSettingDialog from "./MonitorSettingDialog.vue";
import Draggable from "vuedraggable";
import HeartbeatBar from "./HeartbeatBar.vue";
import Uptime from "./Uptime.vue";
import Tag from "./Tag.vue";


export default {
    components: {
        MonitorSettingDialog,
        Draggable,
        HeartbeatBar,
        Uptime,
        Tag,
    },
    props: {
        /** Are we in edit mode? */
        editMode: {
            type: Boolean,
            required: true,
        },
        /** Should tags be shown? */
        showTags: {
            type: Boolean,
        },
        /** Should expiry be shown? */
        showCertificateExpiry: {
            type: Boolean,
        }
    },
    name: 'PublicGroupList',
    data() {
        return {
            searchText: '',
        };
    },
    computed: {
        showGroupDrag() {
            return (this.$root.publicGroupList.length >= 2);
        }
    },
    created() {

    },
    methods: {
        /**
         * Remove the specified group
         * @param {number} index Index of group to remove
         */
        removeGroup(index) {
            this.$root.publicGroupList.splice(index, 1);
        },

        /**
         * Remove a monitor from a group
         * @param {number} groupIndex Index of group to remove monitor
         * from
         * @param {number} index Index of monitor to remove
         */
        removeMonitor(groupIndex, index) {
            this.$root.publicGroupList[groupIndex].monitorList.splice(index, 1);
        },

        /**
         * Should a link to the monitor be shown?
         * Attempts to guess if a link should be shown based upon if
         * sendUrl is set and if the URL is default or not.
         * @param {Object} monitor Monitor to check
         * @param {boolean} [ignoreSendUrl=false] Should the presence of the sendUrl
         * property be ignored. This will only work in edit mode.
         * @returns {boolean}
         */
        showLink(monitor, ignoreSendUrl = false) {
            // We must check if there are any elements in monitorList to
            // prevent undefined errors if it hasn't been loaded yet
            if (this.$parent.editMode && ignoreSendUrl && Object.keys(this.$root.monitorList).length) {
                return this.$root.monitorList[monitor.element.id].type === "http" || this.$root.monitorList[monitor.element.id].type === "keyword" || this.$root.monitorList[monitor.element.id].type === "json-query";
            }
            return monitor.element.sendUrl && monitor.element.url && monitor.element.url !== "https://" && !this.editMode;
        },

        /**
         * Returns formatted certificate expiry or Bad cert message
         * @param {Object} monitor Monitor to show expiry for
         * @returns {string}
         */
        formattedCertExpiryMessage(monitor) {
            if (monitor?.element?.validCert && monitor?.element?.certExpiryDaysRemaining) {
                return monitor.element.certExpiryDaysRemaining + " " + this.$tc("day", monitor.element.certExpiryDaysRemaining);
            } else if (monitor?.element?.validCert === false) {
                return this.$t("noOrBadCertificate");
            } else {
                return this.$t("Unknown") + " " + this.$tc("day", 2);
            }
        },

        /**
         * Returns certificate expiry based on days remaining
         * @param {Object} monitor Monitor to show expiry for
         * @returns {string}
         */
        certExpiryColor(monitor) {
            if (monitor?.element?.validCert && monitor.element.certExpiryDaysRemaining > 7) {
                return "#059669";
            }
            return "#DC2626";
        },
        filteredMonitorList(group) {
            if (!group || !group.element || !group.element.monitorList) {
                return [];
            }
            const search = this.searchText.toLowerCase();
            if (!search) return group.element.monitorList;

            return group.element.monitorList.filter(monitor => {
                return monitor?.name?.toLowerCase().includes(search);
            });
        },
        refreshPage() {
            window.location.reload();
        },
    }
};
</script>

<style lang="scss" scoped>
@import "../assets/vars";

.extra-info {
    display: flex;
    margin-bottom: 0.5rem;
}

.extra-info > div > div:first-child {
    margin-left: 0 !important;
}

.no-monitor-msg {
    position: absolute;
    width: 100%;
    top: 20px;
    left: 0;
}

.monitor-list {
    min-height: 46px;
}

.item-name {
    // padding-left: 5px;
    // padding-right: 5px;
    // margin: 0;
    // display: inline-block;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 3;
    line-clamp: 3;
    align-self: stretch;
    overflow: hidden;
    color:#0D0F13;
    text-overflow: ellipsis;
    font-size: 24px;
    line-height: 30px;
    margin: 0;
    margin-bottom: 5px;
}

.btn-link {
    color: #bbbbbb;
    margin-left: 5px;
}

.link-active {
    color: $primary;
}

.flip-list-move {
    transition: transform 0.5s;
}

.no-move {
    transition: transform 0s;
}

.drag {
    color: #bbb;
    cursor: grab;
}

.remove {
    color: $danger;
}

.group-title {
    span {
        display: inline-block;
        min-width: 15px;
    }
    flex: 1 0 0;
    align-self: stretch;
    color: #000;
    font-size: 32px;
    line-height: 40px;
    letter-spacing: -0.16px;
    font-weight: 700;
    display: flex;
    align-items: flex-end;
    margin: 0;
}

.mobile {
    .item {
        padding: 13px 0 10px;
    }
}

.bg-maintenance {
    background-color: $maintenance;
}

.search-bar {
    width: 16rem;
}

.flex-div {
    display: flex;
    padding: 0 12px 0px 48px;
    align-items: flex-end;
    gap: 80px;
    align-self: stretch;
}

.refresh-container {
    display: flex;
    padding: 32px 48px 0px 48px;
    align-items: flex-end;
    align-self: stretch;
    align-items: center;
}

.column-flex {
    display: flex;
    flex-direction: column;
}

.big-row {
    width: 925px;
}

.status-page-item-list {
    display: inline-block;
    margin-bottom: 22px;
    margin-left: 30px;
}

.refresh-button {
    display: flex;
    align-items: flex-start;
    gap: 4px;
    color: #0058EF;
    font-style: normal;
    letter-spacing: 0.08px;
}

.search-container {
    display: inline-flex;
    flex: 1 0 0;
    justify-content: flex-start;
    align-items: center;
    gap: 12px;
    align-self: stretch;
    border-radius: 8px 8px 0px 0px;
    border-bottom: 1px solid #DBDFE6;
    margin: 48px 0 36px 48px;
}

.search-box {
    color: #868FA2;
    font-weight: 400;
    width: 500px;
    height: 40px;
}

</style>
