<template>
    <component :is="tag">
        <slot />
    </component>
</template>
<script>
export default {
    props: {
        tag: {
            type: String,
            default: 'main',
        },
    },
}
</script>
