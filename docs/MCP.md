# Konfiguracja MCP (GitHub) dla organizacji property-aggregator

Instrukcja dla developerów: jak podłączyć asystenta AI (np. Claude Code) przez **GitHub MCP Server** tak, aby miał dostęp **wyłącznie do repozytoriów organizacji `property-aggregator`**, a nie do całego konta GitHub.

---

## 1. Zasada: najmniejsze możliwe uprawnienia

Token OAuth z `gh auth login` (prefiks `gho_`) ma zakres `repo`, który obejmuje **wszystkie** Twoje repozytoria, także prywatne i osobiste. Nie używaj go do MCP.

Zamiast tego utwórz **fine-grained Personal Access Token** ograniczony do organizacji.

> Fine-grained token zawsze ma odczyt publicznych repozytoriów na GitHubie, niezależnie od ustawień. Prywatne repozytoria i zapis są ograniczone tylko do tego, co zaznaczysz w tokenie.

## 2. Utworzenie tokenu

1. Wejdź na <https://github.com/settings/personal-access-tokens/new>.
2. Ustaw:
   - **Resource owner:** `property-aggregator`. Jeśli organizacji nie ma na liście, właściciel organizacji musi włączyć tokeny fine-grained w ustawieniach organizacji (*Settings → Personal access tokens*).
   - **Repository access:** *All repositories* (obejmuje też nowe repozytoria) lub *Only select repositories*.
   - **Expiration:** 30–90 dni.
3. Nadaj uprawnienia (*Repository permissions*).

### Wariant minimalny: tylko odczyt

| Uprawnienie | Poziom |
|---|---|
| Metadata | Read-only (dodawane automatycznie) |
| Contents | Read-only |
| Issues | Read-only |
| Pull requests | Read-only |
| Actions | Read-only |

### Wariant roboczy: odczyt i zapis

| Uprawnienie | Poziom |
|---|---|
| Metadata | Read-only |
| Contents | Read and write |
| Issues | Read and write |
| Pull requests | Read and write |
| Actions | Read-only |
| Workflows | Read and write (tylko jeśli asystent ma zmieniać `.github/workflows`) |

**Nie zaznaczaj:** Administration, Secrets, Webhooks, Environments ani żadnych *Account permissions*.

Zacznij od wariantu minimalnego. Uprawnienia można później rozszerzyć w ustawieniach tokenu bez tworzenia nowego.

## 3. Podłączenie GitHub MCP Server

### Claude Code (zdalny serwer MCP)

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer $GITHUB_PAT"
```

Zmienna `GITHUB_PAT` powinna zawierać token z punktu 2. Ustaw ją w środowisku (np. w profilu powłoki lub menedżerze haseł), nie w pliku w repozytorium.

Sprawdzenie:

```bash
claude mcp list
```

### Inne klienty (Cursor, VS Code itp.)

Dodaj w konfiguracji MCP klienta serwer HTTP z adresem `https://api.githubcopilot.com/mcp/` i nagłówkiem `Authorization: Bearer <TOKEN>`. Szczegóły formatu pliku zależą od klienta, zobacz dokumentację: <https://github.com/github/github-mcp-server>.

## 4. Ograniczenie pracy do organizacji

Token nie pozwala wyłączyć odczytu publicznych repozytoriów spoza organizacji. Żeby asystent pracował tylko na `property-aggregator`:

- w instrukcjach asystenta (np. `CLAUDE.md` lub pamięć) zapisz regułę: *„Pracuj wyłącznie na repozytoriach organizacji `property-aggregator`, pozostałe ignoruj”*,
- zawsze zawężaj zapytania do organizacji, np. `org:property-aggregator` w wyszukiwaniu.

Twarde ograniczenie zapisu i dostępu do prywatnych repozytoriów zapewnia sam token (punkt 2).

## 5. Weryfikacja

Poproś asystenta o wylistowanie repozytoriów organizacji. Powinny pojawić się:

`.github`, `backend`, `frontend`, `infrastructure`, `property-portals-mock`, `scraper`.

Twoje prywatne repozytoria nie powinny być widoczne.

## 6. Bezpieczeństwo

- **Nie commituj tokenu**, nie wklejaj go w czacie, issues ani PR-ach.
- Używaj krótkiego terminu ważności i odnawiaj token.
- Po wycieku lub zmianie zespołu odwołaj token w <https://github.com/settings/personal-access-tokens>.
- Przy zmianie zakresu prac (np. brak potrzeby zapisu) obniż uprawnienia tokenu.
