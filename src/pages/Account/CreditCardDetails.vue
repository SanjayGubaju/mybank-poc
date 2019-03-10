<template>
  <div class="col-md-12 col-lg-10">
    <b-breadcrumb>
      <b-breadcrumb-item>
        <span class="fi flaticon-home"></span>
      </b-breadcrumb-item>
      <b-breadcrumb-item>Accounts</b-breadcrumb-item>
      <b-breadcrumb-item active>Credit Card Details</b-breadcrumb-item>
    </b-breadcrumb>
    <b-row>
      <b-col>
        <div class="pb-xlg h-100">
          <div class="widgetBody widget-body">
            <div class="widget-title widget-top widget-padding-md clearfix bg-secondary text-white">
              <a href="#" class="float-right btn-transfer btn btn-outline btn-sm mb-2">
                <i class="fa fa-edit mr-2"></i>
                Make a Payment
              </a>
              <h1>{{account.displayName}}</h1>
              <h3>{{account.maskedNumber}}</h3>
              <span class="badge badge-success badge-pill">{{account.openStatus}}</span>
              <h5>
                <div v-for="balance in balances" :key="balance.accountId">
                  <div
                    v-if="account.accountId === balance.accountId"
                    class="stats-item float-right"
                  >
                    <div class="profileStat stat-item float-right stats-row mt-3">
                      <div class="profileStat stat-item">
                        <p
                          class="profileStatValue value text-right"
                        >$ {{ balance.lending.creditLimit.amount }}</p>
                        <h6 class="name text-right">Limit</h6>
                      </div>
                      <div class="profileStat stat-item">
                        <p
                          class="profileStatValue value text-right"
                        >$ {{ balance.lending.availableBalance.amount }}</p>
                        <h6 class="name text-right">Available Balance</h6>
                      </div>
                      <div class="profileStat stat-item">
                        <p
                          class="profileStatValue value text-right"
                        >$ {{ balance.lending.accountBalance.amount }}</p>
                        <h6 class="name text-right">Balance</h6>
                      </div>
                    </div>
                  </div>
                </div>
              </h5>
            </div>
            <div class="row">
              <div xs="12">
                <div class="profileContactContainer">
                  <span class="thumb-xl mb-3" v-if="account !== undefined">
                    <img
                      :src="require('../../assets/banks/' + account.institution + '.png')"
                      alt="..."
                      class="profileAvatar rounded-circle"
                    >
                  </span>

                  <div v-if="account.creditCard" class="stats-row col-md-12" xs="12">
                    <div class="profileStat stat-item">
                      <p
                        class="profileStatValue value text-right"
                      >${{account.creditCard.minPaymentAmount}}</p>
                      <h6 class="name text-right">Min Payment</h6>
                    </div>
                    <div class="profileStat stat-item">
                      <p
                        class="profileStatValue value text-right"
                      >${{account.creditCard.paymentDueAmount}}</p>
                      <h6 class="name text-right">Due Amount</h6>
                    </div>
                    <div class="profileStat stat-item">
                      <p
                        class="profileStatValue value text-right"
                      >{{account.creditCard.paymentDueDate | date('DD MMMM')}}</p>
                      <h6 class="name text-right">Due Date</h6>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <div class="row">
              <h4>Transaction History</h4>
              <transaction-table></transaction-table>
            </div>
          </div>
        </div>
      </b-col>
    </b-row>
  </div>
</template>
<script>
import Widget from "@/components/Widget/Widget";
import { mapState, mapGetters } from "vuex";
import TransactionTable from "./TransactionTable.vue";

const { Messenger } = window;

export default {
  components: {
    Widget,
    "transaction-table": TransactionTable
  },
  data() {
    return {
      accountId: ""
    };
  },
  created() {
    this.accountId = this.$route.params.accountId;
    if (this.accountId !== undefined || this.accountId !== "") {
      this.$store.dispatch("accounts/getAccountById", this.accountId);
      this.$store.dispatch("accounts/loadAccountBalances");
    }
  },
  methods: {},
  computed: mapGetters("accounts", ["account", "balances"])
};
</script>
<style src="./AccountDetails.scss" lang="scss" scoped />