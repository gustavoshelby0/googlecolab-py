# ====================================================================
# PROJETO DE ANÁLISE DE DADOS - VENDAS TECH
# ====================================================================
# Autor: [Seu Nome]
# Objetivo: Limpeza, tratamento e análise exploratória de vendas.
# Status: Código 100% funcional e comentado bloco a bloco.
# ====================================================================

# %%
# ====================================================================
# BLOCO 1 - IMPORTAÇÃO DAS BIBLIOTECAS
# ====================================================================
# Aqui carregamos as ferramentas que vamos usar.
# Pandas -> manipulação de tabelas (DataFrames)
# Numpy -> cálculos matemáticos e vetorizados
# ====================================================================
import pandas as pd
import numpy as np

print("Bibliotecas importadas com sucesso!")

# %%
# ====================================================================
# BLOCO 2 - LEITURA DOS ARQUIVOS (CSV e EXCEL)
# ====================================================================
# low_memory=False evita warnings sobre tipos mistos no CSV.
# ====================================================================
df_vendas = pd.read_csv('vendas_tech.csv', low_memory=False)
df_gerentes = pd.read_excel('gerentes_lojas.xlsx')

print("Arquivos carregados:")
print(f"Vendas: {df_vendas.shape[0]} linhas e {df_vendas.shape[1]} colunas")
print(f"Gerentes: {df_gerentes.shape[0]} linhas e {df_gerentes.shape[1]} colunas")

# %%
# ====================================================================
# BLOCO 3 - INSPEÇÃO INICIAL DOS DADOS (ANÁLISE EXPLORATÓRIA - EDA)
# ====================================================================
# Objetivo: "Conhecer" os dados antes de mexer.
# head() -> primeiras linhas | tail() -> últimas | sample() -> aleatórias
# shape -> dimensões | columns -> nomes das colunas
# info() -> tipos e contagem de não-nulos | describe() -> estatísticas
# ====================================================================
print("\n--- VISUALIZAÇÃO GERAL ---")
print("Primeiras 15 linhas:")
print(df_vendas.head(15))

print("\nÚltimas 15 linhas:")
print(df_vendas.tail(15))

print("\nAmostra aleatória de 15 linhas:")
print(df_vendas.sample(15))

print(f"\nDimensões do DataFrame (linhas, colunas): {df_vendas.shape}")

print("\nNomes de todas as colunas:")
print(df_vendas.columns.tolist())

print("\nResumo dos tipos de dados (info):")
print(df_vendas.info())

print("\nEstatísticas descritivas (describe):")
print(df_vendas.describe())

# %%
# ====================================================================
# BLOCO 4 - TRATAMENTO INICIAL E LIMPEZA (NULOS E TIPOS)
# ====================================================================
# 1. Criamos uma CÓPIA do df_vendas para não perder o original.
# 2. Removemos a coluna 'Data_Base' (inútil, só tem 1 dado preenchido).
# 3. Preenchemos os valores vazios (NaN) da coluna 'Loja' com "Online".
# 4. Convertemos a coluna 'Data' para o tipo DATETIME (data/hora).
# ====================================================================
df_analise = df_vendas.copy()

# Excluindo a coluna inútil
df_analise = df_analise.drop(columns=['Data_Base'])

# Substituindo NaN por "Online" na coluna Loja
df_analise['Loja'] = df_analise['Loja'].fillna('Online')

# Convertendo a coluna de Data para o formato datetime (YYYY-MM-DD)
df_analise['Data'] = pd.to_datetime(df_analise['Data'], format='%Y-%m-%d')

print("Coluna 'Data_Base' removida, nulos tratados e 'Data' convertida.")
print(df_analise[['Loja', 'Data']].head())

# %%
# ====================================================================
# BLOCO 5 - PADRONIZAÇÃO DE TEXTOS (STRINGS)
# ====================================================================
# Dados vêm bagunçados: "Sao Paulo ", "sao paulo", "SP".
# strip() -> remove espaços sobrando nas laterais.
# title() -> deixa a primeira letra de cada palavra Maiúscula.
# Ex: "sao paulo " vira "Sao Paulo".
# ====================================================================
df_analise['Loja'] = df_analise['Loja'].str.strip()
df_analise['Loja'] = df_analise['Loja'].str.title()

print("Textos da coluna 'Loja' padronizados.")
print("Lojas únicas agora:", df_analise['Loja'].unique())

# %%
# ====================================================================
# BLOCO 6 - TRATAMENTO DA TABELA DE GERENTES (PADRONIZAÇÃO)
# ====================================================================
# Atenção: Não podemos mexer na coluna 'Loja' do df_analise usando a 
# lista de gerentes. Cada tabela tem suas próprias linhas.
# Aqui padronizamos APENAS o df_gerentes para depois conseguir juntar (merge).
# ====================================================================
df_gerentes['Loja'] = df_gerentes['Loja'].str.strip()
df_gerentes['Loja'] = df_gerentes['Loja'].str.title()

print("Coluna 'Loja' na tabela de Gerentes padronizada.")
print(df_gerentes.head())

# %%
# ====================================================================
# BLOCO 7 - REMOÇÃO DE LINHAS DUPLICADAS
# ====================================================================
# Se o mesmo 'ID_Pedido' aparece duas vezes, mantemos apenas o primeiro.
# Isso evita distorções nas contas de faturamento.
# ====================================================================
df_analise = df_analise.drop_duplicates(subset=['ID_Pedido'])

print(f"Linhas restantes após remover duplicatas: {df_analise.shape[0]}")

# %%
# ====================================================================
# BLOCO 8 - CRIAÇÃO DE NOVAS COLUNAS (FEATURE ENGINEERING)
# ====================================================================
# 1. Faturamento = Quantidade * Preço Unitário (Total da venda).
# 2. Forma de Venda: Se Loja == "Online" -> "Online", senão "Presencial".
# ====================================================================
df_analise['Faturamento'] = df_analise['Qtd'] * df_analise['Preco_Unitario']

df_analise['Forma_de_Venda'] = np.where(
    df_analise['Loja'] == 'Online', 
    'Online', 
    'Presencial'
)

print("Colunas 'Faturamento' e 'Forma_de_Venda' criadas.")
print(df_analise[['Qtd', 'Preco_Unitario', 'Faturamento', 'Forma_de_Venda']].head())

# %%
# ====================================================================
# BLOCO 9 - MAPEAMENTO DE REGIÕES (DICIONÁRIO)
# ====================================================================
# Criamos um dicionário que funciona como um "tradutor".
# Se na coluna 'Loja' aparecer 'Sao Paulo', o map() coloca 'Sudeste'.
# ====================================================================
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

df_analise['Regiao'] = df_analise['Loja'].map(dic_regioes)

print("Coluna 'Regiao' adicionada.")
print("Verificando se há nulos na nova coluna:")
print(df_analise['Regiao'].isna().sum())

# %%
# ====================================================================
# BLOCO 10 - ORDENAÇÃO E REINDEXAÇÃO
# ====================================================================
# sort_values -> ordena primeiro por 'Data' (mais antiga) e depois por 'Faturamento'.
# reset_index(drop=True) -> renumerar os índices de 0 até o fim, sem criar coluna extra.
# ====================================================================
df_analise = df_analise.sort_values(by=['Data', 'Faturamento'])
df_analise = df_analise.reset_index(drop=True)

print("DataFrame ordenado e índices resetados.")

# %%
# ====================================================================
# BLOCO 11 - FILTROS E CONSULTAS (.loc e .iloc)
# ====================================================================
# .loc -> busca por NOME da linha (condição) e NOME da coluna.
# .iloc -> busca por POSIÇÃO (número da linha e número da coluna).
# ====================================================================
id_pedido = 4
loja = df_analise.loc[df_analise['ID_Pedido'] == id_pedido, 'Loja'].values[0]
produto = df_analise.loc[df_analise['ID_Pedido'] == id_pedido, 'Produto'].values[0]
print(f"ID {id_pedido}: Loja={loja}, Produto={produto}")

# Exemplo de .iloc (linha 3, coluna 2 -> lembrando que começa do 0)
print(f"Valor na posição [3, 2]: {df_analise.iloc[3, 2]}")

# %%
# ====================================================================
# BLOCO 12 - EXPORTAÇÃO DE DATAFRAMES FILTRADOS (CSV)
# ====================================================================
# Salvamos pedaços do DataFrame em arquivos .csv separados.
# Isso é útil para entregar relatórios parciais.
# ====================================================================
# Vendas apenas da loja São Paulo
df_vendas_sp = df_analise[df_analise['Loja'] == 'Sao Paulo']
df_vendas_sp.to_csv('Vendas_SP.csv', index=False)
print("Arquivo 'Vendas_SP.csv' salvo!")

# Vendas apenas do ano de 2024
df_vendas_2024 = df_analise[df_analise['Data'].dt.year == 2024]
print(f"Total de vendas em 2024: {df_vendas_2024.shape[0]} linhas.")

# %%
# ====================================================================
# BLOCO 13 - FILTROS COM DUPLAS CONDIÇÕES
# ====================================================================
# Uso de & (E) para unir duas condições.
# Exemplo: Produto é 'Cabo HDMI' E Regiao é 'Sul'.
# ====================================================================
df_hdmi_sul = df_analise[
    (df_analise['Produto'] == 'Cabo HDMI') & (df_analise['Regiao'] == 'Sul')
]
print(f"Vendas de Cabo HDMI na Região Sul: {df_hdmi_sul.shape[0]} registros.")

# %%
# ====================================================================
# BLOCO 14 - ANÁLISES POR AGRUPAMENTO (GROUPBY) - RANKINGS
# ====================================================================
# groupby -> junta os dados por categoria e aplica uma função (sum, mean, etc).
# 1. Ranking de Faturamento por Loja.
# 2. Ranking de Produtos mais vendidos no Online.
# ====================================================================
# Ranking 1: Lojas que mais faturaram
rank_lojas = df_analise[['Loja', 'Faturamento']].groupby('Loja').sum()
rank_lojas = rank_lojas.sort_values(by='Faturamento', ascending=False)
rank_lojas = rank_lojas.reset_index()
rank_lojas['Faturamento'] = rank_lojas['Faturamento'].map('R${:,.2f}'.format)
print("\n--- RANKING DE FATURAMENTO POR LOJA ---")
print(rank_lojas)

# Ranking 2: Produtos mais vendidos (em quantidade) no canal Online
df_online = df_analise[df_analise['Loja'] == 'Online']
rank_prod_online = df_online[['Produto', 'Qtd']].groupby('Produto').sum()
rank_prod_online = rank_prod_online.rename(columns={'Qtd': 'Vendas Totais'})
rank_prod_online = rank_prod_online.sort_values(by='Vendas Totais', ascending=False)
print("\n--- RANKING DE PRODUTOS ONLINE (por Quantidade) ---")
print(rank_prod_online.head())

# %%
# ====================================================================
# BLOCO 15 - ANÁLISE DE METAS DOS GERENTES (MERGE + JANEIRO/2023)
# ====================================================================
# 1. Filtramos os dados para Janeiro de 2023.
# 2. Agrupamos o faturamento por Loja.
# 3. Damos um merge (junção) com a tabela df_gerentes para puxar a Meta.
# 4. Comparamos se o faturamento >= Meta -> "Sim", senão "Não".
# ====================================================================
df_meta = df_analise[
    (df_analise['Data'].dt.year == 2023) & (df_analise['Data'].dt.month == 1)
]
df_meta = df_meta[['Loja', 'Faturamento']].groupby('Loja', as_index=False).sum()
df_meta = df_meta.merge(df_gerentes, on='Loja', how='left')
df_meta['Bateu a Meta?'] = np.where(
    df_meta['Faturamento'] >= df_meta['Meta_Mensal'], 
    'Sim', 
    'Não'
)

print("\n--- RESULTADO DAS METAS - JANEIRO/2023 ---")
print(df_meta)

# %%
# ====================================================================
# BLOCO 16 - ANÁLISE TEMPORAL E VISUALIZAÇÃO (GRÁFICO)
# ====================================================================
# 1. Criamos uma coluna 'Mes-Ano' (ex: 2023-01).
# 2. Agrupamos o Faturamento total por mês.
# 3. Plotamos um gráfico de linha para ver a evolução.
# ====================================================================
df_analise['Mes-Ano'] = df_analise['Data'].dt.to_period("M")
df_vendas_mes = df_analise[['Mes-Ano', 'Faturamento']].groupby('Mes-Ano').sum()

# O comando abaixo gera o gráfico. 
# Se estiver no VS Code com Python, ele vai abrir uma janela.
# Se estiver no Colab/Jupyter, vai aparecer abaixo da célula.
df_vendas_mes.plot(title="Evolução do Faturamento Mensal", ylabel="Faturamento (R$)")

print("\nGráfico de evolução mensal gerado com sucesso!")

# %%
# ====================================================================
# BLOCO 17 - VISUALIZAÇÃO FINAL DO DATAFRAME TRATADO
# ====================================================================
print("\n--- DATAFRAME FINAL (LIMPO E TRATADO) ---")
print("Primeiras 5 linhas:")
print(df_analise.head())

print("\nÚltimas 5 linhas:")
print(df_analise.tail())

print("\n" + "="*70)
print("✅ PROJETO FINALIZADO COM SUCESSO! TODOS OS BLOCOS FORAM EXECUTADOS.")
print("="*70)
