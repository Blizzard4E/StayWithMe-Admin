<script>
    import "../app.scss";
</script>

<div class="bg">
    <slot />
</div>

<style lang="scss">
    .bg {
        height: 100vh;
        overflow: auto;
        background-color: $bg_dark;
    }
</style>
