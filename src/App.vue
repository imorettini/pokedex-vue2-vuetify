<template>
  <v-app>
    <v-container>
      <v-container>
        <v-row>
          <v-container>
            <v-img
              :src="require('./assets/pokedex.png')"
              class="my-3"
              contain
              height="200"
            />
           <v-img
              :src="require('./assets/pokedex_logo.png')"
              class="my-3"
              contain
              height="150"
            />
          </v-container>
        </v-row>
        <v-text-field
          v-model="search"
          label="Pesquisar"
          placeholder="Pesquisar pokémon"
          solo
        ></v-text-field>
        <v-row>
          <v-col
            cols="2"
            v-for="pokemon in filtered_pokemons"
            :key="pokemon.name"
          >
            <v-card @click="show_pokemon(get_id(pokemon))">
              <v-container>
                <v-row class="mx-0 d-flex justify-center">
                  <img
                    :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${get_id(
                      pokemon
                    )}.png`"
                    :alt="pokemon.name"
                    width="80%"
                  />
                </v-row>
                <h2 class="text-center">{{ get_name(pokemon) }}</h2>
              </v-container>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-container>
    <v-dialog v-model="show_dialog" width="800">
      <v-card v-if="selected_pokemon" class="px-4">
        <v-container>
          <v-row class="d-flex align-center">
            <v-col cols="4">
              <img
                :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${selected_pokemon.id}.png`"
                :alt="selected_pokemon.name"
                width="80%"
              />
            </v-col>
            <v-col cols="8">
              <h1>{{ get_name(selected_pokemon) }}</h1>
              <v-chip
                label
                class="mr-2"
                v-for="type in selected_pokemon.types"
                :key="type.slot"
                >{{ type.type.name }}</v-chip
              >
              <v-divider class="my-4"></v-divider>
              <v-chip>Altura: {{ selected_pokemon.height * 2.54 }}cm</v-chip>
              <v-chip class="ml-2"
                >Peso:
                {{
                  (selected_pokemon.weight * 0.45359237).toFixed(0)
                }}kg</v-chip
              >
            </v-col>
          </v-row>
          <h2>Moves</h2>
          <v-simple-table>
            <thead>
              <tr>
                <th class="text-left">Level</th>
                <th class="text-left">Name</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="item in filter_moves(selected_pokemon)"
                :key="item.move.name"
              >
                <td>{{ get_move_level(item) }}</td>
                <td>{{ item.move.name }}</td>
              </tr>
            </tbody>
          </v-simple-table>
        </v-container>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  name: "App",

  components: {},

  data() {
    return {
      pokemons: [],
      search: "",
      show_dialog: false,
      selected_pokemon: null,
    };
  },

  mounted() {
    axios
      .get("https://pokeapi.co/api/v2/pokemon?limit=151")
      .then((response) => {
        this.pokemons = response.data.results;
      });
  },

  methods: {
    get_id(pokemon) {
      return pokemon.url.split("/")[6];
    },

    get_name(pokemon) {
      return pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1);
    },
    show_pokemon(id) {
      axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`).then((response) => {
        this.selected_pokemon = response.data;
        this.show_dialog = !this.show_dialog;
      });
    },

    get_move_level(move) {
      for (let version of move.version_group_details) {
        if (
          version.version_group.name == "sword-shield" &&
          version.move_learn_method.name == "level-up"
        ) {
          return version.level_learned_at;
        }
      }
    },

    filter_moves(pokemon) {
      return pokemon.moves.filter((item) => {
        let include = false;
        for (let version of item.version_group_details) {
          if (
            version.version_group.name == "sword-shield" &&
            version.move_learn_method.name == "level-up"
          ) {
            include = true;
          }
        }
        return include;
      });
    },
  },
  computed: {
    filtered_pokemons() {
      return this.pokemons.filter((item) => {
        return item.name.toLowerCase().includes(this.search.toLowerCase());
      });
    },
  },
};
</script>

<style>
#app {
  background: rgb(183, 171, 197);
  background: linear-gradient(
    90deg,
    rgba(183, 171, 197, 1) 0%,
    rgba(166, 168, 216, 1) 21%,
    rgba(164, 231, 162, 1) 48%,
    rgba(231, 233, 158, 1) 66%,
    rgba(233, 194, 153, 1) 87%,
    rgba(233, 148, 148, 1) 100%
  );
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover !important;
  background-position: center;
  min-height: 100vh;
}
</style>
