<template>
  <div
    class="relative h-screen w-screen bg-zinc-900 text-stone-300"
    @mousemove="handleMouseMove"
  >
    <div
      class="absolute top-0 left-0 w-full h-full pointer-events-none"
      :style="{ backgroundImage: gradientStyle }"
    ></div>
    <div class="flex flex-col items-center font-mono">
      <h1 class="text-2xl font-bold pt-10 text-pink-200">URL Shortener</h1>
      <form
        @submit.prevent="generateShortUrl"
        class="border rounded-md m-auto flex flex-col gap-5 p-10 w-96"
      >
        <input
          type="text"
          v-model="url"
          placeholder="Enter your URL"
          class="text-gray-950 placeholder-pink-500"
          required
        />
        <button
          type="submit"
          class="bg-rose-950 rounded-sm hover:bg-pink-800 transition-all duration-500 text-stone-50"
        >
          Shorten URL
        </button>
      </form>
      <p v-if="errorMessage">{{ errorMessage }}</p>
      <div v-if="shortUrl" class="mt-5">
        <p class="text-lg">Shorten URL:</p>
        <a
          :href="shortUrl"
          class="text-pink-600 no-underline hover:underline"
          target="_blank"
          >{{ shortUrl }}</a
        >
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      url: "",
      shortUrl: "",
      errorMessage: "",
      gradientStyle: "",
      animationFrameId: null,
      gradientSize: "500px",
    };
  },
  methods: {
    async generateShortUrl(){
      this.shortUrl = "";
      this.errorMessage = "";

      const apiUrl = "http://localhost:5000/api/shorten"

      try {
        const response = await fetch(apiUrl, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            longUrl: this.url,
          }),
        });
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        this.shortUrl = data.shortUrl;
      } catch (error) {
        this.errorMessage = "Error generating short URL. Please try again.";
        console.error("Error: ", error);
      }
    },
    handleMouseMove(event) {
      this.mouseX = event.clientX;
      this.mouseY = event.clientY;

      if (!this.animationFrameId) {
        this.updateGradient();
      }
    },
    updateGradient() {
      this.gradientStyle = `
        radial-gradient(circle ${this.gradientSize} at ${this.mouseX}px ${this.mouseY}px, rgba(249, 168, 212, 0.1), transparent)
      `;

      this.animationFrameId = requestAnimationFrame(this.updateGradient);
    },

    beforeDestroy() {
      if (this.animationFrameId) {
        cancelAnimationFrame(this.animationFrameId);
      }
    },
  },
};
</script>
