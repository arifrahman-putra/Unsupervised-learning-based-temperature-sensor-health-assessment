# Unsupervised-learning-based-temperature-sensor-health-assessment
An Isolation Forest outlier detection implementation for temperature sensor and transmission health scoring 


## 🔍 Overview

![System Flowchart](flowchart.png)

The system operates by:

1. **Connecting to a local PostgreSQL database table** to retrieve raw temperature sensor recording.
2. **Feature extraction** based on how the temperature data fluctuates every hour:
   - num_data: the number of raw temperature data recorded each hour
   - sampling_rate: the number of raw temperature data recorded each hour
   - range: recorded temperature  range (max-min) in each hour
   - standard_deviation of temperature recorded data wihin an hour
   - covar: coefficient of variation (standard_deviationb/mean)


   non feature variables:
   - start_time: datetime (hour) object that shows the starting hour of each recorded temperature data time-window
   - availability_status: flag variable that shows whether there are at least 3 recorded temperature data in each hour; if the availability_status = 0 (there are less than 3 recorded temperature data in an hour), then the extracted features for that hour are considered invalid and filled with 0s except for the hour_index
   
   




## 📦 Files in This Repo

| File                                                   | Description                                     |
|--------------------------------------------------------|-------------------------------------------------|
| `Temperature Sensor Health Assesment_Postgres.ipynb`   | Main script to run the health assesment         |
| `requirements.txt`                                     | Python package dependencies                     |
| `README.md`                                            | This documentation                              |
| `flowchart.png`                                        | Flowchart image of this project                 |

> **Note:** Station inventory XML files are **not included** in this repository due to data-sharing restrictions.  
> You must provide your own inventory files at:  
> `/path/to/inventory/{StationID}.xml`


## 📊 Database Tables

The system uses PostgreSQL database with the following table structures:


### Equipments Table

Stores information about seismic monitoring equipment.

```sql
CREATE TABLE Equipments (
    equipmentid TEXT PRIMARY KEY,
    stationid TEXT,
    name TEXT,
    status TEXT,
    channel TEXT
);
```

**Example Data:**

| equipmentid | stationid | name          | status | channel |
|-------------|-----------|---------------|--------|---------|
| 01          | ABC       | Seismometer   | Active | S       |
| 02          | DEF       | Accelerometer | Active | K       |
| 03          | GHI       | Seismometer   | Dead   | B       |
| 04          | IJK       | Seismometer   | Active | S       |


### Health_Status Table

Records diagnostic results for each channel of equipment.

```sql
CREATE TABLE Health_Status (
    TimeStamp TEXT PRIMARY KEY,
    TimeIndex TEXT,
    StationID TEXT,
    EquipmentID TEXT,
    Channel TEXT,
    HealthStatus INTEGER,
    Diagnosis TEXT,
    Description TEXT
);
```

**Example Data:**

| TimeStamp           | TimeIndex           | StationID | EquipmentID | Channel | HealthStatus | Diagnosis | Description           |
|---------------------|---------------------|-----------|-------------|---------|--------------|-----------|-----------------------|
| 2025-04-10T12:35:00 | 2025-04-10T12:00:00 | ABC       | 01          | SHE     | 1            | Normal    | Earthquake occurrence |
| 2025-04-10T12:35:06 | 2025-04-10T12:00:00 | ABC       | 01          | SHN     | 1            | Normal    | Earthquake occurrence |
| 2025-04-10T12:35:12 | 2025-04-10T12:00:00 | ABC       | 01          | SHZ     | 1            | Normal    | Earthquake occurrence |
| 2025-04-10T12:35:36 | 2025-04-10T12:00:00 | IJK       | 04          | SHE     | 0            | LM        | Diagnosis result      |

> Note: LM = "Locked Mass" seismometer fault type


### Daily_Report Table

Summarizes equipment health on a daily basis.

```sql
CREATE TABLE Daily_Report (
    TimeStamp TEXT PRIMARY KEY,
    DayIndex TEXT,
    StationID TEXT,
    EquipmentID TEXT,
    HealthScore FLOAT
);
```

**Example Data:**

| TimeStamp           | DayIndex   | StationID | EquipmentID | HealthScore |
|---------------------|------------|-----------|-------------|-------------|
| 2025-04-10T23:35:00 | 2025-04-10 | ABC       | 01          | 75.00       |
| 2025-04-10T23:35:06 | 2025-04-10 | GHI       | 03          | 50.00       |
| 2025-04-10T23:35:12 | 2025-04-10 | IJK       | 04          | 33.33       |


## 🚀 Usage

1. Ensure your PostgreSQL database is running and the `Health_Status` and `Daily_Report` tables are set up.  
2. Make sure your station XML inventory files are available in `/path/to/inventory/{StationID}.xml`.  
3. Run the diagnosis script:

```bash
python seismometer_diagnosis.py
```



