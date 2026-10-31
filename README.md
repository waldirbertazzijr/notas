# notas 🎵

Um sintetizador de piano de brinquedo, num arquivo Python só. Você escreve notas como
texto e ele toca:

```bash
notes "E5 G5 C6:2 A5 F5 A5:2 G5 B5 D6 B5 C6:3"
```

Ele também lê arquivos MIDI e converte MIDI em texto.

## Começo rápido

Com o [uv](https://docs.astral.sh/uv/) instalado, no Linux ou no macOS:

```bash
git clone https://github.com/waldirbertazzijr/notas && cd notas && ./notes "E5 G5 C6:2 A5 F5 A5:2 G5 B5 D6 B5 C6:3"
```

## A história

Ao tentar emular um GBA no meu ambiente de experimentos, encontrei um problema: o mGBA
não tinha som.

Meu Ubuntu roda numa VM do UTM (Apple Virtualization) no Mac. O mGBA mandava áudio
de verdade, só que bastava abrir o emulador para **todo** o som da VM sumir, YouTube
incluso.

O culpado era o **quantum** do PipeWire. O mGBA pede latência baixa e o PipeWire
reduziu o pedaço de áudio de 1024 para 256 amostras (~5 ms) para todos os apps. A
placa VirtIO não aguentava. Tocando direto no ALSA, sem o PipeWire:

| fila de áudio | resultado |
|---|---|
| 1024 amostras (~21 ms) | toca ✅ |
| 512 amostras (~11 ms) | `Input/output error` ❌ |

A correção é uma linha de configuração: [`extras/10-min-quantum.conf`](extras/10-min-quantum.conf).

Depois disso veio a curiosidade. Um `.wav` é só um cabeçalho de 44 bytes e uma lista
de números, 48.000 por segundo. Um `.mid` não tem som nenhum: são só instruções
(qual tecla, quando, com que força). Daí nasceu o `notes`.

## Instalação

Só depende do `numpy`.

**Com [uv](https://docs.astral.sh/uv/) (recomendado).** O script declara as próprias
dependências (PEP 723), então o uv cuida do numpy sozinho:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh    # se ainda não tiver o uv
git clone https://github.com/waldirbertazzijr/notas
cd notas
```

Dentro da pasta, qualquer um destes funciona:

```bash
./notes "C4 E4 G4 C5:2"                  # a 1ª linha do arquivo já chama o uv
uv run --script notes "C4 E4 G4 C5:2"    # explícito
```

`uv run notes` sem o `--script` não funciona: como o arquivo não termina em `.py`, o uv
procura um comando `notes` no PATH.

Para usar `notes` de qualquer pasta, crie um link no PATH. Assim um `git pull` já
atualiza o comando:

```bash
ln -s "$PWD/notes" ~/.local/bin/notes
notes "C4 E4 G4 C5:2"
```

**Sem uv:**

```bash
pip install numpy          # ou: sudo apt install python3-numpy
python3 notes "C4 E4 G4 C5:2"
```

Roda no Linux e no macOS do mesmo jeito. Para tocar, ele usa o player que encontrar:
`pw-play` (PipeWire), `afplay` (macOS), `paplay`, `aplay` ou `ffplay`. Se não achar
nenhum, use `-o arquivo.wav`.

No macOS, se `~/.local/bin` não estiver no PATH, adicione no `~/.zshrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

## Uso

```bash
notes "E5 G5 C6:2"                       # melodia
notes "E5 G5 C6:2" "C3+E3+G3:4"          # cada argumento é uma voz
notes -b 0.25 "C4 D4 E4 - E4"            # segundos por tempo (padrão 0.4)
notes "C4 E4 G4" -o arpejo.wav           # salva em vez de tocar
notes --midi musica.mid -o musica.wav    # renderiza um MIDI
notes --midi musica.mid --to-notes       # MIDI → comando notes
```

| sintaxe | significado |
|---|---|
| `C4` | nota (Dó central) |
| `C#4`, `Bb3` | sustenido, bemol |
| `:2`, `:0.5` | duração em tempos (padrão 1) |
| `C3+E3+G3` | acorde |
| `-` | pausa |

| C | D | E | F | G | A | B |
|---|---|---|---|---|---|---|
| Dó | Ré | Mi | Fá | Sol | Lá | Si |

Aqui `C4` é o Dó central, que muitos materiais brasileiros chamam de Dó3.

## Exemplos

Os exemplos chamam `notes`, então precisam do link no PATH (veja Instalação):

```bash
bash exemplos/melodia.sh    # a primeira melodia
bash exemplos/bossa.sh      # 🇧🇷 Corredor de Areia
bash exemplos/musette.sh    # 🇫🇷 Les Couloirs de Montmartre
bash exemplos/barocco.sh    # 🇮🇹 Concerto dei Corridoi
```

## Como o som é feito

Sem nenhuma amostra gravada:

1. `E4` vira a tecla MIDI 64, que vira `440 × 2^((64−69)/12)` = 329,6 Hz.
2. Cada nota soma 8 senoides (fundamental e harmônicos), cada uma duplicada e
   levemente desafinada, como as cordas do piano. Os agudos morrem mais rápido.
3. Ataque de 4 ms e soltura de 0,25 s, para não dar clique.
4. Tudo é mixado em estéreo, convertido para 16 bits e mandado para o player do sistema.

O texto é o código-fonte, o `notes` é o compilador e o `.wav` é o binário.

---

Feito numa madrugada de outubro de 2026. Era só para consertar o som do emulador. 🎶
