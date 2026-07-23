# Projeto de Manutenção Preditiva Industrial

Este repositório contém a solução desenvolvida para prever falhas operacionais em maquinários industriais através de dados de sensores e algoritmos de Machine Learning.

---

## 1. Coleta de Dados

### Problemática e Relevância
Na Indústria 4.0, paradas não planejadas (*downtime*) geram elevados prejuízos financeiros e riscos à segurança operacional. A **Manutenção Preditiva** substitui o modelo reativo (esperar quebrar) e o modelo preventivo baseado apenas em tempo, permitindo prever a falha antes que ela ocorra com base na condição real do equipamento.

### Fonte dos Dados
* **Origem:** *AI4I 2020 Predictive Maintenance Dataset*, disponibilizado pelo repositório **UCI Machine Learning Repository**.
* **Volume:** 10.000 instâncias (registros operacionais de fresadoras CNC).
* **Atributos Analisados:**
  * **Variáveis de Temperatura:** Temperatura do Ar (`Air temperature [K]`) e do Processo (`Process temperature [K]`).
  * **Parâmetros Mecânicos:** Velocidade de Rotação (`Rotational speed [rpm]`), Torque (`Torque [Nm]`) e Tempo de Desgaste da Ferramenta (`Tool wear [min]`).
  * **Qualidade do Produto:** Tipo do Produto (`Type` - L, M, H).
  * **Variável Alvo (*Target*):** `Machine failure` (0 = Operação Normal, 1 = Falha).
  * **Subtipos de Falha:** Falha de Desgaste (TWF), Dissipação de Calor (HDF), Energia (PWF), Sobrecarga (OSF) e Aleatória (RNF).

---

## 2. Modelagem

### Pré-processamento e Engenharia de Atributos
1. **Engenharia de Atributos (*Feature Engineering*):** Criação da variável física derivada `Power [W]` (Potência Mecânica), calculada a partir da velocidade de rotação e do torque ($P = \text{Torque} \times \text{RPM} \times \frac{2\pi}{60}$), fundamental para mapear falhas de energia (PWF) e sobrecarga (OSF).
2. **Tratamento de Dados:** Codificação de variáveis categóricas e padronização das escalas numéricas (`StandardScaler`).
3. **Tratamento do Desbalanceamento de Classes:** Aplicação do **SMOTE** (*Synthetic Minority Over-sampling Technique*) inserido dentro de *Pipelines* de treino para prevenir o vazamento de dados (*Data Leakage*).

### Treinamento e Avaliação dos Modelos
Foram avaliados e comparados os algoritmos **Random Forest Classifier** e **XGBoost Classifier** utilizando Validação Cruzada (*Cross-Validation*) e Busca por Hiperparâmetros (*GridSearchCV*).

**Resumo de Desempenho no Conjunto de Teste (utilizando Cross:**

| Modelo | Métrica Foco | Acurácia | Precisão | Recall (Sensibilidade) | F1-Score |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **XGBoost (Otimizado F1)** | Equilíbrio Operacional | ~98% | ~70% | ~82% | ~76% |
| **XGBoost (Otimizado Recall)** | Máxima Segurança / Detecção | ~96% | ~41% | ~94% | ~57% |
| **Random Forest (Otimizado F1)** | Balanço Conservador | ~98% | ~64% | ~78% | ~70% |
| **Random Forest (Otimizado Recall)** | Alta Sensibilidade Intermediária | ~97% | ~49% | ~87% | ~63% |

---

## 3. Conclusões e Recomendações de Negócio

### Principais Insights
1. **Atributos Determinantes:** O desgaste da ferramenta (`Tool wear`), o torque e a potência mecânica calculada foram identificados como os principais preditores de quebra mecânica.
2. **Métrica Crítica:** A Acurácia isolada não é adequada devido ao alto desbalanceamento (apenas ~3.4% de falhas). A avaliação focou nas métricas de **Recall** (garantir a captura das falhas realistas) e **F1-Score** (manter o equilíbrio comercial).

### Estratégia de Implantação
1. **Modelo Primário Padrão (XGBoost Otimizado para F1-Score):**
   * **Aplicação:** Rotina operacional diária.
   * **Justificativa:** Entrega a melhor combinação entre confiabilidade dos alertas (70% de precisão) e alta taxa de detecção (82%), minimizando os custos de paradas desnecessárias e deslocamento indevido da equipe técnica.
2. **Modelo Secundário de Segurança (XGBoost Otimizado para Recall):**
   * **Aplicação:** Maquinários críticos de alto custo onde a parada por quebra é inaceitável (*Early Warning System*).
   * **Justificativa:** Alcança ~94% de captura de falhas reais, garantindo a continuidade operacional das linhas mais estratégicas da fábrica.
