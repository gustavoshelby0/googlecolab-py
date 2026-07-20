# %% [markdown]
# # 🚀 Análise de Dados - Vendas Tech
# 
# **Autor:** [Seu Nome]  
# **Objetivo:** Limpeza, transformação e análise exploratória de vendas de uma loja de tecnologia.  
# **Status:** Código funcional e organizado em blocos para fácil compreensão.

# %% [markdown]
# ## 1. Importação das Bibliotecas

# %%
import pandas as pd
import numpy as np

print("Bibliotecas carregadas.")

# %% [markdown]
# ## 2. Leitura dos Dados (CSV e Excel)

# %%
df_vendas = pd.read_csv('vendas_tech.csv', low_memory=False)
df_gerentes = pd.read_excel('gerentes_lojas.xlsx')

print(f"Vendas: {df_vendas.shape[0]} linhas e {df_vendas.shape[1]} colunas.")
print(f"Gerentes: {df_gerentes.shape[0]} linhas e {df_gerentes.shape[1]} colunas.")

# %% [markdown]
# ## 3. Análise Exploratória Inicial (EDA)

# %%
print("Primeiras 15 linhas:")
print(df_vendas.head(15))

print("\nÚltimas 15 linhas:")
print(df_vendas.tail(15))

print("\nAmostra aleatória de 15 linhas:")
print(df_vendas.sample(15))

print(f"\nDimensões (linhas, colunas): {df_vendas.shape}")

print("\nNomes das colunas:")
print(df_vendas.columns.tolist())

print("\nResumo dos tipos de dados (info):")
print(df_vendas.info())

print("\nEstatísticas descritivas (describe):")
print(df_vendas.describe())

# %% [markdown]
# ## 4. Limpeza e Tratamento dos Dados
# 
# *   Remoção da coluna `Data_Base` (pouco preenchida).
# *   Substituição de valores nulos em `Loja` por "Online".
# *   Conversão da coluna `Data` para tipo datetime.

# %%
df_analise = df_vendas.copy()
df_analise = df_analise.drop(columns=['Data_Base'])
df_analise['Loja'] = df_analise['Loja'].fillna('Online')
df_analise['Data'] = pd.to_datetime(df_analise['Data'], format='%Y-%m-%d')

print("Coluna 'Data_Base' removida, nulos tratados e 'Data' convertida.")

# %% [markdown]
# ## 5. Padronização de Textos (Strings)
# 
# Aplicamos `strip()` (remove espaços) e `title()` (primeira letra maiúscula) para uniformizar os nomes das lojas.

# %%
df_analise['Loja'] = df_analise['Loja'].str.strip().str.title()

print("Lojas padronizadas. Valores únicos:")
print(df_analise['Loja'].unique())

# %% [markdown]
# ## 6. Padronização da Tabela de Gerentes
# 
# Precisamos tratar a coluna `Loja` do DataFrame de gerentes separadamente para fazer a junção (`merge`) corretamente no futuro.

# %%
df_gerentes['Loja'] = df_gerentes['Loja'].str.strip().str.title()
print("Tabela de gerentes padronizada.")
print(df_gerentes.head())

# %% [markdown]
# ## 7. Remoção de Duplicatas
# 
# Mantemos apenas o primeiro registro de cada `ID_Pedido`.

# %%
df_analise = df_analise.drop_duplicates(subset=['ID_Pedido'])
print(f"Registros restantes: {df_analise.shape[0]}")

# %% [markdown]
# ## 8. Criação de Novas Colunas (Feature Engineering)
# 
# *   **Faturamento:** `Qtd * Preco_Unitario`.
# *   **Forma_de_Venda:** "Online" se `Loja == 'Online'`, senão "Presencial".

# %%
df_analise['Faturamento'] = df_analise['Qtd'] * df_analise['Preco_Unitario']
df_analise['Forma_de_Venda'] = np.where(df_analise['Loja'] == 'Online', 'Online', 'Presencial')

print(df_analise[['Qtd', 'Preco_Unitario', 'Faturamento', 'Forma_de_Venda']].head())

# %% [markdown]
# ## 9. Mapeamento de Regiões (Dicionário)
# 
# Criamos um dicionário para mapear cada loja à sua respectiva região geográfica.

# %%
dic_regioes = {
    'Sao Paulo': 'Sudeste', 'Belo Horizonte': 'Sudeste', 'Rio de Janeiro': 'Sudeste',
    'Online': 'Online',
    'Salvador': 'Nordeste', 'Recife': 'Nordeste',
    'Curitiba': 'Sul', 'Porto Alegre': 'Sul'
}

df_analise['Regiao'] = df_analise['Loja'].map(dic_regioes)
print("Coluna 'Regiao' adicionada.")
print(f"Valores nulos na nova coluna: {df_analise['Regiao'].isna().sum()}")

# %% [markdown]
# ## 10. Ordenação e Reindexação
# 
# Ordenamos por `Data` e `Faturamento`, e resetamos os índices para ficarem sequenciais.

# %%
df_analise = df_analise.sort_values(by=['Data', 'Faturamento']).reset_index(drop=True)
print("DataFrame ordenado e reindexado.")

# %% [markdown]
# ## 11. Filtros e Consultas Específicas (`.loc` e `.iloc`)

# %%
# Usando .loc (condicional)
id_pedido = 4
loja = df_analise.loc[df_analise['ID_Pedido'] == id_pedido, 'Loja'].values[0]
produto = df_analise.loc[df_analise['ID_Pedido'] == id_pedido, 'Produto'].values[0]
print(f"ID_Pedido {id_pedido}: Loja = {loja}, Produto = {produto}")

# Usando .iloc (posicional)
print(f"Valor na posição [3, 2]: {df_analise.iloc[3, 2]}")

# %% [markdown]
# ## 12. Exportação de Subconjuntos (CSV)

# %%
# Apenas loja São Paulo
df_vendas_sp = df_analise[df_analise['Loja'] == 'Sao Paulo']
df_vendas_sp.to_csv('Vendas_SP.csv', index=False)

# Apenas vendas de 2024
df_vendas_2024 = df_analise[df_analise['Data'].dt.year == 2024]
print(f"Arquivos exportados. Vendas 2024: {df_vendas_2024.shape[0]} registros.")

# %% [markdown]
# ## 13. Filtros com Duplas Condições
# 
# Exemplo: Produto 'Cabo HDMI' e Região 'Sul'.

# %%
df_hdmi_sul = df_analise[
    (df_analise['Produto'] == 'Cabo HDMI') & (df_analise['Regiao'] == 'Sul')
]
print(f"Vendas de Cabo HDMI na região Sul: {df_hdmi_sul.shape[0]} registros.")

# %% [markdown]
# ## 14. Análises de Ranking (Groupby)
# 
# Rankings de faturamento por loja e produtos mais vendidos no canal online.

# %%
# Ranking 1: Faturamento por Loja
rank_lojas = df_analise[['Loja', 'Faturamento']].groupby('Loja').sum()
rank_lojas = rank_lojas.sort_values(by='Faturamento', ascending=False).reset_index()
rank_lojas['Faturamento'] = rank_lojas['Faturamento'].map('R${:,.2f}'.format)
print("\n--- Ranking de Faturamento por Loja ---")
print(rank_lojas)

# Ranking 2: Produtos mais vendidos no Online (por Qtd)
df_online = df_analise[df_analise['Loja'] == 'Online']
rank_prod_online = df_online[['Produto', 'Qtd']].groupby('Produto').sum()
rank_prod_online = rank_prod_online.rename(columns={'Qtd': 'Vendas Totais'})
rank_prod_online = rank_prod_online.sort_values(by='Vendas Totais', ascending=False)
print("\n--- Ranking de Produtos (Online) ---")
print(rank_prod_online.head())

# %% [markdown]
# ## 15. Análise de Metas dos Gerentes (Janeiro/2023)
# 
# Filtramos Janeiro/2023, agrupamos o faturamento por loja, e comparamos com a meta mensal via `merge`.

# %%
df_meta = df_analise[
    (df_analise['Data'].dt.year == 2023) & (df_analise['Data'].dt.month == 1)
]
df_meta = df_meta[['Loja', 'Faturamento']].groupby('Loja', as_index=False).sum()
df_meta = df_meta.merge(df_gerentes, on='Loja', how='left')
df_meta['Bateu a Meta?'] = np.where(df_meta['Faturamento'] >= df_meta['Meta_Mensal'], 'Sim', 'Não')

print("\n--- Resultado das Metas (Janeiro/2023) ---")
print(df_meta)

# %% [markdown]
# ## 16. Análise Temporal e Visualização (Gráfico)
# 
# Criamos a coluna `Mes-Ano` e plotamos a evolução do faturamento mensal.

# %%
df_analise['Mes-Ano'] = df_analise['Data'].dt.to_period("M")
df_vendas_mes = df_analise[['Mes-Ano', 'Faturamento']].groupby('Mes-Ano').sum()
df_vendas_mes.plot(title="Evolução do Faturamento Mensal", ylabel="Faturamento (R$)")

print("Gráfico gerado com sucesso!")

# %% [markdown]
# ## 17. Resultado Final

# %%
print("\n--- DataFrame Final (Limpo e Tratado) ---")
print("Primeiras 5 linhas:")
print(df_analise.head())
print("\nÚltimas 5 linhas:")
print(df_analise.tail())

print("\n" + "="*50)
print("✅ Projeto finalizado com sucesso!")
print("="*50)
