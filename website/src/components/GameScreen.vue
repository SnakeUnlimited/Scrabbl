
<template>
  <div>
    <div v-if="!isLoggedIn">
      <LoginScreen :onLogin="handleLogin" />
    </div>

    <div v-else>
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
    }
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
.gamearea {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>