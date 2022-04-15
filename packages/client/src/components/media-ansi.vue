<template>
<div v-if="hide" class="media-ansi-hidden" @click="hide = false">
  <div class="text">
    <b><i class="fas fa-exclamation-triangle"></i> {{ $ts.sensitive }}</b>
    <span>{{ $ts.clickToShow }}</span>
  </div>
</div>
<div v-else class="media-ansi">
  <a
    ref="imageContainer"
    :href="ansiFile.url"
    :title="ansiFile.name"
  ></a>
  <button v-tooltip="$ts.hide" class="_button hide" @click="hide = true"><i class="fas fa-eye-slash"></i></button>
</div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import { escapes } from '@/scripts/escapes'

export default defineComponent({
  props: {
    ansiFile: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      hide: false
    }
  },
  mounted () {
    const self = this;
    escapes(this.ansiFile.url + "?.ans", function () {
      this.alt = self.ansiFile.comment;
      this.title = self.ansiFile.comment;
      this.classList.add("render");
      self.$refs.imageContainer.appendChild(this);
    });
  }
});
</script>

<style lang="scss">
.media-ansi {
  a {
    position: relative;
    width: 100%;
    height: 100%;
  }

  .render {
    display: block;
    width: 100%;
    height: 100%;
    object-fit: contain;
  }
}
</style>
