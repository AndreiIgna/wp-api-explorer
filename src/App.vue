<template>
  <div id="app">
    
    <nav class="navbar navbar-expand-lg fixed-top">
      <router-link to="/" class="navbar-brand">WordPress API explorer</router-link>
      <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">👋 Menu</button>

      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav">
          <!-- <li class="nav-item mr-1">
            <router-link to="/history" class="nav-link">History</router-link>
          </li> -->
        </ul>

        <ul class="navbar-nav ml-auto">
          <li class="nav-item ml-4">
            <a class="nav-link" href="https://github.com/AndreiIgna/wp-api-explorer#readme">Docs</a>
          </li><li class="nav-item ml-4">
            <a class="nav-link" href="https://github.com/AndreiIgna/wp-api-explorer">GitHub</a>
          </li>
        </ul>
      </div>
    </nav>

    <div id="content">
      <router-view/>
    </div>

  </div>
</template>

<style lang="scss">
@import '~@layered/layered-design';

#app {

}

</style>
