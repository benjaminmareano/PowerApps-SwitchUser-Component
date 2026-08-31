# Switch User for Power Apps

A canvas component that lets you view your app as any other user in your directory.

A user reports a bug you can't reproduce. Your app decides what to show based on who's signed in — and that's always you. This routes identity through a variable instead, so you can switch to their account and see exactly what they see.

Setup takes about ten minutes.

---

## What's in this repo

| File | Purpose |
|---|---|
| `People_Picker.zip` | Unmanaged solution containing the `People Picker` component library |
| `People_Picker.msapp` | The component library as a single file, for importing into a library you already maintain |
| `SwitchUser.yaml` | The complete demo screen — chip, banner, picker dialog, and profile card |
| `App.Formulas` | The `AppUser` type and the `BuildUser()` function |
| `App.OnStart` | Variable initialization |

All four sit at the root of the repo.

---

## Requirements

- A Power Apps environment where you can import solutions
- Edit access to the app you're adding this to
- The **Office 365 Users** connector (standard — no premium licence)

---

## Setup

Follow the steps in order. The sequence matters.

### 1. Import the solution

1. Go to **make.powerapps.com** and select your environment.
2. In the left pane, select **Solutions**.
3. Select **Import solution**.
4. Browse to `People_Picker.zip` and select **Next** → **Import**.
5. Wait for the import to finish.

This installs the **People Picker** component library into your environment. One import serves every app in that environment.

#### Alternative: add it to a library you already have

If you already maintain a component library, skip the solution import and pull the component straight into it.

1. Open your component library in Power Apps Studio.
2. In the **Tree view**, select the **Components** tab.
3. Select the **Import components** icon next to **New component**.
4. On the **Canvas** tab, select **Upload file**.
5. Choose `People_Picker.msapp`.
6. Select **cmpPeoplePicker**, then select **Import**.
7. Select **Save**, then **Publish**.

In Step 6 below, select your own library instead of **People Picker**.

### 2. Enable user-defined types

1. Open your app in Power Apps Studio.
2. On the command bar, select **Settings**.
3. Select **Updates** → the **New** tab.
4. Turn on **User-defined types and functions**.
5. Reopen the app if prompted.

`BuildUser()` returns a typed record, which needs this enabled.

### 3. Add the connector

1. In the left pane, select **Data**.
2. Select **Add data**.
3. Search for **Office 365 Users** and select it.

### 4. Add the App.Formulas code

1. In the Tree view, select **App**.
2. In the property dropdown above the formula bar, select **Formulas**.
3. Paste the contents of `App.Formulas`.

### 5. Add the App.OnStart code

1. With **App** still selected, choose **OnStart** in the property dropdown.
2. Paste the contents of `App.OnStart`.

Don't run it yet.

### 6. Add the component to your app

**Do this before pasting the YAML.**

1. In the left pane, select the **+** (Insert) icon.
2. Select **Get more components** at the bottom of the panel.
3. Under the **People Picker** component library, select **cmpPeoplePicker**.
4. Select **Import**.

The component is now available in your app. Skipping this step will break the paste in Step 7.

### 7. Paste the screen

1. Open `SwitchUser.yaml` and copy the entire contents.
2. In the Tree view, select the **Screens** tab.
3. Paste (Ctrl+V).

This creates `scrSwitchUser` with the identity chip, impersonation banner, picker dialog, live profile card, and the picker already placed and wired.

### 8. Verify the picker connection

The paste sets this up for you. Confirm it landed:

1. In the Tree view, expand `scrSwitchUser` → `grpPickerDialog` → `cntPicker`.
2. Select the `cmpPeoplePicker` instance inside it.
3. In the property dropdown, select **OnSelectUser** and check that it reads:

```
Set(varCurrentUser, BuildUser(UserId));
UpdateContext({ locShowPicker: false })
```

If the instance is missing, Step 6 was skipped — add the component, then paste the YAML again. If the formula is empty, paste it in. `UserId` is the event parameter the component passes.

### 9. Run OnStart

1. Right-click **App** in the Tree view.
2. Select **Run OnStart**.

The identity chip should now show your name and photo.

### 10. Test it

1. Select **Preview** (F5).
2. Select the identity chip in the top right.
3. Type at least two characters of a colleague's name and pause.
4. Select a name.
5. The picker closes, the banner appears, and the profile card updates.
6. Select **Switch back** to return.

---

## Wiring it into your own app

The demo screen proves the switch works. To make it do anything useful, replace every `User()` call in your app.

1. Select the **Search** box at the top of Power Apps Studio.
2. Search for `User(`.
3. Replace each result:

| Replace this | With this |
|---|---|
| `User().Email` | `varCurrentUser.Email` |
| `User().FullName` | `varCurrentUser.DisplayName` |
| `User().Image` | `varCurrentUser.Photo` |

4. Search again. The only remaining results should be the two in `App.OnStart`.

Any call you miss won't respond when you switch — that screen keeps showing your own data while everything else changes.

---

## Restricting it to testers

`App.OnStart` sets `varIsTester` to `true` so the chip appears while you're setting up. Replace it before you ship.

```
Set(varIsTester, !IsBlank(LookUp(AppTesters, Email = User().Email)));
```

Use a SharePoint list, a Dataverse table, or an environment variable — whichever you already have. Environment variables are cleanest, since the value differs per environment without a code change.

Use `User().Email` here, not `varCurrentUser.Email`. The gate must check the real signed-in account, or switching to a non-tester locks you out of switching back.

`grpIdentityChip` is already bound to `varIsTester`, so nothing else needs changing.

---

## Component reference

`cmpPeoplePicker` is a search-and-select panel with no dialog frame of its own — the screen YAML supplies that.

| Property | Type | Purpose |
|---|---|---|
| `OnSelectUser` | Event | Fires on selection. Provides `UserId`. |
| `SelectedUserId` | Output (Text) | ID of the last user picked. |
| `SearchPlaceholder` | Input (Text) | Hint text in the search box. |
| `MaxResults` | Input (Number) | Results returned. Default 20. |

Only `OnSelectUser` is required.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Red errors after importing the component | Connector was added afterwards. Add **Office 365 Users**, then close and reopen the app. |
| `AppUser` shows as unknown | User-defined types aren't enabled. See Step 2. |
| `BuildUser` shows as unknown | Code was pasted into **OnStart** instead of **Formulas**. |
| YAML paste fails or references an unknown control | The component wasn't added first. Do Step 6, then paste again. |
| Chip shows "Loading…" | **Run OnStart** wasn't executed. |
| Chip isn't visible | `varIsTester` is false. |
| Picker shows no results | Type at least two characters and pause. |
| Nothing happens on selection | `OnSelectUser` is empty. See Step 8. |
| Picker doesn't close after selecting | The `UpdateContext` line is missing from `OnSelectUser`. |
| Photos show as initials for users who have one | Z-order. Select `imgCardPhoto` → **Reorder** → **Bring to front**. Repeat for `imgChipPhoto`. |
| Some screens don't change | `User()` calls remain. |

---

## What this does and does not change

**Changes:** what the app displays, what it filters on, which controls and screens appear, and what forms default to.

**Does not change:** your Dataverse security roles, your record permissions, who is recorded as the author on saved records, or which account Power Automate flows run under.

Use it to reproduce app behavior. Use a real test account to reproduce permission errors.

---

## Updates

When a new version of the component library is published, apps consuming it show an update prompt. Updates aren't applied automatically — you choose when to take them.
