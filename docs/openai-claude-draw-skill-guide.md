# OpenAI API Key 串接 Claude Code 生圖 Skill 教學

這份教學整理如何把 OpenAI API Key 安全存到本機，並在 Claude Code / VS Code 的 Claude 視窗中建立 `draw` Skill，讓 Claude 可以用 OpenAI Image API 生成圖片。

> 重要提醒：API Key 等同於付費鑰匙，請不要貼在 Claude、ChatGPT、GitHub、截圖或任何公開地方。本文會使用本機環境檔保存 Key。

---

## 0. 適用情境

適用：

- Windows 桌機 / 筆電
- macOS MacBook / iMac
- VS Code 內的 Claude Code 視窗
- 想讓 Claude Code 呼叫 OpenAI 生圖模型產生圖片

不適用：

- 只在 Claude.ai 網頁聊天視窗直接貼 API Key
- 把 API Key 寫進 GitHub 儲存庫
- 把 API Key 放進前端網頁程式碼

---

## 1. 整體流程

簡單說，整條串聯流程是：

1. 在 OpenAI Platform 建立 API Key。
2. 把 API Key 安全存到本機的 `.openai.env`。
3. 在電腦安裝 OpenAI Python 套件。
4. 在 Claude Code 建立全域 Skill：`draw`。
5. Claude Code 執行 `draw.py`。
6. `draw.py` 讀取 `.openai.env` 的 API Key。
7. 呼叫 OpenAI Image API 生成圖片。
8. 圖片存到 `slides/generated/` 或 `generated/`。

---

## 2. 建立 OpenAI API Key

到 OpenAI Platform：

```text
Settings → API keys → Create new secret key
```

建議設定：

| 欄位 | 建議設定 |
|---|---|
| Owned by | You |
| Name | Claude_Draw_OpenAI |
| Project | Default project 或自己的專案 |
| Permissions | 測試時先選 All，之後正式使用可改 Restricted |

建立後會出現 `sk-...` 開頭的 Key。

請立刻按 Copy，因為完整 Key 通常只會顯示一次。

---

## 3. 儲存 API Key：macOS

打開 Terminal，貼上以下指令：

```bash
read -s OPENAI_API_KEY
printf "\nOPENAI_API_KEY=%s\n" "$OPENAI_API_KEY" > ~/.openai.env
chmod 600 ~/.openai.env
unset OPENAI_API_KEY
```

執行後畫面會等待你輸入，但不會顯示文字。

這時候貼上剛剛複製的 `sk-...`，然後按 Enter。

檢查是否成功：

```bash
ls -l ~/.openai.env
```

看到類似這樣代表成功：

```bash
-rw-------  1 你的使用者名稱  staff  ... /Users/你的使用者名稱/.openai.env
```

---

## 4. 儲存 API Key：Windows 桌機 / 筆電

打開 PowerShell，建議用一般權限即可。

### 方式 A：手動輸入 Key

```powershell
$apiKey = Read-Host "請貼上 OpenAI API Key" -AsSecureString
$plainKey = [Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($apiKey))
"OPENAI_API_KEY=$plainKey" | Out-File -FilePath "$env:USERPROFILE\.openai.env" -Encoding utf8
attrib +h "$env:USERPROFILE\.openai.env"
Remove-Variable apiKey
Remove-Variable plainKey
```

檢查檔案是否存在：

```powershell
Test-Path "$env:USERPROFILE\.openai.env"
```

出現：

```powershell
True
```

就代表成功。

### Windows 檔案位置

```text
C:\Users\你的使用者名稱\.openai.env
```

---

## 5. 安裝 OpenAI Python 套件

### macOS

```bash
python3 -m pip install --upgrade openai
```

如果出現權限問題：

```bash
python3 -m pip install --user --upgrade openai
```

### Windows

PowerShell 輸入：

```powershell
py -m pip install --upgrade openai
```

如果 `py` 不能用，改用：

```powershell
python -m pip install --upgrade openai
```

---

## 6. 建立 Claude Code 全域 Skill

這個 Skill 的名稱叫：

```text
draw
```

### macOS 位置

```text
~/.claude/skills/draw/
```

### Windows 位置

```text
C:\Users\你的使用者名稱\.claude\skills\draw\
```

---

## 7. macOS 建立資料夾

```bash
mkdir -p ~/.claude/skills/draw
```

---

## 8. Windows 建立資料夾

PowerShell：

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\draw"
```

---

## 9. 建立 `SKILL.md`

檔案位置：

macOS：

```text
~/.claude/skills/draw/SKILL.md
```

Windows：

```text
C:\Users\你的使用者名稱\.claude\skills\draw\SKILL.md
```

內容：

```markdown
---
name: draw
description: OpenAI gpt-image-2 生圖技能。當使用者要求「畫一張」、「生一張圖」、「做一張圖」、「產生圖片」、「畫封面」、「畫插圖」、「畫示意圖」、「畫分鏡」等需要 AI 生成圖像的情境時，使用此技能呼叫本機 draw.py 產生圖片。預設使用 low 品質以節省成本，輸出至當前專案的 slides/generated/；若無 slides/，則輸出至 ./generated/。
---

# draw：OpenAI 生圖 Skill

## 用途

使用 OpenAI Image API 生成圖片，適合：

- 簡報插圖
- 海報草圖
- 封面圖
- 教學示意圖
- 分鏡概念圖
- 作品集視覺發想

## 使用方式

```bash
python3 ~/.claude/skills/draw/draw.py "一隻可愛的黑貓，扁平插畫風格" --name test
```

## 參數

- `prompt`：要畫什麼，必填
- `--size`：圖片尺寸，預設 `1024x1024`
- `--quality`：品質，預設 `low`
- `--n`：生成張數，預設 `1`
- `--name`：檔名前綴，預設 `image`
- `--outdir`：指定輸出資料夾

## 品質建議

- `low`：預設，適合練習、草圖、簡報、教學示意圖
- `medium`：需要更穩定細節時使用
- `high`：正式印刷、精細商業輸出再使用

不確定時使用 `low`，避免不必要的成本。

## 輸出位置

如果目前專案有 `slides/` 資料夾：

```text
slides/generated/
```

如果沒有：

```text
generated/
```

## 安全提醒

不要讀取、顯示或輸出 `.openai.env` 的內容。程式只需要在背景讀取 `OPENAI_API_KEY`。
```

---

## 10. 建立 `draw.py`

檔案位置：

macOS：

```text
~/.claude/skills/draw/draw.py
```

Windows：

```text
C:\Users\你的使用者名稱\.claude\skills\draw\draw.py
```

內容：

```python
"""
OpenAI gpt-image-2 生圖腳本

用法：
  macOS:
    python3 ~/.claude/skills/draw/draw.py "一隻可愛的黑貓，扁平插畫風格" --name test

  Windows:
    py %USERPROFILE%\.claude\skills\draw\draw.py "一隻可愛的黑貓，扁平插畫風格" --name test

API Key 讀取順序：
  1. 系統環境變數 OPENAI_API_KEY
  2. 目前資料夾 .env
  3. 使用者 home 裡的 .openai.env
"""

import os
import sys
import base64
import argparse
from pathlib import Path
from datetime import datetime

MODEL = "gpt-image-2"
DEFAULT_SIZE = "1024x1024"
DEFAULT_QUALITY = "low"
DEFAULT_N = 1


def load_env_from_file(path: Path):
    if not path.exists():
        return
    with open(path, encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if line and not line.startswith("#") and "=" in line:
                key, value = line.split("=", 1)
                os.environ.setdefault(key.strip(), value.strip().strip('"').strip("'"))


def load_env():
    load_env_from_file(Path.cwd() / ".env")
    load_env_from_file(Path.home() / ".openai.env")


def resolve_outdir(user_outdir):
    if user_outdir:
        return Path(user_outdir)

    cwd = Path.cwd()
    slides_dir = cwd / "slides"

    if slides_dir.exists():
        return slides_dir / "generated"

    return cwd / "generated"


def save_results(result, name, n, outdir):
    stamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    saved = []

    for i, item in enumerate(result.data):
        suffix = f"_{i + 1}" if n > 1 else ""
        out_path = outdir / f"{name}_{stamp}{suffix}.png"
        png_bytes = base64.b64decode(item.b64_json)
        out_path.write_bytes(png_bytes)
        saved.append(out_path)
        print(f"[OK] {out_path}")

    return saved


def draw(prompt, size, quality, n, name, outdir):
    from openai import OpenAI

    if not os.getenv("OPENAI_API_KEY"):
        print("錯誤：找不到 OPENAI_API_KEY。請確認 ~/.openai.env 或 C:/Users/你的使用者/.openai.env 是否存在。", file=sys.stderr)
        sys.exit(1)

    outdir.mkdir(parents=True, exist_ok=True)
    client = OpenAI()

    print(f"畫圖中：{MODEL}, size={size}, quality={quality}, n={n}", file=sys.stderr)
    print(f"輸出位置：{outdir}", file=sys.stderr)

    result = client.images.generate(
        model=MODEL,
        prompt=prompt,
        size=size,
        quality=quality,
        n=n,
    )

    return save_results(result, name, n, outdir)


def main():
    load_env()

    parser = argparse.ArgumentParser(description="OpenAI gpt-image-2 生圖工具")
    parser.add_argument("prompt", nargs="+", help="要生成的圖片描述")
    parser.add_argument("--size", default=DEFAULT_SIZE, help="圖片尺寸，例如 1024x1024、1536x1024、1024x1536")
    parser.add_argument("--quality", default=DEFAULT_QUALITY, choices=["low", "medium", "high", "auto"], help="圖片品質")
    parser.add_argument("--n", type=int, default=DEFAULT_N, help="生成張數")
    parser.add_argument("--name", default="image", help="輸出檔名前綴")
    parser.add_argument("--outdir", default=None, help="指定輸出資料夾")

    args = parser.parse_args()
    prompt = " ".join(args.prompt)
    outdir = resolve_outdir(args.outdir)

    draw(prompt, args.size, args.quality, args.n, args.name, outdir)


if __name__ == "__main__":
    main()
```

---

## 11. macOS 測試

```bash
python3 ~/.claude/skills/draw/draw.py "一隻可愛的黑貓，扁平插畫風格" --name test
```

成功時會看到類似：

```bash
[OK] /Users/你的使用者名稱/generated/test_20260601_145000.png
```

打開輸出資料夾：

```bash
open ./generated
```

如果是在有 `slides/` 的專案中：

```bash
open ./slides/generated
```

---

## 12. Windows 測試

PowerShell：

```powershell
py "$env:USERPROFILE\.claude\skills\draw\draw.py" "一隻可愛的黑貓，扁平插畫風格" --name test
```

成功時會看到類似：

```powershell
[OK] C:\Users\你的使用者名稱\generated\test_20260601_145000.png
```

打開輸出資料夾：

```powershell
explorer .\generated
```

如果是在有 `slides` 的專案中：

```powershell
explorer .\slides\generated
```

---

## 13. 在 Claude Code 視窗怎麼使用

在 VS Code 的 Claude Code 視窗輸入：

```text
請使用 draw skill 幫我畫一張可愛黑貓，扁平插畫風格。
```

或更完整一點：

```text
請使用 draw skill：
prompt：一張 SDGs 永續發展主題海報主視覺，結合科技、環境、未來感
size：1536x1024
quality：low
name：sdgs_poster
```

Claude Code 應該會執行類似：

```bash
python3 ~/.claude/skills/draw/draw.py "一張 SDGs 永續發展主題海報主視覺，結合科技、環境、未來感" --size 1536x1024 --quality low --name sdgs_poster
```

Windows 則會執行類似：

```powershell
py "$env:USERPROFILE\.claude\skills\draw\draw.py" "一張 SDGs 永續發展主題海報主視覺，結合科技、環境、未來感" --size 1536x1024 --quality low --name sdgs_poster
```

---

## 14. 常見錯誤排除

| 錯誤訊息 | 可能原因 | 解法 |
|---|---|---|
| `401 Invalid API key` | API Key 錯誤或沒讀到 | 重新建立 Key，重新寫入 `.openai.env` |
| `403 Organization must be verified` | OpenAI 組織尚未驗證 | 到 OpenAI Platform 的 Organization Verification 做 Individual 驗證 |
| `429 Rate limit` | 額度、速率或付款限制 | 到 Billing 檢查額度與付款方式 |
| `ModuleNotFoundError: openai` | 沒有安裝 OpenAI Python 套件 | 執行 `python3 -m pip install --upgrade openai` 或 `py -m pip install --upgrade openai` |
| 沒有產生圖片 | 輸出資料夾不同 | 檢查 `generated/` 或 `slides/generated/` |

---

## 15. 安全守則

請遵守：

- 不要把 API Key 貼到聊天視窗。
- 不要把 `.openai.env` 上傳到 GitHub。
- 不要截圖 API Key。
- 不要把 API Key 寫進 `draw.py`。
- 一個用途盡量一把 Key，方便日後停用。
- 如果懷疑 Key 外流，馬上刪掉舊 Key，重新建立新的。

---

## 16. 本次測試成果

測試 prompt：

```text
一隻可愛的黑貓，扁平插畫風格
```

成功產生圖片後，代表：

- OpenAI API Key 已可被本機程式讀取
- Python openai 套件可正常使用
- Claude Code 可以透過本機腳本執行生圖流程
- 輸出圖片可保存到 `generated/` 或 `slides/generated/`

---

## 17. 之後可以怎麼用

你可以直接對 Claude Code 說：

```text
請使用 draw skill 幫我畫一張課堂報告封面，主題是 AI 與永續發展，風格簡約、乾淨、有科技感。
```

或：

```text
請使用 draw skill 幫我做一張 YouTube 縮圖，主題是 OpenAI API 串接 Claude Code，風格明亮、教學感、文字區留白。
```

建議預設先使用：

```text
quality：low
```

等確定構圖和方向後，再考慮提高品質。
