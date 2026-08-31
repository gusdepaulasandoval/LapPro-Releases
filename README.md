# LapPro-Releases

Repositório **público**, só com os binários de firmware (OTA) do ApexOne/LapPro.
O código-fonte do firmware fica no repositório privado `LapPro` — este repo
existe só para hospedar releases sem expor código proprietário.

## Como o dispositivo usa isto

No boot (ou quando o usuário pede "verificar atualização"), o firmware baixa:

```
https://raw.githubusercontent.com/gusdepaulasandoval/LapPro-Releases/main/manifest.json
```

Esse `manifest.json` sempre descreve a **última versão publicada**:

```json
{
  "version": "v2026.07.01",
  "version_code": 20260701,
  "url": "https://raw.githubusercontent.com/gusdepaulasandoval/LapPro-Releases/main/releases/v2026.07.01/firmware.bin",
  "sha256": "<64 caracteres hex — sha256 do firmware.bin>",
  "size": 1867234,
  "changelog": "Texto curto descrevendo a release"
}
```

O firmware compara `version_code` com a versão instalada. Se for maior, baixa
o `.bin` do campo `url`, calculando o SHA-256 dos bytes recebidos ao mesmo
tempo que grava na flash. **Só marca a partição como bootável se o hash bater
com o `sha256` do manifest** — protege contra download corrompido/incompleto
e contra um binário adulterado servido no lugar do original.

## Estrutura

```
manifest.json              ← sempre a última versão (é o que o device lê)
releases/
  v2026.06.22/
    firmware.bin
    manifest.json           ← cópia da release, para histórico/rollback
  v2026.07.01/
    firmware.bin
    manifest.json
```

## Como publicar uma nova release

Tudo é gerado automaticamente pelo script `tools/publish_release.py` no
repositório privado do código-fonte (`LapPro`):

1. No repo `LapPro`: bump de `FIRMWARE_VERSION`/`FIRMWARE_VERSION_CODE` em
   `src/version.h`, compile (`pio run -e apex_one_s3`).
2. `python tools/publish_release.py --changelog "..."` — gera
   `manifest.json` + calcula o SHA-256/tamanho a partir do `.bin` compilado,
   deixando tudo pronto em `dist/release/`.
3. Copie o conteúdo de `dist/release/` para este repositório
   (sobrescrevendo `manifest.json` da raiz e adicionando a pasta
   `releases/<versão>/`), depois `git add -A && git commit && git push`.
4. No repo `LapPro`, marque o commit: `git tag <versão> && git push origin <versão>`.

Não edite `manifest.json` ou os arquivos em `releases/` manualmente — o
`sha256` precisa corresponder exatamente ao `.bin` publicado, ou o
dispositivo vai rejeitar o download.
