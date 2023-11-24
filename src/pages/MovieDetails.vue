<template>
    <div class="q-pa-md" >
        <q-spinner-oval
            color="primary"
            size="2em"
            v-if="loading"
        />
        <div class="text-white" v-else>

            <div
                :style="`display: flex;
                    background-image:  linear-gradient(to right, rgba(${backdropColor}, 1) , rgba(${backdropColor}, 0.84) 50%, rgba(${backdropColor}, 0.84) 100%),
                    url(https://image.tmdb.org/t/p/w1920_and_h1080_multi_faces/${movieForId.backdrop_path})`"
                class="q-gutter-sm rounded-borders"
            >
                <img class="q-mb-sm" style="max-height: 500px;" :src="`https://image.tmdb.org/t/p/w600_and_h900_bestv2/${movieForId.poster_path}`" />

                <div>
                    <div class="column q-gutter-lg">
						<div>
							<div class="q-gutter-sm row items-center">
								<div class="text-h5 text-bold">{{  movieForId.title }}</div>
								<div class="text-h5 text-grey-4">({{ movieForId.release_date.split("-")[0] }})</div>
							</div>

							<div class="row text-subtitle2 text-grey-3 q-gutter-sm">
								<div>{{ moment(movieForId.release_date,).format('DD/MM/YYYY') }}</div>
								<div>-</div>

								<div>{{ movieForId.genres.map(genre => genre.name).join(", ") }}</div>
								<div>-</div>

								<div>{{ toHoursAndMinutes(movieForId.runtime) }}</div>
							</div>
						</div>
                        
						<div>
							<div class="row q-gutter-sm items-center">
								<q-circular-progress
									show-value
									font-size="12px"
									:value="movieForId.vote_average"
									size="4rem"
									:thickness="0.2"
									color="positive"
									track-color="grey-5"
									class="q-mt-xs text-h6"
									max="10"
								>
									<div class="row items-center">
										<div class="text-h6 text-bold">{{ Math.floor((100 * movieForId.vote_average) / 10) }}</div>
										<div class="text-caption">%</div>
									</div>
								</q-circular-progress>

								<div class="text-subtitle1" style="max-width: 50px">Avaliação dos usuários</div>
							</div>
						</div>
                        <div class="text-italic text-subtitle1 text-grey-6">{{ movieForId.tagline }}</div>
                        <div class="text-h6" v-if="movieForId.overview "> Sinopse:</div>
						<div class="text-subtitle" v-if="movieForId.overview ">{{ movieForId.overview }}</div>
						<div>
							<q-btn 
								@click="openTrailer = true"
								label="Assistir Trailer"
								icon-right="play_arrow"
								v-if="videoMovie.length"
								outline
							/>
						</div>
						
						<q-dialog v-model="openTrailer">
							<q-card style="max-width: 1200px;">
								<iframe 
									width="1200"
									height="700"
									:src="`https://www.youtube.com/embed/${videoMovie[0].key}`">
								</iframe>
							</q-card>
						</q-dialog>
                    </div>
                </div>
            </div>
        </div>

		<div class="text-h6 text-bold q-mt-md">Filmes Similares:</div>
		<div>
			<q-card>
				<q-virtual-scroll
					class="q-pa-md"
					:items="this.movieSimilar.length > 0 ? this.movieSimilar : null"
					virtual-scroll-horizontal
					v-slot="{ item }"
				>
					<q-card bordered class="q-pa-sm cursor-pointer" style="max-width: 120px;" v-if="item.poster_path && item.vote_average"> 
						<div class="column">
							<img
								:src="`https://image.tmdb.org/t/p/w220_and_h330_bestv2/${item.poster_path}`"
								style="max-height:100px; width: 100px;" 
								@click="openMovie(item.id)"
							/>
							<q-circular-progress
								show-value
								font-size="12px"
								:value="item.vote_average"
								size="2.3rem"
								:thickness="0.2"
								color="positive"
								track-color="grey-5"
								class="q-mt-xs text-h6"
								max="10"
								>
								{{ Math.round(item.vote_average * 100) / 100 }}
							</q-circular-progress>

							<div class="text-subtitle2">
								{{ item.original_title }}
							</div>
						</div>
					</q-card>
				</q-virtual-scroll>
			</q-card>
		</div>

    </div>
</template>
<script>
import { defineComponent } from "vue";
import moment from "moment";

import * as Vibrant from "node-vibrant";

export default defineComponent({
    data(){
        return {
            movieForId:{},
            loading: true,
			backdropColor: null,
            moment,
			videoMovie: {},
			movieSimilar: {},
			openTrailer: false,

        }
    },

    methods:{
		getBackdropDominantColor() {
			Vibrant.from(`https://image.tmdb.org/t/p/w600_and_h900_bestv2/${this.movieForId.poster_path}`).getPalette((err, pallete) => {
				this.backdropColor = pallete.DarkMuted.rgb.join(",");
			});
		},

		openMovie(id){
			window.open('/MovieDetails/'+ id)
		},

        getMovieForId(id){
			this.$q.loading.show()

            this.$api.get(`movie/${id}`)
            .then((res) =>{
                this.movieForId = res.data;

				this.getBackdropDominantColor();

				this.$q.loading.hide();
                if (res.data.length == 0) {
                    this.$q.notify({
                        closeBtn: true,
                        timeout: 3000,
                        message: "Movies not found.",
                        type: "warning",
                    });
                }
            })

			this.$api.get(`movie/${id}/videos`)
            .then((res) =>{
                this.videoMovie = res.data.results;

            })

			this.$api.get(`movie/${id}/similar`)
            .then((res) =>{
                this.movieSimilar = res.data.results;
            })

            this.loading = false
        },

		toHoursAndMinutes(totalMinutes) {
			const hours = Math.floor(totalMinutes / 60);
			const minutes = totalMinutes % 60;

			return `${hours}h ${minutes}m`;
		}
    },

    mounted(){
        this.getMovieForId(this.$route.params.id)
    }
})

</script>
