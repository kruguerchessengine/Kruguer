# Kruguer C 1.1

Motor UCI em C17 de Francisco de Assis M. B. Jr. GPL-3.0-only, em `LICENSE`.
Tabelas PeSTO: Ronald Friederich. Identificação UCI: **Kruguer C 1.1**.

Esta versão deriva de **Kruguer 1.0 Hybrid NPS**. Preserva avaliação clássica
MG/EG, PeSTO, seleção híbrida entre fórmulas e tabelas, caches por thread,
SEE, busca alpha-beta/PVS, quiescência, TT, finais KPK e até oito threads.
Pesos e heurísticas de jogo são os mesmos da híbrida; não houve tuning.

## Código compacto

`src/k.c`: **650 → 398 linhas**, redução de **38,8%**. A 1.0 original tem
426 linhas. Foram reunidas linhas curtas e comentários, preservando os tokens
do programa, exceto a identificação da versão. Isso foi conferido após o
pré-processamento para Linux e Windows. A redução de linhas é de formatação;
o ganho de NPS vem das otimizações já existentes na híbrida.

## Comparação com a 1.0

Medição em 24/10/2026. Benchmark `--bench 13 1`: cinco posições,
**5.302.137 nós** em cada execução, mesmos scores e melhores lances.
Nove execuções por versão, ordem alternada, afinidade à CPU lógica 2.
Compilações Linux iguais: GCC 13.3, C17, `-O3`, `-DNDEBUG`,
`-march=x86-64-v2`, `-mtune=generic`, `-ffunction-sections`, LTO.
Sem PGO nas duas compilações usadas nesta comparação.
CPU: Intel Xeon Platinum 8573C, ambiente compartilhado.

| Código | Mediana de NPS |
| --- | ---: |
| Kruguer 1.0 original | 1.104.611 |
| Kruguer 1.1, híbrida compactada | 1.119.066 |

**Ganho observado: +1,31% de NPS.**

Nas duas baterias anteriores, a híbrida ainda sem compactar ficou à frente:
+0,49% no benchmark original e +3,64% em um programa de conferência que também
mediu tempo de CPU. São ganhos pequenos e há variação entre rodadas. Estes
números comparam o código recompilado para Linux; o resultado no Windows e
no seu computador pode variar. Não houve arena para medir Elo.

Rodadas finais, em NPS:

| Rodada | 1.0 | 1.1 |
| --- | ---: | ---: |
| 1 | 1.086.280 | 1.142.948 |
| 2 | 975.913 | 1.087.840 |
| 3 | 1.120.722 | 1.122.382 |
| 4 | 1.114.831 | 1.113.894 |
| 5 | 1.104.611 | 1.105.533 |
| 6 | 1.065.542 | 1.119.066 |
| 7 | 1.112.725 | 1.053.893 |
| 8 | 1.107.149 | 1.150.138 |
| 9 | 1.094.804 | 1.156.661 |

## Verificação

- 6.338 posições legais, incluindo 4.182 com até dez peças: mesmas avaliações
  e listas de lances entre 1.0, híbrida e 1.1. Lances também conferidos com
  python-chess.
- Perft inicial depth 5: 4.865.609; Kiwipete depth 4: 4.085.603;
  final de torres de referência depth 5: 674.624.
- KPK: 111.282 estados vencedores, `max_dtz 20`.
- UCI: nome 1.1, buscas com 1/2/4/8 threads, retorno a uma thread,
  `Clear Hash`, interrupção da busca e `isready`.

## Conteúdo deste pacote

Este pacote contém somente o código-fonte e a documentação, sem executáveis:

```text
Kruguer-1.1/
├── src/
│   └── k.c
├── README.md
└── LICENSE
```

Para publicar no GitHub, envie o conteúdo dessa pasta para a raiz do
repositório. O fonte mantém suas 398 linhas e os pesos ainda não foram tunados.
O executável deve ser compilado localmente; os comandos abaixo são exemplos
para Windows com MinGW-w64.

## Compilar no Windows

MinGW-w64 GCC x64 >= 12. Na raiz do projeto, no PowerShell:

```powershell
& "C:\msys64\ucrt64\bin\gcc.exe" -std=c17 -O3 -DNDEBUG -march=x86-64-v2 -mtune=generic -ffunction-sections -flto=1 -flto-partition=one src/k.c -static -s "-Wl,--gc-sections,--no-insert-timestamp,--stack,8388608" -o kruguer.exe
if ($LASTEXITCODE -ne 0) { throw "Falha na compilação" }
.\kruguer.exe --perft 4
.\kruguer.exe --kpk-stats
.\kruguer.exe --bench 13 1
```

Esperados: perft `197281`; KPK `111282`, `max_dtz 20`; benchmark com uma thread
`5302137` nós. O comando acima gera uma compilação normal com LTO.

Para gerar PGO completo no seu Windows, use uma pasta de perfis nova:

```powershell
$cc = "C:\msys64\ucrt64\bin\gcc.exe"
$flags = @("-std=c17", "-O3", "-DNDEBUG", "-march=x86-64-v2", "-mtune=generic", "-ffunction-sections", "-flto=1", "-flto-partition=one", "src/k.c", "-static", "-s", "-Wl,--gc-sections,--no-insert-timestamp,--stack,8388608", "-o", "kruguer.exe")
& $cc @flags -fprofile-generate=pgo-1.1 -fprofile-update=atomic
if ($LASTEXITCODE -ne 0) { throw "Falha na compilação instrumentada" }
.\kruguer.exe --pgo-train 500000
if ($LASTEXITCODE -ne 0) { throw "Falha na coleta do perfil" }
& $cc @flags -fprofile-use=pgo-1.1 -fprofile-correction -fprofile-partial-training -Werror=coverage-mismatch -Werror=missing-profile
if ($LASTEXITCODE -ne 0) { throw "Falha na compilação com PGO" }
```

Mantenha fonte, compilador, flags, diretório e nome de saída entre as duas
compilações do treino. PGO otimiza o executável e não altera os pesos do xadrez.
Não reutilize perfis de outras versões depois de compactar o fonte.

## Integridade SHA-256

Fonte: `2c9d7dd831fef696b076db42825c49e6f6903f0836ea2206d1581e0b46d1ca00`

