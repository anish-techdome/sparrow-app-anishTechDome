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
  export let width = "398px";
  export let height = "30px";
  let fontWeight = "500";
  let fontSize = "16px";

  export let id = "";
  export let value = "";
  export let maxlength = 500;
  export let disabled = false;
  export let placeholderColor = "gray";
  export let placeholder = "Type here ...";

  let componentClass = "";
  let componentStyle = "";
  export { componentClass as class };
  export { componentStyle as style };

  export let defaultBorderColor = "transparent";
  export let hoveredBorderColor = "#9B9DA1";
  export let focusedBorderColor = "#6894F9";

  export let iconSize = "14px";
  export let isEditIconRequired = false;
  export let searchIconColor = "var(--defaultcolor)";

  let isHovered = false;
  let isFocused = false;
  export let inputFieldState: States = States.Default;
  const dispatch = createEventDispatcher();

  $: {
    if (size === Size.Small) {
      fontSize = "12px";
      fontWeight = "500";
    } else if (size === Size.Medium) {
      fontSize = "16px";
      fontWeight = "500";
    } else if (size === Size.Large) {
      fontSize = "20px";
      fontWeight = "600";
    }
  }

  $: borderStyle = extractBorderHighlight(inputFieldState);
  $: console.log("Updated border style:", borderStyle);

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
    if (
      inputFieldState !== States.Typing &&
      inputFieldState !== States.Focused
    ) {
      inputFieldState = States.Hover;
    }
  };
  const onMouseLeave = () => {
    // console.log("On Mouse Leave!");
    if (
      inputFieldState !== States.Typing &&
      inputFieldState !== States.Focused
    ) {
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

<!-- <div
  class="position-relative"
  style="min-width: 120px; max-width: 540px; width: {width}; height:{height}; !important; "
  on:mouseenter={() => {
    // console.log("On Mouse Enter!");
    if (inputFieldState !== States.Typing) {
      inputFieldState = States.Hover;
    }
    // inputFieldState = States.Hover;
    isHovered = true;
  }}
  on:mouseleave={() => {
    // console.log("On Mouse Leave!");
    if (inputFieldState !== States.Typing) {
      inputFieldState = States.Default;
    }
    inputFieldState = States.Default;
    isHovered = false;
  }}
> -->
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
  class="rounded-1 p-1 position-relative {componentClass}"
  style="
  min-width: 120px; 
  max-width: 540px; 
  width: {width}; 
  height:{height} ;
  border:{borderStyle} !important; 
  outline: none;
  --placeholder-color: {placeholderColor};
  font-weight: {fontWeight};
  font-size: {fontSize};
  {componentStyle};
    "
/>

<!-- </div> -->

<style>
  .SearchIconClass {
    display: flex;
  }
  input {
    caret-color: var(--border-primary-300);
    border: 1px solid transparent;
  }
  input::placeholder {
    color: var(--placeholder-color);
  }

  input:disabled {
    color: #ffffff !important;
  }

  .api-name-field {
    height: 24px;
    background-color: transparent;
    color: var(--bg-ds-neutral-50);
    padding: 4px 2px;
    outline: none;
    border-radius: 4px !important;
    border: 1px solid var(--bg-ds-primary-300);
    caret-color: var(--bg-ds-primary-300);
  }
  .api-name-field:focus {
    border: 1px solid var(--border-ds-primary-300) !important;
  }
</style>
