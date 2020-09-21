<template>
  <v-app id="inspire">
   <v-navigation-drawer
     v-model="drawer"
     app
   >
     <v-list dense>
       <v-list-item link>
         <v-list-item-action>
           <v-icon>mdi-home</v-icon>
         </v-list-item-action>
         <v-list-item-content>
           <v-list-item-title>Home</v-list-item-title>
         </v-list-item-content>
       </v-list-item>
       <v-list-item link
        @click.stop="dialog = true"
       >
         <v-list-item-action>
           <v-icon>mdi-plus</v-icon>
         </v-list-item-action>
         <v-list-item-content>
           <v-list-item-title>Add rss</v-list-item-title>
         </v-list-item-content>
       </v-list-item>
     </v-list>
   </v-navigation-drawer>
   <v-dialog
      v-model="dialog"
      max-width="290"
    >
      <v-card>
        <v-card-title class="headline">Add New RSS</v-card-title>
        <v-form>
          <v-container>
            <v-row class="px-4">
              <v-col cols="12">
                <v-text-field
                  v-model="new_feed.url"
                  label="URL"
                  clearable
                ></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="new_feed.title"
                  label="Title"
                  clearable
                ></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="new_feed.icon"
                  label="Icon URL"
                  clearable
                ></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-form>

        <v-card-actions>
          <v-spacer></v-spacer>

          <v-btn
            color="green darken-1"
            text
            @click="dialog = false"
          >
            cancel
          </v-btn>

          <v-btn
            color="green darken-1"
            text
            @click="addNewFeed"
          >
            submit
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
   <v-app-bar
     app
     color="indigo"
     dark
   >
     <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
     <v-toolbar-title>RSS READER</v-toolbar-title>
   </v-app-bar>

   <v-main class="main-container">
     <v-container
       class="fill-height top-container"
       fluid
     >
       <v-row
         align="center"
         justify="center"
         class="upper-row"
       >
         <v-tabs
           v-model="tab"
           centered
           grow
           show-arrows
           icons-and-text
           @change="getTabStatus"
         >
           <v-tabs-slider color="teal lighten-3"></v-tabs-slider>

           <v-tab
             v-for="(obj, i) in feed_list"
             :key="obj.id"
             :href="'#tab-' + i"
             class="feed-tab"
             @click="getTab(i)"
           >
             {{obj.title}}
             <v-btn
                text
                class="align-self-center"
              >
             <v-list-item-avatar :style="`border: 1px solid #000000; margin-right:0px`">
                  <v-img :src="obj.icon"></v-img>
              </v-list-item-avatar>
              </v-btn>
           </v-tab>
         </v-tabs>
         <v-progress-linear
           indeterminate
           v-if="loading"
           color="primary"
           class='mt-2'
         ></v-progress-linear>
       </v-row>
       <v-row
         align="center"
         justify="center"
       >
        <v-col>
          <v-card v-for="(item, index) in detail_list"
            class="ma-4 pa-4"
            @click="openLink(item)"
            >
            <h2>{{item.title}}</h2>
            <div v-html="item.description"></div>
          </v-card>
         </v-col>
       </v-row>
     </v-container>
   </v-main>
   <v-footer
     color="indigo"
     app
   >
     <span class="white--text">&copy; {{ new Date().getFullYear() }}</span>
   </v-footer>
 </v-app>
</template>

<script>
// import HelloWorld from './components/HelloWorld';
import rss_list from '@/data/rsslist.js'
var convert = require('xml-js')

export default {
    props: {
      source: String,
    },
    data: () => ({
      drawer: null,
      tab: null,
      feed_list: [],
      text: 'Lorem ipsum',
      detail_list: [],
      loading: false,
      new_feed: {
        url: '',
        title: '',
        icon: ''
      },
      dialog: false,
    }),
    created() {
      let option = {
        method: 'GET',
        url: 'http://localhost:8010/feed/',
        headers: {'content-type': 'Application/json'},
      }
      this.$http(option)
      .then((response) => {
        var data = response.data
        this.feed_list = data.feed_list
        console.log(data)
        console.log('tag:', this.tab)
      })
    },
    computed: {},
    methods: {
      getTabStatus(tabparam) {
        console.log('tabparam', tabparam)
        console.log('current tab:', this.tab)
        console.log('type of ', typeof this.tab)
      },
      getTab(tabparam) {
        console.log('gettabparam', tabparam)
        console.log('ddcurrent tab:', this.tab)
        this.loadFeedDetail(this.feed_list[tabparam].id)
      },
      addNewFeed() {
        let option = {
          method: 'POST',
          url: 'http://localhost:8010/feed/',
          data: this.new_feed,
          headers: {'content-type': 'Application/json'},
        }
        this.loading = true
        this.$http(option)
        .then((response) => {
          console.log('response:', response)
          var data = response.data
          this.feed_list = data.feed_list
          console.log(this.feed_list)
          this.loading = false
          this.dialog = false
        })
        .catch((error) => {
          if (error) {
            console.log('error happens:', error)
            let err = error.response.data
            if (err.code !== 200) {
              alert('Failed to add the feed')
            }
          }
          this.loading = false
          this.dialog = false
        })
      },
      loadFeedDetail(i) {
        let option = {
          method: 'GET',
          url: 'http://localhost:8010/feed/' + i,
          headers: {'content-type': 'Application/json'},
        }
        this.loading = true
        this.$http(option)
        .then((response) => {
          var data = response.data
          this.detail_list = data.channel.item
          console.log(this.detail_list)
          this.loading = false
        })
        .catch((error) => {
          this.loading = false
        })
      },
      openLink(item) {
        window.open(item.link, "_blank");
      }
    }
}
</script>
