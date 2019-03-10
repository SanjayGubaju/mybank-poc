<template>
  <div class="tables-basic col-md-12 col-lg-10">
    <b-breadcrumb>
      <b-breadcrumb-item><span class="fi flaticon-home"></span></b-breadcrumb-item>
      <b-breadcrumb-item active>Manage Payess</b-breadcrumb-item>
    </b-breadcrumb>
    <h1 class="page-title">Manage Payees</h1>
    <b-row>
      <b-col>
        <Widget
          title="" customHeader refresh settings>
            <payee-filter-bar></payee-filter-bar>
            <vuetable ref="vuetable" 
              :fields="fields" 
              :api-mode="false"
              :data-manager="dataManager"
              track-by="payeeId"
              :sort-order="sortOrder"
              pagination-path="pagination"
              :css="css.table"
              @vuetable:pagination-data="onPaginationData"
            >
              <template slot="actions" slot-scope="props">
                <div class="table-button-container">
                    <button class="btn btn-warning" @click="editRow(props.rowData)">
                      <span class="fa fa-edit"></span></button>
                    <button class="btn btn-danger" @click="deleteRow(props.rowData)">
                      <span class="fa fa-trash"></span></button>
                </div>
              </template>
            </vuetable>
             <div class="ui basic segment grid">
              <vuetable-pagination-info ref="paginationInfo"
              :css="css.paginationInfo"
              ></vuetable-pagination-info>
              
              <vuetable-pagination ref="pagination"
                :css="css.pagination"
                @vuetable-pagination:change-page="onChangePage"
              ></vuetable-pagination>
            </div>
        </Widget>
      </b-col>
   </b-row>
  </div>
</template>

<script>
import Vue from 'vue';
import VueEvents from 'vue-events'
import Widget from '@/components/Widget/Widget';
import 'imports-loader?jQuery=jquery,this=>window!jquery-sparkline'; // eslint-disable-line
import 'imports-loader?$=jquery,this=>window!messenger/build/js/messenger'; // eslint-disable-line
import Vuetable from 'vuetable-2'
import VuetablePagination from "vuetable-2/src/components/VuetablePagination";
import VuetablePaginationInfo from "vuetable-2/src/components/VuetablePaginationInfo";
import PayeeFilterBar from './PayeeFilterBar.vue'
import _ from "lodash";
import CssConfig from './VuetableCssConfig.js';
import axios from 'axios';

const { Messenger } = window;

Vue.use(VueEvents)

export default {
  name: 'Payees',
  components: { 
    Widget, 
    Vuetable, 
    'vuetable-pagination': VuetablePagination, 
    VuetablePaginationInfo,
    PayeeFilterBar
  },
  data() {
    return {
      checkboxes1: [false, false, false, false],
      perPage: 20,
      localData: [],
      fields: [
        {
          name:'payeeId'
        }, 
        {
          name:'name',
          title: 'Name',
          sortField: 'name'
        }, 
        {
          name:'description',
          title: 'Description',
          sortField: 'description'
        },
        {
          name:'BSB',
          title: 'BSB'
        }, 
        {name:'accountNumber',
        title: 'Number'}, 
        {name:'payeeType',
        title: 'Payee Type'}, 
        {
          name: 'actions',
          title: 'Actions'
        }
      ],
      sortOrder: [
        {
          field: "name",
          direction: "asc"
        }
      ],
      css: CssConfig
    }
  },
  methods: {
    onPaginationData(paginationData) {
      this.$refs.pagination.setPaginationData(paginationData);
      this.$refs.paginationInfo.setPaginationData(paginationData);
    },
    onChangePage(page) {
      this.$refs.vuetable.changePage(page);
    },
    onFilterSet (filterText) {
      this.searchFor= filterText;
      if(this.$refs.vuetable !== undefined){
        this.$refs.vuetable.refresh();
      }
    },
    editRow(data){
      this.$router.push({ path: "/app/payees/" + data.payeeId });
    },
    deleteRow(data) {
      if (confirm("Are you sure you want to delete the record?")){
        console.log('deleted ' + data.payeeId);
        this.$store.dispatch('payees/deletePayee', data.payeeId);

        let index = this.localData.indexOf(data);
        this.localData.splice(index, 1);

        if(this.$refs.vuetable !== undefined){
          this.$refs.vuetable.refresh();
        }

        Messenger().post({
          extraClasses: this.locationClasses,
          message: data.name + ' was deleted successfully!!!',
          type: 'success',
          showCloseButton: true,
        });
      }
    },
    dataManager(sortOrder, pagination) {
      let local = this.localData;
      if (local === undefined || local.length < 1) return;

      if (this.searchFor) {
        // the text should be case insensitive
        let txt = new RegExp(this.searchFor, 'i')
        
        // search on name, description
        local = _.filter(local, function(item) {
          return item.name.search(txt) >= 0 || item.description.search(txt) >= 0
        })
      }

      // sortOrder can be empty, so we have to check for that as well
      // if (sortOrder.length > 0) {
      //   console.log("orderBy:", sortOrder[0].sortField, sortOrder[0].direction);
      //   local = _.orderBy(
      //     local,
      //     sortOrder[0].sortField,
      //     sortOrder[0].direction
      //   );
      // }

      pagination = this.$refs.vuetable.makePagination(
        local.length,
        this.perPage
      );
      let from = pagination.from - 1;
      let to = from + this.perPage;

      return {
        pagination: pagination,
        data: _.slice(local, from, to)
      };
    }
  },
  created() {
    axios
      .get('https://5a2u1vztie.execute-api.ap-southeast-2.amazonaws.com/dev/payees')
      .then(r => r.data)
      .then(payees => {
        this.localData= payees;
      }).catch(function (error) {
        console.log(error);
    });
  },
  watch: {
    localData (newVal, oldVal) {
      this.$refs.vuetable.refresh()
    }
  },
  mounted() {
    //FIX: this is only used to load payees in the store but not used here.
    //vuetable tries to load the data before this method is executed. 
    //https://glebbahmutov.com/blog/vue-vuex-rest-todomvx/
    //this.$store.dispatch('payees/loadPayees');
    this.$events.$on('filter-set', eventData => this.onFilterSet(eventData))
    this.$events.$on('delete-item', eventData => this.onDeleteItem(eventData))
  }
};
</script>

<style src="./Payees.scss" lang="scss" scoped />