<template>
  <div v-for="video in videos" v-cloak>
    <div class="embed-responsive embed-responsive-4by3">
      <iframe class="embed-responsive-item" v-bind:src="'https://www.youtube.com/embed/' + video.contentDetails.videoId" frameborder="0" allowfullscreen></iframe>
      <h1 v-if="video.isLatest"><span class="label label-success" style="position: absolute">New</span></h1>
    </div>
  </div>
  <div class="text-center" v-show="loadingVideos">
    Loading...
  </div>
</template>

<script>
export default {
  data() {
    return {
      videos: [],
      nextPage: null,
      loadingVideos: false,
      playlistId: 'PLPbW7txEM2AXvs7GkSuCtk9RyCWuSmXQT',
      apiKey: '<YOUR_API_KEY_HERE>'
    }
  },
  methods: {
    getPlayListItems (pageToken) {
      this.loadingVideos = true

      let url = `https://www.googleapis.com/youtube/v3/playlistItems?`
        + `part=contentDetails`
        + `&maxResults=5`
        + `&playlistId=${this.playlistId}`
        + `&key=${this.apiKey}`

      if (pageToken != null) {
        url = url + `&pageToken=${pageToken}`
      }

      this.$http
        .get(url, (data) => {
          this.videos = this.videos.concat(data.items)
          this.nextPage = data.nextPageToken

          this.loadingVideos = false

          this.updateLatestItem(this.videos[0])
        })
        .error((err) => console.log(err))
    },
    handleScroll () {
      if (document.body.scrollHeight - window.innerHeight - document.body.scrollTop <= 5) {
        if (this.nextPage != null) {
          this.getPlayListItems(this.nextPage)
        }
      }
    },
    updateLatestItem(item) {
      if (!item) {
        return
      }

      const lastSavedVideoId = localStorage.getItem('latestVideoId')
      const latestVideoId =  item && item.contentDetails.videoId

      if (latestVideoId !== lastSavedVideoId) {
        localStorage.setItem('latestVideoId', latestVideoId)

        item.isLatest = true
      }
    }
  },
  created: function () {
    this.getPlayListItems()

    window.addEventListener('scroll', this.handleScroll)
  }
}
</script>
