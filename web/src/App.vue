<template>
    <div class='viewport-full relative scroll-hidden'>
        <!-- Map -->
        <div id="map" class='h-full bg-darken10 viewport-twothirds viewport-full-ml absolute top left right bottom'></div>

        <!-- Left Panel -->
        <div class='absolute top-ml left bottom z1 w-full w240-ml hmax-full px12 py12-ml' style="pointer-events: none;">

            <!-- Toolbar -->
            <div class='bg-white round mb12' style='height: 40px; pointer-events:auto;'>
                <div @click="panel === 'deltas' ? panel = false : panel = 'deltas'"class='py12 bg-white bg-darken25-on-hover btn round color-gray-dark cursor-pointer' style='height: 40px; width: 40px;'>
                    <svg class='icon'><use href='#icon-clock'/></svg>
                </div>
                <div @click="panel === 'styles' ? panel = false : panel = 'styles'"class='py12 bg-white bg-darken25-on-hover btn round color-gray-dark cursor-pointer' style='height: 40px; width: 40px;'>
                    <svg class='icon'><use href='#icon-paint'/></svg>
                </div>
                <div @click="panel = false; modal = 'query'"class='py12 bg-white bg-darken25-on-hover btn round color-gray-dark cursor-pointer' style='height: 40px; width: 40px;'>
                    <svg class='icon'><use href='#icon-inspect'/></svg>
                </div>
                <div @click="panel === 'bounds' ? panel = false : panel = 'bounds'"class='py12 bg-white bg-darken25-on-hover btn round color-gray-dark cursor-pointer' style='height: 40px; width: 40px;'>
                    <svg class='icon'><use href='#icon-arrow-down'/></svg>
                </div>
                <div @click="panel = false; modal = 'settings'"class='py12 bg-white bg-darken25-on-hover btn round color-gray-dark cursor-pointer' style='height: 40px; width: 40px;'>
                    <svg class='icon'><use href='#icon-sprocket'/></svg>
                </div>
            </div>

            <template v-if='panel === "deltas"'>
                <deltas :map='map'/></template>
            <template v-else-if='panel === "bounds"'>
                <bounds/>
            </template>
            <template v-else-if='panel === "styles"'>
                <styles :credentials='credentials' v-on:style='modal = "style"; style_id = $event'/>
            </template>
            <template v-else-if='feature'>
                <feature :map='map' :id='feature' v-on:close='feature = false'/>
            </template>
        </div>

        <!-- Login Panel -->
        <div class='none block-ml absolute top-ml left bottom z1 ml240 hmax-full py12-ml' style="pointer-events: none;">
            <div class='bg-white round' style='height: 40px; pointer-events:auto;'>
                <div @click="panel = false; logout(); modal = 'login'"class='py12 bg-white bg-darken25-on-hover btn round color-gray-dark cursor-pointer' style='height: 40px; width: 40px;'>
                    <svg class='icon'><use href='#icon-user'/></svg>
                </div>
                <div v-if='credentials.authed' @click="logout(true)" class='py12 bg-white bg-darken25-on-hover btn round color-gray-dark cursor-pointer' style='height: 40px; width: 40px;'>
                    <svg class='icon'><use href='#icon-logout'/></svg>
                </div>
            </div>
        </div>

        <!-- Modal Opaque -->
        <div v-if='modal' class='absolute top left bottom right z2 bg-black opacity75' style="pointer-events: none;"></div>

        <!--Modals here-->
        <template v-if='modal === "login"'>
            <login
                v-on:login='modal = false; credentials.authed = true'
                v-on:close='modal = false'
                v-on:register='modal = "register"'
                v-on:username='credentials.username = $event'
                v-on:uid='credentials.uid = $event'
            />
        </template>
        <template v-else-if='modal === "register"'>
            <register v-on:close='modal = false' />
        </template>
        <template v-else-if='modal === "settings"'>
            <settings v-on:close='modal = false' />
        </template>
        <template v-else-if='modal === "query"'>
            <query v-on:close='modal = false' :credentials='credentials' />
        </template>
        <template v-else-if='modal === "style"'>
            <stylem v-on:close='modal = false' :id='style_id' :credentials='credentials' :map='map' />
        </template>
    </div>
</template>

<script>
// === Components ===
import Foot from './components/Foot.vue';

// === Panels ===
import Deltas from './panels/Deltas.vue';
import Feature from './panels/Feature.vue';
import Bounds from './panels/Bounds.vue';
import Styles from './panels/Styles.vue';

// === Modals ===
import Login from './modals/Login.vue';
import Register from './modals/Register.vue';
import Settings from './modals/Settings.vue';
import Query from './modals/Query.vue';
import Style from './modals/Style.vue';

export default {
    name: 'app',
    data: function() {
        return {
            credentials: {
                map: { key: 'pk.eyJ1Ijoic2JtYTQ0IiwiYSI6ImNpcXNycTNqaTAwMDdmcG5seDBoYjVkZGcifQ.ZVIe6sjh0QGeMsHpBvlsEA' },
                authed: false,
                username: '',
                uid: false
            },
            map: {
                gl: false,
                layers: [],
                default: function() {
                    const foregroundColor = '#FF0000';

                    this.layers.push('hecate-data-polygons');
                    this.gl.addLayer({
                        id: 'hecate-data-polygons',
                        type: 'fill',
                        source: 'hecate-data',
                        'source-layer': 'data',
                        filter: ['==', '$type', 'Polygon'],
                        paint: {
                            'fill-opacity': 0.1,
                            'fill-color': foregroundColor
                        }
                    });

                    this.layers.push('hecate-data-polygon-outlines');
                    this.gl.addLayer({
                        id: 'hecate-data-polygon-outlines',
                        type: 'line',
                        source: 'hecate-data',
                        'source-layer': 'data',
                        filter: ['==', '$type', 'Polygon'],
                        layout: {
                            'line-join': 'round',
                            'line-cap': 'round'
                        },
                        paint: {
                            'line-color': foregroundColor,
                            'line-width': 0.75,
                            'line-opacity': 0.75
                        }
                    })

                    this.layers.push('hecate-data-lines');
                    this.gl.addLayer({
                        id: 'hecate-data-lines',
                        type: 'line',
                        source: 'hecate-data',
                        'source-layer': 'data',
                        filter: ['==', '$type', 'LineString'],
                        layout: {
                            'line-join': 'round',
                            'line-cap': 'round'
                        },
                        paint: {
                            'line-color': foregroundColor,
                            'line-width': 1.25,
                            'line-opacity': 0.75
                        }
                    });

                    this.layers.push('hecate-data-points');
                    this.gl.addLayer({
                        id: 'hecate-data-points',
                        type: 'circle',
                        source: 'hecate-data',
                        'source-layer': 'data',
                        filter: ['==', '$type', 'Point'],
                        paint: {
                            'circle-color': foregroundColor,
                            'circle-radius': 4,
                            'circle-opacity': 0.85
                        }
                    });
                },
                unstyle: function() {
                    for (let layer of this.layers) {
                        this.gl.removeLayer(layer);
                    }
                    this.layers = [];
                }
            },
            panel: false, //Store the current panel view (Deltas, Styles, Bounds, etc)
            layers: [], //Store list of GL layer names so they can be easily removed
            feature: false, //Store the id of a clicked feature
            style_id: false, //Store the id of the current style - false for generic style
            modal: false
        }
    },
    components: {
        foot: Foot,
        deltas: Deltas,
        bounds: Bounds,
        feature: Feature,
        styles: Styles,
        login: Login,
        register: Register,
        settings: Settings,
        query: Query,
        stylem: Style
    },
    mounted: function(e) {
        mapboxgl.accessToken = this.credentials.map.key;
        this.map.gl = new mapboxgl.Map({
            container: 'map',
            style: 'mapbox://styles/mapbox/satellite-v9',
            center: [-96, 37.8],
            hash: true,
            maxzoom: 14,
            zoom: 3
        });

        this.map.gl.on('load', () => {
            this.map.gl.addSource('hecate-data', {
                type: 'vector',
                maxzoom: 14,
                tiles: [ `http://${window.location.host}/api/tiles/{z}/{x}/{y}` ]
            });

            this.map.gl.addSource('hecate-delta', {
                type: 'geojson',
                data: { type: 'FeatureCollection', features: [] }
            });

            this.map.default();
        });

        this.map.gl.addControl(new MapboxGeocoder({
            accessToken: mapboxgl.accessToken,
        }));

        this.map.gl.on('click', (e) => {
            if (this.modal === 'delta') return; //Don't currently support showing features within a delta

            let clicked = this.map.gl.queryRenderedFeatures(e.point)[0];

            if (clicked && clicked.properties['hecate:id']) {
                this.feature = clicked.properties['hecate:id'];
            }
        });
    },
    methods: {
        logout: function(reload) {
            this.credentials.authed = false;

            fetch(`http://${window.location.host}/api/user/session`, {
                method: 'DELETE',
                credentials: 'same-origin'
            }).then((response) => {
                if (reload) window.location.reload();
            });
        }
    }
}
</script>
