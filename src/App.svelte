<script lang="ts">
  import { saveAs } from "file-saver"
  import { copyImageToClipboard } from "copy-image-clipboard"

  import * as htmlToImage from "html-to-image"
  import Kofi from "./lib/Kofi.svelte"
  import Social from "./lib/Social.svelte"
  import dragElement from "./lib/dragElement"
  import { afterUpdate, onMount, tick } from "svelte"
  import pasteImage from "paste-image"
  import GraphemeSplitter from "grapheme-splitter"
  import CircleType from "circletype"
  import Moveable from "svelte-moveable"
  import { Color, ColorInput } from "color-picker-svelte"
  import { SvgDrawing } from "svg-drawing"
  import bgImage from "./assets/the-wall.jpg"

  let textColor = new Color("#eeeeee")
  let bgColor = new Color("#000000")

  let drawArea, drawing

  let title = "writing"
  let url = "https://the-writing-on-the-wall.narze.live/"

  const curve1 = "M 30, 10 C 130,-25 160,-30 270,-25"
  const curve2 = "M 30, 10 C 130,-30 160,-35 270,-25"
  const curve3 = "M 30, 10 C 130,-35 160,-40 270,-25"
  const underline = "M0,-5 Q60,-20 120,-20 L122,0 Q60,0 3,14 L0,-5"

  const fonts = [
    "Layiji TarMineTine",
    // "Layiji LenRoonRang",
    "FC Catalyst",
    "SOV_RongNang",
    "FCGimmick-Bold",
  ]

  let selectedFont = fonts[0]
  let target

  let frame = {
    matrix: [1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1],
    translate: [0, 0],
    rotate: 0,
    transformOrigin: "50% 50%",
  }

  let avatar,
    fileinput,
    node,
    textNode,
    imageWidth,
    avatarElm,
    svgNode,
    container

  let descriptionSize = 180
  let spacingMicro = 150
  $: spacing = spacingMicro / 10
  let isCopy = false
  let saving = false
  let imageZoom = 90
  // let circleType
  let el: HTMLElement
  const splitter = new GraphemeSplitter()

  const onAvatarLoad = () => {
    imageWidth = avatarElm.width
    // console.dir(avatarElm)
  }

  $: if (drawArea) {
    drawing = new SvgDrawing(drawArea, {penWidth: 3})
  }

  function undoDrawing() {
    drawing.undo()
  }

  function clearDrawing() {
    drawing.clear()
  }

  function onFileSelected(e) {
    let image = e.target.files[0]
    let reader = new FileReader()
    reader.readAsDataURL(image)
    reader.onload = (e) => {
      avatar = e.target.result
      setTimeout(() => onAvatarLoad(), 500)
    }
  }

  function downloadImage() {
    saving = true
    htmlToImage
      .toJpeg(node)
      .then(function (blob) {
        saveAs(blob, `${title}.jpg`)
        saving = false
      })
      .catch(function (error) {
        console.error("oops, something went wrong!", error)
      })
  }

  async function copyImage() {
    saving = true
    htmlToImage
      .toPng(node)
      .then(function (dataUrl) {
        let img = new Image()
        img.src = dataUrl
        copyImageToClipboard(img.src)
        saving = false
      })
      .catch(function (error) {
        console.error("oops, something went wrong!", error)
      })
    isCopy = !isCopy
    setTimeout(() => {
      isCopy = !isCopy
    }, 5000)
  }
</script>

<!-- <svelte:head>
  <link
    href="https://fonts.googleapis.com/css2?family=Prompt&display=swap"
    rel="stylesheet"
  />
</svelte:head> -->

<!-- <Kofi name="narze" label="Support Me" /> -->
<!-- <Social {url} {title} description={""} /> -->

<main class="p-12 min-h-screen grid place-content-center gap-4">
  <div class="flex flex-col">
    <h1
      class="text-6xl mb-4 font-bold text-transparent text-center uppercase bg-clip-text bg-gradient-to-br text-[#eeeeee]"
    >
      The Writing On The Wall <span class="text-xs normal-case">Beta</span>
    </h1>
  </div>

  <div
    bind:this={node}
    class="flex items-center justify-center relative w-[960px] h-[640px] overflow-hidden mx-auto"
    style={`font-family: "${selectedFont}"; background-color: ${bgColor.toHex()}`}
  >
    <img src={bgImage} class="absolute inset-0 w-full h-full" />
    <div bind:this={drawArea} class="absolute w-full h-full">hi</div>
  </div>

  <div class="flex flex-row gap-2">
    <button
      class="text-black text-center text-xl border-2 border-slate-400 rounded px-2 py-4 basis-full bg-gradient-to-r from-[#eeeeee] to-[#eeeeee] hover:border-[#6215f1]"
      on:click={undoDrawing}
    >
      Undo
    </button>
    <button
      class="text-black text-center text-xl border-2 border-slate-400 rounded px-2 py-4 basis-full bg-gradient-to-r from-[#eeeeee] to-[#eeeeee] hover:border-[#6215f1]"
      on:click={clearDrawing}
    >
      Clear
    </button>
    <input
      style="display:none"
      type="file"
      accept=".jpg, .jpeg, .png"
      on:change={(e) => onFileSelected(e)}
      bind:this={fileinput}
    />
    <button
      class="text-black text-center text-xl border-2 border-slate-400 rounded px-2 py-4 basis-full bg-gradient-to-r from-[#eeeeee] to-[#eeeeee] hover:border-[#6215f1]"
      on:click={() => {
        downloadImage()
      }}
    >
      ดาวน์โหลด
    </button>
    <button
      class="text-black text-center text-xl border-2 border-slate-400 rounded px-2 py-4 basis-full bg-gradient-to-r from-[#eeeeee] to-[#eeeeee] hover:border-[#6215f1]"
      on:click={copyImage}
    >
      {isCopy ? "Copied" : "คัดลอก"}
    </button>
  </div>
</main>

<!-- Bottom links -->
<div class="fixed inset-x-0 bottom-16 sm:bottom-4 text-center">
  <a
    href="https://github.com/narze/the-writing-on-the-wall"
    target="_blank"
    class="text-black text-sm bg-gradient-to-r from-[#eeeeee] to-[#eeeeee] px-2 py-1 rounded-md mx-1"
    >Github</a
  >
  <a
    href="https://narze.live"
    target="_blank"
    class="text-black text-sm bg-gradient-to-r from-[#eeeeee] to-[#eeeeee] px-2 py-1 rounded-md mx-1"
    >@narze</a
  >
</div>

<style lang="postcss">
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
    @apply bg-gray-800;
  }

  .bg-gradient {
    background: rgb(0, 0, 0);
    background: linear-gradient(
      0deg,
      rgba(0, 0, 0, 1) 70%,
      rgba(0, 0, 0, 0) 100%
    );
  }

  /* main {
    font-family: "Prompt", sans-serif;
  } */

  input[type="range"] {
    width: 100%;
    margin: 15px 0;
    background-color: transparent;
    user-select: none;
    -webkit-appearance: none;
    -webkit-tap-highlight-color: transparent;
  }

  input[type="range"]::-webkit-slider-runnable-track {
    background-image: linear-gradient(to right, #9084a7, #6671af);
    border-radius: 8px;
    width: 100%;
    height: 8px;
    cursor: pointer;
  }

  input[type="range"]::-webkit-slider-thumb {
    margin-top: -15px;
    width: 35px;
    height: 35px;
    background: #eeeeee;
    box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.3);
    border-radius: 30px;
    cursor: pointer;
    -webkit-appearance: none;
    user-select: none;
  }

  /*TODO: Use one of the selectors from https://stackoverflow.com/a/20541859/7077589 and figure out
how to remove the virtical space around the range input in IE*/
  @supports (-ms-ime-align: auto) {
    /* Pre-Chromium Edge only styles, selector taken from hhttps://stackoverflow.com/a/32202953/7077589 */
    input[type="range"] {
      margin: 0;
      /*Edge starts the margin from the thumb, not the track as other browsers do*/
    }
  }
</style>
