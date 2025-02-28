<template>
    <nav data-component="FILENAME_PLACEHOLDER" class="d-flex w-100 gap-3 top-bar" v-if="displayNavBar">
        <div class="d-flex flex-column flex-grow-1 flex-shrink-1 overflow-hidden top-title">
            <el-breadcrumb v-if="breadcrumb">
                <el-breadcrumb-item v-for="(item, x) in breadcrumb" :key="x">
                    <router-link :to="item.link">
                        {{ item.label }}
                    </router-link>
                </el-breadcrumb-item>
            </el-breadcrumb>
            <h1 class="h5 fw-semibold m-0 d-inline-fle">
                <slot name="title">
                    {{ title }}
                </slot>
                <el-button
                    class="star-button"
                    :class="{'star-active': starred}"
                    :icon="StarOutlineIcon"
                    circle
                    @click="onStarClick"
                />
            </h1>
        </div>
        <div class="d-lg-flex side gap-2 flex-shrink-0 align-items-center mycontainer">
            <div class="d-none d-lg-flex align-items-center">
                <global-search class="trigger-flow-guided-step" />
            </div>
            <div class="d-flex side gap-2 flex-shrink-0 align-items-center">
                <el-button v-if="shouldDisplayDeleteButton && logs !== undefined && logs.length > 0" @click="deleteLogs()">
                    <TrashCan class="me-2" />
                    <span>{{ $t("delete logs") }}</span>
                </el-button>
            </div>
            <slot name="additional-right" />
            <div class="d-flex fixed-buttons icons">
                <el-dropdown popper-class="">
                    <el-button class="no-focus dropdown-button">
                        <HelpBox />
                    </el-button>
                    <template #dropdown>
                        <el-dropdown-menu>
                            <a
                                href="https://kestra.io/slack?utm_source=app&utm_campaign=slack&utm_content=top-nav-bar"
                                target="_blank"
                                class="d-flex gap-2 el-dropdown-menu__item"
                            >
                                <HelpBox class="align-middle" /> {{ $t("live help") }}
                            </a>
                            <a
                                v-if="tourEnabled"
                                @click="restartGuidedTour"
                                class="d-flex gap-2 el-dropdown-menu__item"
                            >
                                <ProgressQuestion class="align-middle" /> {{ $t('Reset guided tour') }}
                            </a>

                            <router-link
                                class="d-flex gap-2 el-dropdown-menu__item"
                                :to="{name: 'docs/view'}"
                            >
                                <BookMultipleOutline class="align-middle" /> {{ $t("documentation.documentation") }}
                            </router-link>

                            <a
                                href="https://github.com/kestra-io/kestra/issues"
                                target="_blank"
                                class="d-flex gap-2 el-dropdown-menu__item"
                            >
                                <Github class="align-middle" /> {{ $t("documentation.github") }}
                            </a>
                            <a
                                href="https://kestra.io/slack?utm_source=app&utm_campaign=slack&utm_content=top-nav-bar"
                                target="_blank"
                                class="d-flex gap-2 el-dropdown-menu__item"
                            >
                                <Slack class="align-middle" /> {{ $t("join community") }}
                            </a>
                            <a
                                href="https://kestra.io/demo?utm_source=app&utm_campaign=sales&utm_content=top-nav-bar"
                                target="_blank"
                                class="d-flex gap-2 el-dropdown-menu__item"
                            >
                                <EmailHeartOutline class="align-middle" /> {{ $t("reach us") }}
                            </a>
                            <a
                                v-if="version"
                                :href="version.url"
                                target="_blank"
                                class="d-flex gap-2 el-dropdown-menu__item"
                            >
                                <Update class="align-middle text-danger" /> <span class="text-danger">{{ $t("new version", {"version": version.latest}) }}</span>
                            </a>
                        </el-dropdown-menu>
                    </template>
                </el-dropdown>

                <impersonating />
                <auth />
            </div>
        </div>
    </nav>
</template>

<script>
    import {mapState, mapGetters} from "vuex";
    import Auth from "override/components/auth/Auth.vue";
    import Impersonating from "override/components/auth/Impersonating.vue";
    import HelpBox from "vue-material-design-icons/HelpBox.vue";
    import BookMultipleOutline from "vue-material-design-icons/BookMultipleOutline.vue";
    import Github from "vue-material-design-icons/Github.vue";
    import Slack from "vue-material-design-icons/Slack.vue";
    import EmailHeartOutline from "vue-material-design-icons/EmailHeartOutline.vue";
    import Update from "vue-material-design-icons/Update.vue";
    import ProgressQuestion from "vue-material-design-icons/ProgressQuestion.vue";
    import GlobalSearch from "./GlobalSearch.vue";
    import TrashCan from "vue-material-design-icons/TrashCan.vue";
    import StarOutlineIcon from "vue-material-design-icons/StarOutline.vue";
    import StarIcon from "vue-material-design-icons/Star.vue";


    export default {
        components: {
            Auth,
            HelpBox,
            BookMultipleOutline,
            Github,
            Slack,
            EmailHeartOutline,
            Update,
            ProgressQuestion,
            GlobalSearch,
            TrashCan,
            Impersonating
        },
        props: {
            title: {
                type: String,
                required: true
            },
            breadcrumb: {
                type: Array,
                default: undefined
            },
        },
        computed: {
            ...mapState("api", ["version"]),
            ...mapState("core", ["tutorialFlows"]),
            ...mapState("log", ["logs"]),
            ...mapState("starred", ["pages"]),
            ...mapGetters("core", ["guidedProperties"]),
            ...mapGetters("auth", ["user"]),
            displayNavBar() {
                return this.$route?.name !== "welcome";
            },
            tourEnabled(){
                // Temporary solution to not showing the tour menu item for EE
                return this.tutorialFlows?.length && !Object.keys(this.user).length
            },
            shouldDisplayDeleteButton() {
                return this.$route.name === "flows/update" && this.$route.params?.tab === "logs"
            },
            StarOutlineIcon() {
                return this.starred ? StarIcon : StarOutlineIcon
            },
            starred() {
                return this.pages.some(page => page.path === this.currentFavURI)
            },
            currentFavURI() {
                // make sure the value changes when the route changes
                // by mentionning the route in the computed properties
                // we create a hook into vues reactivity system to update when it updates
                if(this.$route) {
                    return window.location.pathname
                        + window.location.search
                            // remove the parameters that are permanently changing
                            .replace(/&?page=[^&]*/ig, "")
                            // fix if this resulted in a "?&" url
                            .replace(/\?&/, "?")
                }
                return ""
            }
        },
        methods: {
            restartGuidedTour() {
                localStorage.setItem("tourDoneOrSkip", undefined);
                this.$store.commit("core/setGuidedProperties", {tourStarted: false});

                this.$tours["guidedTour"]?.start();
            },
            deleteLogs() {
                this.$toast().confirm(
                    this.$t("delete_all_logs"),
                    () => this.$store.dispatch("log/deleteLogs", {namespace: this.namespace, flowId: this.flowId}),
                    () => {}
                )
            },
            onStarClick() {
                if (this.starred) {
                    this.$store.dispatch("starred/remove", {
                        path: this.currentFavURI
                    })
                } else {
                    console.log(this.title, this.breadcrumb)
                    this.$store.dispatch("starred/add", {
                        path: this.currentFavURI,
                        label: this.breadcrumb?.length ? `${this.breadcrumb[0].label}: ${this.title}` : this.title,
                    })
                }
            }
        },
    };
</script>

<style lang="scss" scoped>
    nav {
        top: 0;
        position: sticky;
        z-index: 1000;
        padding: var(--spacer) calc(2 * var(--spacer));
        border-bottom: 1px solid var(--bs-border-color);
        background: var(--card-bg);

        .top-title, h1, .el-breadcrumb {
            white-space: nowrap;
            max-width: 100%;
            text-overflow: ellipsis;
            overflow: hidden;
        }

        h1 {
            line-height: 1.6;
            display: flex !important;
            align-items: center;
        }

        .star-button{
            margin-left: var(--spacer);
            border: none;
        }

        .star-active {
            color: #9470FF;
        }

        :deep(.el-breadcrumb__item) {
            display: inline-block;
        }


        :deep(.el-breadcrumb__inner) {
            white-space: nowrap;
            max-width: 100%;
            text-overflow: ellipsis;
            overflow: hidden;
        }

        .side {
            .fixed-buttons {
                align-items: center;

                button, :deep(button), a, :deep(a) {
                    border: none;
                    font-size: var(--font-size-lg);
                    padding: calc(var(--spacer) / 4);
                }
            }

            :slotted(ul) {
                display: flex;
                list-style: none;
                padding: 0;
                margin: 0;
                gap: calc(var(--spacer) / 2);
                align-items: center;
            }
        }
        @media (max-width: 768px) {
            .mycontainer{
                display:grid;
                grid-template-columns:repeat(3, minmax(0,auto));
                grid-template-rows: repeat(2, auto);
                gap:10px;
                overflow: hidden;


            }
            .icons{
                grid-row:2;
                grid-column:2;
                display: contents;
            }

        }
        @media (max-width: 664px){
            .mycontainer{
                display:grid;
                grid-template-columns:repeat(2, minmax(0,auto));
                grid-template-rows: repeat(2, auto);
                gap:10px;
                overflow: hidden;

            }
        }
    }
</style>
