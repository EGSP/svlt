<script lang="ts">
	import InlineButton from './InlineButton.svelte';
	import CarbonCopy from '$lib/icons/CarbonCopy.svelte';

	export let feedbackTimeout: number = 2000;
	export let valueToCopy: string;

	let iconDescription = 'Copy';

	function unsecuredCopyToClipboard(text: string) {
		const textArea = document.createElement('textarea');
		textArea.value = text;
		document.body.appendChild(textArea);
		textArea.focus();
		textArea.select();
		try {
			document.execCommand('copy');
		} catch (err) {
			console.error('Unable to copy to clipboard', err);
		}
		document.body.removeChild(textArea);
	}

	function handleClick() {
		
		if (window.isSecureContext && navigator.clipboard) {
			navigator.clipboard.writeText(valueToCopy);
		} else {
			unsecuredCopyToClipboard(valueToCopy);
		}

		let temporary_iconDescription = iconDescription.slice();
		iconDescription = 'Copied!';
		setTimeout(() => {
			iconDescription = temporary_iconDescription;
		}, feedbackTimeout);
	}
</script>

<InlineButton icon={CarbonCopy} {iconDescription} on:click={handleClick} />
