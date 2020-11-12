<script lang="ts">
    import { onMount } from 'svelte';
    import { WebGLRenderer, PlaneBufferGeometry, Mesh, Scene, OrthographicCamera, MeshBasicMaterial, AmbientLight, BufferGeometry } from 'three';
    import { KTX2Loader } from 'three/examples/jsm/loaders/KTX2Loader.js';

    export let textureData: ArrayBuffer;

    let WIDTH: number;
    let HEIGHT: number;

    let previewEl: HTMLElement;
    let renderer: WebGLRenderer;
    let scene: Scene;
    let camera: OrthographicCamera;
    let material: MeshBasicMaterial;
    let loader: KTX2Loader;

    // Reactive binding: executes when 'loader' or 'textureData' change.
    $: {
        if (loader) decodeTexture(textureData);
    }

    onMount(async () => {
        WIDTH = window.innerWidth - 80;
        HEIGHT = 600;

        // Renderer.
        renderer = new WebGLRenderer()
        renderer.setSize(WIDTH, HEIGHT);
        renderer.setClearColor(0xF0F0F0);
        previewEl.appendChild(renderer.domElement);

        // Material.
        material = new MeshBasicMaterial({color: 0xFFFFFF, transparent: true});

        // Scene.
        scene = new Scene();
        scene.add(new Mesh(flipY(new PlaneBufferGeometry()), material));
        scene.add(new AmbientLight());

        // Camera.
        const xMag = ((WIDTH / HEIGHT) / 2) * 1.05;
        const yMag = 0.5 * 1.05;
        camera = new OrthographicCamera(-xMag, xMag, yMag, -yMag, 0.5, 10);
        camera.position.set(0, 0, 5);
        camera.lookAt(scene.position);
        scene.add(camera);

        // Texture.
        loader = new KTX2Loader().detectSupport(renderer);
    });

    function decodeTexture(textureData: ArrayBuffer) {
        loader.parse(textureData, (texture) => {
            material.map = texture;
            material.transparent = true;
            material.needsUpdate = true;
            texture.needsUpdate = true;

            console.log(texture);
            renderer.render(scene, camera);
        }, (e) => {
            console.error(e);
        });
    }

    /** Correct UVs to be compatible with `flipY=false` textures. */
    function flipY(geometry: BufferGeometry) {
        const uv = geometry.attributes.uv;
        for (let i = 0; i < uv.count; i++) {
            uv.setY(i, 1 - uv.getY(i));
        }
        return geometry;
    }
</script>

<div bind:this="{previewEl}" class="preview"></div>

<style>
	.preview {
        width: 100%;
        height: 600px;
	}
</style>
