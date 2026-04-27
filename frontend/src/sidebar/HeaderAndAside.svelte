<script lang="ts">
  import AsideWithButton from "./AsideWithButton.svelte";
  import Header from "./Header.svelte";

  let active = $state(false);
  const toggle = () => {
    active = !active;
  };
</script>

<Header {toggle} />
<AsideWithButton {active} {toggle} />
