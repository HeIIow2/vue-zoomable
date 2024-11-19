<template>
  <div ref="container" class="container" :class="$style.container">
    <slot></slot>
    <ControlButtons v-if="props.enableControlButton" @button-home="onHome" @button-pan="onPan" @button-zoom="onZoom" />

    <ScrollOverlay v-model="showOverlay">
      Use '{{ props.enableWheelOnKey }}' + 'scroll' to zoom.
    </ScrollOverlay>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch, nextTick, onMounted, onBeforeUnmount } from 'vue';
import ControlButtons from "./ControlButtons.vue";
import ScrollOverlay from './ScrollOverlay.vue';

let props = defineProps({
  selector: {
    type: String,
    default: "* > *",
  },
  maxZoom: {
    type: Number,
    default: -1,
  },
  minZoom: {
    type: Number,
    default: 0.5,
  },
  dblClickZoomStep: {
    type: Number,
    default: 0.4,
  },
  wheelZoomStep: {
    type: Number,
    default: 0.05,
  },
  enableControlButton: {
    type: Boolean,
    default: true,
  },
  buttonPanStep: {
    type: Number,
    default: 15,
  },
  buttonZoomStep: {
    type: Number,
    default: 0.1,
  },
  enableWheelOnKey: {
    type: String,
    default: undefined,
  },
  debug: {
    type: Boolean,
    default: false,
  },
  draggingDelay: {
    type: Number,
    default: 10,
  },
  keepOverlayOpen: {
    type: Number,
    default: 1000,
  }
});

let container = ref();
let transformTarget = computed<HTMLElement>(() => container.value?.querySelector(props.selector))

const dragging = defineModel('dragging', { default: false });
const showOverlay = defineModel('showOverlay', { default: false });
const zoom = defineModel('zoom', { default: 1 });
const pan = defineModel('pan', { default: { x: 0, y: 0 } });

const enablePan = defineModel('enablePan', { default: true });
const enableZoom = defineModel('enableZoom', { default: true });

/*
 * ################################# WATCHERS #################################
 */
{

  watch(transformTarget, onHome, { once: true });

  let zoomWatch = false;
  watch(zoom, (newZoom: number, oldZoom: number) => {
    if (zoomWatch) {
      zoomWatch = false;
      return;
    }

    showOverlay.value = false;

    const setZoom = (z: number) => {
      zoomWatch = true;
      zoom.value = z;
    }

    if (!enableZoom.value) {
      setZoom(oldZoom);
      return;
    }

    if (zoom.value < props.minZoom) setZoom(props.minZoom);
    else if (zoom.value > props.maxZoom && props.maxZoom > 0) setZoom(props.maxZoom);
  });

  let panWatch = false;
  watch(pan, (newPan: { x: number, y: number }, oldPan: { x: number, y: number }) => {
    if (panWatch) {
      panWatch = false;
      return;
    }

    showOverlay.value = false;

    if (!enablePan.value) {
      panWatch = true;
      pan.value = oldPan;
      return;
    }
  }, { deep: true });

  let lastOverlay = Date.now()
  function changeShowOverlay() {
    if (showOverlay.value) {
      lastOverlay = Date.now();

      setTimeout(() => {
        if (lastOverlay + props.keepOverlayOpen < Date.now()) {
          showOverlay.value = false;
        }
      }, props.keepOverlayOpen + 10);
    }
  }
  watch(showOverlay, changeShowOverlay);

  let transform = computed(() => {
    return `translate(${pan.value.x}px, ${pan.value.y}px) scale(${zoom.value})`;
  });

  function setTransform() {
    if (!transformTarget.value) return;
    transformTarget.value.style.transform = transform.value;
  }

  watch(transform, setTransform, { flush: "post" });
}

function getCenter(info: DOMRect) {
  return {
    x: info.x + .5 * info.width,
    y: info.y + .5 * info.height,
  };
}

function centerElement(element: HTMLElement) {
  centerPoint(getCenter(element.getBoundingClientRect()));
}

const logConst = 1 / Math.log(2);
async function centerElementWithZoom(element: HTMLElement, p: number = 0) {
  if (!container.value || !element) return;

  const containerDimensions = container.value.getBoundingClientRect();
  const elementDimensions = element.getBoundingClientRect();
  zoom.value = Math.min(containerDimensions.width / ((elementDimensions.width / zoom.value) + p), containerDimensions.height / ((elementDimensions.height / zoom.value) + p));
  await nextTick();

  centerElement(element);
}

function centerPoint(pointToCenter: { x: number, y: number }) {
  const containerInfo = container.value.getBoundingClientRect();

  pan.value = {
    x: - (pointToCenter.x - containerInfo.x - pan.value.x) + .5 * containerInfo.width,
    y: - (pointToCenter.y - containerInfo.y - pan.value.y) + .5 * containerInfo.height,
  }
}

async function zoomIntoPoint(deltaZoom: number, oldPoint: { x: number, y: number }) {
  // changing the zoom, zooms into the center of transform target. 
  // thus I need to find the point, relative to the transform target center, 
  // to calculate by how much I need to shift the pan for each zoom
  const centerTransformTarget = getCenter(transformTarget.value.getBoundingClientRect());
  const normalizedDistance = {
    x: (oldPoint.x - centerTransformTarget.x) / zoom.value,
    y: (oldPoint.y - centerTransformTarget.y) / zoom.value,
  };

  zoom.value += deltaZoom;
  await nextTick();

  const newPoint = {
    x: (normalizedDistance.x * zoom.value) + centerTransformTarget.x,
    y: (normalizedDistance.y * zoom.value) + centerTransformTarget.y,
  };

  // move new point on the transform target to the old point on the canvas
  pan.value = {
    x: pan.value.x + (oldPoint.x - newPoint.x),
    y: pan.value.y + (oldPoint.y - newPoint.y),
  }
}

/*
 * ################################# DOUBLE CLICK #################################
 */
function onDblclick(event: PointerEvent) {
  centerPoint({ x: event.clientX, y: event.clientY });
  zoom.value += props.dblClickZoomStep;
}
onMounted(() => container.value.addEventListener('dblclick', onDblclick));
onBeforeUnmount(() => container.value.removeEventListener('dblclick', onDblclick))

/*
 * ################################# MOUSEWHEEL #################################
 */
const pressedKeys: Set<String> = new Set<String>();
const isInContainer = ref(false);

const onkeydown = (event: KeyboardEvent) => { pressedKeys.add(event.key); }
const onkeyup = (event: KeyboardEvent) => { pressedKeys.delete(event.key); }
const onmouseenter = () => { isInContainer.value = true };
const onmouseleave = () => { isInContainer.value = false };
const preventingZoom = (event: WheelEvent) => {
  if (!isInContainer.value || props.enableWheelOnKey !== "Control") return;
  if (event.ctrlKey) event.preventDefault();
};
const onwheel = (event: WheelEvent) => {
  if (props.enableWheelOnKey !== undefined && !pressedKeys.has(props.enableWheelOnKey)) {
    showOverlay.value = true;
    return;
  }

  zoomIntoPoint(props.dblClickZoomStep * event.deltaY / Math.abs(event.deltaY), { x: event.clientX, y: event.clientY })
};

onMounted(() => {
  // track the currently pressed keys, and mouse location
  document.addEventListener('keydown', onkeydown);
  document.addEventListener('keyup', onkeyup);
  container.value.addEventListener('mouseenter', onmouseenter);
  container.value.addEventListener('mouseleave', onmouseleave);

  // prevent zooming in the viewport if necessary
  window.addEventListener('wheel', preventingZoom, { passive: false });

  // the actual scrolling event within container
  container.value.addEventListener('wheel', onwheel);
});
onBeforeUnmount(() => {
  // track the currently pressed keys, and mouse location
  document.removeEventListener('keydown', onkeydown);
  document.removeEventListener('keyup', onkeyup);
  container.value.removeEventListener('mouseenter', onmouseenter);
  container.value.removeEventListener('mouseleave', onmouseleave);

  // prevent zooming in the viewport if necessary
  window.removeEventListener('wheel', preventingZoom);

  // the actual scrolling event within container
  container.value.removeEventListener('wheel', onwheel);
})


/*
 * ################################# TOUCH/POINTER PAN/ZOOM #################################
 */
let movingEventId = new Set();
let eventCache = new Array();
let previousEvents: { [key: number]: PointerEvent } = {};
let currentEvents: { [key: number]: PointerEvent } = {};
let previousDistance: number | undefined = undefined;

function pointerdown_handler(event: PointerEvent) {
  if (event.button === 1 || event.button === 2) return;

  eventCache.push(event);
  currentEvents[event.pointerId] = event;
  previousEvents[event.pointerId] = event;
}

function pointerup_handler(event: PointerEvent) {
  for (var i = 0; i < eventCache.length; i++) {
    if (eventCache[i].pointerId == event.pointerId) {
      eventCache.splice(i, 1);
      break;
    }
  }
  delete currentEvents[event.pointerId];
  delete previousEvents[event.pointerId];
  movingEventId.delete(event.pointerId);

  setTimeout(() => {
    dragging.value = movingEventId.size > 0;
  }, props.draggingDelay);

  if (eventCache.length !== 2) previousDistance = undefined;
}

function pointermove_handler(event: PointerEvent) {
  if (!eventCache.some(testEvent => testEvent.pointerId === event.pointerId)) return;
  previousEvents[event.pointerId] = currentEvents[event.pointerId];
  currentEvents[event.pointerId] = event;

  movingEventId.add(event.pointerId);
  dragging.value = true;

  if (eventCache.length == 2) {
    const currentDistance = Math.sqrt(
      Math.pow(currentEvents[eventCache[1].pointerId].clientX - currentEvents[eventCache[0].pointerId].clientX, 2) +
      Math.pow(currentEvents[eventCache[1].pointerId].clientY - currentEvents[eventCache[0].pointerId].clientY, 2),
    );

    const previousPosition = {
      x: (previousEvents[eventCache[0].pointerId].clientX + previousEvents[eventCache[1].pointerId].clientX) / 2,
      y: (previousEvents[eventCache[0].pointerId].clientY + previousEvents[eventCache[1].pointerId].clientY) / 2,
    };

    const currentPosition = {
      x: (currentEvents[eventCache[0].pointerId].clientX + currentEvents[eventCache[1].pointerId].clientX) / 2,
      y: (currentEvents[eventCache[0].pointerId].clientY + currentEvents[eventCache[1].pointerId].clientY) / 2,
    };

    if (previousDistance !== undefined) {
      zoomIntoPoint((currentDistance - previousDistance) * 0.01, currentPosition);
    }

    if (currentPosition.x !== previousPosition.x || currentPosition.y !== previousPosition.y) {
      pan.value = {
        x: pan.value.x + (currentPosition.x - previousPosition.x),
        y: pan.value.y + (currentPosition.y - previousPosition.y),
      };
    }

    previousDistance = currentDistance;
  }

  else if (eventCache.length == 1) {
    const previous = previousEvents[event.pointerId];
    const current = currentEvents[event.pointerId]

    pan.value = {
      x: pan.value.x + (current.clientX - previous.clientX),
      y: pan.value.y + (current.clientY - previous.clientY),
    };
  }
}

onMounted(() => {
  container.value.addEventListener('pointerdown', pointerdown_handler);
  window.addEventListener('pointerup', pointerup_handler);
  window.addEventListener('pointermove', pointermove_handler);

  container.value.style.touchAction = 'none';
});
onBeforeUnmount(() => {
  container.value.removeEventListener('pointerdown', pointerup_handler);
  window.removeEventListener('pointerdown', pointerdown_handler);
  window.removeEventListener('pointermove', pointermove_handler);

  container.value.style.removeProperty('touch-action');
});

/*
 * ################################# BUTTONS #################################
 */
interface PanDirection {
  x: number;
  y: number;
}

let isHolding = false;

function holding(callback: Function, delay: number) {
  if (!isHolding) return;

  callback();
  setTimeout(() => { holding(callback, delay) }, delay);
}

function onPan(direction: PanDirection, usingHold: boolean = false) {
  pan.value = {
    x: pan.value.x + direction.x * props.buttonPanStep,
    y: pan.value.y + direction.y * props.buttonPanStep
  }

  isHolding = usingHold;
  if (isHolding) {
    setTimeout(() => {
      holding(() => {
        pan.value = {
          x: pan.value.x + direction.x * props.buttonPanStep * 0.5,
          y: pan.value.y + direction.y * props.buttonPanStep * 0.5
        }
      }, 50)
    }, 300);
  }
}

function onZoom(direction: number, usingHold: boolean = false) {
  zoomIntoPoint(direction * props.buttonZoomStep, getCenter(container.value.getBoundingClientRect()));


  isHolding = usingHold;
  if (isHolding) {
    setTimeout(() => {
      holding(() => {
        zoomIntoPoint(direction * props.buttonZoomStep * 0.5, getCenter(container.value.getBoundingClientRect()));
        zoom.value += direction * props.buttonZoomStep * 0.5;

      }, 50)
    }, 300);
  }
}

async function onHome() {
  centerElementWithZoom(transformTarget.value, 0);
}

defineExpose({
  centerElement,
  centerElementWithZoom,
  onPan,
  onZoom,
  onHome,
})
</script>

<style module>
.container {
  overflow: hidden;
  position: relative;

  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
</style>
