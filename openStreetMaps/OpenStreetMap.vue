<template>

  <div>

    <div class="input-area__field">
      <input
        ref="input"
        type="text"
        style="width: 100%; height: 34px;"
        v-model="userToFind"
        placeholder="Search company users"
        @keydown.enter="getUserCoordinates(userToFind)"
      />
    </div>

    <!-- user search autocomplete -->
    <div class="user-search-autocomplete" ref="autocompleteCard">
      <div 
        v-for="(user, index) in filteredList"
        :key="index"
        @click="getUserCoordinates(user.name)"
      >
        {{user.name}}
      </div>
    </div>

    <client-only>
      <l-map
        v-if="placesData.length"
        style="height: 350px; width: 90%; margin: 0 auto"
        :zoom="zoom"
        :minZoom="minZoom"
        :center="placesData[0].markerLatLng"
      >
        <!--change tile layers -->
        <l-tile-layer
          v-for="tileProvider in tileProviders"
          :key="tileProvider.name"
          :name="tileProvider.name"
          :visible="tileProvider.visible"
          :url="tileProvider.url"
          layer-type="base"
        >
        </l-tile-layer> 

        <!-- CREATE A MARKER FOR EACH COMPANY USER -->
        <v-marker-cluster>
          <user-markers
            v-for="(user, index) in users"
            :key="user.uid"
            :user="user"
            :index="index"
            @open-user-card="openUserCard"
          >
          </user-markers>
        </v-marker-cluster>

        <!-- USE THE ROOTLO SLOGAN AS A CUSTOM ATTRIBUTION ON THE MAP -->
        <l-control-attribution
          position="topright"
          prefix="Rootlo... Work globally, live locally."
        >
        </l-control-attribution>

        <l-control-layers position="topright"></l-control-layers>

        <l-control-scale
          position="bottomright"
          :imperial="true"
          :metric="false"
        >
        </l-control-scale>

        <l-control position="bottomleft">
          <button-primary :size="'small'" :bgColor="'light'" @click.native="showCompanyUsers">
            <span v-if="!showAllCards">company users</span>
            <span v-if="showAllCards">Hide users</span>
          </button-primary>
        </l-control>

      </l-map>
    </client-only>

    <!-- CREATE A CARD FOR EACH COMPANY USER -->
    <div class="user-cards">
      
      <user-card
        v-for="(user, index) in users"
        :key="user.uid"
        :user="user"
        :index="index"
        :indexClicked="userCardToOpen"
        :showAllCards="showAllCards"
        class="grid-item"
      >
      </user-card>
      
    </div>
  </div>
</template>

<script>
import ButtonPrimary from '~/components/atoms/ButtonPrimary';
import UserMarkers from "./UserMarkers";
import UserCard from "./UserCard";

export default {
  components: {
    ButtonPrimary,
    UserMarkers,
    UserCard,
  },

  props: ['users', 'userCoordinates'],

  data() {
    return {
      companyUsersDetails: [],  // THE DATA THAT WILL BE QUERIED FOR THE COMPANY USERS SEARCH FUNCTIONALITY
      userToFind: "",
      placesData: [],
      search: false,
      location: '',
      showAllCards: false,
      userCardToOpen: "",
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      zoom: 3,
      minZoom: -5,
      tileProviders: [
        {
          name: 'OpenStreetMap',
          visible: true,
          attribution: '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
          url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'
        },
        {
          name: 'OpenTopoMap',
          visible: false,
          url: 'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png',
          attribution: 'Map data: &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)'
        }
      ]
    }
  },

  computed: {
    places() {
      const userCoordinates = [this.userCoordinates.lat, this.userCoordinates.lng];
      return [{ markerLatLng: [...userCoordinates] }];
    },
    filteredList() {
      return this.companyUsersDetails.filter(user => {
        return user.name.toLowerCase().includes(this.userToFind.toLowerCase())
      })
    }
  },

  watch: {
    places: {
      immediate: true,
      handler(newValue) {
        this.placesData = newValue;
      }
    },
    users: {
      immediate: true,
      handler(newValue) {
        if(newValue) {
          newValue.map(async user => {
            let module = this.$store.getters['user/moduleForUser'](user.uid);
            let profile = await module.bind();
            this.companyUsersDetails.push({
              name: profile.name,
              markerLatLng: [profile.currentLocation.d.latitude, profile.currentLocation.d.longitude]
            })
          })
        }
      }
    },
    userToFind(newValue) {
      const autocompleteCard = this.$refs.autocompleteCard;
      if(newValue) {
        autocompleteCard.style.display = "block";
      }
      else {
        autocompleteCard.style.display = "none";
      }
    }
  },

  methods: {
    openUserCard(value) {
      this.userCardToOpen = value;
    },
    showCompanyUsers() {
      if(this.showAllCards === false) {
        this.showAllCards = true;
      } else {
        this.showAllCards = false;
      }
    },
    getUserCoordinates(user) {
      this.userToFind = user;

      this.companyUsersDetails.find(val => {
        if(this.userToFind.toLowerCase() !== val.name.toLowerCase()) return;
        
        // UPDATE THE FOCUS OF THE MAP
        this.placesData[0].markerLatLng = val.markerLatLng;
      })
    }

  },

  mounted() {
    const input = this.$refs.input;
    const autocompleteCard = this.$refs.autocompleteCard;

    // CLOSE THE AUTOCOMPLETE CARD WHEN THE USER CLICKS OUTSIDE OF THE INPUT FIELD
    input.addEventListener("focusout", () => {
      setTimeout(() => {
        autocompleteCard.style.display = "none";
      }, 500)
    })
  }
}
</script>

<style lang="scss" scoped>
  //@import "~leaflet.markercluster/dist/MarkerCluster.css";
 // @import "~leaflet.markercluster/dist/MarkerCluster.Default.css";
  
  .input-area__field {
    width: 90%;
    margin: 0 auto;

    input {
      width: 100%;
    }
  }

.user-cards {
  margin: 20px 0;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
 
  & > * {
    margin: 10px;
  }
}

.user-search-autocomplete {
  width: 90%;
  overflow-y: scroll;
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  z-index: 6000;
  background: #FFF;
  margin: 0 auto;
  box-shadow: 0 7px 12px rgba(0, 0, 0, .15);
  border-radius: 4px;
  display: none;

  & > * {
    padding: 1em;
    cursor: pointer;

    &:hover {
      background: #eee;
    }
  }
    
}

</style>
