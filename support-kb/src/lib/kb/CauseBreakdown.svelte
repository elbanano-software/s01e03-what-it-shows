<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Cause, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { issues, resolutions }: { issues: Record<Cause, string>; resolutions: Record<Cause, string> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === 4471));
  const groups = $derived(groupByCause(merged, tenantOf));
  const tags = (g: { taggedAs: Record<string, number> }) =>
    Object.entries(g.taggedAs)
      .sort((a, b) => b[1] - a[1])
      .map(([tag, n]) => `${tag} ×${n}`)
      .join(' · ');
</script>

<p class="sum">
  <strong>{merged.length}</strong> distinct reports merged into #4471 ·
  {reports.length - merged.length} overlapping export rows counted once
</p>

<table>
  <thead>
    <tr><th>cause</th><th>reports</th><th>intake tag</th><th>tenants</th><th>seen</th><th>resolution</th><th>tracking issue</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td class="tags">{tags(g)}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{resolutions[g.cause as Cause]}</td>
        <td><a href={issues[g.cause as Cause]}>Open {g.cause} issue</a></td>
      </tr>
    {/each}
  </tbody>
</table>

{#each groups as g (g.cause)}
  {@const moved = readAs(g, notes.oneDayMove)}
  {#if moved}
    <p class="note">{moved} reports read as a one-day move; the tenant/form rules still place them under <code>{g.cause}</code>.</p>
  {/if}
{/each}

<style>
  .sum { font-size: 1.1rem; }
  .tags { color: var(--muted); font-size: 0.85em; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
