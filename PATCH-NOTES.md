# Patch notes — claude/audit-clearbot-user-flow-Tbzl4

This branch is part of the Clearbot user-flow audit. The edits below
are already prepared and will land in a follow-up commit on this branch.

## `index.html` edits

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

- **`confirmDeleteAccessCode`** (~line 3550) — append a "This cannot be undone." sentence.

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

## Verification

After the HTML commit lands:
- [ ] Sign in as a non-allowlisted user → confirm the new unauthorized view.
- [ ] Delete each object type → confirm "This cannot be undone." shows.
- [ ] Open a client with no email set → confirm company name shows, not UUID.
- [ ] Open any modal → confirm the keyboard hint is in the footer.
