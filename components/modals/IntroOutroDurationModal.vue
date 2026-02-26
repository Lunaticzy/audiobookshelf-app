<template>
  <modals-modal v-model="show" :width="320" height="100%">
    <template #outer>
      <div class="absolute top-8 left-4 z-40">
        <p class="text-white text-2xl truncate">{{ $strings.HeaderIntroOutroDuration }}</p>
      </div>
    </template>

    <div class="w-full h-full overflow-hidden absolute top-0 left-0 flex items-center justify-center" @click="show = false">
      <div class="w-full overflow-x-hidden overflow-y-auto bg-primary rounded-lg border border-border p-5" style="max-height: 75%" @click.stop>
        <!-- Intro -->
        <div class="mb-5">
          <p class="text-sm text-fg-muted tracking-wider mb-2">{{ $strings.LabelSkipChapterIntro }}</p>
          <div class="flex items-center">
            <ui-toggle-switch v-model="skipIntro" class="flex-shrink-0" />
            <input
              type="range"
              v-model.number="introDuration"
              min="0"
              max="60"
              step="1"
              class="duration-slider flex-1 mx-3"
            />
            <div class="flex items-center flex-shrink-0">
              <input
                type="number"
                v-model.number="introDuration"
                min="0"
                max="60"
                class="w-14 text-center bg-bg text-fg border border-border rounded px-1 py-0.5 text-sm"
                @change="clampIntro"
              />
              <span class="text-fg-muted text-sm ml-1">{{ $strings.UnitSecondsShort.replace('{0}', '').trim() || 's' }}</span>
            </div>
          </div>
        </div>

        <!-- Outro -->
        <div class="mb-6">
          <p class="text-sm text-fg-muted tracking-wider mb-2">{{ $strings.LabelSkipChapterOutro }}</p>
          <div class="flex items-center">
            <ui-toggle-switch v-model="skipOutro" class="flex-shrink-0" />
            <input
              type="range"
              v-model.number="outroDuration"
              min="0"
              max="60"
              step="1"
              class="duration-slider flex-1 mx-3"
            />
            <div class="flex items-center flex-shrink-0">
              <input
                type="number"
                v-model.number="outroDuration"
                min="0"
                max="60"
                class="w-14 text-center bg-bg text-fg border border-border rounded px-1 py-0.5 text-sm"
                @change="clampOutro"
              />
              <span class="text-fg-muted text-sm ml-1">{{ $strings.UnitSecondsShort.replace('{0}', '').trim() || 's' }}</span>
            </div>
          </div>
        </div>

        <!-- Buttons -->
        <div class="flex gap-3">
          <ui-btn class="flex-1" small @click="show = false">{{ $strings.ButtonCancel }}</ui-btn>
          <ui-btn class="flex-1" color="success" small @click="save">{{ $strings.ButtonOk }}</ui-btn>
        </div>
      </div>
    </div>
  </modals-modal>
</template>

<script>
export default {
  props: {
    value: Boolean
  },
  data() {
    return {
      skipIntro: false,
      skipOutro: false,
      introDuration: 10,
      outroDuration: 10
    }
  },
  watch: {
    async show(newVal) {
      if (newVal) {
        const ps = await this.$localStore.getPlayerSettings()
        this.skipIntro = !!ps?.skipIntro
        this.skipOutro = !!ps?.skipOutro
        this.introDuration = ps?.globalIntroDuration ?? 10
        this.outroDuration = ps?.globalOutroDuration ?? 10
      }
    }
  },
  computed: {
    show: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit('input', val)
      }
    }
  },
  methods: {
    clampIntro() {
      this.introDuration = Math.min(60, Math.max(0, Math.round(this.introDuration) || 0))
    },
    clampOutro() {
      this.outroDuration = Math.min(60, Math.max(0, Math.round(this.outroDuration) || 0))
    },
    async save() {
      this.clampIntro()
      this.clampOutro()
      const current = (await this.$localStore.getPlayerSettings()) || {}
      await this.$localStore.setPlayerSettings({
        ...current,
        skipIntro: this.skipIntro,
        skipOutro: this.skipOutro,
        globalIntroDuration: this.introDuration,
        globalOutroDuration: this.outroDuration
      })
      this.$eventBus.$emit('player-settings', {
        skipIntro: this.skipIntro,
        skipOutro: this.skipOutro,
        globalIntroDuration: this.introDuration,
        globalOutroDuration: this.outroDuration
      })
      this.show = false
    }
  }
}
</script>

<style scoped>
.duration-slider {
  -webkit-appearance: none;
  appearance: none;
  background: transparent;
  cursor: pointer;
  width: 100%;
}
.duration-slider:focus {
  outline: none;
}
.duration-slider::-webkit-slider-runnable-track {
  background-color: rgb(var(--color-track) / 0.5);
  border-radius: 9999px;
  height: 0.5rem;
}
.duration-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  margin-top: -0.25rem;
  border-radius: 9999px;
  background-color: rgb(var(--color-track-cursor));
  height: 1rem;
  width: 1rem;
}
.duration-slider::-moz-range-track {
  background-color: rgb(var(--color-track) / 0.5);
  border-radius: 9999px;
  height: 0.5rem;
}
.duration-slider::-moz-range-thumb {
  border: none;
  border-radius: 9999px;
  background-color: rgb(var(--color-track-cursor));
  height: 1rem;
  width: 1rem;
}
input[type='number']::-webkit-inner-spin-button,
input[type='number']::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
input[type='number'] {
  -moz-appearance: textfield;
}
</style>
