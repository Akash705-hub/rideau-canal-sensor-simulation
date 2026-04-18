# Rideau Canal Sensor Simulation

This project simulates three IoT devices that send winter-condition telemetry for Rideau Canal locations to Azure IoT Hub.

## What This App Does

- Loads three device connection strings from a local `.env` file.
- Creates one Azure IoT device client per location:
	- Dow's Lake
	- Fifth Avenue
	- NAC
- Generates synthetic telemetry values for each location.
- Sends telemetry to Azure IoT Hub every 10 seconds in an infinite loop.

## Project Structure

```text
rideau-canal-sensor-simulation/
├── README.md
├── requirements.txt
└── sensor_simulation.py
```

## Prerequisites

- Python 3.9+
- An Azure IoT Hub with 3 registered devices
- One device connection string for each location

## Setup

1. Clone the repository and open it in VS Code.
2. (Recommended) Create and activate a virtual environment.
3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Create a `.env` file in the project root with the following variables:

```env
DOWS_LAKE_CONN_STR=HostName=...;DeviceId=...;SharedAccessKey=...
FIFTH_AVE_CONN_STR=HostName=...;DeviceId=...;SharedAccessKey=...
NAC_CONN_STR=HostName=...;DeviceId=...;SharedAccessKey=...
```

## Run

```bash
python sensor_simulation.py
```

The app will continue sending telemetry until you stop it with `Ctrl+C`.

## Telemetry Schema

Each message includes:

- `ice_thickness` (cm, simulated float in range 1.0-58.0)
- `surface_temperature` (degrees C, simulated float in range -15.0-5.0)
- `snow_accumulation` (cm, simulated float in range 0.0-30.0)
- `external_temperature` (degrees C, simulated float in range -20.0-5.0)
- `timestamp` (UTC, formatted as `%Y-%m-%d %H:%M:%S`)
- `location` (one of the three monitored locations)

Example message payload:

```python
{
	'ice_thickness': 35.72,
	'surface_temperature': -6.41,
	'snow_accumulation': 12.03,
	'external_temperature': -10.54,
	'timestamp': '2026-04-18 20:14:02',
	'location': "Dow's Lake"
}
```

## Code Explanation

### 1. Imports and Environment Loading

In `sensor_simulation.py`, the script imports standard modules (`os`, `time`, `random`) plus Azure IoT and dotenv libraries.

It then resolves the current file directory and loads variables from `.env`:

- `DOWS_LAKE_CONN_STR`
- `FIFTH_AVE_CONN_STR`
- `NAC_CONN_STR`

These are stored in a `locations` dictionary that maps location names to connection strings.

### 2. `get_telemetry(location)`

This function creates one simulated sensor reading dictionary for the provided location.

- Uses `random.uniform(...)` to generate realistic numeric ranges.
- Uses `time.gmtime()` and `time.strftime(...)` for a UTC timestamp.
- Returns a Python dictionary containing all telemetry fields.

### 3. `main()`

The main loop handles device creation and message sending.

1. Build one `IoTHubDeviceClient` per location using the connection string.
2. Enter an infinite `while True` loop.
3. For each location/client pair:
	 - Generate telemetry via `get_telemetry(location)`.
	 - Wrap it in an Azure `Message` object.
	 - Send it to IoT Hub with `client.send_message(...)`.
4. Sleep for 10 seconds.

The `try/except/finally` structure ensures:

- `Ctrl+C` stops execution cleanly.
- All device clients are disconnected in `finally`.

### 4. Script Entry Point

```python
if __name__ == "__main__":
		main()
```

This starts the simulator only when the file is run directly.

## Notes and Troubleshooting

- If the script exits immediately, verify all three `.env` variables are present and valid.
- Ensure each connection string belongs to a registered IoT Hub device.
- If dependency install fails for `dotenv`, use `python-dotenv` in your environment.
