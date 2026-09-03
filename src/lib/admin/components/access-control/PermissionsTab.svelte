<!--
SPDX-FileCopyrightText: 2026 Perter Technology Solutions Private Limited
SPDX-License-Identifier: Apache-2.0
-->

<script lang="ts">
  import type { Permission } from "../../../api/admin/permissions.js";
  import LoadingSpinner from "../LoadingSpinner.svelte";
  import AdminEmptyState from "../AdminEmptyState.svelte";
  import { _ } from "svelte-i18n";
  import {
    formatAction,
    formatDomain,
    getPermissionDescription,
    groupPermissionsByDomain,
  } from "./permissionGroups";

  interface Props {
    permissions: Permission[];
    loading: boolean;
    /** ".layout-toggles" choice, owned by the page. */
    view?: "list" | "grid";
  }

  let { permissions, loading, view = "list" }: Props = $props();

  let query = $state("");

  /**
   * Search matches the three things the row actually shows — domain, action and
   * description — so a query can never hide a row whose visible text contains it.
   */
  const filtered = $derived.by(() => {
    const q = query.trim().toLowerCase();
    if (!q) return permissions;
    return permissions.filter((perm) => {
      const description = getPermissionDescription(perm, $_) ?? "";
      return (
        formatDomain(perm.domain).toLowerCase().includes(q) ||
        formatAction(perm.action).toLowerCase().includes(q) ||
        description.toLowerCase().includes(q)
      );
    });
  });

  const grouped = $derived(groupPermissionsByDomain(filtered));
  const permissionsByDomain = $derived(grouped.permissionsByDomain);
  const domainOrder = $derived(grouped.domainOrder);

  function countLabel(n: number): string {
    return $_("admin.accessControl.permissionsTab.domainCount", {
      values: { count: n },
    });
  }
</script>

{#snippet scopePill(perm: Permission, small: boolean)}
  <span
    class="{small ? 'scope-pill-sm' : 'scope-pill'} {perm.is_scopeable
      ? 'scope-pill--dept'
      : 'scope-pill--global'}"
    title={perm.is_scopeable
      ? $_("admin.accessControl.departmentScopeSupportTooltip")
      : undefined}
  >
    {perm.is_scopeable
      ? $_("admin.accessControl.departmentLabel")
      : $_("admin.accessControl.permissionsTab.globalBadge")}
  </span>
{/snippet}

{#if loading}
  <LoadingSpinner text={$_("admin.accessControl.permissionsTab.loading")} />
{:else if permissions.length === 0}
  <AdminEmptyState
    title={$_("admin.accessControl.permissionsTab.noPermissionsTitle")}
    message={$_("admin.accessControl.permissionsTab.noPermissionsDescription")}
  />
{:else}
  <div class="panel">
    <!-- ".search-row" -->
    <div class="search-row">
      <svg width="16" height="16" viewBox="0 0 16 16" fill="none" aria-hidden="true">
        <circle cx="7" cy="7" r="4.5" stroke="currentColor" stroke-width="1.3" />
        <path d="m13 13-2.5-2.5" stroke="currentColor" stroke-width="1.3" />
      </svg>
      <input
        type="text"
        bind:value={query}
        placeholder={$_("admin.accessControl.searchPermissions")}
        aria-label={$_("admin.accessControl.searchPermissions")}
      />
    </div>

    {#if domainOrder.length === 0}
      <AdminEmptyState
        title={$_("admin.accessControl.permissionsTab.noPermissionsTitle")}
        message={$_("admin.accessControl.noSearchResults", {
          values: { query: query.trim() },
        })}
      />
    {:else if view === "list"}
      <!-- ".perms-list-wrap": one table, category rows breaking up the domains -->
      <div class="perms-list-wrap">
        <div class="table-container">
          <div class="thead-row">
            <span>{$_("admin.accessControl.permissionsTab.columns.permission")}</span>
            <span>{$_("admin.accessControl.permissionsTab.columns.description")}</span>
            <span>{$_("admin.accessControl.permissionsTab.columns.scope")}</span>
          </div>

          {#each domainOrder as domain (domain)}
            <div class="category-row">
              <span class="category-row__name">{formatDomain(domain)}</span>
              <span class="count-pill"
                >{countLabel(permissionsByDomain[domain].length)}</span
              >
            </div>
            {#each permissionsByDomain[domain] as perm (perm.id)}
              {@const description = getPermissionDescription(perm, $_)}
              <div class="perm-row">
                <span class="perm-row__name">{formatAction(perm.action)}</span>
                <span class="perm-row__desc">{description ?? ""}</span>
                {@render scopePill(perm, false)}
              </div>
            {/each}
          {/each}
        </div>
      </div>
    {:else}
      <!-- ".perms-grid-wrap": a card per domain; the grid handles the rows -->
      <div class="perms-grid-wrap">
            {#each domainOrder as domain (domain)}
              <div class="perm-grid-card">
                <div class="perm-grid-card__head">
                  <span class="perm-grid-card__title">{formatDomain(domain)}</span>
                  <span class="perm-grid-card__count"
                    >{countLabel(permissionsByDomain[domain].length)}</span
                  >
                </div>
                <div class="perm-grid-card__stack">
                  {#each permissionsByDomain[domain] as perm (perm.id)}
                    {@const description = getPermissionDescription(perm, $_)}
                    <div class="perm-grid-row-item">
                      <div class="perm-grid-row-item__left">
                        <span class="perm-grid-row-item__name"
                          >{formatAction(perm.action)}</span
                        >
                        {#if description}
                          <span class="perm-grid-row-item__desc" title={description}
                            >{description}</span
                          >
                        {/if}
                      </div>
                      {@render scopePill(perm, true)}
                    </div>
                  {/each}
                </div>
              </div>
            {/each}
      </div>
    {/if}
  </div>
{/if}

<style>
  .panel {
    display: flex;
    flex-direction: column;
    gap: 20px;
    align-items: flex-start;
    align-self: stretch;
    width: 100%;
    min-width: 0;
  }

  /* ---- ".search-row" ---- */
  .search-row {
    height: 37px;
    border-radius: 8px;
    background: var(--gx-card);
    box-shadow: inset 0 0 0 1px var(--gx-hair);
    display: flex;
    gap: 10px;
    padding: 10px 14px;
    align-items: center;
    flex-shrink: 0;
    align-self: stretch;
    color: var(--gx-slate-400);
    box-sizing: border-box;
  }

  .search-row:focus-within {
    box-shadow: inset 0 0 0 1px var(--gx-org-primary-500);
  }

  .search-row svg {
    flex-shrink: 0;
  }

  .search-row input {
    flex-grow: 1;
    min-width: 0;
    /* app.css styles every bare <input> as a full glass field — padding, its own
       radius, a fill, an inset shadow and a focus ring. Inside a search row the
       container IS the field, so all of that has to be neutralised or the input
       draws a second pill inside the first. */
    width: 100%;
    padding: 0;
    border: 0;
    border-radius: 0;
    outline: none;
    background: transparent;
    box-shadow: none;
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
    font-family: var(--gx-font);
    font-weight: 400;
    font-size: 14px;
    line-height: 100%;
    color: var(--gx-slate-900);
  }

  .search-row input:focus {
    box-shadow: none;
    background: transparent;
  }

  .search-row input::placeholder {
    color: var(--gx-slate-400);
    opacity: 1;
  }

  /* ---- ".perms-list-wrap": table view ---- */
  .perms-list-wrap {
    align-self: stretch;
    width: 100%;
    min-width: 0;
  }

  /* The three columns are fixed-width by design, so the table scrolls inside its
     own container on narrow viewports rather than pushing the page sideways. */
  .table-container {
    overflow-x: auto;
    border-radius: 12px;
    box-shadow: inset 0 0 0 1px var(--gx-hair);
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    align-self: stretch;
  }

  .thead-row,
  .category-row,
  .perm-row {
    min-width: 640px;
  }

  .thead-row {
    background: var(--gx-card);
    display: flex;
    gap: 16px;
    padding: 12px 16px;
    align-items: center;
    align-self: stretch;
    font-family: var(--gx-font);
    font-weight: 700;
    font-size: 11px;
    letter-spacing: 0.5px;
    color: var(--gx-slate-400);
    text-transform: uppercase;
    box-sizing: border-box;
  }

  .thead-row span:first-child {
    width: 180px;
    flex-shrink: 0;
  }

  .thead-row span:nth-child(2) {
    flex-grow: 1;
  }

  .thead-row span:last-child {
    width: 90px;
    flex-shrink: 0;
  }

  .category-row {
    min-height: 37px;
    background: var(--gx-org-track);
    border-top: 1px solid var(--gx-hair);
    display: flex;
    gap: 8px;
    padding: 10px 16px;
    align-items: center;
    align-self: stretch;
    box-sizing: border-box;
  }

  .category-row__name {
    font-family: var(--gx-font);
    font-weight: 700;
    font-size: 14px;
    color: var(--gx-slate-900);
    white-space: nowrap;
  }

  .count-pill {
    border-radius: 100px;
    background: var(--gx-hair);
    padding: 2px 6px;
    font-family: var(--gx-font);
    font-weight: 600;
    font-size: 11px;
    color: var(--gx-slate-500);
    white-space: nowrap;
  }

  .perm-row {
    min-height: 51px;
    background: var(--gx-card);
    border-top: 1px solid var(--gx-hair);
    display: flex;
    gap: 16px;
    padding: 14px 16px;
    align-items: center;
    align-self: stretch;
    box-sizing: border-box;
  }

  .perm-row__name {
    width: 180px;
    flex-shrink: 0;
    font-family: var(--gx-font);
    font-weight: 600;
    font-size: 14px;
    color: var(--gx-slate-900);
  }

  .perm-row__desc {
    flex-grow: 1;
    font-family: var(--gx-font);
    font-weight: 400;
    font-size: 14px;
    color: var(--gx-slate-500);
  }

  /* ---- scope pills ---- */
  .scope-pill {
    border-radius: 6px;
    padding: 4px 10px;
    font-family: var(--gx-font);
    font-weight: 600;
    font-size: 12px;
    white-space: nowrap;
    flex-shrink: 0;
  }

  .scope-pill-sm {
    border-radius: 6px;
    padding: 3px 8px;
    font-family: var(--gx-font);
    font-weight: 600;
    font-size: 11px;
    white-space: nowrap;
    flex-shrink: 0;
  }

  .scope-pill--global {
    background: var(--gx-card);
    box-shadow: inset 0 0 0 1px var(--gx-hair);
    color: var(--gx-slate-900);
  }

  .scope-pill--dept {
    background: var(--gx-ac-dept-bg);
    color: var(--gx-ac-dept-fg);
  }

  /* ---- ".perms-grid-wrap": card view ---- */
  /* Same reason as ".roles-grid": a wrapping flex row would stretch a lone card
     on the last line to full width. auto-fit keeps the column width fixed. */
  .perms-grid-wrap {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
    gap: 28px;
    align-items: stretch;
    align-self: stretch;
    width: 100%;
    min-width: 0;
  }

  .perm-grid-card {
    border-radius: 16px;
    background: var(--gx-card);
    box-shadow: inset 0 0 0 1px var(--gx-hair);
    display: flex;
    flex-direction: column;
    padding: 20px;
    align-items: flex-start;
    min-width: 0;
    box-sizing: border-box;
  }

  .perm-grid-card__head {
    border-bottom: 1px solid var(--gx-hair);
    display: flex;
    justify-content: space-between;
    align-items: center;
    align-self: stretch;
    gap: 10px;
    padding-bottom: 12px;
    margin-bottom: 14px;
  }

  .perm-grid-card__title {
    font-family: var(--gx-font);
    font-weight: 700;
    font-size: 16px;
    color: var(--gx-slate-900);
  }

  .perm-grid-card__count {
    font-family: var(--gx-font);
    font-weight: 400;
    font-size: 12px;
    color: var(--gx-slate-500);
    white-space: nowrap;
  }

  .perm-grid-card__stack {
    display: flex;
    flex-direction: column;
    gap: 12px;
    align-items: flex-start;
    align-self: stretch;
  }

  .perm-grid-row-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    align-self: stretch;
    gap: 10px;
  }

  .perm-grid-row-item__left {
    display: flex;
    flex-direction: column;
    gap: 2px;
    min-width: 0;
  }

  .perm-grid-row-item__name {
    font-family: var(--gx-font);
    font-weight: 600;
    font-size: 13px;
    color: var(--gx-slate-900);
    white-space: nowrap;
  }

  .perm-grid-row-item__desc {
    font-family: var(--gx-font);
    font-weight: 400;
    font-size: 11px;
    color: var(--gx-slate-500);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  @media (max-width: 768px) {
    .perms-grid-wrap {
      gap: 16px;
    }
  }
</style>
