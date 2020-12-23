<template>
    <div>
       
        <l-marker 
            :lat-lng="userCoordinates" 
            v-if="profile" 
            @mouseover="openUserCard"
        >
            <l-tooltip>
               <div>
                   <h3>{{userName}}</h3>
                   <h5>{{userCity}}</h5>
                   <img :src="userPhoto" alt="user photo" style="width: 100%">
               </div>
            </l-tooltip>
        </l-marker>
    
    </div>
</template>

<script>
export default {
    props: ["user", "index"],

    data() {
        return {
            profile: "",
            userPhoto: "",
            userCoordinates: "",
            userName: "",
            userCity: "",
        }
    },

    methods: {
        openUserCard() {
            this.$emit("open-user-card", this.index);
        },
        
    },

    fetchOnServer: false,
    async fetch() {
        try {
            const module = this.$store.getters['user/moduleForUser'](this.user.uid);
            this.profile = await module.bind();

            this.userPhoto = this.profile.photoURL;

            this.userCity = this.profile.currentLocation.d.city;

            this.userName = this.profile.name;


            this.userCoordinates = [
                this.profile.currentLocation.d.latitude,
                this.profile.currentLocation.d.longitude
            ]
        }
        catch(err) {
            console.log(err)
        }
    },
}
</script>
