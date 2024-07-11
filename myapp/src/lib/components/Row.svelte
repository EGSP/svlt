
<script lang="ts">
	export let gap: number = 0;
</script>

<div style:gap={gap ?gap+"px": null}><slot/></div>

<style>
	div {
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: start;
	}
</style>
