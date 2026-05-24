
<template class="background">
  <div class="background">
    <div class="logo"> 
      <img @mouseup.left="onHomeScreen" class="face-icon" src="/face.png" alt="Face" />
      SCRABBL
    </div>
    <div v-if="!isLoggedIn" class="background">
      <LoginScreen :onLogin="handleLogin" />
    </div>

    <div v-else class="background">
      <div class="gamearea">
        <LeaderBoard :players="players" />
        <DrawBoard />
      </div>
    </div>
  </div>
</template>


<script>
import DrawBoard from './DrawBoard.vue'
import LeaderBoard from './LeaderBoard.vue'
import LoginScreen from './LoginScreen.vue'



export default {
  name: 'MainGame',
  components: {
    DrawBoard,
    LeaderBoard,
    LoginScreen,
  },
  methods: {
    handleLogin(userData) {
      console.log('Logged in:', userData)
      
      if (userData.mode === "private") {
        this.isLoggedIn = true
      }
    },

    onHomeScreen() {
      this.isLoggedIn = false
    },
  },

  data() {
    return {
      isLoggedIn: false,
      
      lobby: {
        id: 'lobby123',
        name: 'Fun Lobby',
        players: ['Player 1', 'Player 2', 'Player 3']  
      },

      players: {
        currentIndex: 0,
        stats: [
          { name: 'Player 1', score: 0 },
          { name: 'Player 2', score: 0 },
          { name: 'Player 3', score: 0 }
        ]
      },
    }
  }
}
</script>

<style>

.logo {
  font-size: 22px;
  color: #fff;
  background-color: #47aa65;
  height: 44px;
  justify-content: center;
  align-items: center;
  display: flex;
}
.background {
  background-color: #edff9d;
  height: 100%;
  width: 100%;
}

.face-icon {
  width: 30px;
  height: 30px;
  object-fit: contain; /* keeps aspect ratio without distortion */
}

.gamearea {

  display: flex;
  justify-content: center;
  align-items: center;
}
</style>