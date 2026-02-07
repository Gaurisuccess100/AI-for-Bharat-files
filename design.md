# Design Document: AI-Powered Wind Energy Smart Study Lamp

## Overview

The AI-Powered Wind Energy Smart Study Lamp is a distributed embedded system consisting of three main components:

1. **Hardware Layer**: Wind turbine, battery management, and LED control circuitry
2. **Embedded Controller**: Arduino Nano firmware for real-time monitoring and control
3. **AI & Dashboard Layer**: Python-based intelligence and Streamlit visualization

The system operates in a continuous loop: wind energy is harvested and stored, the Arduino monitors battery status and controls LED brightness via PWM, AI logic predicts optimal brightness levels, and a dashboard provides real-time feedback to users. All components work together to maximize study time while efficiently managing limited renewable energy resources.

### Design Philosophy

- **Offline-first**: All critical functions operate without internet connectivity
- **Energy-aware**: Every decision prioritizes maximizing study time from available energy
- **Simple and robust**: Minimal complexity, maximum reliability for rural deployment
- **Data-driven**: Historical usage patterns inform future predictions
- **User-centric**: Clear feedback helps students optimize their study schedules

## Architecture

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     Physical Layer                           │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │  Wind    │───▶│ TP4056   │───▶│ Li-ion   │              │
│  │ Turbine  │    │ Charging │    │ Battery  │              │
│  └──────────┘    └──────────┘    └────┬─────┘              │
│                                        │                     │
│                                        ▼                     │
│                                  ┌──────────┐               │
│                                  │  Boost   │               │
│                                  │Converter │               │
│                                  │  (5V)    │               │
│                                  └────┬─────┘               │
└───────────────────────────────────────┼─────────────────────┘
                                        │
┌───────────────────────────────────────┼─────────────────────┐
│                  Control Layer        │                      │
│                                       ▼                      │
│                              ┌─────────────────┐            │
│                              │  Arduino Nano   │            │
│                              │                 │            │
│  ┌──────────────┐           │ • Battery Read  │            │
│  │ Voltage      │◀──────────│ • PWM Control   │            │
│  │ Divider      │           │ • Serial Comm   │            │
│  └──────────────┘           │ • Timing Logic  │            │
│                              └────┬───────┬────┘            │
│                                   │       │                 │
│                              ┌────▼───┐   │                 │
│                              │  PWM   │   │                 │
│                              │  LED   │   │                 │
│                              └────────┘   │                 │
└────────────────────────────────────────┼──┼─────────────────┘
                                         │  │
                                    Serial  │
                                         │  │
┌────────────────────────────────────────┼──┼─────────────────┐
│              Software Layer            │  │                  │
│                                        ▼  │                  │
│  ┌─────────────────────────────────────┐ │                  │
│  │      Python AI Module               │ │                  │
│  │                                     │ │                  │
│  │  • Serial Reader                   │ │                  │
│  │  • Battery Level Analyzer          │ │                  │
│  │  • Brightness Decision Logic       │ │                  │
│  │  • Study Time Predictor            │ │                  │
│  │  • Data Logger (CSV)               │ │                  │
│  └──────────────┬──────────────────────┘ │                  │
│                 │                         │                  │
│                 ▼                         │                  │
│  ┌─────────────────────────────────────┐ │                  │
│  │      Streamlit Dashboard            │ │                  │
│  │                                     │ │                  │
│  │  • Real-time Visualization         │ │                  │
│  │  • Battery Status Display          │ │                  │
│  │  • Study Hours Chart               │ │                  │
│  │  • Feedback Messages               │ │                  │
│  │  • Historical Data Plots           │ │                  │
│  └─────────────────────────────────────┘ │                  │
└──────────────────────────────────────────┴──────────────────┘
```

### Component Interaction Flow

1. **Energy Harvesting Flow**: Wind → Turbine → Charging Module → Battery
2. **Monitoring Flow**: Battery → Voltage Divider → Arduino ADC → Serial → Python
3. **Control Flow**: Python AI → Serial → Arduino → PWM → LED
4. **Data Flow**: Arduino → Serial → Python → CSV File → Dashboard
5. **Feedback Flow**: Dashboard → User → Behavioral Adjustment

### Communication Protocol

**Arduino → Python (Serial)**
- Format: `BATTERY:<percentage>,BRIGHTNESS:<mode>,DURATION:<minutes>,TIMESTAMP:<unix_time>`
- Example: `BATTERY:75,BRIGHTNESS:HIGH,DURATION:45,TIMESTAMP:1704067200`
- Frequency: Every 10 seconds
- Baud Rate: 9600

**Python → Arduino (Serial)**
- Format: `CMD:<command>,VALUE:<value>`
- Example: `CMD:SET_BRIGHTNESS,VALUE:MEDIUM`
- Used for: Manual overrides, testing, calibration

## Components and Interfaces

### 1. Hardware Components

#### Wind Turbine (DC Motor Based)
- **Purpose**: Convert wind kinetic energy to electrical energy
- **Output**: Variable DC voltage (3-12V depending on wind speed)
- **Interface**: Direct connection to TP4056 charging module input

#### TP4056 Battery Charging Module
- **Purpose**: Safely charge Li-ion battery from turbine output
- **Features**: 
  - Overcharge protection
  - Reverse current protection
  - Charging status LEDs
- **Input**: Variable DC from turbine
- **Output**: Regulated charging current to battery

#### Li-ion 18650 Battery
- **Capacity**: 2000-3000 mAh (typical)
- **Voltage Range**: 3.0V (empty) to 4.2V (full)
- **Purpose**: Energy storage for study sessions
- **Protection**: Built-in or external BMS for safety

#### Boost Converter (5V)
- **Purpose**: Step up battery voltage to stable 5V for Arduino and LED
- **Input**: 3.0-4.2V from battery
- **Output**: 5V regulated
- **Efficiency**: Target 85%+

#### Voltage Divider Circuit
- **Purpose**: Scale battery voltage to Arduino ADC range (0-5V)
- **Design**: R1=10kΩ, R2=10kΩ (divides voltage by 2)
- **Output**: Analog voltage proportional to battery level
- **Connection**: Battery → R1 → ADC Pin (A0) → R2 → GND

#### PWM-Controlled LED
- **Type**: High-brightness white LED (1-3W)
- **Control**: PWM signal from Arduino pin D9
- **Driver**: MOSFET or transistor for current control
- **Brightness Levels**: 30%, 60%, 100% duty cycle

### 2. Arduino Nano Firmware

#### Core Responsibilities
- Read battery voltage via analog pin A0
- Convert ADC reading to battery percentage
- Control LED brightness via PWM on pin D9
- Track study session timing
- Communicate with Python via serial

#### Key Functions

**`setup()`**
- Initialize serial communication (9600 baud)
- Configure pin modes (A0 as input, D9 as PWM output)
- Set initial LED state (off)
- Initialize timing variables

**`loop()`**
- Read battery voltage every 5 seconds
- Calculate battery percentage
- Receive brightness commands from serial
- Update PWM duty cycle
- Track session duration
- Send status updates via serial every 10 seconds

**`readBatteryVoltage()`**
- Read analog value from pin A0 (0-1023)
- Convert to actual voltage: `voltage = (analogValue / 1023.0) * 5.0 * 2.0`
- Apply calibration offset if needed
- Return voltage as float

**`calculateBatteryPercentage(float voltage)`**
- Map voltage range (3.0V - 4.2V) to percentage (0% - 100%)
- Use linear interpolation: `percentage = ((voltage - 3.0) / 1.2) * 100`
- Clamp to 0-100 range
- Return integer percentage

**`setBrightness(String mode)`**
- Parse mode string ("HIGH", "MEDIUM", "LOW", "OFF")
- Set PWM duty cycle:
  - HIGH: `analogWrite(LED_PIN, 255)` (100%)
  - MEDIUM: `analogWrite(LED_PIN, 153)` (60%)
  - LOW: `analogWrite(LED_PIN, 77)` (30%)
  - OFF: `analogWrite(LED_PIN, 0)` (0%)

**`sendSerialData()`**
- Format data string with current status
- Send via `Serial.println()`
- Include battery percentage, brightness mode, session duration, timestamp

**`receiveSerialCommands()`**
- Check if serial data available
- Parse incoming command string
- Execute command (e.g., set brightness, reset timer)
- Send acknowledgment

#### State Management
- **Current Brightness Mode**: Enum (OFF, LOW, MEDIUM, HIGH)
- **Session Start Time**: Milliseconds since boot
- **Session Active**: Boolean flag
- **Last Battery Reading**: Cached percentage value
- **Last Serial Send**: Timestamp for 10-second interval

### 3. Python AI Module

#### Architecture

```python
# Module Structure
ai_module/
├── serial_reader.py      # Arduino communication
├── battery_analyzer.py   # Battery level analysis
├── brightness_optimizer.py # Decision logic
├── study_predictor.py    # ML-based predictions
├── data_logger.py        # CSV storage
└── main.py              # Orchestration
```

#### Core Classes

**`SerialReader`**
```python
class SerialReader:
    def __init__(self, port='/dev/ttyUSB0', baud_rate=9600):
        self.serial_conn = serial.Serial(port, baud_rate)
    
    def read_data(self) -> dict:
        """Read and parse data from Arduino"""
        # Returns: {'battery': int, 'brightness': str, 
        #           'duration': int, 'timestamp': int}
    
    def send_command(self, command: str, value: str):
        """Send command to Arduino"""
```

**`BatteryAnalyzer`**
```python
class BatteryAnalyzer:
    def __init__(self):
        self.history = []  # Recent battery readings
    
    def analyze_level(self, percentage: int) -> dict:
        """Analyze battery level and return status"""
        # Returns: {'level': str, 'critical': bool, 
        #           'trend': str, 'rate': float}
    
    def predict_depletion_time(self, percentage: int, 
                               brightness: str) -> float:
        """Predict minutes until battery depleted"""
```

**`BrightnessOptimizer`**
```python
class BrightnessOptimizer:
    def decide_brightness(self, battery_percentage: int, 
                         time_of_day: int,
                         historical_data: pd.DataFrame) -> str:
        """
        Decide optimal brightness mode
        
        Rules:
        - Battery > 60%: HIGH
        - Battery 30-60%: MEDIUM
        - Battery < 30%: LOW
        - Battery < 5%: OFF
        
        Can be overridden by ML predictions if available
        """
        if battery_percentage < 5:
            return "OFF"
        elif battery_percentage < 30:
            return "LOW"
        elif battery_percentage < 60:
            return "MEDIUM"
        else:
            return "HIGH"
```

**`StudyPredictor`**
```python
class StudyPredictor:
    def __init__(self):
        self.model = None  # Scikit-learn model
        self.use_ml = False
    
    def train_model(self, historical_data: pd.DataFrame):
        """Train prediction model on historical data"""
        # Use RandomForestRegressor or LinearRegression
        # Features: battery_start, brightness_mode, time_of_day
        # Target: duration_minutes
    
    def predict_study_hours(self, battery_percentage: int,
                           brightness_mode: str) -> float:
        """Predict available study hours"""
        if self.use_ml and self.model:
            # Use ML model
            return self._ml_predict(battery_percentage, brightness_mode)
        else:
            # Use rule-based estimation
            return self._rule_based_predict(battery_percentage, brightness_mode)
    
    def _rule_based_predict(self, battery: int, mode: str) -> float:
        """
        Rule-based prediction using power consumption estimates
        
        Assumptions:
        - HIGH mode: 2W consumption
        - MEDIUM mode: 1.2W consumption  
        - LOW mode: 0.6W consumption
        - Battery capacity: 2500mAh at 3.7V = 9.25Wh
        """
        capacity_wh = 9.25 * (battery / 100.0)
        
        power_consumption = {
            'HIGH': 2.0,
            'MEDIUM': 1.2,
            'LOW': 0.6
        }
        
        hours = capacity_wh / power_consumption.get(mode, 1.2)
        return round(hours, 2)
```

**`DataLogger`**
```python
class DataLogger:
    def __init__(self, csv_path='study_sessions.csv'):
        self.csv_path = csv_path
        self._ensure_file_exists()
    
    def _ensure_file_exists(self):
        """Create CSV with headers if it doesn't exist"""
        if not os.path.exists(self.csv_path):
            df = pd.DataFrame(columns=[
                'timestamp', 'battery_start', 'battery_end',
                'brightness_mode', 'duration_minutes'
            ])
            df.to_csv(self.csv_path, index=False)
    
    def log_session(self, session_data: dict):
        """Append session data to CSV"""
        df = pd.DataFrame([session_data])
        df.to_csv(self.csv_path, mode='a', header=False, index=False)
    
    def load_historical_data(self) -> pd.DataFrame:
        """Load all historical session data"""
        return pd.read_csv(self.csv_path)
```

#### Main Orchestration Loop

```python
def main():
    # Initialize components
    serial_reader = SerialReader()
    battery_analyzer = BatteryAnalyzer()
    brightness_optimizer = BrightnessOptimizer()
    study_predictor = StudyPredictor()
    data_logger = DataLogger()
    
    # Load historical data and train model if enough data
    historical_data = data_logger.load_historical_data()
    if len(historical_data) >= 10:
        study_predictor.train_model(historical_data)
    
    session_start_battery = None
    session_active = False
    
    while True:
        # Read data from Arduino
        data = serial_reader.read_data()
        battery = data['battery']
        current_brightness = data['brightness']
        
        # Analyze battery status
        battery_status = battery_analyzer.analyze_level(battery)
        
        # Decide optimal brightness
        optimal_brightness = brightness_optimizer.decide_brightness(
            battery, 
            datetime.now().hour,
            historical_data
        )
        
        # Send brightness command if different from current
        if optimal_brightness != current_brightness:
            serial_reader.send_command('SET_BRIGHTNESS', optimal_brightness)
        
        # Track session
        if current_brightness != 'OFF' and not session_active:
            # Session started
            session_start_battery = battery
            session_active = True
        elif current_brightness == 'OFF' and session_active:
            # Session ended - log it
            session_data = {
                'timestamp': datetime.now().isoformat(),
                'battery_start': session_start_battery,
                'battery_end': battery,
                'brightness_mode': data['brightness'],
                'duration_minutes': data['duration']
            }
            data_logger.log_session(session_data)
            session_active = False
        
        # Predict study hours
        predicted_hours = study_predictor.predict_study_hours(
            battery, optimal_brightness
        )
        
        # Store predictions for dashboard
        # (shared via file or in-memory queue)
        
        time.sleep(10)  # Wait 10 seconds before next iteration
```

### 4. Streamlit Dashboard

#### Dashboard Layout

```
┌─────────────────────────────────────────────────────────┐
│  AI-Powered Wind Energy Smart Study Lamp Dashboard     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │  Battery    │  │ Brightness  │  │ Study Hours │   │
│  │    75%      │  │    HIGH     │  │   2.5 hrs   │   │
│  │  ████████░░ │  │     💡      │  │   today     │   │
│  └─────────────┘  └─────────────┘  └─────────────┘   │
│                                                         │
│  ┌───────────────────────────────────────────────────┐ │
│  │  Feedback Message:                                │ │
│  │  ✓ Good battery level - Full brightness available│ │
│  │  📊 Predicted study time: 3.2 hours remaining    │ │
│  └───────────────────────────────────────────────────┘ │
│                                                         │
│  ┌───────────────────────────────────────────────────┐ │
│  │  Battery Level Over Time (Last 24 Hours)         │ │
│  │                                                   │ │
│  │  100% ┤                    ╭─────╮               │ │
│  │   75% ┤          ╭─────────╯     ╰───╮           │ │
│  │   50% ┤    ╭─────╯                   ╰───╮       │ │
│  │   25% ┤────╯                             ╰───    │ │
│  │    0% └─────────────────────────────────────────  │ │
│  │       0h    6h    12h   18h   24h                │ │
│  └───────────────────────────────────────────────────┘ │
│                                                         │
│  ┌───────────────────────────────────────────────────┐ │
│  │  Study Sessions Today                             │ │
│  │                                                   │ │
│  │  Session 1: 45 min (HIGH)   ████████████          │ │
│  │  Session 2: 30 min (MEDIUM) ████████              │ │
│  │  Session 3: 60 min (HIGH)   ████████████████      │ │
│  │                                                   │ │
│  │  Total: 2.25 hours                                │ │
│  └───────────────────────────────────────────────────┘ │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### Key Components

**`app.py` - Main Dashboard**
```python
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

st.set_page_config(page_title="Study Lamp Dashboard", layout="wide")

# Title
st.title("🌬️ AI-Powered Wind Energy Smart Study Lamp")

# Real-time metrics (top row)
col1, col2, col3 = st.columns(3)

with col1:
    battery_level = get_current_battery()  # From shared data
    st.metric("Battery Level", f"{battery_level}%", 
              delta=f"{get_battery_trend()}%")
    st.progress(battery_level / 100)

with col2:
    brightness = get_current_brightness()
    st.metric("Brightness Mode", brightness)
    st.write("💡" * get_brightness_icon_count(brightness))

with col3:
    study_hours = get_today_study_hours()
    st.metric("Study Hours Today", f"{study_hours:.1f} hrs")

# Feedback message
feedback = generate_feedback_message(battery_level)
st.info(feedback)

# Battery chart
st.subheader("Battery Level Over Time")
battery_history = load_battery_history()
fig, ax = plt.subplots()
ax.plot(battery_history['timestamp'], battery_history['battery'])
ax.set_xlabel('Time')
ax.set_ylabel('Battery %')
ax.set_ylim(0, 100)
st.pyplot(fig)

# Study sessions
st.subheader("Study Sessions Today")
sessions = load_today_sessions()
for idx, session in sessions.iterrows():
    st.write(f"Session {idx+1}: {session['duration_minutes']} min "
             f"({session['brightness_mode']})")
    st.progress(session['duration_minutes'] / 120)  # Max 2 hours

st.write(f"**Total: {sessions['duration_minutes'].sum() / 60:.2f} hours**")
```

**Feedback Message Logic**
```python
def generate_feedback_message(battery_level: int) -> str:
    """Generate user-friendly feedback based on battery level"""
    predicted_hours = get_predicted_study_hours()
    
    if battery_level > 60:
        return (f"✓ Good battery level - Full brightness available. "
                f"Predicted study time: {predicted_hours:.1f} hours remaining.")
    elif battery_level > 30:
        return (f"⚠️ Moderate battery - Medium brightness recommended. "
                f"Predicted study time: {predicted_hours:.1f} hours remaining.")
    elif battery_level > 10:
        return (f"⚠️ Low battery - Conserve energy, low brightness mode. "
                f"Predicted study time: {predicted_hours:.1f} hours remaining.")
    else:
        return (f"🔴 Critical battery - Charge soon or lamp will turn off. "
                f"Less than {predicted_hours:.1f} hours remaining.")
```

## Data Models

### Arduino Data Structures

```cpp
// Battery reading structure
struct BatteryReading {
    float voltage;
    int percentage;
    unsigned long timestamp;
};

// Brightness mode enum
enum BrightnessMode {
    OFF = 0,
    LOW = 1,
    MEDIUM = 2,
    HIGH = 3
};

// Study session structure
struct StudySession {
    unsigned long start_time;
    unsigned long end_time;
    int battery_start;
    int battery_end;
    BrightnessMode brightness;
};
```

### Python Data Models

**CSV Schema: `study_sessions.csv`**
```
timestamp,battery_start,battery_end,brightness_mode,duration_minutes
2024-01-01T10:00:00,85,75,HIGH,45
2024-01-01T15:30:00,75,65,MEDIUM,30
2024-01-01T19:00:00,65,50,HIGH,60
```

**Pandas DataFrame Schema**
```python
study_sessions_df = pd.DataFrame({
    'timestamp': pd.DatetimeIndex,      # Session start time
    'battery_start': int,                # Battery % at start
    'battery_end': int,                  # Battery % at end
    'brightness_mode': str,              # HIGH, MEDIUM, or LOW
    'duration_minutes': int              # Session length in minutes
})
```

**Real-time Data Exchange (JSON)**
```json
{
    "battery": {
        "percentage": 75,
        "voltage": 3.9,
        "trend": "stable",
        "critical": false
    },
    "brightness": {
        "current": "HIGH",
        "recommended": "HIGH",
        "pwm_value": 255
    },
    "session": {
        "active": true,
        "duration_minutes": 45,
        "start_battery": 85
    },
    "predictions": {
        "remaining_hours": 3.2,
        "depletion_time": "2024-01-01T22:15:00"
    }
}
```


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property Reflection

After analyzing all acceptance criteria, I identified the following redundancies and consolidations:

- **Battery monitoring properties (2.1-2.4)** can be consolidated into a single comprehensive property about voltage-to-percentage conversion
- **Brightness decision rules (3.1-3.3)** are three separate ranges but test the same decision function - combine into one property
- **PWM duty cycle mappings (4.2-4.4)** are specific examples that can be tested together as one property
- **Session data recording (5.3-5.5, 5.7)** all test that session data contains required fields - combine into one property
- **Dashboard feedback messages (8.1-8.3)** test the same message generation function - combine into one property
- **Brightness and battery relationship (3.1-3.3 and 14.5)** are redundant - already covered by brightness decision properties

### Properties

**Property 1: Voltage to Percentage Conversion Range**

*For any* battery voltage reading, the calculated battery percentage must be within the range 0-100%, and voltages outside the valid battery range (3.0V-4.2V) must be clamped to 0% or 100%.

**Validates: Requirements 2.2, 2.3**

---

**Property 2: Battery Reading Frequency**

*For any* 10-second time window during system operation, the battery voltage must be read at least once (satisfying the 5-second maximum interval requirement).

**Validates: Requirements 2.1**

---

**Property 3: Battery State Persistence**

*For any* battery reading, after the Arduino controller calculates the percentage, querying the stored battery state must return the same percentage value.

**Validates: Requirements 2.4**

---

**Property 4: Brightness Decision Rules**

*For any* battery percentage value, the AI module's brightness recommendation must follow these rules: battery > 60% → HIGH, battery 30-60% → MEDIUM, battery < 30% → LOW, battery < 5% → OFF.

**Validates: Requirements 3.1, 3.2, 3.3, 4.5**

---

**Property 5: Study Time Prediction Monotonicity**

*For any* two battery levels B1 and B2 where B1 > B2, and the same brightness mode, the predicted study hours for B1 must be greater than or equal to the predicted study hours for B2.

**Validates: Requirements 3.4, 10.3**

---

**Property 6: Brightness Update Timing**

*For any* 15-second time window during system operation, the AI module must update brightness recommendations at least once (satisfying the 10-second interval requirement).

**Validates: Requirements 3.5**

---

**Property 7: PWM Duty Cycle Mapping**

*For any* brightness mode (HIGH, MEDIUM, LOW, OFF), the PWM duty cycle must match the specified mapping: HIGH=100%, MEDIUM=60%, LOW=30%, OFF=0%.

**Validates: Requirements 4.2, 4.3, 4.4**

---

**Property 8: Session Timer Start on LED On**

*For any* system state where the LED transitions from OFF to ON, a new study session timer must be started with the current timestamp.

**Validates: Requirements 5.1**

---

**Property 9: Session Timer Stop on LED Off**

*For any* system state where the LED transitions from ON to OFF, the current study session timer must be stopped and duration calculated.

**Validates: Requirements 5.2**

---

**Property 10: Session Data Completeness**

*For any* completed study session, the recorded session data must contain all required fields: timestamp, battery_start, battery_end, brightness_mode, and duration_minutes.

**Validates: Requirements 5.3, 5.4, 5.5, 5.7**

---

**Property 11: CSV Session Data Round Trip**

*For any* valid study session object, writing it to CSV format and then reading it back must produce an equivalent session object with all fields preserved.

**Validates: Requirements 5.6, 9.1**

---

**Property 12: Serial Data Format**

*For any* serial transmission from Arduino, the data string must be formatted as comma-separated values containing battery percentage, brightness mode, duration, and timestamp.

**Validates: Requirements 6.4**

---

**Property 13: Serial Communication Timing**

*For any* 15-second time window during system operation, the Arduino must transmit status data via serial at least once (satisfying the 10-second interval requirement).

**Validates: Requirements 6.2**

---

**Property 14: Serial Error Resilience**

*For any* serial communication error (timeout, disconnection, invalid data), the system must continue operating and log the error without crashing.

**Validates: Requirements 6.5, 11.4**

---

**Property 15: Daily Study Hours Calculation**

*For any* set of study sessions, the total daily study hours must equal the sum of all session durations (in hours) for sessions occurring on the same calendar day.

**Validates: Requirements 7.3**

---

**Property 16: 24-Hour Data Filtering**

*For any* current timestamp T, the dashboard data filter must return only records with timestamps between T-24 hours and T.

**Validates: Requirements 7.7**

---

**Property 17: Feedback Message Generation**

*For any* battery percentage value, the dashboard feedback message must match the appropriate category: >60% → "Good battery", 30-60% → "Moderate battery", <30% → "Low battery", <10% → "Critical battery".

**Validates: Requirements 8.2, 8.3**

---

**Property 18: CSV Append Behavior**

*For any* existing CSV file with N session records, appending a new session must result in N+1 records, with all original records unchanged.

**Validates: Requirements 9.3**

---

**Property 19: CSV Persistence Across Restarts**

*For any* session data written to CSV, after simulating a system restart (closing and reopening the file), reading the CSV must return the same session data.

**Validates: Requirements 9.5**

---

**Property 20: Historical Data Loading**

*For any* valid CSV file containing session data, the AI module's data loading function must successfully parse all records and return a DataFrame with the correct schema.

**Validates: Requirements 9.6**

---

**Property 21: Average Power Consumption Calculation**

*For any* set of historical sessions grouped by brightness mode, the calculated average power consumption must be a positive number, and HIGH mode average must be greater than MEDIUM, which must be greater than LOW.

**Validates: Requirements 10.2**

---

**Property 22: Automatic LED Activation**

*For any* system state where battery percentage rises above the minimum threshold (5%) and was previously below it, the LED must automatically turn on at the appropriate brightness level.

**Validates: Requirements 14.2**

---

**Property 23: Battery Level Color Coding**

*For any* battery percentage value displayed on the dashboard, the color coding must follow: >60% → green, 30-60% → yellow, <30% → red.

**Validates: Requirements 14.4**

---

**Property 24: Study Time Maximization**

*For any* two brightness strategies S1 and S2 applied to the same battery level, if S1 results in longer total study time than S2, the AI module must prefer S1 over S2.

**Validates: Requirements 15.5**

---

### Edge Cases and Examples

The following are specific test cases that should be verified but don't require property-based testing:

**Edge Case 1: Low Battery Warning Threshold**
- When battery percentage falls below 10%, verify a low battery warning is logged
- **Validates: Requirements 2.5**

**Edge Case 2: Critical Battery Shutdown**
- When battery percentage reaches exactly 5%, verify LED turns off immediately
- **Validates: Requirements 4.5, 11.2**

**Edge Case 3: Critical Battery Dashboard Message**
- When battery percentage is below 10%, verify dashboard displays "Critical battery - Charge soon or lamp will turn off"
- **Validates: Requirements 8.4**

**Example 1: PWM Control Mechanism**
- Verify that brightness controller uses Arduino PWM functions (analogWrite)
- **Validates: Requirements 4.1**

**Example 2: Serial Communication Configuration**
- Verify serial port is initialized with 9600 baud rate
- **Validates: Requirements 6.3**

**Example 3: Dashboard Battery Display**
- Verify dashboard includes both numeric battery percentage and progress bar
- **Validates: Requirements 7.1**

**Example 4: Dashboard Brightness Display**
- Verify dashboard displays current brightness mode as text
- **Validates: Requirements 7.2**

**Example 5: Dashboard Battery Chart**
- Verify dashboard includes a line chart plotting battery percentage over time
- **Validates: Requirements 7.4**

**Example 6: Dashboard Session Chart**
- Verify dashboard includes a bar chart showing study session durations
- **Validates: Requirements 7.5**

**Example 7: Good Battery Feedback Message**
- When battery is above 60%, verify exact message: "Good battery level - Full brightness available"
- **Validates: Requirements 8.1**

**Example 8: Dashboard Prediction Display**
- Verify dashboard displays predicted remaining study hours
- **Validates: Requirements 8.5**

**Example 9: CSV Schema Validation**
- Verify CSV file contains exactly these columns: timestamp, battery_start, battery_end, brightness_mode, duration_minutes
- **Validates: Requirements 9.2**

**Example 10: CSV File Creation**
- When CSV file doesn't exist, verify system creates it with appropriate headers
- **Validates: Requirements 9.4**

**Example 11: Historical Data Analysis**
- Verify AI module has a function that analyzes historical session data
- **Validates: Requirements 10.1**

**Example 12: ML Mode Threshold**
- When historical data contains 10 or more sessions, verify AI module uses machine learning for predictions
- **Validates: Requirements 10.4**

**Example 13: Rule-Based Fallback**
- When historical data contains fewer than 10 sessions, verify AI module uses rule-based predictions
- **Validates: Requirements 10.5**

**Example 14: System Recovery After Power Loss**
- After simulating power loss and restart, verify system resumes normal operation
- **Validates: Requirements 11.3**

**Example 15: Battery Life Prediction**
- With fully charged battery (100%) at medium brightness, verify predicted study time is at least 4 hours
- **Validates: Requirements 11.5**

**Example 16: Offline Operation**
- Verify system makes no network calls during normal operation
- **Validates: Requirements 12.1**

**Example 17: Local AI Predictions**
- Verify AI module prediction functions only access local data sources
- **Validates: Requirements 12.2**

**Example 18: Dashboard Localhost**
- Verify dashboard runs on localhost without external dependencies
- **Validates: Requirements 12.3**

**Example 19: Local Data Storage**
- Verify all data files are created in local filesystem
- **Validates: Requirements 12.4**

**Example 20: Optional Cloud Sync**
- When cloud sync is enabled, verify sync functions are called
- **Validates: Requirements 12.5**

**Example 21: Zero Configuration**
- Verify system operates correctly with default settings and no user configuration
- **Validates: Requirements 14.1**

**Example 22: Browser Accessibility**
- Verify dashboard is accessible via web browser on localhost:8501
- **Validates: Requirements 14.6**

**Example 23: Low Power Mode**
- When LED is off, verify Arduino enters low-power mode
- **Validates: Requirements 15.2**

**Example 24: Energy Efficiency Metrics**
- Verify dashboard displays energy efficiency metrics
- **Validates: Requirements 15.6**

## Error Handling

### Arduino Error Handling

**Sensor Reading Errors**
- If ADC reading returns invalid value (outside expected range), use last known good value
- Log error to serial output for debugging
- Continue operation without crashing

**Serial Communication Errors**
- If serial buffer overflows, clear buffer and continue
- If serial write fails, retry up to 3 times with 100ms delay
- If all retries fail, log error and continue operation

**Timer Overflow**
- Arduino millis() overflows after ~49 days
- Handle overflow by detecting when current millis() < previous millis()
- Reset session timer on overflow

**PWM Errors**
- If PWM write fails, retry once
- If retry fails, maintain current brightness level
- Log error for debugging

### Python Error Handling

**Serial Connection Errors**
- If serial port cannot be opened, retry every 5 seconds
- Display connection status on dashboard
- Queue data for processing once connection restored

**CSV File Errors**
- If CSV file is corrupted, create backup and start new file
- If disk is full, delete oldest records to make space
- If write fails, retry up to 3 times

**Data Parsing Errors**
- If serial data is malformed, log error and skip that reading
- If CSV data is malformed, skip that row and continue
- Maintain running count of parse errors for monitoring

**ML Model Errors**
- If model training fails, fall back to rule-based predictions
- If model prediction fails, use rule-based prediction for that instance
- Log model errors for debugging

**Dashboard Errors**
- If data file is missing, display "No data available" message
- If plot generation fails, show error message but keep dashboard running
- If real-time update fails, retry on next update cycle

### Battery Protection

**Over-discharge Protection**
- Shut down LED at 5% battery to prevent battery damage
- Display critical warning on dashboard
- Prevent LED from turning on until battery reaches 10%

**Voltage Reading Validation**
- Reject voltage readings outside physical range (2.5V - 4.5V)
- Use median of last 3 readings to filter noise
- If voltage drops suddenly (>0.5V in one reading), treat as error

### Data Integrity

**Session Data Validation**
- Reject sessions with duration < 1 minute (likely errors)
- Reject sessions with impossible battery changes (>50% in one session)
- Validate timestamps are monotonically increasing

**CSV Data Validation**
- Verify CSV headers match expected schema before reading
- Skip rows with missing required fields
- Convert data types with error handling (try/except)

## Testing Strategy

### Dual Testing Approach

This system requires both **unit tests** and **property-based tests** for comprehensive coverage:

- **Unit tests**: Verify specific examples, edge cases, and error conditions
- **Property tests**: Verify universal properties across all inputs
- Both approaches are complementary and necessary

### Unit Testing Focus

Unit tests should focus on:
- Specific examples that demonstrate correct behavior (e.g., exact PWM values for each mode)
- Integration points between components (e.g., Arduino-Python serial communication)
- Edge cases and error conditions (e.g., battery at exactly 5%, CSV file doesn't exist)
- Hardware interface mocking (e.g., mock ADC readings, mock serial port)

Avoid writing too many unit tests for cases that property tests will cover (e.g., don't write 20 unit tests for different battery percentages when one property test can cover all values).

### Property-Based Testing Configuration

**Library Selection**:
- **Arduino/C++**: Use [QuickCheck for C++](https://github.com/grogers0/CppQuickCheck) or manual property test implementation
- **Python**: Use [Hypothesis](https://hypothesis.readthedocs.io/) for property-based testing

**Test Configuration**:
- Minimum **100 iterations** per property test (due to randomization)
- Each property test must include a comment tag referencing the design property
- Tag format: `# Feature: wind-energy-study-lamp, Property N: <property description>`

**Property Test Implementation**:
- Each correctness property listed above must be implemented as a SINGLE property-based test
- Use appropriate generators for test data (e.g., battery percentage 0-100, voltage 3.0-4.2V)
- Ensure generators cover edge cases (boundary values, empty data, maximum values)

### Arduino Testing

**Unit Tests**:
```cpp
// Test voltage to percentage conversion
void test_voltage_to_percentage() {
    assert(calculateBatteryPercentage(3.0) == 0);
    assert(calculateBatteryPercentage(4.2) == 100);
    assert(calculateBatteryPercentage(3.6) == 50);
}

// Test PWM mapping
void test_pwm_mapping() {
    setBrightness("HIGH");
    assert(getPWMValue() == 255);
    setBrightness("MEDIUM");
    assert(getPWMValue() == 153);
    setBrightness("LOW");
    assert(getPWMValue() == 77);
}
```

**Property Tests**:
```cpp
// Feature: wind-energy-study-lamp, Property 1: Voltage to Percentage Conversion Range
void property_voltage_to_percentage_range() {
    for (int i = 0; i < 100; i++) {
        float voltage = random(2.5, 4.5);  // Random voltage
        int percentage = calculateBatteryPercentage(voltage);
        assert(percentage >= 0 && percentage <= 100);
    }
}
```

### Python Testing

**Unit Tests**:
```python
def test_csv_file_creation():
    """Example 10: CSV File Creation"""
    # When CSV doesn't exist, verify it's created with headers
    if os.path.exists('test.csv'):
        os.remove('test.csv')
    
    logger = DataLogger('test.csv')
    assert os.path.exists('test.csv')
    
    df = pd.read_csv('test.csv')
    expected_columns = ['timestamp', 'battery_start', 'battery_end', 
                       'brightness_mode', 'duration_minutes']
    assert list(df.columns) == expected_columns
```

**Property Tests**:
```python
from hypothesis import given, strategies as st

# Feature: wind-energy-study-lamp, Property 4: Brightness Decision Rules
@given(battery=st.integers(min_value=0, max_value=100))
def test_brightness_decision_rules(battery):
    """Property 4: Brightness Decision Rules"""
    optimizer = BrightnessOptimizer()
    brightness = optimizer.decide_brightness(battery, 12, pd.DataFrame())
    
    if battery < 5:
        assert brightness == "OFF"
    elif battery < 30:
        assert brightness == "LOW"
    elif battery < 60:
        assert brightness == "MEDIUM"
    else:
        assert brightness == "HIGH"

# Feature: wind-energy-study-lamp, Property 11: CSV Session Data Round Trip
@given(
    battery_start=st.integers(min_value=0, max_value=100),
    battery_end=st.integers(min_value=0, max_value=100),
    brightness=st.sampled_from(['HIGH', 'MEDIUM', 'LOW']),
    duration=st.integers(min_value=1, max_value=300)
)
def test_csv_round_trip(battery_start, battery_end, brightness, duration):
    """Property 11: CSV Session Data Round Trip"""
    session_data = {
        'timestamp': datetime.now().isoformat(),
        'battery_start': battery_start,
        'battery_end': battery_end,
        'brightness_mode': brightness,
        'duration_minutes': duration
    }
    
    logger = DataLogger('test_roundtrip.csv')
    logger.log_session(session_data)
    
    df = logger.load_historical_data()
    last_row = df.iloc[-1].to_dict()
    
    assert last_row['battery_start'] == battery_start
    assert last_row['battery_end'] == battery_end
    assert last_row['brightness_mode'] == brightness
    assert last_row['duration_minutes'] == duration

# Feature: wind-energy-study-lamp, Property 5: Study Time Prediction Monotonicity
@given(
    b1=st.integers(min_value=10, max_value=100),
    b2=st.integers(min_value=10, max_value=100),
    mode=st.sampled_from(['HIGH', 'MEDIUM', 'LOW'])
)
def test_prediction_monotonicity(b1, b2, mode):
    """Property 5: Study Time Prediction Monotonicity"""
    predictor = StudyPredictor()
    
    hours_b1 = predictor.predict_study_hours(b1, mode)
    hours_b2 = predictor.predict_study_hours(b2, mode)
    
    if b1 > b2:
        assert hours_b1 >= hours_b2
    elif b1 < b2:
        assert hours_b1 <= hours_b2
    else:
        assert hours_b1 == hours_b2
```

### Integration Testing

**Arduino-Python Communication**:
- Test serial data transmission and parsing
- Test command sending from Python to Arduino
- Test handling of disconnection and reconnection

**End-to-End Workflow**:
- Simulate complete study session from LED on to LED off
- Verify data flows through all components
- Verify CSV logging and dashboard display

**Dashboard Testing**:
- Test dashboard with various data scenarios (empty, partial, full)
- Test real-time updates with mock data stream
- Test error display when data is unavailable

### Test Data Generators

**Battery Percentage Generator**:
```python
battery_percentage = st.integers(min_value=0, max_value=100)
```

**Voltage Generator**:
```python
voltage = st.floats(min_value=3.0, max_value=4.2)
```

**Brightness Mode Generator**:
```python
brightness_mode = st.sampled_from(['HIGH', 'MEDIUM', 'LOW', 'OFF'])
```

**Session Data Generator**:
```python
session_data = st.fixed_dictionaries({
    'timestamp': st.datetimes(),
    'battery_start': st.integers(min_value=0, max_value=100),
    'battery_end': st.integers(min_value=0, max_value=100),
    'brightness_mode': st.sampled_from(['HIGH', 'MEDIUM', 'LOW']),
    'duration_minutes': st.integers(min_value=1, max_value=300)
})
```

### Test Coverage Goals

- **Arduino firmware**: 80%+ code coverage
- **Python AI module**: 90%+ code coverage
- **Dashboard**: 70%+ code coverage (UI testing is limited)
- **All 24 properties**: 100% implemented as property-based tests
- **All 24 examples/edge cases**: 100% implemented as unit tests

### Continuous Testing

- Run unit tests on every code change
- Run property tests before each commit
- Run integration tests daily
- Monitor test execution time (property tests should complete in < 5 minutes)
- Track and report test failures immediately
