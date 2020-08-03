<template>
  <div id="app">
    <section class="hero has-background-danger">
      <div class="hero-body">
        <div class="container">
          <h1 class="title has-text-white">
            Password Pal
          </h1>
          <h2 class="subtitle has-text-white">
            Play with the below controls to generate a strong password
          </h2>
        </div>
      </div>
    </section>

    <section class="section has-background-light">
      <div class="container">
        <b-field>
          <b-input
            rounded
            v-model="pwd"
            size="is-large"
            type="text"
            placeholder="Password"
            icon-right="redo"
            icon-right-clickable
            @icon-right-click="generatePassword"/>
        </b-field>

        <progress
          class="progress is-primary is-small"
          :value="strength"
          max="100"/>
      </div>
    </section>

    <section class="section">
      <div class="container">

        <h1 class="title has-text-left">
          Options
        </h1>

        <div class="columns">

          <div class="column is-half has-text-left pr-6">
            <label class="label">
              Password length

              <span class="tag is-danger is-light is-rounded ml-3">
                {{characterCount}}
              </span>
            </label>
            <b-field
              class="mb-6">
                <b-slider
                  type="is-danger is-large"
                  :max="25"
                  :min="5"
                  v-model="characterCount"/>
            </b-field>

            <b-field label="Enter some terms">
                <b-taginput
                  v-model="terms"
                  type="is-danger"
                  placeholder="Add a term">
                </b-taginput>
            </b-field>
          </div>

          <div class="column is-half has-text-left pl-6">
            <label class="label">
              Include
            </label>
            <div class="field">
                <b-checkbox
                  v-model="includeUpperCase"
                  type="is-danger">
                    Upper Case
                </b-checkbox>
            </div>
            <div class="field">
              <b-checkbox
                v-model="includeNumbers"
                type="is-danger">
                  Numbers
              </b-checkbox>
            </div>
            <div class="field">
              <b-checkbox
                v-model="includeSpecial"
                type="is-danger">
                  Special characters
              </b-checkbox>
            </div>
          </div>
        </div>
      </div>
    </section>


    <footer class="footer py-6">
      <div class="content has-text-centered">
        <p>
          Built with
          <b-icon
            pack="fas"
            icon="heart"
            size="is-small"
            class="px-3 has-text-danger"/>
          <span>
            and
          </span>
          <b-icon
            pack="fas"
            icon="coffee"
            size="is-small"
            class="px-3 has-text-danger"/>
          by <a href="http://conorhiggins.com">Conor Higgins</a>.
        </p>
      </div>
    </footer>
  </div>
</template>

<script>
  export default {
    name: 'App',
    data: () => {
      return {
        pwd: "",
        terms: [],
        characterCount: 8,
        includeUpperCase: true,
        includeNumbers: true,
        includeSpecial: true,
      }
    },
    computed: {
      strength: (vm) => {

        let score = 100;

        const currentPassword = vm.pwd;
        const numberPattern = /\d+/g;
        const upperCasePattern = /[A-Z]/g;
        const nonAlphaNumericPattern = /[^a-z0-9]/gi;
        const numberMatch = currentPassword.match(numberPattern);
        const upperCaseMatch = currentPassword.match(upperCasePattern);
        const nonAlphaNumericMatch = currentPassword.match(nonAlphaNumericPattern);

        // 8 chars long minimum
        // includes at least 1 upper case
        // includes at least 1 number
        // includes at least 1 special character

        // Immediately mark it as weak if character count is low
        if (currentPassword.length < 8) {
          score = 50;
        }

        if (numberMatch == undefined) {
          score -= 20;
        }
        else {
          if (numberMatch.join('').length == 1) {
            score -= 10;
          }
        }

        if (upperCaseMatch == undefined){
          score -= 20;
        }
        else {
          if (upperCaseMatch.join('').length == 1) {
            score -= 10;
          }
        }

        if (nonAlphaNumericMatch == undefined){
          score -= 20;
        }
        else {
          if (nonAlphaNumericMatch.join('').length == 1) {
            score -= 10;
          }
        }
        return score;
      },
    },
    methods: {
      generatePassword() {
        console.log('here now!');
        let newPassword = '';
        let baseTerms = this.terms;
        const alphaChars = "abcdefghijklmnopqrstuvwxyz";
        const numericChars = "0123456789";
        const specialChars = "?!-+`~[]()|@"

        const characterMap = {
          '1': '!',
          'a': '@',
          'e': '3',
          'i': '1',
          'o': '0',
          't': '7',
          's': '5',
          'S': '$',
          'P': '?'
        }

        if (!baseTerms.length) {
          let arr = Array.from(Array(this.characterCount).keys());
          let randomString = "";

          arr.forEach(() => {
            const factor = Math.random();
            let index = 0;

            if (factor > .8 && this.includeSpecial) {
              index = Math.floor(Math.random() * specialChars.length);
              randomString += specialChars[index];
            }
            else if (factor > .5 && this.includeNumbers) {
              index = Math.floor(Math.random() * numericChars.length);
              randomString += numericChars[index];
            }
            else {
              index = Math.floor(Math.random() * alphaChars.length);
              randomString += alphaChars[index];
            }
          });

          baseTerms = [randomString];
        }

        // If the user has added terms, then use characters from each term
        baseTerms.forEach((t) => {
          // Split all the characters into an array
          const split = t.split('');

          // For each string in the array,
          // randomly mutate the character by:
          // - swapping for a similar character in the map
          // - switching its case from lower to upper
          split.forEach((c) => {
            const factor = Math.random();

            if (factor > .5 && this.includeSpecial && characterMap[c] != undefined){
                newPassword += characterMap[c];
            }
            else {
              if (factor < .2 && this.includeUpperCase) {
                newPassword += c.toUpperCase();
              }
              else {
                newPassword += c;
              }
            }
          })
        });

        this.pwd = newPassword;
      },
    },
    mounted() {
      this.generatePassword();
    },
    watch: {
      characterCount() {
        this.generatePassword();
      },
      includeUpperCase() {
        this.generatePassword();
      },
      includeNumbers() {
        this.generatePassword();
      },
      includeSpecial() {
        this.generatePassword();
      },
      terms() {
        this.generatePassword();
      }
    }
  }
</script>

<style lang="scss">
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }
</style>
