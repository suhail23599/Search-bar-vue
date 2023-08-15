<template>
  <div class="container">
    <div class="title">Search Bar</div>
    <input type="text" @input="callFn" @keydown="handleKeyEvent" placeholder="search for your keyword" v-model="keyword"> 
    <div class="list-wrapper" v-if="showModal">
      <div
        class="item" 
        :class="{'active': activeIndex === index}"
        v-for="(item, index) in searchResult" 
        :key="item.id"
      >
        {{ item.name }}
      </div>
    </div>
    <div v-if="noResult">
      Sorry, No Result Found
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
}
.title {
  font-size: 18px;
}
input {
  height: 30px;
  font-size: 18px;
  width: 98%;
}
.list-wrapper {
  border-radius: 4px;
  background: antiquewhite;
}
.item {
  margin-top: 2px;
  padding: 6px;
  font-size: 16px;
  border-bottom: 1px solid black;
}
.active {
  background: yellow
}
</style>

<script>
export default {
  name: 'SearchBar',
  data () {
    return {
      keyword: '',
      dealy: 30,
      searchResult: [],
      URL:'http://swapi.dev/api/people/?search=',
      activeIndex: null,
      showModal: false,
      noResult: false
    }
  },
  created () {
    this.debounceSearch = this.debounce(this.callAPI, 300)
  },
  methods: {
    async callAPI (){
      try {
        const response = await fetch (`${this.URL}${this.keyword}`)
        const data = await response.json()
        if (this.keyword) {
          this.searchResult = data.results
          if (this.searchResult.length === 0) {
            this.noResult = true
          } else {
            this.noResult = false
          }
        } else {
          this.searchResult = []
        }
      } catch (err) {
        console.log(err, 'error while fetching result')
      }
    },
    callFn (e) {
      this.showModal = true
      if (this.keyword) {
        this.debounceSearch(e.target.value)
      } else {
        this.searchResult = []
      }
    },
    handleKeyEvent (event) {
      const { keyCode } = event
      if (keyCode === 40) {
        // down
        if (this.activeIndex === null || this.activeIndex === this.searchResult.length - 1){
          this.activeIndex = 0
        } else {
          this.activeIndex++
        }
      } else if (keyCode === 38) {
        // up
        if (this.activeIndex === null || this.activeIndex === 0) {
          this.activeIndex = this.searchResult.length - 1
        } else {
          this.activeIndex--
        }
      } else if (keyCode === 13) {
        this.showModal = false
        if (this.activeIndex !== null) {
          this.keyword = this.searchResult[this.activeIndex].name
        }
      } else {
        this.activeIndex = null
      }
    },
    debounce (cb, delay) {
      let timeoutId
      return function (...args) {
        if (timeoutId) {
          clearTimeout(timeoutId)
        }
        timeoutId = setTimeout(() => {
          cb(args[0])
        }, delay)
      }
    },
  }
}
</script>
