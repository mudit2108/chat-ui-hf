<script lang="ts">
	import IconRead from './../icons/IconRead.svelte';
	import type { Message } from "$lib/types/Message";
	import { createEventDispatcher } from "svelte";
	import CarbonSendAltFilled from "~icons/carbon/send-alt-filled";
	import CarbonExport from "~icons/carbon/export";
	import CarbonStopFilledAlt from "~icons/carbon/stop-filled-alt";
	import EosIconsLoading from "~icons/eos-icons/loading";
import Modal from '../Modal.svelte';
	import ChatMessages from "./ChatMessages.svelte";
	import ChatInput from "./ChatInput.svelte";
	import StopGeneratingBtn from "../StopGeneratingBtn.svelte";
	import type { Model } from "$lib/types/Model";
	import type { LayoutData } from "../../../routes/$types";
	import WebSearchToggle from "../WebSearchToggle.svelte";
	import LoginModal from "../LoginModal.svelte";
	import type { WebSearchUpdate } from "$lib/types/MessageUpdate";
	import { page } from "$app/stores";
	import DisclaimerModal from "../DisclaimerModal.svelte";
	import RetryBtn from "../RetryBtn.svelte";

	export let messages: Message[] = [];
	export let loading = false;
	export let pending = false;
	export let shared = false;
	export let currentModel: Model;
	export let models: Model[];
	export let settings: LayoutData["settings"];
	export let webSearchMessages: WebSearchUpdate[] = [];
	export let preprompt: string | undefined = undefined;

	$: isReadOnly = !models.some((model) => model.id === currentModel.id);

	let loginModalOpen = false;
	let message: string;
	let readFile=false;
	let path='';

	const dispatch = createEventDispatcher<{
		message: string;
		share: void;
		stop: void;
		retry: { id: Message["id"]; content: string };
	}>();

	const handleSubmit = () => {
		if (loading) return;
		dispatch("message", message);
		message = "";
	};

	$: lastIsError = messages[messages.length - 1]?.from === "user" && !loading;


const handleReadFile=()=>{
    readFile=true;
	console.log(readFile)
	console.log('clicked')
}

const getpath=(e:any)=>{
path= e.target.value
}






</script>

<div class="relative min-h-0 min-w-0">
	{#if !settings.ethicsModalAcceptedAt}
		<DisclaimerModal {settings} />
	{:else if loginModalOpen}
		<LoginModal
			{settings}
			on:close={() => {
				loginModalOpen = false;
			}}
		/>
	{/if}
	<ChatMessages
		{loading}
		{pending}
		{settings}
		{currentModel}
		{models}
		{messages}
		readOnly={isReadOnly}
		isAuthor={!shared}
		{webSearchMessages}
		{preprompt}
		on:message={(ev) => {
			if ($page.data.loginRequired) {
				loginModalOpen = true;
			} else {
				dispatch("message", ev.detail);
			}
		}}
		on:vote
		on:retry={(ev) => {
			if (!loading) dispatch("retry", ev.detail);
		}}
	/>
	<div
		class="dark:via-gray-80 pointer-events-none absolute inset-x-0 bottom-0 z-0 mx-auto flex w-full max-w-3xl flex-col items-center justify-center bg-gradient-to-t from-white via-white/80 to-white/0 px-3.5 py-4 dark:border-gray-800 dark:from-gray-900 dark:to-gray-900/0 max-md:border-t max-md:bg-white max-md:dark:bg-gray-900 sm:px-5 md:py-8 xl:max-w-4xl [&>*]:pointer-events-auto"
	>
		<div class="flex w-full pb-3">
			{#if settings?.searchEnabled}
				<WebSearchToggle />
			{/if}
			{#if loading}
				<StopGeneratingBtn classNames="ml-auto" on:click={() => dispatch("stop")} />
			{/if}
			{#if lastIsError}
				<RetryBtn
					classNames="ml-auto"
					on:click={() =>
						dispatch("retry", {
							id: messages[messages.length - 1].id,
							content: messages[messages.length - 1].content,
						})}
				/>
			{/if}
		</div>
		<form
			on:submit|preventDefault={handleSubmit}
			class="relative flex w-full max-w-4xl flex-1 items-center rounded-xl border bg-gray-100 focus-within:border-gray-300 dark:border-gray-600 dark:bg-gray-700 dark:focus-within:border-gray-500
			{isReadOnly ? 'opacity-30' : ''}"
		>
			<div class="flex w-full flex-1 border-none bg-transparent">
				{#if lastIsError}
					<ChatInput value="Sorry, something went wrong. Please try again." disabled={true} />
				{:else}
					<ChatInput
						placeholder="Ask anything"
						bind:value={message}
						on:submit={handleSubmit}
						on:keypress={(ev) => {
							if ($page.data.loginRequired) {
								ev.preventDefault();
								loginModalOpen = true;
							}
						}}
						maxRows={4}
						disabled={isReadOnly || lastIsError}
					/>
				{/if}

					<button
				on:click={handleReadFile}
						class="btn  my-1 h-[2.4rem] self-end rounded-lg bg-transparent p-1 px-[0.7rem] text-gray-400 disabled:opacity-60 enabled:hover:text-gray-700 dark:disabled:opacity-40 enabled:dark:hover:text-gray-100"
						
					>
						<IconRead />
				</button>

				{#if loading}
					<button
						class="btn mx-1 my-1 inline-block h-[2.4rem] self-end rounded-lg bg-transparent p-1 px-[0.7rem] text-gray-400 disabled:opacity-60 enabled:hover:text-gray-700 dark:disabled:opacity-40 enabled:dark:hover:text-gray-100 md:hidden"
						on:click={() => dispatch("stop")}
					>
						<CarbonStopFilledAlt />
					</button>
					<div
						class="mx-1 my-1 hidden h-[2.4rem] items-center p-1 px-[0.7rem] text-gray-400 disabled:opacity-60 enabled:hover:text-gray-700 dark:disabled:opacity-40 enabled:dark:hover:text-gray-100 md:flex"
					>
						<EosIconsLoading />
					</div>
				{:else}

					<button
						class="btn mx-1 my-1 h-[2.4rem] self-end rounded-lg bg-transparent p-1 px-[0.7rem] text-gray-400 disabled:opacity-60 enabled:hover:text-gray-700 dark:disabled:opacity-40 enabled:dark:hover:text-gray-100"
						disabled={!message || isReadOnly}
						type="submit"
					>
						<CarbonSendAltFilled />
					</button>
				{/if}

			
			</div>


		</form>

		
		<div class="mt-2 flex justify-between self-stretch px-1 text-xs text-gray-400/90 max-sm:gap-2">
			<p>
				Model: <a
					href={currentModel.modelUrl || "https://huggingface.co/" + currentModel.name}
					target="_blank"
					rel="noreferrer"
					class="hover:underline">{currentModel.displayName}</a
				> <span class="max-sm:hidden">·</span><br class="sm:hidden" /> Generated content may be inaccurate
				or false.
			</p>
			{#if messages.length}
				<button
					class="flex flex-none items-center hover:text-gray-400 hover:underline max-sm:rounded-lg max-sm:bg-gray-50 max-sm:px-2.5 dark:max-sm:bg-gray-800"
					type="button"
					on:click={() => dispatch("share")}
				>
					<CarbonExport class="text-[.6rem] sm:mr-1.5 sm:text-primary-500" />
					<div class="max-sm:hidden">Share this conversation</div>
				</button>
			{/if}
		</div>
	</div>

	{#if readFile}
<Modal>
	<div class="w-[385px] p-5">
		<div class="flex  justify-end px-2 pb-2">
				<button on:click={()=>readFile=false} class=' text-black'> <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24"><path fill="currentColor" d="m8.4 17l3.6-3.6l3.6 3.6l1.4-1.4l-3.6-3.6L17 8.4L15.6 7L12 10.6L8.4 7L7 8.4l3.6 3.6L7 15.6L8.4 17Zm3.6 5q-2.075 0-3.9-.788t-3.175-2.137q-1.35-1.35-2.137-3.175T2 12q0-2.075.788-3.9t2.137-3.175q1.35-1.35 3.175-2.137T12 2q2.075 0 3.9.788t3.175 2.137q1.35 1.35 2.138 3.175T22 12q0 2.075-.788 3.9t-2.137 3.175q-1.35 1.35-3.175 2.138T12 22Z"/></svg></button>
		</div>

		<div class="w-full pl-2 flex gap-3 items-center">

	<span class="min-w-[100px]">File Path</span>
			<input  bind:value={path} on:change={getpath}  class=" font-[300] border border-gray-500 w-[250px] px-2 rounded outline-none py-1 text-sm" type='text' placeholder="Enter your file name here..."/>
			
		</div>

		
<div class="flex justify-center mt-5">
<button 

class="btn mx-auto right-12 rounded-lg border border-gray-200 px-5 py-[6px] text-sm shadow-sm transition-all hover:border-gray-300 active:shadow-inner dark:border-gray-600 dark:hover:border-gray-400	
"> Submit</button>


</div>

	</div>

	
</Modal>
{/if}
	
</div>
