<template>
	<div 
		class="q-pa-md q-gutter-md"
		style="max-width: 1200px; margin: 0 auto"
	>
		<div>
			<q-select
				outlined
				class="full-width"
				placeholder="Pesquisar"
				v-model="search"
				:options="moviesNowSearch"
				use-input
				@filter="getSearchMovie"
				debounce="800"
			>
				<template v-slot:option="scope">
					<q-item
						style="max-width: 1000px"
						v-bind="scope.itemProps"
						v-if="scope.opt.backdrop_path"
						@click="$router.push('/MovieDetails/'+ scope.opt.id)"
					>
						<q-item-section avatar>
							<q-img :src="`https://image.tmdb.org/t/p/w220_and_h330_bestv2/${scope.opt.backdrop_path}`" />
						</q-item-section>

						<q-item-section>
							<q-item-label>{{ scope.opt.original_title }}</q-item-label>
							<q-item-label caption>{{ scope.opt.overview }}</q-item-label>
						</q-item-section>

					</q-item>
				</template>
			</q-select>
		</div>

		<div class="text-h6">
			Tendências
		</div>

		<div>
			<q-card>
				<q-virtual-scroll
					class="q-pa-md"
					:items="this.moviesNowPlaying.length > 0 ? this.moviesNowPlaying : this.moviesNowSearch"
					virtual-scroll-horizontal
					v-slot="{ item }"
				>
					<q-card bordered class="q-pa-sm cursor-pointer" style="max-width: 150px;">
						<div class="column" >
							<img
								:src="`https://image.tmdb.org/t/p/w220_and_h330_bestv2/${item.poster_path}`"
								style="max-height:200px;"
								@click="$router.push('/MovieDetails/'+ item.id)"
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
							<div class="text-subtitle2">{{ item.original_title }}</div>
						</div>
					</q-card>
				</q-virtual-scroll>
			</q-card>
		</div>

        <!--
            <q-select
                label="Genres"
                outlined
                option-label="name"
                option-value="id"
                emit-value
                map-options
                v-model="this.genre"
                :options="this.movieForGenres"
                use-input
                :rules="[(val) => !!val || 'Selecione um Status']"
            />
        -->
        
       
	</div>
</template>

<script>
import { defineComponent } from 'vue';
export default defineComponent({
    data(){
        return {
            movieForGenres: [],
            moviesNowPlaying: [],
            moviesNowSearch: [],
            movieForId: [],
            genre: null,
            search: null,
        }
    },
    methods:{
        getMovieListForGenre(){
            this.$api.get(`genre/movie/list`)
            .then((res) =>{
                this.movieForGenres = res.data.genres;
                if (res.data.length == 0) {
                    this.$q.notify({
                        closeBtn: true,
                        timeout: 3000,
                        message: "Movies not found.",
                        type: "warning",
                    });
                }
            })
        },
        getMovieListNowPlaying(){
            this.$api.get(`movie/now_playing`)
            .then((res) =>{
                this.moviesNowPlaying = res.data.results;
                if (res.data.length == 0) {
                    this.$q.notify({
                        closeBtn: true,
                        timeout: 3000,
                        message: "Movies not found.",
                        type: "warning",
                    });
                }
            })
        },

        getSearchMovie(search, update){
            this.$api.get(`search/movie`,{
                params: {query: search}
            })
            .then((res) =>{
                update(() => {
                    this.moviesNowSearch = res.data.results;
                });

                if (res.data.length == 0) {
                    this.$q.notify({
                        closeBtn: true,
                        timeout: 3000,
                        message: "Not Found Movies.",
                        type: "warning",
                    });
                }
            })
        },
    },
    beforeMount(){
        this.getMovieListForGenre();
        this.getMovieListNowPlaying();
    }
})

</script>
