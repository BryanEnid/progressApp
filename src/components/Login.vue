<template>

<div id="app">
  <div id="loginint">

      
      

      <form id="loginForm">
          <img src="http://res.cloudinary.com/bryanenid/image/upload/v1526920804/progress-logo-compresed.png" alt="progress" class="logoLogin">
          <h2>Login or Register</h2>
        <span class="rel"><i class="fas fa-user-circle fa-inputFormLogin"></i><input type="text" placeholder="Username"></span> 
        <span class="rel"><i class="fas fa-key fa-inputFormLogin"></i><input type="text" placeholder="Password"></span>
        <input type="button" value="Register" class="register">
        <input type="button" value="Sign in" class="signin">   
      </form>

      <center class="repo"><a href="https://github.com/BryanEnid/progressApp">See this app repository</a> </center>

  </div>
</div>
  
</template>

<script>

  export default {

    data(){
      return {
        
      }
    }
  }
</script>

<style>

.rel {
    position: relative;
}

* {
    font-size: 100%;
    font-family: 'Montserrat', sans-serif;
}

#loginint center a {
    font-weight: bold;
    color: #00a99e;
}

#loginint center {
    text-align: center;
    width: 206px;
    position: absolute;
    top: 50%;
    left: 50%;
    margin-left: -100px;
    margin-top: 200px
}

.logoLogin {
    position: absolute;
    top: 70%;
    left: 50%;
    height: 100px;
    width: 100px;
    margin-left: -50px;
    margin-top: -400px;
    z-index: 500;
}

body::-webkit-scrollbar {
    width: 0 !important;
}


#app {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    padding: 10px;
}

#loginForm h2 {
    font-size: 150%;
    text-align: center;
    color: #777;
    margin-bottom: 30px;
    padding-bottom: 15px;
    border-bottom: 1px solid #777; 
}

body {
    background: #222;
}

#loginForm {
    padding: 20px 30px 30px 30px;
    border-radius: 3px;
    max-width: 400px;
    height: 375px;
    border: 1px solid #777;
    background: white;
    display: block;
    margin: 0 auto;
    position: absolute;
    top:50%;
    left:50%;
    margin-left:-200px;
    margin-top:-250px; 
}

#loginForm input[type="text"] {
    border-radius: 3px;
    width: 100%;
    border: 1px solid #888;
    padding-left: 40px;
    height: 40px;
    margin-bottom: 10px;
    outline-color: #00a99e;
    color: #333;
}

#loginForm input[type="button"] {
    width: 100%;
    margin-bottom: 20px;
    border-radius: 3px;
    background: #333;
    color: white;
    border:1px solid white;
    height: 45px;
}

#loginForm .register {
    margin-top: 40px; 
}


#loginForm .fa-inputFormLogin {
    position: relative;
    top: 32px;
    left: 14px;
    color: #999;
}


@media only screen and (max-width: 400px){

    #app #loginint {
        padding-top: 80px; 
    }

    #loginForm {
        position: static;
        margin-left:0;
        margin-top:0; 
    }

    #loginForm .logoLogin {
        top: 420px;
        transform: scale(0.7);
        background: #111;
        border: 25px solid #111;
        box-shadow: 0px 0px 10px black;
        border-radius:50%; 
    }

    #loginint center.repo {
        width: 100%;
        position: static;
        margin: 80px 0 20px 0;
    }
    
}

</style>