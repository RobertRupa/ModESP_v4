<script>
  import { onMount } from 'svelte';
  import { apiGet } from '../lib/api.js';
  import { state } from '../stores/state.js';

  const relays = Array.from({ length: 6 }, (_, i) => ({ id: `relay_${i+1}`, label: `R${i+1}` }));
  const inputs = Array.from({ length: 6 }, (_, i) => ({ id: `din_${i+1}`, label: `DI${i+1}` }));
  let bindings = [];
  let loading = true;
  let error = '';

  onMount(async () => {
    try {
      const data = await apiGet('/api/bindings');
      bindings = Array.isArray(data.bindings) ? data.bindings : [];
    } catch (e) {
      error = e.message || String(e);
    } finally {
      loading = false;
    }
  });

  function bindingFor(id) {
    return bindings.find(b => b.hardware === id);
  }

  function liveValue(binding) {
    if (!binding) return undefined;
    // Only show state explicitly published by backend, never infer electrical relay state.
    const key = `equipment.${binding.role}`;
    return Object.prototype.hasOwnProperty.call($state, key) ? $state[key] : undefined;
  }

  function formatState(value) {
    if (value === undefined || value === null) return '—';
    if (typeof value === 'boolean') return value ? 'ON' : 'OFF';
    return String(value);
  }
</script>

<section class="kcs-io">
  <header>
    <h2>KC868-A6 · Wejścia i wyjścia</h2>
    <p>Podgląd przypisań sprzętowych z /api/bindings. Stany pojawiają się tylko wtedy, gdy backend publikuje odpowiedni klucz equipment.*.</p>
  </header>

  {#if loading}
    <p>Odczytywanie konfiguracji…</p>
  {:else if error}
    <p class="error">Nie można pobrać przypisań: {error}</p>
  {:else}
    <div class="sections">
      <article>
        <h3>Przekaźniki · PCF8574 0x24</h3>
        <div class="io-grid">
          {#each relays as item}
            {@const binding = bindingFor(item.id)}
            <div class="io-cell">
              <strong>{item.label}</strong>
              <span>{binding?.role || 'Nieprzypisany'}</span>
              <small>Stan zgłoszony: {formatState(liveValue(binding))}</small>
            </div>
          {/each}
        </div>
      </article>
      <article>
        <h3>Wejścia · PCF8574 0x22</h3>
        <div class="io-grid">
          {#each inputs as item}
            {@const binding = bindingFor(item.id)}
            <div class="io-cell">
              <strong>{item.label}</strong>
              <span>{binding?.role || 'Nieprzypisany'}</span>
              <small>Stan zgłoszony: {formatState(liveValue(binding))}</small>
            </div>
          {/each}
        </div>
      </article>
    </div>
  {/if}
  <footer>
    <p>I²C: GPIO4 SDA / GPIO15 SCL · 100 kHz. Ta strona nie steruje przekaźnikami — kontrolę wyjść zachowują moduły automatyki i zabezpieczeń.</p>
  </footer>
</section>

<style>
  .kcs-io { display: grid; gap: 20px; color: var(--text-1); }
  header, article, footer { border: 1px solid var(--border); border-radius: 14px; background: var(--surface); padding: 20px; }
  h2 { font-size: 22px; margin-bottom: 10px; }
  h3 { font-size: 16px; margin-bottom: 16px; }
  p { line-height: 1.5; color: var(--text-3); }
  .sections { display: grid; gap: 18px; }
  .io-grid { display: grid; grid-template-columns: repeat(3, minmax(0,1fr)); gap: 10px; }
  .io-cell { padding: 14px; display: grid; gap: 6px; background: var(--surface-2); border-radius: 10px; min-width: 0; }
  strong { font-size: 18px; }
  span { font-size: 12px; overflow-wrap: anywhere; }
  small { color: var(--text-3); font-size: 11px; }
  .error { color: var(--red); }
  @media (max-width: 580px) { .io-grid { grid-template-columns: repeat(2, minmax(0,1fr)); } }
</style>
