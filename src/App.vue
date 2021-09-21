<template>
  <ul id="nav">
    <li><router-link to="/">ICU Beds Per US County</router-link></li>
    <li>
      <router-link to="/states">US State Infection Rates Over Time</router-link>
    </li>
    <li>
      <router-link to="/county-metrics">US County Metrics Datagrid</router-link>
    </li>
    <li>
      <router-link to="/sankey">Sankey Diagram of Vaccine Data</router-link>
    </li>
  </ul>
  <router-view />
</template>

<style lang="scss">
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.container {
  max-width: 900px;
}

#nav {
  background: #333;

  list-style-type: none;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;

  a {
    display: inline-block;
    font-weight: bold;
    color: #eee;
    transition: 1s;
    text-decoration: none;
    padding: 30px;

    &.router-link-exact-active {
      color: #42b983;
      background: #444;
    }
    &:hover {
      background: #555;
      color: #ccc;
    }
  }
}
@media screen and (max-width: 780px) {
  #nav {
    flex-direction: column;
    display:block;

    li {
      flex: 1;
    }

    a {
      display:block;
      padding:15px;
      
    }
  }
}
</style>
