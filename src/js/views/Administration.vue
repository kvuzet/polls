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
	<AppContent class="poll-list">
		<div class="area__header">
			<h2 class="title">
				{{ title }}
			</h2>
		</div>

		<div class="area__main">
			<div>
				<h2 class="title">
					{{ t('polls', 'Manage polls') }}
				</h2>
				<h3 class="description">
					{{ t('polls', 'Manage polls of other users. You can take over the ownership or delete polls.') }}
				</h3>
			</div>

			<EmptyContent v-if="noPolls" icon="icon-polls">
				{{ t('polls', 'No polls found for this category') }}
				<template #desc>
					{{ t('polls', 'Add one or change category!') }}
				</template>
			</EmptyContent>

			<transition-group v-else
				name="list"
				tag="div"
				class="poll-list__list">
				<PollItem key="0" :header="true"
					:sort="sort" :reverse="reverse" @sort-list="setSort($event)" />

				<PollItem v-for="(poll) in sortedList" :key="poll.id" :poll="poll">
					<template #actions>
						<Actions :force-menu="true">
							<ActionButton icon="icon-add" :close-after-click="true"
								@click="confirmTakeOver(poll.id, poll.owner)">
								{{ t('polls', 'Take over') }}
							</ActionButton>

							<ActionButton :icon="poll.deleted ? 'icon-history' : 'icon-delete'" :close-after-click="true"
								@click="switchDeleted(poll.id)">
								{{ poll.deleted ? t('polls', 'Restore poll') : t('polls', 'Set "deleted" status') }}
							</ActionButton>

							<ActionButton icon="icon-delete" class="danger" :close-after-click="true"
								@click="confirmDelete(poll.id)">
								{{ t('polls', 'Delete poll permanently') }}
							</ActionButton>
						</Actions>
					</template>
				</PollItem>
			</transition-group>
		</div>
		<LoadingOverlay v-if="isLoading" />
		<Modal v-if="takeOverModal" @close="takeOverModal = false">
			<div class="modal__content">
				<h2>{{ t('polls', 'Do you want to take over this poll from {username} and change the ownership?', {username: takeOverOwner}) }}</h2>
				<div>{{ t('polls', 'The original owner will be notified.') }}</div>
				<div class="modal__buttons">
					<ButtonDiv :title="t('polls', 'No')"
						@click="takeOverModal = false" />
					<ButtonDiv :primary="true" :title="t('polls', 'Yes')"
						@click="takeOver()" />
				</div>
			</div>
		</Modal>
		<Modal v-if="deleteModal" @close="deleteModal = false">
			<div class="modal__content">
				<h2>{{ t('polls', 'Do you want to delete this poll?') }}</h2>
				<div>{{ t('polls', 'This action cannot be reverted.') }}</div>
				<div>{{ t('polls', 'The original owner will be notified.') }}</div>
				<div class="modal__buttons">
					<ButtonDiv :title="t('polls', 'No')"
						@click="deleteModal = false" />
					<ButtonDiv :primary="true" :title="t('polls', 'Yes')"
						@click="deletePermanently()" />
				</div>
			</div>
		</Modal>
	</AppContent>
</template>

<script>
import { mapGetters } from 'vuex'
import { showError } from '@nextcloud/dialogs'
import { emit } from '@nextcloud/event-bus'
import { Actions, ActionButton, AppContent, EmptyContent, Modal } from '@nextcloud/vue'
import sortBy from 'lodash/sortBy'
import LoadingOverlay from '../components/Base/LoadingOverlay'
import PollItem from '../components/PollList/PollItem'

export default {
	name: 'Administration',

	components: {
		AppContent,
		Actions,
		ActionButton,
		LoadingOverlay,
		PollItem,
		EmptyContent,
		Modal,
	},

	data() {
		return {
			isLoading: false,
			sort: 'created',
			reverse: true,
			takeOverModal: false,
			takeOverOwner: '',
			takeOverPollId: 0,
			deleteModal: false,
			deletePollId: 0,
		}
	},

	computed: {
		...mapGetters({
			filteredPolls: 'pollsAdmin/filtered',
		}),

		title() {
			return t('polls', 'Administration')
		},

		windowTitle() {
			return t('polls', 'Polls') + ' - ' + this.title
		},

		sortedList() {
			if (this.reverse) {
				return sortBy(this.filteredPolls(this.$route.params.type), this.sort).reverse()
			} else {
				return sortBy(this.filteredPolls(this.$route.params.type), this.sort)
			}
		},

		noPolls() {
			return this.sortedList.length < 1
		},

	},

	watch: {
		$route() {
			this.refreshView()
		},
	},

	mounted() {
		this.refreshView()
	},

	methods: {
		confirmTakeOver(pollId, owner) {
			this.takeOverPollId = pollId
			this.takeOverOwner = owner
			this.takeOverModal = true
		},

		confirmDelete(pollId) {
			this.deletePollId = pollId
			this.deleteModal = true
		},

		switchDeleted(pollId) {
			this.$store
				.dispatch('poll/switchDeleted', { pollId: pollId })
				.then(() => {
					emit('update-polls')
				})
				.catch(() => {
					showError(t('polls', 'Error switching deleted status.'))
				})
		},

		deletePermanently() {
			this.$store
				.dispatch('poll/delete', { pollId: this.deletePollId })
				.then(() => {
					emit('update-polls')
					this.deleteModal = false
				})
				.catch(() => {
					showError(t('polls', 'Error deleting poll.'))
					this.deleteModal = false
				})
		},

		takeOver() {
			this.$store
				.dispatch('pollsAdmin/takeOver', { pollId: this.takeOverPollId })
				.then(() => {
					emit('update-polls')
					this.takeOverModal = false
				})
				.catch(() => {
					showError(t('polls', 'Error overtaking poll.'))
					this.takeOverModal = false
				})
		},

		refreshView() {
			window.document.title = t('polls', 'Polls') + ' - ' + this.title
			if (!this.filteredPolls(this.$route.params.type).find(poll => {
				return poll.id === this.$store.state.poll.id
			})) {
				emit('toggle-sidebar', { open: false })
			}

		},

		setSort(payload) {
			if (this.sort === payload.sort) {
				this.reverse = !this.reverse
			} else {
				this.sort = payload.sort
				this.reverse = true
			}
		},

	},
}
</script>

<style lang="scss" scoped>
	.area__header {
		margin-left: 33px !important;
	}

	.poll-list__list {
		width: 100%;
		display: flex;
		flex-direction: column;
		flex-wrap: nowrap;
		overflow: scroll;
		padding-bottom: 14px;
	}
</style>
