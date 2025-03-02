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
	<div class="poll-information">
		<UserBubble v-if="poll.owner" :user="poll.owner" :display-name="poll.ownerDisplayName" />
		{{ t('polls', 'started this poll on {dateString}.', {dateString: dateCreatedString}) }}

		<span v-if="closed && confirmedOptions.length"> {{ t('polls', 'This poll is closed since {dateString}. The confirmed options are marked below.', { dateString: dateExpiryString }) }} </span>

		<span v-if="closed && !confirmedOptions.length"> {{ t('polls', 'This poll is closed since {dateString}, but there are no confirmed options until now.', { dateString: dateExpiryString }) }} </span>

		<span v-if="closed && !confirmedOptions.length && acl.allowEdit"> {{ t('polls', 'You can confirm your favorites now in the options tab in the sidebar.', { dateString: dateExpiryString }) }} </span>

		<span v-if="!closed && poll.expire && acl.allowVote">{{ t('polls', 'You can place your vote until {dateString}.', { dateString: dateExpiryString }) }} </span>

		<span v-if="poll.anonymous">{{ t('polls', 'This is an anonymous poll. Except to the poll owner, participants names are hidden.') }} </span>

		<span v-if="!acl.allowSeeResults">{{ t('polls', 'Results are hidden.') }}</span>

		<span v-if="!acl.allowSeeResults && (poll.showResults === 'closed')">{{ t('polls', 'They will be revealed after the poll is closed.') }}</span>

		<span v-if="poll.type === 'datePoll'">{{ t('polls', 'The used time zone is {timeZone}.', { timeZone: currentTimeZone }) }}</span>
		<div v-if="poll.voteLimit">
			{{ t('polls', 'Your are only allowed to vote for one option.', 'Your votes are limited to %n yes votes.', poll.voteLimit) }}
			<span v-if="voteLimitReached"> {{ t('polls', 'You reached the maximum number of allowed votes.') }}</span>
			<span v-else-if="poll.voteLimit - countYesVotes === 1"> {{ t('polls', 'You have only one vote left.') }}</span>
			<span v-else> {{ n('polls', 'You have %n vote left.', 'You have %n votes left.', poll.voteLimit - countYesVotes) }}</span>
		</div>

		<div v-if="poll.optionLimit === 1">
			{{ t('polls', 'This is an exclusive vote, where only one user is allowed to vote for an option.') }}
		</div>
		<div v-else-if="poll.optionLimit">
			{{ n('polls', 'Only %n vote per option is permitted.', 'Only %n votes per option are permitted.', poll.optionLimit) }}
		</div>
	</div>
</template>

<script>
import { mapState, mapGetters } from 'vuex'
import moment from '@nextcloud/moment'
import { UserBubble } from '@nextcloud/vue'

export default {
	name: 'PollInformation',

	components: {
		UserBubble,
	},

	computed: {
		...mapState({
			acl: state => state.poll.acl,
			poll: state => state.poll,
		}),

		...mapGetters({
			participantsVoted: 'poll/participantsVoted',
			closed: 'poll/closed',
			confirmedOptions: 'poll/options/confirmed',
			countYesVotes: 'poll/votes/countYesVotes',
		}),

		voteLimitReached() {
			return (this.poll.voteLimit > 0 && this.countYesVotes >= this.poll.voteLimit)
		},

		dateCreatedString() {
			return moment.unix(this.poll.created).format('LLLL')
		},

		dateExpiryString() {
			return moment.unix(this.poll.expire).format('LLLL')
		},

		currentTimeZone() {
			return Intl.DateTimeFormat().resolvedOptions().timeZone
		},

	},
}
</script>
