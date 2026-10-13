# GitHub MCP dla organizacji property-aggregator

Wklejaj kolejno w **Wierszu polecenia (cmd)**.

## 1. Token (w przeglądarce)

<https://github.com/settings/personal-access-tokens/new>

- **Resource owner:** `property-aggregator`
- **Repository access:** All repositories
- **Expiration:** 30–90 dni
- **Repository permissions** (tylko odczyt): Contents, Issues, Pull requests, Actions → *Read-only*. Metadata dodaje się samo.
- Nie zaznaczaj Administration, Secrets, Webhooks, Environments ani *Account permissions*.

Kliknij **Generate token** i skopiuj token.

## 2. Instalacja GitHub CLI

```bat
winget install --id GitHub.cli
```

Zamknij i otwórz cmd ponownie.

## 3. Logowanie

```bat
gh auth login
```

Wybierz: **GitHub.com** → **HTTPS** → **Yes** → **Paste an authentication token** → wklej token z kroku 1.

## 4. Dodanie MCP do Claude Code

```bat
for /f %i in ('gh auth token') do claude mcp add --scope user --transport http github https://api.githubcopilot.com/mcp/ --header "Authorization: Bearer %i"
```

Jeśli pojawi się *already exists*:

```bat
claude mcp remove github
```

i wklej polecenie z kroku 4 jeszcze raz.

## 5. Sprawdzenie

```bat
claude mcp list
```

Przy `github` ma być **Connected**.

## 6. Ograniczenie do organizacji

Uruchom `claude` i wklej:

```
Zapamiętaj: na GitHubie pracuj wyłącznie na repozytoriach organizacji property-aggregator, pozostałe ignoruj. Zawężaj zapytania do org:property-aggregator.
```

## 7. Test

W nowej sesji `claude` wpisz: `Wylistuj repozytoria organizacji property-aggregator`.

Powinno być 6: `.github`, `backend`, `frontend`, `infrastructure`, `property-portals-mock`, `scraper`.

## Zasady

- Nie commituj tokenu i nie wklejaj go w czacie, issues ani PR-ach.
- Wyciek lub zmiana zespołu: odwołaj token na <https://github.com/settings/personal-access-tokens>.
- Po odnowieniu tokenu powtórz kroki 3 i 4.
