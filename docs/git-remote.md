# Git Remote Notes

当前项目准备从旧名字 `LocationGET` 逐步改成 `AssetTracker`。

## 当前远程地址

```text
origin  https://github.com/yrr-create/AssetTracker.git
```

如果 GitHub 上还没有创建或改名这个仓库，`git push` 会失败。先在 GitHub 网页端把仓库创建出来，或者把原来的 `LocationGET` 仓库在 Settings 里重命名为 `AssetTracker`。

## PowerShell 里改本地文件夹名

不要在 `G:\Personalportfolio\LocationGET` 里面直接重命名当前目录。先退到父目录：

```powershell
Set-Location G:\Personalportfolio
Rename-Item .\LocationGET .\AssetTracker
Set-Location G:\Personalportfolio\AssetTracker
```

`cd /d` 是 CMD 语法，PowerShell 里不要这样写。

## 如果远程还叫 LocationGET

如果你暂时还想推到旧仓库，可以改回：

```powershell
git remote set-url origin https://github.com/yrr-create/LocationGET.git
git push origin main
```

如果 GitHub 仓库已经改名成 AssetTracker，就保持：

```powershell
git remote set-url origin https://github.com/yrr-create/AssetTracker.git
git push origin main
```
