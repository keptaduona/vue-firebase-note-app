<template>
<div @click="modal_close($event)" id="app">

    <a class="waves-effect waves-light btn-large button flash-message">Message</a>

    <nav v-if="signedIn">
        <div class="nav-wrapper cyan accent-4">
            <a href="#" class="brand-logo left">Notes</a>
            <ul id="nav-mobile" class="right">
                <li>{{user.email}}</li>
                <li><button @click="logout" class="waves-effect waves-light deep-orange accent-2 btn">Log Out</button></li>
            </ul>
        </div>
    </nav>

    <div v-if="!signedIn" class="auth">

      <div class="option-buttons">
          <h3>Notes App</h3>
          <a @click="modal_show('modal_login')" class="waves-effect waves-light cyan accent-4 btn-large button welcome-login">Log In</a>
          <a @click="modal_show('modal_signup')" class="waves-effect waves-light cyan accent-4 btn-large button welcome-signup">Sign Up</a>
      </div>

      <div id="modal_login" class="modal">
        <div class="modal-content">
          <h4>Log In</h4>
          <div class="login-form">
            <label for="email">Email</label>
            <input class="email" type="text" name="email" value="">
            <label for="password">Password</label>
            <input type="password" class="password">
            <button @click="login" type="button" class="btn cyan accent-4 register-login-button">Log in</button>
          </div>
        </div>
      </div>

      <div id="modal_signup" class="modal">
        <div class="modal-content">
          <h4>Sign Up</h4>
          <div class="register-form">
            <label for="email">Email</label>
            <input class="email" type="text" name="email" value="">
            <label for="password">Password</label>
            <input type="password" class="password">
            <label for="password-confirm">Confirm Password</label>
            <input type="password" class="password-confirm">
            <button @click="register" type="button" class="btn cyan accent-4" name="login">Register</button>
          </div>
        </div>
      </div>

      </div>


  <div v-if="signedIn" class="note-editor container">

    <a @click="createnote" class="btn-floating waves-effect waves-light red note-create-button"><i class="material-icons">add</i></a>

    <div class="existing-notes">
      <div v-for="note in reversedNotes" class="note" :id="note['.key']">
        <textarea @blur="text_area_blur($event)" @click="text_area_active($event)" class="note-text" rows="8" :value="note['.value']"></textarea>
        <div class="note-buttons">
            <a @click="deletenote(note)" class="btn-floating waves-effect waves-light red left"><i class="material-icons">delete</i></a>
            <a @click="editnote(note)" class="waves-effect waves-light btn green accent-4"><i class="material-icons">done</i></a>
        </div>
      </div>
    </div>

  </div>



</div>
</template>

<script>
import Firebase from 'firebase'

var autosize = require('autosize');

var config = {
  apiKey: "AIzaSyALFO6jYAdU7pYSA4p1Zk876TgsBUpnRk4",
  authDomain: "vue-notes-app.firebaseapp.com",
  databaseURL: "https://vue-notes-app.firebaseio.com",
  projectId: "vue-notes-app",
  storageBucket: "vue-notes-app.appspot.com",
  messagingSenderId: "1064422270444"
};

let app = Firebase.initializeApp(config);
const auth = Firebase.auth();

let db = app.database();

export default {
  name: 'app',
  beforeCreate: function() {
    // Our earliest lifecycle hook and first access
    // to $bindAsArray() / $bindAsObject() from VueFire
    // Start Firebase authentication here:
        auth.onAuthStateChanged(function(user) {
          if (user) {
            // Cache user - an anonymously authenticated firebase.User account
            //  - https://firebase.google.com/docs/reference/js/firebase.User
            this.user = user
            // Bind this instance's 'messages' property to the 'messages/${uid}'
            // Firebase reference via vuefire.js' $bindAsArray() method
            this.$bindAsArray('notes', db.ref('users/' + user.uid + '/notes'))
            // Note: Child component instances will have access to these
            // references via this.$root.user and this.$root.messages
            this.signedIn = true
            this.flash('Logged in', 'green accent-4');
        } else {
            if(this.signedIn) {
                this.signedIn = false;
            }
        }
        }.bind(this));
  },
  watch: {
      notes: function() {
          if(this.notes) {
              this.reversedNotes = this.notes.map((e, i, a)=> a[(a.length -1) -i]);
          }
      }
  },
  data() {
    return {
      signedIn: false,
      notes: [],
      reversedNotes: []
    }
  },
  methods: {
    register() {
      let email = $('.register-form .email').val();
      let password = $('.register-form .password').val();
      let passwordConfirm = $('.register-form .password').val();

      if (password == passwordConfirm) {
        auth.createUserWithEmailAndPassword(email, password)
          .catch(error => {
            if (error.code == 'auth/weak-password') {
              this.flash('Password has to be longer than 6 characters.', 'red accent-4');
            } else {
              this.flash(error.message, 'red accent-4');
            }
          });
      }
    },
    login() {
      let email = $('.login-form .email').val();
      let password = $('.login-form .password').val();

      auth.signInWithEmailAndPassword(email, password)
        .catch(error => this.flash(error.message, 'red accent-4'));
    },
    logout() {
      auth.signOut()
        .then(() => this.flash('Logged out', 'green accent-4'))
        .catch(error => this.flash(error.message, 'red accent-4'));
    },
    editnote(note) {
      var key = note['.key'];
      var noteText = $('#' + key + ' .note-text').val();
      this.$firebaseRefs.notes.child(note['.key']).set(noteText)
        .then(() => this.flash('Note updated', 'green accent-4'))
        .catch(error => this.flash(error.message, 'red accent-4'));
    },
    deletenote(note) {
      this.$firebaseRefs.notes.child(note['.key']).remove()
        .then(() => this.flash('Note deleted', 'green accent-4'))
        .catch(error => this.flash(error.message, 'red accent-4'));
    },
    createnote() {
      this.$firebaseRefs.notes.push("")
        .catch(error => this.flash(error.message, 'red accent-4'));
    },
    modal_show(modal) {
        $('#' + modal).fadeIn();
    },
    modal_close(event) {
        console.log(event.srcElement.className);
        if(!this.signedIn && event.srcElement.className == 'auth') {
            $('.modal').fadeOut();
        }
    },
    text_area_active(event) {
        let boxClicked = event.srcElement.parentElement.id;
        $('#' + boxClicked).fadeTo("fast", 1);
        autosize($('#' + boxClicked + ' textarea'));
        $('#' + boxClicked + ' .note-buttons').slideDown('fast');
    },
    text_area_blur(event) {
        let boxBlurred = event.srcElement.parentElement.id;
        $('#' + boxBlurred).fadeTo("fast", 0.5);
        $('#' + boxBlurred + ' .note-buttons').slideToggle('fast');
    },
    flash(message, color) {
        let colorArray = color.split(" ");
        if(!$('.flash-message').hasClass(colorArray[0]) || !$('.flash-message').hasClass(colorArray[1])) {
            $('.flash-message')
                .text(message)
                .addClass(color)
                .fadeTo('slow', 1.0)
                .delay(1000)
                .fadeTo('slow', 0, function() {
                    $(this).removeClass(color);
             });
        }
    }
  }
}
</script>

<style>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  height: 100%;
  width: 100%;
    overflow: auto;
    padding-right: 20px;
}

.auth {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}

.option-buttons {
    text-align: center;
}

input {
    text-align: center;
}

.modal {
    position: absolute;
    top: 25%;
    z-index: 999;
}

.note-editor {
    text-align: center;
    margin-top: 50px;
}

.note {
    width: 75%;
    margin: 0 auto;
    margin-bottom: 25px;
    outline: none;
    border: 1px solid #00b8d4;
    opacity: 0.5;
}

.note:hover {
    box-shadow: 0 14px 28px rgba(0,0,0,0.25), 0 10px 10px rgba(0,0,0,0.22);
}

.note-buttons {
    margin: 10px;
    display: none;
}

.register-login-button {
    margin: 25px;
}

nav {
    text-align: left;
}

nav ul li {
    margin-right: 15px;
}

textarea {
    background-attachment: local;
    background-image:
        linear-gradient(to right, white 10px, transparent 10px),
        linear-gradient(to left, white 10px, transparent 10px),
        repeating-linear-gradient(white, white 30px, #ccc 30px, #ccc 31px, white 31px);
    line-height: 31px;
    padding: 8px 10px;
    border: none;
    outline: none;
    -webkit-box-shadow: none;
    -moz-box-shadow: none;
    box-shadow: none;
    overflow: hidden;
    resize: none;
    height: 200px;
}

.note-create-button {
    margin-bottom: 25px;
}

.register-form, .existing-notes {
    display: block !important;
}

.modal-content {
    opacity: 1 !important;
}

.flash-message {
    box-shadow: none;
    position:absolute;
    width: auto;
    bottom: 0;
    left: 0;
    right: 0;
    margin: 0 auto;
    z-index: 9999;
    display: none;
}

</style>
