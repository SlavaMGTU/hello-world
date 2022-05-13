
import dash
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output, State
from dash.exceptions import PreventUpdate
import plotly.graph_objs as go
from plotly.subplots import make_subplots
import pandas as pd
from glob import glob

# Служебные переменные, можно менять настройки перед запуском

ver = '0.1.3'
date = '01/2020'
external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']
graph_mode = 'lines+markers' # 'markers', 'lines', 'lines+markers'
dir_path = 'data/output/'

# Считывание файлов

file_paths = glob(dir_path + '*.csv')
file_names = [file_path[len(dir_path):] for file_path in file_paths]
files = {}

for file_name, file_path in zip(file_names, file_paths):
    files.update({file_name: pd.read_csv(file_path, index_col=0)})

all_cols = list(files[file_names[0]].columns)
for i in range(1, len(file_names)):
    all_cols += list(files[file_names[i]].columns)

unique_cols = list(set(all_cols))

# Функции для построения графиков


def make_sub(dfs, df_keys, params):
    traces = []
    for key, df in zip(df_keys, dfs):
        for par in params:
            if par in df.columns:
                traces.append(go.Scattergl(x=df.index, y=df[par], name=key + ', ' + par, mode=graph_mode,
                                           marker=dict(size=3)))
    return traces


def plot_ex(list_dfs, list_df_keys, list_params, height):

    fig = make_subplots(rows=len(list_dfs), shared_xaxes=True, vertical_spacing=0.02)

    for j in range(1, len(list_dfs)+1):
        traces = make_sub(list_dfs[j-1], list_df_keys[j-1], list_params[j-1])
        for trace in traces:
            fig.append_trace(trace, row=j, col=1)

    fig.update_layout(height=height)
    #fig.update_yaxes(dtick=1)
    return fig

# Код приложения


app = dash.Dash(__name__, external_stylesheets=external_stylesheets)

