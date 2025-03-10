<template>
  <da-card class="card--post" :class="cls" :title="post.title" :url="post.url" :image="post.image"
          :placeholder="post.placeholder" :size="post.size" @click="$emit('click', post)">
    <div slot="content" class="card__tags nuggets" v-tooltip="tagsStr">
      <da-line-clamp :text="tagsStr" :lines="1" :truncate="truncateTags"/>
    </div>
    <template slot="footer">
      <button class="btn-icon btn-small card__footer__publication" v-if="post.publication.name"
              @click="$emit('publication', { pub: post.publication })">
        <img class="card__footer__icon lazyload"
            :data-src="post.publication.image"
            :alt="post.publication.name" v-tooltip="post.publication.name"
            :key="post.publication.name"/>
      </button>
      <span class="card__footer__views micro2"
            v-if="post.readTime">// {{post.readTime}} min read</span>
      <button class="btn-icon btn-small card__footer__bookmark"
              v-tooltip="post.bookmarked ? 'Remove bookmark' : 'Bookmark'"
              @click="$emit('bookmark', { post, bookmarked: !post.bookmarked })">
        <svgicon icon="bookmark"/>
      </button>
      <button class="btn-icon btn-small card__footer__menu" v-tooltip="'More'"
              @click="$emit('menu', { post, event: $event })" v-if="showMenu">
        <svgicon icon="menu" ref="orig"/>
      </button>
      <transition name="post-notification">
        <div class="card__footer__notification nuggets" v-if="notifying">
          {{ notification }}
        </div>
      </transition>
    </template>
    <svgicon icon="menu" class="card__menu--duplicate" ref="dup"
             slot="other" v-if="menuOpened"/>
  </da-card>
</template>

<script>
import 'lazysizes';
import postMixin from '../common/postMixin';
import DaCard from './DaCard.vue';
import DaLineClamp from './DaLineClamp.vue';

export default {
  name: 'DaCardPost',
  mixins: [postMixin],
  components: { DaCard, DaLineClamp },

  watch: {
    menuOpened() {
      this.positionDuplicate();
    },
  },

  computed: {
    cls() {
      return {
        bookmarked: this.post.bookmarked,
        'menu-opened': this.menuOpened,
        hover: this.menuOpened,
      };
    },
    tagsStr() {
      return (this.post.tags || []).map(t => `#${t}`).join(',');
    },
  },

  mounted() {
    import('../../icons/bookmark');
    import('../../icons/menu');

    this.positionDuplicate();
  },

  methods: {
    positionDuplicate() {
      if (this.menuOpened) {
        this.$nextTick(() => {
          const parentRect = this.$el.getBoundingClientRect();
          const childRect = this.$refs.orig.$el.getBoundingClientRect();

          this.$refs.dup.$el.style.top = `${childRect.top - parentRect.top}px`;
          this.$refs.dup.$el.style.left = `${childRect.left - parentRect.left}px`;
          this.$refs.dup.$el.style.width = `${childRect.width}px`;
          this.$refs.dup.$el.style.height = `${childRect.height}px`;
        });
      }
    },
  },
};
</script>
<style>
.card--post {
  position: relative;

  & .card__footer {
    position: relative;
    overflow: hidden;
  }
}

.card--post .card__content {
  border-bottom: 1px solid var(--theme-background-primary);

  @mixin shadow1;
}

.card--post .card__menu--duplicate {
  position: absolute;
  color: var(--theme-primary);
}

.menu-opened.card--post {
  pointer-events: none;

  & .card__link, & .card__footer {
    opacity: 0.4;
  }
}

.card__tags {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  margin: 16px 24px;
  color: var(--theme-disabled);
  text-align: center;
  word-break: break-all;
}

.card__footer__icon {
  width: 20px;
  height: 20px;
  border-radius: 4px;
}

.card__footer__views {
  color: var(--theme-secondary);
}

.card__footer > .card__footer__bookmark {
  margin-left: auto;
}

.card__footer__notification {
  position: absolute;
  display: flex;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  align-items: center;
  justify-content: center;
  margin: 0;
  color: var(--color-salt-10);
  background: var(--color-water-60);
  z-index: 10;
}

.bookmarked .card__footer__bookmark .svg-icon {
  color: var(--color-burger-60);
}

.post-notification-enter-active, .post-notification-leave-active {
  transition: opacity 0.2s, transform 0.2s;
}

.post-notification-enter, .post-notification-leave-to /* .fade-leave-active below version 2.1.8 */
{
  opacity: 0;
  transform: translateY(100%);
}
</style>
