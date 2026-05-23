<template>
  <div class="centered">
    <div class="login-box" >
        <h2 :style="{ backgroundColor: color }">Login</h2>

        <input
            v-model="displayName"
            type="text"
            placeholder="Display Name"
        />
        
        
        <div class="color-row">
            <input
                v-model="color"
                type="color"
            />
            <span>
                <label for="scales">Save Cookie</label>
                <input type="checkbox" id="scales" name="scales" checked />
            </span>
        </div>

        <button @click="playPublic">Play Public</button>
        <button @click="playPrivate">Play Private</button>
    </div>
  </div>
</template>

<script>

export default {
  name: "LoginScreen",
  data() {
    return {
      displayName: "",
      color: "#ffffff"
    };
  },

  props: {
    onLogin: Function
  },

  methods: {
    submitLogin(mode) {
      const payload = {
        displayName: this.displayName,
        color: this.color,
        mode
      };

      console.log(mode === "public" ? "Public game:" : "Private game:", payload);

      if (this.onLogin) {
        this.onLogin(payload);
      }
    },

    playPublic() {
      this.submitLogin("public");
    },
    playPrivate() {
      this.submitLogin("private");
    },
  }
};
</script>

<style>
.centered {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  justify-content: center;
  align-items: center;
}

.color-row {
  display: flex;
  align-items: center;
  gap: 10px;
  justify-content: center;
  align-items: center;
}

.login-box {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  min-width: 240px;
}

.color-preview {
  width: 100%;
  height: 20px;
  border: 1px solid #ddd;
  border-radius: 5px;
}
</style>