<script setup lang="ts">
import { Encoded } from '@aeternity/aepp-sdk/es/utils/encoder';
import { SignatureType } from '../../utils/game-channel/game-channel.types';

export interface TransactionLog {
  id: Encoded.TxHash;
  onChain: boolean;
  description: string;
  signed: SignatureType;
  timestamp: number;
}

defineProps<{
  transaction?: TransactionLog;
}>();

function formatDate(timestamp: number): string {
  const date = new Date(timestamp);
  return (
    ('0' + date.getHours()).slice(-2) +
    ':' +
    ('0' + date.getMinutes()).slice(-2) +
    ':' +
    ('0' + date.getSeconds()).slice(-2)
  );
}
function formatTxId(id: string): string {
  return id.length > 10 ? id.slice(0, 5) + '…' + id.slice(-5) : id;
}
</script>

<template>
  <div
    class="transaction"
    v-if="transaction && transaction.id"
    :class="{ 'on-chain': transaction.onChain }"
  >
    <span class="on-chain-pill" v-if="transaction.onChain">on Chain</span>
    <span :title="transaction.id">
      {{ `TXID ${formatTxId(transaction.id)} ` }} |
    </span>
    <span>
      {{ transaction.description }}
    </span>
    <div class="info">
      <span>{{ formatDate(transaction.timestamp) }} | </span>
      <span> {{ transaction.signed }}</span>
    </div>
  </div>
  <div class="transaction error" v-else-if="transaction && !transaction?.id">
    Something went wrong
  </div>
  <div class="transaction" v-else>pending...</div>
</template>

<style scoped lang="scss">
@import '../../mediaqueries.scss';

.transaction {
  font-family: 'DM Mono', monospace;
  font-size: 16px;
  margin-bottom: 5px;

  text-align: left;
  color: var(--green);
  a {
    color: var(--green);
  }
  @include for-phone-only {
    font-size: 11px;
  }
  @include for-tablet-portrait-up {
    font-size: 12px;
  }
  @include for-tablet-landscape-up {
    font-size: 16px;
  }
  .info {
    margin-left: 20px;
  }
  &.error {
    color: var(--pink);
  }
}
.on-chain {
  color: var(--pink);
}
.on-chain-pill {
  font-weight: bold;
  color: var(--pink);
  margin-right: 3px;
  background-color: #ccc;
  padding: 3px 5px;
  border-radius: 6px;
  position: relative;
  top: -2px;
  @include for-phone-only {
    font-size: 9px;
    padding: 3px;
  }
  @include for-tablet-portrait-up {
    font-size: 10px;
  }
  @include for-tablet-landscape-up {
    font-size: 12px;
  }
}
</style>
