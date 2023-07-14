<template>
    <div
        class="bg-zinc-900 h-screen flex overflow-hidden"
        :class="settings.menuBarTop ? 'flex-col' : 'flex-col-reverse'"
    >
        <!-- Search bar -->
        <div
            v-show="searchVisible"
            class="fixed top-0 left-0 w-full h-full flex flex-col items-center bg-zinc-900 bg-opacity-50"
        >
            <div class="h-[46vh]"></div>

            <div class="relative">
                <input
                    v-model="search"
                    type="text"
                    class="w-[12em] bg-zinc-900 text-4xl px-2 font-mono focus:outline-none caret-zinc-900"
                    placeholder=""
                    ref="search"
                    @blur="hideSearch"
                    @input="searchChange"
                    @keydown="searchChange"
                    @keyup="searchChange"
                    @click="searchChange"
                />

                <!-- Text on top of caret -->
                <span
                    class="absolute inset-0 w-[12em] pt-1 text-4xl px-2 font-mono z-10 pointer-events-none"
                >
                    {{ search }}</span
                >

                <!-- Blinking caret -->
                <span
                    class="absolute w-4 h-full bg-zinc-800 ml-1 caret"
                    ref="caret"
                    :style="{
                        left: caretPosition * 1.36 + 0.5 + 'em',
                    }"
                ></span>
            </div>

            <!-- Results -->
            <div
                class="flex items-center justify-center bg-zinc-900 bg-opacity-50 overflow-x-scroll"
                v-show="searchVisible && search.length > 0"
            >
                <div
                    class="flex flex-col gap-2 p-2 w-[12em] text-4xl font-mono bg-zinc-900 z-20"
                >
                    <div
                        v-for="(link, index) in filteredLinks"
                        :key="index"
                        class="flex"
                    >
                        <span
                            v-if="index === searchOffset"
                            class="text-teal-300 mr-2"
                        >
                            >
                        </span>
                        {{ link.item.title }}
                    </div>
                </div>
            </div>
        </div>

        <!-- Add new link -->
        <div
            v-show="newLinkVisible"
            class="fixed top-0 left-0 w-full h-full flex items-center justify-center bg-zinc-900 bg-opacity-50"
        >
            <form
                class="flex flex-col gap-2 p-2 w-[12em] text-4xl font-mono bg-zinc-900 items-center z-20"
                @submit.prevent="
                    addLink(
                        $refs.url.value,
                        $refs.title.value,
                        $refs.tags.value
                    )
                "
            >
                <!-- URL -->
                <input
                    type="text"
                    class="w-full h-8 bg-zinc-800 text-2xl px-2 font-mono border border-opacity-0 focus:border-opacity-100 border-teal-300 focus:outline-none"
                    placeholder="URL"
                    ref="url"
                />

                <!-- Title -->
                <input
                    type="text"
                    class="w-full h-8 bg-zinc-800 text-2xl px-2 font-mono border border-opacity-0 focus:border-opacity-100 border-teal-300 focus:outline-none"
                    placeholder="TITLE"
                    ref="title"
                />

                <!-- Tags -->
                <input
                    type="text"
                    class="w-full h-8 bg-zinc-800 text-2xl px-2 font-mono border border-opacity-0 focus:border-opacity-100 border-teal-300 focus:outline-none"
                    placeholder="TAG1,TAG2,TAG3"
                    ref="tags"
                />

                <!-- Submit button -->
                <button
                    type="submit"
                    class="w-full h-8 bg-zinc-800 text-2xl px-2 font-mono border border-opacity-0 hover:border-opacity-100 border-teal-300 focus:outline-none"
                >
                    ADD
                </button>
            </form>

            <!-- Form background for click events -->
            <div
                class="fixed top-0 left-0 w-full h-full"
                @click="hideNewLink"
            ></div>
        </div>

        <!-- Menu bar -->
        <div
            class="w-full h-4 hover:h-16 flex gap-4 justify-center font-mono transition-all duration-100 ease-in-out z-50"
            :class="settings.menuBarTop ? 'mb-px' : 'mt-px'"
        >
            <div class="flex gap-[2px]">
                <button
                    class="w-16 h-full flex items-center justify-center hover:bg-zinc-800"
                    @click="openSearch"
                >
                    <!-- Search svg -->
                    <svg
                        class="w-4 h-4"
                        xmlns="http://www.w3.org/2000/svg"
                        viewBox="0 0 20 20"
                        fill="currentColor"
                    >
                        <path
                            fill-rule="evenodd"
                            d="M9 3a6 6 0 014.95 9.475l4.263 4.263a1 1 0 11-1.414 1.414l-4.263-4.263A6 6 0 119 3zm0 2a4 4 0 100 8 4 4 0 000-8z"
                            clip-rule="evenodd"
                        />
                    </svg>
                </button>

                <button
                    class="w-6 h-full flex items-center justify-center hover:bg-zinc-800"
                    @click="newLinkVisible = true"
                >
                    <!-- Plus svg -->
                    <svg
                        class="w-4 h-4 rotate-45"
                        xmlns="http://www.w3.org/2000/svg"
                        viewBox="0 0 20 20"
                        fill="currentColor"
                    >
                        <path
                            fill-rule="evenodd"
                            d="M5.293 5.293a1 1 0 011.414 0L10 8.586l3.293-3.293a1 1 0 111.414 1.414L11.414 10l3.293 3.293a1 1 0 01-1.414 1.414L10 11.414l-3.293 3.293a1 1 0 01-1.414-1.414L8.586 10 5.293 6.707a1 1 0 010-1.414z"
                            clip-rule="evenodd"
                        />
                    </svg>
                </button>
            </div>

            <div class="flex-1 overflow-x-scroll overflow-y-clip">
                <div class="flex h-full">
                    <div
                        v-for="(link, index) in links"
                        :key="index"
                        class="flex h-full justify-center items-center hover:bg-teal-300 hover:text-zinc-900 px-2 group whitespace-nowrap"
                    >
                        <button class="flex items-center h-full translate-y-px">
                            <!-- X svg -->
                            <svg
                                class="w-0 group-hover:w-4 h-4 hover:text-[#EA5D72] mr-1"
                                xmlns="http://www.w3.org/2000/svg"
                                viewBox="0 0 20 20"
                                fill="currentColor"
                                @click="removeLink(index)"
                            >
                                <path
                                    fill-rule="evenodd"
                                    d="M5.293 5.293a1 1 0 011.414 0L10 8.586l3.293-3.293a1 1 0 111.414 1.414L11.414 10l3.293 3.293a1 1 0 01-1.414 1.414L10 11.414l-3.293 3.293a1 1 0 01-1.414-1.414L8.586 10 5.293 6.707a1 1 0 010-1.414z"
                                    clip-rule="evenodd"
                                />
                            </svg>
                        </button>

                        <a
                            @click="updateCurrentLink(index)"
                            class="h-full flex items-center cursor-pointer -translate-y-px"
                        >
                            {{ link.title }}
                        </a>
                    </div>
                </div>
            </div>

            <!-- Move menu bar to bottom button -->
            <button
                class="w-6 h-full flex items-center justify-center hover:bg-zinc-800"
                @click="settings.menuBarTop = !settings.menuBarTop"
            >
                <span :class="settings.menuBarTop ? 'rotate-180' : ''">
                    <!-- svg of down arrow with line at top -->
                    <svg
                        class="w-4 h-4"
                        xmlns="http://www.w3.org/2000/svg"
                        viewBox="0 0 20 20"
                        fill="currentColor"
                    >
                        <path
                            fill-rule="evenodd"
                            d="M10 3a1 1 0 01.707.293l4 4a1 1 0 01-1.414 1.414L11 6.414V16a1 1 0 11-2 0V6.414L6.707 8.707A1 1 0 115.293 7.293l4-4A1 1 0 0110 3z"
                            clip-rule="evenodd"
                        />
                    </svg>
                </span>
            </button>
        </div>

        <iframe
            ref="iframe"
            frameborder="0"
            allow="fullscreen"
            :src="currentLink !== -1 ? links[currentLink].url : ''"
            class="w-full h-full"
            pointer-events="none"
        ></iframe>
    </div>
</template>

<script>
import Fuse from "fuse.js";

const links = JSON.parse(localStorage.getItem("links")) || [];
const settings = JSON.parse(localStorage.getItem("settings")) || {};

const fuseOptions = {
    keys: ["title", "tags"],
};
let fuse = new Fuse(links, fuseOptions);

export default {
    name: "App",
    emits: [],
    components: {},
    props: {},
    data() {
        return {
            searchVisible: false,
            search: "",

            newLinkVisible: false,

            settings: {
                menuBarTop: true,
                ...settings,
            },

            links,
            currentLink: -1,
            searchOffset: 0,

            caretPosition: 0,
        };
    },
    methods: {
        openSearch() {
            this.search = "";
            this.caretPosition = 0;
            this.searchOffset = 0;
            this.searchVisible = true;

            setTimeout(() => {
                this.$refs.search.focus();
            }, 0);
        },

        hideSearch() {
            // If currentLink is -1, force search to be visible
            if (this.currentLink === -1 && !this.newLinkVisible) {
                this.openSearch();
                return;
            }

            this.searchVisible = false;
        },

        searchChange(e) {
            // Limit search to 18 characters
            this.search = this.search.slice(0, 18);

            this.caretPosition = e.target.selectionStart;

            // Remove and re-add caret class to force it to blink
            const caret = this.$refs.caret;
            caret.classList.remove("caret");

            setTimeout(() => {
                caret.classList.add("caret");
            }, 0);
        },

        hideNewLink() {
            // If currentLink is -1, force search to be visible
            if (this.currentLink === -1) {
                this.openSearch();
            }

            this.newLinkVisible = false;
        },

        addLink(url, title, tags) {
            this.links.push({
                url,
                title,
                tags,
            });
        },

        removeLink(index) {
            // Remove link from links array
            this.links.splice(index, 1);
        },

        updateCurrentLink(link) {
            // If currentLink is -1, force search to be visible
            if (link === -1) {
                this.openSearch();
                return;
            }

            // Update iframe src
            this.currentLink = link;
            this.hideSearch();
        },
    },
    mounted() {
        const keyWindowListener = (e) => {
            if (e.key === "Escape") {
                this.hideSearch();
                this.hideNewLink();
            }

            if (e.key === "ArrowUp") {
                if (this.searchOffset > 0) {
                    this.searchOffset--;
                }
            }

            if (e.key === "ArrowDown") {
                if (this.searchOffset < this.filteredLinks.length - 1) {
                    this.searchOffset++;
                }
            }

            if (e.key === "Enter") {
                this.updateCurrentLink(
                    this.filteredLinks.at(this.searchOffset)?.refIndex ?? -1
                );
            }
        };

        window.addEventListener("keydown", keyWindowListener);

        this.openSearch();
    },
    computed: {
        // Filtered links
        filteredLinks() {
            const filteredLinks = fuse.search(this.search);

            // Limit to 5 results
            filteredLinks.length = Math.min(filteredLinks.length, 5);

            // Ensure searchOffset is within bounds
            this.searchOffset = Math.max(
                Math.min(this.searchOffset, filteredLinks.length - 1),
                0
            );

            return filteredLinks;
        },
    },
    watch: {
        links: {
            handler() {
                // Save links to localStorage
                localStorage.setItem("links", JSON.stringify(this.links));

                // Update fuse
                fuse = new Fuse(this.links, fuseOptions);
            },
            deep: true,
        },

        // Save settings to localStorage
        settings: {
            handler() {
                localStorage.setItem("settings", JSON.stringify(this.settings));
            },
            deep: true,
        },
    },
};
</script>

<style scoped>
.bg-zinc-900::-moz-selection {
    /* Code for Firefox */
    @apply bg-zinc-800;
}

.bg-zinc-900::selection {
    @apply bg-zinc-800;
}

::-moz-selection {
    /* Code for Firefox */
    @apply bg-zinc-900;
}

::selection {
    @apply bg-zinc-900;
}

/* Blink should instantly transition from 0 to 1 opacity */
@keyframes blink {
    0% {
        opacity: 1;
    }
    40% {
        opacity: 1;
    }
    50% {
        opacity: 0;
    }
    90% {
        opacity: 0;
    }
    100% {
        opacity: 1;
    }
}
.caret {
    animation: 1s blink 0.25s infinite;
}
</style>
