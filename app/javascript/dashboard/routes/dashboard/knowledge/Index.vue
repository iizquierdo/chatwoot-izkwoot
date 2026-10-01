<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue';
import { useRoute } from 'vue-router';
import { useMapGetter } from 'dashboard/composables/store';

const props = defineProps({
  view: { type: String, default: 'bases' },
  embedded: { type: Boolean, default: false },
});
const frame = ref(null);
const route = useRoute();
const user = useMapGetter('getCurrentUser');
const src = window.chatwootConfig?.guilleKnowledgeUrl || 'http://localhost:3100/';
const frameSrc = computed(() => {
  const url = new URL(src, window.location.origin);
  url.searchParams.set('view', props.view === 'bots' ? 'bots' : 'bases');
  return url.toString();
});
const targetOrigin = computed(() => {
  try {
    return new URL(src).origin;
  } catch {
    return 'http://localhost:3100';
  }
});

const sendToken = () => {
  const token = user.value?.access_token;
  if (!token || !frame.value?.contentWindow) return;
  frame.value.contentWindow.postMessage(
    { type: 'guille-auth', token, accountId: Number(route.params.accountId) },
    targetOrigin.value
  );
};

const onMessage = event => {
  if (event.origin !== targetOrigin.value) return;
  if (event.data !== 'guille-ready') return;
  sendToken();
};

onMounted(() => window.addEventListener('message', onMessage));
onUnmounted(() => window.removeEventListener('message', onMessage));
</script>

<template>
  <iframe
    :key="props.view"
    ref="frame"
    :src="frameSrc"
    :class="
      props.embedded
        ? 'h-[calc(100vh-16rem)] min-h-[36rem] w-full rounded-xl border-0 bg-n-background'
        : 'h-full min-h-[calc(100vh-4rem)] w-full border-0 bg-n-background'
    "
    :title="props.view === 'bots' ? 'Bots' : 'Conocimiento'"
    @load="sendToken"
  />
</template>
