# NBGV Validate

GitHub Composite Action zur **Validierung von [Nerdbank.GitVersioning (NBGV)](https://github.com/dotnet/Nerdbank.GitVersioning)**-Metadaten.  
Die Action prüft optional, ob Git-Tags, PublicRelease-Flags und Prerelease-Versionen den gewünschten Bedingungen entsprechen.

---

## 🚀 Verwendung

In einem Workflow (`.github/workflows/ci.yml`):

```yaml
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate NBGV state
        uses: sst-germany/actions_nbgvValidate@main
        with:
          requiresTagName: "releases/v*"      # optionales Pattern für Tags, "" (leer = keine Validierung)
          requiredMatchedDigits: "2"          # Anzahl der übereinstimmenden Versionsziffern, "0" = keine Prüfung
          requiresPublicRelease: "yes"        # "yes" | "no" | "" (leer = keine Validierung)
          requiresPrereleaseVersion: "no"     # "yes" | "no" | "" (leer = keine Validierung)
````

---

## ⚙️ Inputs

| Name                        | Beschreibung                                                                                                                      | Erforderlich | Standard |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ------------ | -------- |
| `requiresTagName`           | Wenn gesetzt, wird geprüft, ob das aktuelle Git-Tag zum angegebenen Pattern passt (z. B. `v*`).                                   | nein         | `""`     |
| `requiredMatchedDigits`     | Anzahl der führenden Ziffernblöcke der Version, die im NBGV-Versionsstring und im Tag übereinstimmen müssen. `0` = keine Prüfung. | nein         | `0`      |
| `requiresPublicRelease`     | Erwarteter PublicRelease-Status (`yes`, `no` oder leer = keine Prüfung).                                                          | nein         | `""`     |
| `requiresPrereleaseVersion` | Erwarteter Prerelease-Status (`yes`, `no` oder leer = keine Prüfung).                                                             | nein         | `""`     |

---

## ✅ Validierungen

1. **Tag Name**

   * Überspringt, wenn `requiresTagName` leer ist.
   * Prüft, ob `GITHUB_REF` ein Tag (`refs/tags/...`) ist.
   * Prüft, ob das Tag zum angegebenen Pattern passt.
   * Wenn `requiredMatchedDigits > 0`, wird zusätzlich geprüft, dass die ersten `n` Zifferngruppen der NBGV-Version mit denen im Tag übereinstimmen.
     Beispiel:

     * `NBGV_Version = 1.2.3`, `Tag = v1.2.99`, `requiredMatchedDigits = 2` → ✅ (erste beiden Blöcke stimmen überein).
     * `NBGV_Version = 1.2.3`, `Tag = v1.5.0`, `requiredMatchedDigits = 2` → ❌ (zweite Ziffer weicht ab).

2. **PublicRelease**

   * Überspringt, wenn `requiresPublicRelease` leer ist.
   * Erwartet `"True"` im NBGV-Flag, wenn `yes`.
   * Erwartet `"False"` im NBGV-Flag, wenn `no`.

3. **Prerelease Version**

   * Überspringt, wenn `requiresPrereleaseVersion` leer ist.
   * Erwartet eine nicht-leere Prerelease-Version, wenn `yes`.
   * Erwartet eine leere Prerelease-Version, wenn `no`.

---

## ❌ Fehlerbehandlung

* Bei fehlerhafter Validierung wird der Workflow-Schritt mit einem **Fehler (Exitcode 1)** abgebrochen.
* Die Fehlermeldung erscheint direkt im Actions-Log, z. B.:

```
(NBGV-TagName): Validation failed, reason: TAG does not match pattern. Requested(v*), Found(test-123).
```

---

## 📝 Beispiel-Workflows

### Nur PublicRelease prüfen

```yaml
- uses: sst-germany/actions_nbgvValidate@main
  with:
    requiresPublicRelease: "yes"
```

### Nur Tag-Pattern prüfen

```yaml
- uses: sst-germany/actions_nbgvValidate@main
  with:
    requiresTagName: "releases/v*"
```

### Tag-Pattern + Versionsübereinstimmung prüfen

```yaml
- uses: sst-germany/actions_nbgvValidate@main
  with:
    requiresTagName: "releases/v*"
    requiredMatchedDigits: "2"
```

### Kombination aller Prüfungen

```yaml
- uses: sst-germany/actions_nbgvValidate@main
  with:
    requiresTagName: "releases/v*"
    requiredMatchedDigits: "3"
    requiresPublicRelease: "yes"
    requiresPrereleaseVersion: "no"
```

---

## 🔖 Lizenz

– MIT-Lizenz ([LICENSE.md](LICENSE.md)).
