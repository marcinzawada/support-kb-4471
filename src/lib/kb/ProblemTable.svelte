<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { mistagged, splitByProblem, type Problem } from '#lib/kb/split';

  // issues: where each problem is tracked, problem → issue URL and how it ended.
  type Tracked = { url: string; state?: string };
  let { ticket, issues = {} }: { ticket: number; issues?: Partial<Record<Problem, Tracked>> } = $props();

  const rows = $derived((reports as Report[]).filter((r) => r.mergedInto === ticket));
  const merged = $derived(uniqueById(rows));
  const groups = $derived(splitByProblem(merged, tenantOf, notes.oneDayMove));
  // The desk's own view (CauseTable): one-day moves stay under the cause of their tenant.
  const desk = $derived(new Map(groupByCause(merged, tenantOf).map((g) => [g.cause as string, g.reports.length])));
  const deskCount = (p: Problem) => desk.get(p);
  const tags = (t: Record<string, number>) =>
    Object.entries(t)
      .sort((a, b) => b[1] - a[1])
      .map(([tag, n]) => `${tag} ×${n}`)
      .join(' · ');
  const issueNo = (url: string) => '#' + url.split('/').pop();
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong>
  {groups.length === 1 ? 'problem' : 'problems'}
  {#if rows.length !== merged.length}
    <span class="muted">({rows.length - merged.length} rows came twice in the export, counted once)</span>
  {/if}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>desk counts</th><th>tenants</th><th>seen</th><th>intake tagged</th><th>tracked in</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.problem)}
      {@const t = issues[g.problem]}
      {@const n = deskCount(g.problem)}
      <tr>
        <td><code>{g.problem}</code></td>
        <td><strong>{g.reports.length}</strong></td>
        <td>{#if n !== undefined}<strong>{n}</strong>{#if n !== g.reports.length}{' '}<span class="muted">(+{n - g.reports.length} one-day)</span>{/if}{:else}<span class="muted">under <code>parser</code></span>{/if}</td>
        <td title={g.tenants.join(', ')}>{g.tenants.length}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td class="tags">{tags(g.taggedAs)}{#if mistagged(g)}<br /><span class="off">{mistagged(g)} tagged as something else</span>{/if}</td>
        <td>{#if t}<a href={t.url}>{issueNo(t.url)}</a>{#if t.state}{' '}<span class="muted">{t.state}</span>{/if}{:else}<span class="off">no ticket yet</span>{/if}</td>
      </tr>
    {/each}
  </tbody>
  <tfoot>
    <tr>
      <td>total</td>
      <td><strong>{groups.reduce((n, g) => n + g.reports.length, 0)}</strong></td>
      <td><strong>{[...desk.values()].reduce((n, c) => n + c, 0)}</strong></td>
      <td colspan="4"></td>
    </tr>
  </tfoot>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .muted, .tags { color: var(--muted); font-size: 0.85em; }
  .off { color: var(--warn); font-weight: 600; }
</style>
