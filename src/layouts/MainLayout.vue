<template>
  <q-layout view="lHr lpR fFf">
    <q-header bordered class="bg-white text-black relative-p" style="z-index:2;width:65%;">
      <q-toolbar>
        <q-btn dense flat round icon="menu" @click="left = !left" />

        <q-toolbar-title class="text-weight-bold">
          ASM
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-drawer class="left-drawer" show-if-above v-model="left" side="left" bordered :width="283">
      <!-- <q-icon name="fas fa-circle" size="lg" style="height: 60px; max-width: 150px" color="negative" class="q-ma-md"/> -->
      <img src="logo/favicon-96x96.png" style="height: 60px; max-width: 60px" class="q-ma-md" />
      <q-list class="u-list-nav">
        <q-item clickable v-ripple to="/">
          <q-item-section avatar>
            <q-icon color="primary" name="home" size="md" />
          </q-item-section>

          <q-item-section class="text-h6 text-weight-bold">Home</q-item-section>
        </q-item>

        <q-item clickable v-ripple to="/about">
          <q-item-section avatar>
            <q-icon name="help" size="md" />
          </q-item-section>

          <q-item-section class="text-h6 text-weight-bold">About</q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-drawer
      class="right-drawer"
      v-model="right"
      side="right"
      bordered
      show-if-above
    >
      <q-input
        label="Search"
        class="q-ma-md fixed"
        outlined
        rounded
        dense
      >
        <template v-slot:prepend>
          <q-icon name="search" />
        </template>
      </q-input>
    <q-scroll-area class="fit">      

      <q-list padding separator style="margin-top:50px;">
        <q-item class="q-pa-md">
          <q-item-section>
            <q-item-label overline class="text-grey">OVERLINE</q-item-label>
            <q-item-label class="text-weight-bold">Single line item</q-item-label>
            <q-item-label caption>Secondary line text. Lorem ipsum dolor sit amet, consectetur adipiscit elit.</q-item-label>
          </q-item-section>

          <q-item-section side top>
            <q-item-label caption>5 min ago</q-item-label>
          </q-item-section>
        </q-item>

        <q-item class="q-pa-md">
          <q-item-section>
            <q-item-label overline class="text-grey">OVERLINE</q-item-label>
            <q-item-label class="text-weight-bold">Single line item</q-item-label>
            <q-item-label caption>Secondary line text. Lorem ipsum dolor sit amet, consectetur adipiscit elit.</q-item-label>
          </q-item-section>

          <q-item-section side top>
            <q-item-label caption>5 min ago</q-item-label>
          </q-item-section>
        </q-item>

        <q-item class="q-pa-md">
          <q-item-section>
            <q-item-label overline class="text-grey">OVERLINE</q-item-label>
            <q-item-label class="text-weight-bold">Single line item</q-item-label>
            <q-item-label caption>Secondary line text. Lorem ipsum dolor sit amet, consectetur adipiscit elit.</q-item-label>
          </q-item-section>

          <q-item-section side top>
            <q-item-label caption>5 min ago</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-scroll-area>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
export default {
  data() {
    return {
      left: false,
      right: false
    };
  }
};
</script>
