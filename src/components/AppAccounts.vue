<template>
  <div class="jumbotron vertical-center">
    <div class="container">
      <div class="row">
        <div class="col-sm-12">
          <h1>Accounts</h1>
          <hr />
          <br />
          <!-- Alert Message -->
          <b-alert v-if="showMessage" variant="success" show>{{
            message
          }}</b-alert>
          <!-- b-alert v-if="error" variant="danger" show>{{ error }}</b-alert-->

          <button
            type="button"
            class="btn btn-success btn-sm"
            v-b-modal.account-modal
          >
            Create Account
          </button>
          <br /><br />
          <table class="table table-hover">
            <thead>
              <tr>
                <th scope="col">Account Name</th>
                <th scope="col">Account Number</th>
                <th scope="col">Account Balance</th>
                <th scope="col">Account Currency</th>
                <th scope="col">Account Status</th>
                <th scope="col">Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="account in accounts" :key="account.id" @click="handleAccountClick(account)">
                <td>{{ account.name }}</td>
                <td>{{ account.account_number }}</td>
                <td>{{ account.balance }}</td>
                <td>{{ account.currency }}</td>
                <td>
                  <span
                    v-if="account.status == 'Active'"
                    class="badge badge-success"
                    >{{ account.status }}</span
                  >
                  <span v-else class="badge badge-danger">{{
                    account.status
                  }}</span>
                </td>
                <td>
                  <div class="btn-group" role="group">
                    <button
                      type="button"
                      class="btn btn-info btn-sm"
                      v-b-modal.edit-account-modal
                      @click="editAccount(account)"
                    >
                      Edit
                    </button>
                    <button
                      type="button"
                      class="btn btn-danger btn-sm"
                      @click="deleteAccount(account)"
                    >
                      Delete
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
          <footer class="text-center">
            Copyright &copy; All Rights Reserved.
          </footer>
        </div>
      </div>
      <b-modal
        ref="addAccountModal"
        id="account-modal"
        title="Create a new account"
        hide-backdrop
        hide-footer
      >
        <b-form @submit="onSubmit" class="w-100">
          <b-form-group
            id="form-name-group"
            label="Account Name:"
            label-for="form-name-input"
          >
            <b-form-input
              id="form-name-input"
              type="text"
              v-model="createAccountForm.name"
              placeholder="Account Name"
              required
            >
            </b-form-input>
          </b-form-group>
          <b-form-group
            id="form-currency-group"
            label="Currency:"
            label-for="form-currency-input"
          >
            <b-form-input
              id="form-currency-input"
              type="text"
              v-model="createAccountForm.currency"
              :state="currencyState"
              placeholder="$ or €"
              @input="currencyTouched = true"
              required
            >
            </b-form-input>
            <b-form-invalid-feedback v-if="currencyShowError && !currencyValid">
                Currency must be "$" or "€".
            </b-form-invalid-feedback>
          </b-form-group>

          <b-form-group
            id="form-country-group"
            label="Country:"
            label-for="form-country-input"
          >
            <b-form-input
              id="form-country-input"
              type="text"
              v-model="createAccountForm.country"
              placeholder="Country"
              required
            >
            </b-form-input>
          </b-form-group>

          <b-form-group
            id="form-type-group"
            label="Account Type:"
            label-for="form-type-input"
          >
            <b-form-select 
              v-model="createAccountForm.type" 
              :options="createAccountForm.options"
              required
            >
            </b-form-select>
          </b-form-group>

          <b-form-group
            id="form-email-group"
            label="Email:"
            label-for="form-email-input"
          >
            <b-form-input
              id="form-email-input"
              type="text"
              v-model="createAccountForm.email"
              placeholder="Email"
              :state="emailState"
              @input="emailTouched = true"
              required
            >
            </b-form-input>
            <b-form-invalid-feedback v-if="emailShowError && !emailValid">
                Please enter a valid email format.
            </b-form-invalid-feedback>
          </b-form-group>


          <b-button type="submit" variant="outline-info"> Submit</b-button>
        </b-form>
      </b-modal>
      <!-- End of Modal for Create Account-->
      <!-- Start of Modal for Edit Account-->
      <b-modal
        ref="editAccountModal"
        id="edit-account-modal"
        title="Edit the account"
        hide-backdrop
        hide-footer
      >
        <b-form @submit="onSubmitUpdate" class="w-100">
          <b-form-group
            id="form-edit-name-group"
            label="Account Name:"
            label-for="form-edit-name-input"
          >
            <b-form-input
              id="form-edit-name-input"
              type="text"
              v-model="editAccountForm.name"
              placeholder="Account Name"
              required
            >
            </b-form-input>
          </b-form-group>
          <b-button type="submit" variant="outline-info">Update</b-button>
        </b-form>
      </b-modal>
      <!-- End of Modal for Edit Account-->
      <!-- Start of Modal for Display Account-->
      <b-modal ref="accountDetailsModal" id="account-details-modal" title="Account Details" hide-footer>
      <div v-if="selectedAccount">
        <p><strong>Account Name:</strong> {{ selectedAccount.name }}</p>
        <p><strong>Account Number:</strong> {{ selectedAccount.account_number }}</p>
        <p><strong>Account Balance:</strong> {{ selectedAccount.balance }}</p>
        <p><strong>Currency:</strong> {{ selectedAccount.currency }}</p>
        <p><strong>Status:</strong> {{ selectedAccount.status }}</p>
        <p><strong>Country:</strong> {{ selectedAccount.country }}</p>
        <p><strong>Email:</strong> {{ selectedAccount.email }}</p>
        <p><strong>Account Type:</strong> {{ selectedAccount.type }}</p>
      </div>
    </b-modal>
    <!-- End of Modal for Display Account-->
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "AppAccounts",
  data() {
    return {
      accounts: [],
      selectedAccount: null,
      createAccountForm: {
        name: "",
        currency: "",
        country: "",
        type: null,
        options: [
          { value: null, text: 'Please select an option' },
          { value: 'Checking', text: 'Checking' },
          { value: 'Savings', text: 'Savings' },
        ],
        email: "",
      },
      editAccountForm: {
        id: "",
        name: "",
      },
      showMessage: false,
      message: "",
      currencyShowError: false,
      currencyTouched: false,
      emailError: false,
      emailTouched: false,
    };
  },
  computed: {
    currencyValid() {
      // Checks if the input is exactly "$" or "€"
      return (this.createAccountForm.currency === "$" || this.createAccountForm.currency === "€");
    },
    currencyState() {
      // Checks if the input is exactly "$" or "€"
      return this.currencyTouched || this.currencyShowError ? this.currencyValid : null;
    },
    emailValid() {
      // Email regex for basic validation
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return emailRegex.test(this.createAccountForm.email);
    },
    emailState() {
      // Checks if the input is exactly "$" or "€"
      return this.emailTouched || this.emailShowError ? this.emailValid : null;
    },
  },
  methods: {
    handleAccountClick(account){
      this.selectedAccount = account
      this.$refs.accountDetailsModal.show()
    },
    /***************************************************
     * RESTful requests
     ***************************************************/

    //GET function
    RESTgetAccounts() {
      const path = `${process.env.VUE_APP_ROOT_URL}/accounts`;
      axios
        .get(path)
        .then((response) => {
          this.accounts = response.data.accounts;
        })
        .catch((error) => {
          console.error(error);
        });
    },

    // POST function
    RESTcreateAccount(payload) {
      const path = `${process.env.VUE_APP_ROOT_URL}/accounts`;
      axios
        .post(path, payload)
        .then((response) => {
          this.RESTgetAccounts();
          // For message alert
          this.message = "Account Created succesfully!";
          // To actually show the message
          this.showMessage = true;
          // To hide the message after 3 seconds
          setTimeout(() => {
            this.showMessage = false;
          }, 3000);
        })
        .catch((error) => {
          console.error(error);
          this.RESTgetAccounts();
        });
    },

    // Update function
    RESTupdateAccount(payload, accountId) {
      const path = `${process.env.VUE_APP_ROOT_URL}/accounts/${accountId}`;
      axios
        .put(path, payload)
        .then((response) => {
          this.RESTgetAccounts();
          // For message alert
          this.message = "Account Updated succesfully!";
          // To actually show the message
          this.showMessage = true;
          // To hide the message after 3 seconds
          setTimeout(() => {
            this.showMessage = false;
          }, 3000);
        })
        .catch((error) => {
          console.error(error);
          this.RESTgetAccounts();
        });
    },

    // Delete account
    RESTdeleteAccount(accountId) {
      const path = `${process.env.VUE_APP_ROOT_URL}/accounts/${accountId}`;
      axios
        .delete(path)
        .then((response) => {
          this.RESTgetAccounts();
          // For message alert
          this.message = "Account Deleted succesfully!";
          // To actually show the message
          this.showMessage = true;
          // To hide the message after 3 seconds
          setTimeout(() => {
            this.showMessage = false;
          }, 3000);
        })
        .catch((error) => {
          console.error(error);
          this.RESTgetAccounts();
        });
    },

    /***************************************************
     * FORM MANAGEMENT
     * *************************************************/

    // Initialize forms empty
    initForm() {
      this.createAccountForm.name = "";
      this.createAccountForm.currency = "";
      this.createAccountForm.country = "";
      this.createAccountForm.type = "";
      this.createAccountForm.email = "";
      this.editAccountForm.id = "";
      this.editAccountForm.name = "";
    },

    // Handle submit event for create account
    onSubmit(e) {
      e.preventDefault(); //prevent default form submit form the browser

      this.emailShowError = true;
      this.currencyShowError = true;

      if (this.currencyValid){
        this.currencyShowError = false; // Reset error flag after successful submission
        this.currencyTouched = false;
      }
      if (this.emailValid){
        this.emailShowError = false; // Reset error flag after successful submission
        this.emailTouched = false;
      }

      if (this.currencyValid && this.emailValid){
        this.$refs.addAccountModal.hide(); //hide the modal when submitted
        const payload = {
          name: this.createAccountForm.name,
          currency: this.createAccountForm.currency,
          country: this.createAccountForm.country,
          type: this.createAccountForm.type,
          email: this.createAccountForm.email
        };
        this.RESTcreateAccount(payload);
        this.initForm();
      }

    },

    // Handle submit event for edit account
    onSubmitUpdate(e) {
      e.preventDefault(); //prevent default form submit form the browser
      this.$refs.editAccountModal.hide(); //hide the modal when submitted
      const payload = {
        name: this.editAccountForm.name,
      };
      this.RESTupdateAccount(payload, this.editAccountForm.id);
      this.initForm();
    },

    // Handle edit button
    editAccount(account) {
      this.editAccountForm = account;
    },

    // Handle Delete button
    deleteAccount(account) {
      this.RESTdeleteAccount(account.id);
    },
  },

  /***************************************************
   * LIFECYClE HOOKS
   ***************************************************/
  created() {
    this.RESTgetAccounts();
  },
};
</script>
