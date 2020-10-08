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
	<div class="comments" :class="{ 'icon-loading': loading }">
		<!-- Error message -->
		<EmptyContent v-if="error" icon="icon-error">
			{{ error }}
		</EmptyContent>

		<template v-else>
			<!-- Editor -->
			<Comment v-bind="editorData"
				:auto-complete="autoComplete"
				:editor="true"
				:file-info="fileInfo"
				:message.sync="editorData.message"
				class="comments__writer"
				@new="onNewComment" />

			<EmptyContent v-if="comments.length === 0 && !loading" icon="icon-comment">
				{{ t('comments', 'No comments yet, start the conversation!') }}
			</EmptyContent>

			<!-- Comments -->
			<Comment v-for="comment in comments"
				v-else
				:key="comment.id"
				v-bind="comment"
				:auto-complete="autoComplete"
				:file-info="fileInfo"
				:message.sync="comment.message"
				class="comments__list"
				@delete="onDelete" />
		</template>
	</div>
</template>

<script>
import { generateOcsUrl } from '@nextcloud/router'
import axios from '@nextcloud/axios'
import VTooltip from 'v-tooltip'
import Vue from 'vue'

import EmptyContent from '@nextcloud/vue/dist/Components/EmptyContent'

import Comment from '../components/Comment'
import getComments from '../services/GetComments'
import { getCurrentUser } from '@nextcloud/auth'

Vue.use(VTooltip)

export default {
	name: 'CommentsTab',

	components: {
		// Avatar,
		Comment,
		EmptyContent,
	},

	data() {
		return {
			error: '',
			loading: true,

			fileInfo: null,
			offset: 0,
			comments: [],

			message: '',
			editorData: {
				actorDisplayName: getCurrentUser().displayName,
				actorId: getCurrentUser().uid,
			},

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
				this.comments = await getComments(this.fileInfo.id)
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
		 * Add newly created comment to the list
		 * @param {Object} comment the new comment
		 */
		onNewComment(comment) {
			this.comments.unshift(comment)
			this.message = ''
		},

		/**
		 * Remove deleted comment from the list
		 * @param {number} id the deleted comment
		 */
		onDelete(id) {
			const index = this.comments.findIndex(comment => comment.id === id)
			if (index > -1) {
				this.comments.splice(index, 1)
			} else {
				console.error('Could not find the deleted comment in the list', id)
			}
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
