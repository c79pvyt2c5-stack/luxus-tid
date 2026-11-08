# JARVIS X

Selvstændig projektmappe til JARVIS X. Den er bevidst adskilt fra alle andre projekter i dette repository (fx Luxus-TID, Saxo Trading Desk og Crypto AI Trading Desk) og må ikke blandes sammen med dem.

## Status

**Fase 0 / Port A: kun læsning og review.** Der er ikke givet godkendelse til implementering. Der må ikke skrives kode, installeres noget eller startes tjenester, før ejeren giver et nyt, udtrykkeligt GO.

## Indhold

| Mappe / fil | Beskrivelse |
|---|---|
| `00_kildemateriale/JARVIS_X_Samlet_Review_2026-11-08.pdf` | Samlet pakke (83 sider): besked til Claude Code, nye krav (8. nov. 2026), masterplan v1.5 (36 sider), Claudes tidligere review med 337 krav (40 sider) og inspirationsbillede til 3D AI-kontor |

## Faste rammer (fra kildematerialet)

- Bygges helt fra bunden på Mac Studio M5 Max med 48 GB unified memory. Ingen gammel Jarvis-kode.
- Alle 20 AI-roller og hele masterplanen skal leveres. Faserne bestemmer kun rækkefølgen, de skærer ikke noget fra.
- Claude Code er den obligatoriske kritiske reviewer før ændringer, test og merge.
- Safety Core er deterministisk kode og kan aldrig omgås af modeller, roller eller cloud.
