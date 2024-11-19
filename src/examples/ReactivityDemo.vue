<template>
  <form>
    <div>
      <input type="checkbox" v-model="documentFlow" />
      <label>DocumentFlow</label>
    </div>
    <div>
      <label>Pan X</label>
      <input type="number" v-model="pan.x">
    </div>
    <div>
      <label>Pan Y</label>
      <input type="number" v-model="pan.y">
    </div>
    <div>
      <label>Zoom</label>
      <input type="number" v-model="zoom">
    </div>

    <div>
      <input type="checkbox" v-model="enablePan" />
      <label>enablePan</label>
    </div>

    <div>
      <input type="checkbox" v-model="enableZoom" />
      <label>enableZoom</label>
    </div>

    <div>
      <button @click="hidden = !hidden">toggle hidden</button>
    </div>

    <div>
      <p>{{ dragging ? 'is dragging' : 'not dragging' }}</p>
    </div>
  </form>

  <div>
    <VueZoomable v-if="!hidden" ref="child" style="width: 100%; height: 500px; border: 1px solid black"
      selector="#zoomable-content" v-model:dragging="dragging" v-model:pan="pan" v-model:zoom="zoom"
      v-model:enableZoom="enableZoom" v-model:enablePan="enablePan"
      :enableWheelOnKey="documentFlow ? 'Control' : undefined">
      <div id="zoomable-content">
        <div>
          <div></div>
          <div></div>
        </div>
        <div>
          <div></div>
          <div id="to-center">A-312</div>
        </div>

        <div class="center"></div>
      </div>
    </VueZoomable>
    <div v-else>
      hihi
    </div>
  </div>

  <div>
    zoom: {{ zoom }}
  </div>
  <div>
    pan: {{ pan }}
  </div>

  <button @click="center">center</button>

  <section v-if="documentFlow">
    <h1>Chapter 1</h1>
    <p>
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce non ultrices mi, nec cursus justo. Ut efficitur,
      dui
      nec consectetur consequat, lorem elit tincidunt metus, vel ultrices elit velit ac lectus. Proin in eros non nisi
      venenatis bibendum. Integer nec neque sit amet velit varius tempus. Integer viverra ligula nec nunc egestas, non
      ullamcorper mauris venenatis. Nullam sit amet pharetra odio, eget ultrices enim. Aenean non nisl auctor, vulputate
      quam sit amet, tincidunt lectus. Duis vitae elit sed justo tincidunt sodales id ac est. Vivamus eu orci dapibus,
      hendrerit nulla in, vehicula libero.
    </p>
    <p>
      Sed pulvinar bibendum metus, quis lacinia urna varius non. Duis eget velit quam. Vestibulum consectetur vehicula
      facilisis. Nunc egestas et enim ut facilisis. In volutpat augue eget risus faucibus malesuada. Ut et quam elit.
      Fusce quis tincidunt elit. Nullam gravida justo ut feugiat mollis.
    </p>
    <h2>Section 1.1</h2>
    <p>
      Vivamus maximus scelerisque ligula, vel fringilla odio consectetur nec. Duis eleifend, erat quis maximus ultrices,
      elit arcu congue nibh, nec elementum eros purus nec nulla. Fusce aliquet lacus id ligula rhoncus, in sodales
      lectus
      fringilla. Maecenas et purus et erat pulvinar interdum vel sit amet quam. Vestibulum facilisis turpis nec metus
      congue posuere.
    </p>
    <h2>Section 1.2</h2>
    <p>
      Nulla vehicula lectus felis, in feugiat massa dignissim id. Vivamus eget magna ac eros viverra mattis ac sit amet
      nulla. Pellentesque venenatis risus ut ex suscipit, in auctor nulla consequat. Proin rhoncus semper risus, a
      dapibus
      turpis luctus a. Nullam nec tincidunt sapien. Vivamus gravida ultricies lacus, in eleifend nunc cursus ut. In hac
      habitasse platea dictumst. Duis ut purus nec sapien convallis pellentesque vel eu erat.
    </p>
    <!-- Continue with more paragraphs, sections, and chapters as needed to fill a couple of pages -->

    <h1>Chapter 2</h1>
    <p>
      Phasellus blandit velit at eros efficitur, in mollis dui feugiat. Fusce euismod mauris nec varius volutpat.
      Quisque
      dapibus augue et ex ultricies, ac vestibulum dolor facilisis. Sed mattis est sed ipsum feugiat, ut blandit nunc
      tristique. Aliquam et volutpat nulla, vel ullamcorper purus. Proin condimentum lacus ac congue varius. Maecenas a
      cursus elit. Nulla facilisi. Integer eu quam eget arcu laoreet vehicula.
    </p>
    <h2>Section 2.1</h2>
    <p>
      Nunc rhoncus, risus nec euismod porttitor, urna nisi accumsan turpis, vitae euismod turpis arcu eget velit. Nulla
      a
      elit vel enim accumsan egestas. Nulla facilisi. Duis nec magna risus. Etiam euismod hendrerit dolor. Integer vel
      est
      vitae purus auctor vehicula. Vivamus feugiat felis id tortor hendrerit blandit. Nullam nec tortor eu neque
      tincidunt
      mollis. Aenean tincidunt sit amet lacus eu suscipit. Fusce vitae nulla ultrices, convallis odio non, accumsan
      justo.
    </p>
    <h2>Section 2.2</h2>
    <p>
      Aenean in nibh eget velit aliquet iaculis. Donec et purus ut lectus hendrerit efficitur. Maecenas vulputate justo
      nec enim ullamcorper luctus. Integer varius nisi eu massa iaculis, vel auctor erat blandit. Nulla facilisi.
      Praesent
      eget bibendum lectus. Donec dignissim velit id nisl feugiat, a vestibulum felis consequat. Pellentesque habitant
      morbi tristique senectus et netus et malesuada fames ac turpis egestas.
    </p>
    <!-- Continue with more paragraphs, sections, and chapters as needed to fill a couple of pages -->
    <h1>Chapter 1</h1>
    <p>
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce non ultrices mi, nec cursus justo. Ut efficitur,
      dui
      nec consectetur consequat, lorem elit tincidunt metus, vel ultrices elit velit ac lectus. Proin in eros non nisi
      venenatis bibendum. Integer nec neque sit amet velit varius tempus. Integer viverra ligula nec nunc egestas, non
      ullamcorper mauris venenatis. Nullam sit amet pharetra odio, eget ultrices enim. Aenean non nisl auctor, vulputate
      quam sit amet, tincidunt lectus. Duis vitae elit sed justo tincidunt sodales id ac est. Vivamus eu orci dapibus,
      hendrerit nulla in, vehicula libero.
    </p>
    <p>
      Sed pulvinar bibendum metus, quis lacinia urna varius non. Duis eget velit quam. Vestibulum consectetur vehicula
      facilisis. Nunc egestas et enim ut facilisis. In volutpat augue eget risus faucibus malesuada. Ut et quam elit.
      Fusce quis tincidunt elit. Nullam gravida justo ut feugiat mollis.
    </p>
    <h2>Section 1.1</h2>
    <p>
      Vivamus maximus scelerisque ligula, vel fringilla odio consectetur nec. Duis eleifend, erat quis maximus ultrices,
      elit arcu congue nibh, nec elementum eros purus nec nulla. Fusce aliquet lacus id ligula rhoncus, in sodales
      lectus
      fringilla. Maecenas et purus et erat pulvinar interdum vel sit amet quam. Vestibulum facilisis turpis nec metus
      congue posuere.
    </p>
    <h2>Section 1.2</h2>
    <p>
      Nulla vehicula lectus felis, in feugiat massa dignissim id. Vivamus eget magna ac eros viverra mattis ac sit amet
      nulla. Pellentesque venenatis risus ut ex suscipit, in auctor nulla consequat. Proin rhoncus semper risus, a
      dapibus
      turpis luctus a. Nullam nec tincidunt sapien. Vivamus gravida ultricies lacus, in eleifend nunc cursus ut. In hac
      habitasse platea dictumst. Duis ut purus nec sapien convallis pellentesque vel eu erat.
    </p>
    <!-- Continue with more paragraphs, sections, and chapters as needed to fill a couple of pages -->

    <h1>Chapter 2</h1>
    <p>
      Phasellus blandit velit at eros efficitur, in mollis dui feugiat. Fusce euismod mauris nec varius volutpat.
      Quisque
      dapibus augue et ex ultricies, ac vestibulum dolor facilisis. Sed mattis est sed ipsum feugiat, ut blandit nunc
      tristique. Aliquam et volutpat nulla, vel ullamcorper purus. Proin condimentum lacus ac congue varius. Maecenas a
      cursus elit. Nulla facilisi. Integer eu quam eget arcu laoreet vehicula.
    </p>
    <h2>Section 2.1</h2>
    <p>
      Nunc rhoncus, risus nec euismod porttitor, urna nisi accumsan turpis, vitae euismod turpis arcu eget velit. Nulla
      a
      elit vel enim accumsan egestas. Nulla facilisi. Duis nec magna risus. Etiam euismod hendrerit dolor. Integer vel
      est
      vitae purus auctor vehicula. Vivamus feugiat felis id tortor hendrerit blandit. Nullam nec tortor eu neque
      tincidunt
      mollis. Aenean tincidunt sit amet lacus eu suscipit. Fusce vitae nulla ultrices, convallis odio non, accumsan
      justo.
    </p>
    <h2>Section 2.2</h2>
    <p>
      Aenean in nibh eget velit aliquet iaculis. Donec et purus ut lectus hendrerit efficitur. Maecenas vulputate justo
      nec enim ullamcorper luctus. Integer varius nisi eu massa iaculis, vel auctor erat blandit. Nulla facilisi.
      Praesent
      eget bibendum lectus. Donec dignissim velit id nisl feugiat, a vestibulum felis consequat. Pellentesque habitant
      morbi tristique senectus et netus et malesuada fames ac turpis egestas.
    </p>
    <!-- Continue with more paragraphs, sections, and chapters as needed to fill a couple of pages -->
  </section>
</template>

<script setup lang="ts">
import { ref } from "vue";
import VueZoomable from "../components/VueZoomable.vue";

const child = ref();
function center() {
  child.value.centerElementWithZoom(document.getElementById('to-center'), 0);
}

const zoom = ref(1);
const pan = ref({ x: 0, y: 0 });
const dragging = ref(false);
let documentFlow = ref(false);

const enablePan = ref(true);
const enableZoom = ref(true);

const hidden = ref(false);
</script>

<style>
.mysvg {
  height: 100%;
  width: 100%;
}

#zoomable-content {
  box-sizing: border-box;
  border: red 1px solid;

  width: 400px;
  height: 400px;
  display: flex;
  flex-direction: column;
}

#zoomable-content>.center {
  position: absolute;
  top: 195px;
  left: 195px;

  width: 10px;
  height: 10px;

  background-color: greenyellow;
}

#zoomable-content>div {
  display: flex;
  flex-direction: row;
}

#zoomable-content>div>div {
  margin: 50px;
  background-color: blue;
  height: 100px;
  width: 100px;
}

#to-center {
  background-color: aquamarine !important;
}
</style>
