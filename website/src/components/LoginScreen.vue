<template>
  <div class="centered">
    <div class="login-box" >
        <h2 :style="{ backgroundColor: color, color: complementary }">Login</h2>

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

  computed: {
    complementary() {
      return getComplementaryColor(this.color);
    }
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

function getComplementaryColor(hex) {
  hex = hex.replace('#', '');

  let r = parseInt(hex.substring(0, 2), 16) / 255;
  let g = parseInt(hex.substring(2, 4), 16) / 255;
  let b = parseInt(hex.substring(4, 6), 16) / 255;

  const max = Math.max(r, g, b);
  const min = Math.min(r, g, b);

  let h, s, l = (max + min) / 2;

  if (max === min) {
    h = s = 0;
  } else {
    const d = max - min;
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min);

    switch (max) {
      case r: h = (g - b) / d + (g < b ? 6 : 0); break;
      case g: h = (b - r) / d + 2; break;
      case b: h = (r - g) / d + 4; break;
    }

    h /= 6;
  }

  // rotate hue by 180°
  h = (h + 0.5) % 1;

  // convert back to RGB
  function hue2rgb(p, q, t) {
    if (t < 0) t += 1;
    if (t > 1) t -= 1;
    if (t < 1/6) return p + (q - p) * 6 * t;
    if (t < 1/2) return q;
    if (t < 2/3) return p + (q - p) * (2/3 - t) * 6;
    return p;
  }

  let q = l < 0.5 ? l * (1 + s) : l + s - l * s;
  let p = 2 * l - q;

  let r2 = Math.round(hue2rgb(p, q, h + 1/3) * 255);
  let g2 = Math.round(hue2rgb(p, q, h) * 255);
  let b2 = Math.round(hue2rgb(p, q, h - 1/3) * 255);

  if (r2 < 100) {
    r2 += 100;
  }
  if (g2 < 100) {
    g2 += 100;
  }
  if (b2 < 100) {
    b2 += 100;
  }


  if (r2 > 200) {
    r2 -= 100;
  }
  if (g2 > 200) {
    g2 -= 100;
  }
  if (b2 > 200) {
    b2 -= 100;
  }
  

  return `rgb(${r2}, ${g2}, ${b2})`;
}
</script>

<style>
.centered {
  
  display: flex;
  justify-content: center;
  align-items: center;
  background-color:#fdffa3;
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