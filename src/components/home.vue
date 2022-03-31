<template>
  <v-card height="1000px">
    <v-toolbar
        dark
      >
        <v-app-bar-nav-icon 
                  @click.stop="drawer = !drawer"
        ></v-app-bar-nav-icon>

        <v-toolbar-title>CSE6242 Academic Graph</v-toolbar-title>

        <v-spacer></v-spacer>

        <v-btn icon>
          <v-icon>mdi-export</v-icon>
        </v-btn>
      </v-toolbar>
     <v-navigation-drawer
      v-model="drawer"
      absolute
      temporary
    >
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title>CSE6242 Team 9</v-list-item-title>
        </v-list-item-content>
      </v-list-item>

      <v-divider></v-divider>

      <v-list dense>
        <v-list-item
          v-for="(item, index) in items"
          :key="item.title"
          link
          @click="click(index)"
        >
          <v-list-item-icon>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title >{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <graph v-if="show == 0"></graph>
    <chart v-if="show == 1"></chart>
    <chartbar v-if="show == 2"></chartbar>
  </v-card>
</template>

<script>
import Chart from './chart.vue';
import Chartbar from './chartbar.vue';
import Graph from './graph.vue';


  export default {
    name: 'home',

    components: {
      Graph,
      Chart,
      Chartbar
    },
    data(){
      return {
        drawer: false,
        items: [
          { title: 'Heterogenous graphs', icon: 'mdi-graphql' },
          { title: 'Chart bar', icon: 'mdi-chart-bar' },
          { title: 'chart-areaspline', icon: 'mdi-chart-areaspline' },
        ],
        show: 0
      }
    },
    methods: {
      click: function(index) {
        this.show = index
      }
    }
  }
</script>
