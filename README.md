# dsh-app-launcher

把 DSH Web GUI 变成"桌面应用":以独立应用窗口打开,关闭窗口即优雅退出整个
DSH 进程(`appExit`),npx/cmd 包装进程随之自动结束;Windows 下默认还会在桌面
自动生成 `DSH.lnk` 一键启动快捷方式。

## 行为

1. web 服务器绑定成功(`webServer` 服务出现)后,用 **专用浏览器配置目录**
   (`<DSH_HOME>/app-window/<browser>`)打开 Chrome/Edge 应用窗口,不影响日常浏览器;
2. 关闭该窗口 → 插件调用 `appExit(0)` → DSH 进程优雅退出(先 dispose 整棵树);
3. Windows: `createShortcut`(默认开启)自动生成桌面快捷方式(详见下方)。

## 安装

需要 pnpm:`npm i -g pnpm` 或 `corepack enable`。

### 从 GitHub 安装(推荐)

```bash
dsh plugin --profile web add github:Alvin-Somedo/dsh-app-launcher
```

### 从 npm 安装(还未 publish)

```bash
dsh plugin --profile web add dsh-app-launcher
```

安装后**下次启动生效** `dsh web`(或 `npx @deepseek-ai/dsh web`)生效。

## 配置

在 `~/.dsh/profiles/web/cordis.patch.yml` 里按 id 覆盖(**不要写 `insert`,插件行由 bundle 自带**):

```yaml
- id: app-launcher
  config:
    enabled: true       # false = 完全停用
    openOnStart: true   # 启动时打开窗口
    exitOnClose: true   # 关窗即退出 DSH
    browser: auto       # auto | chrome | edge
    # profileDir: C:\custom\profile   # 自定义浏览器配置目录
    createShortcut: true  # 默认开启:启动时自动在桌面创建"隐藏启动 DSH"快捷方式(仅 Windows);false = 关闭
    # shortcutName: DSH        # 快捷方式名(不含 .lnk),默认 DSH
    # launcherDir: ...         # 生成的 start-dsh.vbs / dsh.ico 存放目录,默认 <DSH_HOME>/launcher
    # desktopDir: ...          # 快捷方式生成位置,默认真实桌面
```

`createShortcut`(默认开启)会生成:

- `<DSH_HOME>/launcher/start-dsh.vbs` — 隐藏启动 `npx --yes @deepseek-ai/dsh web`;
- `<DSH_HOME>/launcher/dsh.ico` — 内置的 DSH 图标;
- 桌面 `DSH.lnk` — 指向 wscript + 上面的 vbs。

已存在的文件/快捷方式**不会被覆盖**(生成时会跳过)。桌面快捷方式只负责"不打开终端也能启动";窗口打开与关窗退出仍由插件完成。

## 卸载

```bash
dsh plugin --profile web remove dsh-app-launcher
```

或手动:从 `~/.dsh/profiles/web/package.json` 的 `dsh.profile.bundles` 移除包名,
删除 `~/.dsh/profiles/node_modules/dsh-app-launcher`。

> 升级插件:`dsh plugin --profile web update`(或 remove 后重新 add 新标签)。
