<!--
  - @copyright Copyright (c) 2018 René Gieling <github@dartcafe.de>
  -
  - @author René Gieling <github@dartcafe.de>
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
  - along with this program.  If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<ConfigBox :title="t('polls', 'Public shares')" icon-class="icon-public">
		<TransitionGroup :css="false" tag="div" class="shared-list">
			<PublicShareItem v-for="(share) in publicShares"
				:key="share.id"
				v-bind="share">
				<Actions>
					<ActionButton icon="icon-clippy" @click="copyLink( { url: share.URL })">
						{{ t('polls', 'Copy link to clipboard') }}
					</ActionButton>
				</Actions>
				<Actions>
					<ActionButton icon="icon-delete" @click="removeShare(share)">
						{{ t('polls', 'Remove share') }}
					</ActionButton>
				</Actions>
			</PublicShareItem>
		</TransitionGroup>

		<ButtonDiv :title="t('polls', 'Add a public link')" icon="icon-add" @click="addShare({type: 'public', userId: '', emailAddress: ''})" />
	</ConfigBox>
</template>

<script>
import { mapGetters } from 'vuex'
import { showSuccess, showError } from '@nextcloud/dialogs'
import { Actions, ActionButton } from '@nextcloud/vue'
import ButtonDiv from '../Base/ButtonDiv'
import ConfigBox from '../Base/ConfigBox'
import PublicShareItem from './PublicShareItem'

export default {
	name: 'SharesPublic',

	components: {
		Actions,
		ActionButton,
		ButtonDiv,
		ConfigBox,
		PublicShareItem,
	},

	computed: {
		...mapGetters({
			publicShares: 'poll/shares/public',
		}),
	},

	methods: {
		copyLink(payload) {
			this
				.$copyText(payload.url)
				.then(() => {
					showSuccess(t('polls', 'Link copied to clipboard'))
				})
				.catch(() => {
					showError(t('polls', 'Error while copying link to clipboard'))
				})
		},

		removeShare(share) {
			this.$store.dispatch('poll/shares/delete', { share: share })
		},

		addShare(payload) {
			this.$store
				.dispatch('poll/shares/add', {
					share: payload,
					type: payload.type,
					id: payload.id,
					emailAddress: payload.emailAddress,
				})
				.catch(error => {
					console.error('Error while adding share - Error: ', error)
					showError(t('polls', 'Error while adding share'))
				})
		},
	},
}
</script>

<style lang="scss">
	.shared-list {
		display: flex;
		flex-wrap: wrap;
		flex-direction: column;
		justify-content: flex-start;
		padding-top: 8px;

		> li {
			display: flex;
			align-items: stretch;
			margin: 4px 0;
		}
	}

	.share-item {
		display: flex;
		flex: 1;
		align-items: center;
		max-width: 100%;
	}
</style>
