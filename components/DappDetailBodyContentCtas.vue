<template>
<div class="component-DappDetailBodyContentCtas">
  <div class="wrapper">
    <div class="cta-wrapper">
      <transition name="fade">
        <div :class="added ? 'active' : ''" class="add-popover" v-if="added"><nuxt-link class="add-popover-button" :to="{ name: 'my-list' }">Go to my list</nuxt-link></div>
      </transition>
      <button v-if="!myList.includes(dapp.slug)" class="button -add" @click="addToMyList(dapp.slug)">
        <span class="add-text"><SvgPlus/> Add to my list</span>
      </button>
      <button v-else class="button -add" @click="removeFromMyList(dapp.slug)">
        <span class="add-text">Remove from my list</span>
      </button>
    </div>
    <div v-if="dapp.logoUrl" class="logo-wrapper">
      <img class="logo-image" :src="dapp.logoUrl"/>
    </div>
    <div v-if="dapp.sites.websiteUrl === dapp.sites.dappUrl">
      <a :href="dapp.sites.websiteUrl" class="button" target="_blank" :rel="'noopener noreferrer' + (dapp.nofollow ? ' nofollow' : '')" @click="trackDappSite(['website','dapp'], dapp.sites.websiteUrl)">
        <span v-if="dapp.tags.includes(dappGameTag)">Play game<span v-if="dapp.isNsfw"> (NSFW)</span></span>
        <span v-else>Launch ÐApp/website<span v-if="dapp.isNsfw"> (NSFW)</span></span>
      </a>
    </div>
    <div v-else>
      <a v-if="dapp.sites.dappUrl" :href="dapp.sites.dappUrl" class="button" target="_blank" :rel="'noopener noreferrer' + (dapp.nofollow ? ' nofollow' : '')" @click="trackDappSite(['dapp'], dapp.sites.dappUrl)">
        <span v-if="dapp.tags.includes(dappGameTag)">Play game<span v-if="dapp.isNsfw"> (NSFW)</span></span>
        <span v-else>Launch ÐApp<span v-if="dapp.isNsfw"> (NSFW)</span></span>
      </a>
      <a v-if="dapp.sites.websiteUrl" :href="dapp.sites.websiteUrl" class="button" target="_blank" :rel="'noopener noreferrer' + (dapp.nofollow ? ' nofollow' : '')" @click="trackDappSite(['website'], dapp.sites.websiteUrl)">Visit website<span v-if="dapp.isNsfw"> (NSFW)</span></a>
    </div>
    <ul v-if="dapp.socials.length" class="social-list">
      <li v-for="(social, index) in dapp.socials" :key="index" class="social-item">
        <a class="social-link" :href="social.url" target="_blank" rel="noopener noreferrer" :title="social.platform | capitalize" @click="trackDappSocial(social.platform, social.url)">
          <component :is="svgSocialComponent(social.platform)"></component>
        </a>
      </li>
    </ul>
  </div>
</div>
</template>

<script>
import { dappGameTag, dappSocialComponentMap, myListLimit } from '~/helpers/constants'
import { trackDappSite, trackDappSocial, trackListAdd, trackListRemove } from '~/helpers/mixpanel'
import SvgSocialChat from './SvgSocialChat'
import SvgSocialBlog from './SvgSocialBlog'
import SvgSocialFacebook from './SvgSocialFacebook'
import SvgSocialGithub from './SvgSocialGithub'
import SvgSocialGitter from './SvgSocialGitter'
import SvgPlus from './SvgPlus'
import SvgSocialReddit from './SvgSocialReddit'
import SvgSocialTwitter from './SvgSocialTwitter'
import SvgStar from './SvgStar'

export default {
  data () {
    return {
      added: false,
      dappGameTag,
      myList: []
    }
  },
  components: {
    SvgSocialChat,
    SvgSocialBlog,
    SvgSocialFacebook,
    SvgSocialGithub,
    SvgSocialGitter,
    SvgPlus,
    SvgSocialReddit,
    SvgSocialTwitter,
    SvgStar
  },
  methods: {
    addToMyList (slug) {
      if (this.myList.length < myListLimit) {
        if (!this.myList.includes(slug)) {
          this.myList.push(slug)
          this.$localStorage.set('myList', this.myList)
          this.$store.dispatch('list/setItems', this.myList)
          this.added = true
          const action = trackListAdd(this.dapp.slug)
          this.$mixpanel.track(action.name, action.data)
          setTimeout(() => {
            this.added = false
          }, 8500)
        }
      } else {
        alert('You have reached the limit of 50 dapps on your list. Please remove some before you add more.')
      }
    },
    removeFromMyList (slug) {
      if (this.myList.includes(slug)) {
        let index = this.myList.indexOf(slug)
        this.myList.splice(index, 1)
        this.$localStorage.set('myList', this.myList)
        this.$store.dispatch('list/setItems', this.myList)
        const action = trackListRemove(this.dapp.slug)
        this.$mixpanel.track(action.name, action.data)
      }
    },
    svgSocialComponent (platform) {
      const socialComponent = dappSocialComponentMap[platform]
      return socialComponent
    },
    trackDappSite (type, url) {
      const action = trackDappSite(this.dapp.slug, type, url)
      this.$mixpanel.track(action.name, action.data)
    },
    trackDappSocial (platform, url) {
      const action = trackDappSocial(this.dapp.slug, platform, url)
      this.$mixpanel.track(action.name, action.data)
    }
  },
  mounted () {
    const myList = this.$localStorage.get('myList')
    if (myList) {
      this.myList = myList.split(',')
    }
  },
  props: {
    dapp: {
      required: true
    }
  }
}
</script>


<style lang="scss" scoped>
@import '~assets/css/settings';

.add-icon {
  font-size: 2.1rem;
  line-height: 0;
}

.add-popover {
  position: absolute;
  top: -45px;
  left: calc(50% - 60px);
  width: 100%;
  background: $color--black;
  border-radius: 4px;
  box-shadow: 0 4px 50px rgba($color--black, .2);
  padding: 8px;
  text-align: center;
  width: 120px;
  &:after {
    content: '';
    position: absolute;
    right: calc(50% - 5px);
    bottom: -5px;
    width: 0;
    height: 0;
    border-left: 5px solid transparent;
    border-right: 5px solid transparent;
    border-top: 5px solid $color--black;
  }
}

.add-popover-button {
  color: $color--white;
}

.button {
  text-align: center;
  text-decoration: none;
  border: 1px solid $color--black;
  display: block;
  width: 100%;
  max-width: 200px;
  padding: 8px 5px;
  margin-bottom: 8px;
  margin-left: auto;
  margin-right: auto;
  &.-add {
    display: flex;
    align-items: center;
    justify-content: center;
    background: $color--black;
    color: $color--white;
    line-height: initial;
  }
}

.cta-wrapper {
  position: relative;
}

.logo-image {
  display: block;
  margin: 0 auto;
  max-width: 200px;
  @include tweakpoint('min-width', 1000px) {
    max-width: 100%;
  }
}

.logo-wrapper {
  text-align: center;
  overflow: hidden;
  padding-bottom: 8px;
}

.social-item {
  padding: 3px;
}

.social-link {
  display: block;
  height: 24px;
}

.social-list {
  margin: 0 -3px;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  @include tweakpoint('min-width', 1000px) {
    justify-content: flex-start;
  }
}

.add-icon {
  margin-right: 5px;
}

.add-text {
  display: inline-block;
}

.wrapper {
  margin: 0 10px 24px 10px;
  padding-bottom: 24px;
  border-bottom: 1px solid darken($color--gray, 6%);
}
</style>
