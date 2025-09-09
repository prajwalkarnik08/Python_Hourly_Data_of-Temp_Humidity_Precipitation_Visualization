```python
pip install plotly --timeout=100 --index-url=https://pypi.org/simple
```

    Requirement already satisfied: plotly in c:\users\dhart\anaconda3\lib\site-packages (6.0.0)
    Requirement already satisfied: packaging in c:\users\dhart\anaconda3\lib\site-packages (from plotly) (20.9)
    Requirement already satisfied: narwhals>=1.15.1 in c:\users\dhart\anaconda3\lib\site-packages (from plotly) (1.29.0)
    Requirement already satisfied: pyparsing>=2.0.2 in c:\users\dhart\anaconda3\lib\site-packages (from packaging->plotly) (2.4.7)
    Note: you may need to restart the kernel to use updated packages.
    


```python
import pandas as pd
df = pd.read_csv("POWER_Point_Hourly_20240101_20241231_021d13N_078d85E_LST.csv")
```


```python
# Import necessary visualization library
import plotly.express as px

# Convert "Date" and "Time" to a proper datetime format
df["Datetime"] = pd.to_datetime(df["Date"] + " " + df["Time"], format="%d-%m-%Y %H:%M")

# Set datetime as index
df.set_index("Datetime", inplace=True)

# Create an interactive line plot
fig = px.line(df, x=df.index, y=["T2M", "RH2M", "PRECTOTCORR"],
              labels={"value": "Measurement", "variable": "Weather Parameter"},
              title="Hourly Weather Parameters Over Time")

# Show plot
import plotly.io as pio
pio.renderers.default = "notebook"  # or use "browser" to open in a new tab

pio.renderers.default = "browser"
fig.show(renderer="browser")

fig.show()
```


```python
import pandas as pd
import plotly.graph_objects as go
import plotly.io as pio

# Load data
file_path = "POWER_Point_Hourly_20240101_20241231_021d13N_078d85E_LST.csv"
df = pd.read_csv(file_path)

# Convert Date and Time to Datetime
df["Datetime"] = pd.to_datetime(df["Date"] + " " + df["Time"], errors="coerce", format="%d-%m-%Y %H:%M")

# Remove NaT values if any
df = df.dropna(subset=["Datetime"])

# Set datetime as index
df.set_index("Datetime", inplace=True)
df = df.sort_index()

# Reduce data size by keeping every 12th row
df_sampled = df.iloc[::12]

# Set total animation duration (e.g., 30 seconds)
total_duration = 30_000  # 30,000 milliseconds (30 seconds)
num_frames = len(df_sampled)
frame_duration = total_duration // max(num_frames, 1)  # Avoid division by zero

# Create figure with initial visible traces
fig = go.Figure()

# Initial traces showing data from the beginning
fig.add_trace(go.Scatter(x=df_sampled.index[:1], y=df_sampled["T2M"][:1], mode="lines", name="Temperature (T2M)"))
fig.add_trace(go.Scatter(x=df_sampled.index[:1], y=df_sampled["RH2M"][:1], mode="lines", name="Humidity (RH2M)"))
fig.add_trace(go.Scatter(x=df_sampled.index[:1], y=df_sampled["PRECTOTCORR"][:1], mode="lines", name="Precipitation (PRECTOTCORR)"))

# Create animation frames (building up the graph over time)
frames = [
    go.Frame(
        data=[
            go.Scatter(x=df_sampled.index[:i], y=df_sampled["T2M"][:i], mode="lines", name="Temperature"),
            go.Scatter(x=df_sampled.index[:i], y=df_sampled["RH2M"][:i], mode="lines", name="Humidity"),
            go.Scatter(x=df_sampled.index[:i], y=df_sampled["PRECTOTCORR"][:i], mode="lines", name="Precipitation"),

            # Live data labels (latest point visible)
            go.Scatter(x=[df_sampled.index[i-1]], y=[df_sampled["T2M"].iloc[i-1]], 
                       mode="markers+text", text=[f"{df_sampled['T2M'].iloc[i-1]:.1f}"], 
                       textposition="top right", marker=dict(color="red", size=8), name="T2M (Live)"),
            go.Scatter(x=[df_sampled.index[i-1]], y=[df_sampled["RH2M"].iloc[i-1]], 
                       mode="markers+text", text=[f"{df_sampled['RH2M'].iloc[i-1]:.1f}"], 
                       textposition="top right", marker=dict(color="blue", size=8), name="RH2M (Live)"),
            go.Scatter(x=[df_sampled.index[i-1]], y=[df_sampled["PRECTOTCORR"].iloc[i-1]], 
                       mode="markers+text", text=[f"{df_sampled['PRECTOTCORR'].iloc[i-1]:.1f}"], 
                       textposition="top right", marker=dict(color="green", size=8), name="Precip (Live)")
        ],
        name=str(i)
    )
    for i in range(1, num_frames)
]

# Assign frames
fig.frames = frames  

# Add Play/Pause buttons and Slider for control
fig.update_layout(
    title="Hourly Weather Data Over Time",
    xaxis_title="Time",
    yaxis_title="Measurement",
    updatemenus=[{
        "buttons": [
            {
                "args": [None, {"frame": {"duration": frame_duration, "redraw": True}, "fromcurrent": True}],
                "label": "▶ Play",
                "method": "animate"
            },
            {
                "args": [[None], {"frame": {"duration": 0, "redraw": False}, "mode": "immediate"}],
                "label": "❚❚ Pause",
                "method": "animate"
            }
        ],
        "direction": "left",
        "pad": {"r": 10, "t": 10},
        "showactive": True,
        "type": "buttons",
        "x": 0.115,
        "xanchor": "right",
        "y": 1.13,
        "yanchor": "top"
    }],
    sliders=[{
        "steps": [
            {
                "args": [[f.name], {"frame": {"duration": 0, "redraw": True}, "mode": "immediate"}],
                "label": df_sampled.index[i].strftime("%Y-%m-%d %H:%M"),
                "method": "animate"
            }
            for i, f in enumerate(frames)
        ],
        "active": 0,
        "x": 0.1,
        "y": -0.2,
        "xanchor": "left",
        "yanchor": "top"
    }]
)

# Save as HTML for manual viewing
output_file = "weather_animation_live.html"
fig.write_html(output_file)
print(f"✅ Animation saved as: {output_file}")

```

    ✅ Animation saved as: weather_animation_live.html
    


```python
import pandas as pd
import plotly.graph_objects as go

# Load data
file_path = "POWER_Point_Hourly_20240101_20241231_021d13N_078d85E_LST.csv"
df = pd.read_csv(file_path)

# Convert Date and Time to Datetime
df["Datetime"] = pd.to_datetime(df["Date"] + " " + df["Time"], errors="coerce", format="%d-%m-%Y %H:%M")

# Remove NaT values if any
df = df.dropna(subset=["Datetime"])

# Set datetime as index
df.set_index("Datetime", inplace=True)
df = df.sort_index()

# Reduce data size by keeping every 12th row
df_sampled = df.iloc[::12]

# Set total animation duration (30 seconds)
total_duration = 30_000  # 30,000 milliseconds (30 seconds)
num_frames = len(df_sampled)
frame_duration = total_duration // max(num_frames, 1)  # Avoid division by zero

# Create figure
fig = go.Figure()

# Initial traces (empty for animation start)
fig.add_trace(go.Scatter(x=[], y=[], mode="lines", name="Temperature (T2M)", line=dict(color="red")))
fig.add_trace(go.Scatter(x=[], y=[], mode="lines", name="Humidity (RH2M)", line=dict(color="blue")))
fig.add_trace(go.Scatter(x=[], y=[], mode="lines", name="Precipitation (PRECTOTCORR)", line=dict(color="green")))

# Create frames
frames = []
for i in range(1, num_frames):
    frames.append(go.Frame(
        data=[
            # Full lines up to the current frame
            go.Scatter(x=df_sampled.index[:i], y=df_sampled["T2M"][:i], mode="lines",
                       line=dict(color="red"), name="Temperature"),
            go.Scatter(x=df_sampled.index[:i], y=df_sampled["RH2M"][:i], mode="lines",
                       line=dict(color="blue"), name="Humidity"),
            go.Scatter(x=df_sampled.index[:i], y=df_sampled["PRECTOTCORR"][:i], mode="lines",
                       line=dict(color="green"), name="Precipitation"),

            # Latest value markers
            go.Scatter(x=[df_sampled.index[i-1]], y=[df_sampled["T2M"].iloc[i-1]], 
                       mode="markers", marker=dict(color="red", size=8), name="Latest T2M"),
            go.Scatter(x=[df_sampled.index[i-1]], y=[df_sampled["RH2M"].iloc[i-1]], 
                       mode="markers", marker=dict(color="blue", size=8), name="Latest RH2M"),
            go.Scatter(x=[df_sampled.index[i-1]], y=[df_sampled["PRECTOTCORR"].iloc[i-1]], 
                       mode="markers", marker=dict(color="green", size=8), name="Latest Precip")
        ],
        name=str(i),
        layout=go.Layout(
            annotations=[
                # Temperature Label with Background Box
                dict(
                    x=df_sampled.index[i-1], 
                    y=df_sampled["T2M"].iloc[i-1],
                    text=f"{df_sampled['T2M'].iloc[i-1]:.1f}°C",
                    showarrow=True,
                    arrowhead=2,
                    font=dict(color="white"),
                    align="center",
                    bgcolor="red",
                    opacity=0.8
                ),
                # Humidity Label with Background Box
                dict(
                    x=df_sampled.index[i-1], 
                    y=df_sampled["RH2M"].iloc[i-1],
                    text=f"{df_sampled['RH2M'].iloc[i-1]:.1f}%",
                    showarrow=True,
                    arrowhead=2,
                    font=dict(color="white"),
                    align="center",
                    bgcolor="blue",
                    opacity=0.8
                ),
                # Precipitation Label with Background Box
                dict(
                    x=df_sampled.index[i-1], 
                    y=df_sampled["PRECTOTCORR"].iloc[i-1],
                    text=f"{df_sampled['PRECTOTCORR'].iloc[i-1]:.2f} mm",
                    showarrow=True,
                    arrowhead=2,
                    font=dict(color="white"),
                    align="center",
                    bgcolor="green",
                    opacity=0.8
                )
            ]
        )
    ))

# Assign frames
fig.frames = frames  

# Layout updates (Added Grid)
fig.update_layout(
    title="Hourly Weather Data Over Time",
    xaxis_title="Time",
    yaxis_title="Measurement",
    xaxis=dict(
        showgrid=True, gridwidth=1, gridcolor="lightgray", zeroline=False  # Grid on X-axis
    ),
    yaxis=dict(
        showgrid=True, gridwidth=1, gridcolor="lightgray", zeroline=False  # Grid on Y-axis
    ),
    plot_bgcolor="white",
    updatemenus=[{
        "buttons": [
            {"args": [None, {"frame": {"duration": frame_duration, "redraw": True}, "fromcurrent": True}],
             "label": "▶ Play", "method": "animate"},
            {"args": [[None], {"frame": {"duration": 0, "redraw": False}, "mode": "immediate"}],
             "label": "❚❚ Pause", "method": "animate"}
        ],
        "direction": "left",
        "pad": {"r": 10, "t": 10},
        "showactive": True,
        "type": "buttons",
        "x": 0.115, "xanchor": "right",
        "y": 1.115, "yanchor": "top"
    }],
    sliders=[{
        "steps": [
            {"args": [[f.name], {"frame": {"duration": 0, "redraw": True}, "mode": "immediate"}],
             "label": df_sampled.index[i].strftime("%Y-%m-%d %H:%M"),
             "method": "animate"}
            for i, f in enumerate(frames)
        ],
        "active": 0,
        "x": 0.1,
        "y": -0.2,
        "xanchor": "left",
        "yanchor": "top"
    }]
)

# Save as HTML for manual viewing
output_file = "weather_animation_with_grid.html"
fig.write_html(output_file)
print(f"✅ Animation saved as: {output_file}")

```

    ✅ Animation saved as: weather_animation_with_grid.html
    


```python

```
