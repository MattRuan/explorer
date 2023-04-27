<script lang="ts" setup>
import MdEditor from 'md-editor-v3';
import PriceMarketChart from '@/components/charts/PriceMarketChart.vue';

import { useBlockchain, useFormatter } from '@/stores';
import { onMounted, ref } from 'vue';
import { useIndexModule } from './indexStore';
import { computed } from '@vue/reactivity';

import CardStatisticsVertical from '@/components/CardStatisticsVertical.vue';
import ProposalListItem from '@/components/ProposalListItem.vue';

const blockchain = useBlockchain();
const store = useIndexModule();

const coinInfo = computed(() => {
  return store.coinInfo;
});

onMounted(() => {
  store.loadDashboard();
});

const format = useFormatter();
const ticker = computed(() => store.coinInfo.tickers[store.tickerIndex]);

blockchain.$subscribe((m, s) => {
  if (!Array.isArray(m.events) && ['chainName', 'endpoint'].includes(m.events.key)) {
    store.loadDashboard();
  }
});
function shortName(name: string, id: string) {
  return name.toLowerCase().startsWith('ibc/') || name.toLowerCase().startsWith('0x') ? id : name;
}
</script>

<template>
  <div>
    <VCard v-if="coinInfo && coinInfo.name" class="mb-5">
      <VRow>
        <VCol md="5">
          <VCardItem>
            <VCardTitle>
              {{ coinInfo.name }} (
              <span class="text-uppercase">{{ coinInfo.symbol }}</span>
              )
            </VCardTitle>
            <VCardSubtitle>
              Rank:
              <VChip color="error" size="x-small">#{{ coinInfo.market_cap_rank }}</VChip>
            </VCardSubtitle>
          </VCardItem>
          <VDivider />
          <VCardItem>
            <VBtn variant="text" size="small" :href="store.homepage" prependIcon="mdi-web">
              Website
            </VBtn>
            <VBtn variant="text" size="small" :href="store.twitter" prependIcon="mdi-twitter">
              Twitter
            </VBtn>
            <VBtn variant="text" size="small" :href="store.telegram" prependIcon="mdi-telegram">
              Telegram
            </VBtn>
            <VBtn variant="text" size="small" :href="store.github" prependIcon="mdi-github">
              Github
            </VBtn>
          </VCardItem>
          <VCardItem>
            <!-- SECTION upgrade plan banner -->
            <div class="plan-upgrade-banner d-flex bg-light-secondary rounded align-center pa-3">
              <h3 class="plan-details me-3" :class="store.priceColor">
                {{ store.priceChange }}
                <small>%</small>
              </h3>

              <VMenu open-on-hover>
                <template #activator="{ props }">
                  <div class="d-flex flex-column align-start" v-bind="props">
                    <h3 class="text-base font-weight-semibold">
                      {{ ticker?.market?.name || '' }}
                    </h3>
                    <span class="text-primary text-xs">
                      {{ shortName(ticker?.base, ticker.coin_id) }}/{{
                        shortName(ticker?.target, ticker.target_coin_id)
                      }}
                    </span>
                  </div>
                </template>
                <VList style="max-height: 300px">
                  <VListItem
                    v-for="(item, i) in store.coinInfo.tickers"
                    :key="i"
                    rounded
                    @click="store.selectTicker(i)"
                  >
                    <template #prepend></template>
                    <!-- eslint-disable-next-line vue/no-v-text-v-html-on-component -->
                    <VListItemTitle v-text="item.market.name" />
                    <VListItemSubtitle>
                      {{ shortName(item?.base, item.coin_id) }}/{{
                        shortName(item?.target, item.target_coin_id)
                      }}
                    </VListItemSubtitle>
                    <template #append>
                      <span class="ml-3" :class="`text-${store.tickerColor(item.trust_score)}`">
                        {{ item.converted_last.usd }}
                      </span>
                    </template>
                  </VListItem>
                </VList>
              </VMenu>

              <VSpacer />

              <div class="d-flex align-center">
                <sub>
                  <h6 class="text-xs font-weight-regular">$</h6>
                </sub>
                <span class="text-h5">{{ ticker.converted_last.usd }}</span>
              </div>
            </div>
            <!-- !SECTION -->
            <VSpacer />
            <VBtn block :color="store.trustColor" class="mt-3" :href="ticker.trade_url">
              Buy {{ coinInfo.symbol || '' }}
            </VBtn>
          </VCardItem>
        </VCol>
        <VCol md="7">
          <VCardItem>
            <PriceMarketChart />
          </VCardItem>
        </VCol>
      </VRow>
      <VDivider />
      <VCardText style="max-height: 250px; overflow: auto">
        <MdEditor :model-value="coinInfo.description?.en" previewOnly></MdEditor>
      </VCardText>
      <VCardItem>
        <VChip v-for="tag in coinInfo.categories" size="x-small" class="mr-2">{{ tag }}</VChip>
      </VCardItem>
    </VCard>

    <div class="grid grid-cols-2 gap-4 md:grid-cols-3 lg:grid-cols-6">
      <div v-for="item in store.stats">
        <CardStatisticsVertical v-bind="item" />
      </div>
    </div>

    <VCard class="my-5">
      <VCardItem class="pb-0">
        <VCardTitle>Active Proposals</VCardTitle>
      </VCardItem>
      <VCardItem>
        <ProposalListItem :proposals="store?.proposals" />
      </VCardItem>
      <VCardText v-if="store.proposals.length === 0">No active proposals</VCardText>
    </VCard>

    <VBtn block color="secondary" variant="outlined" class="mt-5">Connect Wallet</VBtn>
  </div>
</template>

<style lang="scss" scoped>
.card-box {
  border: 1px solid rgb(var(--v-theme-primary));
}
</style>

<route>
  {
    meta: {
      i18n: 'dashboard'
    }
  }
</route>