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
	<div class="comment">
		<div class="comment__header">
			<Avatar class="comment__avatar" :display-name="source.actorDisplayName" :user="source.actorId" />
			<span class="comment__author">{{ source.actorDisplayName }}</span>
			<Actions class="comment__actions">
				<ActionButton icon="icon-rename">
					{{ t('comments', 'Edit comment') }}
				</ActionButton>
				<ActionButton icon="icon-delete">
					{{ t('comments', 'Delete comment') }}
				</ActionButton>
			</Actions>
			<span class="comment__timestamp"></span>
		</div>
		<div class="comment__message">
			{{ source.message }}
		</div>
	</div>
</template>

<script>
import Avatar from '@nextcloud/vue/dist/Components/Avatar'
import Actions from '@nextcloud/vue/dist/Components/Actions'
import ActionButton from '@nextcloud/vue/dist/Components/ActionButton'

export default {
	name: 'Comment',
	props: {
		source: {
			type: Object,
			default: () => ({}),
		},
	},

	components: {
		ActionButton,
		Actions,
		Avatar,
	},

}
</script>

<style lang="scss" scoped>

</style>
