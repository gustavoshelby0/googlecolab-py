"""
Projeto de Análise de Dados de Vendas - Tech
Este script faz a leitura, limpeza, transformação e análise exploratória 
de dados de vendas de uma loja de tecnologia.
Autor: [Seu Nome]
Data: [Data]
"""

import pandas as pd
import numpy as np

# ----------------------------
# 1. LEITURA DOS DADOS
# ----------------------------

# Leitura do arquivo CSV de vendas (com low_memory=False para evitar warnings de tipo)
df_vendas = pd.read_csv('vendas_tech.csv', low_memory=False)
print("DataFrame de Vendas carregado:")
print(df_vendas)

# Leitura do arquivo Excel com a lista de gerentes e metas
df_gerentes = pd.read_excel('gerentes_lojas.xlsx')
print("\nDataFrame de Gerentes carregado:")
print(df_gerentes)

# ----------------------------
# 2. INSPEÇÃO INICIAL DOS DADOS (Análise Exploratória)
# ----------------------------

print("\n--- Inspeção Inicial ---")

# Exibe as primeiras 15 linhas
print("Primeiras 15 linhas:")
print(df_vendas.head(15))

# Exibe as últimas 15 linhas
print("\nÚltimas 15 linhas:")
print(df_vendas.tail(15))

# Exibe 15 linhas de forma aleatória (amostra)
print("\nAmostra de 15 linhas aleatórias:")
print(df_vendas.sample(15))

# Mostra quantas linhas e colunas o DataFrame possui (formato: (linhas, colunas))
print("\nDimensões do DataFrame (linhas, colunas):")
print(df_vendas.shape)

# Lista todas as colunas existentes
print("\nNomes das colunas:")
print(df_vendas.columns)

# Resumo geral: tipos de dados (dtypes), memória, contagem de não-nulos, etc.
print("\nResumo das informações do DataFrame (info):")
print(df_vendas.info())

# Estatísticas descritivas: média, desvio padrão, quartis, máximo, mínimo, contagem
print("\nEstatísticas descritivas (describe):")
print(df_vendas.describe())

# ----------------------------
# 3. TRATAMENTO E LIMPEZA DOS DADOS
# ----------------------------

print("\n--- Iniciando Limpeza e Tratamento ---")

# Visualizando colunas específicas para conferência
print("Colunas 'Loja' e 'Cliente' (primeiras 5):")
print(df_vendas[['Loja', 'Cliente']].head())

# Cria uma cópia do DataFrame para trabalharmos sem perder o original
df_analise = df_vendas.copy()

# Exclui a coluna 'Data_Base', pois ela possui apenas 1 linha preenchida (dados inúteis para a análise)
df_analise = df_analise.drop(columns=['Data_Base'])

# Tratamento de valores nulos (NaN)
# Preenche os valores vazios da coluna 'Loja' com a string "Online"
# (Isso indica que, se a loja não foi informada, a venda foi feita pelo canal online)
df_analise['Loja'] = df_analise['Loja'].fillna('Online')

# Conversão do tipo da coluna 'Data' para datetime
# O formato esperado é 'YYYY-MM-DD' (ex: 2026-12-31)
df_analise['Data'] = pd.to_datetime(df_analise['Data'], format='%Y-%m-%d')

# ----------------------------
# 4. PADRONIZAÇÃO DE TEXTOS (STRINGS)
# ----------------------------

# Remove espaços em branco das laterais (strip)
df_analise['Loja'] = df_analise['Loja'].str.strip()

# Converte a primeira letra de cada palavra para maiúscula (title) 
# Exemplo: "sao paulo" -> "Sao Paulo", "SP" -> "Sp" (cuidado: "SP" vira "Sp", mas mantemos)
df_analise['Loja'] = df_analise['Loja'].str.title()

# ----------------------------
# 5. TRATAMENTO DA BASE DE GERENTES (Para não poluir a base de vendas)
# ----------------------------

# Aplicamos a MESMA padronização na coluna 'Loja' do DataFrame de gerentes
# (Isso é essencial para conseguirmos fazer o merge/junção corretamente depois)
df_gerentes['Loja'] = df_gerentes['Loja'].str.strip()
df_gerentes['Loja'] = df_gerentes['Loja'].str.title()

# ----------------------------
# 6. REMOÇÃO DE DUPLICATAS
# ----------------------------

# Remove linhas duplicadas baseadas na coluna 'ID_Pedido' (mantém apenas a primeira ocorrência)
df_analise = df_analise.drop_duplicates(subset=['ID_Pedido'])

# Exibe um resumo do DataFrame após as limpezas
print("\nResumo do DataFrame após limpeza (info):")
print(df_analise.info())

# ----------------------------
# 7. CRIAÇÃO DE NOVAS COLUNAS (FEATURE ENGINEERING)
# ----------------------------

print("\n--- Criando Novas Colunas ---")

# Cria a coluna 'Faturamento' (Quantidade * Preço Unitário)
df_analise['Faturamento'] = df_analise['Qtd'] * df_analise['Preco_Unitario']

# Cria a coluna 'Forma_de_Venda' baseada na coluna 'Loja'
# Se a Loja for "Online", a Forma de Venda é "Online". Caso contrário, é "Presencial".
df_analise['Forma_de_Venda'] = np.where(df_analise['Loja'] == 'Online', 'Online', 'Presencial')

# ----------------------------
# 8. MAPEAMENTO DE REGIÕES (Dicionário)
# ----------------------------

# Exibe as lojas únicas para conferência
print("\nLojas únicas disponíveis no DataFrame:")
print(df_analise['Loja'].unique())

# Dicionário que mapeia cada Loja para sua respectiva Região (funciona como um "tradutor")
dic_regioes = {
    'Sao Paulo': 'Sudeste',
    'Belo Horizonte': 'Sudeste',
    'Online': 'Online',
    'Rio de Janeiro': 'Sudeste',
    'Salvador': 'Nordeste',
    'Recife': 'Nordeste',
    'Curitiba': 'Sul',
    'Porto Alegre': 'Sul'
}

# Aplica o dicionário à coluna 'Loja' para criar a nova coluna 'Regiao'
df_analise['Regiao'] = df_analise['Loja'].map(dic_regioes)

# Verifica se há valores nulos na nova coluna 'Regiao'
print("\nQuantidade de valores nulos por coluna (após adicionar 'Regiao'):")
print(df_analise.isna().sum())

# ----------------------------
# 9. ORDENAÇÃO E REINDEXAÇÃO
# ----------------------------

# Ordena o DataFrame pela 'Data' e depois pelo 'Faturamento'
df_analise = df_analise.sort_values(by=['Data', 'Faturamento'])

# Reinicia os índices para ficarem sequenciais (0, 1, 2, ...)
df_analise = df_analise.reset_index(drop=True)

# ----------------------------
# 10. FILTRAGEM E CONSULTAS ESPECÍFICAS (.loc e .iloc)
# ----------------------------

print("\n--- Exemplos de Filtros e Consultas ---")

# Usando .loc para buscar dados baseados no ID do Pedido (filtro por condição)
id_pedido = 4
loja = df_analise.loc[df_analise['ID_Pedido'] == id_pedido, 'Loja'].values[0]
produto = df_analise.loc[df_analise['ID_Pedido'] == id_pedido, 'Produto'].values[0]
cliente = df_analise.loc[df_analise['ID_Pedido'] == id_pedido, 'Cliente'].values[0]
print(f"ID_Pedido {id_pedido} -> Loja: {loja}, Produto: {produto}, Cliente: {cliente}")

# Usando .iloc para buscar pela posição (linha 3, coluna 2 - lembrando que índices começam em 0)
# Neste caso, buscamos o valor na 4ª linha e 3ª coluna (posição)
print("\nBuscando valores com .iloc (posição 3, 2):")
print(f"Valor na linha 3, coluna 2: {df_analise.iloc[3, 2]}")

# ----------------------------
# 11. EXPORTAÇÃO DE DATAFRAMES FILTRADOS
# ----------------------------

# Filtra apenas as vendas da loja 'Sao Paulo' e exporta para CSV
df_vendas_sp = df_analise[df_analise['Loja'] == 'Sao Paulo']
df_vendas_sp.to_csv('Vendas_SP.csv', index=False)
print("\nArquivo 'Vendas_SP.csv' exportado com sucesso!")

# Filtra as vendas apenas do ano de 2024 (ano inteiro)
df_vendas_2024 = df_analise[df_analise['Data'].dt.year == 2024]
print(f"\nVendas de 2024: {df_vendas_2024.shape[0]} linhas encontradas.")

# ----------------------------
# 12. FILTROS COM DUPLAS CONDIÇÕES
# ----------------------------

# Exemplo: Vendas de 'Cabo HDMI' na região 'Sul'
df_vendas_hdmi_sul = df_analise[
    (df_analise['Produto'] == 'Cabo HDMI') & (df_analise['Regiao'] == 'Sul')
]
print(f"\nVendas de Cabo HDMI na Região Sul: {df_vendas_hdmi_sul.shape[0]} linhas.")

# ----------------------------
# 13. ANÁLISES POR AGRUPAMENTO (GROUPBY)
# ----------------------------

print("\n--- Análises de Agrupamento (Rankings) ---")

# RANK 1: Faturamento total por Loja
analise_lojas = df_analise[['Loja', 'Faturamento']].groupby('Loja').sum()
analise_lojas = analise_lojas.sort_values(by='Faturamento', ascending=False)
analise_lojas = analise_lojas.reset_index()
# Formata a coluna 'Faturamento' para o padrão de moeda brasileira (R$)
analise_lojas['Faturamento'] = analise_lojas['Faturamento'].map('R${:,.2f}'.format)
print("\nRanking de Faturamento por Loja (do maior para o menor):")
print(analise_lojas)

# RANK 2: Produtos que mais venderam no canal Online (com base na quantidade)
df_analise_online = df_analise[df_analise['Loja'] == 'Online']
analise_produtos_online = df_analise_online[['Produto', 'Qtd']].groupby('Produto').sum()
analise_produtos_online = analise_produtos_online.rename(columns={'Qtd': 'Vendas Totais'})
print("\nRanking de Produtos mais vendidos no canal Online (por quantidade):")
print(analise_produtos_online.sort_values(by='Vendas Totais', ascending=False))

# RANK 3: Quais produtos venderam mais em CADA loja (agrupamento duplo)
df_produtos_em_lojas = df_analise[['Loja', 'Produto', 'Qtd']].groupby(['Loja', 'Produto']).sum()
print("\nVendas por Produto em cada Loja (amostra):")
print(df_produtos_em_lojas.head(10))

# ----------------------------
# 14. ANÁLISE DE META DOS GERENTES (JANEIRO/2023)
# ----------------------------

print("\n--- Análise de Metas dos Gerentes (Jan/2023) ---")

# Filtra os dados para Janeiro de 2023
df_meta = df_analise[
    (df_analise['Data'].dt.year == 2023) & (df_analise['Data'].dt.month == 1)
]

# Agrupa o faturamento por Loja para o período filtrado
df_meta = df_meta[['Loja', 'Faturamento']].groupby('Loja', as_index=False).sum()

# Junta (merge) com a tabela de gerentes para trazer a coluna 'Meta_Mensal'
df_meta = df_meta.merge(df_gerentes, on='Loja', how='left')

# Cria uma coluna indicando se o gerente bateu a meta ou não
df_meta['Bateu a Meta?'] = np.where(df_meta['Faturamento'] >= df_meta['Meta_Mensal'], 'Sim', 'Não')

print("Resultado de Metas para Janeiro/2023:")
print(df_meta)

# ----------------------------
# 15. ANÁLISE TEMPORAL E VISUALIZAÇÃO (PLOT)
# ----------------------------

print("\n--- Análise Temporal ---")

# Cria uma coluna com o mês/ano no formato 'YYYY-MM' (ex: 2023-01)
df_analise['Mes-Ano'] = df_analise['Data'].dt.to_period("M")

# Agrupa o faturamento por mês/ano
df_vendas_mes = df_analise[['Mes-Ano', 'Faturamento']].groupby('Mes-Ano').sum()

# Gera um gráfico de linha (evolução do faturamento ao longo do tempo)
# (Se estiver no Jupyter/Colab, o gráfico será exibido automaticamente. 
#  Se estiver rodando como script .py, ele abrirá uma janela com o gráfico)
df_vendas_mes.plot(title="Evolução do Faturamento Mensal", ylabel="Faturamento (R$)")

# Exibe o DataFrame final para conferência
print("\nDataFrame final tratado e enriquecido (primeiras 5 linhas):")
print(df_analise.head())

print("\n--- Fim do Script ---")
