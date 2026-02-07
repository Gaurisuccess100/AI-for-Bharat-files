# Requirements Document: AI-Powered Wind Energy Smart Study Lamp

## Introduction

The AI-Powered Wind Energy Smart Study Lamp is a renewable energy-based educational support system designed for students in rural and low-power areas. The system harvests wind energy through a small turbine, stores it in a battery, and uses AI-driven logic to intelligently control LED brightness, maximizing study time while optimizing limited energy resources. A dashboard provides real-time feedback on power usage, study hours, and system performance.

## Glossary

- **System**: The complete AI-Powered Wind Energy Smart Study Lamp including hardware, firmware, AI logic, and dashboard
- **Arduino_Controller**: The Arduino Nano microcontroller that manages hardware interfaces and control logic
- **AI_Module**: The Python-based artificial intelligence component that predicts study hours and optimizes brightness
- **Dashboard**: The Streamlit-based web interface for visualization and user feedback
- **Battery_Monitor**: The subsystem responsible for reading and reporting battery voltage levels
- **Brightness_Controller**: The PWM-based LED control subsystem
- **Wind_Turbine**: The DC motor-based wind energy harvesting device
- **Charging_Module**: The TP4056 battery charging circuit
- **Study_Session**: A continuous period during which the lamp is actively used for studying
- **Battery_Level**: The current charge state of the Li-ion battery expressed as a percentage
- **Brightness_Mode**: One of three LED intensity levels (High, Medium, Low)

## Requirements

### Requirement 1: Wind Energy Harvesting

**User Story:** As a student in a rural area with unreliable electricity, I want the lamp to harvest wind energy, so that I can study even during power outages.

#### Acceptance Criteria

1. WHEN wind rotates the turbine, THE Wind_Turbine SHALL generate DC electrical energy
2. WHEN the turbine generates energy, THE Charging_Module SHALL charge the Li-ion battery safely
3. THE Charging_Module SHALL prevent overcharging of the battery
4. THE Charging_Module SHALL prevent reverse current flow from the battery to the turbine
5. WHEN the battery reaches full capacity, THE Charging_Module SHALL stop charging automatically

### Requirement 2: Battery Monitoring

**User Story:** As a system, I need to continuously monitor battery status, so that I can make intelligent decisions about energy usage.

#### Acceptance Criteria

1. THE Battery_Monitor SHALL read battery voltage at least once every 5 seconds
2. THE Battery_Monitor SHALL convert voltage readings to battery percentage (0-100%)
3. WHEN battery voltage is read, THE Battery_Monitor SHALL calculate remaining capacity accurately within 5% tolerance
4. THE Arduino_Controller SHALL store the current battery percentage in memory
5. WHEN battery percentage falls below 10%, THE System SHALL log a low battery warning

### Requirement 3: AI-Based Brightness Optimization

**User Story:** As a student with limited energy resources, I want the lamp to automatically adjust brightness based on available power, so that I can maximize my study time.

#### Acceptance Criteria

1. WHEN battery percentage is above 60%, THE AI_Module SHALL recommend High brightness mode
2. WHEN battery percentage is between 30% and 60%, THE AI_Module SHALL recommend Medium brightness mode
3. WHEN battery percentage is below 30%, THE AI_Module SHALL recommend Low brightness mode
4. THE AI_Module SHALL predict available study hours based on current battery level and historical usage patterns
5. THE AI_Module SHALL update brightness recommendations every 10 seconds
6. WHEN brightness mode changes, THE System SHALL transition smoothly within 2 seconds

### Requirement 4: LED Brightness Control

**User Story:** As a student, I want the lamp to provide appropriate lighting for studying, so that I can read comfortably while conserving energy.

#### Acceptance Criteria

1. THE Brightness_Controller SHALL control LED intensity using PWM signals
2. WHEN High brightness mode is active, THE Brightness_Controller SHALL set PWM duty cycle to 100%
3. WHEN Medium brightness mode is active, THE Brightness_Controller SHALL set PWM duty cycle to 60%
4. WHEN Low brightness mode is active, THE Brightness_Controller SHALL set PWM duty cycle to 30%
5. WHEN battery percentage reaches 5%, THE Brightness_Controller SHALL turn off the LED to preserve battery health
6. THE Brightness_Controller SHALL maintain stable brightness without flickering

### Requirement 5: Study Session Tracking

**User Story:** As a student, I want the system to track my study sessions, so that I can understand my study patterns and energy usage.

#### Acceptance Criteria

1. WHEN the LED turns on, THE System SHALL start a new study session timer
2. WHEN the LED turns off, THE System SHALL stop the current study session timer
3. THE System SHALL record study session duration in minutes
4. THE System SHALL record the brightness mode used during each session
5. THE System SHALL record the battery percentage at session start and end
6. THE System SHALL store study session data in CSV format
7. THE System SHALL timestamp each study session with date and time

### Requirement 6: Data Communication

**User Story:** As a system administrator, I want the Arduino to communicate with the dashboard, so that I can monitor system performance in real-time.

#### Acceptance Criteria

1. THE Arduino_Controller SHALL send data to the connected computer via serial communication
2. THE Arduino_Controller SHALL transmit battery percentage, brightness mode, and study duration every 10 seconds
3. THE System SHALL use a baud rate of 9600 for serial communication
4. WHEN serial data is transmitted, THE Arduino_Controller SHALL format data as comma-separated values
5. THE System SHALL handle serial communication errors gracefully without crashing

### Requirement 7: Dashboard Visualization

**User Story:** As a student or teacher, I want to see visual representations of energy usage and study patterns, so that I can make informed decisions about lamp usage.

#### Acceptance Criteria

1. THE Dashboard SHALL display current battery percentage as a numeric value and progress bar
2. THE Dashboard SHALL display current brightness mode (High, Medium, Low)
3. THE Dashboard SHALL display total study hours for the current day
4. THE Dashboard SHALL plot battery percentage over time as a line graph
5. THE Dashboard SHALL plot study session durations as a bar chart
6. THE Dashboard SHALL update visualizations in real-time when new data arrives
7. THE Dashboard SHALL display the last 24 hours of data by default

### Requirement 8: Student Feedback System

**User Story:** As a student, I want to receive feedback messages about my lamp usage, so that I can optimize my study schedule based on available energy.

#### Acceptance Criteria

1. WHEN battery percentage is above 60%, THE Dashboard SHALL display "Good battery level - Full brightness available"
2. WHEN battery percentage is between 30% and 60%, THE Dashboard SHALL display "Moderate battery - Medium brightness recommended"
3. WHEN battery percentage is below 30%, THE Dashboard SHALL display "Low battery - Conserve energy, low brightness mode"
4. WHEN battery percentage is below 10%, THE Dashboard SHALL display "Critical battery - Charge soon or lamp will turn off"
5. THE Dashboard SHALL display predicted remaining study hours based on current battery level
6. THE Dashboard SHALL provide actionable recommendations for maximizing study time

### Requirement 9: Data Persistence

**User Story:** As a researcher or teacher, I want historical data to be saved, so that I can analyze long-term usage patterns and system performance.

#### Acceptance Criteria

1. THE System SHALL store all study session data in a CSV file
2. THE CSV file SHALL contain columns for timestamp, battery_start, battery_end, brightness_mode, duration_minutes
3. THE System SHALL append new session data without overwriting existing records
4. WHEN the CSV file does not exist, THE System SHALL create it with appropriate headers
5. THE System SHALL maintain data integrity even after system restarts
6. THE AI_Module SHALL read historical data from the CSV file for pattern analysis

### Requirement 10: AI Prediction and Learning

**User Story:** As a student, I want the system to learn from my usage patterns, so that it can better predict how long I can study with available energy.

#### Acceptance Criteria

1. THE AI_Module SHALL analyze historical study session data to identify usage patterns
2. THE AI_Module SHALL calculate average power consumption per brightness mode
3. THE AI_Module SHALL predict remaining study hours based on current battery level and average consumption
4. WHEN sufficient historical data exists (minimum 10 sessions), THE AI_Module SHALL use machine learning for predictions
5. WHEN insufficient historical data exists, THE AI_Module SHALL use rule-based predictions
6. THE AI_Module SHALL update prediction models weekly based on new data

### Requirement 11: System Reliability

**User Story:** As a student in a rural area, I need the lamp to work reliably in challenging conditions, so that I can depend on it for my education.

#### Acceptance Criteria

1. THE System SHALL operate correctly in temperatures between 0°C and 45°C
2. THE System SHALL protect the battery from over-discharge by shutting down at 5% capacity
3. WHEN power is lost and restored, THE System SHALL resume normal operation automatically
4. THE Arduino_Controller SHALL handle sensor reading errors without crashing
5. THE System SHALL operate continuously for at least 4 hours on a fully charged battery at medium brightness
6. THE System SHALL have a mean time between failures (MTBF) of at least 1000 hours

### Requirement 12: Offline Operation

**User Story:** As a student in an area with no internet connectivity, I want the system to work completely offline, so that I can use it regardless of network availability.

#### Acceptance Criteria

1. THE System SHALL operate without requiring internet connectivity
2. THE AI_Module SHALL perform all predictions and decisions using local data only
3. THE Dashboard SHALL function on localhost without external dependencies
4. THE System SHALL store all data locally on the connected computer
5. WHERE internet is available, THE System SHALL optionally sync data to cloud storage

### Requirement 13: Cost Efficiency

**User Story:** As a low-income household, I need the lamp to be affordable, so that I can provide educational support for my children.

#### Acceptance Criteria

1. THE System SHALL use components with a total bill of materials (BOM) cost under $30 USD
2. THE System SHALL use readily available, off-the-shelf components
3. THE System SHALL minimize power consumption to reduce battery replacement costs
4. THE System SHALL have a design lifetime of at least 2 years with normal usage
5. THE System SHALL be repairable with basic tools and common replacement parts

### Requirement 14: User Interface Simplicity

**User Story:** As a student with limited technical knowledge, I want the system to be easy to use, so that I can focus on studying rather than managing the lamp.

#### Acceptance Criteria

1. THE System SHALL require no user configuration for basic operation
2. THE LED SHALL turn on automatically when sufficient battery is available
3. THE Dashboard SHALL use clear, simple language appropriate for students aged 10-18
4. THE Dashboard SHALL use intuitive icons and color coding (green=good, yellow=moderate, red=low)
5. THE System SHALL provide visual indicators (LED brightness changes) for battery status
6. THE Dashboard SHALL be accessible via a simple web browser without installation

### Requirement 15: Energy Efficiency

**User Story:** As an environmental advocate, I want the system to maximize energy efficiency, so that it makes the best use of renewable wind energy.

#### Acceptance Criteria

1. THE System SHALL achieve at least 80% energy conversion efficiency from battery to LED
2. THE Arduino_Controller SHALL enter low-power mode when the LED is off
3. THE Boost_Converter SHALL maintain at least 85% efficiency at all load levels
4. THE System SHALL minimize quiescent current draw to less than 10mA when idle
5. THE AI_Module SHALL optimize brightness to maximize total study time rather than peak brightness
6. THE System SHALL track and report energy efficiency metrics via the dashboard
