<script lang="ts">
  import {
    AddRegular,
    ChevronLeftRegular,
    ChevronRightRegular,
  } from "@sparrow/library/icons";
  import { Accordion, Button } from "@sparrow/library/ui";
  import { getMethodStyle } from "@sparrow/common/utils";
  import { captureEvent } from "@app/utils/posthog/posthogConfig";

  export let requestApis: any = [];
  export let expression = "";
  export let environmentVariables: any;
  export let selectedApiRequestType: string;
  export let selectedBlock: any;
  export let dispatcher;
  export let edges = [];

  let selectedAPI: any = null;
  let hoverdIndexRequest: number | null = null;
  let hoveredVariableKey: string | null = null;

  const requestTypes = [
    { requestType: "response.body", label: "response.body" },
    { requestType: "response.headers", label: "response.headers" },
    { requestType: "request.body", label: "request.body" },
    { requestType: "request.headers", label: "request.headers" },
    { requestType: "request.parameters", label: "request.parameters" },
  ];

  const handleSelectApi = (data: any) => {
    selectedAPI = data;
  };

  const handleRemoveSelectApi = () => {
    selectedApiRequestType = "";
    selectedAPI = null;
  };

  const handleSelectVariable = (requestVariable: any) => {
    const cursorPosition = dispatcher?.state?.selection?.main?.head || 0;
    if (cursorPosition) {
      expression =
        expression.slice(0, cursorPosition) +
        `{{${requestVariable.key}}}` +
        expression.slice(cursorPosition);
      return;
    }
    expression = `${expression}{{${requestVariable.key}}}`;
  };

  function hasTwoOrMoreDots(expr: string): boolean {
    return (expr.match(/\./g) || []).length >= 2;
  }

  function hasFourParentheses(expr: string): boolean {
    const open = (expr.match(/\(/g) || []).length;
    const close = (expr.match(/\)/g) || []).length;
    return open >= 2 && close >= 2;
  }

  const handleEventOnSelectVariable = (variable: string) => {
    captureEvent("variables_selected", {
      component: "DynamicContent",
      Variable_name: variable,
    });
  };

  const handleSelectRequestType = (
    requestType: string,
    requestName: string,
  ) => {
    if (requestType === "request.body" || requestType === "response.body") {
      selectedApiRequestType = "body";
    }
    const sanitizedRequestName = requestName.replace(/[^a-zA-Z0-9_]/g, "_");
    const current = `$$${sanitizedRequestName}.${requestType}`;
    const cursorPosition = dispatcher?.state?.selection?.main?.head || 0;
    if (cursorPosition !== null && expression) {
      expression =
        expression.slice(0, cursorPosition) +
        current +
        expression.slice(cursorPosition);
      return;
    }
    if (!hasTwoOrMoreDots(expression)) {
      expression = current;
    } else {
      if (!hasFourParentheses(expression)) {
        expression = `${expression}${current}`;
        return;
      }
      expression += current;
    }
  };

  let currentApis: any[] = [];

  const findConnectedNodes = (
    adj: any[],
    start: number,
    nodes,
    result,
    visited = new Set(),
  ) => {
    if (visited.has(start)) return;

    for (let i = 0; i < nodes.length; i++) {
      if (Number(nodes[i].id) === start && nodes[i]?.data?.requestId) {
        result.push(nodes[i]);
      }
    }

    visited.add(start);

    for (const neighbor of adj[start]) {
      findConnectedNodes(adj, neighbor, nodes, result, visited);
    }
  };

  $: {
    let maxNodeId = 1;
    for (let i = 0; i < requestApis.length; i++) {
      maxNodeId = Math.max(maxNodeId, Number(requestApis[i].id));
    }

    // Initialize adjacency list
    const graph = Array.from({ length: maxNodeId + 1 }, () => []);
    // Populate adjacency list

    for (let i = 0; i < edges.length; i++) {
      graph[Number(edges[i].target)].push(Number(edges[i].source));
    }

    let result = [];
    findConnectedNodes(graph, Number(selectedBlock.id), requestApis, result);
    const res = [...result.reverse()] || [];

    if (res?.length >= 1) {
      res.pop();
      currentApis = res;
    }
  }
</script>

<div>
  <Accordion position="right">
    <div class="d-flex align-items-center ellipsis" slot="accordion-field">
      {#if selectedAPI}
        <div
          class="d-flex justify-conten-start align-items-center ellipsis"
          style="gap: 6px;"
        >
          <Button
            type="teritiary-regular"
            size="small"
            iconSize={16}
            startIcon={ChevronLeftRegular}
            onClick={handleRemoveSelectApi}
          />
          <div
            class="d-flex flex-row align-items-center justify-content-start ellipsis"
            style="gap: 6px;"
          >
            <p
              class="request-block-method text-{getMethodStyle(
                selectedAPI?.requestData?.method,
              )}"
              style="margin:0px;"
            >
              {selectedAPI?.requestData?.method}
            </p>

            <p class="de-header-name ellipsis" style="margin: 0;">
              {selectedAPI?.blockName} | {selectedAPI?.requestData?.name}
            </p>
          </div>
        </div>
      {:else}
        <p style="margin:0px;" class="de-header-name">Previous API Request</p>
      {/if}
    </div>
    <div
      class=""
      slot="accordion-content"
      style="padding: 4px 8px; height: 146px; overflow-y: auto"
    >
      {#if selectedAPI}
        <div class="" style="gap: 2px;">
          {#each requestTypes as type, index}
            <div
              class="d-flex flex-row justify-content-between align-items-start request-type-block"
              style="height:28px; border-radius:4px; padding:5px 8px; cursor: pointer; margin-bottom:4px; margin-top:4px;"
              on:mouseenter={() => (hoverdIndexRequest = index)}
              on:mouseleave={() => (hoverdIndexRequest = null)}
              on:click={() =>
                handleSelectRequestType(
                  type.requestType,
                  selectedAPI?.blockName,
                )}
            >
              <div class="d-flex justify-content-start align-items-center">
                <p style="margin: 0px;">{type.label}</p>
              </div>
              {#if hoverdIndexRequest === index}
                <AddRegular size="16px" color="var(--icon-ds-neutral-50)" />
              {/if}
            </div>
            <hr class="request-line" />
          {/each}
        </div>
      {:else if currentApis.length >= 1}
        {#each currentApis as requestApi}
          <div
            class="d-flex flex-row justify-content-between align-items-center request-api-block"
            style="height:28px; border-radius:4px; padding:5px 8px; cursor: pointer;"
            on:click={() => {
              handleSelectApi(requestApi?.data);
            }}
          >
            <div
              class="d-flex justify-content-start align-items-center ellipsis"
              style="gap: 8px;"
            >
              <p
                class="request-block-method text-{getMethodStyle(
                  requestApi?.data?.requestData?.method,
                )}"
                style="margin: 0;"
              >
                {requestApi?.data?.requestData?.method}
              </p>

              <p class="request-block-title ellipsis" style="margin: 0;">
                {requestApi?.data?.blockName} | {requestApi?.data?.requestData
                  ?.name}
              </p>
            </div>

            <Button
              type="teritiary-regular"
              size="extra-small"
              startIcon={ChevronRightRegular}
              onClick={() => {
                handleSelectApi(requestApi?.data);
              }}
            />
          </div>
        {/each}
      {:else}
        <p class="text-fs-12">No results found.</p>
      {/if}
    </div>
  </Accordion>
  <Accordion position="right">
    <div class="d-flex align-items-center" slot="accordion-field">
      <p style="margin: 0;" class="de-header-name">Variables</p>
    </div>
    <div
      slot="accordion-content"
      style="padding: 4px 8px; height: 146px; overflow-y: auto"
    >
      <p class="variable-header-title mb-1" style="font-size: 12px;">
        Dev Environment
      </p>

      {#each environmentVariables.filtered as variable (variable.key)}
        {#if variable?.environment !== "Global Variables"}
          <div
            class="d-flex justify-content-between align-items-center px-2 py-1 mb-1 request-type-block ellipsis"
            style="height: 28px; border-radius: 4px; cursor: pointer; margin-bottom:4px; margin-top:4px;"
            on:mouseenter={() => (hoveredVariableKey = variable.key)}
            on:mouseleave={() => (hoveredVariableKey = null)}
            on:click={() => {
              handleSelectVariable(variable);
            }}
          >
            <span style="font-size: 12px;" class="ellipsis"
              >{variable?.key}</span
            >
            {#if hoveredVariableKey === variable.key}
              <AddRegular size="16px" color="var(--icon-ds-neutral-50)" />
            {/if}
          </div>
          <hr class="request-line" />
        {/if}
      {/each}

      <p class="mt-2 mb-1 variable-header-title" style="font-size: 12px;">
        Global Environments
      </p>

      {#each environmentVariables.filtered as variable (variable.key)}
        {#if variable?.environment === "Global Variables"}
          <div
            class="d-flex justify-content-between align-items-center px-2 py-1 mb-1 request-type-block ellipsis"
            style="height: 28px; border-radius: 4px; cursor: pointer; margin-bottom:4px; margin-top:4px;"
            on:mouseenter={() => (hoveredVariableKey = variable.key)}
            on:mouseleave={() => (hoveredVariableKey = null)}
            on:click={() => {
              handleEventOnSelectVariable(variable);
              handleSelectVariable(variable);
            }}
          >
            <span style="font-size: 12px;" class="ellipsis"
              >{variable?.key}</span
            >
            {#if hoveredVariableKey === variable.key}
              <AddRegular size="16px" color="var(--icon-ds-neutral-50)" />
            {/if}
          </div>
          <hr class="request-line" />
        {/if}
      {/each}
    </div>
  </Accordion>
</div>

<style>
  .request-block-method {
    color: var(--text-ds-neutral-50);
    font-family: "Inter", sans-serif;
    font-weight: 600;
    font-size: 10px;
    line-height: 1;
    margin-top: 3px;
  }

  .request-block-name {
    color: var(--text-ds-neutral-50);
    font-family: "Inter", sans-serif;
    font-weight: 400;
    font-size: 14px;
    line-height: 1;
    text-align: left;
  }

  .request-block-title {
    color: var(--text-ds-neutral-50);
    font-family: "Inter", sans-serif;
    font-weight: 500;
    font-size: 12px;
    line-height: 1.3;
  }

  .request-api-block:hover {
    background-color: var(--bg-ds-surface-400);
  }
  .request-type-block:hover {
    background-color: var(--bg-ds-surface-400);
  }
  .request-type-block {
    font-family: "Inter", sans-serif;
    font-weight: 500;
    font-size: 12px;
    line-height: 130%;
    letter-spacing: 0%;
    color: var(--text-ds-neutral-50);
  }
  .request-line {
    border: 1px solid var(--border-ds-surface-400);
    margin: 0px;
  }
  .de-header-name {
    color: var(--text-ds-neutral-200);
    font-family: "Inter", sans-serif;
    font-weight: 500;
    font-size: 12px;
    line-height: 1.3;
  }
  .variable-header-title {
    font-family: "Inter", sans-serif;
    font-weight: 400;
    font-size: 12px;
    line-height: 150%;
    letter-spacing: 0%;
    vertical-align: middle;
    color: var(--text-ds-neutral-300);
  }
</style>
