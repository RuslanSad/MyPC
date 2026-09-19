# 💻 MyPC — Полная настройка с нуля (Mac + Windows)

> Один файл, чтобы после сброса ПК/Мака восстановить всё окружение: инструменты, терминал, конфиги.
> Порядок: сверху вниз. Копируй команды блоками.
>
> **Стек:** Git · Go · Python · PostgreSQL · Redis · Kafka · Docker · Kubernetes · PyCharm · VSCode · Claude · Obsidian · красивый терминал.

---

# 🍎 macOS

## 1. Homebrew (пакетный менеджер — ставится первым)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# после установки добавить в PATH (Apple Silicon):
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

## 2. Инструменты разработки (CLI)
```bash
brew install git gh go python@3.12 postgresql@16 redis kafka \
  kubectl minikube helm
```
Запуск сервисов (БД поднимаются в фоне):
```bash
brew services start postgresql@16
brew services start redis
brew services start kafka
```

## 3. GUI-приложения
```bash
brew install --cask \
  pycharm \
  visual-studio-code \
  docker \
  obsidian \
  claude \
  iterm2 \
  font-meslo-lg-nerd-font
```
> `docker` (cask) = Docker Desktop, в нём же включается Kubernetes галочкой в настройках.
> `claude` (cask) = десктоп-приложение Claude.

## 4. Красивый терминал (oh-my-zsh + Powerlevel10k)
```bash
# плагины и утилиты
brew install powerlevel10k zsh-autosuggestions zsh-syntax-highlighting \
  fzf eza bat zoxide fd vim

# oh-my-zsh
export RUNZSH=no CHSH=no KEEP_ZSHRC=yes
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# тема Powerlevel10k (шаблон rainbow)
cp /opt/homebrew/opt/powerlevel10k/share/powerlevel10k/config/p10k-rainbow.zsh ~/.p10k.zsh
```
Потом положить `~/.zshrc` из раздела [Конфиги](#📄-конфиги-mac) ниже и выполнить `source ~/.zshrc`.
Тонкая настройка вида prompt (по желанию): `p10k configure`.

### Установка iTerm2 (Mac)
```bash
brew install --cask iterm2
```

### Шрифт в iTerm2
iTerm2 → Settings → Profiles → Text → Font → **MesloLGS Nerd Font Mono**, размер 14.
(Или импортировать готовый профиль — см. [iTerm-профиль](#iterm2-профиль) ниже.)

## 5. Claude Code (CLI в терминале)
```bash
curl -fsSL https://claude.ai/install.sh | bash
```
Бинарник ставится в `~/.local/bin/claude` (в PATH его добавляет наш `.zshrc`).
Первый запуск — `claude`, залогиниться через браузер.

## 6. Obsidian Local REST API (связь Claude ↔ Obsidian)
1. Obsidian → Settings → Community plugins → Browse → **Local REST API** → Install → Enable.
2. Открыть настройки плагина, скопировать **новый API Key** (после переустановки он всегда новый!).
3. Вставить ключ в `~/.zshrc` (переменная `OBSIDIAN_API_KEY`, см. конфиг ниже).
4. Проверка:
```bash
curl -sk -H "Authorization: Bearer $OBSIDIAN_API_KEY" https://127.0.0.1:27124/ | grep authenticated
```

---

## 📄 Конфиги (Mac)

### `~/.zshrc`
> ⚠️ `OBSIDIAN_API_KEY` вставь свой новый после переустановки плагина.
```zsh
# --- Instant prompt (Powerlevel10k) — держим в начале файла ---
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

# --- Сброс метки вложенной сессии Claude (чтобы сохранялась история) ---
unset CLAUDE_CODE_CHILD_SESSION

# --- Obsidian Local REST API ---
export OBSIDIAN_API_KEY="ВСТАВЬ_СВОЙ_КЛЮЧ"
export OBSIDIAN_BASE_URL="https://127.0.0.1:27124"
export OBSIDIAN_PROJECTS_FOLDER="Projects"

# --- PATH: пользовательские бинарники (claude и др.) ---
export PATH="$HOME/.local/bin:$PATH"

# --- Oh My Zsh ---
export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME=""
zstyle ':omz:update' mode auto
plugins=(git sudo extract colored-man-pages command-not-found)
source "$ZSH/oh-my-zsh.sh"

# --- Powerlevel10k ---
source /opt/homebrew/share/powerlevel10k/powerlevel10k.zsh-theme

# --- Автоподсказки по истории (→ чтобы принять) ---
source /opt/homebrew/share/zsh-autosuggestions/zsh-autosuggestions.zsh
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE="fg=#7a7a7a"

# --- Подсветка синтаксиса (последней) ---
source /opt/homebrew/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

# --- fzf: Ctrl+R история, Ctrl+T файлы, Alt+C cd ---
if command -v fzf >/dev/null; then
  source <(fzf --zsh)
  export FZF_DEFAULT_OPTS="--height 40% --layout=reverse --border --info=inline"
fi

# --- zoxide: умный cd ---
command -v zoxide >/dev/null && eval "$(zoxide init zsh)"

# --- История ---
HISTSIZE=50000
SAVEHIST=50000
setopt HIST_IGNORE_ALL_DUPS HIST_IGNORE_SPACE SHARE_HISTORY AUTO_CD

# --- Автодополнение ---
autoload -Uz compinit && compinit
zstyle ':completion:*' menu select
zstyle ':completion:*' matcher-list 'm:{a-zA-Z}={A-Za-z}'
zstyle ':completion:*' list-colors "${(s.:.)LS_COLORS}"

# --- Алиасы ---
alias ls='eza --icons --group-directories-first'
alias ll='eza -lah --icons --group-directories-first --git'
alias lt='eza --tree --level=2 --icons'
alias cat='bat --style=auto'
alias grep='grep --color=auto'
alias cls='clear'
alias ..='cd ..'
alias ...='cd ../..'
alias reload='source ~/.zshrc'

# --- p10k конфиг ---
[[ -f ~/.p10k.zsh ]] && source ~/.p10k.zsh
```

### `~/.claude/CLAUDE.md` (глобальные инструкции — ответы на русском)
```markdown
# Глобальные инструкции

## Язык
- Всегда отвечай на русском языке — все объяснения, комментарии, пояснения к коду.
- Технические термины, команды, флаги, названия функций/переменных оставляй на английском.
- Сообщения git-коммитов пиши на русском, если не попросили иначе.
```

### `~/.claude/settings.json` (модель + мощность + тема)
```json
{
  "model": "opus",
  "effortLevel": "high",
  "theme": "dark"
}
```
> `"model": "opus"` = самая свежая Opus. Для конкретной версии: `"claude-opus-4-8"`.
> Уровни мощности: low / medium / high / xhigh (менять на лету — `/effort`).

### iTerm2-профиль
Быстрее всего — вручную выставить шрифт **MesloLGS Nerd Font Mono** (Settings → Profiles → Text).
Тёмная схема цветов на вкус: iTerm2 → Settings → Profiles → Colors → Color Presets.

**Сделать профиль дефолтным (чтобы шрифт применялся к каждой новой сессии):**
iTerm2 → Settings (Cmd+,) → Profiles → выбрать свой профиль слева → внизу **Other Actions ▾ → Set as Default**.
> Менять дефолт через `defaults write` бесполезно: iTerm перезаписывает настройки при выходе. Только через меню.

---

# 🪟 Windows

> Homebrew не нужен. Используем встроенный **winget** (в Windows 10/11 из коробки).
> Все команды — в **PowerShell от администратора**.

## 1. Инструменты разработки (CLI)
```powershell
winget install --id Git.Git -e
winget install --id GitHub.cli -e
winget install --id GoLang.Go -e
winget install --id Python.Python.3.12 -e
winget install --id PostgreSQL.PostgreSQL.16 -e
winget install --id Kubernetes.kubectl -e
winget install --id Kubernetes.minikube -e
winget install --id Helm.Helm -e
```

## 2. GUI-приложения
```powershell
winget install --id JetBrains.PyCharm.Community -e
winget install --id Microsoft.VisualStudioCode -e
winget install --id Docker.DockerDesktop -e
winget install --id Obsidian.Obsidian -e
winget install --id Microsoft.WindowsTerminal -e
# Claude Desktop — id может отличаться, проверь:
winget search claude
winget install --id Anthropic.Claude -e
```

## 3. Redis, Kafka, Docker на Windows
Redis и Kafka нативно под Windows живут плохо — держим их в **Docker** (проще всего) или в **WSL2**.
```powershell
# Docker Desktop уже поставлен выше. Включи WSL2 backend в его настройках.
# Redis в контейнере:
docker run -d --name redis -p 6379:6379 redis
# Kafka (KRaft, один узел) в контейнере:
docker run -d --name kafka -p 9092:9092 apache/kafka:latest
```
Kubernetes — включается галочкой в Docker Desktop → Settings → Kubernetes.

## 4. WSL2 (Linux-окружение внутри Windows — рекомендуется для Go/Python)
```powershell
wsl --install
# после перезагрузки поставится Ubuntu; внутри неё можно повторить Linux-команды,
# включая oh-my-zsh точно как на Mac.
```

## 5. Красивый терминал на Windows (Windows Terminal + Oh My Posh)
Аналог oh-my-zsh + Powerlevel10k для PowerShell:
```powershell
winget install --id JanDeDobbeleer.OhMyPosh -e
winget install --id ajeetdsouza.zoxide -e
winget install --id junegunn.fzf -e
# Nerd Font:
oh-my-posh font install Meslo
# модули PowerShell (иконки, история, fuzzy):
Install-Module -Name Terminal-Icons -Repository PSGallery -Force
Install-Module -Name PSFzf -Repository PSGallery -Force
```
Затем в **Windows Terminal** → Settings → профиль PowerShell → Appearance → Font face = **MesloLGM Nerd Font**.

## 6. Claude Code (CLI) на Windows
```powershell
# через PowerShell-инсталлятор:
irm https://claude.ai/install.ps1 | iex
```
> Если команда недоступна — установить внутри **WSL2** тем же способом, что на Mac:
> `curl -fsSL https://claude.ai/install.sh | bash`

---

## 📄 Конфиги (Windows)

### PowerShell `$PROFILE` (аналог .zshrc)
Открыть: `notepad $PROFILE` (если файла нет — `New-Item -Path $PROFILE -Type File -Force`).
```powershell
# --- Красивый prompt ---
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\paradox.omp.json" | Invoke-Expression

# --- Иконки в ls ---
Import-Module Terminal-Icons

# --- Автоподсказки по истории (как серый текст) ---
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle InlineView
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete

# --- fzf: Ctrl+R поиск по истории, Ctrl+T файлы ---
Import-Module PSFzf
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' -PSReadlineChordReverseHistory 'Ctrl+r'

# --- zoxide: умный cd ---
Invoke-Expression (& { (zoxide init powershell | Out-String) })

# --- Obsidian Local REST API ---
$env:OBSIDIAN_API_KEY = "ВСТАВЬ_СВОЙ_КЛЮЧ"
$env:OBSIDIAN_BASE_URL = "https://127.0.0.1:27124"
$env:OBSIDIAN_PROJECTS_FOLDER = "Projects"

# --- Алиасы ---
Set-Alias ll Get-ChildItem
```

### `~/.claude/CLAUDE.md` и `settings.json`
На Windows лежат в `C:\Users\<имя>\.claude\` — содержимое **точно такое же**, как в Mac-разделе выше (скопировать оттуда).

---

# 🧩 VSCode — расширения для Go (Mac и Windows)

Официальное расширение Go + связанные (устанавливаются из терминала):
```bash
code --install-extension golang.go
code --install-extension ms-vscode.makefile-tools
code --install-extension zxh404.vscode-proto3
```
После установки в VSCode: `Cmd/Ctrl+Shift+P` → **Go: Install/Update Tools** → отметить все (gopls, dlv, staticcheck и т.д.).

### Настройки Go в VSCode (`settings.json`)
Открыть: `Cmd/Ctrl+Shift+P` → **Preferences: Open User Settings (JSON)**.
```json
{
  "go.gopath": "/Users/ruslan/go",
  "go.useLanguageServer": true,
  "go.toolsManagement.autoUpdate": true,
  "go.lintTool": "staticcheck",
  "[go]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "golang.go"
  }
}
```
> `go.gopath` — путь к GOPATH. По умолчанию Go использует `~/go`:
> - **Mac:** `/Users/ruslan/go`
> - **Windows:** `C:\\Users\\<имя>\\go`
> Проверить текущий: `go env GOPATH`.

---

# 🔧 Общие заметки

## Obsidian — как перенести настройки
Все настройки, плагины и хоткеи Obsidian хранятся в папке **`.obsidian`** внутри хранилища.
Чтобы восстановить один в один — просто скопировать папку `.obsidian` в новое хранилище (например через облако/git).

## После установки Claude Code — что настроить
- `~/.claude/CLAUDE.md` — ответы на русском (см. выше).
- `~/.claude/settings.json` — модель Opus + effort high.
- Переменные Obsidian в профиле терминала — связь Claude ↔ Obsidian.

## Порядок восстановления (кратко)
1. Пакетный менеджер (Homebrew / winget)
2. CLI-инструменты + сервисы (БД, Docker)
3. GUI-приложения
4. Красивый терминал + конфиги
5. Claude Code + логин
6. Obsidian + Local REST API + ключ

---
*Файл ведётся вместе с Claude. Обновляй при добавлении новых инструментов.*
