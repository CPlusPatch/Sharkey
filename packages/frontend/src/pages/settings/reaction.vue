<!--
SPDX-FileCopyrightText: syuilo and other misskey contributors
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div class="_gaps_m">
	<FromSlot>
		<template #label>{{ i18n.ts.reactionSettingDescription }}</template>
		<div v-panel style="border-radius: var(--radius-sm);">
			<Sortable v-model="reactions" :class="$style.reactions" :itemKey="item => item" :animation="150" :delay="100" :delayOnTouchOnly="true">
				<template #item="{element}">
					<button class="_button" :class="$style.reactionsItem" @click="remove(element, $event)">
						<MkCustomEmoji v-if="element[0] === ':'" :name="element" :normal="true"/>
						<MkEmoji v-else :emoji="element" :normal="true"/>
					</button>
				</template>
				<template #footer>
					<button class="_button" :class="$style.reactionsAdd" @click="chooseEmoji"><i class="ph-plus ph-bold ph-lg"></i></button>
				</template>
			</Sortable>
		</div>
		<template #caption>{{ i18n.ts.reactionSettingDescription2 }} <button class="_textButton" @click="preview">{{ i18n.ts.preview }}</button></template>
	</FromSlot>

	<FromSlot>
		<template #label>Default like emoji</template>
		<MkCustomEmoji v-if="like.startsWith(':')" style="max-height: 3em; font-size: 1.1em;" :useOriginalSize="false" :class="$style.reaction" :name="like" :normal="true" :noStyle="true"/>
		<MkEmoji v-else :emoji="like" style="max-height: 3em; font-size: 1.1em;" :normal="true" :noStyle="true"/>
		<div class="_buttons" style="padding-top: 8px;">
			<MkButton rounded :small="true" inline @click="chooseNewLike"><i class="ph-smiley ph-bold ph-lg"></i> Change</MkButton>
			<MkButton rounded :small="true" inline @click="resetLike"><i class="ph-arrow-clockwise ph-bold ph-lg"></i> Reset</MkButton>
		</div>
	</FromSlot>

	<MkRadios v-model="reactionPickerSize">
		<template #label>{{ i18n.ts.size }}</template>
		<option :value="1">{{ i18n.ts.small }}</option>
		<option :value="2">{{ i18n.ts.medium }}</option>
		<option :value="3">{{ i18n.ts.large }}</option>
	</MkRadios>
	<MkRadios v-model="reactionPickerWidth">
		<template #label>{{ i18n.ts.numberOfColumn }}</template>
		<option :value="1">5</option>
		<option :value="2">6</option>
		<option :value="3">7</option>
		<option :value="4">8</option>
		<option :value="5">9</option>
	</MkRadios>
	<MkRadios v-model="reactionPickerHeight">
		<template #label>{{ i18n.ts.height }}</template>
		<option :value="1">{{ i18n.ts.small }}</option>
		<option :value="2">{{ i18n.ts.medium }}</option>
		<option :value="3">{{ i18n.ts.large }}</option>
		<option :value="4">{{ i18n.ts.large }}+</option>
	</MkRadios>

	<MkSwitch v-model="reactionPickerUseDrawerForMobile">
		{{ i18n.ts.useDrawerReactionPickerForMobile }}
		<template #caption>{{ i18n.ts.needReloadToApply }}</template>
	</MkSwitch>

	<FormSection>
		<div class="_buttons">
			<MkButton inline @click="preview"><i class="ph-eye ph-bold ph-lg"></i> {{ i18n.ts.preview }}</MkButton>
			<MkButton inline danger @click="setDefault"><i class="ph-arrow-clockwise ph-bold ph-lg"></i> {{ i18n.ts.default }}</MkButton>
		</div>
	</FormSection>
</div>
</template>

<script lang="ts" setup>
import { defineAsyncComponent, watch } from 'vue';
import Sortable from 'vuedraggable';
import MkRadios from '@/components/MkRadios.vue';
import FromSlot from '@/components/form/slot.vue';
import MkButton from '@/components/MkButton.vue';
import FormSection from '@/components/form/section.vue';
import MkSwitch from '@/components/MkSwitch.vue';
import * as os from '@/os.js';
import { defaultStore } from '@/store.js';
import { i18n } from '@/i18n.js';
import { definePageMetadata } from '@/scripts/page-metadata.js';
import { deepClone } from '@/scripts/clone.js';

let reactions = $ref(deepClone(defaultStore.state.reactions));
const like = $computed(defaultStore.makeGetterSetter('like'));

const reactionPickerSize = $computed(defaultStore.makeGetterSetter('reactionPickerSize'));
const reactionPickerWidth = $computed(defaultStore.makeGetterSetter('reactionPickerWidth'));
const reactionPickerHeight = $computed(defaultStore.makeGetterSetter('reactionPickerHeight'));
const reactionPickerUseDrawerForMobile = $computed(defaultStore.makeGetterSetter('reactionPickerUseDrawerForMobile'));

function save() {
	defaultStore.set('reactions', reactions);
}

function remove(reaction, ev: MouseEvent) {
	os.popupMenu([{
		text: i18n.ts.remove,
		action: () => {
			reactions = reactions.filter(x => x !== reaction);
		},
	}], ev.currentTarget ?? ev.target);
}

function preview(ev: MouseEvent) {
	os.popup(defineAsyncComponent(() => import('@/components/MkEmojiPickerDialog.vue')), {
		asReactionPicker: true,
		src: ev.currentTarget ?? ev.target,
	}, {}, 'closed');
}

async function setDefault() {
	const { canceled } = await os.confirm({
		type: 'warning',
		text: i18n.ts.resetAreYouSure,
	});
	if (canceled) return;

	reactions = deepClone(defaultStore.def.reactions.default);
}

function chooseEmoji(ev: MouseEvent) {
	os.pickEmoji(ev.currentTarget ?? ev.target, {
		showPinned: false,
	}).then(emoji => {
		if (!reactions.includes(emoji)) {
			reactions.push(emoji);
		}
	});
}

function chooseNewLike(ev: MouseEvent) {
	os.pickEmoji(ev.currentTarget ?? ev.target, {
		showPinned: false,
	}).then(emoji => {
		defaultStore.set('like', emoji as string);
	});
}

function resetLike() {
	defaultStore.set('like', '❤️');
}

watch($$(reactions), () => {
	save();
}, {
	deep: true,
});

const headerActions = $computed(() => []);

const headerTabs = $computed(() => []);

definePageMetadata({
	title: i18n.ts.reaction,
	icon: 'ph-smiley ph-bold ph-lg',
	action: {
		icon: 'ph-eye ph-bold ph-lg',
		handler: preview,
	},
});
</script>

<style lang="scss" module>
.reactions {
	padding: 12px;
	font-size: 1.1em;
}

.reactionsItem {
	display: inline-block;
	padding: 8px;
	cursor: move;
}

.reactionsAdd {
	display: inline-block;
	padding: 8px;
}
</style>
