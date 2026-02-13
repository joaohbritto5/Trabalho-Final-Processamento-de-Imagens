# Trabalho-Final-Processamento-de-Imagens

# Classificação de Tênis (Nike, Adidas e Converse)

## 1. Integrantes e Organização
* **Instituição:** Universidade Federal de Sergipe (UFS) - Departamento de Computação
* **Disciplina:** Processamento de Imagens (COMP 0432)
* **Docente:** Prof. Dr. Leonardo Nogueira Matos
* **Semestre:** 2025.2
* **Discentes:**
  * Arthur Costa Oliveira - 202300027104
  * João Henrique Britto Bomfim - 202300027409

---

## 2. Descrição do Projeto
O projeto tem como objetivo desenvolver um algoritmo capaz de classificar automaticamente imagens de tênis em três categorias de marcas distintas: **Adidas, Converse e Nike**. 

Diferente de abordagens modernas que utilizam Redes Neurais Convolucionais (Deep Learning), este projeto foca na **Engenharia de Características (Feature Engineering)**. O desafio, proposto originalmente na plataforma Roboflow, consiste em identificar padrões geométricos e de textura (como as três listras da Adidas ou o cano alto do Converse) utilizando apenas conceitos fundamentais de Processamento Digital de Imagens vistos em sala de aula, sem o uso de bibliotecas de alto nível como OpenCV ou PIL.

---

## 3. Metodologia
A metodologia adotada para este trabalho não foi linear, mas sim exploratória e iterativa. Dado o desafio de classificar imagens com alta variabilidade intra-classe sem o uso de Aprendizado de Máquina, a equipe optou por investigar três estratégias distintas:

1. **Estratégia de Sinais 1D (Projeção Diagonal):** Baseia-se na hipótese de que as texturas dos tênis geram "assinaturas" únicas quando projetadas em um vetor unidimensional. A ideia central foi transformar a imagem 2D em um sinal 1D somando os pixels ao longo das diagonais. Esperava-se que as três listras da Adidas gerassem três picos periódicos, enquanto Nike e Converse gerariam sinais mais ruidosos ou planos.
2. **Estratégia Morfológica e Geométrica (Excentricidade):** O foco mudou da textura para a forma global do objeto. Utilizou-se a segmentação para isolar a região conexa do tênis e extrair descritores de forma, especificamente a **Excentricidade** e a contagem de **Componentes Conexos**. A premissa era que o Nike (mais aerodinâmico) teria alta excentricidade, enquanto o Adidas apresentaria múltiplos componentes desconexos.
3. **Estratégia de Transformadas e Solidez (Hough + Clusterização):** A abordagem definitiva, refinada a partir das anteriores. Combinou a detecção robusta de linhas retas com métricas de preenchimento. Utilizou-se a **Transformada de Hough (LHT)** para detectar segmentos de reta, com um filtro de clusterização angular para confirmar o paralelismo da Adidas. Para o Converse, adotou-se a métrica de **Solidez** para identificar a geometria maciça do cano alto.

---

## 4. Implementação Prática
A implementação foi dividida em três experimentos de código, testando as estratégias descritas acima:

### 4.1. Código 01: Análise de Sinais e Projeções
* **Pré-processamento:** Conversão para escala de cinza e detecção de bordas com filtro Canny.
* **Adidas (Métrica de Picos):** Busca por picos locais em um histograma de projeção. Se houver 3 picos proeminentes com espaçamento regular, é classificado como Adidas.
* **Converse (Razão Estatística):** Cálculo da dispersão dos pixels brancos (StdX e StdY). Se a razão StdY/StdX for próxima de 1.0 (forma circular/quadrada), é Converse.
* **Nike:** Classificação por exclusão (padrões irregulares).

### 4.2. Código 02: Excentricidade e Componentes Conexos
* **Pré-processamento:** Binarização direta.
* **Métricas:** `ecc` (Excentricidade), `components` (número de regiões) e `orient_score` (orientação dos gradientes).
* **Decisão:**
  * **Adidas:** `components >= 3` e orientação diagonal forte (`orient_score > 0.15`).
  * **Nike:** `ecc > 3.0` (objeto muito fino e alongado).
  * **Converse:** Caso contrário (objeto sólido e pouco excêntrico).

### 4.3. Código 03: Transformada de Hough e Solidez (Final)
* **Pré-processamento Avançado:** Crop de Segurança (5%), Gamma Correction (1.5) e Inversão Inteligente da máscara binária.
* **Decisão:**
  * **Adidas (Cluster de Hough):** Aplica-se a Transformada de Hough em bordas (Sobel). Se detectar 3 linhas agrupadas ou 2 linhas perfeitamente paralelas na faixa diagonal (20°-85°), é Adidas.
  * **Converse (Geometria Sólida):** Razão de Aspecto > 0.60 E Solidez > 0.62 (captura o formato "tijolão" do cano alto).
  * **Nike:** Falha nos testes de paralelismo e geometria alta.

---

## 5. Estrutura dos Arquivos
O repositório está organizado da seguinte maneira:

* `Trabalho Final PIMG.ipynb`: Arquivo principal. Está segmentado em três blocos de código distintos, permitindo a visualização da progressão das técnicas:
  * *Células Iniciais:* Sinais 1D.
  * *Células Intermediárias:* Morfologia e Excentricidade.
  * *Células Finais:* Solução Final (Hough + Solidez).
* `/imagens`: Diretório contendo as amostras utilizadas para teste (ex: `Adidas_01.jpg`, `Nike_03.jpg`, `Converse_03.jpg`).
* `README.md`: Este arquivo de documentação.

---

## 6. Como Executar
Para reproduzir os experimentos:

1. Acesse o ambiente [Google Colab](https://colab.research.google.com/).
2. Faça o upload do arquivo `Trabalho Final PIMG.ipynb`.
3. Carregue as imagens de teste na aba de arquivos (menu lateral esquerdo) ou monte seu Google Drive.
4. Certifique-se de que os caminhos das imagens na lista de imagens teste (na última célula) correspondem aos arquivos carregados.
5. Execute as células sequencialmente para acompanhar a narrativa técnica:
   * **Código 01:** Tentativa via projeção de sinais.
   * **Código 02:** Tentativa via análise de "blobs" e excentricidade.
   * **Código 03:** Solução final e mais robusta (Hough + Geometria).

---

## 7. Vídeo de Apresentação
Foi gravado um vídeo demonstrativo explicando a lógica do código, as dificuldades encontradas e executando a classificação em tempo real.

🎥 **Link para o YouTube:** [INSERIR LINK AQUI]

---

## 8. Dificuldades Encontradas
* **Generalização e Unificação do Código:** A maior dificuldade foi criar um único algoritmo com parâmetros fixos que funcionasse para todas as imagens. Ajustes para acertar uma imagem frequentemente "quebravam" outra.
* **Sensibilidade da Transformada de Hough:** O algoritmo detectou "listras" em costuras e cadarços. Foi necessário implementar filtros matemáticos (desvio padrão dos ângulos) para diferenciar listras reais de ruído.
* **Binarização em Ambientes Não Controlados:** O método de Otsu falhou quando o tênis tinha cor similar ao fundo (ex: branco no branco), prejudicando a extração de máscaras e o cálculo da Razão de Aspecto.

---

## 9. Conclusão
O trabalho cumpriu seu objetivo principal de explorar técnicas de PDI. A restrição de não utilizar IA forçou a investigação profunda do funcionamento matemático de filtros e transformações. 

A abordagem híbrida (Hough + Solidez) obteve os melhores resultados, mas evidenciou que a Engenharia de Características Clássica sofre com a alta variabilidade intra-classe (iluminação, ângulos, fundo). Conclui-se que, embora técnicas clássicas sejam poderosas para ambientes controlados, a complexidade de imagens naturais justifica a necessidade de abordagens estatísticas modernas (Machine Learning/Deep Learning) para lidar com cenários do mundo real.
