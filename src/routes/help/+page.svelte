<script lang="ts">
	import * as Card from '$lib/components/ui/card/index.js';
	import { ndk } from '@/ndk';
	import { NDKKind } from '@nostr-dev-kit/ndk';
	import { onDestroy } from 'svelte';
	import { derived } from 'svelte/store';
	import Heading from '../../components/Heading.svelte';
	import HelpThreadNoteTree from '../../components/HelpThreadNoteTree.svelte';
	import Button from '@/components/ui/button/button.svelte';
	import { Label } from '$lib/components/ui/label/index.js';
	import { Textarea } from '$lib/components/ui/textarea/index.js';
	import { devmode, currentUser } from '@/stores/session';
	import { buildNoteTree, prepareQuestionNoteEvent } from '@/event_helpers/help_thread';
	import type NDKSvelte from '@nostr-dev-kit/ndk-svelte';
	import { HELP_THREAD_ROOT_EVENT_ID } from '@/consts';
	import Login from '../../components/Login.svelte';

	let notes = $ndk.storeSubscribe({
		kinds: [1 as NDKKind],
		'#e': [HELP_THREAD_ROOT_EVENT_ID]
	});

	let treeNotes = derived(notes, ($_notes) => {
		return buildNoteTree($_notes);
	});

	let content: string;

	function publish(ndk: NDKSvelte) {
		if (!ndk.signer) {
			throw new Error('no ndk signer found');
		}
		let author = $currentUser;
		if (!author) {
			throw new Error('no current user');
		}
		const e = prepareQuestionNoteEvent({
			ndk,
			content
		});
		e.publish().then((x) => {
			console.log(x);
		});
	}

	onDestroy(() => {
		notes.unsubscribe();
	});
</script>

<div class="my-4 flex flex-col gap-2">
	{#if $devmode}
		<div>
			<Button
				on:click={() => {
					console.log('notes', $notes);
					console.log('treeNotes', $treeNotes);
				}}
				variant="outline"
			>
				Print to Console
			</Button>
		</div>
	{/if}
	<Heading title="Help" />
	<div>Ask your questions here!</div>
	<form class="relative overflow-hidden">
		<Label for="message" class="sr-only">Question</Label>
		<Textarea
			id="message"
			placeholder="Type your question here..."
			class="w-full resize-none shadow-none"
			bind:value={content}
		/>
		<div class="flex items-center justify-end pt-2">
			{#if $currentUser}
				<Button
					type="submit"
					size="sm"
					on:click={() => {
						publish($ndk);
					}}
				>
					Publish
				</Button>
			{:else}
				<Login />
			{/if}
		</div>
	</form>
	<Card.Root>
		<HelpThreadNoteTree notes={$treeNotes} isRoot />
	</Card.Root>
</div>
