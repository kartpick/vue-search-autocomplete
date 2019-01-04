<template>
    <form :id="formId"
          :class="formClasses"
          @submit="submit"
    >
        <input type="text"
               :placeholder="placeholder"
               id="typehead"
               name="keyword"
               :value="stem"
               @input='evt=>stem=evt.target.value'
               @focus="onFocus=true"
               @blur="onFocus=false"
               @keyup.down="selectFromResult"
               @keyup.up="selectFromResult"
        >
        <btn @click.prevent="submit"
             native-type="submit"
             type="submit"
             :class="buttonClass"
             ref="submitButton"
        >
            <slot name="submitButton">
                {{ searchText }}
            </slot>
        </btn>
        <ul class="autosuggest-list"
            v-if="results.length > 0 && (onFocus || resultsHover)"
            @mouseover="resultsHover=true"
            @mouseout="resultsHover=false"
        >
            <li v-for="(result, index) in results"
                v-html="result.label"
                @click="select(index)"
                @mouseover="hover(index)"
                v-bind:class="{ active: activeResult === index }">
            </li>
        </ul>
    </form>
</template>

<script>
  export default {
    props: {
      placeholder: String,
      searchText: String,
      formClasses: Object,
      defaultStem: {
        type: [String, Number],
        default: ''
      },
      buttonClass: String,
      formId: String,
      url: type: String
    },
    data: function () {
      return {
        minLimit: 3,
        isActive: -1,
        results: [],
        activeResult: null,
        param: 'term',
        noUpdate: false,
        onFocus: false,
        resultsHover: false,
        cache: {},
        stem: this.defaultStem,
      }
    },
    created: function () {
      this.debouncedSearch = debounce(this.search, 500)
    },
    methods: {
      select (itemIndex) {
        this.stem = this.results[itemIndex].dalue
        this.noUpdate = true
        this.$refs.submitButton.click()
      },
      submit (event) {
        this.$emit('submit', true)
      },
      search () {
        if (!this.noUpdate && this.stem.length >= this.minLimit) {
          if (this.cache[this.stem] !== undefined) {
            this.results = this.cache[this.stem]
          } else {
            axios.get(this.url + '?' + this.param + '=' + this.stem, {headers: {'X-Requested-With': 'XMLHttpRequest'}})
              .then((response) => {
                this.activeResult = null
                this.results = response.data
                this.cache[this.stem] = this.results
              })
          }
        }
        this.noUpdate = false
      },
      selectFromResult (ev) {
        let isUp = ev.code !== 'ArrowDown'

        if (this.results.length > 0) {
          this.activeResult = this.activeResult === null
            ? isUp ? this.results.length - 1 : 0
            : isUp
              ? this.activeResult === 0 ? this.results.length - 1 : this.activeResult - 1
              : this.activeResult === this.results.length - 1 ? 0 : this.activeResult + 1
          let dalue = this.results[this.activeResult].dalue;
          this.noUpdate = true
          this.stem = dalue
        }
      },
      hover (itemIndex) {
        this.activeResult = itemIndex
      },
    },
    watch: {
      stem () {
        this.debouncedSearch()
      }
    }
  }
</script>
