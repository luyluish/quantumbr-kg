# QuantumBR Knowledge Graph

Base de conhecimento em RDF, RDFS e OWL sobre a produção científica brasileira em tecnologias quânticas. Os dados são coletados no OpenAlex, convertidos para Turtle (`.ttl`) e consultados com `rdflib` e SPARQL. Todo o fluxo — coleta, geração da base e consultas — roda em um único notebook, `quantumbr_kg.ipynb`.

## Resumo da base

A base modela publicações científicas, pesquisadores, instituições, veículos de publicação e áreas de pesquisa em tecnologias quânticas, com relações de autoria, afiliação, colaboração, citação e classificação temática.

É considerada brasileira toda publicação com pelo menos um autor afiliado a uma instituição no Brasil — critério garantido pelo filtro `institutions.country_code:br` da API, sem depender de nome do autor ou menção textual ao Brasil. A coleta é limitada a um teto de publicações (`MAX_RESULTADOS`, 400 por padrão) para manter o tempo de execução previsível; os mínimos exigidos pelo projeto (25+ instâncias, 50+ triplas) são atingidos com folga bem abaixo desse teto.

**Fonte dos dados:** OpenAlex, endpoint `/works`, sem autenticação (usa o parâmetro `mailto` para o "pool educado" da API).

## Principais entidades e classes

| Classe | Descrição |
|---|---|
| `Publicacao` | Produção científica, com subtipos `ArtigoDePeriodico`, `ArtigoDeConferencia` e `Preprint` |
| `Pesquisador` | Autor de uma ou mais publicações |
| `InstituicaoDePesquisa` | Instituição à qual um pesquisador está afiliado |
| `VeiculoDePublicacao` | Onde a publicação saiu, com subtipos `Periodico` e `Conferencia` |
| `AreaDePesquisa` | Área temática, com a subárvore `TecnologiaQuantica` (Computação, Comunicação, Informação e Sensoriamento Quântico) |

**Propriedades de objeto** (10): `temAutor`/`autorDe` (inversas), `afiliadoA`, `publicadoEm`, `cita`/`citadoPor` (inversas), `possuiArea`, `possuiTopico`, `colaboraCom` (simétrica), `subareaDe` (transitiva).

**Propriedades de dados** (8): `titulo`, `nome`, `anoPublicacao`, `doi`, `numeroCitacoes`, `openAlexId`, `pais`, `acessoAberto`.

**Construções OWL:** `owl:inverseOf` (`temAutor`/`autorDe`, `cita`/`citadoPor`), `owl:FunctionalProperty` (`doi`), `owl:SymmetricProperty` (`colaboraCom`), `owl:TransitiveProperty` (`subareaDe`) e `owl:disjointWith` (`Pessoa`/`Organizacao`, `ArtigoDePeriodico`/`ArtigoDeConferencia`).

## Taxonomia (hierarquia de classes)

A raiz é `Entidade`, com pelo menos dois níveis de subclasses abaixo dela em cada ramo:

```
Entidade
├── Agente
│   ├── Pessoa
│   │   └── Pesquisador
│   └── Organizacao
│       └── InstituicaoDePesquisa
├── ProducaoCientifica
│   └── Publicacao
│       ├── ArtigoDePeriodico
│       ├── ArtigoDeConferencia
│       └── Preprint
├── VeiculoDePublicacao
│   ├── Periodico
│   └── Conferencia
└── AreaDePesquisa
    └── TecnologiaQuantica
        ├── ComputacaoQuantica
        ├── ComunicacaoQuantica
        ├── InformacaoQuantica
        └── SensoriamentoQuantico
```

Publicações, pesquisadores, instituições, veículos e tópicos são **indivíduos**, não classes — a hierarquia acima organiza apenas os tipos aos quais esses indivíduos pertencem.

## Estrutura gerada

- `data/dados.json` — coleta bruta do OpenAlex (cacheada entre execuções).
- `ontology/base_quantica.ttl` — base RDF final, em Turtle.

## Instruções de execução

1. Abra `quantumbr_kg.ipynb` em um ambiente com Jupyter (ou Google Colab).
2. Na célula de configuração, ajuste `MAILTO` para o seu e-mail (exigido pela API do OpenAlex).
3. Rode as células em ordem, de cima para baixo.
   - A coleta precisa de conexão com a internet.
   - Se `data/dados.json` já existir de uma execução anterior, a coleta é pulada automaticamente. Para forçar uma coleta nova, apague o arquivo ou defina `FORCAR_RECOLETA = True`.
   - `MAX_RESULTADOS` controla quantas publicações são baixadas e, portanto, o tempo total de execução.
4. As seções seguintes carregam a base gerada, validam os mínimos do projeto e executam:
   - 5 consultas com `g.triples()`, variando os padrões (sujeito, predicado, objeto fixos e combinações com `None`).
   - 8+ consultas SPARQL, cobrindo `SELECT` (com `FILTER` e com `ORDER BY`/agregação), `ASK`, `CONSTRUCT` e as três formas de update (`INSERT`, `DELETE`, `DELETE`/`INSERT`).
5. A última seção roda testes automáticos que conferem se todos os mínimos da especificação foram atingidos.

## Limitações

A cobertura depende da estratégia de busca (tópicos com "quantum" no domínio de ciências físicas) e da qualidade dos metadados do OpenAlex. Não há garantia de cobertura absoluta da produção científica brasileira em tecnologias quânticas, e o teto de coleta (`MAX_RESULTADOS`) prioriza tempo de execução sobre volume total de dados.
