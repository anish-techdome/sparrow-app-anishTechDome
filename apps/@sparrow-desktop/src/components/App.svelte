<script lang="ts">
  import { Router, Route } from "svelte-navigator";
  import "font-awesome/css/font-awesome.css";
  import { Toast } from "@sparrow/library/ui";
  import Authguard from "../routing/Authguard.svelte";
  import Navigate from "../routing/Navigate.svelte";
  import Dashboard from "@app/pages/dashboard-page/Dashboard.svelte";
  import EntryPoint from "@app/pages/auth-page/Auth.svelte";
  import { maximizeWindow } from "../utils";
  import { onMount } from "svelte";
  import { user } from "@app/store/auth.store";
  import { handleShortcuts } from "@app/utils/shortcuts";
  import { AppUpdater } from "@sparrow/common/features";
  import { getCurrentWindow } from "@tauri-apps/api/window";
  import { singleInstanceHandler } from "@app/utils/singleinstance/app.singleinstance";
  import { AppViewModel } from "./app.ViewModel";
  import { initPostHog } from "@app/utils/posthog/posthogConfig";
  import { getScaleFactor, setScaleFactorToDb } from "@app/utils/zoom";
  import { listen } from "@tauri-apps/api/event";
  import { invoke } from "@tauri-apps/api/core";
  import { platform } from "@tauri-apps/plugin-os";

  const _viewModel = new AppViewModel();

  export let url = "/";
  let isActiveInternet: boolean = true;

  const doOnlineCheck = () => {
    if (!navigator.onLine && isActiveInternet) {
      isActiveInternet = false;
      // notifications.error("The network connection has been lost. Please check your internet and try again. ");
    } else isActiveInternet = true;
  };

  // Hide Toolbar when window is restored from maximized state
  async function setupMaximizeToggleListenerForMac() {
    if (platform() === "macos") {
      listen("tauri://resize", async () => {
        const isMaximized = await getCurrentWindow().isFullscreen();
        if (!isMaximized) {
          await invoke("hide_toolbar");
        }
      });
    }
  }

  onMount(async () => {
    if (typeof window !== "undefined") {
      initPostHog();
    }
    setupMaximizeToggleListenerForMac();
    await _viewModel.registerDeepLinkHandler();
    await singleInstanceHandler();
    await setScaleFactorToDb(await getScaleFactor());
    let isloggedIn;
    user.subscribe((value) => {
      isloggedIn = value;
    });

    window.addEventListener(
      "dragover",
      function (e) {
        e = e || event;
        e.preventDefault();
      },
      false,
    );
    window.addEventListener(
      "drop",
      function (e) {
        e = e || event;
        e.preventDefault();
      },
      false,
    );
    setInterval(() => {
      doOnlineCheck();
    }, 5000);
  });
  window.addEventListener("contextmenu", (e) => {
    e.preventDefault();
  });
</script>

<AppUpdater />
<Router {url}>
  <Authguard>
    <section slot="loggedIn">
      <Route path="/app/*" component={Dashboard} />
      <Route path="/*"><Navigate to="/app/" /></Route>
    </section>
    <section slot="guestUser">
      <Route path="/guest/*" component={Dashboard} />
      <Route path="/*"><Navigate to="/guest/" /></Route>
    </section>
    <section slot="unauthorized">
      <Route path="/init" component={EntryPoint} />
      <Route path="/init/*" component={Dashboard} />
      <Route path="/*"><Navigate to="/init" /></Route>
    </section>
  </Authguard>
</Router>

<Toast />
<svelte:window on:keydown={handleShortcuts} on:keyup={handleShortcuts} />

<style>
</style>
