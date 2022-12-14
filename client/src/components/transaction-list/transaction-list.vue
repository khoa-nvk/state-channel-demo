<script setup lang="ts">
import { ref, onUpdated } from 'vue';
import { storeToRefs } from 'pinia';
import { useTransactionsStore } from '../../stores/transactions';
import SingleTransaction from '../transaction/transaction.vue';
import Accordion from '../accordion/accordion.vue';
import Notification from '../notification/notification.vue';
import { useIsOnMobileStore } from '../../stores/is-on-mobile';

const EXPAND_ICON = new URL('../../assets/svg/expand.svg', import.meta.url)
  .href;
const MINIMISE_ICON = new URL(
  '../../assets/images/minimize.png',
  import.meta.url
).href;

const terminalEl = ref<Element>();
const transactionStore = useTransactionsStore();
const { userTransactions, botTransactions } = storeToRefs(transactionStore);
const isOnMobile = useIsOnMobileStore().isOnMobile;

const didReconnect = !!localStorage.getItem('gameState');

// Channel-open, channel-close and contract-deploy transaction logs
const specialBotTx = botTransactions.value[0];
const specialUserTx = userTransactions.value[0];

const isFullscreen = ref(false);

onUpdated(() => {
  if (terminalEl.value) {
    terminalEl.value.scrollTo(0, terminalEl.value.scrollHeight);
  }
});
</script>

<template>
  <div
    class="transactions"
    data-testid="transactions"
    :class="{ fullscreen: isFullscreen }"
  >
    <div ref="terminalEl" class="terminal">
      <div class="header">
        <div class="title">
          {{ isOnMobile ? 'Tx Terminal' : 'Your Terminal' }}
        </div>
        <div class="title">{{ isOnMobile ? '' : 'Bot Terminal' }}</div>
        <button
          class="expand"
          aria-label="expand_button"
          @click="isFullscreen = !isFullscreen"
        >
          <img
            class="icon"
            :src="isFullscreen ? MINIMISE_ICON : EXPAND_ICON"
            alt="Expand icon"
          />
          <span> {{ isFullscreen ? 'Minimize' : 'Expand' }} </span>
        </button>
      </div>
      <div
        class="transactions-list"
        v-if="Object.keys(botTransactions).length > 0"
      >
        <Notification
          v-if="didReconnect"
          type="info"
          text="After reconnecting only the last 5 rounds are saved."
        />
        <!-- Channel initialization tx logs -->
        <Accordion :isOpen="true" data-testid="channel-init-tx-logs">
          <template v-slot:title>
            <div class="round">Channel Initialization</div>
          </template>
          <template v-slot:content>
            <div
              class="transaction-pair"
              v-for="(transaction, txIdx) in specialBotTx.slice(0, 2)"
              :key="transaction.id"
            >
              <div class="single-transaction">
                <div class="single-transaction-text">You:</div>
                <SingleTransaction :transaction="specialUserTx[txIdx]" />
              </div>
              <div class="single-transaction">
                <div class="single-transaction-text">Bot:</div>
                <SingleTransaction :transaction="transaction" />
              </div>
            </div>
          </template>
        </Accordion>
        <!-- Tx logs for each game round -->
        <Accordion
          data-testid="game-round-tx-logs"
          v-for="roundKey in Object.keys(botTransactions).filter(
            (round) => round !== '0'
          )"
          :key="roundKey"
        >
          <template v-slot:title>
            <div class="round">Round {{ roundKey }}</div>
          </template>
          <template v-slot:content>
            <div
              class="transaction-pair"
              v-for="(transaction, txIdx) in botTransactions[
                parseInt(roundKey)
              ]"
              :key="transaction.id"
            >
              <div class="single-transaction">
                <div class="single-transaction-text">You:</div>
                <SingleTransaction
                  :transaction="userTransactions[parseInt(roundKey)][txIdx]"
                />
              </div>
              <div class="single-transaction">
                <div class="single-transaction-text">Bot:</div>
                <SingleTransaction :transaction="transaction" />
              </div>
            </div>
          </template>
        </Accordion>
        <!-- Channel shutdown tx logs -->
        <Accordion
          :isOpen="true"
          data-testid="channel-shutdown-tx-logs"
          v-if="specialUserTx[2] || specialBotTx[2]"
        >
          <template v-slot:title>
            <div class="round">Channel Shutdown</div>
          </template>
          <template v-slot:content>
            <div class="transaction-pair">
              <div class="single-transaction">
                <div class="single-transaction-text">You:</div>
                <SingleTransaction :transaction="specialUserTx[2]" />
              </div>
              <div class="single-transaction">
                <div class="single-transaction-text">You:</div>
                <SingleTransaction :transaction="specialBotTx[2]" />
              </div>
            </div>
          </template>
        </Accordion>
      </div>
      <div class="transactions-list" v-else>
        <div class="empty-list">No transactions yet</div>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
@import '../../mediaqueries.scss';

.transactions {
  grid-area: transactions;
  background-color: var(--gray);
  padding: var(--padding);
  padding-bottom: min(20px, var(--padding));
  max-width: 100%;
  transition: height 0.6s cubic-bezier(0.25, 1, 0.5, 1);
  display: flex;
  flex-direction: column;

  &.fullscreen {
    position: absolute;
    z-index: 1;
    top: 0;
    left: 0;
    padding-bottom: var(--padding);
    width: calc(100% - var(--padding) * 2);
    height: calc(100vh - var(--padding) * 2);
  }
  .header {
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    padding-bottom: 10px;
    position: sticky;
    top: 0;
    z-index: 1;
    background-color: var(--gray);
    .title {
      width: 50%;
      font-weight: 500;
      width: 100%;
      display: flex;
      justify-content: space-between;
    }
    .expand {
      position: absolute;
      right: 0;
      user-select: none;
      display: flex;
      flex-direction: row;
      align-items: center;
      font-size: 28px;
      font-weight: 500;

      background: unset;
      border: unset;
      font-family: unset;
      color: unset;

      &:hover {
        cursor: pointer;
      }
      @include for-phone-only {
        display: flex;
        span {
          display: none;
        }
      }
      .icon {
        width: 28px;
        height: 28px;
        margin-right: 15px;
        @include for-phone-only {
          width: 16px;
          height: 16px;
          margin-right: 0;
        }
        @include for-tablet-portrait-up {
          width: 18px;
          height: 18px;
          margin-right: 5px;
        }
        @include for-tablet-landscape-up {
          width: 24px;
          height: 24px;
          margin-right: 10px;
        }
        @include for-desktop-up {
          width: 28px;
          height: 28px;
          margin-right: 15px;
        }
      }
    }
    .title,
    .expand {
      @include for-phone-only {
        font-size: 16px;
      }
      @include for-tablet-portrait-up {
        font-size: 18px;
      }
      @include for-tablet-landscape-up {
        font-size: 24px;
      }
      @include for-desktop-up {
        font-size: 28px;
      }
    }
  }
  .terminal {
    display: flex;
    flex-direction: column;
    overflow-y: auto;
    width: 100%;
    height: max-content;
    .transactions-list {
      display: flex;
      flex-direction: column;
      .empty-list {
        font-family: 'DM Mono', monospace;
        font-size: 16px;
      }
      .transaction-pair {
        display: flex;
        flex-direction: row;
        padding: 5px;
        border-radius: 5px;
        &:nth-of-type(odd) {
          background-color: rgba(255, 255, 255, 0.5);
        }
        .single-transaction {
          width: 100%;
          display: flex;
          .single-transaction-text {
            display: none;
            @include for-phone-only {
              display: block;
              font-size: 12px;
              font-weight: 500;
              padding-right: 7px;
            }
          }
        }
        @include for-phone-only {
          flex-direction: column;
        }
      }
    }
  }
}
</style>
