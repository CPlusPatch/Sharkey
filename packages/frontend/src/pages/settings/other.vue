<!--
SPDX-FileCopyrightText: syuilo and other misskey contributors
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div class="_gaps_m">
	<!--
	<MkSwitch v-model="$i.injectFeaturedNote" @update:model-value="onChangeInjectFeaturedNote">
		<template #label>{{ i18n.ts.showFeaturedNotesInTimeline }}</template>
	</MkSwitch>
	-->

	<!--
	<MkSwitch v-model="reportError">{{ i18n.ts.sendErrorReports }}<template #caption>{{ i18n.ts.sendErrorReportsDescription }}</template></MkSwitch>
	-->

	<FormSection first>
		<div class="_gaps_s">
			<MkFolder>
				<template #icon><i class="ph-info ph-bold ph-lg"></i></template>
				<template #label>{{ i18n.ts.accountInfo }}</template>

				<div class="_gaps_m">
					<MkKeyValue>
						<template #key>ID</template>
						<template #value><span class="_monospace">{{ $i.id }}</span></template>
					</MkKeyValue>

					<MkKeyValue>
						<template #key>{{ i18n.ts.registeredDate }}</template>
						<template #value><MkTime :time="$i.createdAt" mode="detail"/></template>
					</MkKeyValue>
				</div>
			</MkFolder>

			<MkFolder>
				<template #icon><i class="ph-database ph-bold ph-lg"></i></template>
				<template #label>{{ i18n.ts._dataRequest.title }}</template>

				<div class="_gaps_m">
					<FormInfo warn>{{ i18n.ts._dataRequest.warn }}</FormInfo>
					<FormInfo>{{ i18n.ts._dataRequest.text }}</FormInfo>
					<MkButton primary @click="exportData">{{ i18n.ts._dataRequest.button }}</MkButton>
				</div>
			</MkFolder>

			<MkFolder>
				<template #icon><i class="ph-warning ph-bold ph-lg"></i></template>
				<template #label>{{ i18n.ts.closeAccount }}</template>

				<div class="_gaps_m">
					<FormInfo warn>{{ i18n.ts._accountDelete.mayTakeTime }}</FormInfo>
					<FormInfo>{{ i18n.ts._accountDelete.sendEmail }}</FormInfo>
					<MkButton v-if="!$i.isDeleted" danger @click="deleteAccount">{{ i18n.ts._accountDelete.requestAccountDelete }}</MkButton>
					<MkButton v-else disabled>{{ i18n.ts._accountDelete.inProgress }}</MkButton>
				</div>
			</MkFolder>

			<MkFolder>
				<template #icon><i class="ph-flask ph-bold ph-lg"></i></template>
				<template #label>{{ i18n.ts.experimentalFeatures }}</template>

				<div class="_gaps_m">
					<MkSwitch v-model="enableCondensedLineForAcct">
						<template #label>Enable condensed line for acct</template>
					</MkSwitch>
				</div>
			</MkFolder>

			<MkFolder>
				<template #icon><i class="ph-code ph-bold ph-lg"></i></template>
				<template #label>{{ i18n.ts.developer }}</template>

				<div class="_gaps_m">
					<MkSwitch v-model="devMode">
						<template #label>{{ i18n.ts.devMode }}</template>
					</MkSwitch>
				</div>
			</MkFolder>
		</div>
	</FormSection>

	<FormSection>
		<FormLink to="/registry"><template #icon><i class="ph-faders ph-bold ph-lg"></i></template>{{ i18n.ts.registry }}</FormLink>
	</FormSection>

	<FormSection>
		<div class="_gaps_s">
			<MkSwitch v-model="defaultWithReplies">{{ i18n.ts.withRepliesByDefaultForNewlyFollowed }}</MkSwitch>
			<MkButton danger @click="updateRepliesAll(true)"><i class="ph-chats ph-bold ph-lg"></i> {{ i18n.ts.showRepliesToOthersInTimelineAll }}</MkButton>
			<MkButton danger @click="updateRepliesAll(false)"><i class="ph-chat ph-bold ph-lg"></i> {{ i18n.ts.hideRepliesToOthersInTimelineAll }}</MkButton>
		</div>
	</FormSection>
</div>
</template>

<script lang="ts" setup>
import { computed, watch } from 'vue';
import MkSwitch from '@/components/MkSwitch.vue';
import FormLink from '@/components/form/link.vue';
import MkFolder from '@/components/MkFolder.vue';
import FormInfo from '@/components/MkInfo.vue';
import MkKeyValue from '@/components/MkKeyValue.vue';
import MkButton from '@/components/MkButton.vue';
import * as os from '@/os.js';
import { defaultStore } from '@/store.js';
import { signout, $i } from '@/account.js';
import { i18n } from '@/i18n.js';
import { definePageMetadata } from '@/scripts/page-metadata.js';
import { unisonReload } from '@/scripts/unison-reload.js';
import FormSection from '@/components/form/section.vue';

const reportError = computed(defaultStore.makeGetterSetter('reportError'));
const enableCondensedLineForAcct = computed(defaultStore.makeGetterSetter('enableCondensedLineForAcct'));
const devMode = computed(defaultStore.makeGetterSetter('devMode'));
const defaultWithReplies = computed(defaultStore.makeGetterSetter('defaultWithReplies'));

function onChangeInjectFeaturedNote(v) {
	os.api('i/update', {
		injectFeaturedNote: v,
	}).then((i) => {
		$i!.injectFeaturedNote = i.injectFeaturedNote;
	});
}

async function deleteAccount() {
	{
		const { canceled } = await os.confirm({
			type: 'warning',
			text: i18n.ts.deleteAccountConfirm,
		});
		if (canceled) return;
	}

	const auth = await os.authenticateDialog();
	if (auth.canceled) return;

	await os.apiWithDialog('i/delete-account', {
		password: auth.result.password,
		token: auth.result.token,
	});

	await os.alert({
		title: i18n.ts._accountDelete.started,
	});

	await signout();
}

async function reloadAsk() {
	const { canceled } = await os.confirm({
		type: 'info',
		text: i18n.ts.reloadToApplySetting,
	});
	if (canceled) return;

	unisonReload();
}

async function updateRepliesAll(withReplies: boolean) {
	const { canceled } = os.confirm({
		type: 'warning',
		text: withReplies ? i18n.ts.confirmShowRepliesAll : i18n.ts.confirmHideRepliesAll,
	});
	if (canceled) return;
	await os.api('following/update-all', { withReplies });
}

const exportData = () => {
	os.api('i/export-data', {}).then(() => {
		os.alert({
			type: 'info',
			text: i18n.ts.exportRequested,
		});
	}).catch((ev) => {
		os.alert({
			type: 'error',
			text: ev.message,
		});
	});
};

watch([
	enableCondensedLineForAcct,
], async () => {
	await reloadAsk();
});

const headerActions = $computed(() => []);

const headerTabs = $computed(() => []);

definePageMetadata({
	title: i18n.ts.other,
	icon: 'ph-dots-three ph-bold ph-lg',
});
</script>
