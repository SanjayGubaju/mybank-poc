<template>
  <div class="col-md-12 col-lg-10">
    <b-breadcrumb>
      <b-breadcrumb-item to="/app/dashboard">
        <span class="fi flaticon-home"></span>
      </b-breadcrumb-item>
      <b-breadcrumb-item active>Term Deposit Details</b-breadcrumb-item>
    </b-breadcrumb>
    <b-row>
      <b-col>
        <div class="pb-xlg h-100">
          <div class="widgetBody widget-body" v-if="account && account.accountId">
            <div
              v-cloak
              class="widget-title widget-top widget-padding-md clearfix bg-danger text-white"
            >
              <button @click="unlinkAccount()" class="float-right btn btn-outline btn-sm mb-2">
                <i class="fa fa-unlink mr-2"></i>
                Unlink
              </button>
              <h1>
                {{account.displayName}}
                <a
                  @click="refresh()"
                  data-widgster="load"
                  class="small text-white la la-refresh"
                ></a>
              </h1>
              <h3>{{account.maskedNumber}}</h3>
              <span class="badge badge-success">{{account.openStatus}}</span>&nbsp;
              <span
                v-if="account.updated"
                class="btn btn-outline btn-xs"
              >Updated {{account.updated | formatDate}}</span>
              <h5>
                <div v-for="balance in balances" :key="balance.accountId">
                  <div
                    v-if="account.accountId === balance.accountId"
                    class="profileStat stat-item float-right stats-row mt-3"
                  >
                    <div class="profileStat stat-item">
                      <p
                        class="profileStatValue value text-right"
                      >$ {{ balance.deposit.availableBalance.amount }}</p>
                      <h6 class="name text-right">Available Balance</h6>
                    </div>
                    <div class="profileStat stat-item">
                      <p
                        class="profileStatValue value text-right"
                      >$ {{ balance.deposit.currentBalance.amount }}</p>
                      <h6 class="name text-right">Balance</h6>
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

                  <div v-if="account.termDeposit" class="stats-row col-md-12" xs="12">
                    <div class="profileStat stat-item">
                      <p
                        class="profileStatValue value text-right"
                      >{{account.termDeposit.lodgementDate | formatLongDate}}</p>
                      <h6 class="name text-right">Lodgement</h6>
                    </div>
                    <div class="profileStat stat-item">
                      <p
                        class="profileStatValue value text-right"
                      >{{account.termDeposit.maturityDate | formatLongDate}}</p>
                      <h6 class="name text-right">Maturity</h6>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <div>
              <transaction-table ref="txnTable"></transaction-table>
            </div>
          </div>
        </div>
      </b-col>
    </b-row>
  </div>
</template>
<script>
import Vue from "vue";
import Widget from "@/components/Widget/Widget";
import { mapState } from "vuex";
import TransactionTable from "./TransactionTable.vue";
import moment from "moment";
import axios from "axios";

Vue.filter("formatDate", function(value) {
  if (value) {
    return moment(value).format("DD MMM");
  }
});
Vue.filter("formatLongDate", function(value) {
  if (value) {
    return moment(value).format("MMM YYYY");
  }
});

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
    this.init();
  },
  methods: {
    init() {
      if (this.accountId !== undefined || this.accountId !== "") {
        this.$store.dispatch("accounts/getAccountById", this.accountId);
        this.$store.dispatch("accounts/loadAccountBalances");
      }
    },
    refresh() {
      //TODO: if userBankAuth is expired, re-authenticate the user
      axios.post("/accounts/" + this.accountId + "/refresh");
      this.init();
      this.$forceUpdate();
      this.$refs.txnTable.getTransactionsByAccountId(true);
    },
    unlinkAccount() {
      this.$router.push({ path: "/app/unlink/" + this.accountId });
    }
  },
  computed: mapState("accounts", ["account", "balances"])
};
</script>
<style src="./AccountDetails.scss" lang="scss" scoped />