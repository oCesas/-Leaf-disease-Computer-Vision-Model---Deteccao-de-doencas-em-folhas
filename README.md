# -Leaf-disease-Computer-Vision-Model---Deteccao-de-doencas-em-folhas
Aplicação de Processamento de Imagens desenvolvida com técnicas implementadas manualmente conforme conteúdos da disciplina. O projeto apresenta solução para desafio selecionado, com foco em originalidade, eficiência algorítmica e documentação clara, organizada em Jupyter Notebook.
Foi utilizado um desafio de Computer Model Vision do roboflow como tema principal: Link - https://universe.roboflow.com/roboflow-100/leaf-disease-nsdsr.

Este projeto implementa um pipeline completo de visão computacional para detectar e quantificar áreas afetadas por fungos em folhas. O sistema combina **índice ExG (Excess Green)** para segmentação da folha e **análise de textura multiescala** com **implementação manual de operações morfológicas** para identificar automaticamente regiões infectadas.


# 🍃 Detecção de Fungos em Folhas (90% de Precisão)

Sistema automatizado para detecção e análise de fungos em imagens de folhas utilizando técnicas avançadas de processamento digital de imagens **com implementação manual de operações morfológicas**.

##  Visão Geral

- Implementação manual de erosão, dilatação, abertura e fechamento morfológico
- Índice ExG para segmentação robusta de vegetação
- Variância de textura multiescala
- Processamento paralelo com joblib

##  Características

-  **Segmentação baseada em ExG** - Índice de excesso de verde
-  **Análise de textura multiescala** - Múltiplos tamanhos de janela
-  **Processamento paralelo** - Análise rápida usando múltiplos cores
-  **Visualizações informativas** - 3 plots (Original, Máscara, Detecção)
-  **Relatórios detalhados** - Lista de todos os fungos com posições
-  **Configuração flexível** - Parâmetros ajustáveis no início do código


### Pipeline de Processamento

```
Imagem RGB Original
    ↓
1. Segmentação da Folha (ExG)
    ├── Cálculo do índice ExG = 2*G - R - B
    ├── Limiarização por percentil
    ├── Preenchimento de buracos
    ├── Erosão das bordas (ROI interna)
    └── Remoção de pequenos objetos
    ↓
2. Análise de Textura Multiescala
    ├── Conversão para escala de cinza
    ├── Cálculo de variância em janelas deslizantes
    ├── Múltiplas escalas: [5, 9, 15] pixels
    ├── Processamento paralelo
    └── Normalização e média das escalas
    ↓
3. Detecção de Fungos
    ├── Limiarização da textura (percentil baixo)
    ├── Aplicação da máscara da folha
    ├── Remoção de regiões pequenas (< AREA_MIN)
    └── Operações morfológicas de fechamento
    ↓
4. Rotulação e Quantificação
    ├── Rotulação de componentes conectadas
    ├── Cálculo de áreas e centroides
    ├── Ordenação por tamanho
    └── Geração de estatísticas
    ↓
Resultado Final
```

### Técnicas Utilizadas

#### 1. **Índice ExG (Excess Green)**
- **Fórmula**: `ExG = 2*G - R - B`
- Realça vegetação separando-a do fundo
  
#### 2. **Variância de Textura**
- Detecta irregularidades na superfície da folha

#### 3. **Análise Multiescala**
- Janelas de tamanho variável: `[5, 9, 15]` pixels
- Detecta fungos de diferentes tamanhos
- Combina múltiplas resoluções espaciais
- **Processamento paralelo** para eficiência

#### 4. **Operações Morfológicas Manuais**
Implementação de:
- **Erosão**: Remove pequenas protuberâncias
- **Dilatação**: Preenche pequenos buracos
- **Abertura**: Erosão + Dilatação (remove ruído)
- **Fechamento**: Dilatação + Erosão (preenche espaços)

#### 5. **Rotulação de Componentes Conectadas**
- Identifica regiões separadas de fungos
- Implementação manual com pilha
- Conectividade 8-vizinhos
- Calcula área e centro de cada região

##  Requisitos

```
Python >= 3.12
numpy >= 1.24.0
matplotlib >= 3.7.0
scikit-image >= 0.21.0
tqdm >= 4.65.0
joblib >= 1.3.0
```

### Uso Básico

```python
# O código está totalmente no notebook
# Basta executar as células em sequência

# 1. Configure os parâmetros no início (PART 1):
PERCENTIL_EXG = 60              # Limiar para segmentação da folha
ESCALAS_TEXTURA = [5, 9, 15]    # Tamanhos de janela para textura
PERCENTIL_TEXTURA_BAIXA = 3     # Limiar para textura (menor = mais sensível)
AREA_MIN = 200                  # Área mínima de um fungo em pixels
EROSAO_BORDAS = 2               # Erosão da borda da folha

# 2. Execute a detecção:
resultado = detectar_todos_fungos("folha.jpg")

# 3. Visualize os resultados (automático)
# - Plot com 3 imagens (Original, Máscara, Detecção)
# - Arquivo PNG salvo automaticamente
# - Arquivo TXT com lista de fungos
```

### Exemplo Visual

O sistema gera automaticamente uma figura com 3 painéis lado a lado mostrando:
- A progressão da análise (original → segmentação → detecção)
- Contornos vermelhos destacando cada fungo detectado
- Título com estatísticas totais

##  Parâmetros Ajustáveis

Todos os parâmetros estão definidos no início do código e podem ser facilmente modificados:

### Configurações Principais

```python
# ================= PARÂMETROS =================

# Folha (ExG)
PERCENTIL_EXG = 60                    # Limiar para segmentação (40-80)
                                      # Maior = mais restritivo

# Textura multiescala  
ESCALAS_TEXTURA = [5, 9, 15]          # Tamanhos das janelas em pixels
                                      # Adicione mais para maior precisão

PERCENTIL_TEXTURA_BAIXA = 3           # Limiar de textura (1-10)
                                      # Menor = mais sensível a fungos

# Área mínima para região de fungo
AREA_MIN = 200                        # Pixels mínimos (100-500)
                                      # Maior = ignora manchas pequenas

# Erosão para ROI interna
EROSAO_BORDAS = 2                     # Pixels de erosão (0-5)
                                      # Maior = ignora bordas da folha

# ==============================================
```


##  Limitações

- **Fundo uniforme**: Funciona melhor com fundos contrastantes
- **Iluminação**: Variações extremas podem afetar o ExG
- **Textura similar**: Fungos com textura muito similar à folha saudável podem não ser detectados
- **Bordas da folha**: Erosão pode remover fungos próximos às bordas
- **Resolução**: Imagens muito pequenas podem ter resultados imprecisos
- **Processamento**: Múltiplas escalas podem ser lentas em imagens grandes

## 👥 Autor

- **Cesai Marinho De Carvalho** - *Desenvolvimento inicial* - [oCesas](https://github.com/oCesas)

## 📞 Contato

- Email: ocesas@outlook.com
- Linkedin: [Cesai Marinho](https://www.linkedin.com/in/cesai-marinho-527846264/)
### Documentação Técnica

- [NumPy Documentation](https://numpy.org/doc/)
- [Matplotlib Documentation](https://matplotlib.org/)
- [Scikit-image Documentation](https://scikit-image.org/)
- [Joblib Documentation](https://joblib.readthedocs.io/)


## 📚 Referências
- [Digital Image Processing - Gonzalez & Woods](https://www.imageprocessingplace.com/)
- [Computer Vision: Algorithms and Applications - Szeliski](https://szeliski.org/Book/)
