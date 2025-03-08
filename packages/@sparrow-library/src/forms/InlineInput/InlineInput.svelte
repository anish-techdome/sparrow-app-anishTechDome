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
  export let placeholder = "placeholder";

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
  // $: console.log(" fffKK ", borderStyle);

  // When the state changes to Typing, focus the input field
  $: {
    if (inputFieldState === States.Typing && inputElement) {
      console.log("Current Input Value: ", inputElement.value); // ✅ Logs the current value
      inputElement.focus();
      inputFieldState = States.Typing;
    }
  }

  const extractBorderHighlight = (state: States) => {
    // console.log("Extracting the state!!");
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

  /**
   * blur input on Enter key press
   * @param event - keyboard event
   */
  const onKeyPress = (event: KeyboardEvent) => {
    if (event.key === "Enter") {
      console.log("Enter clicker");
      inputFieldState = States.Entered;
      inputElement?.blur();
    }
  };
</script>

<div
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
>
  <input
    bind:this={inputElement}
    {id}
    on:focus={() => {
      console.log("On Focus !!");
      inputFieldState = States.Focused;
      isFocused = true;
      dispatch("focused", event?.target?.value);
    }}
    on:blur={() => {
      console.log("On Blur !!");
      // inputFieldState = States.Entered;
      inputFieldState = States.Default;
      isFocused = false;
      dispatch("blur", event?.target?.value);
    }}
    {value}
    on:input={(event) => {
      console.log("On Input !!");
      inputFieldState = States.Typing;
      value = event?.target?.value;
      dispatch("input", event?.target?.value);
    }}
    on:keydown={onKeyPress}
    type="text"
    {maxlength}
    class=" w-100 {componentClass}"
    {placeholder}
    style=" {componentStyle} height: 100%; {type === 'search'
      ? `padding-left:${height} !important;`
      : ''} {type === 'text' && isEditIconRequired && isHovered
      ? 'padding-right:35px !important;'
      : ''} border:{borderStyle}; --placeholder-color: {placeholderColor};
      font-weight: {fontWeight};
      font-size: {fontSize}
      "
    {disabled}
  />

  {#if type === "search"}
    <span
      class="position-absolute d-flex align-items-center justify-content-center m-0 p-0"
      style="top: 0; left: 0; bottom: 0; width: {height}; "
    >
      <span class="SearchIconClass" style="margin-top:1px;">
        <SearchIcon
          height={iconSize}
          width={iconSize}
          color={searchIconColor}
        />
      </span>
    </span>
  {/if}
  {#if type === "text" && isHovered && isEditIconRequired && !disabled}
    <span class="position-absolute" style="top:2px; right: 10px">
      <PencilIcon height={iconSize} width={iconSize} color={"white"} />
    </span>
  {/if}
</div>

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
</style>
