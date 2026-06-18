# SonarQube — Ferramenta de Análise de Qualidade de Código

> Trabalho da disciplina de **Engenharia de Software** — Curso de Ciência da Computação
> Tema da aula: **DevOps** | Ferramenta escolhida: **SonarQube**

---

## Sumário

- [Introdução](#introdução)
- [Sobre a Ferramenta](#sobre-a-ferramenta)
- [Instalação](#instalação)
- [Exemplo de Demonstração Funcional](#exemplo-de-demonstração-funcional)
- [Resultados da Análise](#resultados-da-análise)
- [Dificuldades Encontradas](#dificuldades-encontradas)
- [Conclusão](#conclusão)
- [Referências](#referências)

---

## Introdução

[SonarQube](https://www.sonarsource.com/products/sonarqube/) é uma tecnologia open source que surgiu entre 2006 e 2007, sendo chamada inicialmente apenas de Sonar. Foi criada por três desenvolvedores: [Olivier Gaudin](https://theorg.com/org/sonarsource/org-chart/olivier-gaudin), [Freddy Mallet](https://www.freddymallet.com/) e [Simon Brandhof](https://github.com/simonbrandhof).

A ideia surgiu pela falta de uma ferramenta que analisasse automaticamente a qualidade do código durante o desenvolvimento. Inicialmente focada em Java, atualmente já suporta dezenas de linguagens. Hoje é mantida pela empresa [SonarSource](https://www.sonarsource.com/), realizando análise estática e dinâmica de código.

---

## Sobre a Ferramenta

No contexto de **DevOps**, o SonarQube atua na etapa de integração contínua, garantindo que o código que entra no repositório atenda a critérios de qualidade. Ele identifica:

- **Bugs** — problemas que podem causar comportamento incorreto.
- **Vulnerabilidades** — falhas de segurança no código.
- **Code Smells** — trechos que dificultam a manutenção.
- **Cobertura de testes** e **duplicação de código**.

---

## Instalação

A instalação foi feita em uma **VPS**, utilizando **Docker** via **Portainer**. O processo é relativamente simples e seguiu os passos abaixo.

### 1. Ajustes no sistema da VPS

Conectados na VPS via SSH, foram executados dois comandos necessários para o SonarQube funcionar corretamente (ele exige limites mínimos de memória e de arquivos abertos):

```bash
# Conecta na VPS
ssh usuario@IP-da-sua-VPS

# Ajusta os limites exigidos pelo SonarQube
sudo sysctl -w vm.max_map_count=524288
sudo sysctl -w fs.file-max=131072
```

> `vm.max_map_count` define o número máximo de áreas de memória mapeadas por processo (exigência do Elasticsearch, usado internamente pelo SonarQube) e `fs.file-max` define o limite de arquivos abertos no sistema.

### 2. Criação da Stack no Portainer

Foi criada uma nova **stack** dentro do Portainer, colando o conteúdo do arquivo `docker-compose.yml` (disponível no repositório) e clicando em **Deploy the stack**.

### 3. Primeiro acesso

Acessando o `IP-da-VPS:porta`, fizemos login com as credenciais padrão `admin:admin`, e em seguida o próprio SonarQube solicitou a **troca da senha**.

### 4. Vinculação do projeto

Na tela inicial, o SonarQube pede para vincular o projeto a uma plataforma (GitHub, GitLab, etc.) ou criar um projeto **local manualmente**, que foi a opção escolhida.

### 5. Configuração e execução da análise

Com o projeto local criado, o SonarQube gerou um **token** e forneceu os comandos para rodar o **scanner** na pasta do projeto, executados localmente via terminal.

---

## Exemplo de Demonstração Funcional

Para a demonstração, foi analisado um projeto real: um **site de casamento** desenvolvido para amigos. O projeto está versionado no Git, referenciado no final do README.md e foi rodado localmente para gerar a análise.

O fluxo da demonstração foi:

1. Criar o projeto local no SonarQube.
2. Rodar o scanner na pasta do projeto.
3. Visualizar o relatório gerado no dashboard.
<img width="915" height="275" alt="image" src="https://github.com/user-attachments/assets/2541d36e-4ddc-41f8-829a-5bc7e8e4f02e" />

---

## Resultados da Análise

A análise apontou **206 issues no total** no projeto. Abaixo estão destacadas algumas:

### 🔴 Críticos (Bugs — Alta prioridade)

**IDs duplicados em `ref/index.html`** — 4 ocorrências:

- `menu-evento` duplicado (linhas 176 e 198)
- `nav-menu-principal` duplicado (linhas 169 e 263)
- `clip0_9993_100` duplicado (linhas 1051, 1062 e 1162)

> IDs duplicados quebram JavaScript e acessibilidade. **Solução:** renomear cada ID para ser único.

### 🟡 Segurança (Vulnerabilities — 9 issues)

**Falta de `integrity` em scripts/links externos** nos arquivos `admin.html`, `index.html` e `ref/index.html`.

Os `<script>` e `<link>` que carregam CDNs externos estão sem o atributo `integrity`. Exemplo de correção:

```html
<!-- Antes -->
<script src="https://cdn.exemplo.com/lib.js"></script>

<!-- Depois -->
<script src="https://cdn.exemplo.com/lib.js"
        integrity="sha384-HASH_AQUI"
        crossorigin="anonymous"></script>
```

> O hash é obtido no site do CDN utilizado (Bootstrap, FontAwesome, etc).

### 🔵 Acessibilidade (Bugs — Baixa prioridade)

Problemas de acessibilidade em `ref/index.html`:

- **`<iframe>` sem `title`** (linhas 739, 816, 1701, 2062) → adicionar `title="descrição do conteúdo"`.
- **Eventos de mouse sem equivalente de teclado** (7 ocorrências) → elementos com `onClick` precisam também de `onKeyPress`.
- **`<html>` sem atributo `lang`** → adicionar `<html lang="pt-BR">`.
- **Variáveis reatribuídas sem uso** em JS (linhas 33, 34, 56, 57, 95, 96) → usar `let` ao invés de reutilizar parâmetros.

### 📊 Resumo

| Prioridade | Tipo | Qtd | Ação |
| --- | --- | --- | --- |
| 🔴 Alta | IDs duplicados | 4 | Corrigir agora |
| 🟡 Média | Sem integrity CDN | 9 | Corrigir em breve |
| 🔵 Baixa | Acessibilidade | 18 | Melhorias futuras |

> A tabela acima representa um recorte das issues. No total, o SonarQube identificou **98 issues** no projeto.

---

## Dificuldades Encontradas

- Os ajustes iniciais na VPS (`sysctl`) não eram intuitivos para quem não conhece os requisitos internos do SonarQube (Elasticsearch).
- Vale destacar que **o SonarQube não corrige os problemas automaticamente**: ele identifica, explica e orienta a correção, mas as mudanças precisam ser aplicadas manualmente pelo desenvolvedor. Isso aumentou o esforço de interpretação, já que cada uma das 206 issues teve que ser analisada e resolvida à mão. Vale ressaltar, porém, que existem recursos mais recentes com **IA** (como o *AI CodeFix* e o *quick fix* do [SonarQube for IDE](https://www.sonarsource.com/products/sonarlint/), antigo SonarLint, integrado a IDEs) que geram uma sugestão de correção aplicável com um clique — ainda assim de forma assistida, com revisão do desenvolvedor.

---

## Conclusão

Recomendamos o uso dessa ferramenta para projetos que necessitam de controle da qualidade do código, detecção rápida de bugs e vulnerabilidades de segurança. O SonarQube é especialmente adequado para projetos de médio e grande porte desenvolvidos em equipe, onde garantir a confiabilidade e a evolução sustentável do software é essencial.

---

## Referências

- [SonarQube — Site oficial](https://www.sonarsource.com/products/sonarqube/)
- [SonarSource](https://www.sonarsource.com/)
- [Documentação oficial do SonarQube](https://docs.sonarsource.com/)
- [Material gerado durante a etapa de instalação e teste](https://docs.google.com/document/d/1jYVZp5P_iUYeAyYRR0fektgPBpPiy1pgzHA44iZoaXw/edit?usp=sharing)
- [Projeto analisado na demonstração](https://github.com/felip2ee/lannywerbeth-site-noivos)
- [Apresentação no Canva](https://canva.link/bzv1ix8523aklol)
