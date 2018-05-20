<template>
  <div id="app">
    

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
      <button v-on:click="remove(expense)" class="deleteButton" >&#x2716;</button>
      <h2>{{expense.name}}</h2>
      <p>{{expense.type}}</p>
      <!--button v-on:click="editExpense(expense)" class="editButton">EDIT</button-->
      <p>Due date - {{expense.due}}</p>      
      <p class="expense">${{expense.amount}}</p>
      
    </div>
    <button v-on:click.prevent="addexpense = false" v-show="addexpense" class="button">Add Expense</button>
    </div>
    
    <form id="add-blog" v-show="!addexpense">
      <h2>Add Expense</h2>
      <input type="text" placeholder="Expense name or Company" ref="input" v-model="newExpense.name" maxlength="50"><br>
      <input type="text" placeholder="Type" ref="input" v-model="newExpense.type" maxlength="10"><br>
      <input type="number" placeholder="Amount" ref="amount" v-model="newExpense.amount"><br>
      <input type="date" placeholder="Due Date" ref="due" v-model="newExpense.due" v-bind:min="actualDate()"><br>
      <button v-on:click.prevent="addexpense = true; reset()" class="buttonCancel">Cancel</button>
      <span v-if="newExpense.name != '' && newExpense.type != '' && newExpense.due != '' && newExpense.amount != ''">
        <button v-on:click.prevent="add()" class="buttonSubmit">Submit</button>
      </span>
      <div id="preview" v-if=" newExpense.name != '' || newExpense.type != '' ||  newExpense.due != '' || newExpense.amount != ''">
        <h2>Preview Form</h2>
        <p>Expense: {{ newExpense.name }}</p><br>
        <p>Type: {{ newExpense.type }}</p><br>
        <p>Amount: <span v-if="newExpense.amount != ''">$</span>{{ newExpense.amount }}</p><br>
        <p>Due Date: {{ newExpense.due }}</p><br>
      </div>
      <br>
    </form>
    
    
      

<div class="backgroundW" v-if="loader"></div>
      <fulfilling-bouncing-circle-spinner
      :animation-duration="4000"
      :size="60"
      :color="'#4ed85b'"
      id="loader"
      v-if="loader"
      />
    

    <center><a href="">See this app repository</a> </center>
    
  </div>
</template>

<script>
import Firebase,{ functions } from "firebase";
import Toastr from 'toastr';
import { FulfillingBouncingCircleSpinner } from 'epic-spinners';

Toastr.options = {
  "closeButton": true,
  "debug": false,
  "newestOnTop": false,
  "progressBar": true,
  "positionClass": "toast-bottom-right",
  "preventDuplicates": true,
  "onclick": null,
  "showDuration": "300",
  "hideDuration": "1000",
  "timeOut": "5000",
  "extendedTimeOut": "1000",
  "showEasing": "swing",
  "hideEasing": "linear",
  "showMethod": "fadeIn",
  "hideMethod": "fadeOut"
}

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
let expensesRef = db.ref("expenses")

export default {
  components: {
    FulfillingBouncingCircleSpinner
  },
  data() {
    return {
      newExpense: {
        name: "",
        type: "",
        amount: '',
        due: ""
      },
      edit: false,
      addexpense: true,
      loader: true,
      actualDate(){
        let today = new Date();
        let dd = "0" + today.getDate();
        let mm = "0" + (today.getMonth() + 1);
        let yy = today.getFullYear();
        let dateString =  yy.toString() + "-" + mm.slice(-2).toString() + "-01"
        return dateString
      }
    };
  },
  firebase: {
    expenses: expensesRef
  },
  mounted(){
    db.ref("expenses").once("value", snap => {
      window.setTimeout(this.hideLoader, 3300)
      console.log("All files load...");
    })
  },
  methods: {
    add() {
      Toastr.remove()
      let error = false
      let numbers = ["0","1","2","3","4","5","6","7","8","9"]
      if (this.newExpense.amount < 1){
        Toastr.error('Your amount should be more than 0');
        error = true ;
      }
      for (let letter of this.newExpense.name){
        if (numbers.includes(letter)) {
          Toastr.error("The name of the company should'nt contain numbers");
          error = true ;
          break;
        }
      } 
      for (let letter of this.newExpense.type){
        if (numbers.includes(letter)) {
          Toastr.error("The type should'nt contain numbers");
          error = true ;
          break;
        }
      }
      if (error == false){
        expensesRef.push(this.newExpense);
        Toastr.success('You added a new expense...');
        this.newExpense.name = "";
        this.newExpense.type = "";
        this.newExpense.amount = "";
        this.newExpense.due = "";
        this.addexpense = true;
      }
    },
    remove(expense) {
      expensesRef.child(expense['.key']).remove()
      Toastr.success('You delete ' + '"'+ expense.name +'"')      
    },
    editExpense(expense){
      expensesRef.child(expense['.key']).update()
      Toastr.success('Edit succesfully')
    },
    total(){
        let total = 0;
        for (let expense in this.expenses){
          let number = parseFloat(this.expenses[expense].amount)
          total = total + number
        }
        return total.toFixed(2)
    },
    reset(){
      this.newExpense.name = "";
      this.newExpense.type = "";
      this.newExpense.amount = "";
      this.newExpense.due = "";
    },
    hideLoader(){
      this.loader = false
    }
  }
};


</script>









<style>
.backgroundW {
  top: 0;
  left: 0;
  background:white;
  position: fixed;
  width: 100%;
  height: 100%;
}

#loader {
  position:fixed;
  left: 50%;
  top: 45%;
  margin-top: -30px;
  margin-left: -30px;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  padding: 8px;
}

body::-webkit-scrollbar {
  width: 0 !important
}

p {
  font-size: 14px;
}

label {
  display: block;
  margin: 20px 0 10px;
}

/* Buttons */
button {
  background-color: transparent;
  border-radius: 3px;
  font-size: 15px;
  outline: none;
}
button:active,
button:checked {
  outline: none;
}

.buttonCancel {
  border: 1px solid #f76c6c;
}

.buttonSubmit {
  border: 1px solid #4ed85b;
}

.deleteButton {
  width: auto;
  border: none;
  float: right;
  height: auto;
  margin-top: 20px;
}
.editButton {
  width: auto;
  border: none;
  float: right;
  height: auto;
  margin-top: 20px;
  border: 1px solid #999;
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
  border: 1px solid #888;
  margin: 10px 0;
  border-radius: 3px;
  padding: 10px 30px 15px 30px;
  color: #333;
  position: relative;
}
.box p {
  font-size: 18px;
  margin: 0px;
}

.box h2 {
  margin: 20px 0 110px 0;
}

#total {
  border-radius: 3px;
  max-width: 500px;
  background: #4ed85b;
  color: white;
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

#preview {
  margin-top: 35px;
}

#preview p {
  display: inline;
  font-size: 18px;
}

#add-blog {
  margin: 35px auto;
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
}

input[type="date"] {
  display: block;
  width: 50%;
  padding: 8px;
  border: 1px solid #888;
  border-radius: 3px;
}
</style>
