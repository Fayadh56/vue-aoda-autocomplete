<template>
    <div class="autocomplete">
        <div class="autocomplete__input-group" 
            role="combobox"
            aria-haspopup="listbox"
            aria-owns="autocomplete-results"
            :aria-expanded="isOpen">
            <input 
                type="text" 
                :name="inputName" 
                :id="inputId" 
                autocomplete="off" 
                aria-autocomplete="list"
                aria-label="Choose a country"
                role="searchbox"
                aria-controls="autocomplete-results"
                :aria-activedescendant="activedescendant"
                v-model="searchInput"
                @input="changeText"
                class="autocomplete__input"
                @blur="closeOnBlur"
                @focus="activedescendant"
                @keydown.down="onArrowDown"
                @keydown.up="onArrowUp"
                @keydown.enter="onEnter"
                @keydown.esc="onEsc"
                :placeholder="placeholder"
                ref="autcompleteInput"
            >

            <ul v-show="isOpen && suggestions.length" id="autocomplete-results" role="listbox" class="autocomplete-results">
                <li 
                    v-for="(suggestion, index) in suggestions"
                    role="option"
                    :aria-selected="index === arrowCounter"
                    :key="index"
                    :id="suggestion.placeId"
                    :class="{'is-active': index === arrowCounter }"
                    class="autocomplete-result"
                    @click="setResult(suggestion, index)"
                    @keydown.enter="onEnter"
                    @keydown.esc="onEsc"
                >
                    {{ suggestion.description }}
                </li>
            </ul>
            <span class="sr-only" role="status" aria-live="assertive" aria-relevant="additions">{{ currentFocusedOption }}</span>
        </div>
    </div>
</template>
<script>
import { required, minLength } from 'vuelidate/lib/validators';
export default {
    name: "AutoComplete",
    props: {
        placeholder: {
            type: String,
            required: false,
            default: ""
        },
        suggestions: {
            type: Object,
            required: true,
        },
    },
    data () {
        return {
            id: this._uid,
            inputId: "autocomplete-input-" + this._uid,
            inputText: "",
            isOpen: false,
            arrowCounter: -1,
            currentFocusedOption: "",
            selectedSuggestion: null,
            searchInput: '',
            address: {},
            activedescendant: ''
        }
    },
    mounted() {
        document.addEventListener('click', this.handleClickOutside);
    },  
    methods: {
        getId(index) {
            return `result-item-${index}`;
        },
        changeText (e) {
            this.$emit('onSearchInput', this.searchInput.trim());
            this.open();
        },
        open () {
            if (!this.isOpen) this.arrowCounter = -1;
            this.isOpen = true;
        },
        setResult (suggestion, index) {
            this.$emit("autocompleteSelected", suggestion);
            this.searchInput = suggestion.description;
            this.close();
        },
        onArrowDown (e) {
            e.preventDefault();
            const { key } = event

            switch (key) {
            case 'Down': // IE/Edge
            case 'ArrowDown': {
                if (this.arrowCounter < this.suggestions.length - 1) {
                    this.arrowCounter = this.arrowCounter + 1;
                } else {
                    this.arrowCounter = 0;
                }
                this.currentFocusedOption = this.suggestions[this.arrowCounter].description;
                this.setActiveDescendant();
                break
            }
            default: return;}
        },
        onArrowUp (e) {
            if (this.arrowCounter > 0) {
                this.arrowCounter = this.arrowCounter - 1;
            } else {
                this.arrowCounter = this.suggestions.length - 1;
            }
            this.currentFocusedOption = this.suggestions[this.arrowCounter].description;
            this.setActiveDescendant();
        },
        onEnter (e) {
            if (this.isOpen) {
                e.preventDefault();
                if (this.arrowCounter > -1) {
                    this.setResult(this.suggestions[this.arrowCounter], this.arrowCounter);
                    this.close();
                }
            }
        },
        onEsc () {
            this.close();
        },
        closeOnBlur () {
            setTimeout(() => {
                if (document.activeElement !== this.$refs.autcompleteInput) {
                    this.close();
                }
            }, 200);
        },
        openOnFocus (e) {
            if (e.target.value && !this.isOpen) {
                this.open();
            }
        },
        close () {
            this.currentFocusedOption = "";
            this.isOpen = false;
        },
        handleClickOutside (e) {
            if (!this.$el.contains(e.target)) {
                this.close();
            }
        },
        setActiveDescendant () {
            this.activedescendant = this.getId(this.arrowCounter);
        }
    },
    destroyed () {
        document.removeEventListener('click', this.handleClickOutside);
    }
}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

.autocomplete, .autocomplete__input {
  position: relative;
  width: 680px;
}

.autocomplete-results {
  padding: 0;
  margin: 0;
  border: 1px solid #eeeeee;
  height: 150px;
  overflow: auto;
  width: 100%;
}

.autocomplete-result {
  list-style: none;
  text-align: left;
  padding: 4px 2px;
  cursor: pointer;
}

.autocomplete-result.is-active,
.autocomplete-result:hover {
  background-color: #4aae9b;
  color: white;
}

.sr-only {
    display: none;
}
</style>