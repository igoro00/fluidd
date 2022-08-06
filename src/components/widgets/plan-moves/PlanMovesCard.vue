<template>
  <collapsable-card
    :title="$t('app.general.title.plan_moves')"
    icon="$limits"
    :draggable="true"
    layout-path="dashboard.plan-moves-card"
  >
    <plan-moves />
  </collapsable-card>
</template>

<script lang="ts">
import { Component, Mixins, Prop } from 'vue-property-decorator'
import PlanMoves from '@/components/widgets/plan-moves/PlanMoves.vue'
import StateMixin from '@/mixins/state'

@Component({
  components: {
    PlanMoves
  }
})
export default class PlanMovesCard extends Mixins(StateMixin) {
  @Prop({ type: Boolean, default: true })
  public enabled!: boolean

  get inLayout (): boolean {
    return (this.$store.state.config.layoutMode)
  }
}
</script>
