<template>
    <div style="width: 100%; height: 100%">
        <div ref="map" id="map" style="width: 100%; height: 100%">
            <div id="popup"></div>
        </div>
        <div class="text-center">
            <v-dialog
                :dark="dark"
                v-model="dialog"
                width="500"
            >
                <PopUpMarker
                    v-if="dadosPopUp"
                    :dados="dadosPopUp"
                />
            </v-dialog>
        </div>
    </div>
</template>

<script>
import Map from 'ol/Map'
import 'ol/ol.css'
import View from 'ol/View'
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'
import { transform } from 'ol/proj.js';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point'
import {Icon, Style} from 'ol/style';
import VectorSource from 'ol/source/Vector';
import VectorLayer from 'ol/layer/Vector';
import Overlay  from 'ol/Overlay';

import PopUpMarker from './PopUpMarker.vue';


const urlStationType = 'http://localhost:3000/station_type';

export default {
    name: 'MapOpenLayers',
    components: {
        PopUpMarker
    },
    props: {
        markers: Array,
        dark: Boolean,
    },
    data: () => ({
        tiposEstacoes: [],
        layerAntigo: null,
        overlayerAntigo: null,
        dialog: false,
        dadosPopUp: {},
        mapOl: null,
    }),
    mounted() {
        this.initMap()
        this.getListaTiposEstacoes()
    },
    watch:{
        markers(a){
            this.getMarkers(a)
        }
    },
    methods:{
        initMap(){
            this.mapOl = new Map({
                target: 'map',
                layers: [
                    new TileLayer({
                        source: new OSM()
                    }),
                ],
                view: new View({
                    zoom: 6,
                    center: transform([-51.245200, -24.632100], 'EPSG:4326', 'EPSG:3857'),
                    constrainResolution: true
                }),
            })
        },
        getMarkers(markers){
            if(this.overlayerAntigo){
                this.mapOl.removeOverlay(this.overlayerAntigo);
                this.overlayerAntigo = null;
            }
            if(this.layerAntigo){
                this.mapOl.removeLayer(this.layerAntigo);
                this.layerAntigo = null;
            }
            var features = [];

            markers.forEach(item =>{
                var iconFeature = new Feature({
                    geometry: new Point(transform([item.longitude, item.latitude], 'EPSG:4326', 'EPSG:3857')),
                    dados: item
                });

                const iconStyle = new Style({
                    image: new Icon({
                        anchor: [0.5, 46],
                        anchorXUnits: 'fraction',
                        anchorYUnits: 'pixels',
                        src: `icons/pin_icon_${this.getColor(item.station_type_id)}.png`,
                    }),
                });
                
                iconFeature.setStyle(iconStyle);
                features.push(iconFeature);
            })

            var vectorSource = new VectorSource({
                features: features
            });

            var vectorLayer = new VectorLayer({
                source: vectorSource
            });

            this.mapOl.addLayer(vectorLayer);
            this.layerAntigo = vectorLayer;

            const element = document.getElementById('popup');

            const popup = new Overlay({
                element: element,
                positioning: 'bottom-center',
                stopEvent: false,
            });

            this.mapOl.addOverlay(popup);
            this.overlayerAntigo = popup

            let functionSetPopUpOn = this.setPopUpOn

            this.mapOl.on('click', function (evt) {
                const feature = this.forEachFeatureAtPixel(evt.pixel, function(feature) {
                    return feature;
                });
                if (feature) {
                    functionSetPopUpOn(feature.values_)
                }
            });
            // change mouse cursor when over marker
            let mudarCursor = this.changeCursorStyle
            this.mapOl.on('pointermove', function (e) {
                const pixel = this.getEventPixel(e.originalEvent);
                const hit = this.hasFeatureAtPixel(pixel);
                hit ? mudarCursor(this.getTarget(), 'pointer') : mudarCursor(this.getTarget(), '')
            });
        },
        setPopUpOn(dados){
            this.dadosPopUp = dados.dados;
            this.dialog = true;
        },
        changeCursorStyle(target, styleCursor){
            this.$refs[target].style.cursor = styleCursor
        },
        async getListaTiposEstacoes(){
            await fetch(urlStationType).then(res =>{
                return res.json();
            }).then(data =>{
                this.tiposEstacoes = data
            })
        },
        getColor(tipo){
            let tipoEncontrado = this.tiposEstacoes.find(item => item.id === tipo)

            return tipoEncontrado.color.replace(/#/g, '')
        },
    }
}
</script>




