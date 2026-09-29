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
     + (MRO × WMRO) + (DH × WDH) + RS

