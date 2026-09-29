# Advance_level
Advanced cricket fielding analysis project focused on collecting and analyzing ball-by-ball fielding data for three players in a T20 match. The project records fielding actions, positions, catches, throws, runs saved/conceded, and calculates player performance scores to identify strengths and improvement areas.

## 🏏 Task 3 – Advanced Level: Cricket Fielding Analysis

This project focuses on collecting and analyzing detailed fielding performance data for three players from a T20 cricket match. The objective is to evaluate individual fielding contributions and understand their impact on the team's defensive performance.

### 📊 Data Collected

The fielding dataset contains the following features:

- Match Number
- Innings
- Team
- Player Name
- Ball Count
- Fielding Position
- Short Description
- Pick Type
- Throw Type
- Runs Saved / Conceded
- Over Count
- Venue

### 📈 Performance Analysis

A Performance Score (PS) is calculated using fielding actions and runs saved:

```text
PS = (CP × WCP) + (GT × WGT) + (C × WC)
     + (DC × WDC) + (ST × WST) + (RO × WRO)
     + (MRO × WMRO) + (DH × WD)

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import os

pd.set_option("display.max_columns", None)
pd.set_option("display.max_rows", 100)

# ============================================================
# 1. LOAD DATASET
# ============================================================

file_path = "C:/Users/EshRoh/OneDrive/Desktop/task list/Python Developer/IPL sample data.xlsx"

df = pd.read_excel(
    file_path,
    sheet_name="Sheet1",
    header=4
)

print("Data loaded successfully!")
print("Shape:", df.shape)

# ============================================================
# 2. CLEAN DATA
# ============================================================

# Remove completely empty columns
df = df.dropna(axis=1, how="all")

# Keep actual ball-by-ball records
ball_data = df[df["Match No."].notna()].copy()

# Reset index
ball_data.reset_index(drop=True, inplace=True)

# Clean column names
ball_data.columns = (
    ball_data.columns
    .astype(str)
    .str.strip()
)

# Remove extra spaces from text values
for col in ball_data.select_dtypes(include="object").columns:
    ball_data[col] = (
        ball_data[col]
        .astype(str)
        .str.strip()
    )

# Convert numeric columns
for col in ["Runs", "BallCount", "Overcount"]:
    if col in ball_data.columns:
        ball_data[col] = pd.to_numeric(
            ball_data[col],
            errors="coerce"
        )

print("\nBall-by-ball records:", len(ball_data))

# ============================================================
# 3. DISPLAY DATA INFORMATION
# ============================================================

print("\nColumn Names:")
print(list(ball_data.columns))

print("\nMissing Values:")
print(ball_data.isnull().sum())

print("\nFirst 5 Records:")
display(ball_data.head())

# ============================================================
# 4. STANDARDIZE PICK AND THROW VALUES
# ============================================================

ball_data["Pick"] = (
    ball_data["Pick"]
    .fillna("")
    .astype(str)
    .str.upper()
    .str.strip()
)

ball_data["Throw"] = (
    ball_data["Throw"]
    .fillna("")
    .astype(str)
    .str.upper()
    .str.strip()
)

print("\nUnique Pick Values:")
print(ball_data["Pick"].unique())

print("\nUnique Throw Values:")
print(ball_data["Throw"].unique())

# ============================================================
# 5. CREATE FIELDING ACTION COLUMNS
# ============================================================

# Pick-related actions
ball_data["Clean Pick"] = (
    ball_data["Pick"] == "Y"
).astype(int)

ball_data["Fumble"] = (
    ball_data["Pick"] == "N"
).astype(int)

ball_data["Catch"] = (
    ball_data["Pick"] == "C"
).astype(int)

ball_data["Dropped Catch"] = (
    ball_data["Pick"] == "DC"
).astype(int)

ball_data["Stumping"] = (
    ball_data["Pick"] == "S"
).astype(int)

# Throw-related actions
ball_data["Good Throw"] = (
    ball_data["Throw"] == "Y"
).astype(int)

ball_data["Bad Throw"] = (
    ball_data["Throw"] == "N"
).astype(int)

ball_data["Direct Hit"] = (
    ball_data["Throw"] == "DH"
).astype(int)

ball_data["Run Out"] = (
    ball_data["Throw"] == "RO"
).astype(int)

ball_data["Missed Run Out"] = (
    ball_data["Throw"] == "MR"
).astype(int)

print("\nFielding action columns created successfully.")

# ============================================================
# 6. DISPLAY FIELDING ACTIONS
# ============================================================

action_columns = [
    "Player Name",
    "Pick",
    "Throw",
    "Clean Pick",
    "Fumble",
    "Catch",
    "Dropped Catch",
    "Stumping",
    "Good Throw",
    "Bad Throw",
    "Direct Hit",
    "Run Out",
    "Missed Run Out",
    "Runs"
]

display(ball_data[action_columns])

# ============================================================
# 7. LIST AVAILABLE PLAYERS
# ============================================================

players = (
    ball_data["Player Name"]
    .dropna()
    .astype(str)
    .str.strip()
    .unique()
)

print("\nPlayers available in dataset:")

for i, player in enumerate(players, start=1):
    print(f"{i}. {player}")

# ============================================================
# 8. SELECT THREE PLAYERS
# ============================================================

selected_players = [
    "Rilee russouw",
    "Phil Salt",
    "Yash Dhull"
]

print("\nSelected Players:")

for player in selected_players:
    print("-", player)

selected_data = ball_data[
    ball_data["Player Name"].isin(selected_players)
].copy()

selected_data.reset_index(drop=True, inplace=True)

print("\nSelected player records:", len(selected_data))

display(selected_data)

# ============================================================
# 9. PERFORMANCE WEIGHTS
# ============================================================

weights = {
    "Clean Pick": 1,
    "Good Throw": 1,
    "Catch": 3,
    "Dropped Catch": -3,
    "Stumping": 3,
    "Run Out": 3,
    "Missed Run Out": -2,
    "Direct Hit": 2
}

print("\nPerformance Weights:")

for metric, weight in weights.items():
    print(f"{metric}: {weight}")

# ============================================================
# 10. CALCULATE PERFORMANCE SCORE
# ============================================================

def calculate_performance_score(group):

    CP = group["Clean Pick"].sum()
    GT = group["Good Throw"].sum()
    C = group["Catch"].sum()
    DC = group["Dropped Catch"].sum()
    ST = group["Stumping"].sum()
    RO = group["Run Out"].sum()
    MRO = group["Missed Run Out"].sum()
    DH = group["Direct Hit"].sum()

    # Runs saved / conceded
    RS = group["Runs"].sum()

    # Performance Score
    PS = (
        CP * weights["Clean Pick"]
        + GT * weights["Good Throw"]
        + C * weights["Catch"]
        + DC * weights["Dropped Catch"]
        + ST * weights["Stumping"]
        + RO * weights["Run Out"]
        + MRO * weights["Missed Run Out"]
        + DH * weights["Direct Hit"]
        + RS
    )

    return pd.Series({
        "Clean Picks (CP)": CP,
        "Good Throws (GT)": GT,
        "Catches (C)": C,
        "Dropped Catches (DC)": DC,
        "Stumpings (ST)": ST,
        "Run Outs (RO)": RO,
        "Missed Run Outs (MRO)": MRO,
        "Direct Hits (DH)": DH,
        "Runs Saved (RS)": RS,
        "Performance Score (PS)": PS
    })

# ============================================================
# 11. PERFORMANCE MATRIX
# ============================================================

performance_matrix = (
    selected_data
    .groupby("Player Name", sort=False)
    .apply(calculate_performance_score)
    .reset_index()
)

performance_matrix = performance_matrix.round(2)

print("\n" + "=" * 70)
print("FINAL FIELDING PERFORMANCE MATRIX")
print("=" * 70)

display(performance_matrix)

# ============================================================
# 12. INDIVIDUAL PLAYER RESULTS
# ============================================================

for player in selected_players:

    print("\n" + "=" * 50)
    print("PLAYER:", player)
    print("=" * 50)

    player_result = performance_matrix[
        performance_matrix["Player Name"] == player
    ]

    if len(player_result) > 0:
        display(player_result)
    else:
        print("No fielding record found for this player.")

# ============================================================
# 13. PERFORMANCE SCORE GRAPH
# ============================================================

plt.figure(figsize=(10, 6))

plt.bar(
    performance_matrix["Player Name"],
    performance_matrix["Performance Score (PS)"]
)

plt.title("Cricket Fielding Performance Score")
plt.xlabel("Player")
plt.ylabel("Performance Score")

plt.xticks(rotation=20)

plt.tight_layout()
plt.show()

# ============================================================
# 14. RUNS SAVED / CONCEDED GRAPH
# ============================================================

plt.figure(figsize=(10, 6))

plt.bar(
    performance_matrix["Player Name"],
    performance_matrix["Runs Saved (RS)"]
)

plt.title("Runs Saved / Conceded by Fielder")
plt.xlabel("Player")
plt.ylabel("Runs Saved (+) / Conceded (-)")

plt.axhline(
    y=0,
    linewidth=1
)

plt.xticks(rotation=20)

plt.tight_layout()
plt.show()

# ============================================================
# 15. FIELDING ACTION COMPARISON
# ============================================================

metrics = [
    "Clean Picks (CP)",
    "Good Throws (GT)",
    "Catches (C)",
    "Dropped Catches (DC)",
    "Stumpings (ST)",
    "Run Outs (RO)",
    "Missed Run Outs (MRO)",
    "Direct Hits (DH)"
]

comparison_data = performance_matrix.set_index(
    "Player Name"
)[metrics]

comparison_data.plot(
    kind="bar",
    figsize=(14, 7)
)

plt.title("Comparison of Fielding Actions")
plt.xlabel("Player")
plt.ylabel("Number of Actions")

plt.xticks(rotation=20)

plt.tight_layout()
plt.show()

# ============================================================
# 16. BALL-BY-BALL ANALYSIS
# ============================================================

for player in selected_players:

    print("\n" + "=" * 70)
    print("BALL-BY-BALL FIELDING DATA:", player)
    print("=" * 70)

    player_balls = selected_data[
        selected_data["Player Name"] == player
    ]

    display(
        player_balls[
            [
                "Overcount",
                "BallCount",
                "Player Name",
                "Position",
                "Pick",
                "Throw",
                "Runs"
            ]
        ]
    )

# ============================================================
# 17. FIELDING EVENT SUMMARY
# ============================================================

event_summary = selected_data.groupby(
    "Player Name"
)[
    [
        "Clean Pick",
        "Fumble",
        "Catch",
        "Dropped Catch",
        "Stumping",
        "Good Throw",
        "Bad Throw",
        "Direct Hit",
        "Run Out",
        "Missed Run Out"
    ]
].sum()

print("\nFIELDING EVENT SUMMARY")
display(event_summary)

# ============================================================
# 18. SAVE RESULTS AS EXCEL
# ============================================================

output_file = "IPL_Fielding_Performance_Analysis.xlsx"

with pd.ExcelWriter(
    output_file,
    engine="openpyxl"
) as writer:

    ball_data.to_excel(
        writer,
        sheet_name="Ball By Ball Data",
        index=False
    )

    selected_data.to_excel(
        writer,
        sheet_name="Selected Players",
        index=False
    )

    performance_matrix.to_excel(
        writer,
        sheet_name="Performance Matrix",
        index=False
    )

    event_summary.to_excel(
        writer,
        sheet_name="Event Summary"
    )

print("\nExcel report created successfully!")
print("File:", output_file)

# ============================================================
# 19. SAVE PERFORMANCE MATRIX AS CSV
# ============================================================

csv_file = "IPL_Fielding_Performance_Matrix.csv"

performance_matrix.to_csv(
    csv_file,
    index=False
)

print("\nCSV file created successfully!")
print("File:", csv_file)

# ============================================================
# 20. SAVE PROJECT FILES TO DESKTOP
# ============================================================

desktop = os.path.join(
    os.path.expanduser("~"),
    "OneDrive",
    "Desktop"
)

if not os.path.exists(desktop):
    desktop = os.path.join(
        os.path.expanduser("~"),
        "Desktop"
    )

project_folder = os.path.join(
    desktop,
    "Cricket_Fielding_Analysis"
)

os.makedirs(
    project_folder,
    exist_ok=True
)

# Final Excel output
excel_output = os.path.join(
    project_folder,
    "IPL_Fielding_Performance_Analysis.xlsx"
)

with pd.ExcelWriter(
    excel_output,
    engine="openpyxl"
) as writer:

    ball_data.to_excel(
        writer,
        sheet_name="Ball By Ball Data",
        index=False
    )

    selected_data.to_excel(
        writer,
        sheet_name="Selected Players",
        index=False
    )

    performance_matrix.to_excel(
        writer,
        sheet_name="Performance Matrix",
        index=False
    )

    event_summary.to_excel(
        writer,
        sheet_name="Event Summary"
    )

# Final CSV output
csv_output = os.path.join(
    project_folder,
    "IPL_Fielding_Performance_Matrix.csv"
)

performance_matrix.to_csv(
    csv_output,
    index=False
)

print("\n" + "=" * 60)
print("PROJECT SAVED SUCCESSFULLY!")
print("=" * 60)

print("\nProject Folder:")
print(project_folder)

print("\nExcel File:")
print(excel_output)

print("\nCSV File:")
print(csv_output)



