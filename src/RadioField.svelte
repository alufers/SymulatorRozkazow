<script context="module" lang="ts">
  export interface Option {
    name: string | number;
    label?: string;
  }
</script>

<script lang="ts">
  export let value: number = 0;
  export let label = "";
  export let options: Option[] = [];
  let id = Math.random().toString(36).substring(2);
</script>

<div class="radio-field">
  <label for="" class="main-label">{label}</label>
  {#each options as option}
    <div class="radio-option">
      <input
        type="radio"
        id={id + option.name}
        name={id}
        value={option.name}
        checked={option.name.toString() === value.toString()}
        on:click={(e) => {
          value = options.find(
            (o) => o.name.toString() === e.target.value
          ).name;
        }}
      />
      <label for={id + option.name}>{option.label}</label>
    </div>
  {/each}
</div>

<style lang="less">
  .main-label {
    margin-top: 8px;
    margin-bottom: 4px;
  }
  label {
    text-transform: uppercase;
    font-weight: bold;
    font-size: 0.7rem;
  }
  .radio-option {
    display: flex;
    flex-direction: row;
    flex-wrap: nowrap;
    input {
      margin-right: 5px;
      margin-bottom: 0;
    }
    label {
      font-weight: normal;
    }
    margin-bottom: 4px;
    padding-left: 8px;
  }
</style>
