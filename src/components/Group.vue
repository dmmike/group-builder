<template>
  <b-card id="group" class="mb-0 text-center">
    <b-row slot="header">
      <b-col>
        <b-dropdown
          id="character-dropdown"
          class="float-right"
          :text="orderedFilteredOptions.length === 0 ? 'No Options' : 'Add a character'"
          :disabled="orderedFilteredOptions.length === 0">
          <b-dropdown-item-button
            v-for="(option, index) in orderedFilteredOptions"
            :key="index" @click="$parent.characters.push(option)">
            {{option.name}} ({{option.cost}})
          </b-dropdown-item-button>
        </b-dropdown>
      </b-col>
    </b-row>

    <b-row v-for="(character, index) in $parent.characters" :key="index">
      <character class="character" :character="character" @deleteCharacter="$parent.characters.splice(index, 1); character.resetCharacter()"></character>
    </b-row>
  </b-card>
</template>

<script>
    import _ from 'lodash'
    import Characters from "@/controllers/DataTables/Characters"
    import Affiliations from "@/controllers/DataTables/Affiliations"
    import Character from "@/components/Character"

    export default {
        name: "group",
        components: {Character},
        methods: {
        },
        computed: {
            galleonsRemaining() {
                return this.$parent.galleonLimit - this.$parent.galleonsSpent
            },
            options() {
                let options = []

                Affiliations[this.$parent.chosenAffiliation].forEach(identifier => {
                    options.push(Characters[identifier])
                })

                return options
            },
            orderedFilteredOptions() {
                let finalOptions = [...this.options]

                this.$parent.characters.forEach(character => {
                    if (!character.isHorde) {
                        let allCharactersFound = false
                        while (!allCharactersFound) {
                            let index = finalOptions.findIndex((option) => {
                                return option.name === character.name
                            })

                            if (index === -1) {
                                allCharactersFound = true
                            } else {
                                finalOptions.splice(index, 1)
                            }
                        }
                    }
                })

                finalOptions = finalOptions.filter(character => {
                    return character.cost <= this.galleonsRemaining
                })

                return _.orderBy(finalOptions, 'name')
            },
        }
    }
</script>

<style scoped>
  .character {
    margin-bottom: 10px
  }
</style>
