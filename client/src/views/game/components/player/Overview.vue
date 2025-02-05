<template>
<div v-if="player">
  <player-title :player="player"/>

  <div class="row">
      <div class="col-auto text-center pl-0 pr-0">
        <img v-if="player.avatar" :src="getAvatarImage()">
        <i v-if="!player.avatar" class="far fa-user mr-2 mt-2 ml-2 mb-2" style="font-size:100px;"></i>
      </div>
      <div class="col bg-secondary">
          <statistics :playerId="playerId"/>
      </div>
  </div>

  <div class="row pt-2 pb-2 bg-primary" v-if="!(!userPlayer || !gameHasStarted || player.userId)">
    <div class="col">
      <button class="btn btn-secondary mr-1" @click="onOpenDiplomacyRequested" title="Open Diplomacy" v-if="isFormalAlliancesEnabled">
        <i class="fas fa-globe-americas"></i>
      </button>
      <button class="btn btn-secondary mr-1" @click="onOpenLedgerRequested" title="Open Ledger" v-if="isTradeEnabled">
        <i class="fas fa-file-invoice-dollar"></i>
      </button>
      <button class="btn btn-secondary" @click="onViewCompareIntelRequested" title="Compare Intel" v-if="!isDarkModeExtra">
        <i class="fas fa-chart-line"></i>
      </button>
    </div>
    <div class="col-auto">
      <button class="btn btn-success mr-1" @click="onViewConversationRequested"
        :class="{'btn-warning': conversation && conversation.unreadCount}"
        v-if="canCreateConversation">
        <i class="fas fa-envelope"></i>
        <span v-if="conversation && conversation.unreadCount" class="ml-1">{{conversation.unreadCount}}</span>
      </button>
      <button class="btn btn-info" v-if="!gameHasFinished && isTradeEnabled" @click="onOpenTradeRequested">
        <i class="fas fa-handshake"></i>
        Trade
      </button>
    </div>
  </div>
</div>
</template>

<script>
import eventBus from '../../../../eventBus'
import MENU_STATES from '../../../../services/data/menuStates'
import Statistics from './Statistics'
import PlayerTitleVue from './PlayerTitle'
import gameHelper from '../../../../services/gameHelper'
import ConversationApiService from '../../../../services/api/conversation'
import DiplomacyHelper from '../../../../services/diplomacyHelper'

export default {
  components: {
    'statistics': Statistics,
    'player-title': PlayerTitleVue
  },
  props: {
    playerId: String
  },
  data () {
    return {
      userPlayer: null,
      player: null,
      gameHasStarted: null,
      gameHasFinished: null,
      conversation: null
    }
  },
  async mounted () {
    this.userPlayer = gameHelper.getUserPlayer(this.$store.state.game)
    this.player = gameHelper.getPlayerById(this.$store.state.game, this.playerId)

    this.gameHasStarted = this.$store.state.game.state.startDate != null
    this.gameHasFinished = this.$store.state.game.state.endDate != null

    await this.loadConversation()
  },
  methods: {
    onViewConversationRequested (e) {
      if (this.conversation) {
        eventBus.$emit('onViewConversationRequested', {
          conversationId: this.conversation._id
        })
      } else {
        eventBus.$emit('onViewConversationRequested', {
          participantIds: [
            this.userPlayer._id,
            this.player._id
          ]
        })
      }
    },
    onViewCompareIntelRequested (e) {
      this.$emit('onViewCompareIntelRequested', this.player._id)
    },
    onOpenTradeRequested (e) {
      this.$emit('onOpenTradeRequested', this.playerId)
    },
    onOpenDiplomacyRequested (e) {
      this.$store.commit('setMenuState', {
        state: MENU_STATES.DIPLOMACY
      })
    },
    onOpenLedgerRequested (e) {
      this.$store.commit('setMenuState', {
        state: MENU_STATES.LEDGER
      })
    },
    getAvatarImage () {
      try {
        return require(`../../../../assets/avatars/${this.player.avatar}`)
      } catch (err) {
        console.error(err)
        
        return null
      }
    },
    async loadConversation () {
      if (this.userPlayer && this.userPlayer._id !== this.player._id) {
        try {
          let response = await ConversationApiService.privateChatSummary(this.$store.state.game._id, this.player._id)

          if (response.status === 200) {
            this.conversation = response.data
          }
        } catch (err) {
          console.error(err)
        }
      }
    }
  },
  computed: {
    isDarkModeExtra () {
      return gameHelper.isDarkModeExtra(this.$store.state.game)
    },
    isTradeEnabled () {
      return gameHelper.isTradeEnabled(this.$store.state.game)
    },
    canCreateConversation: function () {
      return this.$store.state.game.settings.general.playerLimit > 2
        && !gameHelper.isTutorialGame(this.$store.state.game)
    },
    isFormalAlliancesEnabled () {
      return DiplomacyHelper.isFormalAlliancesEnabled(this.$store.state.game)
    }
  }
}
</script>

<style scoped>
</style>
