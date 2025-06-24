Código para Média Mensal de VMC

\begin{lstlisting}[language=Python, caption={Código para Média Mensal de VMC }, label={lst:codigo-classificacao}]
import pandas as pd
import matplotlib.pyplot as plt
import re

# Carregar e preparar dados
df = pd.read_csv("SBGL_2024_IOWA.csv")
df["valid"] = pd.to_datetime(df["valid"])
df["month"] = df["valid"].dt.month

# Função para identificar condições VFR
def is_vfr(metar):
    if isinstance(metar, str):
        vis_match = re.search(r"\s(\d{4})\s", metar)
        if not vis_match or int(vis_match.group(1)) < 5000:
            return False

        cloud_matches = re.findall(r"(BKN|OVC)(\d{3})", metar)
        if cloud_matches:
            bases = [int(c[1]) * 100 for c in cloud_matches]
            return min(bases) >= 1500
        else:
            return True
    return False

# Aplicar análise VFR
df["vfr"] = df["metar"].apply(is_vfr)

# Calcular média mensal de VMC (%)
vmc_by_month = df.groupby("month")["vfr"].mean() * 100

# Plotar gráfico
plt.figure(figsize=(10, 5))
vmc_by_month.plot(kind="bar", color="mediumseagreen", edgecolor="black")
plt.title("Média Mensal de VMC (%) – SBGL em 2024")
plt.xlabel("Mês")
plt.ylabel("VMC (%)")
plt.ylim(0, 100)
plt.xticks(ticks=range(0, 12), labels=[
    "Jan", "Fev", "Mar", "Abr", "Mai", "Jun",
    "Jul", "Ago", "Set", "Out", "Nov", "Dez"
], rotation=0)
plt.grid(axis="y", linestyle="--", alpha=0.7)
plt.tight_layout()
plt.show()
\end{lstlisting}

Código para Média horária de VMC

\begin{lstlisting}[language=Python, caption={Código para Média horária de VMC  }, label={lst:codigo-classificacao}]
import pandas as pd
import matplotlib.pyplot as plt
import re

# Carregar o arquivo
df = pd.read_csv("SBGL_2024_IOWA.csv")
df["valid"] = pd.to_datetime(df["valid"])

# Função para identificar condições VFR
def is_vfr(metar):
    if isinstance(metar, str):
        vis_match = re.search(r"\s(\d{4})\s", metar)
        if not vis_match or int(vis_match.group(1)) < 5000:
            return False

        cloud_matches = re.findall(r"(BKN|OVC)(\d{3})", metar)
        if cloud_matches:
            bases = [int(c[1]) * 100 for c in cloud_matches]
            min_ceiling = min(bases)
            return min_ceiling >= 1500
        else:
            # Sem BKN ou OVC = céu limpo → VFR
            return True
    return False

# Aplicar análise VFR
df["vfr"] = df["metar"].apply(is_vfr)
df["hour"] = df["valid"].dt.hour
df["month"] = df["valid"].dt.month

# Criar gráfico para cada mês
import matplotlib.pyplot as plt

for month in range(1, 13):
    df_month = df[df["month"] == month]
    vmc_by_hour = df_month.groupby("hour")["vfr"].mean() * 100

    # Plotar gráfico
    plt.figure(figsize=(12, 6))
    vmc_by_hour.plot(kind="bar", color="royalblue", edgecolor="black")
    plt.title(f"VMC por Hora do Dia – Mês {month:02d}/2024 (SBGL)")
    plt.xlabel("Hora do Dia")
    plt.ylabel("VMC (%)")
    plt.xticks(rotation=0)
    plt.ylim(0, 100)
    plt.grid(axis="y", linestyle="--", alpha=0.7)
    plt.tight_layout()
    plt.show()
\end{lstlisting}
