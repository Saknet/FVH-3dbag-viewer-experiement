<template>
  <div id="viewer">
    <section
      id="map-options"
      class="field has-addons"
    >
      <DropDownSelector
        v-model="basemapPreset"
        :options="basemaps"
        :title="$t('viewer.baselayer2')"
      />
      <DropDownSelector
        v-model="tileset"
        title="Tilesets"
        :options="tilesets"
      />
      <search-bar
        @select-place="moveToPlace"
      />
      <Compass
        ref="compass"
        :rotation="camRotationZ"
        @orient-north="orientNorth"
      />
    </section>
    <BuildingInformation
      :building="pickedBuilding"
      :show="showBuildingInfo"
      @close-info="showBuildingInfo = false"
      @report-data="getReportDataIssuePathWithId( pickedBuilding.attributes.identificatie )"
    />
    <ThreeViewer
      ref="threeviewer"
      :tiles-url="tilesUrl"
      :basemap-options="basemapOptions"
      @object-picked="objectPicked"
      @cam-offset="onCamOffset"
      @cam-rotation-z="onCamRotationZ"
    />
    <div
      v-if="true"
      id="attribution"
      class="has-background-white has-text-grey"
    >
      <p>
        <span v-if="basemapOptions.attribution">
          {{ $t("viewer.baselayer1") }}
          <a
            v-if="basemapOptions.attributionURL"
            href="https://www.pdok.nl/"
          >
            {{ basemapOptions.attribution }}
          </a>
          <span v-else>
            {{ basemapOptions.attribution }}
          </span> |
        </span>
        <a :href="'https://docs.3dbag.nl/'+this.$route.params.locale+'/copyright' ">© 3D BAG by tudelft3d</a>
      </p>
    </div>
    <div id="debug-panel" />
  </div>
</template>

<script>
import BuildingInformation from '@/components/BuildingInformation.vue';
import DropDownSelector from '@/components/DropDownSelector.vue';
import SearchBar from '@/components/SearchBar.vue';
import ThreeViewer from '@/components/ThreeViewer.vue';
import Compass from '@/components/Compass.vue';

export default {

	name: 'Viewer',

	components: {
		BuildingInformation,
		DropDownSelector,
		SearchBar,
		ThreeViewer,
		Compass
	},

	data() {

		return {

			customTilesUrl: 'https://geo.fvh.fi/tilesets/viikki/tileset.json',
			BAG3DVersion: this.$root.$data[ 'versions' ][ this.$root.$data[ "latest" ] ],

			camOffset: {
				x: 400,
				y: 400,
				z: 400
			},
			camRotationZ: 0,

			basemapPreset: 'ortoilmakuva_2020',
			basemaps: {
				kantakarttahelsinki: {
					name: "KantaKartta",
					icon: "map"
				},
				opaskarttahelsinki: {
					name: "Opaskartta",
					icon: "map"
				},
				ortoilmakuva20helsinki2015: {
					name: "Ortoilmakuva 2015",
					icon: "map"
				},
				karttasarjapks: {
					name: "Karttasarja PKS",
					icon: "map"
				},
				vaaravariortoilmakuva2017_20: {
					name: "Vääräväri ortoilmakuva 2017",
					icon: "map"
				},
				ajantasa_asemakaava_maanpaallinen: {
					name: "Ajantasainen asemakaava maanpäällinen",
					icon: "map"
				},
				ortoilmakuva_2018_5cm: {
					name: "Ortoilmakuva 2018",
					icon: "map"
				},
				ajantasa_asemakaava: {
					name: "Ajantasainen asemakaava",
					icon: "map"
				},
				ortoilmakuva_2017_20cm: {
					name: "Ortoilmakuva 2017",
					icon: "map"
				},
				kiinteistokartta: {
					name: "Kiinteistokartta",
					icon: "map"
				},
				vaaravariortoilmakuva_2015: {
					name: "Vääräväri ortoilmakuva 2015",
					icon: "map"
				},
				//
				opaskartta_pks: {
					name: "Opaskartta PKS",
					icon: "map"
				},
				ortoilmakuva_2020: {
					name: "Ortoilmakuva 2020",
					icon: "map"
				},
				kantakartan_maastotiedot: {
					name: "Kantakartan maastotiedot",
					icon: "map"
				},
				ortoilmakuva_2014: {
					name: "Ortoilmakuva 2014",
					icon: "map"
				},
				tosiortoilmakuva_2017: {
					name: "Tosi ortoilmakuva 2017",
					icon: "map"
				},
				ortoilmakuva_2016: {
					name: "Ortoilmakuva 2016",
					icon: "map"
				},
				vaaravariortoilmakuva_2020: {
					name: "Vääräväri ortoilmakuva 2020",
					icon: "map"
				},
				ortoilmakuva: {
					name: "Ortoilmakuva",
					icon: "map"
				},
				ortoilmakuva_2019: {
					name: "Ortoilmakuva 2019",
					icon: "map"
				},
				vaaravariortoilmakuva_2019: {
					name: "Vääräväriortoilmakuva 2019",
					icon: "map"
				},
				//
				ajantasa_asemakaava_maanpaallinen_varillinen: {
					name: "Ajantasa asemakaava maanpäällinen värillinen",
					icon: "map"
				},
				karttasarja: {
					name: "Karttasarja",
					icon: "map"
				},
				ortoilmakuva_2017: {
					name: "Ortoilmakuva 2017",
					icon: "map"
				},
				karttasarja_harmaa: {
					name: "Karttasarja harmaa",
					icon: "map"
				},
				kiinteistokartan_maastotiedot: {
					name: "Kiinteistökartan maastotiedot",
					icon: "map"
				}
			},

			tileset: 'viikki',
			tilesets: {
				viikki: {
					name: "viikki",
					icon: "home"
				}
			},

			pickedBuilding: {

				batchID: "-",
				attributes: []

			},

			showBuildingInfo: false,

		};

	},
	computed: {

		filteredDataArray() {

			return this.data.filter( ( option ) => {

				return option
					.toString()
					.toLowerCase()
					.indexOf( this.name.toLowerCase() ) >= 0;

			} );

		},

		tilesUrl: function () {

			if ( this.tileset == 'custom' ) {

				return this.customTilesUrl;

			}

			return this.BAG3DVersion[ '3DTilesets' ][ this.tileset ];

		},

		basemapOptions: function () {

			const sources = {

				kantakarttahelsinki: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Kantakartta',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				opaskarttahelsinki: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Opaskartta_Helsinki',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				ortoilmakuva20helsinki2015: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ortoilmakuva_2015_20cm',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				karttasarjapks: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Karttasarja_PKS',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				vaaravariortoilmakuva2017_20: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Vaaravariortoilmakuva_2017_20cm',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				ajantasa_asemakaava_maanpaallinen: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ajantasa_asemakaava_maanpaallinen',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				ortoilmakuva_2018_5cm: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ortoilmakuva_2018_5cm',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				ajantasa_asemakaava: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ajantasa_asemakaava',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				ortoilmakuva_2017_20cm: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ortoilmakuva_2017_20cm',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				kiinteistokartta: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Kiinteistokartta',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				vaaravariortoilmakuva_2015: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Vaaravariortoilmakuva_2015_20cm',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},
				//
				opaskartta_pks: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Opaskartta_PKS',
						style: 'rasteri',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				ortoilmakuva_2020: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ortoilmakuva_2020_5cm',
						style: 'rasteri',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				kantakartan_maastotiedot: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Kantakartan_maastotiedot',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				ortoilmakuva_2014: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ortoilmakuva_2014',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				tosiortoilmakuva_2017: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'TosiOrtoilmakuva_2017_8cm',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				ortoilmakuva_2016: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ortoilmakuva_2016',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				vaaravariortoilmakuva_2020: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Vaaravariortoilmakuva_2020_5cm',
						style: 'rasteri',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				ortoilmakuva: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ortoilmakuva',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				ortoilmakuva_2019: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ortoilmakuva_2019_20cm',
						style: 'rasteri',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				vaaravariortoilmakuva_2019: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Vaaravariortoilmakuva_2019_20cm',
						style: 'rasteri',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				ajantasa_asemakaava_maanpaallinen_varillinen: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ajantasa_asemakaava_maanpaallinen_varillinen',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				karttasarja: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Karttasarja',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				ortoilmakuva_2017: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Ortoilmakuva_2017_8cm',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/jpeg"
					}
				},

				karttasarja_harmaa: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Karttasarja_harmaa',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},

				kiinteistokartan_maastotiedot: {
					type: "wmts",
					attribution: "hri",
					attributionURL: "https://hri.fi/data/en_GB/dataset/helsingin-ortoilmakuvat",
					options: {
						url: 'https://kartta.hel.fi/ws/geoserver/avoindata/gwc/service/wmts?',
						layer: 'Kiinteistokartan_maastotiedot',
						style: 'default',
						tileMatrixSet: "ETRS-GK25",
						service: "WMTS",
						request: "GetTile",
						version: "1.0.0",
						format: "image/png"
					}
				},
			};

			return sources[ this.basemapPreset ];

		},

		reportDataIssueUrl: function () {

			return `https://docs.google.com/forms/d/e/
				1FAIpQLScIVBEWkpOraOIpOb1SOwRvpSnlQxLFDDYsqK4MrZgOqvNjWw/viewform?
				entry.401142300=${ this.tilesUrl }&
				entry.1880096492=${ escape( "https://3dbag.nl/#" + this.$route.fullPath ) }`;

		}

	},

	watch: {

		$route( to, from ) {

			if ( to.query.lod ) {

				this.tileset = to.query.lod;

			}

		},

		tileset( to, from ) {

			if ( to != from ) {

				let q = Object.assign( {}, this.$router.currentRoute.query );
				q.lod = to;

				this.$router.push(
					{ url: '/', query: q }
				).catch( err => {} );

			}

		}

	},
	mounted() {

		if ( this.$router.currentRoute.query.lod ) {

			this.tileset = this.$router.currentRoute.query.lod;

		}

	},
	methods: {

		onCamOffset: function ( event ) {

			this.camOffset = event;

		},

		onCamRotationZ: function ( value ) {

			this.$refs.compass.setRotation( value );

		},

		orientNorth: function ( value ) {

			this.$refs.threeviewer.pointCameraToNorth();

		},

		moveToPlace: function ( res ) {

			if ( res ) {

				this.$router.push( {
					path: '/' + this.$route.params.locale + '/viewer',
					query: {
						rdx: res.rd_x,
						rdy: res.rd_y,
						ox: this.camOffset.x,
						oy: this.camOffset.y,
						oz: this.camOffset.z,
						placeMarker: true
					}
				} );

			}

		},

		objectPicked: function ( event ) {

			if ( event ) {

				this.pickedBuilding = event;
				this.showBuildingInfo = true;

			} else {

				this.showBuildingInfo = false;

			}

		},

		getReportDataIssuePathWithId: function ( identificatie ) {

			window.open( this.reportDataIssueUrl + `&entry.547110854=${ identificatie }`, '_blank' );

		}

	}

};
</script>

<style>
#building-info {
	position: absolute;
	bottom: 0.5rem;
	margin: 0 0.5rem;
}
.table-value {
	overflow-x: auto;
}
#building-info .message-body {
	overflow: auto;
	max-height: 1%;
}

#map-options {
	position: absolute;
	margin: 0px;
	top: 3.75rem;
	margin: 0 0.5rem;
}

#viewer {

	width: 100%;
	height: 100%;

}

#attribution {
	position: absolute;
	padding: 0 0.1rem;
	font-size: 13px;
	line-height: 15px;
	right: 0;
	bottom: 0;
	opacity: 0.8;
}
</style>
