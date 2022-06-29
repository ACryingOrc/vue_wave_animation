<template>
  <div>
    <Canvas
      :scatter-start="0"
      :scatter-end="25"
      :y-from="500"
      :y-to="300"
      :lines="3"
      :opacity="0.5"
      :colors="colors"
      :speed="200"
      :amplitude="{ type: 0, value: 150, y: 500 }"
    />
  </div>
</template>

<script>
import Canvas from "@/components/Canvas";
export default {
  name: "App",
  components: { Canvas },
  data() {
    return {
      colors: [
        { color: "#6a11cb", colorStop: 0.6 },
        { color: "#2575fc", colorStop: 1 },
      ],
    };
  },
};
</script>
