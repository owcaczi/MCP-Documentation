# Konfiguracja MCP w Claude Desktop — Windows

> Przewodnik instalacji i uruchomienia na nowym PC  
> Opracowany: Maj 2026

---

## 1. Ważna uwaga — Claude Desktop i SSH

> ⚠️ **Claude Desktop na Windows NIE obsługuje MCP serwerów przez SSH.**  
> Mimo że SSH samo działa poprawnie, Claude Desktop zamyka połączenie zaraz po starcie. Jest to znany bug. Rozwiązanie: uruchamiaj MCP serwery **lokalnie na Windows**.

---

## 2. Wymagania wstępne

- Claude Desktop (wersja 1.5354.0 lub nowsza)
- Node.js zainstalowany na Windows
- Windows 10/11

### Instalacja Node.js przez Chocolatey

Otwórz PowerShell jako Administrator:

```powershell
# Zainstaluj Chocolatey
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Zainstaluj Node.js
choco install nodejs-lts
```

> Zamknij i otwórz PowerShell ponownie po instalacji.

---

## 3. Instalacja MCP Filesystem (lokalny)

### Krok 1 — Włącz skrypty PowerShell

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Potwierdź literą `Y`.

### Krok 2 — Zainstaluj pakiet MCP

```powershell
npm install -g @modelcontextprotocol/server-filesystem
```

---

## 4. Konfiguracja Claude Desktop

### Krok 1 — Znajdź plik konfiguracyjny

```powershell
Get-ChildItem "$env:LOCALAPPDATA\Packages" | Where-Object { $_.Name -like "*Claude*" }
```

Plik konfiguracyjny znajduje się w:

```
C:\Users\<TWÓJ_USER>\AppData\Local\Packages\Claude_<ID>\LocalCache\Roaming\Claude\claude_desktop_config.json
```

### Krok 2 — Edytuj plik konfiguracyjny

Wklej poniższą zawartość (zamień `<TWÓJ_USER>` na swoją nazwę użytkownika):

```json
{
  "preferences": {
    "coworkWebSearchEnabled": true,
    "coworkScheduledTasksEnabled": false,
    "ccdScheduledTasksEnabled": false
  },
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": ["C:\\Users\\<TWÓJ_USER>"]
    }
  }
}
```

### Krok 3 — Zrestartuj Claude Desktop

Zamknij całkowicie (z zasobnika systemowego) i otwórz ponownie.

---

## 5. Weryfikacja — logi

Logi znajdziesz w:

```
C:\Users\<TWÓJ_USER>\AppData\Local\Packages\Claude_<ID>\LocalCache\Roaming\Claude\logs\
```

Sprawdź plik `mcp-server-filesystem.log`. Poprawne połączenie wygląda tak:

```
[filesystem] Server started and connected successfully
[filesystem] Message from client: {"method":"initialize"...}
[filesystem] Message from server: {"result":{"protocolVersion":"2025-11-25"...}}
[filesystem] Message from client: {"method":"notifications/initialized"...}
[filesystem] Message from client: {"method":"tools/list"...}
[filesystem] Message from server: {"result":{"tools":[...]}}
```

---

## 6. Dodawanie kolejnych MCP serwerów

Możesz dodać wiele serwerów w sekcji `mcpServers`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": ["C:\\Users\\<TWÓJ_USER>"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<TWÓJ_TOKEN>"
      }
    }
  }
}
```

### Przydatne MCP serwery

| Serwer | Pakiet | Do czego służy |
|--------|--------|----------------|
| Filesystem | `@modelcontextprotocol/server-filesystem` | Dostęp do plików lokalnych |
| GitHub | `@modelcontextprotocol/server-github` | Repozytoria, issues, PR-y |
| Brave Search | `@modelcontextprotocol/server-brave-search` | Wyszukiwanie w internecie |
| Puppeteer | `@modelcontextprotocol/server-puppeteer` | Automatyzacja przeglądarki |
| Google Drive | Wbudowany w claude.ai | Dokumenty, arkusze |

---

## 7. Rozwiązywanie problemów

| Objaw | Rozwiązanie |
|-------|-------------|
| `Server transport closed` po ~25ms | Problem z SSH — użyj lokalnego MCP |
| `scripts disabled` przy npm | Uruchom `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` |
| `Request timed out` po 60s | Serwer startuje ale nie odpowiada — sprawdź wersję protokołu |
| `Could not attach to MCP server` | Sprawdź logi, błąd JSON w configu (brakujący przecinek itp.) |
| `mcp-server-filesystem not found` | Uruchom `npm install -g @modelcontextprotocol/server-filesystem` |

---

## 8. Szybki start na nowym PC — checklist

- [ ] Zainstaluj Node.js
- [ ] Włącz skrypty: `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`
- [ ] Zainstaluj MCP: `npm install -g @modelcontextprotocol/server-filesystem`
- [ ] Znajdź `claude_desktop_config.json` w folderze `Packages\Claude_*`
- [ ] Dodaj konfigurację `mcpServers` do pliku JSON
- [ ] Zrestartuj Claude Desktop
- [ ] Sprawdź logi — szukaj `tools/list` z listą narzędzi

---

*Dokumentacja wygenerowana po wdrożeniu z 1 maja 2026.*
