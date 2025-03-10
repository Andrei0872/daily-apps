<template>
  <div class="page" :class="clsObj">
    <da-header @go="onGoClicked" @login="onLogin('Header')"
               @profile="onProfile" @menu="onDndMenu"></da-header>
    <da-dnd-message v-if="dndMode" @dndOff="onDisableDndMode"/>
    <da-sidebar ref="sidebar" :disabled="showBookmarks"
                @requested-source="showRequestModal = true"
                @login="onLogin('Sidebar')"></da-sidebar>
    <div class="line-numbers" @mouseenter="$refs.sidebar.open()">
      <svgicon icon="hamburger" class="line-numbers_icon"/>
      <div class="line-numbers__lines" ref="lineNumbers">
        <pre v-for="n in lineNumbers" class="micro2" :key="n">{{ n }}</pre>
      </div>
      <svg class="line-numbers__collapse" width="10" height="12" viewBox="0 0 10 12"
           xmlns="http://www.w3.org/2000/svg">
        <g fill="none" fill-rule="evenodd">
          <g>
            <path class="fill" d="M0 0h10v7.826L5 12 0 7.826z"></path>
            <path class="stroke" d="M.5.5v7.092L5 11.35 9.5 7.59V.5h-9z"></path>
          </g>
          <path d="M3 5h4" class="stroke" stroke-linecap="square"></path>
        </g>
      </svg>
    </div>
    <main class="content">
      <div class="content__header">
        <template v-if="filter && !showBookmarks">
          <button class="btn btn-nav content__header__back-home" @click="onBackHome">
            <svgicon icon="arrow"/>
            <span>Back Home</span>
          </button>
          <img :src="filter.info.image" :alt="filter.info.name"
               v-if="filter.type === 'publication'" class="content__header__pub-image"/>
          <h4>// {{ filter.info.name }}</h4>
          <transition name="fade">
            <button class="btn btn-water content__header__add-filter" v-if="!hasFilter"
                    @click="onAddFilter">
              <svgicon icon="plus"/>
              <span>Add To Feed</span>
            </button>
          </transition>
        </template>
        <template v-else>
          <h4 v-if="!emptyBookmarks" class="uppercase">/* {{ title }} */</h4>
          <a class="header__cta shadow1 " :href="cta.link" target="_blank"
             @mouseup="ctaClick" :style="cta.style">
            <span class="header__cta__text">// {{cta.text}}</span>
            <img class="header__cta__image" :src="`/logos/${cta.logo}.svg`" v-if="cta.logo"/>
            <svgicon class="header__cta__image" :icon="cta.icon" v-else/>
          </a>
        </template>
      </div>
      <div class="content__empty-bookmarks" v-if="emptyBookmarks">
        <da-svg :src="`/graphics/bookmark${theme === 'bright' ? '_bright' : ''}.svg`"
                class="bookmarks-placeholder"/>
        <h1 class="content__empty-bookmarks__title">Nothing here, yet</h1>
        <p class="content__empty-bookmarks__text">
          Bookmark articles on the main feed and it will be shown here.
        </p>
      </div>
      <da-feed/>
    </main>
    <div id="anchor" ref="anchor"></div>
    <da-go v-if="showGoModal" @close="showGoModal = false"/>
    <da-congrats v-if="showCongratsModal" @close="confirmNewUser"/>
    <da-request v-if="showRequestModal" @close="showRequestModal = false"/>
    <da-welcome v-if="showReadyModal" @close="nextInstruction"/>
    <da-login v-if="showLoginModal" @close="showLoginModal = false"/>
    <da-profile v-if="showProfileModal" @close="showProfileModal = false"/>
    <da-terminal v-if="showNotifications" class="notifications" @close="hideNotifications">
      <template slot="title">Terminal</template>
      <template slot="content">
        <div class="notifications__item" v-for="(item, index) in notifications" :key="index">
          <div class="notifications__item__time">{{ item.timestamp | terminalTime }}</div>
          <div v-html="item.html"></div>
        </div>
        <div class="notifications__empty" v-if="!notifications.length">
          From time to time the terminal will announce
          new product releases and other surprises so stay tuned
        </div>
      </template>
    </da-terminal>
    <da-context ref="dndContext" class="dnd-context" @open="onDndMenuOpened"
                @close="setShowDndMenu(false)">
      <template v-if="!dndMode">
        <button class="btn btn-menu"
                @click="enableDndMode('hour')">For 1 hour
        </button>
        <button class="btn btn-menu"
                @click="enableDndMode('tomorrow')">Until tomorrow
        </button>
        <button class="btn btn-menu"
                @click="enableDndMode('forever')">Forever
        </button>
      </template>
      <button v-else class="btn btn-menu" @click="onDisableDndMode">Turn Off</button>
    </da-context>
    <div class="instructions sidebar-instructions invert" v-if="sidebarInstructions">
      <div class="instructions__desc">
        Hover on the sidebar to filter your feed based on tags and sources.
      </div>
      <button class="btn btn-invert" @click="nextInstruction">
        Got it
      </button>
    </div>
  </div>
</template>

<script>
import {
  mapState, mapActions, mapMutations, mapGetters,
} from 'vuex';
import DaHeader from '../components/DaHeader.vue';
import DaSidebar from '../components/DaSidebar.vue';
import DaDndMessage from '../components/DaDndMessage.vue';
import DaSvg from '../components/DaSvg.vue';
import DaFeed from '../components/DaFeed.vue';
import ctas from '../ctas';
import { trackPageView } from '../common/analytics';

export default {
  name: 'Home',

  components: {
    DaSidebar,
    DaDndMessage,
    DaHeader,
    DaSvg,
    DaFeed,
    DaTerminal: () => import('@daily/components/src/components/DaTerminal.vue'),
    DaContext: () => import('@daily/components/src/components/DaContext.vue'),
    DaLogin: () => import('../components/DaLogin.vue'),
    DaProfile: () => import('../components/DaProfile.vue'),
    DaGo: () => import('../components/DaGo.vue'),
    DaWelcome: () => import('../components/DaWelcome.vue'),
    DaCongrats: () => import('../components/DaCongrats'),
    DaRequest: () => import('../components/DaRequest'),
  },

  data() {
    return {
      cta: ctas[Math.floor(Math.random() * ctas.length)],
      showGoModal: false,
      showRequestModal: false,
      showLoginModal: false,
      showProfileModal: false,
      lineNumbers: 1,
    };
  },

  methods: {
    ctaClick() {
      ga('send', 'event', this.cta.name, 'Click');
    },

    updateLines() {
      this.$nextTick(() => {
        setTimeout(() => {
          const pres = this.$refs.lineNumbers.querySelectorAll('pre');
          const lines = Math.ceil(this.$refs.lineNumbers.clientHeight / pres[0].clientHeight);
          if (lines > this.lineNumbers) {
            this.lineNumbers = lines;
          }
        });
      });
    },

    onDndMenu(event) {
      ga('send', 'event', 'Dnd', 'Menu');
      this.setShowDndMenu(true);
      this.$refs.dndContext.open(event);
    },

    onDndMenuOpened(event) {
      const rect = event.target.getBoundingClientRect();
      this.$refs.dndContext.positionMenu({ top: rect.bottom + 8, right: rect.right });
    },

    onDisableDndMode() {
      this.$refs.dndContext.close();
      this.disableDndMode();
    },

    enableDndMode(type = 'hour') {
      let dndDate = new Date();
      if (type === 'hour') {
        dndDate.setHours(dndDate.getHours() + 1);
      } else if (type === 'tomorrow') {
        dndDate = new Date(dndDate.getFullYear(), dndDate.getMonth(), dndDate.getDate() + 1);
      } else if (type === 'forever') {
        dndDate = new Date(dndDate.getFullYear() + 100, dndDate.getMonth(), dndDate.getDate());
      }

      this.$refs.dndContext.close();
      this.setDndModeTime(dndDate.getTime());
      ga('send', 'event', 'Dnd', type);
    },

    onGoClicked() {
      ga('send', 'event', 'Header', 'Go');
      this.showGoModal = true;
    },

    onLogin(section) {
      ga('send', 'event', section, 'Login');
      this.showLoginModal = true;
    },

    onProfile() {
      ga('send', 'event', 'Header', 'Profile');
      this.showProfileModal = true;
    },

    onBackHome() {
      ga('send', 'event', 'Feed', 'Home');
      this.clearFilter();
    },

    onAddFilter() {
      ga('send', 'event', 'Feed', 'Add Filter');
      this.addFilterToFeed();
    },

    async initHome() {
      Promise.all([
        this.fetchPublications(),
        this.fetchTags(),
      ]).then(() => this.fetchNextFeedPage())
        .then(() => this.contentObserver.observe(this.$refs.anchor))
      // TODO: handle error
      // eslint-disable-next-line no-console
        .catch(console.error);

      this.fetchNotifications()
      // TODO: handle error
      // eslint-disable-next-line no-console
        .catch(console.error);
    },

    trackPageView() {
      const { showBookmarks, filter } = this;

      if (showBookmarks) {
        trackPageView('/bookmarks');
      } else if (filter) {
        trackPageView(`/${filter.type}/${filter.info.id || filter.info.name}`);
      } else {
        trackPageView('');
      }
    },

    ...mapActions({
      fetchNextFeedPage: 'feed/fetchNextFeedPage',
      fetchTags: 'feed/fetchTags',
      fetchPublications: 'feed/fetchPublications',
      clearFilter: 'feed/clearFilter',
      addFilterToFeed: 'feed/addFilterToFeed',
      fetchNotifications: 'ui/fetchNotifications',
      refreshToken: 'user/refreshToken',
    }),

    ...mapMutations({
      setDndModeTime: 'ui/setDndModeTime',
      disableDndMode: 'ui/disableDndMode',
      hideNotifications: 'ui/hideNotifications',
      nextInstruction: 'ui/nextInstruction',
      setShowDndMenu: 'ui/setShowDndMenu',
      confirmNewUser: 'user/confirmNewUser',
    }),
  },

  computed: {
    ...mapState('ui', ['notifications', 'showNotifications', 'theme']),
    ...mapGetters('ui', ['sidebarInstructions', 'showReadyModal', 'dndMode']),
    ...mapState('feed', ['showBookmarks', 'filter']),
    ...mapState({
      title(state) {
        let res = '';
        if (state.feed.showBookmarks) {
          res += 'your personal bookmarks';
        } else {
          res += 'news for you';
        }

        if (state.ui.insaneMode) {
          res += ' - insane mode';
        }

        return res;
      },

      clsObj(state) {
        return {
          'animate-cards': state.ui.enableCardAnimations,
        };
      },

      showCongratsModal(state) {
        if (this.isLoggedIn) {
          return state.user.profile.newUser;
        }

        return false;
      },
    }),

    ...mapGetters({
      posts: 'feed/feed',
      hasFilter: 'feed/hasFilter',
      isLoggedIn: 'user/isLoggedIn',
    }),

    emptyBookmarks() {
      return !this.posts.length && this.showBookmarks;
    },
  },

  watch: {
    posts() {
      this.updateLines();
    },
    showBookmarks() {
      this.trackPageView();
    },
    filter() {
      this.trackPageView();
    },
  },

  created() {
    this.trackPageView();

    this.contentObserver = new IntersectionObserver(async (entries) => {
      if (entries[0].isIntersecting) {
        if (await this.fetchNextFeedPage() && this.page > 0) {
          ga('send', 'event', 'Feed', 'Scroll', 'Next Page', this.page);
        }
      }
    }, { root: null, rootMargin: '0px', threshold: 1 });
  },

  async mounted() {
    import('@daily/components/icons/arrow');
    import('@daily/components/icons/plus');
    import('@daily/components/icons/hamburger');

    if (this.cta.icon) {
        import(`@daily/components/icons/${this.cta.icon}`);
    }

    this.updateLines();
    await this.refreshToken();

    requestIdleCallback(async () => {
      await this.initHome();
    });
  },
};
</script>

<style>
.content {
  display: flex;
  flex-direction: column;
  margin: 24px 40px 76px 76px;
}

.content__header {
  display: flex;
  flex-direction: row;
  align-items: center;
  margin-bottom: 24px;

  & h4 {
    color: var(--theme-secondary);

    &.uppercase {
      text-transform: uppercase;
    }
  }

  & .btn .svg-icon {
    width: 20px;
    height: 20px;
    margin-right: 4px;
  }

  & .content__header__back-home {
    margin-right: 16px;

    & .svg-icon {
      transform: rotate(-90deg);
    }
  }

  & .content__header__pub-image {
    width: 24px;
    height: 24px;
    margin: 0 8px 0 0;
    border-radius: 4px;
  }

  & .content__header__add-filter {
    margin-left: auto;
  }
}

.header__cta {
  display: flex;
  height: 32px;
  flex-direction: row;
  align-items: center;
  margin-left: auto;
  border-radius: 8px;
}

.header__cta__text {
  margin: 0 8px 0 16px;

  & {
    @mixin micro2;
  }
}

.header__cta__image {
  width: 20px;
  height: 20px;
  margin: 0 8px;
  color: var(--color-salt-10);
}

.content__empty-bookmarks {
  display: flex;
  margin-top: 120px;
  flex-direction: column;
  align-items: center;

  & img {
    height: 185px;
  }
}

.content__empty-bookmarks__title {
  margin: 32px 0 8px;
  color: var(--theme-primary);
  text-transform: uppercase;
}

.content__empty-bookmarks__text {
  margin: 8px 0;
  color: var(--theme-secondary);

  & {
    @mixin jr;
  }
}

#anchor {
  position: absolute;
  bottom: 100vh;
  left: 0;
  height: 1px;
  width: 1px;
  opacity: 0;
}

.notifications {
  position: fixed;
  width: 300px;
  height: 234px;
  right: 35px;
  top: 44px;
  z-index: 100;

  & .notifications__item {
    margin: 16px 0;

    &:first-child {
      margin-top: 0;
    }

    &:last-child {
      margin-bottom: 0;
    }

    & > * {
      margin: 4px 0;

      &:first-child {
        margin-top: 0;
      }

      &:last-child {
        margin-bottom: 0;
      }
    }
  }

  & .notifications__item__time {
    color: var(--theme-disabled);
  }

  & a {
    text-decoration: none;
    color: var(--theme-primary);

    &:visited, &:active {
      color: inherit;
    }
  }
}

.sidebar-instructions {
  top: 72px;
  left: 45px;
  width: 188px;

  & .btn {
    margin-top: 8px;
    align-self: stretch;
    justify-content: flex-end;
  }
}

.v-context.context {
  &:focus {
    outline: none;
  }
}

.bookmarks-placeholder {
  height: 185px;
}

.line-numbers {
  position: absolute;
  display: flex;
  left: 0;
  top: 0;
  width: 36px;
  height: 100%;
  flex-direction: column;
  padding: 68px 0 0 0;
  background: var(--theme-background-highlight);
  border-right: 1px solid var(--theme-separator);

  & > * {
    margin: 8px 0;
  }
}

.line-numbers_icon {
  align-self: center;
}

.line-numbers__lines {
  flex: 1;
  overflow: hidden;
  margin-right: 8px;

  & pre {
    color: var(--theme-disabled);
    text-align: right;
    margin: 0;
  }
}

.line-numbers__collapse {
  position: absolute;
  width: 10px;
  height: 12px;
  right: -5px;
  top: 118px;
  margin: 0;

  & .fill {
    fill: var(--theme-background-primary);
  }

  & .stroke {
    stroke: var(--theme-separator);
  }
}
</style>
