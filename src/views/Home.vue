<template>
	<div class="home">
		<section class="page-head">
			<div class="container text-center py-4">
				<h3 class="mb-3">Url of any WordPress site</h3>
				<div class="row justify-content-center">
					<div class="col-7">
						<input type="url" class="form-control form-control-lg wp-url" v-model="q" placeholder="Ex: https://ma.tt/">
					</div>
				</div>
			</div>
		</section>

		<div class="container my-3">
			<div v-if="state === 'loading'">
				<div class="spinner-border text-primary" role="status">
					<span class="sr-only">Loading...</span>
				</div>
			</div>
			<div v-if="state === 'error'">
				<div class="alert alert-danger">
					<strong>Oops.</strong> Probably this is not a WordPress site. ({{ this.site.error }})
				</div>
			</div>
			<div v-if="state === 'idle'">

				<ul class="nav nav-tabs mb-3">
					<li class="nav-item">
						<router-link :to="`/${$route.params.host}`" class="nav-link" :class="{active: !$route.params.tab}">Overview</router-link>
					</li>
					<!--
					<li class="nav-item">
						<router-link :to="`/${$route.params.host}/taxonomies`" class="nav-link" :class="{active: $route.params.tab === 'taxonomies'}">Taxonomies</router-link>
					</li>
					-->
					<li v-for="type in site.types" :key="type.slug" class="nav-item">
						<router-link :to="`/${$route.params.host}/${type.slug}`" class="nav-link" :class="{active: $route.params.tab === type.slug}">{{ type.name }} <span v-if="type.totalTotal" class="badge badge-light-secondary badge-pill">{{ type.totalTotal }}</span></router-link>
					</li>
				</ul>

				<div v-if="$route.params.tab === 'taxonomies'">
					<pre>{{ site.taxonomies }}</pre>
				</div>
				<div v-else-if="$route.params.tab === 'attachment' && site.types.attachment">
					
					<div class="form-row mb-3">
						<div class="col-3">
							<input type="search" class="form-control" v-model="site.filters.search" placeholder="Search query">
						</div>
						<div class="col-3">
							<select class="form-control" v-model="site.filters.media_type">
								<option value="">Media Type</option>
								<option value="image">Image</option>
								<option value="video">Video</option>
								<option value="application">Application</option>
								<option value="text">Text</option>
								<option value="audio">Audio</option>
							</select>
						</div>
					</div>

					<div class="row row-cols-1 row-cols-md-2" :class="{'row-cols-md-3': site.filters.media_type !== 'video'}">
						<div v-for="media in site.types.attachment.items" :key="media.id" class="col my-2">
							<div class="card">

								<div v-if="media.media_type === 'file' && media.mime_type.includes('video/')" class="embed-responsive embed-responsive-16by9">
									<video controls preload="metadata" loading="lazy">
										<source :src="media.source_url" :type="media.mime_type">
										Sorry, your browser doesn't support embedded videos.
									</video>
								</div>
								<audio v-else-if="media.media_type === 'file' && media.mime_type.includes('audio/')" controls :src="media.source_url" loading="lazy">
									Sorry, your browser doesn't support embedded audio.
								</audio>
								<a v-else-if="media.media_type === 'image'" :href="media.source_url" target="_blank" rel="noreferer">
									<img :src="media.media_details.sizes.full ? (media.media_details.sizes.medium || media.media_details.sizes.full).source_url : media.source_url" class="card-img-top" loading="lazy" :alt="media.title.rendered">
								</a>

								<div class="card-body">
									<p>
										<span class="badge badge-light-info mr-1">{{ media.mime_type }}</span>
										<span v-if="media.media_details.length_formatted" class="badge badge-light-secondary mr-1">{{ media.media_details.length_formatted }}</span>
										<span v-if="media.media_details.width" class="badge badge-light-secondary mr-1">{{ media.media_details.width || '-' }}x{{ media.media_details.height || '-' }}</span>
										<span v-if="media.media_details.filesize" class="badge badge-light-secondary mr-1">{{ bytesToSize(media.media_details.filesize) }}</span>
									</p>
									<h6 class="card-title"><a :href="media.source_url" target="_blank" rel="noreferer" v-html="media.title.rendered"></a></h6>
									<div class="card-text" v-html="media.caption.rendered"></div>

									<p class="card-text mb-0">
										<a :href="media.guid.rendered" target="_blank" rel="noreferer" class="mr-2">📄 File link</a>
										<a v-if="media.post" :href="`${site.info.home}/?p=${media.post}`" target="_blank" rel="noreferer">🔗 Linked post</a>
									</p>
								</div>

								<div class="card-footer">
									<p class="mb-1">
										Date: {{ new Date(media.date).toLocaleString() }}
									</p>
									<p v-if="media._embedded && media._embedded.author && media._embedded.author.length" class="mb-0">
										Author:
										<a v-for="author in media._embedded.author" :key="author.id" v-show="author.id" :href="author.link" target="_blank" rel="noreferer"><img v-if="author.id && author.avatar_urls" :src="author.avatar_urls['24']" height="18" class="rounded-circle" :alt="author.name"> {{ author.name }}</a>
									</p>
								</div>
							</div>
						</div>
					</div>

					<div v-if="site.types[$route.params.tab].state === 'idle' && !site.types[$route.params.tab].items.length" class="alert alert-info text-center">
						<strong>Oops,</strong> looks there aren't any items to display. Try changing the filters
					</div>

					<div v-if="site.types[$route.params.tab].error" class="alert alert-danger">
						{{ site.types[$route.params.tab].error }}
					</div>

					<div v-if="site.types[$route.params.tab].state === 'loading'" class="spinner-border spinner-border-sm ml-1" role="status"></div>

					<div v-if="site.types[$route.params.tab].totalPages" class="bg-white rounded p-2 my-3">
						<div class="row align-items-center">
							<div class="col">
								<strong>{{ site.types[$route.params.tab].total }}</strong> items over <strong>{{ site.types[$route.params.tab].totalPages }}</strong> pages
							</div>
							<div class="col-auto">
								<ul class="pagination pagination-sm mb-0">
									<li v-for="page in Math.min(site.types[$route.params.tab].totalPages || 1, 25)" :key="page" class="page-item" :class="{active: page == site.types[$route.params.tab].page}"><button class="page-link" @click="site.types[$route.params.tab].page = page; load($route.params.tab)">{{ page }}</button></li>
								</ul>
							</div>
						</div>
					</div>

				</div>
				<div v-else-if="$route.params.tab && site.types[$route.params.tab]">

					<div class="form-row mb-3">
						<div class="col-3">
							<input type="search" class="form-control" v-model="site.filters.search" placeholder="Search query">
						</div>
						<div v-for="taxonomy in site.taxonomies" :key="taxonomy.slug" v-show="taxonomy.types.includes($route.params.tab)" class="col-3">
							<select class="form-control" v-model="site.filters[taxonomy.rest_base]">
								<option value="">{{ taxonomy.name }}</option>
								<option v-for="term in taxonomy.items" :key="term.id" :value="term.id">{{ term.name }} ({{ term.count }})</option>
							</select>
						</div>
					</div>

					<div class="row row-cols-1 row-cols-md-3 mb-3">
						<div v-for="item in site.types[$route.params.tab].items" :key="item.id" class="col my-2">
							<div class="card h-100">
								<a v-if="item._embedded && item._embedded['wp:featuredmedia'] && item._embedded['wp:featuredmedia'].length" :href="item.link" target="_blank" rel="noreferer">
									<img :src="item._embedded['wp:featuredmedia'][0].source_url" class="card-img-top" loading="lazy" :alt="item._embedded['wp:featuredmedia'][0].alt_text">
								</a>
								<div class="card-body">
									<h6 class="card-title"><a :href="item.link" target="_blank" rel="noreferer" v-html="item.title.rendered"></a></h6>
									<div v-if="item.excerpt || item.content" class="card-text" v-html="(item.excerpt || item.content).rendered"></div>
								</div>
								<div class="card-footer">
									<p class="mb-1">
										Date: {{ new Date(item.date).toLocaleString() }}
									</p>
									<p v-if="item._embedded && item._embedded.author && item._embedded.author.length" class="mb-1">
										Author:
										<a v-for="author in item._embedded.author" :key="author.id" v-show="author.id" :href="author.link" target="_blank" rel="noreferer"><img v-if="author.id && author.avatar_urls" :src="author.avatar_urls['24']" height="18" class="rounded-circle" :alt="author.name"> {{ author.name }}</a>
									</p>
									<div v-if="item._embedded && item._embedded['wp:term'] && item._embedded['wp:term'].length">
										<p v-for="(taxonomy, index) in item._embedded['wp:term']" :key="index" class="mb-1">
											<span v-if="taxonomy.length" class="text-capitalize">{{ taxonomy[0].taxonomy }}: </span>
											<a v-for="term in taxonomy" :key="term.id" :href="term.link" class="mr-1" target="_blank" rel="noreferer">{{ term.name }}</a>
										</p>
									</div>
								</div>
							</div>
						</div>
					</div>

					<div v-if="site.types[$route.params.tab].state === 'idle' && !site.types[$route.params.tab].items.length" class="alert alert-info text-center">
						<strong>Oops,</strong> looks there aren't any items to display. Try changing the filters
					</div>

					<div v-if="site.types[$route.params.tab].error" class="alert alert-danger">
						{{ site.types[$route.params.tab].error }}
					</div>

					<div v-if="site.types[$route.params.tab].state === 'loading'" class="spinner-border spinner-border-sm ml-1" role="status"></div>

					<div v-if="site.types[$route.params.tab].totalPages" class="bg-white rounded p-2 my-3">
						<div class="row align-items-center">
							<div class="col">
								<strong>{{ site.types[$route.params.tab].total }}</strong> items over <strong>{{ site.types[$route.params.tab].totalPages }}</strong> pages
							</div>
							<div class="col-auto">
								<ul class="pagination pagination-sm mb-0">
									<li v-for="page in Math.min(site.types[$route.params.tab].totalPages || 1, 25)" :key="page" class="page-item" :class="{active: page == site.types[$route.params.tab].page}"><button class="page-link" @click="site.types[$route.params.tab].page = page; load($route.params.tab)">{{ page }}</button></li>
								</ul>
							</div>
						</div>
					</div>
					
				</div>
				<div v-else>
					<table class="table">
						<tr>
							<th>Site URL</th>
							<td>{{ site.info.url }}</td>
						</tr>
						<tr>
							<th>Homepage</th>
							<td>{{ site.info.home }}</td>
						</tr>
						<tr>
							<th>Name</th>
							<td>{{ site.info.name }}</td>
						</tr>
						<tr>
							<th>Description</th>
							<td>{{ site.info.description }}</td>
						</tr>
						<tr>
							<th>Timezone</th>
							<td>{{ site.info.timezone_string }}, {{ site.info.gmt_offset }}h from UTC</td>
						</tr>
						<tr>
							<th>Namespaces</th>
							<td><span v-for="(namespace, index) in site.info.namespaces" :key="index" class="badge badge-light-secondary mr-1">{{ namespace }}</span></td>
						</tr>
						<tr>
							<th>Authentication</th>
							<td>{{ site.info.authentication }}</td>
						</tr>
					</table>
				</div>
			</div>
		</div>

		<p class="text-right"><button class="btn btn-link" @click="useProxy = !useProxy"><small>{{ useProxy ? 'Don\'t use proxy' : 'Use proxy' }}</small></button></p>
	</div>
</template>

<script>
import { debounce } from 'vue-debounce'
import axios from 'axios'
import jQuery from 'jquery'

export default {
	name: 'Home',
	components: {
		// HelloWorld
	},
	props: ['query'],
	data() {
		return {
			q: this.query || '',
			state: 'waiting',
			perPage: 100,
			apiUrl: '',
			site: {
				info: null,
				taxonomies: {},
				filters: {
					search: '',
					media_type: '',
				},
				types: {},
				error: '',
			},
			reqCancelToken: null,
			useProxy: false,
		}
	},
	created() {
		if (this.$route.params.host) {
			this.q = this.$route.params.host
		}
	},
	mounted() {
		document.querySelector('.wp-url').focus()
	},
	methods: {
		loadWpApi: debounce(function(url) {
			this.state = 'loading'
			this.site.error = ''

			url = url.trim()

			if (!url.startsWith('http://') && !url.startsWith('https://')) {
				// TODO which is better?
				url = `https://${url}`
			}

			try {
				url = new URL(url)
			} catch (error) {
				this.state = 'error'
				this.site.error = error.message
				return
			}

			if (url.host !== this.$route.params.host) {
				let params = {
					host: url.host,
				}

				// if loading a new website, go to Overview tab
				if (this.$route.params.host === url.host) {
					params.tab = this.$route.params.tab
				}

				this.$router.push({ name: 'Home', params })
			}

			this.site.apiUrl = `${url.origin}/wp-json`

			this.apiRequest().then(({ data, headers }) => {
				this.site.apiUrl = data.home + '/wp-json'
				this.site.info = data

				console.log(headers)
				console.log(data)

				this.apiRequest('/wp/v2/types').then(({ data }) => {

					for (const type in data) {
						data[type].total = 0
						data[type].totalPages = 0
						data[type].page = 1
						data[type].items = []
						data[type].state = 'idle'
						data[type].error = null
					}

					this.site.types = data

					if (this.$route.params.tab && data[this.$route.params.tab]) {
						this.load(this.$route.params.tab)
					}
				})

				this.apiRequest('/wp/v2/taxonomies').then(({ data }) => {
					for (const taxonomy in data) {
						const restBase = data[taxonomy].rest_base

						data[taxonomy].items = []
						this.$set(this.site.filters, restBase, '')

						this.apiRequest(`/wp/v2/${data[taxonomy].rest_base}?per_page=100`).then((re) => {
							data[taxonomy].items.push(...re.data)
						})
					}

					this.site.taxonomies = data
				})

				this.state = 'idle'
			}, error => {
				this.state = 'error'
				this.site.error = error.message
			})
		}, 250),
		apiRequest(path, data) {
			path = path || ''
			data = data || {}

			let url = `${this.site.apiUrl}${path}`

			// to bypass unallowed Origin
			if (this.useProxy) {
				url = `https://url-proxy.layered.workers.dev/?url=${encodeURIComponent(url)}`
			}

			return axios.get(url, data)
		},
		load: debounce(function(type) {
			if (!type) {
				return
			}

			const t = this.site.types[type]
			t.state = 'loading'
			t.error = null
			t.items = []

			let params = {
				per_page: this.perPage,
				page: t.page,
				_embed: 1,
			}

			for (const filter in this.site.filters) {
				const val = this.site.filters[filter]

				if (val && val !== '') {
					params[filter] = this.site.filters[filter]
				}
			}

			// cancel previous request, if started
			if (this.reqCancelToken) {
				this.reqCancelToken.cancel()
			}

			// create an about signal
			this.reqCancelToken = axios.CancelToken.source()

			this.apiRequest(`/wp/v2/${t.rest_base}?${jQuery.param(params)}`, {
				cancelToken: this.reqCancelToken.token,
			}).then(({ data, headers }) => {
				// store total number of items, not including filters
				if (!t.totalTotal || headers['x-wp-total'] > t.totalTotal) {
					t.totalTotal = headers['x-wp-total'] || 0
				}

				t.total = headers['x-wp-total'] || 0
				t.totalPages = parseInt(headers['x-wp-totalpages'] || 0, 10)

				t.items.push(...data)
				t.state = 'idle'
			}, error => {
				t.state = 'error'
				t.error = error.message
				console.log('error', error)
			})
		}, 350),
		bytesToSize(bytes) {
			var sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB'];
			if (bytes == 0) return '0 Byte';
			var i = parseInt(Math.floor(Math.log(bytes) / Math.log(1024)));
			return Math.round(bytes / Math.pow(1024, i), 2) + ' ' + sizes[i];
		}
	},
	watch: {
		q(q, queryOld) {
			q = q.replace(/\s/g, '').toLowerCase().trim()

			if (q === queryOld.replace(/\s/g, '').toLowerCase().trim()) {
				return
			}

			this.loadWpApi(q)
		},
		$route(route, routeOld) {
			console.log(route.params, routeOld.params)

			if (route.params.tab && route.params.tab !== routeOld.params.tab && this.site.types[route.params.tab]) {
				this.site.types[route.params.tab].page = 1
				for (const slug in this.site.filters) {
					this.site.filters[slug] = ''
				}

				this.load(route.params.tab)
			}

			if (route.params.tab && this.site.types[route.params.tab] && this.site.types[route.params.tab].page === 1) {
				//this.load(route.params.tab)
			}
		},
		'site.filters': {
			deep: true,
			handler() {
				this.load(this.$route.params.tab)
			}
		}
	}
}
</script>
