# Base Install
## clash
- [download clash](https://github.com/clash-verge-rev/clash-verge-rev/releases)
- directory: D:\Software\Install\Clash Verge
- Import link

## telegram
-  [download](https://telegram.org/)
-  directory: C:\Program Files\Telegram Desktop

## VS Studio
- [download Community version](https://visualstudio.microsoft.com/zh-hans/?icid=SSM_AS_VisualStudio)
- select: C#, C++, Python
- directory: C:\Program Files\Microsoft Visual Studio

## VS code
- [download VS code](https://code.visualstudio.com/Download)
- directory: C:\Program Files\Microsoft VS Code

## Git
- [download Git](https://git-scm.com/downloads/win)
- directory: C:\Program Files\Git
- Install settings: defalut
- settings:
  - git config --global user.email "xxxxxxx@163.com"
  - git config --global user.name "xxxxx_x411"

## Go
- [download Go](https://studygolang.com/dl)
- directory: C:\Program Files\Go
- evn
  - GOPATH: D:\Projects
  - Path: C:\Program Files\Go\bin
> go env <br>
> go env -w GOPROXY=https://goproxy.cn,direct

## nodejs
- [download nodejs](https://nodejs.org/zh-cn/download)
- directory: C:\Program Files\
> where node <br>
> node -v <br>
> npm -v <br>
> npx -v <br>

## Visual Studio
- [download Visual Studio2022](https://visualstudio.microsoft.com/zh-hans/vs/)
- directory: C:\Program Files\Microsoft Visual Studio\2022\Community
- select:
  - .NET Desktop APP
  - Python
  - C++ Desktop APP
  - tools: Data Store, Visual Studio extension development

---

# Program
## [DCEDataFetcher](https://github.com/lxex/DCEDataFetcher)
> cd D:\projects\DCEDataFetcher\DCEDataFetcher<br>
> dotnet tool install --global Microsoft.Playwright.CLI<br>
> dotnet add package Microsoft.Playwright
> playwright install<br>
