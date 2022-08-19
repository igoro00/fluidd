<template>
  <v-card-text>
    <v-row>
      <v-col cols="6">
        <v-text-field
          v-model="filename"
          hide-details
          outlined
          dense
          :label="'File'"
          @keydown.enter.prevent="loadFile(filename)"
        />
      </v-col>
      <v-col
        cols="2"
      >
        <app-btn
          :elevation="2"
          block
          @click="loadFile(filename)"
        >
          Load
        </app-btn>
      </v-col>
      <v-col
        cols="2"
      >
        <app-btn
          :elevation="2"
          block
          @click="saveFile(filename)"
        >
          Save
        </app-btn>
      </v-col>
      <v-col
        cols="2"
      >
        <app-btn
          :elevation="2"
          block
          @click="filename='';moves=[]"
        >
          Clear
        </app-btn>
      </v-col>
      <v-col
        cols="12"
      >
        <app-btn
          block
          @click="sendGcode(moves.join('\n'))"
        >
          Run All
        </app-btn>
      </v-col>
    </v-row>
    <v-col
      :key="updateGcodeList"
      cols="12"
    >
      <div class="pt-4 pb-2">
        G-Code:
      </div>
      <move-row
        v-for="(move,i) in moves"
        :key="i"
        :i="i"
        :move="move"
        :set-move="setMove"
        :add-row-above="addRowAbove"
      />
      <v-col
        cols="12"
        class="pt-4"
      >
        <app-btn
          :elevation="2"
          block
          @click="setMove('',moves.length)"
        >
          Add row
        </app-btn>
      </v-col>
    </v-col>
  </v-card-text>
</template>

<script lang="ts">
import { Component, Mixins, Prop } from 'vue-property-decorator'
import StateMixin from '@/mixins/state'
import MoveRow from '@/components/widgets/plan-moves/MoveRow.vue'
import ToolheadMixin from '@/mixins/toolhead'
import FilesMixin from '@/mixins/files'
import { AxiosError } from 'axios'

@Component({
  components: {
    MoveRow
  }
})
export default class PlanMoves extends Mixins(StateMixin, ToolheadMixin, FilesMixin) {
  @Prop({ type: Boolean, default: false })
  public forceMove!: boolean

  get moves ():Array<string|undefined> {
    return this.$store.getters['config/getPlannerTmp']
  }

  set moves (m) {
    this.$store.dispatch('config/updatePlannerTmp', m)
  }

  setMove (move:string|undefined, i:number) {
    const tmp = this.moves
    tmp[i] = move
    this.moves = tmp
    this.updateGcodeList++
  }

  addRowAbove (i:number) {
    const tmp = this.moves
    tmp.splice(i, 0, '')
    this.moves = tmp
    this.updateGcodeList++
  }

  get filename () {
    return this.$store.getters['config/getPlannerFile']
  }

  set filename (f) {
    this.$store.dispatch('config/updatePlannerFile', f)
  }

  updateGcodeList = 0

  loadFile (filename: string) {
    (async () => {
      try {
        if (filename === '') {
          this.moves = []
          return
        }
        const file = await this.getFile(filename, 'gcodes', undefined, undefined, true)
        this.moves = (file.data as string).split('\n')
      } catch (e) {
        let msg:any = ''
        if (e instanceof AxiosError) {
          if (e.response?.status === 404) {
            msg = 'Couldn\'t find file'
          } else {
            msg = `Getting file failed: ${e.response?.statusText}`
          }
        } else {
          msg = e
        }

        this.$store.dispatch('notifications/pushNotification', {
          id: 'updates-available',
          title: msg, // i18n.t('app.version.label.updates_available'),
          type: 'error',
          merge: true,
          snackbar: true
        })
      }
    })()
  }

  saveFile (filename: string) {
    (async () => {
      const file = new File([this.moves.join('\n')], filename)
      await this.uploadFile(file, '', 'gcodes', false, undefined, false)
    })()
  }
}
</script>
