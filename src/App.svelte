<script lang="ts">
  import HexInput from "./HexInput.svelte";
  import RadioField, { options } from "./RadioField.svelte";
  import type { Option } from "./RadioField.svelte";

  enum CopyMode {
    RegisterToRegister,
    RegisterToMemory,
    MemoryToRegister,
  }
  const copyModeOptions: Option[] = [
    { name: CopyMode.RegisterToRegister, label: "Rejestr do rejestru" },
    { name: CopyMode.RegisterToMemory, label: "Rejestr do pamięci" },
    { name: CopyMode.MemoryToRegister, label: "Pamięć do rejestru" },
  ];
  enum AddressingMode {
    Base,
    Indexed,
    BaseIndexed,
  }
  const addressingModeOptions: Option[] = [
    { name: AddressingMode.Base, label: "Bazowy" },
    { name: AddressingMode.Indexed, label: "Indeksowy" },
    { name: AddressingMode.BaseIndexed, label: "Bazowo indeksowy" },
  ];

  let copyMode: CopyMode = CopyMode.RegisterToRegister;
  let addressingMode: AddressingMode = AddressingMode.Base;

  let registers = {
    DISP: 0,

    AX: 0,
    BX: 0,
    CX: 0,
    DX: 0,

    BP: 0,
    DI: 0,
    SI: 0,

    SP: 0xa9,
  };
  let registersOrder = [
    "AX",
    "BX",
    "CX",
    "DX",
    "---",
    "BP",
    "DI",
    "SI",
    "---",
    "SP",
  ];
  let source = "AX";
  let destination = "BX";

  let instructions = [
    {
      name: "MOV",
      execute() {
        destinationOptions
          .find((o) => o.name === destination)
          .setValue(sourceOptions.find((o) => o.name === source).getValue());
        // flashField(destination, "write");
        // flashField(source, "read");
      },
    },
    {
      name: "XCHG",
      execute() {
        let temp = sourceOptions.find((o) => o.name === source).getValue();
        sourceOptions
          .find((o) => o.name === source)
          .setValue(
            destinationOptions.find((o) => o.name === destination).getValue()
          );
        destinationOptions.find((o) => o.name === destination).setValue(temp);
        // flashField(destination, "write");
        // flashField(source, "write");
      },
    },
    {
      name: "PUSH",
      execute() {
        let value = sourceOptions.find((o) => o.name === source).getValue();
        registers.SP -= 1;
        memory[registers.SP] = value;
        flashField("mem_" + registers.SP, "write");
        flashField("SP", "write");
      },
    },
    {
      name: "POP",
      execute() {
        let value = memory[registers.SP];
        registers.SP += 1;
        destinationOptions.find((o) => o.name === destination).setValue(value);
        flashField("mem_" + registers.SP, "read");
        flashField("SP", "write");
      },
    },
  ];

  interface Operand {
    name: string;
    getValue(): number;
    setValue(value: number): void;
  }
  const registerOperands: Operand[] = [
    ...["AX", "BX", "CX", "DX"].map((name) => ({
      name,
      getValue() {
        flashField(name, "read");
        return registers[name];
      },
      setValue(value) {
        registers[name] = value;
        flashField(name, "write");
      },
    })),
  ];
  const baseOperands: Operand[] = [
    {
      name: "[BP+DISP]",
      getValue() {
        flashField("BP", "read-index");
        flashField("DISP", "read-index");
        flashField("mem_" + (registers["BP"] + registers["DISP"]), "read");
        return memory[registers["BP"] + registers["DISP"]];
      },
      setValue(value) {
        flashField("BP", "read-index");
        flashField("DISP", "read-index");
        flashField("mem_" + (registers["BP"] + registers["DISP"]), "write");
        memory[registers["BP"] + registers["DISP"]] = value;
      },
    },
  ];
  $: sourceOptions = (() => {
    if (
      copyMode === CopyMode.RegisterToRegister ||
      copyMode === CopyMode.RegisterToMemory
    ) {
      return registerOperands;
    }
    if (copyMode === CopyMode.MemoryToRegister) {
      if (addressingMode === AddressingMode.Base) {
        return baseOperands;
      }
      if (addressingMode === AddressingMode.Indexed) {
        return [
          {
            name: "[SI+DISP]",
            getValue() {
              flashField("SI", "read-index");
              flashField("DISP", "read-index");
              flashField(
                "mem_" + (registers["SI"] + registers["DISP"]),
                "read"
              );
              return memory[registers["SI"] + registers["DISP"]];
            },
            setValue(value) {
              flashField("SI", "read-index");
              flashField("DISP", "read-index");
              flashField(
                "mem_" + (registers["SI"] + registers["DISP"]),
                "write"
              );
              memory[registers["SI"] + registers["DISP"]] = value;
            },
          },
        ];
      }
      if (addressingMode === AddressingMode.BaseIndexed) {
        return [
          {
            name: "[BP+SI+DISP]",
            getValue() {
              flashField("BP", "read-index");
              flashField("SI", "read-index");
              flashField("DISP", "read-index");
              flashField(
                "mem_" +
                  (registers["BP"] + registers["SI"] + registers["DISP"]),
                "read"
              );
              return memory[
                registers["BP"] + registers["SI"] + registers["DISP"]
              ];
            },
            setValue(value) {
              flashField("BP", "read-index");
              flashField("SI", "read-index");
              flashField("DISP", "read-index");
              flashField(
                "mem_" +
                  (registers["BP"] + registers["SI"] + registers["DISP"]),
                "write"
              );
              memory[registers["BP"] + registers["SI"] + registers["DISP"]] =
                value;
            },
          },
        ];
      }
    }
  })();
  $: destinationOptions = (() => {
    if (
      copyMode === CopyMode.RegisterToRegister ||
      copyMode === CopyMode.MemoryToRegister
    ) {
      return registerOperands;
    }

    if (copyMode === CopyMode.RegisterToMemory) {
      if (addressingMode === AddressingMode.Base) {
        return baseOperands;
      }
      if (addressingMode === AddressingMode.Indexed) {
        return [
          {
            name: "[DI+DISP]",
            getValue() {
              flashField("DI", "read-index");
              flashField("DISP", "read-index");
              flashField(
                "mem_" + (registers["DI"] + registers["DISP"]),
                "read"
              );
              return memory[registers["DI"] + registers["DISP"]];
            },
            setValue(value) {
              flashField("DI", "read-index");
              flashField("DISP", "read-index");
              flashField(
                "mem_" + (registers["DI"] + registers["DISP"]),
                "write"
              );
              memory[registers["DI"] + registers["DISP"]] = value;
            },
          },
        ];
      }
      if (addressingMode === AddressingMode.BaseIndexed) {
        return [
          {
            name: "[BP+DI+DISP]",
            getValue() {
              flashField("BP", "read-index");
              flashField("DI", "read-index");
              flashField("DISP", "read-index");
              flashField(
                "mem_" +
                  (registers["BP"] + registers["DI"] + registers["DISP"]),
                "read"
              );
              return memory[
                registers["BP"] + registers["DI"] + registers["DISP"]
              ];
            },
            setValue(value) {
              flashField("BP", "read-index");
              flashField("DI", "read-index");
              flashField("DISP", "read-index");
              flashField(
                "mem_" +
                  (registers["BP"] + registers["DI"] + registers["DISP"]),
                "write"
              );
              memory[registers["BP"] + registers["DI"] + registers["DISP"]] =
                value;
            },
          },
        ];
      }
    }
  })();
  $: (() => {
    // check if destination is in not destinationOptions
    if (!destinationOptions.find((o) => o.name === destination)) {
      // set it to the first one
      destination = destinationOptions[0].name;
    }
    // check if source is in not sourceOptions
    if (!sourceOptions.find((o) => o.name === source)) {
      // set it to the first one
      source = sourceOptions[0].name;
    }
  })();
  let selectedInstruction = 0;

  let memory = Array(0xff).fill(0);
  // returns an array of arrays of indices of memory cells
  let memoryColumns = () => {
    let COL_HEIGHT = 32;
    return Array(Math.ceil(memory.length / COL_HEIGHT))
      .fill(null)
      .map((_, i) =>
        Array(COL_HEIGHT)
          .fill(0)
          .map((_, j) => i * COL_HEIGHT + j)
      );
  };

  let flashedFields: { [key: string]: string | null } = {};
  let flashField = (fieldName: string, type: string) => {
    flashedFields = { ...flashedFields, [fieldName]: type };
    setTimeout(() => {
      flashedFields = { ...flashedFields, [fieldName]: null };
    }, 300);
  };

  let executeInstruction = () => {
    instructions[selectedInstruction].execute();
  };

  $: instructionText = (() => {
    let instruction = instructions[selectedInstruction];
    let text = instruction.name;
    if (instruction.name !== "POP") {
      text += " " + source;
    }
    if (instruction.name !== "PUSH") {
      text += ", " + destination;
    }
    if (registers.DISP !== 0) {
      text = text.replace(
        "DISP",
        "0x" + registers.DISP.toString(16).padStart(2, "0")
      );
    } else {
      text = text.replace("DISP", "");
    }
    return text;
  })();
</script>

<main>
  <h1>Symulator rozkazów MOV, XCHG, PUSH, POP procesora 8086</h1>
  <p>Albert Koczy Nr indeksu: 13942</p>
  <div class="main-columns">
    <div class="main-column">
      <h3>Rejestry</h3>

      <div class="register-list">
        {#each registersOrder as registerName}
          {#if registerName === "---"}
            <div class="register-divider" />
          {:else}
            <div class="register-field">
              <span class="register-name">{registerName}</span>
              <HexInput
                bind:value={registers[registerName]}
                flash={flashedFields[registerName]}
              />
            </div>
          {/if}
        {/each}
      </div>
    </div>
    <div class="main-column">
      <h3>Instrukcja</h3>
      <RadioField
        options={copyModeOptions}
        bind:value={copyMode}
        label="Tryb działania"
      />
      {#if copyMode !== CopyMode.RegisterToRegister}
        <RadioField
          options={addressingModeOptions}
          bind:value={addressingMode}
          label="Tryb adresowania"
        />
      {/if}
      <div class="disp">
        <div>
          <label>Lokalne przemieszczenie [DISP]</label>
          <HexInput bind:value={registers.DISP} flash={flashedFields.DISP} />
        </div>
      </div>
      <!-- <pre>
		  {JSON.stringify({ copyMode, addressingMode })}
	  </pre> -->
      <div class="instruction-selector">
        <div class="select-field">
          <label for="">Instrukcja</label>
          <select bind:value={selectedInstruction}>
            {#each instructions as instruction, index}
              <option value={index} selected={index === selectedInstruction}
                >{instruction.name}</option
              >
            {/each}
          </select>
        </div>
        {#if selectedInstruction !== instructions.findIndex((i) => i.name === "POP")}
          <div class="select-field">
            <label for="" class="source">Źródło</label>
            <select bind:value={source}>
              {#each sourceOptions as operand}
                <option value={operand.name}>{operand.name}</option>
              {/each}
            </select>
          </div>
        {/if}
        {#if selectedInstruction !== instructions.findIndex((i) => i.name === "PUSH")}
          <div class="select-field">
            <label for="" class="dest">Cel</label>
            <select bind:value={destination}>
              {#each destinationOptions as operand}
                <option value={operand.name}>{operand.name}</option>
              {/each}
            </select>
          </div>
        {/if}
      </div>
	  <pre>{instructionText}</pre>
	 
      <button on:click={executeInstruction}> Wykonaj </button>
    </div>
    <div class="main-column">
      <h3>Pamięć</h3>
      <div class="memory-view">
        {#each memoryColumns() as col}
          <div class="memory-column">
            {#each col as index}
              <div class="memory-cell">
                <div class="memory-cell-index">
                  <span class="hex-0x">0x</span>{index
                    .toString(16)
                    .padStart(2, "0")
                    .toUpperCase()}
                </div>
                <HexInput
                  bind:value={memory[index]}
                  flash={flashedFields["mem_" + index]}
                />
              </div>
            {/each}
          </div>
        {/each}
      </div>
    </div>
  </div>
</main>

<style lang="less">
  main {
    font-family: Roboto, Helvetica, Arial, sans-serif;
    padding-left: 30px;
    padding-right: 30px;
  }
  .main-columns {
    display: flex;
    flex-wrap: nowrap;
    flex-direction: row;
    .main-column {
      margin-right: 1rem;
    }
  }
  .register-field {
    display: flex;
    flex-wrap: nowrap;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    margin-bottom: 8px;
    .register-name {
      width: 100px;
    }
  }
  .register-divider {
    margin: 0.5rem 0;
    border-top: 1px solid #ccc;
  }
  .memory-cell {
    display: flex;
    fler-wrap: nowrap;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    margin-left: 8px;
    margin-bottom: 4px;
  }
  .memory-view {
    display: flex;
    flex-wrap: nowrap;
    flex-direction: row;
  }
  .memory-cell-index {
    font-family: monospace;
    width: 35px;
  }
  .instruction-selector {
    display: flex;
    flex-wrap: nowrap;
    flex-direction: row;
    margin-top: 8px;
  }
  .select-field {
    margin-left: 4px;
    margin-right: 4px;
  }
  button {
    width: 100%;
    background: #fd79a8;
    color: #fff;
    border: none;
    border-radius: 4px;
  }
  .hex-0x {
    opacity: 0.7;
    font-weight: lighter;
  }
  label {
    text-transform: uppercase;
    font-weight: bold;
    font-size: 0.7rem;
    &.dest {
      color: #fd79a8;
    }
    &.source {
      color: #00b894;
    }
  }
  .disp {
    margin-top: 8px;
    label {
      margin-bottom: 4px;
    }
  }
  select {
    font-size: 0.8rem;
    width: 150px;
  }
  h1 {
    font-size: 1.5rem;
  }
</style>
