<!-- Docs: https://jvm-docs.vercel.app/docs/markers -->
<template>
    <div>
        <b>Top 5 country/ region by clicks</b>
        <div id="container">
            <div id="map"></div>
        </div>
    </div>
</template>


<script>
import jsVectorMap from 'jsvectormap';
import 'jsvectormap/dist/maps/world';
import 'jsvectormap/src/scss/jsvectormap.scss'; // default Vue styling

export default {
    name: 'ReflinkRankMap',
    data() {
        return {
            map: null,
            countries: [],
            coords: {
                'hi': {
                    'lat': 0,
                    'lng': 0
                },
            },
        };
    },
    beforeMount() {
        this.fetchCountryData();
    },
    mounted() {
        this.initMap();
    },
    methods: {
        async fetchCountryData() {
            try {
                const response = await fetch('https://staging-sls.g2g.com/locale/country');
                const data = await response.json();
                this.countries = data.payload.results;
                // this.testAllCountry(); // for testing which country is supported
                this.dataFetched = true;
            }
            catch (error) {
                console.error(error);
            }

        },
        initMap() {
            this.map = new jsVectorMap({
                selector: '#map',
                map: 'world',
                regionStyle: {
                    initial: {
                        fill: 'lightgray',
                        stroke: 'black',
                        strokeWidth: 0.5,
                        fillOpacity: 1,
                    },
                    // hover settings
                    hover: {
                        fill: 'red',
                        fillOpacity: 1,
                    },
                    selected: {
                        fill: '#548AF7',
                    }
                },
                // Can add markers and labels, dynamic
                markers: [{ coords: [this.coords['hi'].lat, this.coords['hi'].lng], labelName: 'HiHi' }],
                labels: {
                    markers: {
                        render(marker) {
                            return marker.labelName || 'Not available'
                        }
                    }
                },
                showTooltip: true,
                draggable: false,
                zoomButtons: false,
                zoomOnScroll: false,
                onRegionTooltipShow(event, tooltip) {
                    tooltip.css({ backgroundColor: 'black' }); // tooltip customization
                    tooltip.text(
                        tooltip.text(),
                        true // enable HTML
                    );
                },

                // enable the country_code argument if you want dynamic tooltip content
                //         onRegionTooltipShow(event, tooltip, code) {
                //             if (code === 'US') {
                //                 tooltip.text(`
                //     <h3>USA</h3>
                //     <p>The United States</p>
                //   `, true)
                //             }
                //         }

            });
            // Adds removeSelection method to instance
            this.map.extend("removeSelectedRegion", function (code) {
                for (const key in this.regions) {
                    if (this.regions[key].element.isSelected) {
                        if (key === code) {
                            this.regions[key]
                                .element.select(false)
                        }
                    }
                }
            });
            // Adds addSelection method to instance
            this.map.extend("addSelectedRegion", function (code) {
                for (const key in this.regions) {
                    if (key === code) {
                        this.regions[key]
                            .element.select(true)
                    }
                }
            });

            this.updateFills();
        },
        updateFills() {
            // For testing purposes
            // const topCountries = this.countries.sort((a, b) => b.dial_code - a.dial_code)
            //     .slice(0, 5)
            //     .map(country => country.country.toUpperCase());

            // For demo purposes
            const topCountries = ["US", "BR", "MY", "CN", "ID"]

            // Add top countries to selectedRegions
            topCountries.forEach(country => {
                this.map.addSelectedRegion(country);
            });
        },

        testAllCountry() {
            const allCountries = this.countries.map(country => country.country.toUpperCase());
            allCountries.forEach(country => {
                this.map.addSelectedRegion(country);
            });
        }

    },
};
</script>

<style>
#container {
    display: flex;
    flex-direction: column;
    padding-right: 50px;
    padding-left: 50px;
}

/* CSS styling */
#map {
    width: 90%;
    height: 600px;
    margin-bottom: 200px;
}

/* Outer scope tooltip customization */
.jvm-tooltip {
    background-color: orange;
}
</style>