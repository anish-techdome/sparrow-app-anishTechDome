<script lang="ts">
  import { GraphqlRequestSectionTabEnum } from "@sparrow/common/types/workspace/graphql-request-tab";
  import { Navigator } from "@sparrow/library/ui";
  import { RefreshSchemaIcon } from "@sparrow/library/icons";
  export let requestStateSection: string;
  export let authHeaderLength = 0;
  export let headersLength = 0;
  export let autoGeneratedHeadersLength = 0;
  export let onUpdateRequestState;
  export let isSchemaFetching = false;
  export let onRefreshSchema: () => void = () => {};

  let tabs: {
    name: string;
    id: GraphqlRequestSectionTabEnum;
    count: number;
  }[] = [];

  /**
   * @description - refresh tabs label count
   * @param _headersLength - headers length
   * @param _autoGeneratedHeadersLength - auto generated headers length
   * @param _authHeaderLength - auth headers length
   */
  const refreshTabs = (
    _headersLength: number,
    _autoGeneratedHeadersLength: number,
    _authHeaderLength: number,
  ) => {
    return [
      {
        name: "Query",
        id: GraphqlRequestSectionTabEnum.QUERY,
        count: 0,
      },
      {
        name: "Variables",
        id: GraphqlRequestSectionTabEnum.VARIABLES,
        count: 0,
      },
      {
        name: "Headers",
        id: GraphqlRequestSectionTabEnum.HEADERS,
        count:
          _headersLength + _autoGeneratedHeadersLength + _authHeaderLength - 1,
      },
      {
        name: "Auth",
        id: GraphqlRequestSectionTabEnum.AUTHORIZATION,
        count: 0,
      },
    ];
  };

  /**
   * @description - re-calculates value when dependency changes
   */
  $: {
    if (authHeaderLength || headersLength || autoGeneratedHeadersLength) {
      tabs = refreshTabs(
        headersLength,
        autoGeneratedHeadersLength,
        authHeaderLength,
      );
    }
  }

  /**
   * @description - handles different key press
   * @param event - keyboard events
   */
  const handleKeyPress = (event: KeyboardEvent) => {
    if (event.altKey && event.code === "KeyQ") {
      onUpdateRequestState({
        requestNavigation: GraphqlRequestSectionTabEnum.QUERY,
      });
    } else if (event.altKey && event.code === "KeyH") {
      onUpdateRequestState({
        requestNavigation: GraphqlRequestSectionTabEnum.HEADERS,
      });
    } else if (event.altKey && event.code === "KeyV") {
      onUpdateRequestState({
        requestNavigation: GraphqlRequestSectionTabEnum.VARIABLES,
      });
    } else if (event.altKey && event.code === "KeyA") {
      onUpdateRequestState({
        requestNavigation: GraphqlRequestSectionTabEnum.AUTHORIZATION,
      });
    }
  };

  const onTabClick = (tabId: GraphqlRequestSectionTabEnum) => {
    onUpdateRequestState({ requestNavigation: tabId });
  };
</script>

<div class="d-flex" style="justify-content: space-between; margin-bottom:12px;">
  <Navigator {tabs} {onTabClick} currentTabId={requestStateSection} />
  <div>
    <button
      class="input-cleaner px-2 border-radius-2 py-1 text-fs-12 text-secondary-200"
      on:click={async () => {
        onRefreshSchema();
      }}
      disabled={isSchemaFetching}
    >
      <span class="me-2 {isSchemaFetching ? 'rotating' : ''}">
        <RefreshSchemaIcon
          height={"14px"}
          width={"12px"}
          color={"var(--text-secondary-200)"}
        />
      </span>
      Schema</button
    >
  </div>
</div>

<svelte:window on:keydown={handleKeyPress} />

<style>
  .input-cleaner {
    cursor: pointer;
    background-color: transparent;
    border: none;
    outline: none;
  }

  .input-cleaner:disabled {
    cursor: not-allowed;
  }

  .input-cleaner:hover:not(:disabled) {
    background-color: var(--bg-tertiary-400);
  }

  /* Add rotation animation */
  @keyframes rotate {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }

  /* Rotating class */
  .rotating {
    animation: rotate 1s linear infinite;
    display: inline-block;
  }
</style>
