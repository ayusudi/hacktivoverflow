<template>
  <div class="row">
    <div class="vote">
      <button @click="upVote">
        <i class="fas fa-caret-up fa-3x"></i>
      </button>
      <hr />
      <h1>{{totalVote}}</h1>
      <hr />
      <button @click="downVote">
        <i class="fas fa-caret-down fa-3x"></i>
      </button>
    </div>
    <div class="one">
      <div class="authz" v-if="authz">
        <button @click="editQuestion">
          <i class="fas fa-pencil-alt fa-2x"></i>
        </button>
        <button @click="deleteMe">
          <i class="fas fa-trash-alt fa-2x"></i>
        </button>
      </div>
      <p class="date mini">{{createdAt}}</p>
      <h1 class="title" @click="readThis">{{question.title}}</h1>
      <div v-html="question.description"></div>
      <br />
      <p class="normal">{{question.userId.name}}</p>
      <p class="mini">{{timePass}}</p>
    </div>
  </div>
</template>

<script>
import { mapActions, mapState } from 'vuex'
import axios from 'axios'
import Swal from 'sweetalert2'

export default {
  props: ['question'],
  data () {
    return {
      createdAt: '',
      timePass: '',
      baseUrl: 'http://3.0.89.125',
      totalVote: 0,
      authz: false,
      up: false,
      down: false
    }
  },
  methods: {
    upVote () {
      // this.checkMyVote()
      if (!localStorage.token) {
        this.$emit('toLogin')
      } else {
        if (!this.down) {
          axios({
            method: 'put',
            url: `${this.baseUrl}/question/like/${this.question._id}`,
            headers: {
              token: localStorage.token
            }
          })
            .then(data => {
              this.updateVote()
            })
            .catch(err => {
              if (err.response.data.message === 'you cannot like your own question!') {
                Swal.fire({
                  title: 'Error',
                  text: 'you cannot vote your own question!',
                  type: 'error',
                  confirmButtonText: 'Oke'
                })
              }
              this.normalize()
            })
        } else {
          this.normalize()
        }
      }
    },
    downVote () {
      if (!localStorage.token) {
        this.$emit('toLogin')
      } else {
        if (!this.up) {
          axios({
            method: 'put',
            url: `${this.baseUrl}/question/dislike/${this.question._id}`,
            headers: {
              token: localStorage.token
            }
          })
            .then(data => {
              this.updateVote()
            })
            .catch(err => {
              if (err.response.data.message === 'you cannot dislike your own question!') {
                Swal.fire({
                  title: 'Error',
                  text: 'you cannot vote your own question!',
                  type: 'error',
                  confirmButtonText: 'Oke'
                })
              }
              // console.log(err.response);
              this.normalize()
            })
        } else {
          this.normalize()
        }
      }
    },
    normalize () {
      axios({
        method: 'put',
        url: `${this.baseUrl}/question/normalize/${this.question._id}`,
        headers: {
          token: localStorage.token
        }
      })
        .then(data => {
          // console.log("normalize");
          this.updateVote()
        })
        .catch(err => {
          console.log(err.response)
        })
    },
    updateVote () {
      axios({
        method: 'get',
        url: `${this.baseUrl}/question/${this.question._id}`
      }).then(data => {
        let likes = data.data.question.likes.length
        let dislikes = data.data.question.dislikes.length
        let total = likes - dislikes
        this.totalVote = total
        this.checkMyVote(data.data.question)
      })
    },
    checkAuthz () {
      if (localStorage.token) {
        if (this.question.userId._id === localStorage._id) {
          this.authz = true
        }
      }
    },
    editQuestion () {
      this.$router.push({ path: `/update/${this.question._id}` })
    },
    readThis () {
      this.$router.push({ path: `/question/${this.question._id}` })
    },
    checkMyVote (question) {
      this.up = false
      this.down = false
      for (let k = 0; k < question.likes.length; k++) {
        if (
          question.likes[k].userId === localStorage._id &&
          question.likes[k].userId
        ) {
          this.up = true
        }
      }
      for (let k = 0; k < question.dislikes.length; k++) {
        if (
          question.dislikes[k].userId === localStorage._id &&
          question.dislikes[k].userId
        ) {
          this.down = true
        }
      }
    },
    deleteMe () {
      axios({
        method: 'delete',
        url: `${this.baseUrl}/question/${this.question._id}`,
        headers: {
          token: localStorage.token
        }
      })
        .then(() => {
          Swal.fire({
            title: 'Success Delete',
            text: '',
            type: 'success',
            confirmButtonText: 'Oke'
          })
        })
        .catch(err => {
          console.log(err.data)
        })
    }
  },
  computed: {
    ...mapActions(['putToken']),
    ...mapState(['token'])
  },
  created () {
    this.createdAt = new Date(this.question.createdAt).toDateString()
    this.totalVote = this.question.likes.length - this.question.dislikes.length
    this.timePass = moment(new Date(this.question.createdAt)).fromNow()
    this.checkAuthz()
    this.updateVote()
  }
}
</script>

<style scoped>
.one {
  margin: 10px 10px 25px 10px;
  display: flex;
  justify-content: flex-start;
  flex-direction: column;
  align-items: flex-start;
  width: 80%;
}
.date {
  align-self: flex-end;
  margin-right: 10px;
}
.vote {
  width: 150px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}
.row {
  display: flex;
  justify-content: flex-start;
}
.authz {
  align-self: flex-end;
  margin-right: 10px;
}
.authz button {
  width: 30px;
  height: 30px;
  margin-right: 5px;
  margin-bottom: 6px;
}
h1 {
  font-size: 16pt;
  font-family: Arial, Helvetica, sans-serif;
  color: #0266d6;
}

.mini {
  font-size: 12px;
  font-family: Arial, Helvetica, sans-serif;
}

.vote button {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  align-self: center;
  color: rgb(12, 67, 131);
}

.vote h1 {
  font-size: 18pt;
  font-style: inline;
  color: rgb(77, 76, 76);
}
.vote hr {
  width: 40%;
  align-self: center;
  margin: 8px;
}

.title:hover {
  text-decoration: underline;
  cursor: pointer;
}
p.normal {
  color: rgb(87, 13, 156);
  font-weight: 450;
}
</style>
