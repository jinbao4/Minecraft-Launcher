<script lang="ts">
  import { invoke } from '@tauri-apps/api/core';
  import { listen } from '@tauri-apps/api/event';
  import { onMount } from 'svelte';
  import Layout from './Layout.svelte';

  interface Account {
    uuid: string;
    name: string;
    mc_token: string;
    refresh_token: string;
    expires_at: number;
  }

  let instances: string[] = [];
  let status: string = "Idle";
  let isLoggingIn: boolean = false;
  let account: Account | null = null;
  let availableVersions: string[] = ["1.20.1", "1.19.4", "1.18.2"];
  let selectedVersion: string = "1.20.1";

  onMount(async () => {
    loadInstances();

    const savedJson = localStorage.getItem("mc_account");
    if (savedJson) {
      try {
        const savedAccount: Account = JSON.parse(savedJson);
        status = `Refreshing session for ${savedAccount.name}...`;
        isLoggingIn = true;
        
        account = await invoke<Account>('refresh_login', { 
          refreshToken: savedAccount.refresh_token 
        });
        
        localStorage.setItem("mc_account", JSON.stringify(account));
        status = `Welcome back, ${account.name}`;
      } catch (error) {
        console.error("Auto-login failed:", error);
        status = "Session expired. Please login again.";
        localStorage.removeItem("mc_account");
        account = null;
      } finally {
        isLoggingIn = false;
      }
    }

    await listen("login-success", (event) => {
      account = event.payload as Account;
      localStorage.setItem("mc_account", JSON.stringify(account));
      status = `Logged in as ${account.name}`;
      isLoggingIn = false;
    });

    await listen("login-error", (event) => {
      status = `Login Failed: ${event.payload}`;
      isLoggingIn = false;
    });

    await listen('install-status', (event) => {
      status = event.payload as string;
    });

    await listen('install-error', (event) => {
      status = `Error: ${event.payload}`;
    });
  });

  async function loadInstances(): Promise<void> {
    try {
      instances = await invoke<string[]>('list_instances');
    } catch (error) {
      console.error("Failed to load instances:", error);
      instances = [];
    }
  }

  async function handleLogin(): Promise<void> {
    isLoggingIn = true;
    status = "Opening Microsoft Login...";
    try {
      await invoke('start_login');
    } catch (error) {
      console.error("Login error:", error);
      status = "Failed to open login window";
      isLoggingIn = false;
    }
  }

  function handleLogout(): void {
    account = null;
    localStorage.removeItem("mc_account");
    status = "Logged out.";
  }

  async function loginAndLaunch(name: string): Promise<void> {
    if (!account) {
      await handleLogin();
      return;
    }

    try {
      status = `Launching ${name}...`;
      await invoke('launch_instance', { 
        instanceName: name,
        uuid: account.uuid,
        name: account.name,
        accessToken: account.mc_token
      });
    } catch (error) {
      console.error("Launch error:", error);
      status = `Launch Error: ${error}`;
    }
  }

  async function install(): Promise<void> {
    if (!selectedVersion) return;
    
    const newInstanceName = `Instance-${selectedVersion}`;
    status = `Installing Minecraft ${selectedVersion}...`;
    
    try {
      await invoke('install_instance', { 
        instanceName: newInstanceName,
        mcVersion: selectedVersion 
      });
      setTimeout(loadInstances, 1000);
    } catch (error) {
      console.error("Install error:", error);
      status = `Install Error: ${error}`;
    }
  }
</script>

<Layout>
  <div class="flex justify-between items-center mb-4">
    <div class="flex-1">
      <p>Status: <b>{status}</b></p>
    </div>
    
    {#if account}
      <div class="flex items-center gap-2">
        <img 
          src="https://mc-heads.net/avatar/{account.name}" 
          alt="Avatar for {account.name}" 
          width="32" 
          height="32"
          class="rounded-full" 
        />
        <div class="flex flex-col items-end">
          <span>{account.name}</span>
          <button class="rounded-lg border border-transparent px-[1.2em] py-[0.6em] text-base font-medium font-inherit bg-primary text-primary-foreground cursor-pointer transition-opacity duration-250 hover:opacity-90 disabled:opacity-50 disabled:cursor-not-allowed" on:click={handleLogout}>Log Out</button>
        </div>
      </div>
    {:else}
      <button class="rounded-lg border border-transparent px-[1.2em] py-[0.6em] text-base font-medium font-inherit bg-primary text-primary-foreground cursor-pointer transition-opacity duration-250 hover:opacity-90 disabled:opacity-50 disabled:cursor-not-allowed" on:click={handleLogin} disabled={isLoggingIn}>
        {isLoggingIn ? "Authenticating..." : "Login with Microsoft"}
      </button>
    {/if}
  </div>

  <hr />

  <div class="my-8">
    <h3>Create New Instance</h3>
    <div class="flex items-center justify-center gap-4">
      <select bind:value={selectedVersion} aria-label="Minecraft version" class="rounded-lg border border-border bg-background text-foreground px-[1.2em] py-[0.6em]">
        {#each availableVersions as version}
          <option value={version}>{version}</option>
        {/each}
      </select>
      <button class="rounded-lg border border-transparent px-[1.2em] py-[0.6em] text-base font-medium font-inherit bg-primary text-primary-foreground cursor-pointer transition-opacity duration-250 hover:opacity-90 disabled:opacity-50 disabled:cursor-not-allowed" on:click={install} disabled={isLoggingIn}>
        Install {selectedVersion}
      </button>
    </div>
  </div>

  <hr />

  <h3>My Instances</h3>
  <ul class="m-0 flex list-none flex-col gap-4 p-0">
    {#each instances as name}
      <li class="flex justify-between items-center rounded-lg border border-border bg-card p-4">
        <span class="font-medium">{name}</span>
        <button 
          class="rounded-lg border border-transparent px-[1.2em] py-[0.6em] text-base font-medium font-inherit bg-primary text-primary-foreground cursor-pointer transition-opacity duration-250 hover:opacity-90 disabled:opacity-50 disabled:cursor-not-allowed" 
          on:click={() => loginAndLaunch(name)} 
          disabled={isLoggingIn}
        >
          {account ? "Play" : "Login & Play"}
        </button>
      </li>
    {:else}
      <li>
        <p class="text-muted-foreground my-4">No instances found. Install one above!</p>
      </li>
    {/each}
  </ul>
</Layout>
