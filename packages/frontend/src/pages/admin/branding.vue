<!--
SPDX-FileCopyrightText: syuilo and other misskey contributors
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div>
	<MkStickyContainer>
		<template #header><XHeader :tabs="headerTabs"/></template>
		<MkSpacer :contentMax="700" :marginMin="16" :marginMax="32">
			<FormSuspense :p="init">
				<div class="_gaps_m">
					<MkInput v-model="iconUrl" type="url">
						<template #prefix><i class="ph-link ph-bold ph-lg"></i></template>
						<template #label>{{ i18n.ts._serverSettings.iconUrl }}</template>
					</MkInput>

					<MkInput v-model="app192IconUrl" type="url">
						<template #prefix><i class="ph-link ph-bold ph-lg"></i></template>
						<template #label>{{ i18n.ts._serverSettings.iconUrl }} (App/192px)</template>
						<template #caption>
							<div>{{ i18n.t('_serverSettings.appIconDescription', { host: instance.name ?? host }) }}</div>
							<div>({{ i18n.ts._serverSettings.appIconUsageExample }})</div>
							<div>{{ i18n.ts._serverSettings.appIconStyleRecommendation }}</div>
							<div><strong>{{ i18n.t('_serverSettings.appIconResolutionMustBe', { resolution: '192x192px' }) }}</strong></div>
						</template>
					</MkInput>

					<MkInput v-model="app512IconUrl" type="url">
						<template #prefix><i class="ph-link ph-bold ph-lg"></i></template>
						<template #label>{{ i18n.ts._serverSettings.iconUrl }} (App/512px)</template>
						<template #caption>
							<div>{{ i18n.t('_serverSettings.appIconDescription', { host: instance.name ?? host }) }}</div>
							<div>({{ i18n.ts._serverSettings.appIconUsageExample }})</div>
							<div>{{ i18n.ts._serverSettings.appIconStyleRecommendation }}</div>
							<div><strong>{{ i18n.t('_serverSettings.appIconResolutionMustBe', { resolution: '512x512px' }) }}</strong></div>
						</template>
					</MkInput>

					<MkInput v-model="bannerUrl" type="url">
						<template #prefix><i class="ph-link ph-bold ph-lg"></i></template>
						<template #label>{{ i18n.ts.bannerUrl }}</template>
					</MkInput>

					<MkInput v-model="backgroundImageUrl" type="url">
						<template #prefix><i class="ph-link ph-bold ph-lg"></i></template>
						<template #label>{{ i18n.ts.backgroundImageUrl }}</template>
					</MkInput>

					<FromSlot>
						<template #label>Default like emoji</template>
						<MkCustomEmoji v-if="defaultLike.startsWith(':')" style="max-height: 3em; font-size: 1.1em;" :useOriginalSize="false" :class="$style.reaction" :name="defaultLike" :normal="true" :noStyle="true"/>
						<MkEmoji v-else :emoji="defaultLike" style="max-height: 3em; font-size: 1.1em;" :normal="true" :noStyle="true"/>
						<MkButton rounded :small="true" @click="chooseNewLike"><i class="ph-smiley ph-bold ph-lg"></i> Change</MkButton>
					</FromSlot>

					<MkInput v-model="notFoundImageUrl" type="url">
						<template #prefix><i class="ph-link ph-bold ph-lg"></i></template>
						<template #label>{{ i18n.ts.notFoundDescription }}</template>
					</MkInput>

					<MkInput v-model="infoImageUrl" type="url">
						<template #prefix><i class="ph-link ph-bold ph-lg"></i></template>
						<template #label>{{ i18n.ts.nothing }}</template>
					</MkInput>

					<MkInput v-model="serverErrorImageUrl" type="url">
						<template #prefix><i class="ph-link ph-bold ph-lg"></i></template>
						<template #label>{{ i18n.ts.somethingHappened }}</template>
					</MkInput>

					<MkColorInput v-model="themeColor">
						<template #label>{{ i18n.ts.themeColor }}</template>
					</MkColorInput>

					<MkTextarea v-model="defaultLightTheme">
						<template #label>{{ i18n.ts.instanceDefaultLightTheme }}</template>
						<template #caption>{{ i18n.ts.instanceDefaultThemeDescription }}</template>
					</MkTextarea>

					<MkTextarea v-model="defaultDarkTheme">
						<template #label>{{ i18n.ts.instanceDefaultDarkTheme }}</template>
						<template #caption>{{ i18n.ts.instanceDefaultThemeDescription }}</template>
					</MkTextarea>

					<MkTextarea v-model="manifestJsonOverride">
						<template #label>{{ i18n.ts._serverSettings.manifestJsonOverride }}</template>
					</MkTextarea>
				</div>
			</FormSuspense>
		</MkSpacer>
		<template #footer>
			<div :class="$style.footer">
				<MkSpacer :contentMax="700" :marginMin="16" :marginMax="16">
					<MkButton primary rounded @click="save"><i class="ph-check ph-bold ph-lg"></i> {{ i18n.ts.save }}</MkButton>
				</MkSpacer>
			</div>
		</template>
	</MkStickyContainer>
</div>
</template>

<script lang="ts" setup>
import { } from 'vue';
import JSON5 from 'json5';
import XHeader from './_header_.vue';
import MkSwitch from '@/components/MkSwitch.vue';
import MkInput from '@/components/MkInput.vue';
import MkTextarea from '@/components/MkTextarea.vue';
import FormSection from '@/components/form/section.vue';
import FormSplit from '@/components/form/split.vue';
import FromSlot from '@/components/form/slot.vue';
import FormSuspense from '@/components/form/suspense.vue';
import * as os from '@/os.js';
import { instance, fetchInstance } from '@/instance.js';
import { i18n } from '@/i18n.js';
import { definePageMetadata } from '@/scripts/page-metadata.js';
import MkButton from '@/components/MkButton.vue';
import MkColorInput from '@/components/MkColorInput.vue';
import { host } from '@/config.js';

let iconUrl: string | null = $ref(null);
let app192IconUrl: string | null = $ref(null);
let app512IconUrl: string | null = $ref(null);
let bannerUrl: string | null = $ref(null);
let backgroundImageUrl: string | null = $ref(null);
let themeColor: any = $ref(null);
let defaultLightTheme: any = $ref(null);
let defaultDarkTheme: any = $ref(null);
let defaultLike: any = $ref(null);
let serverErrorImageUrl: string | null = $ref(null);
let infoImageUrl: string | null = $ref(null);
let notFoundImageUrl: string | null = $ref(null);
let manifestJsonOverride: string = $ref('{}');

async function init() {
	const meta = await os.api('admin/meta');
	iconUrl = meta.iconUrl;
	app192IconUrl = meta.app192IconUrl;
	app512IconUrl = meta.app512IconUrl;
	bannerUrl = meta.bannerUrl;
	backgroundImageUrl = meta.backgroundImageUrl;
	themeColor = meta.themeColor;
	defaultLightTheme = meta.defaultLightTheme;
	defaultDarkTheme = meta.defaultDarkTheme;
	defaultLike = meta.defaultLike;
	serverErrorImageUrl = meta.serverErrorImageUrl;
	infoImageUrl = meta.infoImageUrl;
	notFoundImageUrl = meta.notFoundImageUrl;
	manifestJsonOverride = meta.manifestJsonOverride === '' ? '{}' : JSON.stringify(JSON.parse(meta.manifestJsonOverride), null, '\t');
}

function save() {
	os.apiWithDialog('admin/update-meta', {
		iconUrl,
		app192IconUrl,
		app512IconUrl,
		bannerUrl,
		backgroundImageUrl,
		themeColor: themeColor === '' ? null : themeColor,
		defaultLightTheme: defaultLightTheme === '' ? null : defaultLightTheme,
		defaultDarkTheme: defaultDarkTheme === '' ? null : defaultDarkTheme,
		infoImageUrl,
		notFoundImageUrl,
		serverErrorImageUrl,
		manifestJsonOverride: manifestJsonOverride === '' ? '{}' : JSON.stringify(JSON5.parse(manifestJsonOverride)),
	}).then(() => {
		fetchInstance();
	});
}

function chooseNewLike(ev: MouseEvent) {
	os.pickEmoji(ev.currentTarget ?? ev.target, {
		showPinned: false,
	}).then(emoji => {
		os.apiWithDialog('admin/update-meta', {
			defaultLike: emoji,
		}).then(() => {
			fetchInstance();
			defaultLike = emoji;
		});
	});
}

const headerTabs = $computed(() => []);

definePageMetadata({
	title: i18n.ts.branding,
	icon: 'ph-paint-roller ph-bold ph-lg',
});
</script>

<style lang="scss" module>
.footer {
	-webkit-backdrop-filter: var(--blur, blur(15px));
	backdrop-filter: var(--blur, blur(15px));
}
</style>
