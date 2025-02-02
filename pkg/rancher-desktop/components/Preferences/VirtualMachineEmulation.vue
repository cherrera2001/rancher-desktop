<script lang="ts">
import os from 'os';

import { Checkbox, RadioButton, RadioGroup } from '@rancher/components';
import Vue from 'vue';

import LabeledBadge from '@pkg/components/form/LabeledBadge.vue';
import RdFieldset from '@pkg/components/form/RdFieldset.vue';
import { Settings, VMType } from '@pkg/config/settings';
import { RecursiveTypes } from '@pkg/utils/typeUtils';

import type { PropType } from 'vue';

export default Vue.extend({
  name:       'preferences-virtual-machine-emulation',
  components: {
    Checkbox,
    LabeledBadge,
    RadioGroup,
    RdFieldset,
    RadioButton,
  },
  props: {
    preferences: {
      type:     Object as PropType<Settings>,
      required: true,
    },
  },
  computed: {
    options(): { label: string, value: VMType, description: string, experimental: boolean, disabled: boolean }[] {
      const defaultOption = VMType.QEMU;

      return Object.values(VMType)
        .map((x) => {
          return {
            label:        this.t(`virtualMachine.type.options.${ x }.label`),
            value:        x,
            description:  this.t(`virtualMachine.type.options.${ x }.description`, {}, true),
            experimental: x !== defaultOption, // Mark experimental option
            disabled:     x === VMType.VZ && this.vzDisabled,
          };
        });
    },
    groupName(): string {
      return 'vmType';
    },
    vZSelected(): boolean {
      return this.preferences.experimental.virtualMachine.type === VMType.VZ;
    },
    vzDisabled(): boolean {
      return parseInt(os.release()) < 22;
    },
    rosettaDisabled(): boolean {
      return os.arch() !== 'arm64';
    },
  },
  methods: {
    onChange<P extends keyof RecursiveTypes<Settings>>(property: P, value: RecursiveTypes<Settings>[P]) {
      this.$store.dispatch('preferences/updatePreferencesData', { property, value });
    },
    disabledVmTypeTooltip(disabled: boolean): { content: string } | {} {
      let tooltip = {};

      if (disabled) {
        tooltip = { content: this.t('prefs.onlyFromVentura') };
      }

      return tooltip;
    },
  },
});
</script>

<template>
  <div class="virtual-machine-emulation">
    <div class="row">
      <div class="col span-6">
        <rd-fieldset
          data-test="vmType"
          :legend-text="t('virtualMachine.type.legend')"
        >
          <radio-group
            :options="options"
            :name="groupName"
          >
            <template
              v-for="(option, index) in options"
              #[index]
            >
              <radio-button
                :key="groupName+'-'+index"
                :name="groupName"
                :value="preferences.experimental.virtualMachine.type"
                :label="option.label"
                :val="option.value"
                :description="option.description"
                :disabled="option.disabled"
                @input="onChange('experimental.virtualMachine.type', $event)"
              >
                <template #label>
                  <div
                    v-tooltip="disabledVmTypeTooltip(option.disabled)"
                  >
                    {{ option.label }}
                    <labeled-badge
                      v-if="option.experimental"
                      :text="t('prefs.experimental')"
                    />
                  </div>
                </template>
              </radio-button>
            </template>
          </radio-group>
        </rd-fieldset>
      </div>
      <div
        v-if="vZSelected && !rosettaDisabled"
        class="col span-6 vz-sub-options"
      >
        <rd-fieldset
          data-test="useRosetta"
          :legend-text="t('virtualMachine.useRosetta.legend')"
        >
          <checkbox
            :label="t('virtualMachine.useRosetta.label')"
            :value="preferences.experimental.virtualMachine.useRosetta"
            @input="onChange('experimental.virtualMachine.useRosetta', $event)"
          />
        </rd-fieldset>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .vz-sub-options {
    border-left: 1px solid var(--border);
    padding-left: 1rem;
    display: flex;
    flex-direction: column;
  }
</style>
