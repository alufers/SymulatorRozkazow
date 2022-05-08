<script lang="ts">
  export let value: number = 0;
  export let flash: null | string = null;
  let isError: boolean = false;
  $: valueHexStr = ((value: number) => {
    if (isNaN(value)) {
      return "0x00";
    }
    if (value < 0) {
      return "0x00";
    }
    if (value > 0xff) {
      return "0xff";
    }

    return "0x" + value.toString(16).padStart(2, "0").toUpperCase();
  })(value);
  const onInput = (e) => {
    let val = e.target.value;
    if (val.match(/^[0-9a-fA-F]{1,2}$/)) {
      val = "0x" + val;
    }
    if (!val.match(/^0x[0-9a-fA-F]{1,3}$/)) {
      isError = true;
      return;
    }
    isError = false;
    value = parseInt(val.replace("0x", ""), 16);
  };
  const clicked = (e) => {
    // select all
    e.target.select();
  };
</script>

<input
  value={valueHexStr}
  on:input={onInput}
  class:error={isError}
  class:flashed-write={flash === "write"}
  class:flashed-read={flash === "read"}
  class:flashed-read-index={flash === "read-index"}
  on:click={clicked}
/>

<style lang="less">
  input {
    font-family: monospace;
    width: 50px;
    height: 30px;
    margin-bottom: 0;
    &.error {
      background: #ff000044;
    }
    &.flashed-write {
      background: #e84393aa;
      animation: flashed-write 0.5s;
    }
    &.flashed-read {
      background: #55efc4aa;
      animation: flashed-read 0.5s;
    }
    &.flashed-read-index {
      background: #0984e3aa;
      animation: flashed-read-index 0.5s;
    }
  }
  @keyframes flashed-write {
    0% {
      background: #e84393aa;
    }
    100% {
      background: #00000000;
    }
  }
  @keyframes flashed-read {
    0% {
      background: #55efc4aa;
    }
    100% {
      background: #00000000;
    }
  }
  @keyframes flashed-read-index {
    0% {
      background: #0984e3aa;
    }
    100% {
      background: #00000000;
    }
  }
</style>
