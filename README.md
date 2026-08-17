# Software-in-the-Loop Attitude Control and Estimation Stack

**A concise 1-DOF SIL prototype demonstrating nonlinear estimation and cascaded feedback control for aerospace systems.**

## Problem Statement
In aerospace vehicles (like quadrotors or rockets), control systems must track commanded orientations. However, true vehicle attitude cannot be measured directly. It must be estimated using raw, noisy sensor data (IMU) and then fed into a feedback control algorithm. This project demonstrates the full pipeline of processing noisy sensor inputs into stable physical motion.

## Architecture
The repository separates the concerns of physical simulation, sensor modeling, estimation, and control, allowing the software side to map directly to an embedded RTOS environment.

```text
Simulated Vehicle (True State)
       ↓
Simulated IMU (Adds Noise/Bias)
       ↓
Mahony Estimator (Quaternion-based)
       ↓
Estimated Attitude & Rate
       ↓
Cascaded PID (Angle -> Rate)
       ↓
Control Torque
       ↓
Simulated Plant (Rigid Body Dynamics)
       ↓
Vehicle State
```

## Engineering Decisions
* **Why Mahony instead of EKF?** An Extended Kalman Filter is powerful but computationally heavy. The Mahony filter relies on cross-product error and a PI loop on the quaternion derivative. It is extremely computationally lightweight, numerically stable, and highly appropriate for high-rate (>500Hz) embedded execution on microcontrollers like the STM32. 
* **Why Cascaded PID?** Splitting the controller into an outer angle loop and inner rate loop prevents violent actuator oscillations, provides natural damping, and allows for explicit limiting of the vehicle's angular rate.
* **Why single-axis pitch?** Focusing strictly on 1-DOF proves out the control logic, anti-windup, and estimator tuning without muddying the code with complex 6-DOF cross-coupling aerodynamics. 
* **Software-in-the-Loop (SIL):** Validating in Python first enables rapid tuning of gains, fault injection testing, and automated unit testing before touching fragile real-world hardware.

## Limitations
* The plant dynamics are highly simplified and omit aerodynamic drag curves.
* Only the 1-DOF pitch axis is simulated; yaw and roll coupling are ignored.
* There is no real hardware validation or real-time performance claim here; it runs strictly as offline Python verification.
