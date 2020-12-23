<template>
  <div class="user-card-container" style="border-radius: 4px" v-if="showCard">
    <div
      :style="{
        'background-image': `linear-gradient(70deg, rgba(255, 255, 255, .9) 0%, rgba(255, 255, 255, .9) 100%, transparent 100%), 
            url(${userPhoto})`
      }"
      class="user-card"
    >
      <div class="user-card__content">
        <div class="user-card__image-container" v-if="userPhoto">
          <img :src="userPhoto" alt="user photo" class="user-card__image"/>
        </div>
        <div class="user-card__text">
          <h3>{{ userName }}</h3>
          <div style="padding-bottom: 1.25em">
            <i class="fas fa-map-marker-alt icons" ></i>
            <span>{{ userCity }}</span>
          </div>
          <div>
            <i class="fas fa-envelope icons"></i>
            <a
              :href="`mailto:${userEmail}`"
              style="color: currentColor; text-decoration: none;"
            >
              {{ userEmail }}
            </a>
          </div>
          <div class="user-card__bio" v-html="userBio"> </div>
        </div>

        <div class="user-card__btn-icon">
          <button-icon
            :bgColor="'green'"
            :size="'small'"
            v-if="user.uid != myProfile.uid"
            @click.native="directMessage(user.uid)"
          >
            <i class="fas fa-comments-alt" style="color: #FFF"></i>
          </button-icon>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ButtonIcon from '~/components/atoms/ButtonIcon'
import { mapState } from 'vuex'

export default {
  components: {
    ButtonIcon
  },

  props: ['user', 'index', 'indexClicked', 'showAllCards'],

  data() {
    return {
      profile: '',
      userPhoto: '',
      userCity: '',
      userName: '',
      userEmail: this.user.email,
      userBio: '',
      userCoordinates: ''
    }
  },

  computed: {
    ...mapState('user', ['myProfile']),
    showCard() {
      return this.index === this.indexClicked || this.showAllCards
    }
  },

  methods: {
    directMessage(userId) {
      const dm = this.$store.getters['user/directMessageChatId']
      dm(userId).then((chatId) => {
        this.$router.push('/chats/' + chatId)
      })
    }
  },

  fetchOnServer: false,
  async fetch() {
    try {
      const module = this.$store.getters['user/moduleForUser'](this.user.uid)
      this.profile = await module.bind();

      this.userPhoto = this.profile.photoURL;

      this.userCity = this.profile.currentLocation.d.city;

      this.userName = this.profile.name;

      this.userBio = this.profile.bio;

      this.userCoordinates = [
        this.profile.currentLocation.d.latitude,
        this.profile.currentLocation.d.longitude
      ]
    } catch (err) {
      console.log(err)
    }
  }
}
</script>

<style lang="scss" scoped>
  .user-card-container {
    max-width: 350px;
    width: 90%;
    margin: 20px auto;
    overflow: hidden;
    box-shadow: 0 10px 15px rgba(0, 0, 0, 0.15);
    transition: all 0.4s;

    &:hover {
      transform: translateY(-10px);
    }
  }

  .user-card {
    height: 100%;
    background-size: cover;
    background-position: center;

    &__content {
      width: 100%;
      text-align: center !important;
      padding: 1.25em;
    }

    &__image-container {
      display: flex;
      justify-content: center;
    }

    &__image {
      width: 6.25rem;
      height: 6.25rem;
      border-radius: 50%;
    }

    &__bio {
      margin: .625em 0 1.25em;
      line-height: 1.875em;
    }

    &__text {

      & > *:not(:last-child) {
        padding-bottom: 0.5rem;
      }
    }

    &__btn-icon {
      display: flex;
      justify-content: center;
    }
  }

  .icons {
    color: #aaa;
  }
</style>
