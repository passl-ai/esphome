# Solar Excess Water Boiler

## Motivation for solar excess heating coil

Target is to use the excess electrical power produced by the solar panels to heat the water in the hotwater buffer storage of a pellet heating. Due to German energy market regulations, no compensation is paid, if the excess power is delivered to the power grid (for so-called "Balkonkraftwerk", for others about 8-10 cents/kWh in 2024). Thus it makes sense to consome as much as possible.

For small installations of only a few panels (e.g. 1 - 4 panels) batteries hardly make sense from an economic point of view. However, the hardware cost for storing the energy by heating up water is much lower, if the buffer storage is already installed: metering, relay, some wiring, control unit, heating coil. This can be built for less than 100 Euros.

There are a few **challenges** to overcome:

safety considerations: prevent overheating, peak currents, ...

as no power should be consumed from the grid (cost per kWh much higher than gain from using excess energy instead of just not using it), it has to be made sure that only the excess production is consumed -> heating coil has to be "throttled" accordingly.

This is tricky because a) the excess has to be measured and be taken into account and b) the throttling has to be done by burst control ("Wellenpaketsteuerung") for EMV reasons (+ German regulation). This is achieved by controlling a zero crossing solid state relay via (slow) PWM via a ESP32.

## Loosely inspired by

<https://www.photovoltaikforum.com/thread/188108-balkon-pv-100-energie-nutzung-ww-erw%C3%A4rmung/?postID=3284339#post3284339>

<https://github.com/passl-tech/homelab/blob/main/esphome/diff_boiler/PhotovoltaikForumSolution/README_PhtotovoltaikForum.md>

PWM values: <https://www.photovoltaikforum.com/core/attachment/365781-tabelle-png/>

## Own ESPHome based solution

Mainly based on zero-crossing solid state relay controlled by an ESP32 board using the **Slow PWM Output** component. This controls a heating coil which is in the (main) water tank.

1. Zero Crossing Solid State Relay (here: SSR-50 DA (8990ZM); 24-380VAC; 3-32VDC; ca. 10ms Schaltzeit)
2. heating coil (here 1,5kW; 1,5"; max. temperature control)

ESP GND to minus (-) @SSR relay, ESP PIN to pkus (+) @ SSR on DC side.

### Safety functions

- power off, if no change in value for x seconds
- power off, if no wifi connection
- power off, if water temperature above a certain temperature

## Calculations

### One-time loss of hot water on installation

As in my case the water buffer is already in use, it has to be emptied a little more than half-way to install the heating coil. Here is the worst-case calculation for the energy lost during this process:
$$450l * 50K * 4.2 kJ = 94.5MJ = 26.25kWh$$
This would result in about 3 Euros of money wasted at a energy price of about 10 cents per kWh (estimation includes losses) of heat for pellet heatings (2024). Thus it is very reasonable to do this at any time (ideally not during the heating period).

### Example values for burst control

- 50 Hz AC -> 50 full sinus waves per s -> 100 half waves per s ^= on/off points at zero crossing within 1s
- assumption: smartmeter integrating at 5Hz -> buckets of 200ms -> 20 levels -> 5% of max heating coil power added/reduced per step. PROD: total power of coil: 1.5kW -> 75W per step
