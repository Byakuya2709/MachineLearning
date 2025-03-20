<template>
  <div class="container">
    <h2>Teachable Machine Image Classifier</h2>
    
    <input type="file" accept="image/*" @change="handleImageUpload" />

    <div v-if="imageSrc" class="image-container">
      <img :src="imageSrc" alt="Selected Image" ref="imageRef" />
    </div>

    <button @click="predictImage" :disabled="!imageSrc">Dự đoán</button>

    <div v-if="predictions.length" class="results">
      <h3>Kết quả dự đoán:</h3>
      <ul>
        <li v-for="(prediction, index) in predictions" :key="index">
          {{ prediction.className }} - {{ (prediction.probability * 100).toFixed(2) }}%
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import * as tmImage from "@teachablemachine/image";

export default {
  data() {
    return {
      model: null,
      maxPredictions: 0,
      imageSrc: null,
      predictions: [],
      URL: "https://teachablemachine.withgoogle.com/models/2jyArQKf6/", // Thay bằng đường dẫn model của bạn
    };
  },
  async mounted() {
    await this.loadModel();
  },
  methods: {
    async loadModel() {
      try {
        const modelURL = this.URL + "model.json";
        const metadataURL = this.URL + "metadata.json";

        this.model = await tmImage.load(modelURL, metadataURL);
        this.maxPredictions = this.model.getTotalClasses();
        console.log("Model loaded successfully");
      } catch (error) {
        console.error("Lỗi tải model:", error);
      }
    },
    handleImageUpload(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          this.imageSrc = e.target.result;
        };
        reader.readAsDataURL(file);
      }
    },
    async predictImage() {
      if (!this.model || !this.imageSrc) return;
      
      await this.$nextTick(); // Đợi ảnh hiển thị
      const imgElement = this.$refs.imageRef;

      try {
        const prediction = await this.model.predict(imgElement);
        this.predictions = prediction.map(p => ({
          className: p.className,
          probability: p.probability,
        }));
      } catch (error) {
        console.error("Lỗi dự đoán:", error);
      }
    },
  },
};
</script>

<style scoped>
.container {
  text-align: center;
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  background-color: #f9f9f9;
}
h2 {
  color: #333;
}
input[type="file"] {
  margin-top: 10px;
}
.image-container img {
  max-width: 100%;
  max-height: 200px;
  margin-top: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}
button {
  margin-top: 10px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  transition: background-color 0.3s;
}
button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
button:hover:not(:disabled) {
  background-color: #0056b3;
}
.results {
  margin-top: 20px;
  text-align: left;
}
.results h3 {
  color: #333;
}
.results ul {
  list-style-type: none;
  padding: 0;
}
.results li {
  background-color: #fff;
  padding: 10px;
  margin-bottom: 5px;
  border: 1px solid #ddd;
  border-radius: 4px;
}
</style>
