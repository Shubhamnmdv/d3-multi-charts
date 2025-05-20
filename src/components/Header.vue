<template>
    <header class="header">
      <div class="title">Analytics</div>
      <div class="date-select">May 2023 ▼</div>
    </header>
  </template>
  
  <script>
  export default {
    name: "Header"
  };
  </script>
  
  <style scoped lang="scss">
  @import "@/assets/styles/variables.scss";
  
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: $background-color;
    color: $text-color;
    padding: 20px;
  
    .title {
      font-size: 24px;
      font-weight: bold;
    }
  
    .date-select {
      background: $card-background;
      padding: 8px 14px;
      border-radius: $border-radius;
      cursor: pointer;
      font-size: 14px;
    }
  }
  </style>
  