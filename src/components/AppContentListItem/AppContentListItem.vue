<!--
  - @copyright Copyright (c) 2019 Marco Ambrosini <marcoambrosini@pm.me>
  -
  - @author Marco Ambrosini <marcoambrosini@pm.me>
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
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
-->

<template>
	<nav-element
		class="acli_wrapper"
		v-bind="navElement">
		<a
			:id="anchorId"
			ref="acli"
			:class="{ 'active' : active }"
			href="#"
			class="acli"
			:aria-label="conversationLinkAriaLabel"
			@mouseover="handleHover"
			@focus="handleFocus"
			@blur="handleBlur"
			@mouseleave="handleMouseleave"
			@keydown.tab="handleTab"
			@click="onClick"
			@keydown.esc="displayActions = false">
			<!-- default slot for avatar or icon -->
			<slot name="icon" />
			<div class="acli-content">
				<div class="acli-content__main">
					<div class="acli-content__line-one">
						<span
							class="acli-content__line-one__title"
							:class="{'bold': bold}">
							{{ title }}
						</span>
						<span
							v-if="hasDetails && !displayActions"
							class="acli-content__line-one__details">
							{{ details }}
						</span>
					</div>
					<div class="acli-content__line-two"
						:class="{'bold': bold}">
						<span v-if="hasSubtitle" class="acli-content__line-two__subtitle">
							<slot name="subtitle" />
						</span>
						<span v-if="!displayActions" class="acli-content__line-two__counter">
							<slot
								name="counter" />
						</span>
					</div>
				</div>
				<div
					v-show="displayActions"
					class="acli-content__actions"
					@click.prevent.stop="">
					<Actions
						ref="actions"
						menu-align="right"
						:aria-label="conversationSettingsAriaLabel"
						@update:open="handleActionsUpdateOpen">
						<slot
							name="actions" />
					</Actions>
				</div>
			</div>
		</a>
	</nav-element>
</template>

<script>
import Actions from '../Actions'

export default {
	name: 'AppContentListItem',

	components: {
		Actions,
	},

	props: {
		/**
		 * The details text displayed in the upper right part
		 */
		details: {
			type: String,
			default: '',
		},
		/**
		 * Title
		 */
		title: {
			type: String,
			required: true,
		},
		/**
		* Pass in `true` if you want the matching behaviour to
		* be non-inclusive: https://router.vuejs.org/api/#exact
		*/
		exact: {
			type: Boolean,
			default: false,
		},
		/**
		* The route for the router link.
		*/
		to: {
			type: [String, Object],
			default: '',
		},
		/**
		 * Id for the <a> element
		 */
		anchorId: {
			type: String,
			default: '',
		},

		/**
		 * Bold title and subtitle
		 */
		bold: {
			type: Boolean,
			default: false,
		},

		/**
		 * Toggle the active state of the component.
		 */
		active: {
			type: Boolean,
			default: false,
		},
	},

	data() {
		return {
			hovered: false,
			focused: false,
			displayActions: false,
			menuOpen: false,
		}
	},

	computed: {

		hasDetails() {
			return this.details !== ''
		},

		hasActions() {
			return (!!this.$slots.actions)
		},

		// This is used to decide which outer element type to use
		// li or router-link
		navElement() {
			if (this.to !== '') {
				return {
					is: 'router-link',
					tag: 'li',
					to: this.to,
					exact: this.exact,
				}
			}
			return {
				is: 'li',
			}
		},

		conversationLinkAriaLabel() {
			return t('spreed', 'Conversation "{conversationName}"', { conversationName: this.title })
		},

		conversationSettingsAriaLabel() {
			return t('spreed', 'Settings for conversation "{conversationName}"', { conversationName: this.title })
		},

		// Check if the subtitle slot is populated
		hasSubtitle() {
			return !!this.$slots.subtitle
		},
	},

	watch: {
		menuOpen(newValue) {
			console.debug('menuopen', newValue)
			// A click outside both the menu and the root element hides the actions again
			if (!newValue && !this.hovered) {
				this.displayActions = false
			}
		},
	},

	methods: {

		// forward click event
		onClick(event) {
			this.$emit('click', event)
		},

		handleHover() {
			this.showActions()
		},

		handleFocus() {
			this.focused = true
			this.showActions()
		},

		handleBlur(e) {
			this.focused = false
		},

		showActions() {
			if (this.hasActions) {
				this.displayActions = true
			}
		},

		handleMouseleave() {
			if (!this.menuOpen) {
				this.displayActions = false
			}
		},

		handleTab(e) {
			if (this.focused && this.hasActions) {
				console.debug('preventdefault')
				e.preventDefault()
				this.$refs.actions.$refs.menuButton.focus()
				this.focused = false
			} else {
				this.displayActions = false
				this.$refs.actions.$refs.menuButton.blur()
			}
		},

		handleActionsUpdateOpen(e) {
			this.menuOpen = e
		},
	},
}
</script>

<style lang="scss" scoped>

.acli_wrapper{
	position: relative;
	width: 100%;
	.actions {
		position: absolute;
		top: 4px;
		right: 12px;
	}
}

// AppContentListItem
.acli {
	position: relative;
	display: flex;
	align-items: center;
	flex: 0 0 auto;
	justify-content: flex-start;
	padding: 2px 2px 2px 8px;
	height: 64px;
	border-radius: 16px;
	margin: 2px 0;
	width: 100%;
	cursor: pointer;
	transition: background-color 200ms ease-in-out;
	&:hover,
	&:focus  {
		background-color: var(--color-background-hover);
	}
	&.active,
	&:active,
	&:active ~ .app-navigation-entry__utils {
		background-color: var(--color-primary-light);
	}

	&-content {
		display: flex;
		flex: 1 1 auto;
		justify-content: space-between;
		padding: 0 8px;

		&__main {
			flex: 1 1 auto;
			flex-direction: column;
			width: 0;
			margin: auto 0;
		}

		&__line-one {
			display: flex;
			align-items: center;
			justify-content: space-between;
			white-space: nowrap;
			margin: 0 auto;

			&__title {
				overflow: hidden;
				flex-grow: 1;
				cursor: pointer;
				text-overflow: ellipsis;
				color: var(--color-main-text);
			}

			&__details {
				color: var(--color-text-lighter);
				margin: 0 8px;
			}
		}

		&__line-two {
			display: flex;
			align-items: flex-start;
			justify-content: space-between;
			white-space: nowrap;

			&__subtitle {
				overflow: hidden;
				flex-grow: 1;
				padding-right: 4px;
				cursor: pointer;
				white-space: nowrap;
				text-overflow: ellipsis;
				color: var(--color-text-lighter);
			}
		}
		&__actions {
			flex: 0 0 auto;
			align-self: center;
			justify-content: center;

		}
	}
}

// Counter
::v-deep .counter {
	font-size: 12px;
	/*
	 * Always add the bubble
	 */
	padding: 4px 6px !important;
	border-radius: 10px;

	&:not(.app-navigation-entry__counter--highlighted) {
		background-color: var(--color-background-darker);
	}

	span {
		padding: 2px 6px;
	}
}

.bold {
	font-weight: bold;
}

</style>
