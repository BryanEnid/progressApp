<template>
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
</template>

<script>
import Firebase from "firebase";

let config = {
  apiKey: "AIzaSyDDILZWKSq_SwyEilYP3t07lNu3XVaA7YM",
  authDomain: "progress-51318.firebaseapp.com",
  databaseURL: "https://progress-51318.firebaseio.com",
  projectId: "progress-51318",
  storageBucket: "progress-51318.appspot.com",
  messagingSenderId: "759681807256"
};

/*let app = Firebase.initializeApp(config);
let db = app.database();
let expensesRef = db.ref("expenses");*/

export default {
  data() {
    return {
      newExpense: {
        name: "",
        type: "",
        amount: '',
        due: ""
      },
      addexpense: true,
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
    /*expenses: expensesRef*/
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
    },/*
    edit(expense){
      console.log(expensesRef.child(expense['.key']))
    },*/
    reset(){
      this.newExpense.name = "";
      this.newExpense.type = "";
      this.newExpense.amount = "";
      this.newExpense.due = "";
    }    
  }
}; 
</script>


<style>

/* Buttons */
button {
  background-color: transparent;
  border-radius:3px; 
  font-size: 15px;
  outline: none;
}
button:active, button:checked,{
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

input[type="date"]{
  display: block;
  width: 50%;
  padding: 8px;
  border: 1px solid #888;
  border-radius: 3px;
}
</style>