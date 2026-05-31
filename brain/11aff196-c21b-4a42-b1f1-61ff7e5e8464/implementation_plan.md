# Goal Description

Implement Issue #428: Keyboard Shortcuts for Power Users by creating a reusable hook, setting up a global shortcut overlay, and adding scoped keyboard navigation and actions on specific pages. The goal is to enhance power user workflows without disrupting accessibility or adding new third-party dependencies.

## User Review Required

- **Grants Filter Input**: The issue mentions `f focuses filters/search on grants page`. Currently, there is no `SearchInput` on `/grants`. I plan to add a simple client-side `SearchInput` filter above the grants list to fulfill this requirement, preserving existing business logic while giving the `f` shortcut a target.
- **Vote Confirmation Flow**: The issue mentions `Open existing rejection confirmation` and `Trigger existing approval confirmation flow`. Currently, clicking `Approve`/`Reject` in `VotePanel.tsx` directly calls `vote(true)` / `vote(false)` without a `ConfirmationDialog`. I will add a `ConfirmationDialog` in `VotePanel.tsx` for these actions to fulfill the "confirmation flow" requirement, and ensure both buttons and shortcuts trigger it.

## Proposed Changes

### Phase 1 — Build Reusable Keyboard Shortcut Hook
#### [NEW] `hooks/useKeyboardShortcuts.ts`
- Implement a custom hook `useKeyboardShortcuts` accepting a `ShortcutDefinition[]`.
- Maintain a pending key sequence buffer (e.g. `["g", "g"]`), resetting after 1000ms.
- Guard against firing when the user is focused in `INPUT`, `TEXTAREA`, `SELECT`, or `contenteditable` elements.
- Properly add/remove global `keydown` event listeners and manage dependencies to prevent duplicates.

---

### Phase 2 & 3 — Create Keyboard Shortcuts Overlay and Register Global Shortcuts
#### [NEW] `components/ui/KeyboardShortcutsOverlay.tsx`
- Build a global overlay displaying the mapping of shortcuts.
- Reuse the visual pattern from `ConfirmationDialog.tsx` (using standard semantic dialog tags, a fixed backdrop, etc.).
- Allow closing via `Esc` or clicking the backdrop.

#### [MODIFY] `app/providers.tsx`
- Render the `KeyboardShortcutsOverlay`.
- Use `useKeyboardShortcuts` here to register global navigation sequences (`g g`, `g d`, `g r`, `g l`, `g c`, `g s`).
- Add `/` shortcut to either focus an existing `input[type="search"]` or push `/search` to the router.
- Add `?` shortcut to toggle the overlay.

---

### Phase 4 — Grants List Shortcuts
#### [MODIFY] `app/grants/page.tsx`
- Add a client-side `SearchInput` so the `f` shortcut can focus it.
- Track `focusedGrantIndex` state for navigating up and down the list.
- Use `useKeyboardShortcuts` to register `f` (focus search), `j` (next card), `k` (previous card), and `Enter` (navigate to the focused grant).
- Add visual indicator (`ring-2 ring-accent-primary` or similar) to the focused grant card.

---

### Phase 5 — Grant Detail Shortcuts
#### [MODIFY] `app/grants/[id]/GrantDetailClient.tsx`
- Use `useKeyboardShortcuts` for `F` (Fund Grant — sets `fundModalOpen(true)`) and `h` (History — navigates to `/grants/[id]/history`).

---

### Phase 6 — Milestone Detail Shortcuts
#### [MODIFY] `components/milestones/VotePanel.tsx`
- Integrate `ConfirmationDialog` for approve and reject actions.
- Use `useKeyboardShortcuts` to listen for `a` and `r`. 
- Ensure these shortcuts only trigger the confirmation flow if the user is a reviewer, the milestone isn't fully approved, and no transaction is already in-flight.

---

### Phase 7 — Add Tests
#### [NEW] `tests/hooks/useKeyboardShortcuts.test.ts`
- Write tests confirming single/multi-key sequences, the 1000ms timeout behavior, and that events are ignored on input elements.

## Verification Plan

### Automated Tests
- Run `npm run test` (Vitest) to ensure the newly added `useKeyboardShortcuts.test.ts` passes correctly.

### Manual Verification
- Start `npm run dev` in the terminal.
- Test typing globally to ensure no inputs are hijacked by the new hook.
- Press `?` to toggle the help overlay.
- Test all global navigation keys (`g g` to grants list, etc.).
- On `/grants`, verify `j`, `k`, `f`, and `Enter` correctly select and navigate.
- On a Grant Details page, press `F` to open the funding modal and `h` to navigate to history.
- On a Milestone page (as a simulated reviewer if possible), press `a` and `r` to ensure the correct confirmation modals open.
