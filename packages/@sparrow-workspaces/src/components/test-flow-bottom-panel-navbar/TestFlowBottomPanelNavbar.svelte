<script>
  import { ArrowOutwardIcon, MathFormulaRegular } from "@sparrow/library/icons";
  import { Tooltip } from "@sparrow/library/ui";
  import TestFlowTourGuide from "../test-flow-tour-guide/TestFlowTourGuide.svelte";
  import { currentStep, isTestFlowTourGuideOpen } from "../../stores";
  import { Select } from "@sparrow/library/forms";
  import { Button } from "@sparrow/library/ui";
  import {
    extractQueryParams,
    httpMethodData,
  } from "../../../../@sparrow-common/src/utils/testFlow.helper";
  import HttpUrlSection from "../../features/testflow-explorer/components/http-url-section/HttpUrlSection.svelte";

  export let selectedBlock;
  export let onRedirect;
  export let handleUpdateRequestData;
  export let showRedirectButton = false;
  export let truncateName;
  export let environmentVariables;
  export let userRole;
  export let onUpdateEnvironment;
  export let handleClickTestButton;
  export let isTestFlowRuning = false;
  export let handleOpenCurrentDynamicExpression;

  let requestUrl = selectedBlock?.data?.requestData?.url ?? "";

  $: {
    if (selectedBlock) {
      requestUrl = selectedBlock?.data?.requestData?.url ?? "";
    }
  }

  let dispatcher;
</script>

<div class="header-container">
  <div class="request-name">
    <Select
      variant={"secondary"}
      id={"bottom-panel-navbar"}
      size={"medium"}
      data={httpMethodData}
      borderRounded={"4px"}
      titleId={selectedBlock?.data?.requestData?.method === ""
        ? "GET"
        : selectedBlock?.data?.requestData?.method}
      onclick={(e) => handleUpdateRequestData("method", e)}
      borderHighlight={"active"}
      headerHighlight={"hover"}
      minHeaderWidth={"110px"}
      maxHeaderWidth={"110px"}
      borderActiveType={"none"}
      zIndex={500}
      borderType={"none"}
      menuItem={"v2"}
      bodyTheme={"surface"}
      isDropIconFilled={true}
      highlightTickedItem={false}
      headerFontSize={"12px"}
      headerHeight={"36px"}
    />
    <div class="request-name-text">
      {truncateName(
        selectedBlock?.data?.requestData?.name === ""
          ? "Select API Request"
          : selectedBlock?.data?.requestData?.name,
        10,
      )}
    </div>
  </div>

  <div class="request-url gap-2">
    <HttpUrlSection
      {requestUrl}
      onUpdateRequestUrl={(e) => {
        handleUpdateRequestData("url", e);
      }}
      {onUpdateEnvironment}
      {userRole}
      {environmentVariables}
      handleOpenCurrentDynamicExpression={(obj) => {
        handleOpenCurrentDynamicExpression({
          ...obj,
          type: "url",
        });
      }}
      bind:dispatcher
    />
    <Button
      size="medium"
      type="teritiary-regular"
      startIcon={MathFormulaRegular}
      onClick={() => {
        handleOpenCurrentDynamicExpression({
          type: "url",
          dispatch: dispatcher,
        });
      }}
    />
  </div>

  <Button
    disable={isTestFlowRuning}
    title={"Test"}
    textClassProp={"fs-6"}
    type={"primary"}
    onClick={handleClickTestButton}
    loader={false}
  />

  {#if showRedirectButton}
    <div class="arrow-icon-container">
      <Tooltip title="Redirect" placement={"bottom-center"} zIndex={100}>
        <span on:click={onRedirect} class="pe-2">
          <ArrowOutwardIcon
            width={"12px"}
            height={"12px"}
            color={"var(--icon-secondary-200)"}
          />
        </span>
      </Tooltip>
    </div>
  {/if}
</div>

<style>
  .header-container {
    border-radius: 4px;
    border-top-left-radius: 0px;
    padding: 10px;
    display: flex;
    flex-direction: row;
    background-color: var(--bg-ds-surface-300);
    position: relative;
    height: 60px;
    flex-grow: inherit;
  }

  .request-name {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    background-color: var(--bg-ds-surface-600);
    border-radius: 4px;
    padding-left: 2px;
  }

  .request-name-text {
    margin-left: 12px;
    padding-left: 12px;
    padding-right: 12px;
    border-left: 1px solid var(--border-ds-neutral-500);
    color: var(--text-ds-neutral-500);
    font-size: 14px;
    line-height: 143%;
    letter-spacing: 0;
    min-width: 125px;
  }

  .request-url {
    padding: 1px;
    margin: 0px 10px;
    flex: 1;
    display: flex;
    overflow: visible;
    border-radius: 4px;
  }

  .arrow-icon-container {
    margin-left: 15px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 2px;
    cursor: pointer;
  }
</style>
