<script lang="ts">
	import { onMount } from 'svelte';
	import { SimpleDropzone } from 'simple-dropzone';
	import Preview from './Preview.svelte';

	export let threeRevision: string;

	let dropzoneEl: HTMLDivElement;
	let inputEl: HTMLInputElement;
	let textureData: ArrayBuffer;

	onMount(() => {
		const dropzone = new SimpleDropzone(dropzoneEl, inputEl);
		dropzone.on('drop', ({files}) => load(files));
	});

	async function load (fileMap: Map<string, File>) {
		let file: File = Array.from(fileMap)
			.find(([_, file]) => file.name.match(/\.ktx2$/))[1];

		if (!file) throw new Error('No .gltf or .glb asset found.');

		textureData = await file.arrayBuffer();
	}
</script>

<main bind:this="{dropzoneEl}">
	<h1>KTX 2.0 Viewer</h1>
	<p>
		KTX is a container format for GPU textures. GPU textures are used in
		WebGL, Vulkan, Metal, and other graphics APIs for their advantages in
		GPU memory cost and upload time, compared to traditional image formats.
		This viewer supports KTX 2.0 files containing Basis Universal texture
		codecs. 1D textures, cubemaps, and 3D array textures not yet supported.
		Made with <a href="http://threejs.org/" target="_blank"><i>three.js {threeRevision}</i></a>.
	</p>
	<p><small></small></p>

	{#if textureData}
		<Preview textureData={textureData}></Preview>
	{:else}
		<div class="dropzone-wrap">
			<div id="dropzone">
				<input type="file" id="asset" bind:this="{inputEl}">
				<label for="asset"><small><i>Select or drop a .ktx2 file</i></small></label>
			</div>
		</div>
	{/if}

	<img alt="KTX" src="ktx_logo.png">
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: rgba(48, 100, 150, 1);
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	p {
		max-width: 600px;
		margin: 4em auto;
		text-align: left;
		text-indent: 1em;
		line-height: 1.5em;
	}

	img {
		margin-top: 4em;
		width: 100px;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>