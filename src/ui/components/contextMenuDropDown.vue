<template>
<div
    class="c-so-view__context-actions c-disclosure-button"
    @click="showContextMenu"
></div>
</template>

<script>
import contextMenu from '../mixins/context-menu-gesture'

export default {
    mixins: [contextMenu],
    props: {
        objectPath: {
            type: Array,
            required: true
        }
    }
}
</script>

