<template>
  <div class="appcard relative">
    <div
      class="bg-secondary max-w-sm rounded overflow-hidden shadow-md px-2 py-3 my-1 flex flex-col align-center relative z-20"
    >
      <div class="appcard__controls flex justify-end">
        <button
          class="appcard__controls__btn menu-btn focus:outline-none rounded-full focus:shadow-primaryoutline"
          @click="isMenuExpanded = !isMenuExpanded"
        >
          <MenuIcon />
        </button>
      </div>
      <div class="flex flex-row">
        <div class="appcard__icon flex justify-center items-center mr-3">
          <div class="flex justify-center items-center">
            <img v-show="faviconUrl" :src="faviconUrl" />
            <LoadingCircleIcon v-show="!faviconUrl" />
          </div>
        </div>
        <dt class="appcard__name">
          <a :href="appDoc.url">
            <h4 class="text-lg truncate ...">{{ appDoc.name }}</h4>
          </a>
        </dt>
      </div>
      <div class="appcard__controls flex justify-end">
        <button
          class="appcard__controls__btn copy-btn focus:outline-none rounded-full focus:shadow-primaryoutline mx-2"
        >
          <CopyIcon />
        </button>
        <button
          :class="{
            'appcard__controls__btn expand-btn mx-2 \
            focus:outline-none rounded-full focus:shadow-primaryoutline': true,
            'appcard__controls__btn--is-expanded': isDetailsExpanded
          }"
          @click="isDetailsExpanded = !isDetailsExpanded"
        >
          <ExpandIcon />
        </button>
      </div>
    </div>
    <transition name="slide-in-top">
      <div
        v-show="isDetailsExpanded"
        class="
        appcard__details max-w-sm m-auto h-auto
        rounded overflow-hidden shadow-md 
        px-1 py-1 my-1 flex flex-col 
        relative z-10"
      >
        <p
          class="appcard__details__descr px-2 mb-1 h-24 overflow-hidden text-justify"
        >
          {{ appDoc.description }}
        </p>
        <div
          class="appcard__details__tags px-2 py-1 overflow-hidden overflow-x-scroll whitespace-no-wrap"
        >
          <span class="text-grey" v-if="appDoc.tags.length === 0">
            No tags available
          </span>
          <span
            v-else
            :class="{
              'bg-grey rounded-full text-white text-xxs p-1 mr-1':
                appDoc.tags.length
            }"
            :key="index"
            v-for="(tag, index) in appDoc.tags"
          >
            #{{ tag }}
          </span>
        </div>
      </div>
    </transition>

    <transition
      name="scale-up-down"
      enter-active-class="scale-up-tr"
      leave-active-class="scale-down-tr"
    >
      <AppCardMenu
        v-show="isMenuExpanded"
        @onCloseAppMenu="closeAppMenu"
        :isExpanded="isDetailsExpanded"
      />
    </transition>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator'
import { AppDocument } from '@/common/interfaces'
import AppCardMenu from '@/components/AppCardMenu.vue'
import MenuIcon from './icons/MenuIcon.svg'
import CopyIcon from './icons/CopyIcon.svg'
import ExpandIcon from './icons/ExpandIcon.svg'
import LoadingCircleIcon from './icons/LoadingCircleIcon.svg'
import axios from 'axios'

interface Icon {
  sizes?: string
  src: string
  type?: string
}

interface FavIcon {
  domain: string
  icons: Icon[]
}

@Component({
  components: {
    AppCardMenu,
    MenuIcon,
    CopyIcon,
    ExpandIcon,
    LoadingCircleIcon
  },
  // Don't add props as html attribute
  inheritAttrs: false
})
export default class AppCard extends Vue {
  public faviconUrl = ''
  public isDetailsExpanded = false
  public isMenuExpanded = false
  private readonly targetIconSize = '32'
  @Prop() private appDoc!: AppDocument
  @Prop() private docId!: PouchDB.Core.DocumentId
  @Prop() private docKey!: PouchDB.Core.DocumentKey
  @Prop() private docRev!: PouchDB.Core.RevisionId

  created() {
    this.getFaviconUrl(this.$props.appDoc.url)
  }

  async getFaviconUrl(appUrl: string) {
    const domain = new URL(appUrl).host
    const fallbackUrl = `https://www.google.com/s2/favicons?domain=${domain}`

    try {
      const response = await axios.get<FavIcon>(
        `http://favicongrabber.com/api/grab/${domain}`
      )
      const icons: Icon[] = response.data.icons

      const bySizeOrType = (icon: Icon) => {
        switch (icon.sizes ?? icon.type) {
          case '32x32':
          case 'image/png':
          case 'image/x-icon':
            return true
          default:
            return false
        }
      }

      const iconUrls = icons.filter(bySizeOrType).map(icon => icon.src)

      this.faviconUrl = iconUrls[0] !== undefined ? iconUrls[0] : fallbackUrl
    } catch (error) {
      this.faviconUrl = fallbackUrl
    }
  }

  closeAppMenu() {
    this.isMenuExpanded = false
  }
}
</script>

<style lang="scss">
.clamp {
  display: -webkit-box;
  -webkit-line-clamp: 3; // amount of line you want
  -webkit-box-orient: vertical;
}

.appcard {
  width: inherit;
  height: inherit;

  &__name {
    max-width: 220px;
  }

  &__icon {
    width: 36px;
  }

  &__icon div,
  &__icon svg {
    width: 24px;
    height: 24px;
  }

  &__controls {
    &__btn {
      width: 16px;
      height: 16px;
      transform: rotate(0deg);
      transition: transform ease-in-out 0.6s;
    }

    &__btn--is-expanded {
      transform: rotate(180deg);
    }
  }

  &__details {
    background-color: var(--color-bg);

    &__tags::-webkit-scrollbar {
      height: 5px;
    }
    &__tags {
      scrollbar-width: thin;
      scrollbar-color: var(--color-primary) var(--color-bg);
    }
    &__tags::-webkit-scrollbar-track {
      background: var(--color-bg);
    }
    &__tags::-webkit-scrollbar-thumb {
      background-color: var(--color-primary);
      border-radius: 9999px;
      border: 3px solid vvar(--color-bg);
    }
  }
}

.slide-in-top-enter-active {
  -webkit-animation: slide-in-top 0.6s both;
  animation: slide-in-top 0.6s both;
}

.slide-in-top-leave-active {
  -webkit-animation: slide-in-top 0.6s reverse both;
  animation: slide-in-top 0.6s reverse both;
}

.scale-up-tr {
  -webkit-animation: scale-up-tr 0.4s cubic-bezier(0.39, 0.575, 0.565, 1) both;
  animation: scale-up-tr 0.4s cubic-bezier(0.39, 0.575, 0.565, 1) both;
}

.scale-down-tr {
  -webkit-animation: scale-down-tr 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94)
    both;
  animation: scale-down-tr 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
}

/* ----------------------------------------------
 * Generated by Animista on 2020-3-29 22:54:37
 * Licensed under FreeBSD License.
 * See http://animista.net/license for more info. 
 * w: http://animista.net, t: @cssanimista
 * ---------------------------------------------- */

/**
 * ----------------------------------------
 * animation slide-in-top
 * ----------------------------------------
 */
@-webkit-keyframes slide-in-top {
  0% {
    -webkit-transform: translateY(-83px);
    transform: translateY(-83px);
    opacity: 0;
  }
  100% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    opacity: 1;
  }
}
@keyframes slide-in-top {
  0% {
    -webkit-transform: translateY(-83px);
    transform: translateY(-83px);
    opacity: 0;
  }
  100% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    opacity: 1;
  }
}

/**
 * ----------------------------------------
 * animation scale-up-tr
 * ----------------------------------------
 */
@-webkit-keyframes scale-up-tr {
  0% {
    -webkit-transform: scale(0.5);
    transform: scale(0.5);
    -webkit-transform-origin: 100% 0%;
    transform-origin: 100% 0%;
  }
  100% {
    -webkit-transform: scale(1);
    transform: scale(1);
    -webkit-transform-origin: 100% 0%;
    transform-origin: 100% 0%;
  }
}
@keyframes scale-up-tr {
  0% {
    -webkit-transform: scale(0.5);
    transform: scale(0.5);
    -webkit-transform-origin: 100% 0%;
    transform-origin: 100% 0%;
  }
  100% {
    -webkit-transform: scale(1);
    transform: scale(1);
    -webkit-transform-origin: 100% 0%;
    transform-origin: 100% 0%;
  }
}

/**
 * ----------------------------------------
 * animation scale-down-tr
 * ----------------------------------------
 */
@-webkit-keyframes scale-down-tr {
  0% {
    -webkit-transform: scale(1);
    transform: scale(1);
    -webkit-transform-origin: 100% 0%;
    transform-origin: 100% 0%;
  }
  100% {
    -webkit-transform: scale(0.5);
    transform: scale(0.5);
    -webkit-transform-origin: 100% 0%;
    transform-origin: 100% 0%;
  }
}
@keyframes scale-down-tr {
  0% {
    -webkit-transform: scale(1);
    transform: scale(1);
    -webkit-transform-origin: 100% 0%;
    transform-origin: 100% 0%;
  }
  100% {
    -webkit-transform: scale(0);
    transform: scale(0);
    -webkit-transform-origin: 100% 0%;
    transform-origin: 100% 0%;
  }
}
</style>
