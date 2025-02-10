<template>
    <teleport to="body">
        <div v-if="modelValue" class="drawer-overlay" @click="handleOverlayClick">
            <div ref="drawer" class="drawer-content" :class="positionClass" @click.stop :style="drawerStyles"
                @mousedown="startDrag" @touchstart="startDrag">
                <div class="title">
                    <h1>{{ title }}</h1>
                    <p class="description"> {{ description }}</p>
                </div>
                <slot />
                <div class="drag-handle" @mousedown="startDrag" @touchstart="startDrag"></div>
            </div>
        </div>
    </teleport>
</template>

<script setup>
import { ref, defineProps, defineEmits, computed, watch, nextTick } from 'vue';

const props = defineProps({
    modelValue: Boolean,
    height: { type: Number, default: 300 },
    position: { type: String, default: 'bottom' },
    title: { type: String, default: 'Drawer title' },
    description: { type: String, default: 'Drawer description' },
    marginLeft: { type: String, default: '' },
    width: { type: String, default: '100%' },
});

const emit = defineEmits(['update:modelValue']);

const translateY = ref(0);
const isDragging = ref(false);
const startY = ref(0);
const currentY = ref(0);

const positionClass = computed(() => props.position === 'bottom' ? 'bottom' : 'top');

const drawerStyles = computed(() => {
    return {
        transform: `translateY(${translateY.value}px)`,
        height: `${props.height}px`,
        marginLeft: props.marginLeft,
        width: props.width,
    };
});

const startDrag = (event) => {
    isDragging.value = true;
    startY.value = event.touches ? event.touches[0].clientY : event.clientY;

    window.addEventListener('mousemove', drag);
    window.addEventListener('mouseup', stopDrag);
    window.addEventListener('touchmove', drag);
    window.addEventListener('touchend', stopDrag);
};

const drag = (event) => {
    if (!isDragging.value) return;
    currentY.value = event.touches ? event.touches[0].clientY : event.clientY;
    const delta = currentY.value - startY.value;

    if (props.position === 'top') {
        // Prevent dragging the bottom drawer above 1px
        translateY.value = Math.min(delta, 1);
    } else {
        // Prevent dragging the top drawer beyond -10px
        translateY.value = Math.max(delta, -2);
    }
};


const stopDrag = () => {
    isDragging.value = false;
    const delta = currentY.value - startY.value;
    const threshold = 0.3 * props.height;

    if (props.position === 'top' && delta < -threshold) {
        close();
    } else if (props.position === 'bottom' && delta > threshold) {
        close();
    } else {
        translateY.value = 0;
    }

    window.removeEventListener('mousemove', drag);
    window.removeEventListener('mouseup', stopDrag);
    window.removeEventListener('touchmove', drag);
    window.removeEventListener('touchend', stopDrag);
};

const close = () => {
    translateY.value = props.position === 'top' ? -props.height : props.height;
    setTimeout(() => {
        emit('update:modelValue', false);
    }, 200);
};

const handleOverlayClick = () => {
    close();
};

watch(() => props.modelValue, (newVal) => {
    if (newVal) {
        translateY.value = props.position === 'top' ? -props.height : props.height;
        setTimeout(() => {
            translateY.value = 0;
        }, 10);
    }
});
</script>

<style scoped>
.drawer-overlay {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
}

/* Drawer Panel */
.drawer-content {
    background: white;
    width: 100%;
    position: absolute;
    transition: transform 0.2s ease-out;
    border-top-left-radius: 12px;
    border-top-right-radius: 12px;
}

/* Positioning Classes */
.drawer-content.top {
    top: 0;
}

.drawer-content.bottom {
    bottom: 0;
}

/* Drag Handle */
.drag-handle {
    width: 70px;
    height: 10px;
    background: gray;
    border-radius: 5px;
    cursor: grab;
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    margin-top: 10px;
}

/* For top drawer, place the handle at the bottom edge */
.drawer-content.top .drag-handle {
    bottom: 0;
}

/* For bottom drawer, place the handle at the top edge */
.drawer-content.bottom .drag-handle {
    top: 0;
}

.title {
    font-size: 0.6rem;
    margin-top: 30px;
    width: 50%;
    display: table;
    padding-left: 20px;
}

.description {
    margin-top: -5px;
    font-size: 1rem;
}
</style>
