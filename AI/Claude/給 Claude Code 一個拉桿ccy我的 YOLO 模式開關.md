# 給 Claude Code 一個拉桿:`ccy`,我的 YOLO 模式開關

## 起源:那個又長又嚇人的 flag

寫 Claude Code 一段時間之後,你大概會遇到一個分水嶺:有些任務你需要它一個個跟你確認 ——「我要編輯這個檔案,可以嗎?」、「我要跑這個 bash 指令,可以嗎?」—— 確認得有理有據,擋下過好幾次蠢事。

但另外一些任務,你已經很清楚要幹嘛,例如:

- 在乾淨的 sandbox repo 裡讓它連續 refactor 二十個檔案
- 跑批次 codemod、整理一整個資料夾的檔名
- 讓 agent 跑長任務,你要去接小孩、洗碗、衝浪

這時候每三十秒被打斷一次 prompt confirmation,體感極差。Anthropic 給了一個逃生口:

```bash
claude --dangerously-skip-permissions
```

注意這個 flag 的名字 —— `dangerously`,危險地。命名本身就是設計文件:**我們知道你想用,但你要清楚自己在做什麼**。它會關掉所有 permission prompt,Claude Code 想跑什麼就跑什麼,不問你。

問題來了:這個 flag 長得實在不方便。每次要打那一大串,手會酸,而且打字打到 `dangerously` 心理壓力會莫名上升,有種「你確定?」的儀式感 —— 這其實是好的,但開發節奏被打斷也是事實。

## `ccy`:一個八行的 wrapper

最低需求的版本長這樣:

```bash
#!/usr/bin/env bash
# ccy - claude with --dangerously-skip-permissions injected
exec claude --dangerously-skip-permissions "$@"
```

放到 `~/.local/bin/ccy`,`chmod +x`,完事。

用法:

```bash
ccy                          # 純 YOLO session
ccy --resume                 # resume + YOLO
ccy -p "fix the failing tests"   # headless + YOLO
ccy --model opus              # 任何 flag 都能照常 pass
```

兩個寫法細節值得講:

**`exec`** 把當前 shell process 直接替換成 claude,不留一層多餘的 parent shell。Ctrl+C、SIGTERM 這些 signal 直接打到 claude,行為跟你裸跑 `claude` 完全一致 —— 沒有中介層,沒有 signal forwarding 的雷。

**`"$@"`** 一定要加雙引號。`$*` 或不加引號的 `$@` 在遇到帶空格的參數會被拆開,`ccy -p "fix the build with multiple words"` 會炸掉。雙引號版本保留參數邊界,你想接什麼 flag 它就乖乖傳什麼 flag。

## 但是 ——「方便」的代價

到這裡停下來,事情其實沒結束。`ccy` 是個輕量工具,但它把一個重量級決定變得太便宜了。

`--dangerously-skip-permissions` 危險不在於 Claude 本身會「叛變」,而在於 **你會忘記自己在哪個模式**。一旦這個 flag 變得輸入成本 = 三個字母,你會開始在不該開 YOLO 的地方開 YOLO:

- 在 production config 的 repo
- 在你 `$HOME` 根目錄(對,我見過)
- 在那種 working tree 還沒 commit、改壞了無法 `git reset` 救的時刻

這跟「為什麼 `rm -rf` 在大多數系統不會 alias 成 `rm`」是同一件事。**摩擦力本身就是一種 guardrail**。你把摩擦力拿掉,要記得在別的地方補回來。

## 加上 guardrail 的版本

我自己實際用的版本長這樣:

```bash
#!/usr/bin/env bash
set -euo pipefail

# 黑名單:這些目錄禁止 YOLO
BLOCKED_PATHS=(
  "$HOME"                          # 不要在 home root 跑
  "$HOME/work/production-configs"
  "/etc"
)

cwd="$(pwd)"
for blocked in "${BLOCKED_PATHS[@]}"; do
  if [[ "$cwd" == "$blocked" ]]; then
    echo "❌ Refuse to run YOLO mode in: $cwd" >&2
    echo "   Use plain 'claude' instead." >&2
    exit 1
  fi
done

# Git working tree dirty 時警告
if git rev-parse --git-dir >/dev/null 2>&1; then
  if [[ -n "$(git status --porcelain)" ]]; then
    echo "⚠️  Working tree is dirty. Stash or commit first? (y/N)" >&2
    read -r ans
    [[ "$ans" != "y" && "$ans" != "Y" ]] && exit 1
  fi
fi

echo "⚠️  YOLO mode enabled — cwd: $cwd" >&2
exec claude --dangerously-skip-permissions "$@"
```

加了三層 guard:

1. **絕對黑名單**:某些路徑直接拒跑,沒得商量。`$HOME` root 是經典陷阱,Claude 一個 `rm` 下去你會想哭。
2. **Git dirty check**:在 git repo 裡如果還有未 commit 的改動,先問一次。YOLO 跑歪了,有 commit 過的至少 `git reset --hard` 救得回來;沒 commit 的就真的沒了。
3. **視覺提示**:每次啟動都印一行 cwd,逼你看一眼「我現在在哪」。

這三層加總起來,大概可以擋掉 90% 的「靠我為什麼會在這裡跑 YOLO」時刻。

## 同場加映:`/exit` vs Ctrl+C

順帶提一個我自己踩過的點。YOLO session 結束時,**請用 `/exit`,不要 Ctrl+C**。

Ctrl+C 在 Claude Code 裡的第一個用途是「中斷當下正在跑的操作」—— 不是離開 session。要連按兩次或在 idle prompt 按,才會真的離開,而且是 abrupt shutdown,SessionEnd hook 可能來不及跑完。

`/exit`(或在空 prompt 按 Ctrl+D)是 graceful shutdown,session 紀錄、hook 收尾、pending tool 都會正確處理。

對 YOLO session 來說這特別重要,因為你可能掛了一堆 audit hook、notification hook 在記錄這次到底跑了什麼 —— 用 Ctrl+C 硬切的話,稽核資料會缺一截。

## 結語

`ccy` 本身是個八行 shell script,沒什麼了不起。但它代表一個我越來越相信的模式:**讓 LLM 的危險選項變得方便,同時用 thin deterministic layer 把護欄補回來**。

LLM 不會記得你不想在 `$HOME` 跑 `rm`。Shell script 會。

把判斷留給能 100% 重現的那一層,把創造力留給不能 100% 重現的那一層 —— 這大概是我這陣子寫 agent 周邊工具最一致的直覺。