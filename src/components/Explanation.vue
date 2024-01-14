<template>
  <div class="landing-page">
    <header>
      <h1>The Electric Car <br> Cost Calculator</h1>
      <p>Calculate the Energy Cost of Driving an Electric Car</p>
    </header>

    <section class="how-it-works">
      <h2>How It Works</h2>
      <div class="info-row">
        <div class="info-block" v-for="(step, index) in steps" :key="index">
          <div class="step-number">{{ index + 1 }}</div>
          <div class="step-content">
            <h3>{{ step.title }}</h3>
            <p>{{ step.description }}</p>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
export default {
  data() {
    return {
      steps: [
        { title: "Select Your Car", description: "Choose your electric car's make, model, and variant from our list." },
        { title: "Choose Your Location", description: "Pick your state for accurate energy cost based on local electricity rates." },
        { title: "Charging Method", description: "Decide between solar or power grid charging. Enter your cost per KWH." },
        { title: 'Hit "Calculate"', description: "Hit 'Calculate' to see your projected annual energy cost instantly." },
      ],
    };
  },
};
</script>

<style>
header{
  text-align: center;
}
header h1{
  font-family:Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
  color: #087c7c;
  font-size: 3em;
}

body{
  background: linear-gradient(to bottom right, #7fd7d7, #fcfeff); /* Replace these colors with your desired gradient */
  margin: 0;
  font-family:Arial, Helvetica, sans-serif
}

h2, h3 {
  color: #087c7c;
}

p {
  margin-bottom: 20px;
}

.how-it-works {
  margin-top: 40px;
  text-align: center;
}

.info-row {
  display: flex;
  justify-content: space-around;
  margin-top: 1em;
  padding: 0 10px;
  gap: 1.5em;
}

.info-block {
  flex: 1;
  text-align: center; 
}

.step-number {
  background-color: #087c7c;
  color: #fff;
  border-radius: 50%; 
  width: 50px; 
  height: 50px; 
  margin: 0 auto 10px; 
  display: flex;
  justify-content: center;
  align-items: center;
}

.step-content {
  max-width: 300px; /* Adjust the max-width as needed */
  margin: 0 auto;
}
.step-content h3 {
  white-space: nowrap;
}
</style>
