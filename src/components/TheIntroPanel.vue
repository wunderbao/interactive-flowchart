<template>
  <PrimaryButton
    class="open"
    icon="info"
    title="Open introduction"
    :tabindex="viewStore.introPanelVisible ? -1 : ''"
    @click="$emit('toggleIntroPanel')"
  />
  <div ref="intro" class="intro" :class="{ visible: viewStore.introPanelVisible }">
    <div class="outer">
      <div class="inner">
        <h1>I Want A Better Catastrophe</h1>
        <h2>A flowchart for navigating our climate predicament</h2>
        <div v-if="resetActionAvailable" class="reset">
          <span>Want to re-explore from the beginning?</span>
          <button @click="clearLocalStorageAndReload()">Reset progress and start over</button>
        </div>
        <div class="description" v-html="description" />
      </div>
    </div>
    <button class="close" title="Close introduction" @click="$emit('toggleIntroPanel')"></button>
  </div>
</template>

<script>
import { mapStores, mapState, mapWritableState, mapActions } from 'pinia';
import { marked } from 'marked';

import PrimaryButton from '@/components/PrimaryButton.vue';

import { useFlowchartStore } from '@/stores/FlowchartStore.js';
import { useViewStore } from '@/stores/ViewStore.js';

export default {
  name: 'TheIntroPanel',

  components: {
    PrimaryButton
  },

  emits: [
    'togglePlayback',
    'toggleIntroPanel'
  ],

  data() {
    return {
      resetAppearanceDelay: 500,
      description: ''
    }
  },
  
  computed: {
    ...mapStores(
      useViewStore
    ),
    ...mapState(useFlowchartStore, [
      'currentNodeId'
    ]),
    ...mapWritableState(useFlowchartStore, [
      'resetActionAvailable'
    ])
  },

  methods: {
    ...mapActions(useFlowchartStore, [
      'clearLocalStorageAndReload'
    ]),
    async fetchDescription() {
      fetch('intro.md')
        .then(response => response.text())
        .then(data => {
          const html = marked(data.replaceAll('# ', '### '));
          this.description = html;
        })
        .catch(error => console.log(error));
    }
  },

  watch: {
    // when the node ID changes for the first time, unhide reset prompt
    currentNodeId() {
      if (!this.resetActionAvailable) {
        setTimeout(() => {
          this.resetActionAvailable = true;
        }, this.resetAppearanceDelay);
      }
    },
    // make interactive elements non-keyboard-selectable when intro panel is hidden
    'viewStore.introPanelVisible'() {
      if (this.viewStore.introPanelVisible) {
        this.$refs.intro.querySelectorAll('a, button').forEach(element => {
          element.removeAttribute('tabindex');
        });
      } else {
        this.$refs.intro.querySelectorAll('a, button').forEach(element => {
          element.setAttribute('tabindex', -1);
        });
      }
    }
  },

  created() {
    this.fetchDescription();
  }
}
</script>

<style lang="scss">
@import '@/assets/variables.css';

:root {
  --panel-width: 416px;
}

button.open {
  position: absolute;
  top: 16px;
  left: 16px;
  box-shadow: 0 0 0 2px var(--background-color);
}

.intro {
  position: absolute;
  z-index: 100;
  overflow: hidden;
  top: 0;
  bottom: 0;
  left: 0;
  width: 0;
  color: rgba(255,255,255,0.9);
  background: var(--intro-background-color);
  transition: width var(--transition-duration-long) var(--transition-timing);

  &.visible {
    width: var(--panel-width);

    &:after {
      opacity: 1;
    }

    .inner > *, .inner .instructions *:not(.reset) {
      opacity: 1;
    }
  }

  &:before, &:after {
    position: absolute;
    display: block;
    content: '';
    left: 0;
    width: var(--panel-width);
    pointer-events: none;
  }

  &:before {
    z-index: 1;
    bottom: 0;
    height: var(--panel-width);
    background: linear-gradient(180deg, rgba(62,62,62,0) 0%, #3E3E3E 100%), linear-gradient(180deg, rgba(62,62,62,0) 0%, #3E3E3E 100%);
  }

  &:after {
    z-index: 2;
    opacity: 0;
    bottom: 80px;
    height: 340px;
    background-size: var(--panel-width);
    background-position: center bottom;
    background-repeat: no-repeat;
    background-image: url('@/assets/tentacles.png');
    transition: opacity var(--transition-duration-long) var(--transition-timing);
  }

  *::selection {
    color: #fff;
    background: rgb(var(--accent-color));
  }

  .outer {
    position: absolute;
    overflow-y: auto;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
  }

  .inner {
    position: relative;
    width: var(--panel-width);
    padding: 24px 28px var(--panel-width);
    box-sizing: border-box;

    > *, .instructions *:not(.reset) {
      opacity: 0;
      transition: opacity var(--transition-duration-long) var(--transition-timing);
    }
  }

  h1 {
    max-width: 284px;
    margin: 0;
    font-size: 30px;
    font-weight: 600;
    line-height: 100%;
    letter-spacing: 0.03em;
    text-transform: uppercase;
  }

  h2 {
    max-width: 256px;
    margin: 6px 0 20px;
    font-size: 20px;
    font-weight: 400;
    line-height: 105%;
  }

  h3 {
    margin: 24px 0 0;
    font-size: 15px;
    font-weight: 600;
    line-height: 105%;
    letter-spacing: 0.01em;
    text-transform: uppercase;
  }

  p {
    margin: 12px 0;
    font-family: 'Vesper Libre', serif;
    font-size: 16px;
    line-height: 130%;

    strong {
      display: block;
      margin: 6px 0 0;
    }
  }

  a {
    color: inherit;
    text-decoration-color: rgba(var(--accent-color), 0.6);
    text-decoration-thickness: 2px;
    text-underline-offset: 2px;
    transition: color var(--transition-duration) var(--transition-timing);

    &:hover {
      color: rgb(var(--accent-color));
    }
  }

  .instructions {
    position: relative;
    display: grid;
    align-items: start;
    opacity: 1 !important;
    margin: 20px 0 72px;

    > * {
      grid-column-start: 1;
      grid-row-start: 1;
    }

    img {
      display: block;
      width: 100%;
      max-width: 290px;
      margin: 0 auto;
    }

    .reset {
      display: flex;
      flex-direction: column;
      justify-content: center;
      box-sizing: border-box;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
      min-height: 118px;
      margin: 2px -12px 0 -12px;
      padding: 16px 24px 20px;
      text-align: center;
      text-wrap: balance;
      line-height: 1.25;
      border-radius: 8px;
      background: rgba(255,255,255,0.08);
      -webkit-backdrop-filter: blur(24px);
      backdrop-filter: blur(24px);

      button {
        display: block;
        margin: 12px auto 2px;
        padding: 8px 16px;
        appearance: none;
        font-weight: 600;
        border: none;
        border-radius: 64px;
        color: #fff;
        background: rgb(var(--accent-color));
        cursor: pointer;
        transition: opacity var(--transition-duration-long) var(--transition-timing), transform var(--transition-duration) var(--transition-timing) !important;

        &:hover {
          transform: scale(1.0625);
        }
      }
    }
  }

  .logos {
    display: flex;
    align-items: top;
    gap: 32px;
    margin: 58px 0 -8px;

    a {
      flex: 1;
      transition: opacity var(--transition-duration) var(--transition-timing);

      &:first-child {
        flex-basis: 25%;
      }

      &:hover {
        opacity: 0.65;
      }

      img {
        width: 100%;
      }
    }
  }

  button.close {
    position: absolute;
    opacity: 0;
    top: 12px;
    right: 12px;
    width: 32px;
    height: 32px;
    padding: 0;
    appearance: none;
    border: none;
    border-radius: 100%;
    background-position: center;
    background-image: url('@/assets/icons/close.svg');
    background-color: rgba(111,111,111,0.75);
    -webkit-backdrop-filter: blur(16px);
    backdrop-filter: blur(16px);
    cursor: pointer;
    transition-property: opacity, transform, box-shadow;
    transition-duration: var(--transition-duration-long), var(--transition-duration), var(--transition-duration);
    transition-timing-function: var(--transition-timing);

    &:hover {
      transform: scale(1.125);
    }

    @at-root .intro.visible button.close {
      opacity: 1;
    }
  }

  *:focus-visible {
    position: relative;
    background: var(--intro-background-color);
    box-shadow: 0 0 0 2px var(--intro-background-color), 0 0 0 4px var(--focus-color) !important;
  }
}

@media (max-width: 600px) {
  .intro.visible,
  .intro:before,
  .intro:after,
  .intro .inner {
    width: 100vw;
  }
}

@media (max-height: 650px) {
  .intro:before {
    height: 33vh;
  }
  
  .intro:after {
    display: none;
  }

  .intro .inner {
    padding-bottom: 33vh;
  }
}
</style>