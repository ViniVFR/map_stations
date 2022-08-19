<template>
    <div >
        <v-app id="app" style="width: 100%; height: 100%;">
            <v-navigation-drawer
                style="min-width:250px"
                v-model="drawer"
                app
                :dark="dark"
            >
                <FormFiltros
                    @filtroMarkers="markers = $event, drawer = false;"
                />
            </v-navigation-drawer>

            <v-app-bar app :dark="dark">
                <v-btn 
                    icon
                    @click="drawer = !drawer"
                >
                    <v-icon>mdi-magnify</v-icon>
                </v-btn>

                <v-toolbar-title class="cyan--text">Localizar Estações</v-toolbar-title>
                <v-spacer></v-spacer>

                <v-switch
                    class="pt-5"
                    v-model="dark"
                    inset
                >
                    <template v-slot:label>
                        <v-icon v-if="dark">mdi-weather-night</v-icon>
                        <v-icon v-else>mdi-white-balance-sunny</v-icon>
                    </template>
                </v-switch>
            </v-app-bar>

            <v-main :dark="dark">
                <MapOpenLayers :dark="dark" :tiposEstacoes="tiposEstacoes" :markers="markers"/>
            </v-main>
        </v-app>
    </div>
</template>

<script>
import FormFiltros from './components/FormFiltros';
import MapOpenLayers from './mapa/MapOpenLayers';

export default {
    name: 'App',

    components: {
        FormFiltros,
        MapOpenLayers
    },

    data: () => ({
        markers: [],
        tiposEstacoes: [],
        drawer: false,
        dark: false
    }),
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
