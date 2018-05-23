<template>
  <div id="app">
    
    <div v-if="!loader"><img src="http://res.cloudinary.com/bryanenid/image/upload/v1526920804/progress-logo-compresed.png" alt="progress" class="logo">
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
      <button v-on:click="ask(expense)" class="deleteButton" ><i class="fas fa-times "></i></button>
      <h2>{{expense.name}}</h2>
      <p>{{expense.type}}</p>
      <button v-on:click="editExpenseF(expense)" class="editButton"><i class="far fa-edit "></i></button>
      <p>Due date - {{expense.due}}</p>      
      <p class="expense">${{expense.amount}}</p>
    </div>
    <b-btn v-b-modal.addForm class="addbutton"><i class="fas fa-plus "></i></b-btn>
  </div>
    
    <b-modal id="addForm" class="formExpenses" title="Add Expense" @keydown.enter.prevent="submitF()"  hide-footer no-close-on-backdrop no-close-on-esc hide-header-close centered lazy @ok="submitF" @cancel="cancelF('addForm')">
      <form id="add-blog" >
        <span class="rel"><i class="far fa-building fa-inputform"></i>Name:<input @keydown.enter.prevent="submitF()" type="text" placeholder="Expense name or Company" ref="input" v-model="newExpense.name" maxlength="50"></span>
        <span class="rel"><i class="fas fa-angle-double-right fa-inputform"></i>Type:<select name="type" v-model="newExpense.type" v-bind:class="typeSelect"><option value="" disabled>Please select the expense type</option><option v-for="type in typeExpenses" :key="type">{{ type }}</option></select></span>
        <span class="rel"><i class="fas fa-credit-card fa-inputform"></i>Amount:<input @keydown.enter.prevent="submitF()" type="number" placeholder="$0.00" ref="amount" v-model="newExpense.amount"></span>
        <span class="rel"><i class="far fa-calendar-alt fa-inputform"></i>Date:<input @keydown.enter.prevent="submitF()" type="date" placeholder="Due Date" ref="due" v-model="newExpense.due" v-bind:min="actualDate()"></span>
        
        <span v-if="newExpense.name != '' && newExpense.type != '' && newExpense.due != '' && newExpense.amount != ''">
          <button v-on:click.prevent="submitF()" class="buttonSubmit">Submit</button>
        </span>
        <button v-on:click.prevent="cancelF('addForm')" class="buttonCancel"> Cancel </button>
      </form>
    </b-modal>

    <b-modal id="editForm" class="formExpenses" title="Edit" hide-footer no-close-on-backdrop no-close-on-esc hide-header-close centered lazy @ok="submitEditF" @cancel="cancelF('editForm')">
      <form id="add-blog">

        <span class="rel"><i class="far fa-building fa-inputform"></i>Name:<input @keydown.enter.prevent="submitF()" type="text" placeholder="Expense name or Company" ref="input" v-model="editExpense.name" maxlength="50"></span>        
        <span class="rel"><i class="fas fa-angle-double-right fa-inputform"></i>Type:<input @keydown.enter.prevent="submitF()" type="text" placeholder="Monthly recurring, loan, credit, etc..." ref="input" v-model="editExpense.type" maxlength="20"></span> 
        <span class="rel"><i class="fas fa-credit-card fa-inputform"></i>Amount:<input @keydown.enter.prevent="submitF()" type="number" placeholder="$0.00" ref="amount" v-model="editExpense.amount"></span> 
        <span class="rel"><i class="far fa-calendar-alt fa-inputform"></i>Date:<input @keydown.enter.prevent="submitF()" type="date" placeholder="Due Date" ref="due" v-model="editExpense.due" v-bind:min="actualDate()"></span> 

        <span v-if="editExpense.name != '' && editExpense.type != '' && editExpense.due != '' && editExpense.amount != ''">
          <button v-on:click.prevent="submitEditF()" class="buttonSubmit"><i class="fas fa-check "></i> Submit</button>
        </span>

        <button v-on:click.prevent="cancelF('editForm')" class="buttonCancel"><i class="fas fa-times "></i> Cancel</button>
        
      </form>
    </b-modal>

    <b-modal id="confirmation" class="formExpenses" title="Are you sure you want to delete this expense?" hide-footer no-close-on-backdrop no-close-on-esc hide-header-close centered lazy @ok="removeData" @cancel="cancelF('confirmation')">
      <div class="center">
      <button v-on:click.prevent="removeData()" class="buttonSubmit">Yes</button>
      <button v-on:click.prevent="cancelF('confirmation')" class="buttonCancel">No</button>
      </div>
    </b-modal>
    
 
       
    <center class="repo"><a href="https://github.com/BryanEnid/progressApp">See this app repository</a> </center>
    
    
    </div>

    
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
  closeButton: false,
  debug: false,
  newestOnTop: false,
  progressBar: false,
  positionClass: "toast-top-right",
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
      typeExpenses: ['College Fund', 'Snack money', 'Savings','Entertainment / Clothes'],
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
    this.$nextTick(function(){
      db.ref("expenses").once("value", snap => {
        window.setTimeout(this.hideLoader, 0);
        console.log("All files load...");
      });
    })
  },
  methods: {
    submitF() {
      Toastr.clear();
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
    ask(expense) {
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



/*body {
  background: #222;
}*/

center.repo a {
  font-weight: bold;
  color: #00a99e;
}

center.repo {
  margin: 30px 0 20px 0;
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
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  padding: 10px;
}

body::-webkit-scrollbar {
  width: 0 !important;
}

p {
  font-size: 100%;
}

label {
  display: block;
  margin: 20px 0 10px;
}

/* Buttons */
.addbutton {
  border-radius: 3px;
  font-size: 100%;
  outline: none;
  border: 1px solid #999;
  box-shadow: 0px 0px 10px #00a99e;
  max-width: 500px;
  width: 100%;
  height: 50px;
  margin-top: 20px;
  margin-bottom: 50px;
  background: white;
  color: #666;
}

.logo {
  display: block;
  height: 100px;
  width: 100px;
  margin: 30px auto;
}

button:active,
button:checked {
  outline: none;
}

.buttonCancel {
  border: 1px solid #f76c6c;
  box-shadow: 0px 0px 10px #f76c6c;
  height: 40px;
  width: 110px;
  margin-right: 10px;
  background: #f76c6c;
  float: right;
  margin: 10px;
  border-radius: 3px;
  color:white;
}
#confirmation .center {
  width: 260px;
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
  width: 110px;
  margin-right: 10px;
  background: #4ed85b;
  float: right;
  margin: 10px;
  border-radius: 3px;
  color: white;
}

.deleteButton {
  color: rgb(253, 154, 154);
  position: absolute;
  bottom: 14px;
  right: 100px;
  border: 1px solid rgb(253, 154, 154);
  background: #222;
  border-radius: 3px 0px 0px 3px;
  padding: 0px 15px;
  height: 40px;
  width: 65px;
}

.editButton {
  position: absolute;
  bottom: 14px;
  right: 35px;
  color: #83e0da;
  border: 1px solid #83e0da;
  border-radius: 0px 3px 3px 0px;
  margin-top: 0px;
  color: #83e0da;
  background: #222;
  width: 65px;
  height: 40px;
}

.editButton > * {
  margin:0 auto;
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
  font-size: 100%;
}

.box {
  border: 1px solid #777;
  margin: 10px 0;
  border-radius: 3px;
  padding: 10px 30px 15px 30px;
  color: white;
  position: relative;
  background: #444;
}

.box h2 {
  padding: 5px 0 8px 0;
  border-bottom: 1px solid #777; 
  color: white;
  font-size: 150%;
}

.box p {
  font-size: 100%;
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

.formExpenses {
  color:#333;
  margin: 0;
}

#total p {
  color: white;
  font-weight: bold;
  font-size: 150%;
}

#total * {
  display: inline;
}

#total p {
  margin-left: 25px;
}

/*FONT AWESOME*/
.fa-inputform {
  position:absolute;
  bottom:-31px;
  left: 12px;
  z-index: 2;
}


/* Form */

#preview p {
  display: inline;
  font-size: 150%;
}

#add-blog {
  margin: 0px auto;
  max-width: 500px;
}

.formExpenses input[type="text"],
.formExpenses input[type="number"],
.formExpenses textarea {
  display: block;
  width: 100%;
  padding: 8px 8px 8px 40px;
  border: 1px solid #888;
  border-radius: 3px;
  margin-bottom: 10px;
}

.formExpenses input[type="date"], .typeSelect {
  display: block;
  height: 40px;
  width: 100%;
  padding: 8px 8px 8px 40px;
  border: 1px solid #888;
  border-radius: 3px;
  margin-bottom: 10px;
}

.typeSelect {
  appearance: none;
  background: none; 
  color: #999;
  text-align-last: right;
}

/* <Select> Tag on Google Chrome alternative*/
@media all and (-webkit-min-device-pixel-ratio:0) {
  .typeSelect {
    text-align-last: left;
    color: #999;
  }
  .fa-angle-double-right {
    display: none;
  }
}

/* Iphone Fix FontAwesome*/
@media screen and (max-device-width: 480px) {
  .fa-angle-double-right {
    display: initial;
  }
}

@media only screen and (max-width: 500px) {
   .deleteButton {
    border: 1px solid rgb(253, 154, 154);
    background: #222;
    border-radius: 3px 0px 0px 3px;
    padding: 0px 15px; 
    bottom: 15px;
    height: 40px;
    width: 50px;
    right: 70px;
  }
   .editButton {
    border: 1px solid #83e0da;
    border-radius: 0px 3px 3px 0px;
    margin-top: 0px;
    color: #83e0da;
    background: #222;
    width: 50px;
    height: 40px;
    bottom: 15px;
    right: 20px;
  }
   .box {
    padding: 10px 20px 10px 20px;
  }
}
</style>
