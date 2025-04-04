<template>
  <q-card
    flat
    bordered
    :class="['panel-service-details', {'panel-service-details-dense':isDense}]"
  >
    <q-scroll-area
      :thumb-style="appThumbStyle"
      style="height:100%;"
    >
      <q-card-section>
        <div class="row items-start no-wrap">
          <div
            v-if="data.type"
            class="col"
          >
            <div class="text-subtitle2">
              TYPE
            </div>
            <q-chip
              dense
              class="app-chip app-chip-entry-points"
            >
              {{ data.type }}
            </q-chip>
          </div>
          <div class="col">
            <div class="text-subtitle2">
              PROVIDER
            </div>
            <div class="block-right-text">
              <q-avatar class="provider-logo">
                <q-icon :name="`img:${getProviderLogoPath}`" />
              </q-avatar>
              <div class="block-right-text-label">
                {{ data.provider }}
              </div>
            </div>
          </div>
        </div>
      </q-card-section>
      <q-card-section>
        <div class="row items-start no-wrap">
          <div class="col">
            <div class="text-subtitle2">
              STATUS
            </div>
            <div class="block-right-text">
              <avatar-state :state="status(data.status)" />
              <div :class="['block-right-text-label', `block-right-text-label-${data.status}`]">
                {{ statusLabel(data.status) }}
              </div>
            </div>
          </div>
        </div>
      </q-card-section>
      <q-card-section v-if="data.mirroring">
        <div class="row items-start no-wrap">
          <div class="col">
            <div class="text-subtitle2">
              Main Service
            </div>
            <q-chip
              dense
              class="app-chip app-chip-name app-chip-overflow"
            >
              {{ data.mirroring.service }}
              <q-tooltip>{{ data.mirroring.service }}</q-tooltip>
            </q-chip>
          </div>
        </div>
      </q-card-section>
      <q-card-section v-if="data.loadBalancer && $route.meta.protocol !== 'tcp'">
        <div class="row items-start no-wrap">
          <div class="col">
            <div class="text-subtitle2">
              Pass Host Header
            </div>
            <boolean-state :value="data.loadBalancer.passHostHeader" />
          </div>
        </div>
      </q-card-section>

      <q-card-section v-if="data.loadBalancer && data.loadBalancer.proxyProtocol">
        <div class="row items-start no-wrap">
          <div class="col">
            <div class="text-subtitle2">
              Proxy Protocol
            </div>
            <q-chip
              dense
              class="app-chip app-chip-name"
            >
              Version {{ data.loadBalancer.proxyProtocol.version }}
            </q-chip>
          </div>
        </div>
      </q-card-section>

      <q-card-section v-if="data.failover && data.failover.service">
        <div class="row items-start no-wrap">
          <div class="col">
            <div class="text-subtitle2">
              Main Service
            </div>
            <q-chip
              dense
              class="app-chip app-chip-name app-chip-overflow"
            >
              {{ data.failover.service }}
              <q-tooltip>{{ data.failover.service }}</q-tooltip>
            </q-chip>
          </div>
        </div>
      </q-card-section>

      <q-card-section v-if="data.failover && data.failover.fallback">
        <div class="row items-start no-wrap">
          <div class="col">
            <div class="text-subtitle2">
              Fallback Service
            </div>
            <q-chip
              dense
              class="app-chip app-chip-name app-chip-overflow"
            >
              {{ data.failover.fallback }}
              <q-tooltip>{{ data.failover.fallback }}</q-tooltip>
            </q-chip>
          </div>
        </div>
      </q-card-section>

      <q-separator v-if="sticky" />
      <StickyServiceDetails
        v-if="sticky"
        :sticky="sticky"
        :dense="dense"
      />
    </q-scroll-area>
  </q-card>
</template>

<script>
import { defineComponent } from 'vue'
import AvatarState from './AvatarState.vue'
import BooleanState from './BooleanState.vue'
import StickyServiceDetails from './StickyServiceDetails.vue'

export default defineComponent({
  name: 'PanelServiceDetails',
  components: {
    BooleanState,
    AvatarState,
    StickyServiceDetails
  },
  props: {
    data: { type: Object, default: undefined, required: false },
    dense: { type: Boolean, default: undefined }
  },
  computed: {
    isDense () {
      return this.dense !== undefined
    },
    sticky () {
      if (this.data.weighted && this.data.weighted.sticky) {
        return this.data.weighted.sticky
      }

      if (this.data.loadBalancer && this.data.loadBalancer.sticky) {
        return this.data.loadBalancer.sticky
      }

      return null
    },
    getProviderLogoPath () {
      const name = this.data.provider.toLowerCase()

      if (name.startsWith('plugin-')) {
        return 'providers/plugin.svg'
      }
      if (name.startsWith('consul-')) {
        return 'providers/consul.svg'
      }
      if (name.startsWith('consulcatalog-')) {
        return 'providers/consulcatalog.svg'
      }
      if (name.startsWith('nomad-')) {
        return 'providers/nomad.svg'
      }

      return `providers/${name}.svg`
    }
  },
  methods: {
    status (value) {
      if (value === 'enabled') {
        return 'positive'
      }
      if (value === 'disabled') {
        return 'negative'
      }
      return value || 'negative'
    },
    statusLabel (value) {
      if (value === 'enabled') {
        return 'success'
      }
      if (value === 'disabled') {
        return 'error'
      }
      return value || 'error'
    }
  }
})
</script>

<style scoped lang="scss">
  @import "../../css/sass/variables";

  .panel-service-details {
    height: 600px;
    &-dense{
      height: 400px;
    }
    .q-card__section {
      padding: 24px;
      + .q-card__section {
        padding-top: 0;
      }
    }

    .block-right-text{
      height: 32px;
      line-height: 32px;
      .q-avatar{
        float: left;
      }
      &-label{
        font-size: 14px;
        font-weight: 600;
        color: $app-text-grey;
        float: left;
        margin-left: 10px;
        text-transform: capitalize;
        &-enabled {
          color: $positive;
        }
        &-disabled {
          color: $negative;
        }
        &-warning {
          color: $warning;
        }
      }
    }

    .text-subtitle2 {
      font-size: 11px;
      color: $app-text-grey;
      line-height: 16px;
      margin-bottom: 4px;
      text-align: left;
      letter-spacing: 2px;
      font-weight: 600;
      text-transform: uppercase;
    }

    .provider-logo {
      width: 32px;
      height: 32px;
      img {
        width: 100%;
        height: 100%;
      }
    }
  }

</style>
