<script lang="ts">
  import type { KeyValuePair } from "@sparrow/common/interfaces/request.interface";
  import { invoke } from "@tauri-apps/api/core";
  import { TabularInputTheme } from "../../utils";
  import { CodeMirrorInput } from "..";
  import { Button, notifications, Tooltip } from "@sparrow/library/ui";
  import { afterUpdate, onMount } from "svelte";
  import { Base64Converter } from "@sparrow/common/utils";
  import { Checkbox } from "@sparrow/library/forms";
  import LazyElementFormData from "./LazyElementFormData.svelte";
  import {
    handleEventOnClickApplyUndoAI,
    handleEventonClickApplyChangesAI,
  } from "@sparrow/common/utils";
  export let keyValue: {
    key: string;
    value: string;
    checked: boolean;
    type: "text" | "file";
    base: string;
  }[];
  export let callback: (pairs: KeyValuePair[]) => void;

  export let environmentVariables;
  export let onUpdateEnvironment;
  export let isCheckBoxEditable = true;
  export let isTopHeaderRequired = true;
  export let isBulkEditRequired = false;
  export let isInputBoxEditable = true;

  export let handleOpenCurrentDynamicExpression;
  export let dynamicExpression;

  export let isWebApp = false;

  // New props for merge/diff view
  export let showMergeView = false;
  export let isMergeViewLoading = false;
  export let newModifiedPairs: {
    key: string;
    value: string;
    checked: boolean;
    type: "text" | "file";
    base: string;
  }[] = [];

  let hasChanges = false;
  let pairs = keyValue;
  let controller: boolean = false;
  let pairsContainer: HTMLElement;

  // This is a flag to determine if we should scroll to the bottom
  let shouldScrollToBottom = false;

  // This is for scrolling into view when a new row is added and the component updates
  afterUpdate(() => {
    if (shouldScrollToBottom && pairsContainer) {
      const lastRow = pairsContainer.lastElementChild;
      if (lastRow) {
        lastRow.scrollIntoView({ behavior: "auto", block: "end" });
      }
      shouldScrollToBottom = false;
    }
  });

  const theme = new TabularInputTheme().build();

  $: {
    if (keyValue) {
      identifySelectAllState();
    }
  }

  // ************ Diff/Merge View Methods - Start ************

  // Calculate diff between original and current data
  type DiffType = "added" | "deleted" | "modified" | "unchanged";
  type DiffPair = {
    key: string;
    value: string;
    checked: boolean;
    type: "text" | "file";
    base: string;
    diffType: DiffType;
    originalIndex?: number;
    currentIndex?: number;
  };
  let diffPairs: DiffPair[] = [];

  // Function to calculate diff between original data (pairs) and new data (newModifiedPairs)
  function calculateDiff(): DiffPair[] {
    // If table data is empty but we have new data, then consider everything as an addition
    // if (!pairs || pairs.length === 0) {
    //   return newModifiedPairs.map((pair) => ({
    //     ...pair,
    //     diffType: "added",
    //     currentIndex: newModifiedPairs.indexOf(pair),
    //   }));
    // }

    const result: DiffPair[] = [];
    const origMap = new Map();

    // Create a map of original items using key as a unique identifier
    pairs.forEach((pair, index) => {
      origMap.set(pair.key, { pair, index });
    });

    // Check for modified or added items
    newModifiedPairs.forEach((currentPair, currentIndex) => {
      if (currentPair.key === "") return; // Skip empty rows

      const originalEntry = origMap.get(currentPair.key);

      if (originalEntry) {
        // Key exists in both - check if value changed
        if (
          originalEntry.pair.value !== currentPair.value
          // || // type and base should also be compared, disabling it temporaraliy
          // originalEntry.pair.type !== currentPair.type ||
          // originalEntry.pair.base !== currentPair.base
        ) {
          result.push({
            ...currentPair,
            diffType: "modified",
            originalIndex: originalEntry.index,
            currentIndex,
          });

          // Also add the original version to show what changed
          result.push({
            ...originalEntry.pair,
            diffType: "deleted",
            originalIndex: originalEntry.index,
          });
        } else {
          // Unchanged
          result.push({
            ...currentPair,
            diffType: "unchanged",
            originalIndex: originalEntry.index,
            currentIndex,
          });
        }

        // Mark as processed
        origMap.delete(currentPair.key);
      } else {
        // New item
        result.push({
          ...currentPair,
          diffType: "added",
          currentIndex,
        });
      }
    });

    // Add deleted items (those in original but not in current)
    origMap.forEach(({ pair, index }) => {
      if (pair.key !== "") {
        // Skip empty rows
        result.push({
          ...pair,
          diffType: "deleted",
          originalIndex: index,
        });
      }
    });

    // Comparator: Custom Sorting Method for Sorting by original position to maintain a logical order
    result.sort((a, b) => {
      // First priority: Group items with the same key together
      if (a.key === b.key) {
        // When keys match, deleted comes before added
        if (
          a.diffType === "deleted" &&
          (b.diffType === "added" || b.diffType === "modified")
        ) {
          return -1; // Deletion comes first
        }
        if (
          (a.diffType === "added" || a.diffType === "modified") &&
          b.diffType === "deleted"
        ) {
          return 1; // Addition comes second
        }
      }

      // Second priority: Put completely new items at the end
      const aIsNew = a.diffType === "added" && a.originalIndex === undefined;
      const bIsNew = b.diffType === "added" && b.originalIndex === undefined;

      if (aIsNew && !bIsNew) {
        return 1; // a is new but b is not - push a to the end
      } else if (!aIsNew && bIsNew) {
        return -1; // b is new but a is not - push b to the end
      }

      // Then sort by original position for existing items
      if (a.originalIndex !== undefined && b.originalIndex !== undefined) {
        return a.originalIndex - b.originalIndex;
      }

      // Fall back to current position
      if (a.currentIndex !== undefined && b.currentIndex !== undefined) {
        return a.currentIndex - b.currentIndex;
      }

      return 0;
    });

    const lastEmptyRow: DiffPair = {
      key: "",
      value: "",
      checked: false,
      type: "text",
      base: "",
      diffType: "unchanged",
      originalIndex: result.length - 1,
      currentIndex: result.length - 1,
    };

    return [...result, lastEmptyRow];
  }

  /**
   * Checks if there are any actual changes between the original data and the new data
   * Sets hasChanges to true if there are added, deleted, or modified items
   */
  function checkForChanges(): boolean {
    if (!diffPairs || diffPairs.length === 0) return false;

    // Check if there are any changes (added, deleted, or modified items)
    const changes = diffPairs.filter(
      (pair) =>
        pair.diffType === "added" ||
        pair.diffType === "deleted" ||
        pair.diffType === "modified",
    );

    // Update hasChanges state
    hasChanges = changes.length > 0;

    if (!hasChanges) {
      undoChanges(); // resetting the mergeview states and props
      notifications.success("You already have updated changes.");
    }
    return hasChanges;
  }

  const updateDiffPairsWithLoading = async () => {
    isMergeViewLoading = true;
    await sleep(2000);
    diffPairs = calculateDiff();
    checkForChanges();
    isMergeViewLoading = false;
  };
  $: if (showMergeView) updateDiffPairsWithLoading();

  // Toggle merge view
  const toggleMergeView = async (show: boolean) => {
    showMergeView = show;
    if (show) {
      isMergeViewLoading = true;
      await sleep(2000);
      diffPairs = calculateDiff();
      checkForChanges();
    } else hasChanges = false;
  };

  // Function to apply all changes from diff view to the original data
  const applyChanges = async () => {
    if (!showMergeView) return;
    isMergeViewLoading = true;
    // Extract all valid pairs from diffPairs (excluding deleted ones)
    const updatedPairs = diffPairs
      .filter((pair) => pair.diffType !== "deleted")
      .map((pair) => ({
        key: pair.key,
        value: pair.value,
        checked: pair.checked || false,
        type: pair.type || "text",
        base: pair.base || "",
      }));

    // Add an empty row at the end if needed
    if (
      updatedPairs.length > 0 &&
      (updatedPairs[updatedPairs.length - 1].key !== "" ||
        updatedPairs[updatedPairs.length - 1].value !== "")
    ) {
      updatedPairs.push({
        key: "",
        value: "",
        checked: false,
        type: "text",
        base: "",
      });
    }

    pairs = updatedPairs; // Update the pairs array with the new data
    showMergeView = false; // Turn off merge view after applying changes
    callback(pairs); // Notify parent component of the changes
    newModifiedPairs = [];
    hasChanges = false;

    await sleep(2000);
    isMergeViewLoading = false; // Reset loading state
    handleEventonClickApplyChangesAI("TabularInputFormData", "Body-FormData");
  };

  // Function to undo all changes and revert to original state
  const undoChanges = async () => {
    if (!showMergeView) return;
    showMergeView = false;
    newModifiedPairs = [];
    diffPairs = []; // Reset any potential changes by discarding diffPairs
    callback(pairs); // Notify parent of unchanged data
    hasChanges = false;

    isMergeViewLoading = true;
    await sleep(2000);
    isMergeViewLoading = false; // Reset loading state
    handleEventOnClickApplyUndoAI("TabularInputFormData", "Body-FormData");
  };

  // Utility function to create a delay
  const sleep = (ms: number): Promise<void> => {
    return new Promise((resolve) => setTimeout(resolve, ms));
  };

  // ************ Diff/Merge View Methods - Ends ************

  /**
   * @description - calculates the select all checkbox state - weather checked or not
   */
  const identifySelectAllState = () => {
    pairs = [];
    pairs = keyValue;
    controller = false;
    if (pairs.length > 1) {
      let isUncheckedExist: boolean = false;
      for (let i = 0; i < pairs.length - 1; i++) {
        if (pairs[i].checked === false) {
          isUncheckedExist = true;
          break;
        }
      }
      if (isUncheckedExist) {
        controller = false;
      } else {
        controller = true;
      }
    }
  };

  const updateParam = async (index: number): Promise<void> => {
    pairs = pairs;
    if (
      pairs.length - 1 === index &&
      isInputBoxEditable &&
      (pairs[index].key !== "" || pairs[index].value !== "")
    ) {
      pairs[pairs.length - 1].checked = true;
      pairs.push({
        key: "",
        value: "",
        checked: false,
        type: "text",
        base: "",
      });
      shouldScrollToBottom = true;
      pairs = pairs;
      callback(pairs);

      // Recalculate diff if merge view is active
      if (showMergeView) {
        diffPairs = calculateDiff();
        checkForChanges();
      }
    } else {
      callback(pairs);
    }
  };

  const deleteParam = (index: number): void => {
    if (pairs.length > 1) {
      let filteredKeyValue = pairs.filter((elem, i) => {
        if (i !== index) {
          return true;
        }
        return false;
      });
      pairs = filteredKeyValue;
    }
    callback(pairs);
    setTimeout(() => {
      pairs[pairs.length - 1].key = "";
      pairs[pairs.length - 1].value = "";
    }, 0);
  };

  const updateCheck = (index: number): void => {
    let filteredKeyValue = pairs.map((elem, i) => {
      if (i === index) {
        elem.checked = !elem.checked;
      }
      return elem;
    });
    pairs = filteredKeyValue;
    callback(pairs);
  };

  const extractFileName = (url: string) => {
    const parts = url.split("\\");
    const fileName = parts[parts.length - 1];
    return fileName;
  };

  let fileInput: HTMLInputElement;
  let selectedFileIndex = 0;
  const uploadFormFile = async (index: number) => {
    if (isWebApp) {
      selectedFileIndex = index;
      fileInput?.click();
      return;
    }
    const filePathResponse = (await invoke("fetch_file_command")) as string;
    if (filePathResponse !== "Canceled") {
      const filename = extractFileName(filePathResponse);
      const updatedFilePath = filePathResponse;
      let filteredPair = pairs.map((elem, i) => {
        if (i == index) {
          elem.type = "file";
          elem.value = filename;
          elem.base = updatedFilePath;
        }
        return elem;
      });
      pairs = filteredPair;
      callback(pairs);
      updateParam(index);
    }
  };

  onMount(() => {
    fileInput = document.getElementById("fileInput") as HTMLInputElement;
    fileInput?.addEventListener("change", async (event: Event) => {
      const target = event.target as HTMLInputElement;
      const files = target.files;
      if (files?.length) {
        const filename = files[0].name;
        const base64Converter = new Base64Converter();
        const data = await base64Converter.fileToBase64(files[0]);
        let filteredPair = pairs.map((elem, i) => {
          if (i == selectedFileIndex) {
            elem.type = "file";
            elem.value = filename;
            elem.base = data;
          }
          return elem;
        });
        pairs = filteredPair;
        callback(pairs);
        updateParam(selectedFileIndex);
      }
      // Reset the input to allow re-uploading the same file
      target.value = "";
    });

    if (showMergeView) {
      diffPairs = calculateDiff();
      checkForChanges();
    }
  });

  const removeFormFile = (index: number) => {
    let filteredPair = pairs.map((elem, i) => {
      if (i == index) {
        elem.value = "";
        elem.base = "";
        elem.type = "text";
      }
      return elem;
    });
    pairs = filteredPair;
    callback(pairs);
  };

  const handleCheckAll = (): void => {
    let flag: boolean;
    if (controller === true) {
      flag = false;
    } else {
      flag = true;
    }
    let filteredKeyValue = pairs.map((elem, i) => {
      if (i !== pairs.length - 1) {
        elem.checked = flag;
      } else if (!isInputBoxEditable) {
        elem.checked = flag;
      }
      return elem;
    });
    pairs = filteredKeyValue;
    callback(pairs);
  };
</script>

<input type="file" id="fileInput" style="display: none" />

<div
  class="mb-0 me-0 py-0 section-layout w-100"
  style="overflow:visible; border-radius:4px;"
>
  <div
    class="w-100 d-flex align-items-center pair-header-row {!isTopHeaderRequired
      ? 'd-none'
      : ''}"
    style="position:relative; "
  >
    <div style=" width:24px; margin-right:12px;" class="">
      <Checkbox
        disabled={pairs.length === 1 || !isCheckBoxEditable}
        checked={controller}
        on:input={handleCheckAll}
      />
    </div>

    <div class="d-flex gap-0" style="width: calc(100% - 180px);">
      <div
        class="w-50 position-relative header-text"
        style="padding-left: 6px;"
      >
        Key
      </div>
      <div
        class="w-50 position-relative header-text"
        style="padding-left: 62px;"
      >
        Value
      </div>
    </div>
    <div style="width:140px;" class="ms-3 d-flex align-items-center">
      <div class="w-100 d-flex">
        <div class="w-100 d-flex justify-content-end">
          <button
            class="bg-transparent border-0 mt-1 d-flex {!isBulkEditRequired
              ? 'invisible'
              : ''}"
            style=""
          >
            <p
              class="text-nowrap text-primary-300 mb-0 me-0"
              style="font-size: 10px; font-weight:400;"
            >
              Bulk Edit
            </p>
          </button>
        </div>
      </div>
    </div>
  </div>
  <div
    class="w-100"
    style="display:block; position:relative;"
    bind:this={pairsContainer}
  >
    <!-- Showing Duplicate Fake Rows For Diff/Merge View -->
    {#if !isMergeViewLoading && showMergeView && hasChanges}
      {#each diffPairs as element, index (index)}
        <LazyElementFormData
          element={{
            key: element.key,
            value: element.value,
            checked: element.checked,
            type: element.type,
            base: element.base,
            diffType: element.diffType,
          }}
          {index}
          pairs={diffPairs}
          {theme}
          {environmentVariables}
          {onUpdateEnvironment}
          {updateParam}
          {updateCheck}
          {deleteParam}
          isInputBoxEditable={false}
          isCheckBoxEditable={true}
          customClass={`diff-row diff-${element.diffType}`}
          {uploadFormFile}
          {removeFormFile}
          {handleOpenCurrentDynamicExpression}
          {dynamicExpression}
        />
      {/each}

      <!-- Diff view action buttons -->
      <div
        class="d-flex justify-content-end mt-3 me-1 gap-2 merge-view-act-btns"
      >
        <Button
          title="Keep the Changes"
          size={"small"}
          type="primary"
          onClick={applyChanges}
        ></Button>
        <Button
          title="Undo"
          size={"small"}
          type="secondary"
          onClick={undoChanges}
        ></Button>
      </div>
    {:else}
      {#each pairs as element, index}
        <LazyElementFormData
          {element}
          {index}
          {pairs}
          {theme}
          {environmentVariables}
          {onUpdateEnvironment}
          {isInputBoxEditable}
          {isCheckBoxEditable}
          {updateParam}
          {updateCheck}
          {deleteParam}
          {uploadFormFile}
          {removeFormFile}
          {handleOpenCurrentDynamicExpression}
          {dynamicExpression}
        />
      {/each}
    {/if}
  </div>
</div>

<style>
  .pair-header-row {
    padding-top: 3px;
    padding-bottom: 3px;
    background-color: var(--bg-ds-surface-400);
    height: 28px;
    padding-left: 4px;
    padding-right: 1rem;
  }
  .header-text {
    color: var(--text-ds-neutral-200);
    font-family: "Inter", sans-serif;
    font-weight: 500;
    font-size: 12px;
  }
  .pair-data-row:hover .button-container {
    opacity: 1;
    visibility: visible;
  }
  .button-container {
    opacity: 0;
    visibility: hidden;
    transition:
      opacity 0.1s ease-in-out,
      visibility 0.1s;
  }

  .merge-view-act-btns {
    position: sticky;
    bottom: 4px;
  }
</style>
