<template>
	<div class="home">
		<section class="page-head">
			<div class="container text-center py-5">
				<h3 class="mb-3">Url of any WordPress site</h3>
				<div class="row justify-content-center">
					<div class="col-7">
						<input type="url" class="form-control form-control-lg wp-url" v-model="q" placeholder="Ex: https://ma.tt/">
					</div>
				</div>
			</div>
		</section>

		<div class="container my-5">
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
					<li v-for="type in site.types" :key="type.slug" class="nav-item">
						<router-link :to="`/${$route.params.host}/${type.slug}`" class="nav-link" :class="{active: $route.params.tab === type.slug}">{{ type.name }} <span v-if="type.total" class="badge badge-light-secondary badge-pill">{{ type.total }}</span></router-link>
					</li>
				</ul>

				<div v-if="$route.params.tab === 'attachment' && site.types.attachment">
					<div class="row row-cols-1 row-cols-md-3 row-cols-lg-3">
						<div v-for="media in site.types.attachment.items" :key="media.id" class="col my-2">
							<div v-if="media.media_type === 'file' && media.mime_type.includes('video/')" class="card">
								<div class="embed-responsive embed-responsive-16by9">
									<video controls width="250" loading="lazy">
										<source :src="media.source_url" :type="media.mime_type">
										Sorry, your browser doesn't support embedded videos.
									</video>
								</div>
								<div class="card-body">
									<h6 class="card-title"><a :href="media.source_url" target="_blank" rel="noreferer">{{ media.title.rendered }}</a></h6>
									<p class="card-text">
										<span class="badge badge-light-success mr-1">{{ media.mime_type }}</span>
										<span class="badge badge-light-secondary mr-1">{{ media.media_details.length_formatted }}</span>
										<span class="badge badge-light-secondary mr-1">{{ media.media_details.width || '-' }}x{{ media.media_details.height || '-' }}</span>
										<span class="badge badge-light-secondary">{{ media.date }}</span>
									</p>
									<p class="card-text" v-html="media.caption.rendered"></p>
								</div>
							</div>
							<div v-else-if="media.media_type === 'image'" class="card">
								<a :href="media.source_url" target="_blank" rel="noreferer">
									<img :src="media.media_details.sizes.full ? (media.media_details.sizes.medium || media.media_details.sizes.full).source_url : media.source_url" class="card-img-top" loading="lazy" :alt="media.title.rendered">
								</a>
								<div class="card-body">
									<h6 class="card-title"><a :href="media.source_url" target="_blank" rel="noreferer">{{ media.title.rendered }}</a></h6>
									<p class="card-text">
										<span class="badge badge-light-info mr-1">{{ media.mime_type }}</span>
										<span class="badge badge-light-secondary mr-1">{{ media.media_details.width || '-' }}x{{ media.media_details.height || '-' }}</span>
									</p>
									<p class="card-text">{{ media.caption.rendered }}</p>
								</div>
							</div>
							<div v-else class="card">
								<div class="card-body">
									<h6 class="card-title">{{ media.title.rendered }}</h6>
									<p class="card-text"><span class="badge badge-light-secondary">{{ media.media_type }}</span> {{ media.caption.rendered }}</p>
									<p class="card-text"><a :href="media.guid.rendered" target="_blank" rel="noreferer">link</a></p>
								</div>
							</div>
						</div>
					</div>
					<p v-if="!site.types[$route.params.tab].total || site.types[$route.params.tab].items.length < site.types[$route.params.tab].total">
						<button @click="load('attachment')" class="btn btn-sm btn-light">Load more</button>
						<span v-if="site.types[$route.params.tab].state === 'loading'" class="spinner-border spinner-border-sm" role="status"></span>
					</p>
					<div v-if="site.types[$route.params.tab].error" class="alert alert-danger">
						{{ site.types[$route.params.tab].error }}
					</div>
				</div>
				<div v-else-if="$route.params.tab && site.types[$route.params.tab]">

					<div class="bg-white rounded p-2 mb-3">
						<strong>{{ site.types[$route.params.tab].total }}</strong> items over <strong>{{ site.types[$route.params.tab].totalPages }}</strong> pages
					</div>

					<div class="row row-cols-1 row-cols-md-3">
						<div v-for="item in site.types[$route.params.tab].items" :key="item.id" class="col my-2">
							<div class="card h-100">
								<a v-if="item._embedded['wp:featuredmedia'] && item._embedded['wp:featuredmedia'].length" :href="item.link" target="_blank" rel="noreferer">
									<img :src="item._embedded['wp:featuredmedia'][0].source_url" class="card-img-top" loading="lazy" :alt="item._embedded['wp:featuredmedia'][0].alt_text">
								</a>
								<div class="card-body">
									<h6 class="card-title"><a :href="item.link" target="_blank" rel="noreferer" v-html="item.title.rendered"></a></h6>
									<div class="card-text" v-html="item.excerpt.rendered"></div>
								</div>
								<div class="card-footer">
									<p class="mb-1">
										Date: {{ item.date }}
									</p>
									<p v-if="item._embedded.author && item._embedded.author.length" class="mb-1">
										Author:
										<a v-for="author in item._embedded.author" :key="author.id" v-show="author.id" :href="author.link" target="_blank" rel="noreferer"><img v-if="author.id" :src="author.avatar_urls['24']" height="18" class="rounded-circle" :alt="author.name"> {{ author.name }}</a>
									</p>
									<div v-if="item._embedded['wp:term'] && item._embedded['wp:term'].length">
										<p v-for="(taxonomy, index) in item._embedded['wp:term']" :key="index" class="mb-1">
											<span v-if="taxonomy.length" class="text-capitalize">{{ taxonomy[0].taxonomy }}: </span>
											<a v-for="term in taxonomy" :key="term.id" :href="term.link" class="mr-1" target="_blank" rel="noreferer">{{ term.name }}</a>
										</p>
									</div>
								</div>
							</div>
						</div>
					</div>

					<ul>
						<li v-for="item in site.types[$route.params.tab].items" :key="item.id"><a :href="item.link" v-html="item.title.rendered" target="_blank" rel="noreferer"></a> <span class="badge badge-light-secondary">{{ item.date }}</span></li>
					</ul>
					<p v-if="!site.types[$route.params.tab].total || site.types[$route.params.tab].items.length < site.types[$route.params.tab].total">
						<button @click="load($route.params.tab)" class="btn btn-sm btn-light">Load more</button>
						<span v-if="site.types[$route.params.tab].state === 'loading'" class="spinner-border spinner-border-sm ml-1" role="status"></span>
					</p>
					<div v-if="site.types[$route.params.tab].error" class="alert alert-danger">
						{{ site.types[$route.params.tab].error }}
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


	</div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'

import axios from 'axios'

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
				types: {},
				error: '',
			},
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
		loadWpApi(url) {
			this.state = 'loading'
			this.site.error = ''

			if (!url.startsWith('http://') && !url.startsWith('https://')) {
				// TODO which is better?
				url = `https://${url}`
			}

			url = new URL(url)

			if (url.host !== this.$route.params.host) {
				this.$router.push({ name: 'Home', params: { host: url.host, tab: this.$route.params.tab } })
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

				this.state = 'idle'
			}, error => {
				this.state = 'error'
				this.site.error = error.message
			})
		},
		apiRequest(path, data) {
			path = path || ''
			data = data || {}

			return axios.get(`${this.site.apiUrl}${path}`, data)
		},
		load: debounce(function(type) {
			const t = this.site.types[type]
			t.state = 'loading'
			t.error = null

			let params = {
				per_page: this.perPage,
				page: t.page++,
				_embed: 1,
			}

			if (type === 'attachment') {
				//params.media_type = 'video'
			}

			this.apiRequest(`/wp/v2/${t.rest_base}`, { params }).then(({ data, headers }) => {
				t.total = headers['x-wp-total'] || 0
				t.totalPages = headers['x-wp-totalpages'] || 0

				t.items.push(...data)
				t.state = 'idle'
			}, error => {
				t.state = 'error'
				t.error = error.message
				console.log('error', error)
			})
		}, 350),
	},
	watch: {
		q(q, queryOld) {
			q = q.replace(/\s/g, '').toLowerCase().trim()

			if (q === queryOld.replace(/\s/g, '').toLowerCase().trim()) {
				return
			}

			// TODO add debounce

			this.loadWpApi(q)
		},
		$route(route, routeOld) {
			console.log(route.params, routeOld.params)

			if (route.params.tab && this.site.types[route.params.tab] && this.site.types[route.params.tab].page === 1) {
				this.load(route.params.tab)
			}
		}
	}
}
</script>
