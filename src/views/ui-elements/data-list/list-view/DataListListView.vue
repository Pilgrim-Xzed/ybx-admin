< !-- =========================================================================================
  File Name: DataListListView.vue
Description: Data List - List View
----------------------------------------------------------------------------------------
  Item Name: Vuesax Admin - VueJS Dashboard Admin Template
Author: Pixinvent
Author URL: http://www.themeforest.net/user/pixinvent
========================================================================================== -->

<template>
  <div id="data-list-list-view" class="data-list-container">
    <add-new-data-sidebar
      :isSidebarActive="addNewDataSidebar"
      @closeSidebar="addNewDataSidebar = false"
    />

    <vs-table
      ref="table"
      multiple
      v-model="selected"
      pagination
      :max-items="itemsPerPage"
      search
      :data="users"
    >
      <template slot="thead">
        <vs-th sort-key="name">Product ID</vs-th>

        <vs-th sort-key="order_status">Action</vs-th>
      </template>

      <template slot-scope="{ data }">
        <tbody>
          <vs-tr v-for="(tr, indextr) in data">
            <vs-td>
              <p class="product-name font-medium">
                {{ tr.productCode.toUpperCase() }}
              </p>
            </vs-td>

            <vs-td>
              <vs-button
                color="primary"
                @click="setProductKey(tr.productCode)"
                type="border"
                >Fund</vs-button
              >
            </vs-td>
          </vs-tr>
        </tbody>
      </template>
    </vs-table>

    <vs-popup
      classContent="popup-example"
      title="Yoda Box Auto Top Up"
      :active.sync="popupActive2"
    >
      <center>
        <h4>User Product Code : {{ productID.toUpperCase() }}</h4>
        <br />
        <form @submit.prevent="topUpUser()">
          <vs-input
            class="inputx mb-3"
            placeholder="Email Address"
            v-model="v$.form.email.$model"
            type="email"
          />
          <div
            class="input-errors"
            v-for="(error, index) of v$.form.email.$errors"
            :key="index"
          >
            <div class="error-msg">{{ error.$message }}</div>
          </div>
          <vs-input
            class="inputx mb-3"
            placeholder="Phone"
            v-model="v$.form.phone.$model"
            type="text"
          />
          <div
            class="input-errors"
            v-for="(error, index) of v$.form.phone.$errors"
            :key="index"
          >
            <div class="error-msg">{{ error.$message }}</div>
          </div>
          <vs-input
            class="inputx mb-3"
            placeholder="Amount"
            v-model="v$.form.amount.$model"
            type="number"
          />
          <div
            class="input-errors"
            v-for="(error, index) of v$.form.amount.$errors"
            :key="index"
          >
            <div class="error-msg">{{ error.$message }}</div>
          </div>

          <vs-button :disabled="v$.form.$invalid" color="primary" type="filled"
            >Top Up User Wallet</vs-button
          >
        </form>
      </center>
    </vs-popup>
  </div>
</template>

<script>
import AddNewDataSidebar from "../AddNewDataSidebar.vue";
import firebase from "firebase/app";
import "firebase/firestore";

import useVuelidate from "@vuelidate/core";
import { required, email, minLength, sameAs } from "@vuelidate/validators";

export function validName(name) {
  let validNamePattern = new RegExp("^[a-zA-Z]+(?:[-'\\s][a-zA-Z]+)*$");
  if (validNamePattern.test(name)) {
    return true;
  }
  return false;
}

export default {
  setup() {
    return { v$: useVuelidate() };
  },

  components: {
    AddNewDataSidebar,
  },
  data() {
    return {
      selected: [],
      users: [],
      itemsPerPage: 50,
      isMounted: false,
      addNewDataSidebar: false,
      form: {
        email: "",
        phone: "",
        amount: "",
      },
      productID: "",
      popupActive2: false,
      popupActive3: false,
    };
  },
  validations() {
    return {
      form: {
        email: { required, email },
        phone: { required },
        amount: { required },
      },
    };
  },
  computed: {
    currentPage() {
      if (this.isMounted) {
        return this.$refs.table.currentx;
      }
      return 0;
    },
  },
  methods: {
    formatData(data) {
      // formats data received from API
      let formattedData = data.map((item) => {
        const fields = item.data().fields;
        let obj = {};
        obj["id"] = item.id;
        obj["productCode"] = item.data().productCode;

        return obj;
      });
      return formattedData;
    },

    setProductKey(key) {
      this.productID = key;
      this.popupActive2 = true;
    },

    resetForm() {
      firebase
        .firestore()
        .collection("RecentTopUp")
        .doc(this.productID)
        .set({
          balance: parseInt(this.form.amount),
          email: this.form.email,
          fullName: "",
          productCode: this.productID,
          subscriptions: [],
        });
      this.popupActive2 = false;
      this.form.email = "";
      this.form.phone = "";
      this.form.amount = "";
    },

    topUpUser() {
      firebase
        .firestore()
        .collection("AutoTopUp")
        .doc(this.productID)
        .set({
          balance: parseInt(this.form.amount),
          email: this.form.email,
          fullName: "",
          productCode: this.productID,
          subscriptions: [],
        })
        .then((_) => {
          this.$http
            .post("https://yodaboxotp-7kvy5dvszq-uc.a.run.app/notify", {
              amount: this.form.amount,
              email: this.form.email,
            })
            .then((_) => {
              this.$vs.notify({
                title: "Top Up Success",
                text: "User account has been funded the user has been notified!",
                iconPack: "feather",
                icon: "icon-alert-circle",
                color: "success",
              });

              this.resetForm();
            })
            .catch((_) => {
              this.$vs.notify({
                title: "Notification",
                text: "Failed to top up user wallet",
                iconPack: "feather",
                icon: "icon-alert-circle",
                color: "warning",
              });
            });
        });
    },
  },
  created() {
    const thisIns = this;
    firebase
      .firestore()
      .collection("Users")
      .onSnapshot((querySnapshot) => {
        thisIns.users = thisIns.formatData(querySnapshot.docs);
      });
  },
  mounted() {
    this.isMounted = true;
  },
};
</script>

<style lang="scss">
#data-list-list-view {
  .vs-con-table {
    .vs-table--header {
      display: flex;
      flex-wrap: wrap-reverse;
      margin-left: 1.5rem;
      margin-right: 1.5rem;
      > span {
        display: flex;
        flex-grow: 1;
      }

      .vs-table--search {
        padding-top: 0;

        .vs-table--search-input {
          padding: 0.9rem 2.5rem;
          font-size: 1rem;

          & + i {
            left: 1rem;
          }

          &:focus + i {
            left: 1rem;
          }
        }
      }
    }

    .vs-table {
      border-collapse: separate;
      border-spacing: 0 1.3rem;
      padding: 0 1rem;

      tr {
        box-shadow: 0 4px 20px 0 rgba(0, 0, 0, 0.05);
        td {
          padding: 20px;
          &:first-child {
            border-top-left-radius: 0.5rem;
            border-bottom-left-radius: 0.5rem;
          }
          &:last-child {
            border-top-right-radius: 0.5rem;
            border-bottom-right-radius: 0.5rem;
          }
        }
        td.td-check {
          padding: 20px !important;
        }
      }
    }

    .vs-table--thead {
      th {
        padding-top: 0;
        padding-bottom: 0;

        .vs-table-text {
          text-transform: uppercase;
          font-weight: 600;
        }
      }
      th.td-check {
        padding: 0 15px !important;
      }
      tr {
        background: none;
        th.td-check {
          padding: 0 15px !important;
        }
        box-shadow: none;
      }
    }

    .vs-table--pagination {
      justify-content: center;
    }
  }
}
</style>
