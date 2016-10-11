# _zapi
Another way to do APIs

## status

Draft :) 

# Node

## Setup

Just tell Zapi how to **get**, **create**, **update** or **delete** resources in your "database"

```javascript

var zapi = require('zapi');
var express = require('express');
var app = express();
var emailHelper = require('./helpers/email');

var api = zapi({
  users:{
    fields:{
      email:true,
      firstname:true,
      lastname:{ default:'' },
      products:{ rel:'products', multiple:true },
    },

    get:function(options){
      return new Promise(function(resolve){
        resolve({
          email:'qzapaia@gmail.com',
          firstname:'m',
          lastname:'z',
        });
      });    
    },
    
    create:function(){
      return new Promise(function(resolve){
        resolve({
          message:"this will be returned as extra data, it doesn't change the state on the client"
        })
      });
    },
    update:function(){},
    delete:function(){},
    effects:{
      create:function(newUser){
        emailHelper.send({
          to:newUser.email,
          message:`Welcome ${newUser.firstname}`
        })
      }
    }
  },
  products:{
    owner:{ rel:'users' }
  }
});

// programmatically
api.pull({
  users:{
    me:true,
    populate:{
      products:{
        owner:true
      },
    },
  },
}).then(function(me){
  // receive me with expanded products
  // recreate the users 'create' side effect
  api.users._effects.create(me);
})

app.use('/api/v1/',api.middleware());

```

# Browser


## Basic

```html
    <script src="http://api.server.com/api/v1/_client.js"></script> 
    <script>
      zapi.pull({
        users:{
          me:true,
          populate:{
            products:true
          }
        },
      }).then(function(){
        console.log(zapi.state);
      });
    </script>
  </body>

```

## Pushing data

```html
    <form id="update-profile">
      <input name="firstname" />
      <button>Save</button>
    </form>
    
    <script src="http://api.server.com/api/v1/_client.js"></script> 
    <script>      
      document.getElementById('update-profile').addEventListener('submit', function(e){
        e.preventDefault();
        
        zapi.set({
          users:{
            me:{
              firstname:this.firstname.value
            }
          }
        })
        .push()
        .then(function(response){
          console.log(response);
        })
        .catch(function(){
          console.error(e);
        });
      })
    </script>
  </body>

```
