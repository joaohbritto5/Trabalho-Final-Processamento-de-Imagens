# Trabalho-Final-Processamento-de-Imagens

# Classificação de Tênis (Nike, Adidas e Converse)

## 1. Integrantes e Organização

- **Instituição:** Universidade Federal de Sergipe (UFS) - Departamento de Computação
- **Disciplina:** Processamento de Imagens (COMP 0432)
- **Docente:** Prof. Dr. Leonardo Nogueira Matos
- **Semestre:** 2025.2
- **Discentes:**
    - Arthur Costa Oliveira - 202300027104
    - João Henrique Britto Bomfim - 202300027409

---

## 2. Descrição do Projeto

O projeto tem como objetivo desenvolver um algoritmo capaz de classificar automaticamente imagens de tênis em três categorias de marcas distintas: **Adidas, Converse e Nike**.

O projeto tem como objetivo desenvolver um algoritmo capaz de classificar automaticamente imagens de tênis em três categorias de marcas distintas: Adidas, Converse e Nike. Diferente de abordagens modernas que utilizam Redes Neurais Convolucionais (Deep Learning), este projeto foca na Engenharia de Características (Feature Engineering). O desafio, proposto originalmente na plataforma Roboflow, consiste em identificar padrões geométricos e de textura (como as três listras da Adidas ou o cano alto do Converse) utilizando apenas conceitos fundamentais de processamento de imagens vistos em sala de aula, sem o uso de bibliotecas de alto nível como OpenCV ou PIL.

---

## 3. Metodologia

A metodologia adotada para este trabalho não foi linear, mas sim exploratória e iterativa. Dado o desafio de classificar imagens com alta variabilidade intra-classe sem o uso de Aprendizado de Máquina, a equipe optou por investigar três estratégias distintas de Engenharia de Características (Feature Engineering), baseadas em diferentes conceitos fundamentais de Processamento Digital de Imagens (PDI):

1. **Estratégia de Sinais 1D (Projeção Diagonal):** Esta abordagem baseia-se na hipótese de que as texturas dos tênis geram "assinaturas" únicas quando projetadas em um vetor unidimensional. A ideia central foi transformar a imagem 2D em um sinal 1D somando os pixels ao longo das diagonais (projeção anti diagonal). Esperava-se que as três listras da Adidas gerassem três picos periódicos (ondas) nesse sinal, enquanto Nike e Converse gerariam sinais mais ruidosos ou planos.
2. **Estratégia Morfológica e Geométrica (Excentricidade):** Nesta segunda abordagem, o foco mudou da textura para a forma global do objeto. Utilizou-se a segmentação para isolar o "blob" (região conexa) do tênis e extrair descritores de forma, especificamente a Excentricidade (o quão "esticado" ou "achatado" o objeto é) e a contagem de Componentes Conexos. A premissa era que o Nike, sendo mais aerodinâmico, teria alta excentricidade, enquanto o Adidas apresentaria múltiplos componentes desconexos devido à separação das listras.
3. **Estratégia de Transformadas e Solidez (Hough + Clusterização):** A terceira e definitiva abordagem, refinada a partir das falhas das anteriores, combinou a detecção robusta de linhas retas com métricas de preenchimento. Utilizou-se a Transformada de Hough (LHT) para detectar segmentos de reta, mas com um filtro inteligente de clusterização angular (agrupamento de linhas com ângulos similares) para confirmar o paralelismo da Adidas. Para o Converse, adotou-se a métrica de Solidez (relação área/convexo) para identificar a geometria maciça do cano alto.

---

## 4. Implementação Prática

A implementação prática foi dividida em três experimentos de código (Código 01, Código 02 e Código 03), cada um testando uma das estratégias metodológicas descritas acima. Abaixo detalham-se os algoritmos e métricas de cada versão:

### 4.1. Código 01: Análise de Sinais e Projeções

Neste primeiro experimento, a imagem foi tratada como uma matriz de intensidades para análise de frequência espacial.

- **Pré-processamento:** Conversão para escala de cinza e detecção de bordas com filtro Canny.
- **Adidas (Métrica de Picos):** A matriz de bordas foi rotacionada e somada ao longo do eixo vertical para gerar um histograma de projeção. O algoritmo busca por picos locais (máximos locais) nesse histograma.
    - **Critério:** Se o sinal apresentar 3 picos proeminentes com espaçamento regular (periodicidade), a imagem é classificada como Adidas.
- **Converse (Razão Estatística):** Calculou-se a dispersão dos pixels brancos nos eixos X e Y (StdX e StdY).
    -  **Critério:** Se a razão StdY/StdX for próxima de 1.0 (indicando uma forma "quadrada" ou circular, típica do logo ou do cano alto), classifica-se como Converse.
- **Nike:** Classificação por exclusão (padrões irregulares).

### 4.2. Código 02: Excentricidade e Componentes Conexos

Esta versão focou em simplificar a imagem para formas binárias puras.

- **Pré-processamento:** Binarização direta.
- **Métricas:**
    - `ecc` (Excentricidade): Mede o alongamento da elipse que melhor se ajusta ao objeto (0 = círculo, 1 = linha).
    - `components` (número de regiões): Número de regiões brancas separadas na imagem.
    - `orient_score` (orientação dos gradientes): Medida da orientação predominante dos gradientes.
- **Decisão:**
    - **Adidas:** `components >= 3` e orientação diagonal forte (`orient_score > 0.15`), ssume-se que os componentes são as listras separadas.
    - **Nike:** `ecc > 3.0`, classifica-se como Nike (objeto muito fino e alongado, típico de tênis de corrida de perfil baixo)..
    - **Converse:** Caso contrário (objeto sólido e pouco excêntrico).

### 4.3. Código 03: Transformada de Hough e Solidez (Exemplo para Imagem Única)

A abordagem final corrigiu a instabilidade dos códigos anteriores implementando "filtros de sanidade" mais rigorosos.

- **Pré-processamento Avançado:**
  - Crop de Segurança (5%) para remover molduras.
  - Gamma Correction (1.5) para realçar listras claras em tênis brancos.
  - Inversão Inteligente da máscara binária.
- **Algoritmo de Detecção:**
    - **Binarização por Limiarização Global (Otsu):** O algoritmo calcula o histograma da imagem e encontra o limiar ($T$) que minimiza a variância dentro de cada classe (fundo e objeto).
    - **Refinamento Morfológico (Fechamento):** O fechamento serve para conectar pequenos pontos que deveriam estar juntos, mas foram separados por ruído ou reflexos na foto.
    - **Detecção de Bordas via Gradiente (Sobel):** Como a imagem é binária após o Otsu, o gradiente será máximo exatamente na transição entre o logo e o tênis, gerando um contorno de 1 pixel de espessura que delimita perfeitamente as marcas.
    - **Análise de Componentes Conectados e Centros de Massa:** O algoritmo percorre a matriz procurando grupos de pixels brancos vizinhos (8-conectividade). Para cada grupo (listra), calculamos o Momento de Ordem Zero (área) e os Momentos de Primeira Ordem (centroide/centro de massa).
        - **Validação:** A prova final de que é um logo Adidas ocorre quando o algoritmo detecta três componentes cujos centros de massa estão alinhados e cujas áreas são estatisticamente semelhantes.

---

## 5. Estrutura dos Arquivos

O repositório está organizado da seguinte maneira:

- `Trabalho Final PIMG.ipynb`: Arquivo principal. Está segmentado em três blocos de código distintos, permitindo a visualização da progressão das técnicas:
    - _Células Iniciais:_ Sinais 1D.
    - _Células Intermediárias:_ Morfologia e Excentricidade.
    - _Células Finais:_ Solução Final (Hough + Solidez).
- `/imagens`: Diretório contendo as amostras utilizadas para teste (ex: `Adidas_01.jpg`, `Nike_03.jpg`, `Converse_03.jpg`).
- `README.md`: Este arquivo de documentação.
- `Link-Video-Youtube.txt`: Este arquivo contem o link da apresentação do vídeo no youtube.

---

## 6. Como Executar

Para reproduzir os experimentos:

1. Acesse o ambiente [Google Colab](https://colab.research.google.com/).
2. Faça o upload do arquivo `Trabalho Final PIMG.ipynb`.
3. Carregue as imagens de teste na aba de arquivos (menu lateral esquerdo) ou monte seu Google Drive.
4. Certifique-se de que os caminhos das imagens na lista de imagens teste (na última célula) correspondem aos arquivos carregados.
5. Execute as células sequencialmente para acompanhar a narrativa técnica:
    - **Código 01:** Tentativa via projeção de sinais.
    - **Código 02:** Tentativa via análise de "blobs" e excentricidade.
    - **Código 03:** Solução final e mais robusta (Hough + Geometria).

---

## 7. Vídeo de Apresentação

Foi gravado um vídeo demonstrativo explicando a lógica do código, as dificuldades encontradas e executando a classificação em tempo real.

🎥 **Link para o YouTube:** [[Apresentação Trabalho Final PIMG](https://youtu.be/6AvJCSvjRYQ)]

---

## 8. Dificuldades Encontradas

- **Generalização e Unificação do Código (O Maior Desafio):** A maior dificuldade encontrada foi criar um único algoritmo com um conjunto fixo de parâmetros que funcionasse para todas as imagens. Percebemos que os parâmetros ideais para segmentar um tênis Adidas (ex: um limiar de contraste alto para pegar listras) eram prejudiciais para a detecção do Nike (que exigia suavização para ignorar texturas).
    - **Consequência:** Ao tentar ajustar o código para corrigir o erro em uma imagem específica, frequentemente "quebrávamos" a classificação de outra imagem que já estava correta. Isso nos obrigou a criar ramificações lógicas complexas (if/else) baseadas em tentativas e erros, tratando quase cada caso de forma isolada, em vez de ter um modelo verdadeiramente genérico.

- **Sensibilidade da Transformada de Hough:** O algoritmo detectou "listras" onde não existiam (como costuras, cadarços ou sombras no chão). Foi necessário implementar filtros matemáticos rigorosos (cálculo de desvio padrão dos ângulos) para diferenciar o que era uma listra da marca Adidas de um ruído aleatório.

- **Binarização em Ambientes Não Controlados:** O método de Otsu falhou em diversas situações onde o tênis tinha cor similar ao fundo (ex: tênis branco em fundo branco ou tênis preto em fundo escuro). A segmentação do objeto, passo fundamental para o cálculo da Razão de Aspecto, foi comprometida nessas situações, gerando máscaras binárias imprecisas.

---

## 9. Conclusão

O trabalho cumpriu seu objetivo principal de explorar e aplicar técnicas fundamentais de Processamento Digital de Imagens (PDI) em um problema de classificação real. A restrição de não utilizar Inteligência Artificial nos forçou a investigar profundamente o funcionamento matemático de filtros e transformações.

Ao longo do desenvolvimento, a equipe testou três abordagens distintas. As duas primeiras (Sinais 1D e Excentricidade Pura) mostraram-se promissoras teoricamente, mas frágeis na prática diante de ruídos e variações de iluminação. A terceira abordagem (Híbrida: Hough + Solidez) apresentou os melhores resultados, conseguindo equilibrar a detecção de textura (listras da Adidas) com a geometria (forma do Converse).

Embora não exista um "algoritmo perfeito" único usando apenas técnicas clássicas para este cenário complexo, a combinação de múltiplos descritores (forma + textura) é o caminho mais viável. Este projeto evidenciou que a Engenharia de Características exige um processo iterativo de tentativa e erro, onde cada falha (Códigos 01 e 02) fornece o insight necessário para a construção de uma solução mais robusta (Código 03).

No entanto, é importante ressaltar que não foi possível obter uma solução universalmente robusta. A abordagem baseada em regras rígidas (Engenharia de Características Clássica) mostrou-se limitada diante da variabilidade intra-classe. Fatores como iluminação, ângulo da foto, cor do calçado e poluição visual no fundo impediram que um único algoritmo classifica-se 100% das imagens corretamente sem ajustes manuais específicos para cada caso.

Conclui-se que, embora as técnicas clássicas sejam poderosas para ambientes controlados e padrões industriais repetitivos, elas sofrem para generalizar em imagens naturais. Este projeto evidenciou a complexidade da Visão Computacional e justificou, na prática, a necessidade de abordagens estatísticas mais modernas (como Machine Learning e Deep Learning) para lidar com a diversidade de cenários do mundo real.
