# customneko

a Svelte port of [Neko](https://en.wikipedia.org/wiki/Neko_(software)) for use with custom spritesheets and custom behavior

play with an example [here](https://spiritov.github.io/customneko/build/index) :)  

![customneko](https://github.com/user-attachments/assets/75cc9689-864f-4c69-b91e-87951548ba1e)

## Svelte usage
```html
<!-- +page.svelte -->
<script>
  import CustomNeko from '$lib/components/CustomNeko.svelte';

  // props..
  // image_path: url or path to a spritesheet image, see below for the expected layout
  // sprite_width, sprite_height: each spritesheet frame is expected to have the same width and height
  // optional..
  // update_ms: how fast sprite animations should cycle frames
  // sleep time: how many total frames there should be in-between sleep frames
  // move_speed: sprite move speed in pixels, every update_ms ms
</script>

<CustomNeko
	image_path={'image_path_here'}
	sprite_width={40}
	sprite_height={40}
/>
```

### expected spritesheet layout..

<img width="360" height="360" alt="image" src="https://github.com/user-attachments/assets/7c4b0a30-1b53-4ab8-bbac-2578da13b79c" />

> [!TIP]
> reference the component's animation map to see how you can customize the layout of the spritesheet. it's very easy to modify as long as each sprite has the same grid size!

## development

to run locally..

```sh
npm i
npm run dev
```

to build..

```sh
npm run build
```

thanks to the [PMD Sprite Repository](https://sprites.pmdcollab.org/) for easily accessible spritesheets
