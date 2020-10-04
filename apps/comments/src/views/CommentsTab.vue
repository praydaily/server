<!--
  - @copyright Copyright (c) 2020 John Molakvoæ <skjnldsv@protonmail.com>
  -
  - @author John Molakvoæ <skjnldsv@protonmail.com>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<div :class="{ 'icon-loading': loading }">
		<!-- error message -->
		<EmptyContent v-if="error" icon="icon-error">
			{{ error }}
		</EmptyContent>

		<template v-else>
			<RichContenteditable v-model="message" :auto-complete="autoComplete" />

			<EmptyContent v-if="comments.length === 0" icon="icon-comment">
				{{ t('comments', 'No comments yet, start the conversation!') }}
			</EmptyContent>

			<!-- comments -->
			<Comment v-else v-for="comment in comments" :key="comment.id" :source="comment" />
			<!-- <VirtualList
				data-key="'uid'"
				:data-sources="comments"
				:data-component="Comment" /> -->
		</template>
	</div>
</template>

<script>
import { generateOcsUrl } from '@nextcloud/router'
import axios from '@nextcloud/axios'
import VirtualList from 'vue-virtual-scroll-list'

// import Avatar from '@nextcloud/vue/dist/Components/Avatar'
import EmptyContent from '@nextcloud/vue/dist/Components/EmptyContent'
import RichContenteditable from '@nextcloud/vue/dist/Components/RichContenteditable'

import Comment from '../components/Comment'
import fetchComments from '../services/Comments'

export default {
	name: 'CommentsTab',

	components: {
		// Avatar,
		Comment,
		EmptyContent,
		RichContenteditable,
		VirtualList,
	},

	data() {
		return {
			error: '',
			loading: true,

			fileInfo: null,
			offset: 0,
			comments: [],
			message: '',

			Comment,
		}
	},

	computed: {
	},

	methods: {
		/**
		 * Update current fileInfo and fetch new data
		 * @param {Object} fileInfo the current file FileInfo
		 */
		async update(fileInfo) {
			this.fileInfo = fileInfo
			this.resetState()
			this.getComments()
		},

		/**
		 * Get the existing shares infos
		 */
		async getComments() {
			try {
				this.loading = true

				// Fetch comments
				this.comments = await fetchComments(this.fileInfo.id)
			} catch (error) {
				this.error = t('comments', 'Unable to load the comments list')
				console.error('Error loading the comments list', error)
			} finally {
				this.loading = false
			}
		},

		/**
		 * Autocomplete @mentions
		 * @param {string} search the query
		 * @param {Function} callback the callback to process the results with
		 */
		async autoComplete(search, callback) {
			const results = await axios.get(generateOcsUrl('core', 2) + 'autocomplete/get', {
				params: {
					search,
					itemType: 'files',
					itemId: this.fileInfo.id,
					sorter: 'commenters|share-recipients',
					limit: OC.appConfig?.comments?.maxAutoCompleteResults || 25,
				},
			})
			return callback(results.data.ocs.data)
		},

		/**
		 * Reset the current view to its default state
		 */
		resetState() {
			this.loading = true
			this.error = ''
			this.comments = []
		},
	},
}
</script>
