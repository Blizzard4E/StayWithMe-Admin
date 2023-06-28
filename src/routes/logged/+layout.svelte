<script>
  import "../../app.scss";
</script>


<div class="bg">
  <slot />
</div>

<style lang="scss">
  .bg{
    height: 100vh;
    overflow: hidden;
    background-color: $bg_dark;
    display: grid;
    grid-template-columns: 15% 85%;
  }
</style>