<!--
SPDX-FileCopyrightText: Linca
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div class="_gaps_m">
	<MkInfo>
		{{ i18n.ts._stpvPlus.info }}
	</MkInfo>

	<FormSection first>
		<template #label>{{ i18n.ts.appearance }}</template>

		<div class="_gaps_m">
			<div class="_gaps_s">
				<MkSelect v-model="defaultFont.fontFace">
					<template #label>{{ i18n.ts._stpvPlus.defaultFont.label }}</template>
					<template #caption>
						{{ i18n.ts._stpvPlus.defaultFont.caption }}
					</template>
					<option
						v-for="item in defaultFont.fontList"
						:key="item.id"
						:value="item.id"
					>
						{{ item.name }}
					</option>
				</MkSelect>
				<MkRadios v-if="defaultFont.availableTypes.length > 0" v-model="defaultFont.fontFaceType">
					<template #label>{{ i18n.ts._stpvPlus.fontType.label }}</template>
					<template #caption>
						{{ i18n.ts._stpvPlus.fontType.caption }}
					</template>
					<option
						v-for="item in defaultFont.availableTypes"
						:key="item.id"
						:value="item.id"
					>
						{{ item.name }}
					</option>
				</MkRadios>
			</div>
			<div class="_gaps_s">
				<MkSwitch v-model="stpvDisableAllReactions">
					{{ i18n.ts._stpvPlus.disableAllReactions.label }}
					<template #caption>{{ i18n.ts._stpvPlus.disableAllReactions.caption }}</template>
				</MkSwitch>
			</div>
			<div v-show="collapsedInReplyTo" class="_gaps_s">
				<MkSwitch v-model="stpvHideReplyAcct">
					{{ i18n.ts._stpvPlus.hideReplyAcct.label }}
					<template #caption>{{ i18n.ts._stpvPlus.hideReplyAcct.caption }}</template>
				</MkSwitch>
			</div>
		</div>
	</FormSection>

	<FormSection>
		<template #label>{{ i18n.ts.behavior }}</template>
		<div class="_gaps_m">
			<div class="_gaps_s">
				<MkSwitch v-model="stpvAdvancedPostForm">
					{{ i18n.ts._stpvPlus.advancedPostForm.label }}
					<template #caption>{{ i18n.ts._stpvPlus.advancedPostForm.caption }}</template>
				</MkSwitch>
			</div>
			<div class="_gaps_s">
				<MkSwitch v-model="stpvCombineRepliesQuotes">
					{{ i18n.ts._stpvPlus.combineRepliesQuotes.label }}
					<template #caption>{{ i18n.ts._stpvPlus.combineRepliesQuotes.caption }}</template>
				</MkSwitch>
			</div>
			<div class="_gaps_s">
				<MkRadios v-model="autoSpacingBehaviour">
					<template #label>自动空格</template>
					<option :value="null">{{ i18n.ts.disabled }}</option>
					<option value="all">{{ i18n.ts.all }}</option>
					<option value="special">智能</option>
					<template #caption>在帖子正文的中文与英文之间自动加入缺失的空格。当选择“智能”时，一部分通常认为是混合词的（B超，X光等）会被保留</template>
				</MkRadios>
			</div>
		</div>
	</FormSection>

	<FormSection>
		<template #label>{{ i18n.ts.muteAndBlock }}</template>
		<div class="_gaps_m">
			<div class="_gaps_s">
				<MkTextarea v-model="stpvMutedUsersList" manualSave>
					<template #label>{{ i18n.ts._stpvPlus.softMutedUsers.label }}</template>
					<template #caption>{{ i18n.ts._stpvPlus.softMutedUsers.caption }}</template>
				</MkTextarea>
			</div>
			<div class="_gaps_s">
				<MkTextarea v-model="stpvMutedNotesList" manualSave>
					<template #label>{{ i18n.ts._stpvPlus.softMutedNotes.label }}</template>
					<template #caption>{{ i18n.ts._stpvPlus.softMutedNotes.caption }}</template>
				</MkTextarea>
			</div>
			<div class="_gaps_s">
				<MkTextarea v-model="stpvMutedDomainsList" manualSave>
					<template #label>{{ i18n.ts._stpvPlus.softMutedDomains.label }}</template>
					<template #caption>{{ i18n.ts._stpvPlus.softMutedDomains.caption }}</template>
				</MkTextarea>
			</div>
		</div>
	</FormSection>

	<FormSection>
		<template #label>{{ i18n.ts.operations }}</template>
		<div class="_gaps_m">
			<div class="_gaps_s">
				<MkButton link to="/make-private-many"><i class="ph-eye-slash ph-bold ph-lg"></i> {{ i18n.ts.makePrivate.bulkText }}</MkButton>
			</div>
			<div class="_gaps_s">
				<MkButton link :to="`/stpv/reactions-stat`"><i class="ph-chart-bar ph-bold ph-lg"></i> {{ i18n.ts.stpvReactionsStat }} </MkButton>
			</div>
		</div>
	</FormSection>
</div>
</template>

<script lang="ts" setup>
import { computed, ref, watch } from 'vue';
import MkSwitch from '@/components/MkSwitch.vue';
import FormLink from '@/components/form/link.vue';
import MkFolder from '@/components/MkFolder.vue';
import FormInfo from '@/components/MkInfo.vue';
import MkKeyValue from '@/components/MkKeyValue.vue';
import MkButton from '@/components/MkButton.vue';
import MkRadios from '@/components/MkRadios.vue';
import MkSelect from '@/components/MkSelect.vue';
import * as os from '@/os.js';
import { misskeyApi } from '@/scripts/misskey-api.js';
import { defaultStore } from '@/store.js';
import { signout, signinRequired } from '@/account.js';
import { i18n } from '@/i18n.js';
import { definePageMetadata } from '@/scripts/page-metadata.js';
import { unisonReload } from '@/scripts/unison-reload.js';
import FormSection from '@/components/form/section.vue';
import MkInfo from '@/components/MkInfo.vue';
import { getDefaultFontSettings } from '@/scripts/font-settings';
import MkTextarea from '@/components/MkTextarea.vue';

const $i = signinRequired();
const meId = $i.id;

const defaultFont = getDefaultFontSettings();
console.log(defaultFont);

const collapsedInReplyTo = defaultStore.reactiveState.collapseNotesRepliedTo;

const autoSpacingBehaviour = computed(defaultStore.makeGetterSetter('chineseAutospacing'));
const stpvDisableAllReactions = computed(defaultStore.makeGetterSetter('stpvDisableAllReactions'));
const stpvHideReplyAcct = computed(defaultStore.makeGetterSetter('stpvHideReplyAcct'));
const stpvAdvancedPostForm = computed(defaultStore.makeGetterSetter('stpvAdvancedPostForm'));
const stpvCombineRepliesQuotes = computed(defaultStore.makeGetterSetter('stpvCombineRepliesQuotes'));

const stpvMutedUsersList = computed({
	get: () => defaultStore.reactiveState.stpvClientMutedUsers.value.filter(x => x).join('\n'),
	set: (v) => {
		defaultStore.set('stpvClientMutedUsers', v.split('\n').filter(x => x.trim()).slice(0, 100));
	},
});
const stpvMutedNotesList = computed({
	get: () => defaultStore.reactiveState.stpvClientMutedNotes.value.filter(x => x).join('\n'),
	set: (v) => {
		defaultStore.set('stpvClientMutedNotes', v.split('\n').filter(x => x.trim()).slice(0, 100));
	},
});
const stpvMutedDomainsList = computed({
	get: () => defaultStore.reactiveState.stpvClientMutedDomains.value.filter(x => x).join('\n'),
	set: (v) => {
		defaultStore.set('stpvClientMutedNotes', v.split('\n').filter(x => x.trim()).slice(0, 100));
	},
});

// const headerActions = computed(() => []);

// const headerTabs = computed(() => []);

definePageMetadata(() => ({
	title: i18n.ts._stpvPlus.title,
	icon: 'ti ti-dots',
}));
</script>

