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

<template lang="html">
	<AppNavigation>
		<AppNavigationNew button-class="icon-add" :text="t('polls', 'Add new Poll')" @click="toggleCreateDlg" />
		<CreateDlg v-show="createDlg" ref="createDlg" @close-create="closeCreate()" />
		<template #list>
			<AppNavigationItem v-for="(pollCategory) in pollCategories" :key="pollCategory.id"
				:title="pollCategory.title" :allow-collapse="true" :pinned="pollCategory.pinned"
				:icon="pollCategory.icon" :to="{ name: 'list', params: {type: pollCategory.id}}" :open="false">
				<ul>
					<PollNavigationItems v-for="(poll) in filteredPolls(pollCategory.id)"
						:key="poll.id"
						:poll="poll"
						@switch-deleted="switchDeleted(poll.id)"
						@clone-poll="clonePoll(poll.id)"
						@delete-permanently="deletePermanently(poll.id)" />
				</ul>
			</AppNavigationItem>
		</template>
		<template #footer>
			<AppNavigationItem v-if="showAdminSection"
				:title="t('core', 'Administration')"
				icon="icon-settings"
				:to="{ name: 'administration' }" />
			<AppNavigationItem :title="t('core', 'Settings')" icon="icon-settings" @click="showSettings()" />
		</template>
	</AppNavigation>
</template>

<script>

import { AppNavigation, AppNavigationNew, AppNavigationItem } from '@nextcloud/vue'
import { mapGetters } from 'vuex'
import { showError } from '@nextcloud/dialogs'
import { getCurrentUser } from '@nextcloud/auth'
import CreateDlg from '../Create/CreateDlg'
import PollNavigationItems from './PollNavigationItems'
import { emit } from '@nextcloud/event-bus'

export default {
	name: 'Navigation',
	components: {
		AppNavigation,
		AppNavigationNew,
		AppNavigationItem,
		CreateDlg,
		PollNavigationItems,
	},

	data() {
		return {
			showSettingsDlg: false,
			createDlg: false,
			// reloadInterval: 30000,
			pollCategories: [
				{
					id: 'relevant',
					title: t('polls', 'Relevant'),
					icon: 'icon-details',
					pinned: false,
				},
				{
					id: 'my',
					title: t('polls', 'My polls'),
					icon: 'icon-user',
					pinned: false,
				},
				{
					id: 'public',
					title: t('polls', 'Public polls'),
					icon: 'icon-link',
					pinned: false,
				},
				{
					id: 'all',
					title: t('polls', 'All polls'),
					icon: 'icon-polls',
					pinned: false,
				},
				{
					id: 'closed',
					title: t('polls', 'Closed polls'),
					icon: 'icon-polls-closed',
					pinned: false,
				},
				{
					id: 'deleted',
					title: t('polls', 'Deleted polls'),
					icon: 'icon-delete',
					pinned: true,
				},
			],

		}
	},

	computed: {
		...mapGetters({
			filteredPolls: 'polls/filtered',
		}),
		showAdminSection() {
			return getCurrentUser().isAdmin
		},
	},

	// created() {
	// 	this.timedReload()
	// },
	//
	// beforeDestroy() {
	// 	window.clearInterval(this.reloadTimer)
	// },

	methods: {
		closeCreate() {
			this.createDlg = false
		},

		// timedReload() {
		// 	// reload poll list periodically
		// 	this.reloadTimer = window.setInterval(() => {
		// 		emit('update-polls')
		// 	}, this.reloadInterval)
		// },

		toggleCreateDlg() {
			this.createDlg = !this.createDlg
			if (this.createDlg) {
				this.$refs.createDlg.setFocus()
			}
		},

		clonePoll(pollId) {
			this.$store
				.dispatch('poll/clone', { pollId: pollId })
				.then((response) => {
					emit('update-polls')
					this.$router.push({ name: 'vote', params: { id: response.id } })
				})
				.catch(() => {
					showError(t('polls', 'Error cloning poll.'))
				})
		},

		showSettings() {
			emit('show-settings')
		},

		switchDeleted(pollId) {
			this.$store
				.dispatch('poll/switchDeleted', { pollId: pollId })
				.then(() => {
					emit('update-polls')
				})
				.catch(() => {
					showError(t('polls', 'Error deleting poll.'))
				})

		},

		deletePermanently(pollId) {
			this.$store
				.dispatch('poll/delete', { pollId: pollId })
				.then(() => {
					// if we permanently delete current selected poll,
					// reload deleted polls route
					if (this.$route.params.id && this.$route.params.id === pollId) {
						this.$router.push({ name: 'list', params: { type: 'deleted' } })
					}
					emit('update-polls')
				})
				.catch(() => {
					showError(t('polls', 'Error deleting poll.'))
				})

		},
	},
}
</script>

<style lang="scss">
	.closed {
		.app-navigation-entry-icon, .app-navigation-entry__title {
			opacity: 0.6;
		}
	}

</style>
