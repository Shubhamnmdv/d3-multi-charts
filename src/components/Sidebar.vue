<template>
  <aside :class="['sidebar', { collapsed }]">
    <!-- Move user section here -->
    <div class="user-profile">
      <div class="user" v-if="!collapsed">
        <img src="https://i.pravatar.cc/40" alt="avatar" class="avatar" />
        <div>
          <div class="name">Shubham Namdev</div>
          <div class="settings">Software Engineer | Data Visualization Enginner</div>
        </div>
      </div>
      <!-- <button class="template-btn" v-if="!collapsed">Get template →</button> -->
    </div>

    <!-- Navigation menu -->
    <nav>
      <ul>
        <li v-for="item in navItems" :key="item.label" :class="{ active: item.active }">
          <i :class="item.icon"></i>
          <span v-if="!collapsed">{{ item.label }}</span>
        </li>
      </ul>
    </nav>

    <!-- Collapse Button only -->
    <div class="collapse-container">
      <button class="collapse-btn" @click="toggleCollapse">
        <i :class="collapsed ? 'fas fa-chevron-right' : 'fas fa-chevron-left'"></i>
      </button>
    </div>
  </aside>
</template>


<script>
export default {
  name: "Sidebar",
  data() {
    return {
      collapsed: false,
      navItems: [
        { label: "Details", icon: "fas fa-user-circle", active: true },
        { label: "D3 Charts", icon: "fas fa-chart-bar" },
        { label: "High Charts", icon: "fas fa-chart-bar" },
        { label: "Features", icon: "fas fa-star" },
        { label: "Users", icon: "fas fa-user" },
        { label: "Pricing", icon: "fas fa-dollar-sign" },
      ]
    };
  },
  methods: {
    toggleCollapse() {
      this.collapsed = !this.collapsed;
    }
  }
};
</script>

<style scoped lang="scss">
// @import "@/assets/styles/variables.scss";
@import "/src/assets/styles/variables.scss";

.sidebar {
  background: $card-background;
  color: $text-color;
  padding: 20px;
  height: 100vh;
  width: 250px;
  transition: width 0.3s;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  overflow: hidden;

 &.collapsed {
  width: 80px;

  .user-profile {
    .user {
      justify-content: center;

      .avatar {
        margin-right: 0;
      }

      .name,
      .settings {
        display: none;
      }
    }
  }

  nav ul li {
    justify-content: center;

    i {
      margin-right: 0;
    }

    span {
      display: none;
    }
  }

  .collapse-container {
    align-items: center;
  }
}


  .logo {
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 30px;
  }

  nav ul {
    list-style: none;
    padding: 0;

    li {
      display: flex;
      align-items: center;
      padding: 10px;
      margin-bottom: 10px;
      border-radius: $border-radius;
      cursor: pointer;
      transition: background 0.3s;

      i {
        margin-right: 10px;
      }

      &.active,
      &:hover {
        background: $primary-color;
      }
    }
  }

  .bottom-section {
    display: flex;
    flex-direction: column;
    align-items: stretch;
    gap: 10px;

    .user {
      display: flex;
      align-items: center;

      .avatar {
        border-radius: 50%;
        margin-right: 10px;
      }

      .name {
        font-weight: bold;
      }

      .settings {
        font-size: 12px;
        color: $text-muted;
      }
    }

    .collapse-btn {
      background: none;
      border: none;
      color: $text-color;
      font-size: 18px;
      cursor: pointer;
      align-self: flex-end;
    }
  }
}

.user-profile {
  margin-bottom: 30px;

  .user {
    display: flex;
    align-items: center;

    .avatar {
      border-radius: 50%;
      margin-right: 10px;
    }

    .name {
      font-weight: bold;
    }

    .settings {
      font-size: 12px;
      color: $text-muted;
    }
  }
}

.collapse-container {
  margin-top: auto;
  .collapse-btn {
    background: none;
    border: none;
    color: $text-color;
    font-size: 18px;
    cursor: pointer;
    align-self: flex-end;
  }
}

</style>