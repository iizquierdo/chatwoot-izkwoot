<script setup>
import { computed } from 'vue';
import { useRoute } from 'vue-router';
import { useStore } from 'dashboard/composables/store';
import { useBranding } from 'shared/composables/useBranding';
import SettingsLayout from '../SettingsLayout.vue';
import BaseSettingsHeader from '../components/BaseSettingsHeader.vue';
import KnowledgeFrame from '../../knowledge/Index.vue';

const route = useRoute();
const store = useStore();
const { replaceInstallationName } = useBranding();

const view = computed(() => (route.query.view === 'bots' ? 'bots' : 'bases'));
const integration = computed(
  () => store.getters['integrations/getIntegration']('izkwoot') || {}
);
const description = computed(() =>
  replaceInstallationName(integration.value.description || '')
);

store.dispatch('integrations/get');

const tabClass = active =>
  [
    'rounded-lg px-3 py-1.5 text-sm font-medium no-underline',
    active
      ? 'bg-n-solid-1 text-n-slate-12 shadow-sm'
      : 'text-n-slate-11 hover:text-n-slate-12',
  ].join(' ');
</script>

<template>
  <SettingsLayout>
    <template #header>
      <BaseSettingsHeader
        :title="integration.name || 'izkwoot'"
        :description="description"
        :back-button-label="$t('INTEGRATION_SETTINGS.HEADER')"
      >
        <template #tabs>
          <div class="flex items-center gap-1 rounded-xl bg-n-alpha-2 p-1">
            <router-link
              :to="{
                name: 'settings_integrations_izkwoot',
                params: { accountId: route.params.accountId },
                query: { view: 'bases' },
              }"
              :class="tabClass(view === 'bases')"
            >
              Conocimiento
            </router-link>
            <router-link
              :to="{
                name: 'settings_integrations_izkwoot',
                params: { accountId: route.params.accountId },
                query: { view: 'bots' },
              }"
              :class="tabClass(view === 'bots')"
            >
              Bots
            </router-link>
          </div>
        </template>
      </BaseSettingsHeader>
    </template>
    <template #body>
      <KnowledgeFrame embedded :view="view" />
    </template>
  </SettingsLayout>
</template>
