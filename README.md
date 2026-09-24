# CrewManager-LAN-Releases

Public release channel for **CrewManager-LAN (SHIP DESK)**.

The in-app updater (`UpdateService.cs`) checks this repository's **latest Release** anonymously and downloads the attached portable build ZIP. That is why this repo is **public** while the source code repo stays private.

---

## How the in-app updater works

1. On "Check for updates", the app calls `https://api.github.com/repos/saman1ahmadi/CrewManager-LAN-Releases/releases/latest`.
2. It reads the Release `tag_name` and the first `.zip` asset's `browser_download_url`.
3. If the release version is **newer** than the running app version, it downloads the ZIP, extracts it, and swaps the files over the current install using a helper script, then restarts.

For this to work, every release must satisfy the rules below.

---

## Publishing a new release (manual, on Windows)

> These steps must be done by a human on a Windows machine with .NET installed. They cannot be automated from here (no create-release / asset-upload API tool, and no build environment).

### 1. Bump the version in code
In the **private** repo `saman1ahmadi/CrewManager-LAN`, edit `src/CrewManager.Client/CrewManager.Client.csproj`:

```xml
<Version>1.1.1</Version>
```

The release tag must be **higher** than the version currently shipped. Commit and push.

### 2. Build a portable (self-contained) release
From the client project folder:

```powershell
dotnet publish src/CrewManager.Client/CrewManager.Client.csproj -c Release -r win-x64 --self-contained true /p:PublishSingleFile=false -o publish
```

### 3. Create the ZIP with files at the ROOT
Zip the **contents** of the `publish` folder, not the folder itself. `CrewManager.Client.exe` must sit at the top level of the ZIP:

```
CrewManager-LAN-1.1.1.zip
├── CrewManager.Client.exe   <-- at ZIP root
├── CrewManager.Client.dll
├── ClosedXML.dll
└── ... (all other publish output)
```

### 4. Create the GitHub Release
- Go to **Releases → Draft a new release** in this repo.
- **Tag:** `v1.1.1` (must match/exceed the csproj `<Version>`).
- **Target:** default branch is fine (assets are what matter).
- **Attach** the ZIP from step 3 as a release asset.
- Publish. It becomes "latest" automatically.

### 5. Verify
Open the running app → **Check for updates**. It should detect `v1.1.1`, download, and update.

---

## Rules checklist
- [ ] csproj `<Version>` bumped and committed in the private repo.
- [ ] Release tag is strictly higher than the previously shipped version.
- [ ] Exactly one `.zip` asset attached (the updater grabs the first `.zip`).
- [ ] `CrewManager.Client.exe` is at the ZIP **root**, not inside a subfolder.
- [ ] App is installed in a **writable** folder (not `Program Files` unless run as admin) so the updater can overwrite files.
