<template>
  <div id="app">
    
    <div v-if="!loader"><img src="https://lh3.googleusercontent.com/wlWGrD9xRLVXuniUwAa0nzncN0dLarnjbY9ecF_WUrlvr8YkSnfLFX4gvSYYNI7gc8-V-SjrmV5alSUBmnZtvu7OmrmvSES3puuY2CZLUeIuQASIiwYhhP9iBUS1BA9UrGXI97uYOV1eT3TH67FES3xrGMtDPLw-XRKwQAAkOwdGTIGYZsCO-JGnFftO8dq6glBomW1fiz5agEg00tESujZ97ItMJorCjBvCcvhVmHfN3bT0pga-xWM6GDifHcStd73PVqzndVMjDArElhHFTEbYq6_p89qJZKnoqZ0u5cXz9rdadEE-og9RW1yIV0qfH-Xf2Vsw5erBHnxlJt3AyHEBGFOiZzasCDrVzN8D5ljnixi5aF6ogu8QqGN-PvLCqF3Zaeq5JRQo6TXs1-u24EHZ_xdrG_eUe3x21vTE9OGXovHJQjj38rSY37FmwTl2bzbW_1OoQeWnFu5U0dD3CZhTyMpVUZIA158HTm4EcW96VKnKRJs9E-nxDbLjW-6OMD7gXMDD_n9ZnDoULDpcvEqz5ebz0N4sLGZfQSiyeqV2g-FF4c7RslCOB_qNAeMs9mHcxtSPorQm0NiEkZaJNO7Qi-n76l0pxAHP_2I=s500-no" alt="progress" class="logo">
    <div id="items">
      <div id="total">
      <p>Total:</p>
      <p>{{ total() }}</p>
    </div>

    <!-- Done** Boxes of expenses located in Server -->
    <div class="empty" v-if="expenses == 0">
      <p>It seems that you dont have any expenses...</p>
    </div>
    <div v-for="expense in expenses" :key='expense.id' class="box">
      <button v-on:click="confirmation(expense)" class="deleteButton" >&#x2716;</button>
      <h2>{{expense.name}}</h2>
      <p>{{expense.type}}</p>
      <button v-on:click="editExpenseF(expense)" class="editButton">EDIT</button>
      <p>Due date - {{expense.due}}</p>      
      <p class="expense">${{expense.amount}}</p>
    </div>
    <b-btn v-b-modal.addForm class="button">&#x2795;</b-btn>
  </div>
    
    <b-modal id="addForm" class="form" title="Add Expense" hide-footer no-close-on-backdrop hide-header-close>
      <form id="add-blog">
        Name:<input type="text" placeholder="Expense name or Company" ref="input" v-model="newExpense.name" maxlength="50">
        Type:<input type="text" placeholder="Monthly recurring, loan, credit, etc..." ref="input" v-model="newExpense.type" maxlength="20">
        Amount: <input type="number" placeholder="$0.00" ref="amount" v-model="newExpense.amount">
        Date: <input type="date" placeholder="Due Date" ref="due" v-model="newExpense.due" v-bind:min="actualDate()">
        <span v-if="newExpense.name != '' && newExpense.type != '' && newExpense.due != '' && newExpense.amount != ''">
          <button v-on:click.prevent="submitF()" class="buttonSubmit">Submit</button>
        </span>
        <button v-on:click.prevent="cancelF('addForm')" class="buttonCancel">Cancel</button>
      </form>
    </b-modal>

    <b-modal id="editForm" class="form" title="Edit" hide-footer no-close-on-backdrop hide-header-close>
      <form id="add-blog">
        Name:<input type="text" placeholder="Expense name or Company" ref="input" v-model="editExpense.name" maxlength="50">
        Type:<input type="text" placeholder="Monthly recurring, loan, credit, etc..." ref="input" v-model="editExpense.type" maxlength="20">
        Amount: <input type="number" placeholder="$0.00" ref="amount" v-model="editExpense.amount">
        Date: <input type="date" placeholder="Due Date" ref="due" v-model="editExpense.due" v-bind:min="actualDate()">
        <span v-if="editExpense.name != '' && editExpense.type != '' && editExpense.due != '' && editExpense.amount != ''">
          <button v-on:click.prevent="submitEditF()" class="buttonSubmit">Submit</button>
        </span>
        <button v-on:click.prevent="cancelF('editForm')" class="buttonCancel">Cancel</button>
      </form>
    </b-modal>

    <b-modal id="confirmation" class="form" title="Are you sure you want to delete this expense?" no-close-on-backdrop hide-header-close hide-footer>
      <div class="center">
      <button v-on:click.prevent="removeData()" class="buttonSubmit">Yes</button>
      <button v-on:click.prevent="cancelF('confirmation')" class="buttonCancel">No</button>
      </div>
    </b-modal>
    
 
       

    <center><a href="https://github.com/BryanEnid/progressApp">See this app repository</a> </center></div>

    
    <div class="backgroundW" v-if="loader"></div>
      <fulfilling-bouncing-circle-spinner
      :animation-duration="4000"
      :size="60"
      :color="'#00a99e'"
      id="loader"
      v-if="loader"
      />
    
  </div>
</template>

<script>
import Firebase, { functions } from "firebase";
import Toastr from "toastr";
import { FulfillingBouncingCircleSpinner } from "epic-spinners";

Toastr.options = {
  closeButton: true,
  debug: false,
  newestOnTop: false,
  progressBar: true,
  positionClass: "toast-bottom-right",
  preventDuplicates: true,
  onclick: null,
  showDuration: "300",
  hideDuration: "1000",
  timeOut: "5000",
  extendedTimeOut: "1000",
  showEasing: "swing",
  hideEasing: "linear",
  showMethod: "fadeIn",
  hideMethod: "fadeOut"
};

let config = {
  apiKey: "AIzaSyDDILZWKSq_SwyEilYP3t07lNu3XVaA7YM",
  authDomain: "progress-51318.firebaseapp.com",
  databaseURL: "https://progress-51318.firebaseio.com",
  projectId: "progress-51318",
  storageBucket: "progress-51318.appspot.com",
  messagingSenderId: "759681807256"
};

let app = Firebase.initializeApp(config);
let db = app.database();
let expensesRef = db.ref("expenses");

export default {
  components: {
    FulfillingBouncingCircleSpinner
  },
  data() {
    return {
      newExpense: {
        name: "",
        type: "",
        amount: "",
        due: ""
      },
      editExpense: {
        name: "",
        type: "",
        amount: "",
        due: ""
      },
      key: '',
      edit: false,
      loader: true,
      actualDate() {
        let today = new Date();
        let dd = "0" + today.getDate();
        let mm = "0" + (today.getMonth() + 1);
        let yy = today.getFullYear();
        let dateString = yy.toString() + "-" + mm.slice(-2).toString() + "-01";
        return dateString;
      }
    };
  },
  firebase: {
    expenses: expensesRef
  },
  mounted() {
    db.ref("expenses").once("value", snap => {
      window.setTimeout(this.hideLoader, 1000);
      console.log("All files load...");
    });
  },
  methods: {
    submitF() {
      Toastr.remove();
      let error = false;
      let numbers = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"];
      if (this.newExpense.amount < 1) {
        Toastr.error("Your amount should be more than 0");
        error = true;
      }
      for (let letter of this.newExpense.name) {
        if (numbers.includes(letter)) {
          Toastr.error("The name of the company should'nt contain numbers");
          error = true;
          break;
        }
      }
      for (let letter of this.newExpense.type) {
        if (numbers.includes(letter)) {
          Toastr.error("The type should'nt contain numbers");
          error = true;
          break;
        }
      }
      if (error == false) {
        expensesRef.push(this.newExpense);
        Toastr.success("You added a new expense...");
        this.newExpense.name = "";
        this.newExpense.type = "";
        this.newExpense.amount = "";
        this.newExpense.due = "";
        this.$root.$emit('bv::hide::modal','addForm')
      }
    },
    submitEditF() {
      Toastr.remove();
      let error = false;
      let numbers = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"];
      if (this.editExpense.amount < 1) {
        Toastr.error("Your amount should be more than 0");
        error = true;
      }
      for (let letter of this.editExpense.name) {
        if (numbers.includes(letter)) {
          Toastr.error("The name of the company should'nt contain numbers");
          error = true;
          break;
        }
      }
      for (let letter of this.editExpense.type) {
        if (numbers.includes(letter)) {
          Toastr.error("The type should'nt contain numbers");
          error = true;
          break;
        }
      }
      if (error == false) {
        console.log(this.key)
        expensesRef.child(this.key).set({
          name: this.editExpense.name,
          type: this.editExpense.type,
          amount: this.editExpense.amount,
          due: this.editExpense.due
        });
        this.key = "";
        Toastr.success("It was succesfully updated!");
        this.editExpense.name = "";
        this.editExpense.type = "";
        this.editExpense.amount = "";
        this.editExpense.due = "";
        this.$root.$emit('bv::hide::modal','editForm')
      }
    },
    removeData(){
      let key = this.key;
      this.$root.$emit('bv::hide::modal','confirmation')
      expensesRef.child(key).remove();
      Toastr.success("Removed!"); 
      this.key = "";
    },
    confirmation(expense) {
      this.$root.$emit('bv::show::modal','confirmation')
      this.key = expense['.key']
    },
    editExpenseF(expense) {
      this.$root.$emit("bv::show::modal","editForm")
      this.key = expense['.key']
      this.editExpense.name = expense.name;
      this.editExpense.type = expense.type;
      this.editExpense.due = expense.due;
      this.editExpense.amount = expense.amount;
      this.editExpense.name = expense.name;
    },
    total() {
      let total = 0;
      for (let expense in this.expenses) {
        let number = parseFloat(this.expenses[expense].amount);
        total = total + number;
      }
      return total.toFixed(2);
    },
    cancelF(name) {
      this.$root.$emit('bv::hide::modal', name)
      this.newExpense.name = "";
      this.newExpense.type = "";
      this.newExpense.amount = "";
      this.newExpense.due = "";
    },
    hideLoader() {
      this.loader = false;
    }
  }
};
</script>









<style>
.logo {
  display: block;
  height: 100px;
  width: 100px;
  margin: 30px auto;
}

center a {
  font-weight: bold;
  color: #00a99e;
}

body {
  background: #222;
}

center {
  margin-top: 100px;
}

.backgroundW {
  top: 0;
  left: 0;
  background: #222;
  position: fixed;
  width: 100%;
  height: 100%;
}

#loader {
  position: fixed;
  left: 50%;
  top: 45%;
  margin-top: -30px;
  margin-left: -30px;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: white;
  padding: 8px;
}

body::-webkit-scrollbar {
  width: 0 !important;
}

p {
  font-size: 14px;
}

label {
  display: block;
  margin: 20px 0 10px;
}

/* Buttons */
.button {
  border-radius: 3px;
  font-size: 15px;
  outline: none;
  border: 1px solid #999;
  box-shadow: 0px 0px 10px #00a99e;
  max-width: 500px;
  width: 100%;
  height: 50px;
  margin-top: 40px;
  background: white;
  color: black;
}
button:active,
button:checked {
  outline: none;
}

.buttonCancel {
  border: 1px solid #f76c6c;
  box-shadow: 0px 0px 10px #f76c6c;
  height: 40px;
  width: 80px;
  margin-right: 10px;
  background: white;
  color: black;
  float: right;
  margin: 10px;
  border-radius: 3px;
}
#confirmation .center {
  width: 200px;
  margin:0 auto;
}

#confirmation .buttonSubmit {
  color: white;
  background: #4ed85b;
  font-weight:bold;
  position: relative;

}

#confirmation .buttonCancel {
  color: white;
  background: #f76c6c;
  font-weight:bold;
  position: relative;

}

.buttonSubmit {
  border: 1px solid #4ed85b;
  box-shadow: 0px 0px 10px #00a99e;
  height: 40px;
  width: 80px;
  margin-right: 10px;
  background: white;
  color: black;
  float: right;
  margin: 10px;
  border-radius: 3px;
}

.deleteButton {
  width: auto;
  border: none;
  height: auto;
  color: white;
  background: #444;
  position: absolute;
  bottom: 15px;
  right: 75px;
  width: 50px;
  border-right: 1px solid white;
}

.editButton {
  position: absolute;
  bottom: 15px;
  right: 15px;
  width: auto;
  border: none;
  height: auto;
  margin-top: 0px;
  color: white;
  background: #444;
}

/*Expenses listed from server & Total*/

#items {
  margin: 0 auto;
  display: block;
  max-width: 500px;
  border-radius: 3px;
}

.empty {
  margin: 10px;
  color: #999;
}

.empty p {
  font-size: 15px;
}

.box {
  border: 1px solid #00a99e;
  margin: 10px 0;
  border-radius: 3px;
  padding: 10px 30px 15px 30px;
  color: white;
  position: relative;
  background: #444;
}

.box h2 {
  color: white;
}

.box p {
  font-size: 18px;
  margin: 0px;
}

#total {
  border: 1px solid #00847b;
  border-radius: 3px;
  max-width: 500px;
  background: #00a99e;
  color: white;
  margin: 30px 0;
}

.form {
  color:#333;
  margin: 0;
}

#total p {
  color: white;
  font-weight: bold;
  font-size: 30px;
}

#total * {
  display: inline;
}

#total p {
  margin-left: 25px;
}


/* Form */

#preview p {
  display: inline;
  font-size: 18px;
}

#add-blog {
  margin: 0px auto;
  max-width: 500px;
}

input[type="text"],
input[type="number"],
textarea {
  display: block;
  width: 100%;
  padding: 8px;
  border: 1px solid #888;
  border-radius: 3px;
  margin-bottom: 10px;
}

input[type="date"] {
  display: block;
  height: 40px;
  width: 100%;
  padding: 8px;
  border: 1px solid #888;
  border-radius: 3px;
  margin-bottom: 30px;
}

@media only screen and (max-width: 500px) {
  .deleteButton {
    width: auto;
    height: 50px;
    right: 90px;
    background: #00a99e;
    border-right: none;
    border-radius: 3px;
  }
  .editButton {
    border: 1px solid #00a99e;
    border-radius: 3px;
    margin-top: 0px;
    height: 50px;
    color: white;
    background: #00a99e;
  }
  .box {
    padding: 10px;
  }
}
</style>
