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

Tudo é feito por um único comando no repositório privado do código-fonte
(`LapPro`), via `tools/publish_release.py`:

1. No repo `LapPro`: bump de `FIRMWARE_VERSION`/`FIRMWARE_VERSION_CODE` em
   `src/version.h`.
2. `python tools/publish_release.py --changelog "..."`

   O script builda o firmware, calcula o SHA-256/tamanho, gera o
   `manifest.json` e então pergunta, um passo de cada vez:
   - comitar o bump de versão no `LapPro`;
   - dar push nesse commit;
   - copiar os arquivos pra este repositório (`LapPro-Releases`) e dar
     push aqui — usa um clone local salvo depois da primeira vez (ou
     oferece pra clonar se ainda não existir);
   - criar e enviar a tag git da versão no `LapPro`.

   No VS Code, dá pra rodar via **Tasks: Run Task → "Release Firmware
   (build + publica OTA)"**.

Não edite `manifest.json` ou os arquivos em `releases/` manualmente — o
`sha256` precisa corresponder exatamente ao `.bin` publicado, ou o
dispositivo vai rejeitar o download.
