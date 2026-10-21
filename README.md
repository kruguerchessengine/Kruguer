# Kruguer C 1.0

Motor de xadrez UCI em C17. Autor: Francisco de Assis M. B. Jr.
Licença: GNU GPL 3.0 (`GPL-3.0-only`), disponível em `LICENSE`.
Tabelas PeSTO: Ronald Friederich.

Esta é a versão anterior ao tuning, com os pesos manuais fixos e todas as
features de jogo preservadas: avaliação clássica MG/EG, PeSTO, alpha-beta/PVS,
quiescência, tabela de transposição, finais KPK e até oito threads.

O código completo está em `src/k.c`, com 426 linhas. Não depende de outros
arquivos de parâmetros, redes ou livros de abertura.

## Compilar no Windows

Use MinGW-w64 GCC x64 >= 12. No PowerShell, dentro desta pasta, por exemplo
com o GCC do MSYS2 UCRT64:

```powershell
& "C:\msys64\ucrt64\bin\gcc.exe" -std=c17 -O3 -DNDEBUG -march=x86-64-v2 -mtune=generic -ffunction-sections -flto=1 -flto-partition=one src/k.c -static -s "-Wl,--gc-sections,--no-insert-timestamp,--stack,8388608" -o kruguer.exe
```

Ajuste o caminho do compilador conforme sua instalação. O comando gera
`kruguer.exe` nesta pasta; cadastre esse arquivo em uma GUI compatível com UCI.

## Compilar no Linux

```sh
gcc -std=c17 -O3 -DNDEBUG -march=x86-64-v2 -mtune=generic -ffunction-sections -flto=1 -flto-partition=one src/k.c -pthread -lm -Wl,--gc-sections -s -o kruguer
```

## Conferência rápida

```powershell
.\kruguer.exe --perft 4
.\kruguer.exe --kpk-stats
.\kruguer.exe --bench 12 4
```

Resultados esperados: perft inicial `197281`; KPK `111282` posições ganhas,
`max_dtz 20`. Na GUI, as opções UCI incluem Hash, Threads e Move Overhead.

O pacote contém somente o fonte, esta documentação e a licença. Não inclui
executáveis, pasta de build, scripts de compilação ou ferramentas de tuning.
