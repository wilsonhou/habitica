<template>
  <div class="row">
    <div class="col-6">
      <h1>{{ $t('gemTransactions') }}</h1>
      <span v-if="gemTransactions.length === 0">{{ $t('noGemTransactions') }}</span>
      <table class="table">
        <tr
          v-for="entry in gemTransactions"
          :key="entry.createdAt"
        >
          <td>
            <span
              v-b-tooltip.hover="entry.createdAt"
            >{{ entry.createdAt | timeAgo }}</span>
          </td>
          <td>
            <span
              class="svg-icon inline icon-24"
              aria-hidden="true"
              v-html="icons.gem"
            ></span>
            <span
              class="amount gems"
              :class="entry.amount < 0 ? 'deducted' : 'added'"
            >{{ entry.amount * 4 }}</span>
          </td>
          <td>
            <span>{{ transactionTypeText(entry.transactionType) }}</span>
          </td>
          <td>
            <span v-html="entryReferenceText(entry)"></span>
          </td>
        </tr>
      </table>
    </div>
    <div class="col-6">
      <h1>{{ $t('hourglassTransactions') }}</h1>
      <span v-if="hourglassTransactions.length === 0">{{ $t('noHourglassTransactions') }}</span>
      <table class="table">
        <tr
          v-for="entry in hourglassTransactions"
          :key="entry.createdAt"
        >
          <td>
            <span
              v-b-tooltip.hover="entry.createdAt"
            >{{ entry.createdAt | timeAgo }}</span>
          </td>
          <td>
            <span
              class="svg-icon inline icon-24"
              aria-hidden="true"
              v-html="icons.hourglass"
            ></span>
            <span
              class="amount hourglasses"
              :class="entry.amount < 0 ? 'deducted' : 'added'"
            >{{ entry.amount }}</span>
          </td>
          <td>
            <span>{{ transactionTypeText(entry.transactionType) }}</span>
          </td>
          <td>
            <span v-html="entryReferenceText(entry)"></span>
          </td>
        </tr>
      </table>
    </div>
  </div>
</template>

<style lang="scss">
  @import '~@/assets/scss/colors.scss';

  .svg-icon {
    vertical-align: middle;
  }

  .amount {
    font-weight: bold;
    font-size: 1.1rem;
    margin-left: 4px;
  }


  .added::before {
    content: "+";
  }

  .gems {
    color: $gems-color;

    &.deducted {
      color: $red-10;
    }
  }

  .hourglasses {
    font-weight: bold;
    color: $hourglass-color;
    &.deducted {
      color: $red-10;
    }
  }
</style>

<script>
import moment from 'moment';
import svgGem from '@/assets/svg/gem.svg';
import svgHourglass from '@/assets/svg/hourglass.svg';

export default {
  filters: {
    timeAgo (value) {
      return moment(value).fromNow();
    },
    date (value) {
      // @TODO: Vue doesn't support this so we cant user preference
      return moment(value).toDate().toString();
    },
  },
  props: {
    gemTransactions: {
      type: Array,
      required: true,
    },
    hourglassTransactions: {
      type: Array,
      required: true,
    },
  },
  data () {
    return {
      icons: Object.freeze({
        gem: svgGem,
        hourglass: svgHourglass,
      }),
    };
  },
  methods: {
    entryReferenceText (entry) {
      if (entry.reference === undefined && entry.referenceText === undefined) {
        return '';
      }
      if (entry.referenceText) {
        return entry.referenceText;
      }
      return entry.reference;
    },
    transactionTypeText (transactionType) {
      return this.$t(`transaction_${transactionType}`);
    },
  },
};
</script>
