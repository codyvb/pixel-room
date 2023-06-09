<script lang="ts">
  import { createEventDispatcher, onMount } from "svelte";
  import { useMyPresence } from "../lib-liveblocks";
  import { hexToRgb } from "$lib/utils/hexToRgb";
  import type { Brush } from "../types";

  // Allows changing color value outside of component
  let colorValue: string = "";
  export let updateColor = (hex: string) => (colorValue = hex);

  export let swatch: string[] = [];

  const dispatch = createEventDispatcher();
  const myPresence = useMyPresence();

  let colorPicker: { getFormattedValue; swatches };

  // Default brush
  let brush: Brush = {
    opacity: 100,
    hue: 0,
    saturation: 0,
    color: "#000",
    lightness: 0,
    rgb: { r: 255, g: 255, b: 255 },
  };

  // Change brush event on any brush changes
  $: dispatch("brushChange", brush);
  onMount(async () => {
    dispatch("brushChange", brush);
    await import(
      "@shoelace-style/shoelace/dist/components/color-picker/color-picker.js"
    );
    colorValue = '#28c2ffff';
    applyCustomStyles(colorPicker);
  });

  $: if (colorPicker) {
    colorPicker.swatches = swatch;
  }

  // TODO: we may need this code later, potentially 
  // $: if (colorPicker && colorValue) {
  //   // @ts-ignore
  //   colorPicker.value = colorValue;
  //   colorChange({
  //     target: {
  //       value: colorValue,
  //       alpha: brush.opacity,
  //       hue: brush.hue,
  //       saturation: brush.saturation,
  //       lightness: brush.lightness
  //     }
  //   });
  // }

  // Workaround for custom elements
  function applyCustomStyles(host) {
    const style = document.createElement("style");
    style.innerHTML = `
      .color-picker__controls, .color-picker__user-input, .color-picker__swatches { padding-left: 0 !important; padding-right: 0 !important; }
      .color-picker__grid { border-radius: 4px !important; box-shadow: inset 0 0 0 1px rgba(0, 0, 0, 0.2); }
      div.color-picker__swatches { border-top: 0; padding-top: 2px; margin-left: -2px; margin-right: -2px; }
    `;
    host.shadowRoot.appendChild(style);
    host.swatches = swatch;
  }

  // When color changes, update presence
  function colorChange({ target }) {
    if (!colorPicker) {
      return;
    }

    let col = target.value;
    if (col[0] !== "#") {
      col = colorPicker.getFormattedValue("hex");
    }

    brush = {
      color: col,
      opacity: parseInt(target.alpha),
      hue: parseInt(target.hue),
      saturation: parseInt(target.saturation),
      lightness: parseInt(target.lightness),
      rgb: hexToRgb(col.slice(0, 7)),
    };

    if ($myPresence.tool === "eraser") {
      myPresence.update({ tool: "brush" });
    }
  }
</script>

<!-- this is where we set colors hide here -->

<div class="p-5 pb-2 ">
  <div class="pb-3 text-sm font-semibold hidden text-gray-500">Colour</div>
  <div>
    <div class="hidden">
    <sl-color-picker
      bind:this={colorPicker}
      inline
      on:sl-change={colorChange}
      opacity
      value={colorValue}
    ></sl-color-picker>
  </div>
  </div>

  <div class="text-white">
    <br>
    <button class="w-12 h-12" style="background-color: #f5c361ff" on:click={() => {
      updateColor('#f5c361ff');
    }}></button>
    <button class="w-12 h-12" style="background-color: #28c2ffff" on:click={() => {
      updateColor('#28c2ffff');
    }}></button>
    <button class="w-12 h-12" style="background-color: #00c600ff" on:click={() => {
      updateColor('#00c600ff');
    }}></button>
  </div>
</div>
