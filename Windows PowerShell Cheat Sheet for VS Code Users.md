# Windows PowerShell Cheat Sheet for VS Code Users

A practical cheat sheet for using **Windows PowerShell** in the VS Code integrated terminal.

If you're coming from Linux, macOS, Git Bash, or tutorials that use Bash syntax, some commands may look familiar but behave differently in PowerShell.

---

## Why PowerShell is Different

The VS Code terminal on Windows commonly uses **PowerShell**.

PowerShell has its own syntax and command aliases. A command that works perfectly in Bash may fail in PowerShell.

For example, this Bash-style command:

```bash
mkdir src/{components,pages,layouts}
```

can cause a PowerShell parser error.

Instead, PowerShell supports **comma-separated paths**:

```powershell
mkdir src/components, src/pages, src/layouts
```

Or navigate into the directory first:

```powershell
cd src
mkdir components, pages, layouts
```

> **⚠️ IMPORTANT:** PowerShell commands such as `rm` can permanently delete files and folders. Terminal deletions **bypass the Windows Recycle Bin**. Always verify the path before running a delete command.

---

# Quick Reference

| Action | PowerShell Command / Alias | Example |
|---|---|---|
| Create folders | `mkdir` | `mkdir src/components, src/pages` |
| Create files | `ni` | `ni App.jsx, index.css` |
| Delete files | `rm` | `rm App.jsx` |
| Delete folders | `rm -Recurse` | `rm -Recurse src` |
| Move files | `mv` | `mv App.jsx src/` |
| Move + rename | `mv` | `mv script.js src/app.js` |

> **💡 PowerShell is case-insensitive.** Commands such as `mkdir`, `MKDIR`, `Mkdir`, and `mKdIr` are treated the same. However, consistent lowercase syntax is recommended for readability.

---

# 1. Create Multiple Folders

## Recommended PowerShell Syntax

Use a comma-separated list:

```powershell
mkdir path/sub1, path/sub2
```

### Example

```powershell
mkdir src/components, src/pages, src/layouts
```

This creates:

```text
src/
├── components/
├── pages/
└── layouts/
```

---

## Create Multiple Folders After `cd`

You can also navigate to the parent directory first.

```powershell
cd src
mkdir components, pages, layouts
```

Result:

```text
src/
├── components/
├── pages/
└── layouts/
```

---

## ❌ Don't Use Bash Brace Expansion

Avoid:

```bash
mkdir src/{components,pages,layouts}
```

That syntax is associated with Bash-style brace expansion and should **not** be used for PowerShell folder creation.

Also avoid trying to work around it with quoted braces:

```powershell
mkdir "src/{components,pages,layouts}"
```

This does not create three folders.

### Use this instead:

```powershell
mkdir src/components, src/pages, src/layouts
```

---

# 2. Delete Folders with Contents

To delete a folder and everything inside it:

```powershell
rm -Recurse folder-name
```

### Example

```powershell
rm -Recurse src/components
```

This removes:

```text
components/
├── Header.jsx
├── Footer.jsx
└── Navigation.jsx
```

along with the `components` folder.

> **🚨 DANGER:** `rm -Recurse` permanently deletes the specified folder and its contents. The deleted files **do not go to the Windows Recycle Bin**.

---

## Safer Alternative: VS Code Explorer

If you want the deleted files to go through the normal Windows deletion workflow, use the **VS Code Explorer sidebar**.

1. Open the **Explorer** sidebar.
2. Right-click the file or folder.
3. Select **Delete**.
4. Confirm the deletion.

This is preferable when you want a **Recycle Bin-backed** deletion instead of permanent terminal deletion.

---

## Check Before Deleting

Before running a destructive command, inspect the current directory:

```powershell
pwd
```

Then list its contents:

```powershell
ls
```

Example:

```powershell
pwd
ls
```

Verify that you're in the correct directory before using:

```powershell
rm -Recurse folder-name
```

---

# 3. Create Multiple Files

PowerShell provides the `ni` alias for `New-Item`.

Use:

```powershell
ni file1.ext, file2.ext
```

### Example

```powershell
ni App.jsx, index.css, README.md
```

This creates:

```text
App.jsx
index.css
README.md
```

---

## What Does `ni` Mean?

`ni` is a PowerShell alias for:

```powershell
New-Item
```

Therefore, these commands are equivalent:

```powershell
ni App.jsx
```

and:

```powershell
New-Item App.jsx
```

The shorter alias is convenient when working quickly in the VS Code terminal.

---

## Create Multiple React Files

For example:

```powershell
ni Header.jsx, Footer.jsx, Navigation.jsx
```

Or:

```powershell
ni Home.jsx, About.jsx, Contact.jsx, NotFound.jsx
```

---

## Suppress Verbose Output

`New-Item` may display information about the files it creates.

If you don't want that output, use:

```powershell
ni App.jsx, index.css -Force | Out-Null
```

### Why this works

```text
-Force
   ↓
Allows the command to proceed when applicable

|
   ↓
Pipes the output

Out-Null
   ↓
Suppresses the displayed output
```

For multiple files:

```powershell
ni Header.jsx, Footer.jsx, Navigation.jsx -Force | Out-Null
```

---

# 4. Delete Multiple Files

Use `rm` followed by a comma-separated list:

```powershell
rm file1.ext, file2.ext
```

### Example

```powershell
rm test.txt, old-file.txt
```

This deletes both files.

> **⚠️ WARNING:** Terminal deletion is permanent and bypasses the Recycle Bin.

---

## Delete Files Using Wildcards

PowerShell supports wildcards.

For example:

```powershell
rm *.log
```

This removes all `.log` files in the current directory.

Example:

```text
logs/
├── error.log
├── server.log
├── debug.log
└── README.md
```

Running:

```powershell
rm *.log
```

targets:

```text
error.log
server.log
debug.log
```

but not:

```text
README.md
```

> **🚨 Be careful with wildcards.** `*.log` can match many files at once. Always check what you're about to delete.

---

## Ask for Confirmation

Use `-Confirm` when you want PowerShell to ask before deleting:

```powershell
rm *.log -Confirm
```

You can also use it with individual files:

```powershell
rm file1.txt, file2.txt -Confirm
```

This is a useful safety measure when you're unsure about the target.

---

# 5. Move Files

Use `mv` to move files:

```powershell
mv file1.ext, file2.ext destination/
```

### Example

```powershell
mv App.jsx, index.css src/
```

This moves both files into:

```text
src/
├── App.jsx
└── index.css
```

---

## Move a File Up One Directory

Use:

```text
..
```

to refer to the parent directory.

For example:

```powershell
mv App.jsx ..
```

If you're currently inside:

```text
src/pages/
```

the command moves `App.jsx` to:

```text
src/
```

### Example

Current location:

```text
project/
└── src/
    └── pages/
        └── App.jsx
```

Run:

```powershell
mv App.jsx ..
```

Result:

```text
project/
└── src/
    ├── App.jsx
    └── pages/
```

---

# 6. Move and Rename a File at the Same Time

PowerShell's `mv` can move a file while giving it a new name.

Example:

```powershell
mv script.js src/app.js
```

This does two things:

1. Moves `script.js` into `src/`
2. Renames it to `app.js`

Before:

```text
project/
├── script.js
└── src/
```

After:

```text
project/
└── src/
    └── app.js
```

---

# 7. Common PowerShell Aliases

PowerShell provides convenient aliases for many common commands.

## `mkdir`

Alias for creating directories:

```powershell
mkdir
```

Example:

```powershell
mkdir components, pages, layouts
```

---

## `ni`

Alias for:

```powershell
New-Item
```

Example:

```powershell
ni App.jsx, index.css
```

---

## `rm`

Alias for:

```powershell
Remove-Item
```

Example:

```powershell
rm old-file.txt
```

Delete a folder and its contents:

```powershell
rm -Recurse old-folder
```

---

## `mv`

Alias for:

```powershell
Move-Item
```

Example:

```powershell
mv App.jsx src/
```

---

# 8. Useful Navigation Commands

Although this cheat sheet focuses on file operations, these commands are useful when working in the VS Code terminal.

## Show Current Directory

```powershell
pwd
```

`pwd` displays your current working directory.

---

## List Files and Folders

```powershell
ls
```

or:

```powershell
dir
```

Example:

```powershell
ls
```

Output might look like:

```text
Mode                 Name
----                 ----
d----                 src
-a---                 package.json
-a---                 README.md
```

---

## Change Directory

```powershell
cd src
```

---

## Move Up One Directory

```powershell
cd ..
```

---

## Go to a Specific Directory

```powershell
cd path/to/project
```

On Windows, you can also use Windows-style paths:

```powershell
cd C:\Projects\my-app
```

---

# 9. Practical React Project Example

Suppose you start with:

```text
my-app/
└── src/
```

You can create a common React project structure with:

```powershell
cd src
mkdir components, layouts, pages
```

Then create the files:

```powershell
cd components
ni Header.jsx, Footer.jsx
```

Move to the layouts directory:

```powershell
cd ../layouts
ni Layout.jsx
```

Move to the pages directory:

```powershell
cd ../pages
ni Home.jsx, About.jsx, Contact.jsx, NotFound.jsx
```

The resulting structure:

```text
src/
├── components/
│   ├── Header.jsx
│   └── Footer.jsx
│
├── layouts/
│   └── Layout.jsx
│
└── pages/
    ├── Home.jsx
    ├── About.jsx
    ├── Contact.jsx
    └── NotFound.jsx
```

---

# 10. Bash vs PowerShell

If you're following a tutorial written for Linux or macOS, watch out for syntax differences.

| Task | Bash Example | PowerShell Example |
|---|---|---|
| Multiple directories | `mkdir src/{a,b,c}` | `mkdir src/a, src/b, src/c` |
| Multiple files | `touch a.js b.js` | `ni a.js, b.js` |
| Delete folder | `rm -rf folder` | `rm -Recurse folder` |
| Move file | `mv a.js src/` | `mv a.js src/` |
| Move up | `mv a.js ..` | `mv a.js ..` |

Some commands look identical because PowerShell provides aliases with familiar names. **Do not assume that Bash syntax works everywhere just because the command name looks the same.**

---

# 11. Safety Rules

Keep these rules in mind whenever you're using file commands in the VS Code terminal.

> **🚨 RULE 1 — Terminal deletion is permanent.**  
> `rm` / `Remove-Item` does not provide the same Recycle Bin workflow as deleting through Windows Explorer or the VS Code Explorer.

> **🚨 RULE 2 — Be careful with `-Recurse`.**  
> `rm -Recurse folder` removes the folder and its contents.

> **🚨 RULE 3 — Be careful with wildcards.**  
> Commands such as `rm *.log` can affect multiple files at once.

> **✅ RULE 4 — Use `-Confirm` when unsure.**  
> Example: `rm *.log -Confirm`

> **✅ RULE 5 — Check your location first.**  
> Use `pwd` and `ls` before running destructive commands.

> **✅ RULE 6 — Use the VS Code Explorer when you want Recycle Bin-backed deletion.**  
> Right-click the file or folder and choose **Delete**.

---

# 12. Quick Copy-Paste Cheat Sheet

## Create Folders

```powershell
mkdir src/components, src/pages, src/layouts
```

## Create Folders After Navigation

```powershell
cd src
mkdir components, pages, layouts
```

## Create Files

```powershell
ni App.jsx, index.css, README.md
```

## Create Files Without Output

```powershell
ni App.jsx, index.css, README.md -Force | Out-Null
```

## Delete a File

```powershell
rm file.txt
```

## Delete Multiple Files

```powershell
rm file1.txt, file2.txt
```

## Delete Matching Files

```powershell
rm *.log
```

## Delete With Confirmation

```powershell
rm *.log -Confirm
```

## Delete Folder and Contents

```powershell
rm -Recurse folder-name
```

## Move Files

```powershell
mv file1.ext, file2.ext destination/
```

## Move Up One Directory

```powershell
mv file.ext ..
```

## Move and Rename

```powershell
mv script.js src/app.js
```

## Check Current Directory

```powershell
pwd
```

## List Directory Contents

```powershell
ls
```

## Change Directory

```powershell
cd folder-name
```

## Move Up

```powershell
cd ..
```

---

# 13. The Four Commands to Remember

If you're working on a React project in VS Code, these four PowerShell commands cover many everyday file operations:

```powershell
# Create directories
mkdir components, pages, layouts

# Create files
ni Header.jsx, Footer.jsx, Layout.jsx

# Delete files/folders
rm old-file.txt
rm -Recurse old-folder

# Move files
mv App.jsx src/
```

### Mental Model

```text
mkdir → make directory
ni    → new item
rm    → remove item
mv    → move item
```

Once you remember these four commands, basic project file management in the Windows PowerShell terminal becomes much easier.

---

## Final Reminder

PowerShell is **not Bash**.

When a tutorial gives you a terminal command, first check which shell it targets.

For **Windows PowerShell**, prefer:

```powershell
mkdir src/components, src/pages, src/layouts
ni App.jsx, index.css
rm -Recurse old-folder
mv App.jsx src/
```

Avoid blindly copying Bash-specific syntax such as:

```bash
mkdir src/{components,pages,layouts}
```

And most importantly:

> **⚠️ Treat terminal `rm` commands as permanent. Check the path before deleting, especially when using `-Recurse` or wildcards.**