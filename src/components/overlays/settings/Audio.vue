<!--
//  Audio.vue
//
//  Created by Kalila L. on May 11th, 2021.
//  Copyright 2021 Vircadia contributors.
//
//  Distributed under the Apache License, Version 2.0.
//  See the accompanying file LICENSE or http://www.apache.org/licenses/LICENSE-2.0.html
-->

<template>
    <OverlayShell
        icon="headphones"
        title="Audio"
        :managerProps="propsToPass"
        :defaultHeight="500"
        :defaultWidth="400"
    >
        <q-card
            class="column full-height"
            v-if="$store.state.Audio.input"
        >

            <q-tabs
                v-model="tab"
                active-color="primary"
                indicator-color="primary"
                align="justify"
                narrow-indicator
            >
                <q-tab name="input" label="Input" />
                <q-tab name="output" label="Output" />
            </q-tabs>

            <q-separator />

            <q-scroll-area class="col">
                <q-tab-panels v-model="tab" animated>
                    <q-tab-panel name="input">
                        <div class="row">
                            <div class=".col-4">
                                <q-btn
                                    fab
                                    class="q-mr-sm"
                                    :color="$store.state.Audio.input.hasInputAccess ? 'primary' : 'red'"
                                    :icon="$store.state.Audio.input.hasInputAccess ? 'mic' : 'mic_off'"
                                    @click="micToggled"
                                />
                            </div>

                            <div class=".col-8 row items-center">
                                <span
                                    v-if="selectedInputStore"
                                    class="text-subtitle1 items-center"
                                >
                                    Using {{ selectedInputStore.label }}
                                </span>

                                <span
                                    v-else
                                    class="text-subtitle1 items-center"
                                >
                                    No microphone selected.
                                </span>
                            </div>
                        </div>

                        <div
                            v-show="$store.state.Audio.input.hasInputAccess"
                            class="row q-mt-sm"
                        >
                            <q-btn
                                flat
                                dense
                                round
                                class="q-mr-sm"
                                :disabled="!$store.state.Audio.input.hasInputAccess"
                                :color="isListeningToFeedback ? 'primary' : 'red'"
                                :icon="isListeningToFeedback ? 'hearing' : 'hearing_disabled'"
                                @click="toggleInputFeedback"
                            />

                            <span
                                v-if="!isListeningToFeedback"
                                class="text-caption row items-center"
                            >
                                Click to test your microphone.
                            </span>

                            <span
                                v-else
                                class="text-caption row items-center"
                            >
                                Speak into your microphone to listen.
                            </span>

                            <audio ref="audioInputFeedbackPlayer"></audio>
                        </div>

                        <q-separator
                            class="q-my-md"
                        />

                        <div
                            v-if="$store.state.Audio.input.hasInputAccess === false"
                            class="text-subtitle1 text-grey text-center"
                        >
                            Please grant mic access to the app in order to speak.
                        </div>

                        <q-list v-else>
                            <div v-for="input in $store.state.Audio.input.inputsList" :key="input.deviceId">
                                <q-item v-show="input.label" tag="label" v-ripple>
                                    <q-item-section avatar>
                                        <q-radio
                                            @click="requestSpecificInputAccess(input.deviceId)"
                                            v-model="selectedInputStore"
                                            :val="input"
                                            color="teal"
                                        />
                                    </q-item-section>
                                    <q-item-section>
                                        <q-item-label>{{ input.label }}</q-item-label>
                                    </q-item-section>
                                </q-item>
                            </div>
                        </q-list>
                    </q-tab-panel>

                    <q-tab-panel name="output">
                        Coming soon!
                    </q-tab-panel>
                </q-tab-panels>
            </q-scroll-area>
        </q-card>

        <q-inner-loading :showing="!$store.state.Audio.input">
            <q-spinner-gears size="50px" color="primary" />
        </q-inner-loading>
    </OverlayShell>
</template>

<script>
import OverlayShell from '../OverlayShell.vue';

export default {
    name: 'Audio',

    props: {
        propsToPass: { type: Object, default: () => ({}), required: false }
    },

    components: {
        OverlayShell
    },

    data: () => ({
        tab: 'input',
        isListeningToFeedback: false
    }),

    computed: {
        selectedInputStore: {
            get () {
                return this.$store.state.Audio.input.currentInputDevice;
            },
            set () {
                // @click will set for us...
            }
        }
    },

    methods: {
        setAudioInputStream: function (stream) {
            if (!this.$refs.audioInputFeedbackPlayer) return;

            if (window.URL) {
                this.$refs.audioInputFeedbackPlayer.srcObject = stream;
            } else {
                this.$refs.audioInputFeedbackPlayer.src = stream;
            }
        },

        requestInputAccess: function () {
            this.$store.state.Audio.input.requestInputAccess()
                .then(this.setAudioInputStream);
        },

        requestSpecificInputAccess: function (deviceId) {
            this.$store.state.Audio.input.requestSpecificInputAccess(deviceId)
                .then(this.setAudioInputStream);
        },

        micToggled: function () {
            if (this.$store.state.Audio.input.hasInputAccess === true) {
                // Should mute/unmute
            } else {
                this.requestInputAccess();
            }
        },

        toggleInputFeedback: function () {
            this.isListeningToFeedback = !this.isListeningToFeedback;

            if (this.isListeningToFeedback === true) {
                this.$refs.audioInputFeedbackPlayer.muted = false;
                this.$refs.audioInputFeedbackPlayer.play();
            } else {
                this.$refs.audioInputFeedbackPlayer.pause();
            }
        }
    },

    created: function () {
    },

    mounted: function () {
        this.$nextTick(() => {
            this.requestInputAccess();
        });
    }
};
</script>
