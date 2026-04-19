# Patch notes — claude/audit-clearbot-user-flow-Tbzl4

This branch is part of the Clearbot user-flow audit. The edits below
are already prepared in the local working copy and will land in a
follow-up commit on this branch. Most of the audit changes are in
`index.html`; a working-today standalone **Bookings** page has already
been committed as `bookings.html`.

## Already landed

- `bookings.html` — live at `team.clearbot.io/bookings.html`. Self-contained page listing the `booking_requests` table with search, status filter, detail modal, inline status updates, and delete-with-confirm.

## `index.html` edits — still to land

### 1. Rewritten destructive-action modals (remove "orphaned" jargon, add "cannot be undone")

- **`confirmDeleteProject`** (~line 2692):
  ```diff
  -    bodyHtml: '<p class="modal-prose">This removes the project row permanently. Deliverables, invoices and activity linked to it by ID will be orphaned.</p><div class="modal-error" data-modal-error hidden></div>',
  +    bodyHtml: '<p class="modal-prose">This deletes the project permanently. Any deliverables, invoices, and activity linked to it will stay in the database but will no longer be grouped under this project.</p><p class="modal-prose"><strong>This cannot be undone.</strong> Consider archiving the project instead.</p><div class="modal-error" data-modal-error hidden></div>',
  ```

- **`confirmDeleteInvoice`** (~line 2790):
  ```diff
  -    bodyHtml: '<p class="modal-prose">This removes the invoice permanently. Consider marking it as paid or changing status instead.</p><div class="modal-error" data-modal-error hidden></div>',
  +    bodyHtml: '<p class="modal-prose">This deletes the invoice permanently.</p><p class="modal-prose"><strong>This cannot be undone.</strong> Consider marking it as paid or changing its status instead.</p><div class="modal-error" data-modal-error hidden></div>',
  ```

- **`confirmDeleteDeliverable`** (~line 2871):
  ```diff
  -    bodyHtml: '<p class="modal-prose">This removes the deliverable row. External files at the URL are not affected.</p><div class="modal-error" data-modal-error hidden></div>',
  +    bodyHtml: '<p class="modal-prose">This deletes the deliverable record. External files at the URL itself are not affected.</p><p class="modal-prose"><strong>This cannot be undone.</strong></p><div class="modal-error" data-modal-error hidden></div>',
  ```

- **`confirmDeleteAccessCode`** (~line 3550) — append `<p class="modal-prose"><strong>This cannot be undone.</strong></p>` to `bodyHtml`.

### 2. Client header fallback no longer reveals raw UUIDs

- **`renderUserDetail`** (~line 2335):
  ```diff
  -  if (eyeEl) eyeEl.textContent = 'Client · ' + (profile && profile.email ? profile.email : state.selectedUserId);
  +  if (eyeEl) {
  +    const eyebrowLabel =
  +      (profile && profile.company_name) ||
  +      (profile && profile.email) ||
  +      [profile && profile.first_name, profile && profile.last_name].filter(Boolean).join(' ').trim() ||
  +      state.selectedUserId;
  +    eyeEl.textContent = 'Client · ' + eyebrowLabel;
  +  }
  ```

### 3. Rewritten unauthorized view

Replace the "Your email is not on the team allowlist" empty state with an explicit "You don't have access to this dashboard yet" page that:
- Explains what the team dashboard is for.
- Points clients to `client.clearbot.io`.
- Tells teammates what they need (allowlist + `profiles.role = 'admin' | 'team'`).
- Offers a "Go to client portal" button in addition to Sign out.

### 4. Access codes explainer

Add an explainer `.panel.bracketed` section above the search bar in the `codes-all` view, clarifying what access codes are for and when to issue them.

### 5. Role dropdown copy + modal keyboard hint

- Role `<option>`s in the user-detail form now describe what each role grants, and the `form-help` text mentions the `TEAM_EMAILS` allowlist requirement.
- `openModal` footer adds a small `Enter to confirm · Esc to cancel` hint.

---

## 6. Bookings: in-dashboard integration

The bookings data is already live on a separate `bookings.html` page.
The edits below promote it into the main dashboard shell (sidebar + home
tile + its own view-pane).

### 6.1 Sidebar nav — insert just before the "Access Codes" button (~line 1116)

```html
<button class="nav-item" type="button" data-view="bookings-all">
  <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="3" y="4" width="18" height="18" rx="2"/><path d="M16 2v4M8 2v4M3 10h18"/></svg>
  Bookings <span class="mono" data-sidebar-bookings-badge hidden style="margin-left:6px;padding:1px 6px;border:1px solid var(--rule-strong);border-radius:2px;font-size:9px;">0</span>
</button>
```

### 6.2 Home command-center — add a 6th tile

Change `grid-template-columns: repeat(5, 1fr);` to `repeat(6, 1fr);`, then append after the `activity-7d` tile:

```html
<div class="tile" data-tile="new-bookings" style="cursor:pointer;" data-view-link="bookings-all" title="Go to bookings">
  <div class="label">New Bookings</div>
  <div class="val">&mdash;</div>
  <div class="delta">&nbsp;</div>
</div>
```

### 6.3 View pane — insert just before `<section class="view-pane" data-view-pane="codes-all" hidden>` (~line 1530)

```html
<!-- BOOKINGS -->
<section class="view-pane" data-view-pane="bookings-all" hidden>
  <div class="page-head">
    <div>
      <div class="eyebrow">Inbound</div>
      <h1>Intro call <em>bookings</em>.</h1>
    </div>
    <div class="actions">
      <select class="input" id="bookings-status-filter" style="width:auto;padding:8px 12px;font-size:12px;">
        <option value="">All statuses</option>
        <option value="new">New</option>
        <option value="contacted">Contacted</option>
        <option value="scheduled">Scheduled</option>
        <option value="dropped">Dropped</option>
      </select>
    </div>
  </div>
  <section class="panel bracketed" style="margin-bottom:20px;">
    <p class="modal-prose" style="margin:0;">
      Booking requests submitted at <strong>signup.clearbot.io/book</strong>. Click a row for the full brief, to update status, or to delete.
    </p>
  </section>
  <div class="search-bar">
    <input class="input" id="bookings-search" type="search" placeholder="Search name, email, company, focus&hellip;" autocomplete="off" />
    <span class="count" data-bookings-count>&mdash;</span>
  </div>
  <div class="table-wrap" data-bookings-wrap>
    <table class="table">
      <thead>
        <tr>
          <th style="width: 22%;">Name</th>
          <th>Company</th>
          <th>Focus</th>
          <th>Preferred slot</th>
          <th>Status</th>
          <th>Received</th>
          <th></th>
        </tr>
      </thead>
      <tbody data-bookings-tbody></tbody>
    </table>
  </div>
  <div data-bookings-empty></div>
</section>
```

### 6.4 VIEW_TITLES — add the entry (~line 2114)

```diff
+  'bookings-all': 'Bookings',
   'codes-all': 'Access Codes',
```

### 6.5 `showView` dispatch (~line 2142) — append one branch

```diff
     else if (name === 'codes-all')         await renderCodesAll();
+    else if (name === 'bookings-all')      await renderBookingsAll();
```

### 6.6 Data + render + detail modal — insert directly before the `// Identity + bootstrap` comment block (~line 3743)

```js
// ============================================================================
// Bookings (inbound from signup.clearbot.io/book)
// ============================================================================
async function listBookingRequests() {
  return safeList(() =>
    supabase.from('booking_requests')
      .select('id, name, email, company, focus, preferred_slot, preferred_slot_label, timezone, notes, source, status, created_at')
      .order('created_at', { ascending: false })
      .limit(500)
  );
}
async function updateBookingStatus(id, status) {
  return safeWrite(() => supabase.from('booking_requests').update({ status }).eq('id', id));
}
async function deleteBookingRequest(id) {
  return safeWrite(() => supabase.from('booking_requests').delete().eq('id', id));
}
async function countNewBookings() {
  try {
    const r = await supabase.from('booking_requests')
      .select('*', { count: 'exact', head: true })
      .eq('status', 'new');
    return r.count || 0;
  } catch (_) { return 0; }
}
function bookingStatusBadge(status) {
  const s = (status || 'new').toLowerCase();
  const map = {
    new:       ['accent', 'New'],
    contacted: ['warn',   'Contacted'],
    scheduled: ['ok',     'Scheduled'],
    dropped:   ['',       'Dropped'],
  };
  const hit = map[s] || ['', status || 'New'];
  return '<span class="badge ' + hit[0] + '"><span class="dot"></span>' + escapeHtml(hit[1]) + '</span>';
}
async function renderBookingsAll() {
  const rows = await listBookingRequests();
  const tbody = $('[data-bookings-tbody]');
  const wrap  = $('[data-bookings-wrap]');
  const emptyRoot = $('[data-bookings-empty]');
  const searchEl  = $('#bookings-search');
  const filterEl  = $('#bookings-status-filter');
  const countEl   = $('[data-bookings-count]');
  if (!tbody) return;

  const draw = () => {
    const q = (searchEl && searchEl.value || '').trim().toLowerCase();
    const status = (filterEl && filterEl.value || '').toLowerCase();
    const filtered = rows.filter(r => {
      if (status && (r.status || 'new').toLowerCase() !== status) return false;
      if (!q) return true;
      const hay = [r.name, r.email, r.company, r.focus, r.notes, r.status]
        .filter(Boolean).join(' ').toLowerCase();
      return q.split(/\s+/).every(p => hay.indexOf(p) !== -1);
    });
    if (countEl) countEl.textContent = filtered.length + (filtered.length === 1 ? ' booking' : ' bookings');
    if (!rows.length) {
      if (wrap) wrap.hidden = true;
      if (emptyRoot) emptyRoot.innerHTML = emptyState(
        'No booking requests yet',
        'When someone submits the form at signup.clearbot.io/book it will show up here.'
      );
      return;
    }
    if (!filtered.length) {
      if (wrap) wrap.hidden = true;
      if (emptyRoot) emptyRoot.innerHTML = emptyState('No matches', 'Try a different search term or clear the filters.');
      return;
    }
    if (wrap) wrap.hidden = false;
    if (emptyRoot) emptyRoot.innerHTML = '';
    tbody.innerHTML = filtered.map(r =>
      '<tr data-booking-row="' + escapeHtml(r.id) + '" style="cursor:pointer;">' +
        '<td>' +
          '<div class="proj-name">' + escapeHtml(r.name || '\u2014') + '</div>' +
          '<div class="sub">' + escapeHtml(r.email || '') + '</div>' +
        '</td>' +
        '<td class="mono">' + escapeHtml(r.company || '\u2014') + '</td>' +
        '<td class="mono">' + escapeHtml(r.focus || '\u2014') + '</td>' +
        '<td class="mono">' + escapeHtml(r.preferred_slot_label || (r.preferred_slot ? fmtShortDate(r.preferred_slot) : '\u2014')) + '</td>' +
        '<td>' + bookingStatusBadge(r.status) + '</td>' +
        '<td class="mono">' + fmtRelTime(r.created_at) + '</td>' +
        '<td><button type="button" class="btn btn-sm" data-booking-open="' + escapeHtml(r.id) + '">Open</button></td>' +
      '</tr>'
    ).join('');
    const open = (id) => { const row = rows.find(x => x.id === id); if (row) showBookingDetail(row); };
    tbody.querySelectorAll('[data-booking-row]').forEach(tr => {
      tr.addEventListener('click', () => open(tr.getAttribute('data-booking-row')));
    });
    tbody.querySelectorAll('[data-booking-open]').forEach(btn => {
      btn.addEventListener('click', (e) => { e.stopPropagation(); open(btn.getAttribute('data-booking-open')); });
    });
  };

  if (searchEl && !searchEl.dataset.bound) {
    searchEl.dataset.bound = '1';
    let t = null;
    searchEl.addEventListener('input', () => { if (t) clearTimeout(t); t = setTimeout(draw, 120); });
  }
  if (filterEl && !filterEl.dataset.bound) {
    filterEl.dataset.bound = '1';
    filterEl.addEventListener('change', draw);
  }
  draw();
  refreshBookingsBadge(rows);
}
function refreshBookingsBadge(rows) {
  const badge = $('[data-sidebar-bookings-badge]');
  if (!badge) return;
  const newCount = (rows || []).filter(r => (r.status || 'new').toLowerCase() === 'new').length;
  if (newCount > 0) { badge.hidden = false; badge.textContent = newCount; }
  else { badge.hidden = true; }
}
function showBookingDetail(booking) {
  const statusButtons = ['new','contacted','scheduled','dropped'].map(s =>
    '<button type="button" class="btn btn-sm" data-booking-status="' + s + '"' +
    ((booking.status || 'new') === s ? ' style="border-color:var(--accent);color:var(--accent);"' : '') +
    '>' + s.charAt(0).toUpperCase() + s.slice(1) + '</button>'
  ).join('');
  const mailHref = 'mailto:' + encodeURIComponent(booking.email || '') +
    '?subject=' + encodeURIComponent('Re: your ClearBot intro call') +
    '&body=' + encodeURIComponent('Hi ' + (booking.name ? booking.name.split(/\s+/)[0] : 'there') + ',\n\nThanks for booking an intro call.\n\n');

  openModal({
    eyebrow: 'Booking request',
    title: '<em>' + escapeHtml(booking.name || 'Unknown') + '</em>',
    wide: true,
    bodyHtml:
      '<div style="display:flex;flex-wrap:wrap;gap:12px;align-items:center;margin-bottom:20px;">' +
        bookingStatusBadge(booking.status) +
        '<span class="mono" style="font-size:11px;color:var(--ink-dim);">Received ' + fmtRelTime(booking.created_at) + '</span>' +
      '</div>' +
      '<div style="display:grid;grid-template-columns:1fr 1fr;gap:16px;margin-bottom:24px;padding:16px 18px;border:1px solid var(--rule);border-radius:2px;background:rgba(243,241,234,0.012);">' +
        '<div><div class="modal-eyebrow">Email</div><div class="mono" style="font-size:13px;margin-top:6px;"><a href="' + mailHref + '" style="color:var(--ink);">' + escapeHtml(booking.email || '\u2014') + '</a></div></div>' +
        '<div><div class="modal-eyebrow">Company</div><div style="font-family:var(--display);font-size:15px;margin-top:6px;color:var(--ink);">' + escapeHtml(booking.company || '\u2014') + '</div></div>' +
        '<div><div class="modal-eyebrow">Focus</div><div style="font-family:var(--display);font-size:15px;margin-top:6px;color:var(--ink);">' + escapeHtml(booking.focus || '\u2014') + '</div></div>' +
        '<div><div class="modal-eyebrow">Preferred slot</div><div style="font-family:var(--display);font-size:15px;margin-top:6px;color:var(--ink);">' + escapeHtml(booking.preferred_slot_label || '\u2014') + '</div></div>' +
        '<div><div class="modal-eyebrow">Timezone</div><div class="mono" style="font-size:12px;margin-top:6px;color:var(--ink);">' + escapeHtml(booking.timezone || '\u2014') + '</div></div>' +
        '<div><div class="modal-eyebrow">Source</div><div class="mono" style="font-size:12px;margin-top:6px;color:var(--ink);">' + escapeHtml(booking.source || '\u2014') + '</div></div>' +
      '</div>' +
      (booking.notes
        ? '<section style="margin-bottom:20px;"><div class="modal-eyebrow">Notes</div><p style="font-family:var(--display);font-size:15px;font-weight:300;line-height:1.55;color:var(--ink);margin-top:8px;white-space:pre-wrap;">' + escapeHtml(booking.notes) + '</p></section>'
        : '') +
      '<section>' +
        '<div class="modal-eyebrow" style="margin-bottom:10px;">Update status</div>' +
        '<div style="display:flex;flex-wrap:wrap;gap:8px;align-items:center;">' +
          statusButtons +
          '<button type="button" class="btn btn-sm btn-danger" data-booking-delete style="margin-left:auto;">Delete</button>' +
        '</div>' +
      '</section>' +
      '<div class="modal-error" data-modal-error hidden></div>',
    confirmLabel: 'Close',
    cancelLabel: 'Reply by email',
    onCancel: () => { window.location.href = mailHref; },
    onConfirm: () => true,
  });

  const root = document.querySelector('.modal-backdrop');
  if (!root) return;
  root.querySelectorAll('[data-booking-status]').forEach(btn => {
    btn.addEventListener('click', async () => {
      const s = btn.getAttribute('data-booking-status');
      root.querySelectorAll('[data-booking-status]').forEach(b => b.setAttribute('disabled', 'true'));
      const res = await updateBookingStatus(booking.id, s);
      if (!res.ok) {
        setModalError(root, res.error || 'Update failed.');
        root.querySelectorAll('[data-booking-status]').forEach(b => b.removeAttribute('disabled'));
        return;
      }
      booking.status = s;
      toast('Status set to ' + s);
      root.remove();
      await renderBookingsAll();
    });
  });
  const delBtn = root.querySelector('[data-booking-delete]');
  if (delBtn) {
    delBtn.addEventListener('click', async () => {
      openModal({
        eyebrow: 'Delete booking',
        title: 'Delete booking from <em>' + escapeHtml(booking.name || 'unknown') + '</em>?',
        bodyHtml: '<p class="modal-prose">This removes the record from the database.</p><p class="modal-prose"><strong>This cannot be undone.</strong></p><div class="modal-error" data-modal-error hidden></div>',
        confirmLabel: 'Delete',
        confirmVariant: 'danger',
        onConfirm: async (r) => {
          setModalBusy(r, true, 'Deleting\u2026');
          const res = await deleteBookingRequest(booking.id);
          if (!res.ok) { setModalError(r, res.error || 'Failed.'); setModalBusy(r, false); return false; }
          toast('Booking deleted');
          const outer = document.querySelector('.modal-backdrop');
          if (outer) outer.remove();
          await renderBookingsAll();
          return true;
        },
      });
    });
  }
}
```

### 6.7 Home wiring — inside `renderHome()`, after `updateSidebarUnreadBadge(unreadThreads);`

```js
// Surface pending booking requests on the command-center and sidebar.
const newBookings = await countNewBookings();
setTile('new-bookings', newBookings, newBookings === 0 ? 'No new requests' : 'Awaiting reply');
const bookingsTile = document.querySelector('[data-tile="new-bookings"]');
if (bookingsTile && !bookingsTile.dataset.bound) {
  bookingsTile.dataset.bound = '1';
  bookingsTile.addEventListener('click', () => showView('bookings-all'));
}
const sideBadge = $('[data-sidebar-bookings-badge]');
if (sideBadge) {
  if (newBookings > 0) { sideBadge.hidden = false; sideBadge.textContent = newBookings; }
  else sideBadge.hidden = true;
}
```

---

## Verification

After the HTML commit lands:
- [ ] Click the new "Bookings" sidebar link → confirm the view pane renders with the explainer, search, status filter, and table.
- [ ] From the home command-center → confirm the "New Bookings" tile shows the count of rows with `status = 'new'` and that clicking it routes to the Bookings view.
- [ ] Open a row → confirm the detail modal renders the full brief and that status buttons update the row and the sidebar badge.
- [ ] Delete a booking → confirm the "This cannot be undone." confirmation + the row disappears.
- [ ] Existing destructive modals (project / invoice / deliverable / access code) each end with "This cannot be undone."
- [ ] Client with no email set → confirm the eyebrow falls back to company/first+last, not UUID.
- [ ] Non-allowlisted user → confirm the rewritten unauthorized view.
