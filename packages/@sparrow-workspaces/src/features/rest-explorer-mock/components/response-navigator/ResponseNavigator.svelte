<script lang="ts">
  import { type UpdateRequestStateType } from "@sparrow/workspaces/type";
  import { ResponseSectionEnum } from "@sparrow/common/types/workspace";
  import { Navigator } from "@sparrow/library/ui";
  export let requestStateSection: string;
  export let onUpdateRequestState;
  export let responseHeadersLength = 0;

  let tabs: {
    name: string;
    id: ResponseSectionEnum;
    count: number;
  }[] = [];

  /**
   * @description - refresh tabs label count
   * @param _responseHeadersLength - response headers length
   */
  const refreshTabs = (_responseHeadersLength: number) => {
    return [
      { name: "Body", id: ResponseSectionEnum.RESPONSE, count: 0 },
      // {
      //   name: "Headers",
      //   id: ResponseSectionEnum.HEADERS,
      //   count: _responseHeadersLength,
      // },
    ];
  };

  /**
   * @description - re-calculates value when dependency changes
   */
  $: {
    if (responseHeadersLength) {
      tabs = refreshTabs(responseHeadersLength);
    }
  }

  const onTabClick = (tabId: ResponseSectionEnum) => {
    onUpdateRequestState({ responseNavigation: tabId });
  };
</script>

<Navigator {tabs} {onTabClick} currentTabId={requestStateSection} />
