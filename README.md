![TonnieJava](https://github.com/user-attachments/assets/7c213a67-451f-4fde-88ba-a48f690e2452)

# Jogo de Sudoku em Java — MVC + Backtracking + Swing

> **Bootcamp TONNIE — Java and AI in Europe | DIO**

---

## 1. Problema de Negócio

Aplicações de entretenimento educacional enfrentam um paradoxo técnico claro: quanto mais dinâmico é o conteúdo gerado, maior a complexidade algorítmica necessária para garantir sua validade. Jogos de lógica como o Sudoku tornam esse paradoxo concreto.

Qualquer desenvolvedor consegue criar uma grade 9x9 e validar duplicatas em linhas e colunas. O problema real aparece quando as perguntas ficam mais difíceis:

- Como **gerar automaticamente** um tabuleiro com solução única e válida para qualquer tamanho — de 9x9 a 32x32 — sem hard-coding de estados?
- Como **remover células** do tabuleiro completo para criar o desafio sem torná-lo insolúvel ou trivialmente fácil?
- Como organizar o código para que **lógica de jogo, interface gráfica e controle de entrada** do jogador não se misturem, permitindo que qualquer um dos três evolua sem risco de regressão?

Este projeto resolve os três problemas: um jogo de Sudoku completo com geração automática de tabuleiros via backtracking, interface gráfica reativa em Java Swing, suporte a 6 tamanhos de grade, modo rascunho, validação em tempo real e arquitetura MVC com separação de responsabilidades aplicada de forma funcional.

---

## 2. Contexto

O projeto foi desenvolvido no **Bootcamp TONNIE — Java and AI in Europe** como desafio de consolidação de Programação Orientada a Objetos, algoritmos recursivos e desenvolvimento de interfaces gráficas com Java Swing.

O Sudoku foi escolhido como domínio porque combina três desafios que raramente aparecem juntos em projetos de portfólio:

- **Algoritmo não trivial** — gerar um tabuleiro válido exige backtracking recursivo com aleatorização de candidatos, não simples randomização de números;
- **Interface gráfica reativa** — o tabuleiro precisa responder a mouse e teclado, destacar a célula selecionada, diferenciar células fixas de editáveis e renderizar rascunhos em sub-grid dentro de cada célula;
- **Arquitetura extensível por design** — o mesmo código que funciona para 9x9 precisa funcionar para 16x16 e 32x32 sem duplicação de lógica. Extensibilidade como requisito, não como feliz acidente.

---

## 3. Premissas

- **O tabuleiro é sempre gerado, nunca hard-coded** — não existem estados pré-montados no código. Cada novo jogo é criado dinamicamente pelo algoritmo de backtracking, garantindo variedade ilimitada de partidas;
- **`Celula.fixo` é imutável após a geração** — células do tabuleiro inicial não podem ser alteradas pelo jogador. Essa restrição é aplicada pela estrutura da própria classe, não por validação condicional no controller;
- **Arquitetura MVC com separação real de responsabilidades** — `Tabuleiro` (Model) não conhece a existência de interface gráfica. `PainelJogo` (View) não contém lógica de jogo. `ControladorJogo` (Controller) coordena os dois sem cruzar suas fronteiras;
- **O modo rascunho não afeta o valor da célula** — `Celula.rascunhos` é uma estrutura independente de `Celula.valor`. O jogador anota candidatos sem comprometer o estado válido do jogo;
- **Suporte a múltiplos tamanhos é paramétrico** — o tamanho do tabuleiro é passado como parâmetro na criação de `Tabuleiro`. Nenhuma lógica é duplicada para cada tamanho suportado.

---

## 4. Estratégia da Solução

A construção seguiu uma progressão da camada mais interna para a mais externa — do algoritmo ao menu:

```
src/
├── main/
│   └── Main.java               → Entry point; EDT via SwingUtilities.invokeLater()
├── model/
│   ├── Tabuleiro.java          → Backtracking, validação, remoção de células
│   └── Celula.java             → Estado individual: valor, fixo, rascunhos
├── view/
│   ├── PainelJogo.java         → Renderização, interação com mouse
│   └── MenuInicial.java        → Coleta de dados e inicialização da stack MVC
└── controller/
    └── ControladorJogo.java    → KeyListener, delegação ao Model, atualização da View
```

**Fluxo de uma jogada:**

```
Jogador pressiona tecla numérica
        ↓
ControladorJogo (KeyListener)
        ↓
Tabuleiro.setValor() → isValorValido() verifica linha + coluna + sub-grade
        ↓
Celula.setValor() atualiza estado (se válido)
        ↓
PainelJogo.repaint() → paintComponent() relê o estado e redesenha
        ↓
Tabuleiro.isCompleto() → verificação de vitória
```

**Algoritmo de backtracking — duas fases:**

1. **Preencher o tabuleiro completo:** percorre célula a célula, tenta inserir um número da lista embaralhada (1 a N) que não viole linha, coluna e sub-grade. Se nenhum número é válido, retrocede e tenta o próximo candidato da célula anterior;
2. **Criar o desafio:** remove `N * N / 2` células aleatórias do tabuleiro completo, marcando-as como editáveis.

**Stack técnica:**

| Camada | Tecnologia | Função |
|---|---|---|
| Linguagem | Java 17+ | Runtime principal |
| Interface gráfica | Java Swing (`JPanel`, `JFrame`) | Grade e menu |
| Renderização | `paintComponent()` | Desenho customizado do tabuleiro |
| Entrada | `KeyListener` + `MouseAdapter` | Captura de teclado e mouse |
| Thread safety | `SwingUtilities.invokeLater()` | Criação de componentes na EDT |
| Algoritmo | Backtracking recursivo | Geração e validação de tabuleiros |
| Aleatorização | `Collections.shuffle()` | Candidatos embaralhados a cada célula |
| Arquitetura | MVC | Separação de Model, View e Controller |

---

## 5. Decisões Técnicas e Trade-offs

**Por que backtracking recursivo em vez de um gerador baseado em templates?**
Templates pré-montados (um conjunto fixo de tabuleiros válidos que são rotacionados ou transpostos) são mais rápidos computacionalmente, mas limitam a variedade e não escalam para tamanhos arbitrários. O backtracking com `Collections.shuffle()` garante variedade genuinamente ilimitada e funciona para qualquer tamanho de grade quadrada perfeita, ao custo de tempo de geração proporcional ao tamanho. Para tabuleiros 32x32, esse custo é perceptível — registrado como dívida técnica resolvível com `SwingWorker`.

**Por que `Celula.fixo = true` para células removidas, em vez de uma lista separada de posições imutáveis?**
Manter a imutabilidade como atributo da própria célula elimina a necessidade de o Controller consultar uma estrutura externa para decidir se pode ou não editar uma posição. A lógica de "esta célula pode ser alterada?" vive no Model, não no Controller. Isso reduz o acoplamento e torna o Controller agnóstico à procedência do estado de cada célula.

**Por que `paintComponent()` relê o estado a cada repaint em vez de manter cache de renderização?**
A primeira versão mantinha variáveis de estado de renderização dentro do método. Quando o Swing decidia repintar a janela (ao minimizar ou redimensionar), o estado era perdido e a grade aparecia corrompida. Mover todo o estado para os objetos `Celula` tornou `paintComponent()` uma função pura de leitura — o que o Swing exige por design. O custo é um repaint completo a cada jogada; o ganho é previsibilidade absoluta da renderização.

**Por que suporte a múltiplos tamanhos via parâmetro e não por subclasse?**
Subclasses por tamanho (`Tabuleiro9x9`, `Tabuleiro16x16`) teriam triplicado o código de backtracking e validação sem nenhum benefício funcional. O parâmetro `tamanho` no construtor de `Tabuleiro`, combinado com `subGradeTamanho = (int) Math.sqrt(tamanho)`, torna a lógica paramétrica por design. Adicionar suporte a um novo tamanho exige apenas incluí-lo no `JComboBox` do `MenuInicial` — as demais classes não precisam saber que um novo tamanho existe.

**Por que `ControladorJogo` implementa `KeyListener` em vez de usar `InputMap` + `ActionMap`?**
`KeyListener` requer que o componente tenha foco, o que foi resolvido com `painelJogo.setFocusable(true)` e `requestFocusInWindow()`. A escolha foi deliberada pela legibilidade — `InputMap/ActionMap` é o padrão recomendado para componentes reutilizáveis, mas para um controller de jogo de escopo controlado, `KeyListener` expõe o fluxo de lógica de forma mais direta. Em uma evolução do projeto com múltiplos modos de controle, a migração seria o primeiro refactor indicado.

---

## 6. Resultados

| Funcionalidade | Status |
|---|---|
| Geração automática via backtracking com aleatorização | ✅ |
| 6 tamanhos de grade: 9x9, 12x12, 16x16, 25x25, 30x30, 32x32 | ✅ |
| Interface gráfica Swing com mouse e teclado | ✅ |
| Navegação por TAB apenas entre células editáveis | ✅ |
| Modo rascunho por célula (estado independente do valor) | ✅ |
| Validação de jogadas em tempo real (linha + coluna + sub-grade) | ✅ |
| Verificação de vitória com opção de reiniciar | ✅ |
| Arquitetura MVC com fronteiras de responsabilidade reais | ✅ |

**Mapeamento de responsabilidades MVC:**

| Camada | Classe | Responsabilidade |
|---|---|---|
| **Model** | `Tabuleiro.java` | Geração via backtracking, validação, verificação de vitória |
| **Model** | `Celula.java` | Estado individual: valor, fixo, rascunhos |
| **View** | `PainelJogo.java` | Renderização da grade, seleção visual, exibição de rascunhos |
| **Controller** | `ControladorJogo.java` | Captura de teclado, delegação ao Model, atualização da View |
| **Entry Point** | `MenuInicial.java` | Coleta de dados do jogador e inicialização da stack MVC |
| **Entry Point** | `Main.java` | Inicialização da aplicação na EDT |

O resultado concreto da separação MVC: quando 25x25 e 32x32 foram adicionados como opções, a única classe modificada foi `MenuInicial` — um `JComboBox` com dois novos itens. `Tabuleiro`, `Celula`, `PainelJogo` e `ControladorJogo` permaneceram intocados.

---

## 7. Próximos Passos

- **Sistema de dificuldade parametrizável** — expor no `MenuInicial` a escolha entre Fácil, Médio e Difícil, controlando a quantidade de células removidas pelo gerador por nível;
- **Geração em thread separada com `SwingWorker`** — para tabuleiros 25x25+ o backtracking é perceptível; mover a geração para uma worker thread exibe barra de progresso e elimina o congelamento da UI;
- **Persistência de partidas em JSON** — salvar e recarregar o estado do tabuleiro permite retomar uma partida inacabada entre sessões;
- **Resolução automática passo a passo** — botão "Resolver" que aplica backtracking sobre o estado atual do jogador e anima a solução célula a célula;
- **Cronômetro e ranking persistido em CSV** — medir tempo por jogador e tamanho de tabuleiro, mantendo histórico das melhores partidas;
- **Testes unitários para `Tabuleiro`** — validar que `isValorValido()` rejeita corretamente duplicatas nas três dimensões e que o backtracking nunca entrega um tabuleiro com estado inválido.

---

## ⚙️ Como Executar

**Pré-requisito:** Java 17+ instalado.

```bash
# 1. Clone o repositório
git clone https://github.com/Santosdevbjj/Sudoku-o-Jogo-Java.git
cd Sudoku-o-Jogo-Java

# 2. Compile o projeto
javac -d out src/**/*.java

# 3. Execute o jogo
java -cp out main.Main
```

**No menu inicial:**
- Digite seu nome
- Escolha o tamanho do tabuleiro (9x9 a 32x32)
- Clique em **Iniciar Jogo**

**Controles durante o jogo:**

| Ação | Controle |
|---|---|
| Selecionar célula | Clique do mouse |
| Inserir número | Teclas numéricas |
| Navegar entre células editáveis | TAB |
| Ativar/desativar modo rascunho | CTRL |
| Apagar número ou rascunho | DELETE / BACKSPACE |


---

[![Portfólio Sérgio Santos](https://img.shields.io/badge/Portfólio-Sérgio_Santos-111827?style=for-the-badge&logo=githubpages&logoColor=00eaff)](https://portfoliosantossergio.vercel.app)
[![LinkedIn Sérgio Santos](https://img.shields.io/badge/LinkedIn-Sérgio_Santos-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/santossergioluiz)
