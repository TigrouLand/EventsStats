<template>
  <div class="h-full flex bg-gray-100 overflow-y-hidden">
    <div class="overflow-x-auto hidden md:flex md:flex-shrink-0">
      <div class="flex flex-col w-64">
        <div class="flex flex-col h-0 flex-1">
          <div class="flex justify-center items-center h-20 bg-gray-900">
            <h2 class="text-white text-xl font-medium mt-3 p-3">
              Liste des parties
            </h2>
          </div>
          <div v-for="game in games" :key="game._id" class="px-4 py-3 bg-gray-900">
            <GameCard :game="game" :selected="selected(game)" />
          </div>
          <div v-for="game in archivedGames" :key="game._id" class="px-4 py-3 bg-gray-900" @click="selectedGame = game;">
            <GameCard :game="game" :selected="selected(game)" />
          </div>
          <div class="flex-1 flex flex-col overflow-y-auto">
            <nav class="flex-1 px-2 py-4 bg-gray-900 space-y-1" />
          </div>
        </div>
      </div>
    </div>
    <main class="flex-1 relative overflow-y-auto focus:outline-none bg-gray-800">
      <div class="md:hidden items-center flex bg-gray-900 overflow-x-auto">
        <div v-for="game in games" :key="game._id" class="flex flex-shrink-0 px-4 py-3 bg-gray-900">
          <GameCard :game="game" :selected="selected(game)" />
        </div>
        <div v-for="game in archivedGames" :key="game._id" class="flex flex-shrink-0 px-4 py-3 bg-gray-900" @click="selectedGame = game;">
          <GameCard :game="game" :selected="selected(game)" />
        </div>
      </div>
      <div v-if="selectedGame" class="flex flex-col sm:flex-row w-full px-4 sm:px-0">
        <div class="flex flex-col flex-shrink-0 flex-grow-0 sm:w-1/2 h-full sm:py-6 sm:mx-auto sm:px-6 md:px-8">
          <div class="bg-gray-900 border border-gray-500 rounded shadow text-white w-full my-4">
            <div class="w-full p-7">
              <div class="font-bold text-lg text-center">
                {{ selectedGame.name }}
              </div>
              <div v-if="isScheduled(selectedGame)" class="font-bold text-lg text-center mb-4 text-gray-400">
                <font-awesome-icon :icon="faClock" class="text-gray-400" />
                Partie programmée
              </div>
              <div v-else-if="isArchived(selectedGame)" class="font-bold text-lg text-center mb-4 text-gray-400">
                <font-awesome-icon :icon="faArchive" class="text-gray-400" />
                Partie archivée
              </div>
              <div v-else class="font-bold text-lg text-center mb-4 text-green-500">
                <font-awesome-icon :icon="faGamepad" class="text-green-500" />
                Partie en cours
              </div>
              <div class="mb-1">
                <font-awesome-icon :icon="faGamepad" class="text-white" />
                <span class="font-medium">Type de jeu :</span> {{ formatEventType(selectedGame.type) }}
              </div>
              <div class="mb-1">
                <font-awesome-icon :icon="faUsers" class="text-white" />
                <span class="font-medium">Participants :</span> {{ selectedGame.players ? selectedGame.players.length : 0 }}
              </div>
              <div v-if="isArchived(selectedGame)" class="mb-1">
                <font-awesome-icon :icon="faClock" class="text-white" />
                <span class="font-medium">Temps de jeu :</span> {{ formatTime() }}
              </div>
              <div class="mb-1">
                <font-awesome-icon :icon="faCalendar" class="text-white" />
                <span class="font-medium">Date de début :</span> {{ formatDate(selectedGame.startDate) }}
              </div>
              <div class="mb-1">
                <font-awesome-icon :icon="faPlug" class="text-white" />
                <span class="font-medium">Scénarios :</span> {{ selectedGame.modifiers ? selectedGame.modifiers.join(', '): '' }}
              </div>
              <div v-if="selectedGame.events" class="mb-1">
                <font-awesome-icon :icon="faBolt" class="text-white" />
                <span class="font-medium">Événements aléatoires :</span> {{ selectedGame.events.join(', ') }}
              </div>
            </div>
          </div>
          <div class="bg-gray-900 border border-gray-500 rounded shadow text-white w-full my-4 p-7">
            <div class="flex font-bold text-lg justify-center mb-2">
              Joueurs
            </div>
            <div class="table-auto">
              <template v-if="selectedGame.teams">
                <template v-for="team in selectedGame.teams.filter(t => t.name !== '')">
                  <div :key="team.name" class="text-lg font-semibold mt-3" :style="`color: rgb(${team.colors.join(', ')};`">
                    Équipe {{ team.name }}
                  </div>
                  <tr v-for="{ username, uuid } in getPlayersInTeam(team.name)" :key="uuid" class="flex items-center">
                    <td>
                      <img class="m-4 h-10 w-10 rounded-full" :src="'https://cravatar.eu/helmavatar/' + uuid + '/96'" alt="">
                    </td>
                    <td>
                      <div v-if="isAlive(uuid)" class="flex flex-col">
                        <div class="flex-row items-center font-bold">
                          {{ username }}
                        </div>
                        <div v-if="selectedGame.type === 'TaupeGun' && selectedGame.moles.includes(uuid)" class="flex items-center text-red-300">
                          Taupe<font-awesome-icon class="ml-2" :icon="faHandshakeSlash" />
                        </div>
                      </div>
                      <div v-else class="flex flex-col">
                        <div class="flex-row items-center text-gray-400">
                          {{ username }}<font-awesome-icon class="ml-2" :icon="faSkull" />
                        </div>
                        <div v-if="selectedGame.type === 'TaupeGun' && selectedGame.moles.includes(uuid)" class="flex items-center text-red-300">
                          Taupe<font-awesome-icon class="ml-2" :icon="faHandshakeSlash" />
                        </div>
                      </div>
                    </td>
                  </tr>
                </template>
              </template>
              <tr v-for="player in selectedGame.players" v-else :key="player" class="flex items-center">
                <td>
                  <img class="m-4 h-10 w-10 rounded-full" :src="'https://cravatar.eu/helmavatar/' + player + '/96'" alt="">
                </td>
                <td>
                  <div v-if="isAlive(player)" class="flex flex-col">
                    <div class="flex-row items-center font-bold">
                      {{ getUsernameByUuid(player) }}
                    </div>
                    <div v-if="selectedGame.type === 'LGUHC'" class="text-gray-200">
                      {{ getRoleForPlayer(player) }}
                    </div>
                  </div>
                  <div v-else class="flex flex-col">
                    <div class="flex-row items-center text-gray-400">
                      {{ getUsernameByUuid(player) }}<font-awesome-icon class="ml-2" :icon="faSkull" />
                    </div>
                    <div v-if="selectedGame.type === 'LGUHC'" class="text-gray-200">
                      {{ getRoleForPlayer(player) }}
                    </div>
                  </div>
                </td>
              </tr>
            </div>
          </div>
        </div>
        <div class="flex flex-col flex-shrink-0 flex-grow-0 sm:w-1/2 h-full sm:py-6 sm:mx-auto sm:px-6 md:px-8">
          <div class="bg-gray-900 border border-gray-500 rounded shadow text-white w-full my-4">
            <div class="w-full p-7">
              <div class="flex font-bold text-lg justify-center mb-2">
                Événements de la partie
              </div>
              <div v-for="(log, i) in selectedGame.logs" :key="log">
                <GameLog :log="log" :last="i === selectedGame.logs.length - 1" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import {
  faArchive,
  faClock,
  faGamepad,
  faUsers,
  faPlug,
  faCalendar,
  faSkull,
  faHandshakeSimpleSlash, faBolt
} from '@fortawesome/free-solid-svg-icons';
import { unix } from 'moment';
import GameCard from '../components/GameCard';

export default {
  name: 'GamesPage',
  components: { GameCard },
  data() {
    return {
      games: [],
      members: [],
      archivedGames: [],
      selectedGame: null,
      interval: null
    };
  },
  head: {
    title: 'Parties'
  },
  computed: {
    faUsers: () => faUsers,
    faGamepad: () => faGamepad,
    faArchive: () => faArchive,
    faClock: () => faClock,
    faPlug: () => faPlug,
    faCalendar: () => faCalendar,
    faSkull: () => faSkull,
    faHandshakeSlash: () => faHandshakeSimpleSlash,
    faBolt: () => faBolt
  },
  mounted() {
    this.fetch();
    this.interval = setInterval(this.refreshGames, 10000);
  },
  destroyed() {
    clearInterval(this.interval);
  },
  methods: {
    fetch() {
      this.$nuxt.$nextTick(async () => {
        this.$nuxt.$loading.start();
        const { games, archivedGames } = await this.$axios.$get('/games').catch(this.$nuxt.$loading.fail);
        this.games = games;
        this.archivedGames = archivedGames;
        this.members = await this.$axios.$get('/members').catch(this.$nuxt.$loading.fail);
        this.$nuxt.$loading.finish();
      });
    },
    selected(game) {
      if (!this.selectedGame)
        return false;
      return this.selectedGame.id === game.id;
    },
    isMole(uuid) {
      if (!this.selectedGame && this.selectedGame.moles)
        return Boolean(this.selectedGame.moles.includes(uuid));
      return false;
    },
    getRoleForPlayer(uuid) {
      if (!this.selectedGame && this.selectedGame.playerRoles)
        return;
      const username = this.getUsernameByUuid(uuid);
      return this.selectedGame.playerRoles[username];
    },
    getPlayersInTeam(name) {
      if (this.selectedGame && this.selectedGame.playerTeams)
        return Object.entries(this.selectedGame.playerTeams)
          .filter(([_, v]) => v === name)
          .map(([k, _]) => {
            return {
              username: k,
              uuid: this.getUuidByUsername(k)
            };
          });
    },
    getUuidByUsername(name) {
      return this.members.find(p => p.name === name)?.uuid;
    },
    getUsernameByUuid(uuid) {
      return this.members.find(p => p.uuid === uuid)?.name;
    },
    async refreshGames() {
      const { games, archivedGames } = await this.$axios.$get('/games').catch(this.$nuxt.$loading.fail);
      this.games = games;
      this.archivedGames = archivedGames;
    },
    isAlive(player) {
      if (this.selectedGame && this.selectedGame.alive)
        return this.selectedGame.alive.includes(player);
      return false;
    },
    isArchived(game) {
      return game.archiveDate !== -1;
    },
    isScheduled(game) {
      return game.startDate === -1 && game.scheduleDate !== -1;
    },
    formatDate(date) {
      return unix(date / 1000).format('DD/MM/YYYY à HH:mm');
    },
    formatTime() {
      let { minutes, seconds } = this.selectedGame;
      const hours = this.selectedGame.hours;

      if (minutes < 10)
        minutes = `0${minutes}`;
      if (seconds < 10)
        seconds = `0${seconds}`;
      return `${hours}:${minutes}:${seconds}`;
    },
    formatEventType(type) {
      switch (type) {
        case 'LGUHC':
          return 'LG-UHC';
        case 'UHCRun':
          return 'UHC-Run';
        default:
          return type || 'Inconnu';
      }
    }
  }
};
</script>

<style scoped>

</style>
