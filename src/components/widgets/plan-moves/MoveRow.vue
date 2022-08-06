<template>
  <v-row
    :align="'center'"
  >
    <v-col
      cols="7"
    >
      <v-text-field
        v-model="move"
        hide-details
        outlined
        dense
        :label="i.toString()"
      />
    </v-col>
    <v-col cols="2">
      <app-btn
        :disabled="move==''"
        block
        @click="sendGcode(move)"
      >
        Run
      </app-btn>
    </v-col>
    <v-col cols="3">
      <app-btn
        :disabled="!klippyReady || !allHomed || printerPrinting"
        block
        @click="handleGetPos()"
      >
        Current Pos.
      </app-btn>
    </v-col>
  </v-row>
</template>

<script lang="ts">
import { Component, Mixins, Prop } from 'vue-property-decorator'
import ToolheadMoves from '@/components/widgets/toolhead/ToolheadMoves.vue'
import ToolheadPosition from '@/components/widgets/toolhead/ToolheadPosition.vue'
import StateMixin from '@/mixins/state'
import ToolheadMixin from '@/mixins/toolhead'

@Component({
  components: {
    ToolheadMoves,
    ToolheadPosition
  }
})
export default class MoveRow extends Mixins(StateMixin, ToolheadMixin) {
  @Prop({ type: String })
  public move!: string

  @Prop({ type: Number })
  public i!: number

  @Prop()
  public setMove!: (gcode: string, i:number)=>void

  get gcodePosition () {
    return this.$store.state.printer.printer.gcode_move.gcode_position
  }

  handleGetPos () {
    this.setMove(`G1 X${this.gcodePosition[0]} Y${this.gcodePosition[1]} Z${this.gcodePosition[2]} F6000`, this.i)
  }
}

</script>
<style type="scss" scoped>
  /* .move-row{
    padding:8px;
  } */
</style>
