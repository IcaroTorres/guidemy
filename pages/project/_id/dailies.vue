<template>
  <v-card >
    <!-- <v-container fluid grid-list-xl> -->
    <v-card-text style="position: sticky; top: 46px; z-index: 2;" :class="{'secondary darken-1':lightOut, 'grey lighten-3': !lightOut}">
      <v-layout row wrap justify-space-between align-center>
        <v-flex xs12 sm6 v-if="isManager">
          <v-select
            dense
            hide-details
            solo-inverted
            flat
            style="opacity: .5;"
            v-model="selectedWorker"
            :items="team"
            item-value="id"
            item-text="username"
            label="project team">
            <v-list-tile slot="prepend-item" disabled>
              <v-list-tile-avatar color="primary title">
                M
              </v-list-tile-avatar>
              <v-list-tile-content>
                <v-list-tile-title><span class="subheading primary--text">{{loggedUserObj.username}}</span> - {{selectionSentense[0]}}</v-list-tile-title>
                <v-list-tile-sub-title class="caption grey--text" v-text="selectionSentense[1]"/>
              </v-list-tile-content>
            </v-list-tile>
            <template slot="item" slot-scope="data">
              <template v-if="typeof data.item !== 'object'">
                <v-list-tile-content v-text="data.item"></v-list-tile-content>
              </template>
              <template v-else>
                <v-list-tile-avatar>
                  <img :src="useravatar(data.item.id)">
                </v-list-tile-avatar>
                <v-list-tile-content>
                  <v-list-tile-title v-html="data.item.username"></v-list-tile-title>
                  <v-list-tile-sub-title class="caption grey--text" v-html="data.item.displayName"></v-list-tile-sub-title>
                </v-list-tile-content>
              </template>
            </template>
            <v-divider slot="prepend-item" class="my-1"></v-divider>
          </v-select>
        </v-flex>
        <v-flex xs12 :sm4="isManager" :md3="isManager" :sm12="!isManager" :class="{'caption text-xs-center': true, 'mt-2': xsOnly}">
          <v-layout row justify-space-between align-center>
            <div><span :class="`${resultColor}--text title`"> {{coworkerResults.participation}}%</span><br> Participation</div>
            <div><span :class="`${resultColor}--text title`"> {{coworkerResults.attended}}</span><br> Attended</div>
            <div><span :class="`${resultColor}--text title`"> {{coworkerResults.missed}}</span><br> Missed</div>
            <div><span :class="`${resultColor}--text title`"> {{coworkerResults.total}}</span><br> Total</div>
          </v-layout>
        </v-flex>
      </v-layout>
    </v-card-text>
    <!-- </v-container> -->
    <v-divider/>
    <v-container fluid grid-list-xl class="px-4 pt-4 pb-2 new-daily-container" v-if="newDaily">
      <v-layout row justify-center align-content-start >
        <v-flex>
          <v-layout row align-center>
            <v-avatar  size="48px" color="grey lighten-4 mb-2">
              <img :src="loggedUserObj.picture" alt="avatar">
            </v-avatar>
            <div class="ml-2">
              <a class="primary--text subheading">{{loggedUserObj.username}}</a><br>
              {{newDaily.created | locale}} <span class="body-2 ml-2">TODAY!</span>
            </div>
            <v-spacer></v-spacer>
            <v-icon @click="open = !open">{{open ? 'unfold_less' : 'unfold_more'}}</v-icon>
            <v-btn flat small :disabled="!open" color="warning" @click.stop="onAnswerDaily"> responder <v-icon class="pl-2">send</v-icon></v-btn>
          </v-layout>
          <v-layout row wrap align-start class="px-2 pt-2">
            <v-flex xs12 v-for="n in 3" :key="n" class="pa-0 ma-0" v-if="open">
              <v-textarea
                box
                placeholder="Question was not responded"
                rows="2"
                row-height="16"
                :color="dailyColor(newDaily)"
                :append-icon="icons[n-1]"
                class="text-xs-right warning--text"
                :hint="`Answer for question r${n}`"
                persistent-hint
                :label="questions[n-1]"
                v-model="newDaily['r'+(n)]"
                auto-grow
              ></v-textarea>
            </v-flex>
          </v-layout>
        </v-flex>
      </v-layout>
    </v-container>
    <v-divider></v-divider>
    <v-card-text height="900">
      <v-container fluid grid-list-xl class="py-1">
        <v-layout row wrap>
          <v-flex xs12 class="followline-decorated">
            <template v-for="(daily, didx) in dailies">
              <v-divider class="my-2 mr-5" :key="didx" v-if="didx !== 0"/>
              <v-layout row justify-center align-content-start :key="daily.id" class="connected-left">
                <v-flex>
                  <div :key="didx">
                    <v-layout row>
                      <v-avatar  size="48px" color="grey lighten-4 mb-2" >
                        <img :src="useravatar(daily.assigned)" alt="avatar">
                      </v-avatar>
                      <div class="ml-2">
                        <nuxt-link
                          :to="{name: 'daily-id', params: {id: daily.id}}"
                          class="primary--text subheading">
                          {{username(daily.assigned)}}
                        </nuxt-link><br>
                        <div v-if="daily.finished" class="caption grey--text mr-2">
                          responded: {{daily.finished | postFormat}}
                        </div>
                      </div>
                      <v-spacer></v-spacer>
                    </v-layout>
                  </div>
                  <v-layout row wrap align-start class="pa-2">
                    <v-flex xs12 v-for="n in 3" :key="n" class="pa-0 ma-0">
                      <div >                          
                        <span class="caption primary--text">{{questions[n-1]}}</span>
                        <span class="error--text pl-2" style="font-size: 10px" v-if="daily.status === -1">(Question was not accepted)</span>
                      </div>
                      <p :class="dailyColor(daily) + '--text'">
                        {{daily['r'+n] || 'Question was not responded'}}
                      </p>
                    </v-flex>
                  </v-layout>
                </v-flex>
                <v-flex style="max-width: 40px;">
                  <v-layout column fill-height align-center justify-start style="position: relative;">
                    <div style="transform: translateY(-18px)" class="caption">
                      <span v-if="didx === 0 && new Date(daily.created).getDate() === new Date().getDate()">TODAY!</span>
                      <span v-else>{{daily.created | locale}}</span>
                    </div>
                    <v-icon :class="{'success--text':daily.status === 1,  'action-middle-1': true}"
                      :disabled="!isManager" @click.stop="judgeDaily({id: daily.id, status: daily.status === 1 ? 0 : 1})">
                      done_outline
                    </v-icon>
                    <v-icon :class="{'error--text':daily.status === -1,  'action-middle-2': true}"
                      :disabled="!isManager" @click.stop="judgeDaily({id: daily.id, status: daily.status === -1 ? 0 : -1})">
                      gavel
                    </v-icon>
                  </v-layout>
                </v-flex>
              </v-layout>
            </template>
          </v-flex>
        </v-layout>
      </v-container>
    </v-card-text>
  </v-card>
</template>

<script>
import { mapGetters, mapState, mapActions } from 'vuex'
export default {
  validate ({ params, store }) {
    return !!store.state.projects[params.id] // Must be a valid project id
  },
  data () {
    return {
      dialog: false,
      questions: ['What did you do yesterday?', 'What will you do today?', 'Are there any impediments in the way?'],
      icons: ['alarm_on', 'alarm_add', 'alarm_off'],
      selectionSentense: [
        'you are the project manager',
        'View project daily meetings and evaluate team progress.'
      ],
      selectedWorker: undefined,
      newDaily: undefined,
      open: false
    }
  },
  created () {
    this.selectedWorker = this.project.team[0]
  },
  computed: {
    ...mapState(['projects', 'users', 'loggedUser', 'lightOut']),
    ...mapGetters([
      'temperColor',
      'username',
      'useravatar',
      'projectDailies',
      'loggedUserObj',
      'mdAndUp',
      'xsOnly'
    ]),
    project () { return this.projects[this.$route.params.id] },
    manager () { return this.users[this.project.manager] },
    team () { return this.project.team.map(uid => this.users[uid]) },
    assigned () { return this.selectedWorker ? this.users[this.selectedWorker] : this.loggedUserObj },
    predailes () {
      const uid = this.loggedUser === this.project.manager ? this.selectedWorker : this.loggedUser
      return (this.projectDailies(this.project.id)[uid] || []).sort(this.sortByStart())
    },
    dailies () {
      // const uid = this.loggedUser === this.project.manager ? this.selectedWorker : this.loggedUser
      let dailyList = this.predailes
      const iHaveNewDaily = (
        dailyList.length > 0 &&
        dailyList[0].created &&
        new Date(dailyList[0].created).getDate() === new Date().getDate() &&
        dailyList[0].status === 0 &&
        this.loggedUser === dailyList[0].assigned
      )
      // console.log(
      //   dailyList.length > 0,
      //   dailyList[0].created,
      //   new Date(dailyList[0].created).getDate() === new Date().getDate(),
      //   dailyList[0].status === 0,
      //   this.loggedUser === dailyList[0].assigned,
      //   'new daily?'
      // )

      if (iHaveNewDaily) {
        this.newDaily = { ...dailyList[0] }
        this.open = true
        return dailyList.slice(1)
      } else {
        return dailyList
      }
    },
    isManager () { return this.loggedUser === this.project.manager },
    coworkerResults () {
      const pendingDaily = this.newDaily && (this.loggedUser === this.selectedWorker) ? 1 : 0
      let total = this.dailies.length + pendingDaily
      let attended = this.dailies.filter(d => d.status === 1).length
      let participation = (attended * 100 / total).toFixed(2)
      return {
        participation: participation,
        attended: attended,
        missed: total - attended,
        total: total
      }
    },
    resultColor () {
      return this.temperColor(this.coworkerResults.total, this.coworkerResults.attended)
    }
  },
  methods: {
    ...mapActions(['answerDaily', 'judgeDaily']),
    onAnswerDaily () {
      this.answerDaily(this.newDaily)
      this.newDaily = undefined
    },
    dailyColor (d) {
      return d.status === -1 ? 'error' : d.status === 0 ? 'warning' : 'success'
    }
  }
}
</script>
<style scoped>
  .v-expansion-panel__header {
    padding-left: 0 !important ;
    padding-right: 0 !important; 
  }
  .connected-left{position: relative}
  .connected-left::before, .connected-left::after {
    content: ' ';
    width: 1px;
    position: absolute;
    background-color: #555;
  }
  .connected-left::before {
    margin: 2.5px 20px 20px 0;
    top:0;
    bottom:50%;
    right:0;
  }
  .connected-left::after {
    margin: 35px 20px 5px 0;
    top:50%;
    bottom:0;
    right:0;
  }
  .action-middle-1 {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translateX(-50%) translateY(-35%);
  }
  .action-middle-2 {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translateX(-50%) translateY(70%);
  }
</style>
