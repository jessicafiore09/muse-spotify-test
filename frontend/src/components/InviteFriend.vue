<template>
  <br>
  <br>

  
  <div class="container">
    <div class="row mx-auto d-flex align-items-center justify-content-center">
      <div class="col">
        <div class="rounded-circle bg-dark d-flex align-items-center justify-content-center mx-auto"
          style="height: 150px; width: 150px;">
          <span class="text-white" style="font-size: 3em;">you</span>
        </div>
        <div class="ml-5 d-flex align-items-center justify-content-center">
          {{ email }}
        </div>
      </div>

      <div class="col" v-for="(friend, idx) in friends">
        <div class="rounded-circle bg-dark d-flex align-items-center justify-content-center mx-auto"
          style="height: 150px; width: 150px;">
          <span class="text-white" style="font-size: 10px;">{{ friend }}</span>
        </div>
        <div class="d-flex align-items-center justify-content-center">
          Friend {{ idx+1 }}
        </div>
        <div>
          <button class="btn btn-light btn-outline-dark" v-bind:data-friend="friend" v-on:click="deleteFriend(friend)">Delete this friend!</button>
        </div>
        
      </div>

      <div class="col" v-for="i in num" data-bs-toggle="modal" data-bs-target="#btn1Modal">
        <div class="rounded-circle bg-dark d-flex align-items-center justify-content-center mx-auto"
          style="height: 150px; width: 150px;">
          <span class="text-white font-weight-bold" style="font-size: 3em;">+</span>
        </div>
      </div>

    </div>
  </div>

  <div class="modal fade" id="btn1Modal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="btn1ModalLabel">Invite!</h5>
          <button @click="closeModal" type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          <form>
            <div class="mb-3">
              <label for="exampleInputEmail1" class="form-label">Email address</label>
              <input v-model="friend_email" type="email" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp">
            </div>
          </form>
        </div>
        <div class="modal-footer">
          <div v-if="feedbackMessage" class="alert alert-dismissible alert-info">{{ feedbackMessage }}</div>
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
          <button @click="addFriend" type="button" class="btn btn-primary">Send!</button>
        </div>
      </div>
    </div>
  </div>

  <!-- <div v-if="UserDetails" @click="toggleUserView(UserDetails.name, UserDetails.id)">

  </div> -->
</template>

<script>
import axios from 'axios'
const email = localStorage.getItem('email')

export default {
  props: ["group_name"],

  data() {
    return {
      email: "",
      friends: [],
      friend_email: "",
      feedbackMessage: ""
    };
  },

  created() {
    this.email = email
    this.loadFriends()
  },

  methods: {
    closeModal() {
      if (this.feedbackMessage) {
        location.reload(); // Reload the page
        this.feedbackMessage = ''; // Reset feedbackMessage to ''
      } else {
        this.feedbackMessage = ''; // Just reset feedbackMessage without reloading the page
      }
    }, 
    
    loadFriends() {
      axios.get('http://localhost:8000/api/v1/get_friends', {
        headers: {
          'Email': `${email}`,
          "group_name": this.group_name
        }
      })
        .then((response) => {
          console.log(response.data)

          this.friends = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },

    addFriend() {
      axios.post('http://localhost:8000/api/v1/add_friend', {
        friend_email: this.friend_email,
        group_name: this.group_name
      })
        
        .then(response => {
          console.log(response.data)

          // If the API call succeeds, clear the email input and show success feedback
          this.friend_email = '';
          this.feedbackMessage = 'Friend added successfully!';
        })
        .catch(error => {
          // If the API call fails, show an error message based on the response status code
          if (error.response && error.response.status === 404) {
            this.feedbackMessage = 'Friend is already in the group!';
          } else {
            this.feedbackMessage = 'An error occurred while adding the friend.';
          }
        });
    },

    deleteFriend(friend) {
      axios.post('http://localhost:8000/api/v1/remove_friend', 
      {
        "group_name": this.group_name,
        "friend_email": friend
      })
      .then(response => {
        console.log('friend removed? ' + response.data)
        setTimeout(function(){
          window.location.reload();
        }, 3000);
      })
      .catch(error => {
        console.log(error)
      })
    }
  },

  computed: {
    num() {
      return 4 - this.friends.length;
    }
  }
}

// export default{
//   props:['UserDetails'],

//   methods:{
//     toggleUserView = router.push(UserDetails)
//   }
// }

</script>