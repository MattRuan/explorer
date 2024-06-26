<script lang="ts" setup>
import {
  useBlockchain,
  useFormatter,
  useStakingStore,
  useTxDialog,
} from '@/stores';
import { select } from '@/components/dynamic/index';
import type { PaginatedProposals } from '@/types';
import ProposalProcess from './ProposalProcess.vue';
import type { PropType } from 'vue';
import { computed, ref } from 'vue';
const dialog = useTxDialog();
defineProps({
  proposals: { type: Object as PropType<PaginatedProposals> },
});

const format = useFormatter();
const staking = useStakingStore();
const chain = useBlockchain();
function showType(v: string) {
  if (v) {
    return v.substring(v.lastIndexOf('.') + 1);
  }
  return v;
}

const statusMap: Record<string, string> = {
  PROPOSAL_STATUS_VOTING_PERIOD: 'VOTING',
  PROPOSAL_STATUS_PASSED: 'PASSED',
  PROPOSAL_STATUS_REJECTED: 'REJECTED',
};
const voterStatusMap: Record<string, string> = {
  VOTE_OPTION_NO_WITH_VETO: '',
  VOTE_OPTION_YES: 'success',
  VOTE_OPTION_NO: 'error',
  VOTE_OPTION_ABSTAIN: 'warning',
};

const proposalInfo = ref();

function metaItem(metadata: string|undefined): { title: string; summary: string } {
  return metadata ? JSON.parse(metadata) : {}
}

</script>
<template>
  <div class="text-sm">
    <table class="table-compact w-full table-fixed hidden lg:!table">
      <tbody>
        <tr v-for="(item, index) in proposals?.proposals" :key="index" class="border-b !border-white-10">
          <td class="px-4 w-20">
            <label
              for="proposal-detail-modal"
              class="text-main text-base hover:text-primary cursor-pointer"
              @click="proposalInfo = item"
            >
              #{{ item?.proposal_id }}</label
            >
          </td>
          <td class="w-full">
            <div>
              <RouterLink
                :to="`/${chain.chainName}/gov/${item?.proposal_id}`"
                class="text-primary text-base mb-1 block hover:text-primary truncate"
              >
                {{ item?.content?.title || item?.title || metaItem(item?.metadata)?.title }}
              </RouterLink>
              <div
                v-if="item.content"
                class="mr-2 mb-2 text-xs text-main bg-white-10 px-3 rounded-full py-1 w-fit"
              >
                {{ showType(item.content['@type']) }} 
              </div>
            </div>
          </td>
          <td class="w-60">
            <ProposalProcess
              :pool="staking.pool"
              :tally="item.final_tally_result"
            ></ProposalProcess>
          </td>
          <td class="w-36">
            <div class="pl-4">
              <div
                class="flex items-center"
                :class="
                  statusMap?.[item?.status] === 'PASSED'
                    ? 'text-yes'
                    : statusMap?.[item?.status] === 'REJECTED'
                    ? 'text-no'
                    : 'text-info'
                "
              >
                <div
                  class="w-1 h-1 rounded-full mr-2"
                  :class="
                    statusMap?.[item?.status] === 'PASSED'
                      ? 'bg-yes'
                      : statusMap?.[item?.status] === 'REJECTED'
                      ? 'bg-no'
                      : 'bg-info'
                  "
                ></div>
                <div class="text-xs">
                  {{ statusMap?.[item?.status] || item?.status }}
                </div>
              </div>
              <div
                class="truncate col-span-2 md:!col-span-1 text-xs text-gray-3 text-right md:!flex md:!justify-start"
              >
                {{ format.toDay(item.voting_end_time, 'from') }}
              </div>
            </div>
          </td>

          <td v-if="statusMap?.[item?.status] === 'VOTING'" class="w-40">
            <div class="">
              <label
                for="vote"
                class="btn btn-xs btn-mw-primary rounded-sm"
                @click="
                  dialog.open('vote', {
                    proposal_id: item?.proposal_id,
                  })
                "
              >
                <span v-if="item?.voterStatus !== 'VOTE_OPTION_NO_WITH_VETO'">{{
                  item?.voterStatus?.replace('VOTE_OPTION_', '')
                }}</span>

                <span v-else>Vote</span>
              </label>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="lg:!hidden">
      <div
        v-for="(item, index) in proposals?.proposals"
        :key="index"
        class="px-4 py-4"
      >
        <div
          class="text-main text-base mb-1 flex justify-between hover:text-primary"
        >
          <RouterLink
            :to="`/${chain.chainName}/gov/${item?.proposal_id}`"
            class="flex-1 w-0 truncate mr-4"
            >{{ item?.content?.title || item?.title || metaItem(item?.metadata)?.title }}</RouterLink
          >
          <label
            for="proposal-detail-modal"
            class="text-main text-base hover:text-primary cursor-pointer"
            @click="proposalInfo = item"
          >
            #{{ item?.proposal_id }}</label
          >
        </div>

        <div class="grid grid-cols-4 mt-2 mb-2 flex justify-between">
          <div class="col-span-2">
            <div
              v-if="item.content"
              class="mr-1 md:!mr-2 mb-1 md:!mb-2 text-xs text-main bg-white-10 px-3 rounded-full py-1 w-fit"
            >
              {{ showType(item.content['@type']) }}
            </div>
          </div>

          <div
            class="text-xs text-gray-3 text-right flex items-center justify-end"
          >
            {{ format.toDay(item.voting_end_time, 'from') }}
          </div>
        </div>

        <div>
          <ProposalProcess
            :pool="staking.pool"
            :tally="item.final_tally_result"
          ></ProposalProcess>
        </div>

        <div class="mt-4" v-if="statusMap?.[item?.status] === 'VOTING'">
          <div class="flex justify-between">
            <div
              class="flex items-center"
              :class="
                statusMap?.[item?.status] === 'PASSED'
                  ? 'text-yes'
                  : statusMap?.[item?.status] === 'REJECTED'
                  ? 'text-no'
                  : 'text-info'
              "
            >
              <div
                class="w-1 h-1 rounded-full mr-2"
                :class="
                  statusMap?.[item?.status] === 'PASSED'
                    ? 'bg-yes'
                    : statusMap?.[item?.status] === 'REJECTED'
                    ? 'bg-no'
                    : 'bg-info'
                "
              ></div>
              <div class="text-xs flex items-center">
                {{ statusMap?.[item?.status] || item?.status }}
              </div>
            </div>
            <label
              for="vote"
              class="btn btn-xs btn-mw-primary rounded-sm"
              @click="
                dialog.open('vote', {
                  proposal_id: item?.proposal_id,
                })
              "
            >
              <span v-if="item?.voterStatus !== 'VOTE_OPTION_NO_WITH_VETO'">{{
                item?.voterStatus?.replace('VOTE_OPTION_', '')
              }}</span>

              <span v-else>Vote</span></label
            >
           
          </div>
        </div>
      </div>
    </div>

    <input type="checkbox" id="proposal-detail-modal" class="modal-toggle" />
    <label for="proposal-detail-modal" class="modal">
      <label class="modal-box !w-11/12 !max-w-5xl" for="">
        <label
          for="proposal-detail-modal"
          class="btn btn-sm btn-circle absolute right-2 top-2"
          >✕</label
        >
        <h3 class="font-bold text-lg">Description</h3>
        <p class="py-4">
          <Component
            v-if="proposalInfo?.content?.description || proposalInfo?.summary || metaItem(proposalInfo?.metadata)?.summary"
            :is="select(proposalInfo?.content?.description || proposalInfo?.summary || metaItem(proposalInfo?.metadata)?.summary, 'horizontal')"
            :value="proposalInfo?.content?.description || proposalInfo?.summary || metaItem(proposalInfo?.metadata)?.summary"
          >
          </Component>
        </p>
      </label>
    </label>
  </div>
</template>
