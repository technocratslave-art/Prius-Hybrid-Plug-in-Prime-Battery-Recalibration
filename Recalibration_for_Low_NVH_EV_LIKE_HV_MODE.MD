# Executive Brief: Stealth Comfort Mode
### A Firmware-Only NVH Calibration for the 5th Gen Prius Prime

## The Core Concept

The 5th-generation Prius Prime carries a 13.6 kWh plug-in battery and a 2.0L Atkinson engine that is among the most thermally efficient gasoline engines in production. Today the car uses that hardware only to chase fuel economy. Stealth Comfort Mode re-calibrates the existing architecture to chase something the vehicle has never offered: luxury-class silence.

By using the plug-in battery as an active macro-buffer, the power management ECU decouples engine speed from the driver's foot. Instead of the familiar hybrid drone that rises and falls with acceleration, the engine slips in as a low, steady, unchanging hum while the battery absorbs every transient load spike. The result is the instant torque and quiet cabin of a premium EV, backed by the zero-anxiety range of a gas tank. Below about half pedal it behaves as a refined, EV-like cruiser; above it, the full hybrid system wakes for performance. Two personalities on one button.

This is a refinement feature first, not an economy play, and it is worth being clear about that. The mode is opt-in, so the certified EPA label, measured in the car's standard configuration, does not change. Preliminary modeling across multiple city and highway mixes indicates fuel economy stays neutral to modestly better, from no measurable loss up to a few percent, because a steady, loaded engine avoids the frantic short-cycling of standard hybrid logic. That is upside to confirm on a dynamometer, not the reason to build it. The mode integrates behind the existing **[Auto EV/HV]** button, requiring zero hardware changes, zero user retraining, and zero modifications to the manufacturing line.

```
[ Driver Pedal Input ] ---> [ Power Management ECU ]
                                    |
       +----------------------------+----------------------------+
       v                                                         v
 [ 13.6 kWh Battery Buffer ]                          [ M20A-FXS Engine ]
 Instantly deploys 60+ kW                             Held to a steady, quiet,
 to absorb transient spikes                           highly efficient rpm floor
       |                                                         |
       +----------------------------+----------------------------+
                                    v
                          [ eCVT Transaxle / MG2 ]
                                    |
                                    v
                     [ Linear, Silent Acceleration ]
```

## The Three Engineering Payoffs

A naive series-hybrid calibration would introduce real NVH, thermal, and kinematic penalties. This strategy clears all three with precise, production-grade software safeguards, each with precedent in shipping vehicles:

* **Speed-Linked RPM Floor.** To avoid transaxle power-recirculation losses and inverter back-EMF stress at highway speed, the engine is not pinned to one speed. It holds a serene 1,600 rpm up to 45 mph, then scales smoothly to about 2,100 rpm by 70 mph. That keeps the motor-generators inside their efficient electrical range, while wind and tire noise fully mask the slight rise in engine pitch.
* **Anti-Shudder Boom Mitigation.** To eliminate the 53.3 Hz second-order structural boom inherent to a four-cylinder under load, the firmware caps engine load near 45% inside the resonance zone, softening the firing pulse at its source. A Schmitt-trigger hysteresis band stops boundary chatter, and MG2 backfills the exact torque deficit through the planetary set, so the pedal stays perfectly linear and the cabin stays smooth.
* **Predictive Thermal Window.** To hold SULEV30 through the cold-start FTP-75 cycle, the ECU runs a predictive model. When navigation data or driver pedal patterns indicate the engine will soon be needed, it runs a decoupled, ultra-low-load catalyst warm-up at about 1,200 rpm beforehand, so the catalyst is fully lit before it ever faces a high-load generator demand. No cold-cat hydrocarbon slug, no torque hole.

## Why It Wins

Most manufacturers calibrate plug-in hybrids with the nervous, short-cycling logic of a standard hybrid, because it is the safe, known baseline for emissions and economy testing. This calibration uses the Prime's plug-in pack and its electrical throughput to treat the engine as a stabilized, deliberate power plant rather than a frantic mechanical assistant.

It bridges the gap between spreadsheet efficiency and an emotional, premium user experience, with no penalty to the rating sheet and no new parts. It is an EV feel that runs on infrastructure available at every street corner on earth, delivered entirely through smarter code. And it is newly possible. No earlier Prius had the battery capacity or the discharge power to buffer the engine this way; the 13.6 kWh pack is the first that can. The hardware just caught up to the idea.

# Technical Appendix: Powertrain Calibration Specification

## Part 1 — The Idea

### What it is

The idea is to lean on the big battery and the electric motors so the gas engine almost never has to raise its voice. You get most of what people brag about in a Tesla, the instant torque and the quiet cabin, except you are doing it on petrol with a gas tank backing you up, so you never think about range or hunt for a charger.

### What it feels like to drive

You pull away in silence on electric, like any EV. As you settle into traffic the engine slips in underneath you, but you do not get the familiar Prius drone that climbs and falls with your foot. You get a low, steady hum that holds one pitch, the way a well-insulated luxury car sounds. Roll on the throttle to merge and the surge is instant, the engine note staying flat, because the battery is covering the spike, not the engine. On the highway the hum lifts a little, but by then wind and tire noise own the cabin and you do not notice it. The only time it ever truly raises its voice is when you ask for everything, past about half pedal, and then it sounds like a car that means it. Lift off and it sinks right back into the quiet.

What makes that work is a deliberate inversion of stock hybrid behavior. In a normal Prius a moderate press produces an immediate jump in rpm, so the engine sounds busy before the car actually feels fast. Stealth reverses the relationship: light throttle stays quiet, medium throttle stays quiet, and only heavy throttle brings the engine into the party. That is exactly how premium turbocharged cars are tuned, the cabin stays calm until you make it clear you want to go. The side effect is that the car feels more powerful, not less, because the engine arriving late reads as a surge rather than constant background strain.

Toyota could frame it as two personalities on one button. Below about half pedal it is Stealth, EV-like refinement, the battery absorbing the transients so the engine never chases your foot. Above half pedal it becomes Prime Boost, the full hybrid system awake, rpm climbing fast with maximum battery assist. It will never be identical to a Tesla, because there are still engine starts and stops, but for commuting, traffic, suburban roads, and moderate highway driving it gets surprisingly close to the EV sensation while keeping the gas tank.

### The button you already have

Nothing new to learn. The Prime already has the two buttons this needs.

* **[Auto EV/HV]** is the automatic mode, the one most people leave the car in, where you let the car decide. This is where Stealth Comfort lives. The new logic becomes the brain behind this button, so the driver gets the quiet cruiser by default without reading a word of the manual.
* **[HV/EV]** is the manual override, and it does not change at all. Force HV to lock and preserve your charge for later. Force EV to draw the pack down to minimum on the last leg home, so you arrive nearly empty and can take a full 44 mile charge overnight on cheap power.

```
[ Battery SOC ]
      |
      |-- [HV/EV] forced EV ----> drain pack to minimum (arrive empty, take full 44 mi charge)
      |-- [HV/EV] forced HV ----> lock and preserve current SOC for later
      |
      +-- [Auto EV/HV] ---------> STEALTH COMFORT
                                     |
                                     |-- low/mid speed, light pedal --> series-priority quiet cruise
                                     +-- high speed / hard pedal     --> seamless parallel blend
```

By putting the logic behind a button drivers already expect to mean "let the car optimize itself," you hand them a luxury EV experience by default and add zero user confusion.

## Part 2 — Why It Works

### Low rpm is not the enemy, light load is

Everybody assumes low rpm means low efficiency. It does not. What matters is load fraction, how hard the engine is working relative to what it could do at that speed. A lightly loaded engine bleeds most of its fuel to friction and pumping no matter the rpm. Load it up and even low rpm lands on its best efficiency island. The M20A-FXS has a wide island thanks to its 14:1 compression and electric variable valve timing, which is why this works as well as it does.

So the trick is not to baby the engine. It is to run it low and loaded, using it as a generator that carries the car and tops the battery at the same time. In a roughly 1,600 rpm window it sits near 60% load and lands around 39% thermal efficiency. Peak for this engine is about 41%, so you are giving up almost nothing.

The question worth checking was whether buying this refinement costs fuel. The model was run repeatedly to find out: a 300-mile day with no wall charging at all, the battery just buffering, across several city and highway mixes, plus an rpm sweep. The result comes back in a narrow band every time, from no measurable loss to about 4% better than stock, because the steadier engine policy avoids short-cycling and hunting. So the NVH strategy does not appear to cost economy, and may modestly help. Treat that as upside to confirm rather than the headline, because it is modeled rather than dyno-measured. The certified label does not move regardless, since the mode is opt-in.

### The 13.6 kWh macro-buffer

This is where the Prime beats the cars that already run series-hybrid. Nissan e-Power and Honda i-MMD carry tiny buffers, around 1.5 to 2 kWh. When the driver suddenly asks for a big slug of power, the engine has to jump up right now, because the little battery cannot bridge the gap. That is why those cars still surge their engine note under hard throttle.

The Prime carries 13.6 kWh and a pack that can dump well over 60 kW on demand. The engine can sit at a leisurely 1,600 rpm putting out a steady, efficient 20 kW. When you press in for traffic, the battery instantly covers the difference and you feel clean electric torque while the engine never changes its tone.

```
Driver demand: 45 kW
   |
   |-- Engine locked at 1600 rpm  -- steady 20 kW --+
   |                                                v
   +-- 13.6 kWh battery buffer  -- instant 25 kW --> MG2 --> 45 kW to wheels (quiet)
```

Instead of the engine mimicking your foot step for step, it simply experiences a shift in electrical load while its physical rpm stays a serene hum.

This is also why the idea is new rather than a missed opportunity nobody noticed. It was not possible on earlier Prius generations, and not because nobody tried. Their packs simply did not have the capacity or the discharge power to act as a macro-buffer, so the engine had to track the driver's foot because nothing else could cover the transients. The 13.6 kWh pack is the first one with enough throughput to let the engine sit still. The hardware finally caught up to the concept.

### The engine already cycles, this just governs how

It is worth being precise about the baseline. Stock HV does not hold the engine steady. It flips it on and off in indeterminate windows as demand rises and falls, closer to a light switch than a dimmer. That cycling exists for good reasons, efficiency, emissions, and catalyst management, but it is also the source of the switch-like character a driver notices. So Stealth Comfort is not fighting a stable baseline, and it does not pretend the engine is a static generator in all conditions.

What it changes is the on/off policy. A minimum dwell time once the engine starts, revised hysteresis so it does not chatter between states, and the 13.6 kWh buffer absorbing short demand swings so the engine does not need to flip as often. Engine shutoff is still permitted, but only in clearly safe windows where catalyst temperature and road load are both favorable. The unresolved question was never whether the engine is allowed to cycle, it already does. It is whether you can constrain that cycling without breaking emissions or thermal margins, and the dwell-and-hysteresis policy is exactly that constraint. What the driver feels is a steadier hum in place of a switch.

### The quiet window

The 2.0 Atkinson has two rough spots, not one. Past about 1,700 rpm it drones, the airborne noise everyone knows. Down around 1,400 to 1,500 rpm under heavy load it booms, a structural shake through the mounts that feels lugged. The target sits between them, roughly 1,550 to 1,650 rpm, with a slow sub-audible dither across the band so it never parks long enough to build a standing acoustic wave.

## Part 3 — The Engineering

### Transaxle kinematics, power recirculation, and the speed-linked rpm floor

The one hard physical limit lives inside the eCVT. On the 5th gen transaxle the engine sits on the planet carrier, MG1 on the sun gear, MG2 and the output on the ring, with a planetary ratio around 2.6. Their speeds are locked by the geometry. With ratio k near 2.6, the planetary relation reduces to MG1 speed = 3.6 x engine speed - 2.6 x ring speed. Ring speed rises with road speed, so pinning the engine low while the car speeds up forces MG1 to spin backward harder and harder.

| Vehicle speed | Ring / MG2 | Engine if pinned | MG1 / sun | Status |
|---|---|---|---|---|
| 35 mph | ~2,500 rpm | 1,600 rpm | -740 rpm | clean series zone |
| 55 mph | ~4,000 rpm | 1,600 rpm | -4,640 rpm | edge of efficient EMF range |
| 65 mph | ~4,700 rpm | 1,600 rpm | -6,460 rpm | recirculation losses climbing |
| 75 mph | ~5,450 rpm | 1,600 rpm | -8,410 rpm | unacceptable, must blend |

The structural redline on MG1 is well north of 17,000 rpm, so rotor structural failure is not the issue. Instead, two operating limits intervene first:

1. Spinning the permanent magnet rotor backward rapidly generates heavy back-EMF whose current loads the inverter thermally.
2. The device enters a mechanical-electrical-mechanical power loop called **power recirculation**. MG1 generates electrical power that MG2 must immediately consume to balance the planet carrier against pavement torque. The conversion penalty for this recirculating energy climbs steeply as the speed split widens.

A hard 65 mph cliff is the wrong fix. The correct calibration uses a **speed-linked rpm floor**. Hold ~1,600 rpm up to 45 mph, then let the engine target rise gently along a minimum-loss curve to ~1,900 rpm at 60 mph and ~2,100 rpm by 70 mph. Raising the engine to 2,100 rpm at 70 mph cuts MG1's reverse rotation speed by roughly 1,800 rpm, keeping it cleanly inside its efficient electrical range and mitigating power recirculation. The driver does not notice the extra revs because road and wind noise dominate the cabin at those speeds. Above this practical ceiling, the system drops into a normal parallel blend.

### Transient power delivery

Because the engine is locked to MG1 and MG2 through the planetary set, holding it steady turns the electrical side of the transaxle into a dynamic scale. Here is how the device handles a real transient, pacing traffic on a rolling hill, with the engine pinned and generating a steady 20 kW.

| Driver input | Road | MG1 behavior (sun) | Battery action | Outcome |
|---|---|---|---|---|
| Cruising, ~15 kW | flat | brakes the engine to hold target rpm | absorbs spare ~5 kW to recharge | pure quiet, engine load ~45% |
| Tip-in, ~35 kW | gentle incline | torque command drops toward zero or assists | instantly discharges ~15 kW to MG2 | engine note never changes, battery eats the spike |
| Floor it, ~75 kW | passing a truck | commands maximum torque split | dumps 60+ kW to MG2 | ECU breaks out of Stealth, engine flashes to ~4500 rpm for parallel push |

In a small-buffer hybrid the engine would have to mimic the driver's foot step for step. With the macro-buffer it just sees a change in electrical load while its rpm stays flat, right up until you genuinely floor it.

### The boom node is felt, not just heard

A four-cylinder engine fires twice per revolution, so its second-order vibration at 1,600 rpm sits at exactly 53.3 Hz. That frequency lands squarely in the cabin boom band for a compact unibody platform. The sub-audible dither smears that energy across a 51 to 55 Hz window to prevent standing acoustic waves, but smearing alone does not stop the kinetic energy from transferring through the engine mounts into the floorboards as a low hum.

The real fix attacks the source by capping engine load near 45% inside this resonance zone, softening the cylinder firing pulses. The 13.6 kWh buffer covers the torque shortfall through MG2. This is done feed-forward via a pre-mapped resonance table in firmware, requiring no cabin microphone or accelerometers. The one honest cost is a couple points of thermal efficiency while load is capped, so it is applied only inside the resonance zone, not everywhere.

#### Crossing the cap without a shudder

A naive implementation of this load cap creates a control trap due to an **actuator latency mismatch** at the boundary. The engine sheds torque via manifold airflow over roughly 100 to 150 ms, while MG2 responds through the inverter in about 10 ms. A driver hovering their foot right at the threshold would cause the ECU to toggle the cap on and off rapidly, and the two actuators, fighting out of phase, would set up a longitudinal driveline oscillation.

Three standard calibration strategies remove this shudder entirely:

1. **Schmitt trigger hysteresis.** Engage the load cap at 47% load and do not release it until the raw request falls below 43%, preventing chatter around a single point.
2. **Slew-rate limiting.** Apply a smooth ramp to the engine load command so the mechanical transition matches the physical constraints of the intake tract.
3. **MG2 active backfill.** Compute the exact engine torque error and command MG2 to backfill the deficit through the planetary ratio. Because the inverter updates far faster than the engine breathes, the electrical side cleanly masks the mechanical lag, keeping pedal response perfectly linear.

This backfill operates well within the inverter's existing thermal envelope, covering a load delta of only a handful of kW, far less than MG2 already handles launching the car or grabbing regen. If the inverter does hit a thermal limit and derate during an extended climb, it clamps MG2 current. The backfill then falls short, allowing the 53 Hz boom to return to the cabin, acting as a natural physical warning to the driver that the electrical buffer is spent.

### Catalyst thermal management and the cold-start trap

Emissions compliance is the true validation hurdle for this mode, and the primary enemy is the cold-start FTP-75 cycle.

If a driver leaves their driveway in Stealth mode with the battery at 60% SOC, the vehicle defaults to EV operation. The engine stays off and the catalyst stays stone cold. If the driver then executes a heavy highway merge, the engine fires under load against a cold cat, producing a hydrocarbon slug that violates SULEV30 limits. A reactive 15-second spark-retard light-off run during that merge creates a torque hole that can drag pack voltage toward its low-voltage cutoff.

The solution is a **predictive thermal window**. When the vehicle operates in Auto mode, the ECU monitors state-of-charge math, navigation data, route history, and runs of sharp pedal transients. If these inputs indicate the engine will be needed within the next few minutes, the ECU initiates a quiet, fully decoupled catalyst warm-up phase at ~1,200 rpm with near-zero output load. The catalyst bed reaches its ~400 C light-off temperature silently before a high-load generator demand ever occurs. Once lit, steady loaded operation maintains catalyst temperature far better than stock short-cycling logic.

### The revised ECU logic loop

The power management ECU executes this control loop every cycle within the Auto button path. If any operating boundary is crossed, it gracefully exits the quiet target to protect the hardware and returns to Stealth logic as soon as conditions clear.

```
        [ Auto EV/HV selected, Stealth Comfort active ]
                          |
        [ Read: road speed V, pedal a, cat temp T, MG1 rpm ]
                          |
              [ Boundary conditions ]
              V within speed-linked rpm-floor range ?
              pedal a < 50% ?
              MG1 inside efficient EMF range (no recirculation) ?
                          |
            +-------------+-------------+
           YES                         NO
            |                           |
   [ Thermal + cabin check ]   [ Protective fallback ]
   cat temp > 380 C ?          - scale engine rpm up with road speed
   in 53 Hz boom zone ?        - parallel blend if past the ceiling
            |                  - free rev on hard throttle
     +------+------+           - predictive / 15 s cat heat run if cold
    OK            NO
     |             |
     v             v
 [ Series      [ Corrective ]
   priority ]   - run predictive cat warm-up
 lock to        - cap load ~45% to kill 53 Hz floor hum,
 speed-linked     buffering the torque gap with MG2
 rpm, dithered,
 battery buffers
 all transients
```

If the driver outruns the battery capacity, the pack drifts to its lower state-of-charge floor and the vehicle reverts to normal, stock HV hybrid behavior. No limp mode, no warning light, no service call.

### Calibration spec sheet

* **Feature:** Stealth Comfort mode, the new logic behind the existing [Auto EV/HV] button. The manual [HV/EV] button remains unmodified to force standard EV or charge-sustaining behavior.
* **Operating envelope:** active up to 50% accelerator pedal input. Blends into a standard parallel mechanical path if pedal input exceeds 50% or if speed exceeds the dynamic ceiling.
* **Engine target schedule:** speed-linked rpm floor holding ~1,600 rpm up to 45 mph, scaling smoothly to ~2,100 rpm by 70 mph, with a slow +/-50 rpm sub-audible dither across the band to break up standing acoustic waves.
* **NVH mitigation:** feed-forward engine load cap near 45% inside the 53.3 Hz second-order boom zone, controlled via a Schmitt-trigger hysteresis band (engage 47%, release 43%) paired with a slew-rate-limited load command and synchronous MG2 torque backfill.
* **Battery state-of-charge band:** target hold between 30% and 70% SOC, using the 13.6 kWh pack as a high-throughput macro-buffer for all transient smoothing.
* **Engine state policy:** a minimum dwell time once the engine starts plus revised on/off hysteresis to suppress switch-like cycling, with the buffer covering short transients so the engine changes state less often. Shutoff permitted only when catalyst temperature and road load are both favorable.
* **Catalyst thermal strategy:** predictive thermal window, pre-warming the bed to ~400 C at ~1,200 rpm decoupled before predicted engine loading, backed by a hard 380 C floor that triggers a spark-retard heat run if reached during extended coasting.
* **Hardware requirements:** none. Executed entirely via a firmware update to the power management control ECU. Active mounts or ANC only if cabin boom proves stubborn beyond the load cap.

### Open validation items

Across this analysis the hurdles moved from "can it work" to "how much validation," which is where serious production concepts are supposed to end up. The remaining questions are not physics, they are testing, and naming them honestly is part of the pitch:

* **Battery throughput durability.** Using the pack as a macro-buffer raises lifetime energy throughput. Probably not a problem given the pack's size and the modest depth of the cycling, but Toyota would want multi-year durability data before committing.
* **Thermal under sustained bursts.** Repeated 20 to 30 kW discharge in hot weather has to be validated against battery and inverter temperature limits, with the derate behavior characterized rather than assumed.
* **Emissions certification across corner cases.** The predictive catalyst strategy is sound in principle, but proving it across the full corner-case matrix of the cert cycles is expensive and slow.
* **Customer consistency at the buffer floor.** When SOC reaches its lower limit the experience reverts to stock HV. That handoff has to be transparent and predictable, because Toyota is obsessive about consistent behavior, and an experience that quietly changes character is exactly what it guards against.

None of these are reasons the concept cannot work. They are the cost of proving that it does. And they are all manufacturer-side costs, calibration effort, validation time, throughput and thermal accounting. The customer-visible tradeoffs, by contrast, appear minimal: peak power, EV-mode function, HV charge preservation, charging behavior, and range all remain unchanged, in exchange for less engine hunting, less drone, and a more EV-like response. That is the honest scope of this document: a physics-clean concept whose path to production runs through validation and a willingness to trade a fraction of absolute efficiency for a more premium experience, not through any new invention.

### Proposed validation plan

Those open items resolve through a standard mixed regime, powertrain hardware-in-the-loop plus environmental chassis-dyno testing. Three tests carry most of the risk. Note that the numbers in the table below are acceptance gates and design targets the calibration would be held to, not predicted results. The baselines are representative reference values, and the target column states the goal to confirm on the rig, not a measurement that has been taken.

**Test 1, FTP-75 cold-start and catalyst thermal.** Objective: verify the 1,200 rpm decoupled pre-warm does not compromise emissions. Setup: environmental chamber with a 20 C thermal soak. Procedure: run FTP-75 Phase 1, commanding the engine to 1,200 rpm decoupled at minimum load on the predictive trigger. Measure catalyst bed temperature and raw tailpipe HC and NOx over the first 120 seconds.

**Test 2, high-speed eCVT recirculation and thermal stress.** Objective: confirm the speed-linked floor keeps MG1 out of the back-EMF recirculation zone. Setup: powertrain dyno coupled to the output shafts with simulated aero drag. Procedure: hold 75 mph at 3% grade and compare stock behavior against the speed-linked floor. Measure MG1 speed, phase current, inverter IGBT temperature, and net battery throughput.

**Test 3, transient anti-shudder tip-in.** Objective: confirm the MG2 backfill masks the engine's manifold lag during an EV-to-hybrid handoff. Setup: HIL rig with crankshaft and MG2 torque sensors. Procedure: rapid tip-in from 10% to 60% demand at 45 mph. Measure torsional vibration at the output shafts with high-speed encoders.

| Test | Metric | Reference baseline | Design target | Pass gate |
|---|---|---|---|---|
| Cold-start emissions | time to cat light-off (300 C) | ~45 s | under 30 s | within 35 s, no HC breakthrough |
|  | Phase 1 cumulative NOx | representative | reduced | meets LEV3 / SULEV30 |
| eCVT recirculation | MG1 minimum speed | ~-6,200 rpm | ~-4,100 rpm | above -4,500 rpm |
|  | inverter core temp rise | representative | sharply reduced | stabilize below 95 C |
| Anti-shudder tip-in | output-shaft torsional vibration | ~+/-4.5 Nm | ~+/-0.8 Nm | under +/-1.2 Nm |
|  | torque-delivery latency | ~120 ms (engine alone) | ~12 ms (MG2 masked) | within 15 ms |

The expected direction, the one the speed-linked floor was designed for, is that keeping MG1 out of the recirculation zone sharply cuts inverter thermal stress during highway cruising, buying headroom that offsets the modest rise in battery throughput from low-speed smoothing. The magnitude of that win is exactly what Test 2 exists to quantify, and it should not be asserted before it is measured.

## Conclusion

This strategy leverages the physical overhead of the 5th-generation Prius Prime's 13.6 kWh battery pack and transaxle to shift calibration priorities from marginal fuel-consumption gains to premium cabin NVH. By managing transaxle kinematics through a speed-linked floor, isolating structural resonance via a rate-limited load cap with active electric backfill, and managing emissions through a predictive thermal window, the calibration remains entirely production-feasible. It delivers a premium electric-vehicle driving experience using the existing mass-market component catalog, requiring only optimization within the power management software architecture.

The conceptual leap underneath all of it is simple. The 13.6 kWh pack stops being merely an energy store for EV range and becomes a real-time NVH actuator, a torque reservoir that smooths engine behavior, masks actuator latency, suppresses transient vibration, reduces engine cycling, and reshapes the subjective character of the car. Most refinement is bought with hardware, acoustic glass, active mounts, extra insulation. This buys it with energy-management strategy instead. It is still firmware only, with no new hardware, but it is genuine calibration work, not a weekend flash.

There is also a clean precedent for the single judgment call this asks Toyota to make, trading a sliver of efficiency for desirability. The lineup already does exactly that at the wheel wells. The XSE's 19-inch wheels drop the Prime from 52 to 48 mpg combined, an 8% hit, purely for how the car looks, and because trims certify separately the headline 52 mpg survives on the SE. If Toyota will spend 4 mpg on appearance, an opt-in software mode that trades a far smaller fraction, possibly none, for an EV-like cabin is the same trade made for a better reason. The argument that efficiency must be protected at all costs is one the product lineup has already answered for itself.

---

*Figures are from an independent first-principles model, road load plus a friction and pumping efficiency map anchored to EPA dynamometer data on Toyota's Dynamic Force Atkinson engines, with kinematics from the 5th gen planetary geometry. The economy range reflects repeated model runs across multiple drive scenarios, not a dyno measurement. This is an engineering estimate intended to justify a prototype and a cert-cycle measurement, not a finished calibration, but the shape and the conclusions hold.*

