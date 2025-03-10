<script lang="ts">
  import { SearchIcon } from "@sparrow/library/assets";
  import { PencilIcon } from "@sparrow/library/icons";
  import { createEventDispatcher } from "svelte";

  // Types Declarations
  //   type Size = "small" | "medium" | "large";
  enum Size {
    Small = "small",
    Medium = "medium",
    Large = "large",
  }

  enum States {
    Default = "default",
    Hover = "hover",
    Typing = "typing",
    Entered = "entered", // Entered and Default is almost similar
    Focused = "focused",
  }

  export let size: Size = Size.Medium;
  export let type: "text" | "password" | "search" = "text";

  let inputElement: HTMLInputElement; // Reference to the input field
  let width = "398px";
  let height = "40px";

  export let id = "";
  export let value = "";
  export let maxlength = 500;
  export let disabled = false;
  export let placeholder = "Type here ...";

  let componentClass = "";
  let componentStyle = "";
  export { componentClass as class };
  export { componentStyle as style };

  export let defaultBorderColor = "transparent";
  export let hoveredBorderColor = "#9B9DA1";
  export let focusedBorderColor = "#6894F9";

  export let inputFieldState: States = States.Default;
  const dispatch = createEventDispatcher();

  $: {
    if (size === Size.Small) {
    } else if (size === Size.Medium) {
    } else if (size === Size.Large) {
    }
  }

  $: borderStyle = extractBorderHighlight(inputFieldState);
  // $: console.log("Updated border style:", borderStyle);

  // When the state changes to Typing, focus the input field
  $: {
    if (inputFieldState === States.Typing && inputElement) {
      // console.log("Current Input Value: ", inputElement.value); // ✅ Logs the current value
      inputElement.focus();
      inputFieldState = States.Typing;
    }
  }

  const extractBorderHighlight = (state: States) => {
    // console.log("Extracting the state!! ", state);
    let borderSize = "1px";
    let color = defaultBorderColor;

    switch (state) {
      case States.Hover:
        color = hoveredBorderColor;
        break;
      case States.Typing:
        color = focusedBorderColor;
        break;
      case States.Focused:
        borderSize = "2px";
        color = focusedBorderColor;
        break;
      default:
        color = defaultBorderColor;
    }

    // console.log("Updated border:", `${borderSize} solid ${color}`);
    return `${borderSize} solid ${color}`;
  };

  const onMouseEnter = () => {
    // console.log("input mouse enter !!!");
    if (![States.Typing, States.Focused].includes(inputFieldState)) {
      inputFieldState = States.Hover;
    }
  };
  const onMouseLeave = () => {
    // console.log("On Mouse Leave!");
    if (![States.Typing, States.Focused].includes(inputFieldState)) {
      inputFieldState = States.Default;
    }
  };
  const onFocus = (event) => {
    // console.log("On Focus !! State after:", inputFieldState);
    inputFieldState = States.Focused;
    dispatch("focused", event?.target?.value);
  };
  const onBlur = (event) => {
    // console.log("On Blur !! State after:", inputFieldState);
    inputFieldState = States.Default;
    dispatch("blur", event?.target?.value);
  };
  const onInput = (event) => {
    // console.log("On Blur !! State after:", inputFieldState);
    inputFieldState = States.Typing;
    value = event?.target?.value;
    dispatch("input", event?.target?.value);
  };
  const onKeyPress = (event: KeyboardEvent) => {
    if (event.key === "Enter") {
      // console.log("Enter click !!");
      inputFieldState = States.Entered;
      inputElement?.blur(); // Trigger onBlur event
    }
  };
</script>

<input
  bind:this={inputElement}
  {id}
  {type}
  {value}
  {disabled}
  {maxlength}
  {placeholder}
  on:mouseenter={onMouseEnter}
  on:mouseleave={onMouseLeave}
  on:focus={onFocus}
  on:blur={onBlur}
  on:input={onInput}
  on:keydown={onKeyPress}
  class="input-{size} position-relative bg-transparent ellipsis {componentClass}"
  style="
        border:{borderStyle} !important; 
        {componentStyle};
      "
/>

<style>
  input {
    min-width: 120px;
    max-width: 540px;
    width: 398px;
    border-radius: 4px !important;
    outline: none;
    font-family: "Inter", sans-serif;
    caret-color: var(--border-primary-300);
  }
  input::placeholder {
    color: gray;
  }
  input:disabled {
    color: #ffffff !important;
  }

  .input-small {
    height: 24px;
    font-size: 12px;
    font-weight: 500;
    padding: 8px 4px;
  }
  .input-medium {
    height: 40px;
    font-size: 16px;
    font-weight: 500;
    padding: 8px;
  }
  .input-large {
    height: 40px;
    font-size: 20px;
    font-weight: 600;
    padding: 8px;
  }
</style>
