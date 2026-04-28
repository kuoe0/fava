<script lang="ts">
  import { 
    Book,
    Box,
    Briefcase,
    Calendar,
    ChartNoAxesColumn,
    CircleQuestionMark,
    Download,
    FileText,
    ListChecks,
    Plus, 
    Scale,
    SlidersVertical,
    SquarePen,
    Terminal,
    TrendingUp,
    Upload  } from "@lucide/svelte";

  import { urlFor } from "../helpers.ts";
  import { _ } from "../i18n.ts";
  import { keyboardShortcut } from "../keyboard-shortcuts.ts";
  import { errors, extensions, ledgerData } from "../stores/index.ts";
  import AccountSelector from "./AccountSelector.svelte";
  import Link from "./SidebarLink.svelte";

  const truncate = (s: string) => (s.length < 25 ? s : `${s.slice(25)}…`);

  let user_queries = $derived($ledgerData.user_queries);
  let upcoming_events_count = $derived($ledgerData.upcoming_events_count);
  let sidebar_links = $derived($ledgerData.sidebar_links);
  let extension_reports = $derived(
    $extensions.filter((e) => e.report_title != null),
  );
</script>

{#if sidebar_links.length}
  <ul class="navigation">
    {#each sidebar_links as [label, link] (link)}
      <Link report={link} name={label} remote />
    {/each}
  </ul>
{/if}
<ul class="navigation">
  <Link report="income_statement" name={_("Income Statement")} key="g i" icon={TrendingUp} />
  <Link report="balance_sheet" name={_("Balance Sheet")} key="g b" icon={Scale} />
  <Link report="trial_balance" name={_("Trial Balance")} key="g t" icon={ListChecks} />
  <Link report="journal" name={_("Journal")} key="g j" icon={Book} />
  <Link report="query" name={_("Query")} key="g q" icon={Terminal}>
    {#if user_queries.length}
      <ul class="submenu">
        {#each user_queries as { query_string, name } (query_string)}
          <li>
            <a href={$urlFor("query/", { query_string })}>{truncate(name)}</a>
          </li>
        {/each}
      </ul>
    {/if}
  </Link>
  <AccountSelector />
</ul>
<ul class="navigation">
  <Link report="holdings" name={_("Holdings")} key="g h" icon={Briefcase} />
  <Link report="commodities" name={_("Commodities")} key="g c" icon={Box} />
  <Link report="documents" name={_("Documents")} key="g d" icon={FileText} />
  <Link
    report="events"
    name={_("Events")}
    key="g E"
    bubble={[upcoming_events_count, "info"]}
    icon={Calendar}
  />
  <Link report="statistics" name={_("Statistics")} key="g s" icon={ChartNoAxesColumn} />
</ul>
<ul class="navigation">
  <Link report="editor" name={_("Editor")} key="g e" icon={SquarePen}>
    {#snippet actions()}
      <a
        href="#add-transaction"
        class="secondary add-transaction"
        title={_("Add Journal Entry")}
        {@attach keyboardShortcut("n")}
        aria-label={_("Add Journal Entry")}
      >
        <Plus size={16} />
      </a>
    {/snippet}
  </Link>
  {#if $errors.length > 0}
    <Link
      report="errors"
      name={_("Errors")}
      bubble={[$errors.length, "error"]}
      icon={FileText}
    />
  {/if}
  <Link report="import" name={_("Import")} key="g n" icon={Upload}>
    {#snippet actions()}
      <a href="#export" class="secondary" title={_("Export")} aria-label={_("Export")}>
        <Download size={16} />
      </a>
    {/snippet}
  </Link>
  <Link report="options" name={_("Options")} key="g o" icon={SlidersVertical} />
  <Link report="help" name={_("Help")} key="g H" icon={CircleQuestionMark} />
</ul>
{#if extension_reports.length}
  <ul class="navigation">
    {#each extension_reports as ext (ext.name)}
      <Link report={`extension/${ext.name}`} name={ext.report_title ?? ""} />
    {/each}
  </ul>
{/if}

<style>
  .navigation {
    padding: 0 0 0.5rem 0;
    margin: 0;
  }

  .navigation + .navigation {
    padding-top: 0.5rem;
    border-top: 1px solid var(--sidebar-border);
  }

  a {
    display: block;
    padding: 0.25em 0.5em 0.25em 1em;
    color: inherit;
  }

  a:hover {
    color: var(--sidebar-hover-color);
    background-color: var(--sidebar-border);
  }

  .secondary {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 30px;
    height: 30px;
    padding: 0;
    color: inherit;
    background-color: transparent;
    border-radius: 50%;
    transition: background-color 0.2s;
  }

  .secondary:hover {
    background-color: rgba(255, 255, 255, 0.1);
  }

  .add-transaction {
    font-size: 23px; /* Fallback or override if needed */
  }

  .submenu {
    width: 100%;
    margin: 0 0 0.5em;
  }

  .submenu a {
    width: 100%;
    padding-left: 35px;
  }

  .submenu li {
    font-size: 0.9em;
  }
</style>
