<template>
    <div class="autocomplete">
        <label :for="inputId">{{ label }}</label>
        <div class="autocomplete__input-group" 
            :class="{ 'autocomplete__input-group--error': validatonMessage }"
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
                aria-activedescendant="activedescendant"
                v-model="inputText"
                @input="changeText"
                class="autocomplete__input"
                @keydown.down="onArrowDown"
                @keydown.up="onArrowUp"
                @keydown.enter="onEnter"
                @keydown.esc="onEsc"
                @blur="closeOnBlur"
                @focus="openOnFocus"
                :placeholder="placeholder"
                ref="autcompleteInput"
            >

            <ul v-show="isOpen && suggestions.length" id="autocomplete-results" role="listbox" class="autocomplete-results">
                <li 
                    v-for="(suggestion, index) in suggestions"
                    role="option"
                    :aria-selected="index === arrowCounter"
                    :key="index"
                    :id="getId(index)"
                    :class="{'is-active': index === arrowCounter }"
                    class="autocomplete-result"
                    @click="setResult(suggestion, index)"
                >
                    {{ suggestion.label }}
                </li>
            </ul>
            <span class="sr-only" role="status" aria-live="assertive" aria-relevant="additions">{{ currentFocusedOption }}</span>
        </div>
        <label :for="inputId" v-show="validatonMessage" class="form-validation form-validation--error">{{ validatonMessage }}</label>
    </div>
</template>
<script>

import { required, minLength } from 'vuelidate/lib/validators';
export default {
    name: "AutoComplete",
    props: {
        label: {
            type: String,
            required: true
        },
        placeholder: {
            type: String,
            required: false,
            default: ""
        },
        validatonMessage: {
            type: String,
            required: false,
            default: ""
        },
        inputName: {
            type: String,
            required: false,
            default: "email"
        },
        items: {
            type: Array,
            required: false,
            default: () => [],
        },
        isLoading: {
            type: Boolean,
            required: false,
            default: false
        },
    },
    data () {
        return {
            id: this._uid,
            inputId: "autocomplete-input-" + this._uid,
            inputText: "",
            isOpen: false,
            arrowCounter: -1,
            currentFocusedOption: ""
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
            this.$emit('autocompleteInput', this.inputText.trim());
            this.open();
        },
        open () {
            if (!this.isOpen) this.arrowCounter = -1;
            this.isOpen = true;
        },
        setResult (suggestion, index) {
            this.$emit("autocompleteSelected", suggestion);
            this.inputText = suggestion.label;
            this.close();
        },
        onArrowDown (e) {
            e.preventDefault();
            if (this.arrowCounter < this.suggestions.length - 1) {
                this.arrowCounter = this.arrowCounter + 1;
            } else {
                this.arrowCounter = 0;
            }
            this.currentFocusedOption = this.suggestions[this.arrowCounter].value;
        },
        onArrowUp (e) {
            if (this.arrowCounter > 0) {
                this.arrowCounter = this.arrowCounter - 1;
            } else {
                this.arrowCounter = this.suggestions.length - 1;
            }
            this.currentFocusedOption = this.suggestions[this.arrowCounter].value;
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
        }
    },
    computed: {
        suggestions () {
            if (this.items.length) {
                if (typeof this.items[0] !== "object") throw new Error("Items in autocomplete must be a list of objects with a label and a value.");
            }
            let items = this.items.filter(item => item.label.toUpperCase().indexOf(this.inputText.toUpperCase().trim()) > -1);
            if (items.length) return items;
            return [];
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

.autocomplete {
  position: relative;
  width: 180px;
}

.autocomplete-results {
  padding: 0;
  margin: 0;
  border: 1px solid #eeeeee;
  height: 120px;
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
</style>