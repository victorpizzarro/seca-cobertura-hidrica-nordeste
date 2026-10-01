# Seca e Cobertura Hídrica — Nordeste (CE / PB)

*Ciência de Dados Geoespacial*. Monitora o volume armazenado nos açudes do Ceará e da Paraíba e o cruza com a população rural dos municípios, para apoiar a gestão de recursos hídricos no semiárido.

- **Gestor-alvo:** Gestão de recursos hídricos do NE
- **Sistema de referência:** EPSG:31984 (SIRGAS 2000 / UTM 24S)
- **Período analisado:** **2020 a 2026** — séries mensais de volume dos açudes de **01/2020 a 09/2026** (81 meses); população rural do **Censo 2022**

## Estrutura

```
├── data/
│   ├── raw/                 # dados brutos baixados da ANA/IBGE (não versionado)
│   └── processed/           # .geojson do ETL e .csv de resultados da análise
├── notebooks/
│   ├── 01_ETL.ipynb
│   ├── 02_Analise.ipynb
│   └── 03_Visualizacao.ipynb
├── .gitignore
└── README.md
```

## Fontes de dados

| Dado | Fonte | Observação |
|---|---|---|
| Reservatórios | ANA — [SAR, consulta de medições](https://www.ana.gov.br/sar0/Medicao) e camadas do [SNIRH](https://www.snirh.gov.br/arcgis/rest/services/IG/SAR/FeatureServer/1) | Substitui o Hidroweb, que não traz volume de açudes. Coordenadas: camada SAR Nordeste; 19 açudes ausentes dela foram localizados na camada *Massas d'Água* (ponto dentro do polígono). |
| Municípios CE e PB | IBGE — [API de malhas](https://servicodados.ibge.gov.br/api/v3/malhas/estados/23) (GeoJSON) | Mesma origem do GeoJSON por estado listado na tarefa, direto da fonte oficial. |
| População rural por município | IBGE — Censo 2022, [SIDRA tabela 9923](https://apisidra.ibge.gov.br/values/t/9923) | Situação do domicílio (urbana/rural). |

## ETL (`01_ETL`)

1. Download dos volumes dos açudes da ANA (período 2020-01 a 2026-09).
2. Reprojeção para EPSG:31984 e join com os municípios de CE e PB (município por localização do açude).
3. Volume armazenado (%) por reservatório e mês = média mensal do volume ÷ capacidade × 100 (limitado a 0–100).
4. Exportação para `data/processed/`:
   - `reservatorios-ne.geojson` — pontos em EPSG:31984, uma feição por reservatório e mês (20.703 feições; 279 açudes, 155 no CE e 124 na PB).
   - `municipios-ne.geojson` — polígonos dos 407 municípios, com população total, urbana e rural.

## Limitações conhecidas

- Os dados vêm do SAR por consulta à página (a ANA não oferece API/CSV); o método pode quebrar se a página mudar.
- Faltam 2 açudes cadastrados no SAR para CE/PB: `BREJINHO` (localizado em Triunfo-PE, fora do escopo) e `CUREMA/MÃE D'ÁGUA` (sem medições no período).
- Alguns registros da ANA são inconsistentes (ex.: `ALBINO` com volume maior que a capacidade); o percentual foi limitado a 100%.
- 25 açudes (incluindo os 19 complementares) não têm bacia hidrográfica informada e ficam fora das médias por bacia.

## Tecnologias e bibliotecas

| Tecnologia | Uso no projeto |
|---|---|
| **Python 3.12** | Linguagem de todo o pipeline |
| **Jupyter** | Notebooks executáveis (ETL, análise e visualização) |
| **pandas** | Tabelas, agregações mensais e junções |
| **GeoPandas** (com Shapely e PyProj) | Leitura e escrita geoespacial, reprojeção para EPSG:31984, join espacial e buffers de 50 km |
| **requests** | Download das APIs e da página do SAR, com novas tentativas |
| **SciPy** | Correlação de Pearson e Spearman |
| **Matplotlib** | Mapas, séries temporais, barras, scatter e mapa de bolhas |
| **GeoJSON e CSV** | Formatos abertos de entrada e saída dos dados |
| **ArcGIS REST (SNIRH/ANA), SAR (ANA), API de malhas e SIDRA (IBGE)** | Serviços de onde os dados são obtidos |
| **Git** | Controle de versão, com `.gitignore` para dados brutos e arquivos locais |

## Notebooks

| Notebook | Conteúdo |
|---|---|
| `01_ETL` | Download (ANA/IBGE), reprojeção para EPSG:31984, join com municípios, volume (%) por açude e mês, e exportação dos GeoJSON. Precisa de internet e leva alguns minutos. |
| `02_Analise` | Volume médio por bacia, municípios críticos (<30%) por mês, área de influência de 50 km e correlação população rural × distância. Gera os `data/processed/analise_*.csv`. |
| `03_Visualizacao` | Mapa de nível dos reservatórios, séries temporais (estado e bacias), barras de municípios críticos, scatter e mapa de bolhas. |

Definições da análise (a tarefa não as detalha): o estoque hídrico de um município é Σ volume ÷ Σ capacidade dos açudes a até 50 km do seu centroide; é crítico quando fica abaixo de 30%.

`data/raw/` não é versionado: execute o `01_ETL` primeiro para baixá-lo (o `02_Analise` e o `03_Visualizacao` leem só `data/processed/`).

Para executar os notebooks: `pip install geopandas pandas matplotlib scipy requests jupyter`.

## Autores

- André Luiz
- Arthur Antunes
- Leonardo Sandes
- Victor Pizzarro

Estudo realizado como material avaliativo da disciplina 4UBD na FAETERJ.
