<script>
import missingno from "@/assets/images/missingno.png";

export default {
  data() {
    return {
      timer: undefined,
      pokemonName: "",
      pokemonData: undefined,
      showNextPokemon: false,
      animationHelperClass: "",
      notFound: false,
      overlayOpen: false,
    };
  },
  methods: {
    searchPokeDex() {
      clearTimeout(this.timer);
      this.timer = setTimeout(() => {
        this.getData();
      }, 2000);
    },
    getData() {
      if (this.pokemonName === "") {
        return;
      }
      if (this.showNextPokemon) {
        this.animationHelperClass = "hideOldResults";
      }
      fetch(
        "https://pokeapi.co/api/v2/pokemon/" + this.pokemonName.toLowerCase(),
        {
          method: "GET",
        }
      ).then((res) => {
        this.animationHelperClass = "";
        setTimeout(() => {
          this.animationHelperClass = "showNextResults";
        }, 500);
        if (res.status == 200) {
          res.json().then((response) => {
            this.notFound = false;
            this.pokemonData = response;
          });
        } else {
          this.notFound = true;
          this.pokemonData = {
            abilities: [{ name: "-" }],
            name: "MissingNo.",
            order: 0,
            sprites: {
              front_default: missingno, // "/src/assets/images/missingno.png",
            },
            stats: [
              { base_stat: 33, stat: { name: "hp" } },
              { base_stat: 136, stat: { name: "attack" } },
              { base_stat: 0, stat: { name: "defense" } },
              { base_stat: 6, stat: { name: "special-attack" } },
              { base_stat: 6, stat: { name: "special-defense" } },
              { base_stat: 29, stat: { name: "speed" } },
            ],
            types: [{ type: { name: "unknown" } }],
            weight: 10,
          };
        }
        this.showNextPokemon = true;
      });
    },
    closeOverlay() {
      this.overlayOpen = false;
    },
  },
};
</script>

<template>
  <div class="overlay" v-if="overlayOpen" :on-click="closeOverlay"></div>
  <div class="pokedex-wrapper">
    <div class="result-wrapper">
      <div class="not-found" v-if="notFound">
        <span>Über dieses Pokémon ist uns nichts bekannt</span>
      </div>
      <div
        class="pokemon-wrapper"
        v-if="pokemonData"
        :class="animationHelperClass"
      >
        <img
          :src="pokemonData.sprites.front_default"
          alt="pokemon sprite"
          class="pokemon-sprite"
        />
        <span class="pokemon-name">{{ pokemonData.name }}</span>
        <span class="pokemon-order"
          >Pokédex Entry: #{{ pokemonData.order }}</span
        >
        <div class="pokemon-types-wrapper">
          <span
            v-for="type in pokemonData.types"
            class="pokemon-type"
            :class="type.type.name"
            :key="type.name"
          >
            {{ type.type.name }}
          </span>
        </div>
        <div class="pokemon-stats-wrapper">
          <div
            v-for="(stat, index) in pokemonData.stats"
            class="pokemon-stat"
            :class="stat.stat.name"
            :key="index"
          >
            <span class="stat-text">
              {{ stat.stat.name }} : {{ stat.base_stat }}</span
            >
            <div
              class="stat-bar"
              :style="{ width: (stat.base_stat * 100) / 255 + '%' }"
            ></div>
          </div>
        </div>
      </div>
    </div>
    <div class="form-wrapper">
      <h1>Pokédex</h1>
      <form>
        <input
          type="text"
          placeholder="Pokemon Name"
          v-model="pokemonName"
          @keyup="searchPokeDex"
        />
        <small>* In Englisch (z.B. Charizard)</small>
      </form>
    </div>
  </div>
</template>
