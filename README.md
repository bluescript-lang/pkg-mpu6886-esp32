# MPU6886 Library for ESP32

A driver library for the **MPU6886** 6-axis IMU (Inertial Measurement Unit) sensor for **BlueScript**.
This package provides a high-level interface to easily initialize the sensor, configure ranges, and read accelerometer, gyroscope, and internal temperature data via the I2C bus.

## Installation

Install this package in your BlueScript project:

```bash
bscript project install https://github.com/bluescript-lang/pkg-mpu6886.git
```

## Usage

### Basic: Read Acceleration and Gyroscope Data

```typescript
import * as time from "time";
import { I2CMasterBus, I2CPort } from "i2c";
import { MPU6886, GyroRange, AccelRange } from "mpu6886";

// 1. Initialize I2C Bus (e.g., SDA: 21, SCL: 22)
const bus = new I2CMasterBus(I2CPort.I2C0, 21, 22);

// 2. Create MPU6886 instance and initialize
const imu = new MPU6886(bus);

if (!imu.init()) {
    console.log("Failed to initialize MPU6886!");
    bus.close();
    // Stop execution or handle error appropriately
} else {
    console.log("MPU6886 initialized successfully.");
}

// Optional: Change measurement ranges (Defaults are 8G and 2000 DPS)
imu.setAccelRange(AccelRange.G4);
imu.setGyroRange(GyroRange.Dps500);

// 3. Read data in a loop
while (true) {
    const accel = imu.getAcceleration();
    const gyro = imu.getGyroscope();
    const temp = imu.getTemperature();

    if (accel !== null && gyro !== null) {
        console.log("Accel (G)   X: " + accel.x + ", Y: " + accel.y + ", Z: " + accel.z);
        console.log("Gyro  (DPS) X: " + gyro.x + ", Y: " + gyro.y + ", Z: " + gyro.z);
        console.log("Temp  (C) " + temp);
    } else {
        console.log("Failed to read sensor data.");
    }

    time.delay(100);
}

// imu.deinit();
// bus.close();
```

## API Reference

### Class: `MPU6886`

The primary class representing the MPU6886 sensor device.

#### `constructor(bus: I2CMasterBus)`
Creates an instance of the MPU6886 sensor using an existing I2C bus.
- **bus**: An initialized `I2CMasterBus` object.
- *Note: Automatically targets I2C address `0x68` at `400kHz`. Defaults to `AccelRange.G8` and `GyroRange.Dps2000`.*

#### `init(): boolean`
Wakes up and configures the sensor. Checks device ID (`WHO_AM_I`), resets power management, and applies default configurations.
- **Returns**: `true` if initialization is successful; `false` if the device is not found or fails to respond.

#### `deinit(): void`
Puts the sensor into sleep mode and closes the underlying I2C device handle.

#### `setGyroRange(range: GyroRange): boolean`
Sets the full-scale measurement range of the gyroscope and updates the internal resolution multiplier.
- **range**: A `GyroRange` enum value.
- **Returns**: `true` on success, `false` on failure.

#### `setAccelRange(range: AccelRange): boolean`
Sets the full-scale measurement range of the accelerometer and updates the internal resolution multiplier.
- **range**: An `AccelRange` enum value.
- **Returns**: `true` on success, `false` on failure.

#### `getAcceleration(): Vector3D | null`
Reads the current accelerometer data (X, Y, Z axes).
- **Returns**: A `Vector3D` object representing acceleration in **G**, or `null` if the I2C read fails.

#### `getGyroscope(): Vector3D | null`
Reads the current gyroscope data (X, Y, Z axes).
- **Returns**: A `Vector3D` object representing angular velocity in **DPS** (degrees per second), or `null` if the I2C read fails.

#### `getTemperature(): float`
Reads the internal temperature sensor.
- **Returns**: Temperature in **Celsius**.

#### `setGyroOffset(x: integer, y: integer, z: integer): boolean`
Manually applies raw hardware offsets to the gyroscope axes.
- **Returns**: `true` on success, `false` on failure.


### Class: `Vector3D`

A simple data container used for returning 3-axis float readings.

#### **Properties**
- `x: float`
- `y: float`
- `z: float`


## Enums

### `GyroRange`
Defines the full-scale range for the gyroscope.

| Name | Value | Measurement Range |
| :--- | :--- | :--- |
| `Dps250` | 0 | ±250 deg/s |
| `Dps500` | 1 | ±500 deg/s |
| `Dps1000` | 2 | ±1000 deg/s |
| `Dps2000` | 3 | ±2000 deg/s |

### `AccelRange`
Defines the full-scale range for the accelerometer.

| Name | Value | Measurement Range |
| :--- | :--- | :--- |
| `G2` | 0 | ±2 G |
| `G4` | 1 | ±4 G |
| `G8` | 2 | ±8 G |
| `G16` | 3 | ±16 G |