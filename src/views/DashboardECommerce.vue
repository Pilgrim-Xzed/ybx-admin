<!-- =========================================================================================
  File Name: DashboardAnalytics.vue
  Description: Dashboard Analytics
  ----------------------------------------------------------------------------------------
  Item Name: Vuesax Admin - VueJS Dashboard Admin Template
  Author: Pixinvent
  Author URL: http://www.themeforest.net/user/pixinvent
========================================================================================== -->

<template>
  <div id="dashboard-analytics">
    <div class="vx-row">
      <!-- CARD 1: CONGRATS -->
      <div class="vx-col w-full lg:w-1/2 mb-base">
        <vx-card
          slot="no-body"
          class="text-center bg-primary-gradient greet-user"
        >
          <img
            src="../assets/images/elements/decore-left.png"
            class="decore-left"
            alt="Decore Left"
            width="200"
          />
          <img
            src="../assets/images/elements/decore-right.png"
            class="decore-right"
            alt="Decore Right"
            width="175"
          />
          <feather-icon
            icon="AwardIcon"
            class="
              p-6
              mb-8
              bg-primary
              inline-flex
              rounded-full
              text-white
              shadow
            "
            svgClasses="h-8 w-8"
          ></feather-icon>
          <h1 class="mb-6 text-white">Welcome Back,</h1>
          <p class="xl:w-3/4 lg:w-4/5 md:w-2/3 w-4/5 mx-auto text-white">
            Get Insight <strong></strong> from Yoda Box analytics.
          </p>
        </vx-card>
      </div>

      <!-- CARD 2: SUBSCRIBERS GAINED -->
      <div class="vx-col w-full sm:w-1/2 md:w-1/2 lg:w-1/4 xl:w-1/4 mb-base">
        <statistics-card-line
          icon="UsersIcon"
          :statistic="users.length"
          statisticTitle="Total Users"
          :chartData="analyticsData.subscribersGained"
          type="area"
        ></statistics-card-line>
      </div>

      <!-- CARD 3: ORDER RECIEVED -->
      <div class="vx-col w-full sm:w-1/2 md:w-1/2 lg:w-1/4 xl:w-1/4 mb-base">
        <statistics-card-line
          icon="ShoppingBagIcon"
          statistic="97"
          statisticTitle="Auto Top Up"
          color="warning"
          type="area"
        ></statistics-card-line>
      </div>
    </div>

    <div class="vx-row">
      <!-- CARD 9: DISPATCHED ORDERS -->
      <div class="vx-col w-full">
        <vx-card title="Recently Funded Users">
          <div slot="no-body" class="mt-4">
            <vs-table :data="recentTopUp">
              <template slot="thead">
                <vs-th></vs-th>
                <vs-th>PRODUCT CODE</vs-th>
                <vs-th>EMAIL</vs-th>
                <vs-th>AMOUNT</vs-th>
              
              </template>

              <template slot-scope="{ data }">
                <vs-tr :key="indextr" v-for="(tr, indextr) in recentTopUp">
                  <vs-td :data="data[indextr].productCode">
                    
                  </vs-td>
                  <vs-td :data="data[indextr].productCode">
                    <span><b>{{ data[indextr].productCode.toUpperCase() }}</b></span>
                  </vs-td>
                 
                  
                  
                  <vs-td :data="data[indextr].email">
                    <span>{{ data[indextr].email }}</span>
                  </vs-td>
                  <vs-td :data="data[indextr].balance">
                    <span>₦ {{ data[indextr].balance }}</span>
                  </vs-td>
                </vs-tr>
              </template>
            </vs-table>
          </div>
        </vx-card>
      </div>
    </div>
  </div>
</template>

<script>
import VueApexCharts from "vue-apexcharts";
import StatisticsCardLine from "@/components/statistics-cards/StatisticsCardLine.vue";
import analyticsData from "./ui-elements/card/analyticsData.js";
import ChangeTimeDurationDropdown from "@/components/ChangeTimeDurationDropdown.vue";
import firebase from "firebase/app";
import "firebase/firestore";

export default {
  data() {
    return {
      analyticsData: analyticsData,
      isImp: false,
      navbarSearchAndPinList: this.$store.state.navbarSearchAndPinList,
      show: false,
      items: [1, 2, 3, 4, 5, 6, 7, 8, 9],
      recentTopUp: [],
      nextNum: 10,
      users:[],
      tableList: [
        "vs-th: Component",
        "vs-tr: Component",
        "vs-td: Component",
        "thread: Slot",
        "tbody: Slot",
        "header: Slot",
      ],
     
        
    };
  },
  methods: {
    formatData(data) {
      // formats data received from API
      let formattedData = data.map((item) => {
        const fields = item.data().fields;
        let obj = {};
        obj["id"] = item.id;
        obj["productCode"] = item.data().productCode;
        obj["email"] = item.data().email;
        obj["balance"] = item.data().balance;

        return obj;
      });
      return formattedData;
    },
  },
  components: {
    VueApexCharts,
    StatisticsCardLine,
    ChangeTimeDurationDropdown,
  },
  created() {
    const thisIns = this;
    firebase
      .firestore()
      .collection("RecentTopUp")
      .onSnapshot((querySnapshot) => {
        thisIns.recentTopUp = thisIns.formatData(querySnapshot.docs);
      });



       firebase
      .firestore()
      .collection("Users")
      .onSnapshot((querySnapshot) => {
        thisIns.users = thisIns.formatData(querySnapshot.docs);
      });
  },
};
</script>

<style lang="scss">
#dashboard-analytics {
  .greet-user {
    position: relative;
    .decore-left {
      position: absolute;
      left: 0;
      top: 0;
    }
    .decore-right {
      position: absolute;
      right: 0;
      top: 0;
    }
  }

  @media (max-width: 576px) {
    .decore-left,
    .decore-right {
      width: 140px;
    }
  }
}
</style>
