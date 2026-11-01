<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { problemOf, type Problem } from '#lib/kb/split';

  // The reports behind one problem, so a reader can open them.
  let { ticket, problem }: { ticket: number; problem: Problem } = $props();

  const mine = $derived(
    uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket && problemOf(r, tenantOf(r), notes.oneDayMove) === problem),
  );
</script>

<details>
  <summary>{mine.length} reports · <code>{problem}</code></summary>
  <ul>
    {#each mine as r (r.id)}
      <li><code>{r.id}</code> · {r.tenant} · {r.channel} · {r.receivedAt.slice(0, 10)} — {r.body}</li>
    {/each}
  </ul>
</details>

<style>
  details { margin: 0.5rem 0 1rem; }
  summary { cursor: pointer; color: var(--muted); font-size: 0.9rem; }
  ul { font-size: 0.85rem; padding-left: 1.2rem; }
  li { margin: 0.2rem 0; }
</style>
