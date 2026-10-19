Project 2 Traffic Light Controller
Name: Vichayasan  Sreshthaputra   Student ID: 6872087421
1 Objective and system overview
This project implements a three-mode traffic light controller in Simulink and Stateflow and connects its outputs to an Arduino Nano 33 BLE. The controller uses conditional transitions and timed events to produce Normal, Emergency, and Maintenance light sequences. Separate simulation runs demonstrate the required sequence in each mode, and onboard operation was observed to follow the same behavior.
The assignment specifies the following sequences [1]. Each mode begins with the first lighting condition listed in Table 1. Only the indicated lights are on.
Table 1. Operating modes and required timing.
Mode input	Mode	Repeating sequence
0	Normal	Green 5 s → Yellow 2 s → Red 5 s
1	Emergency	Red 1 s → Yellow 1 s
2	Maintenance	All on 1 s → All off 1 s
 
Figure 1. Top-level Simulink model. The mode input enters the Stateflow chart; green, red, and yellow outputs feed the scope, dashboard indicators, and Arduino output blocks.
The selector changes the Control Signal supplied to the chart as mode. The chart produces three binary output commands: green, red, and yellow. A value of 1 turns the corresponding LED on, while 0 turns it off. The scope displays these commands over time. The Arduino blocks connect the same commands to D10, D12, and D11, respectively, linking the simulation logic to the external LEDs.
 
2 Stateflow chart
 
Figure 2. Six-state controller with mode-dependent guards and output assignments on transitions. Normal uses Green, Yellow, and Red; Emergency shares Yellow and Red; Maintenance uses maintenanceOn and maintenanceOff. Guard selects the initial mode.
The chart uses six states at one level, with no nested refinement states. Sharing Yellow and Red between Normal and Emergency avoids duplicating the same output conditions. The mode conditions distinguish the routes and durations. Guard is an ordinary state used for startup selection; the Boolean conditions in square brackets are transition guards. These are different uses of the word guard.
 
3 Transition logic and timing
Lee and Seshia describe a transition guard as a Boolean expression that enables a transition when it evaluates to true. Their discussion of Mealy machines associates output production with transitions [2, Section 3.3.1 and p. 58]. In this implementation, the labels combine a mode condition, a timed condition where needed, and assignments to the three output variables. This explains the transition-based output design shown in Figure 2.
At initialization, the default transition checks ~isnan(mode) and assigns green = 0, yellow = 0, and red = 0 before entering Guard. This condition excludes a not-a-number input; it does not restrict the input to the three supported mode values. Guard then selects Green for mode = 0, Red for mode = 1, or maintenanceOn for mode = 2 and sets the corresponding outputs.
For example, the Green-to-Yellow transition is labelled [mode == 0 && after(5,sec)]/{green=0; yellow=1; red=0;}. The equality test selects Normal mode, && requires both conditions to hold, and after(5,sec) becomes true after the source state has been active for the specified duration. The action after the slash updates all three output commands. Between transitions, those commands retain their assigned values.
Table 2. Timed transitions in the implemented chart. Output tuples are ordered (green, red, yellow); 1 means on and 0 means off.
Mode	Transition	Delay	Output after transition
Normal	Green → Yellow	5 s	(0, 0, 1)
Normal	Yellow → Red	2 s	(0, 1, 0)
Normal	Red → Green	5 s	(1, 0, 0)
Emergency	Red → Yellow	1 s	(0, 0, 1)
Emergency	Yellow → Red	1 s	(0, 1, 0)
Maintenance	maintenanceOn → maintenanceOff	1 s	(0, 0, 0)
Maintenance	maintenanceOff → maintenanceOn	1 s	(1, 1, 1)
The configured durations give a 12 s Normal cycle and 2 s cycles for Emergency and Maintenance. The displayed waveforms are consistent with these nominal durations. Exact event times depend on when the chart executes, so the screenshots support sequence and approximate timing checks rather than a numerical timing-error measurement.
For each test, the desired mode was selected before starting a separate run. The chart screenshot does not show transitions back to Guard from the operating states. Therefore, the tests establish operation with a fixed mode; they do not establish unrestricted switching among modes during a continuous run.

 4 Hardware implementation
Figure 3 shows the external LED connections to the Arduino Nano 33 BLE. The red LED is connected to D12, the yellow LED to D11, and the green LED to D10. Each branch places a 220 Ω resistor in series with the LED, with its cathode connected to the common ground. The diagram identifies the board as USB powered with 3.3 V logic.
 
Figure 3. Arduino Nano 33 BLE wiring. D12 drives red, D11 drives yellow, and D10 drives green. Each LED branch includes a 220 Ω series resistor and returns to GND.
An asserted output drives the corresponding LED branch, while a low output turns it off. The series resistors limit current through the LEDs. The schematic and the Arduino blocks in Figure 1 use the same pin mapping, so the red, yellow, and green commands correspond to the intended physical indicators.
The controller was also run on the board, where the LED sequences were observed to match the simulation for the three modes. This is a functional observation; no separate measured hardware timing error is reported. Links to the demonstration video and supporting screenshots can be entered on the final page.

 5 Normal mode results
For the Normal-mode test, mode was set to 0 before starting the run. Figure 4 displays green at the top, red in the middle, and yellow at the bottom over a 60 s interval. The horizontal axis is time in seconds, and the vertical axis is the binary output command, with 1 meaning on and 0 meaning off.
 
Figure 4. Normal mode over 60 s. Green remains on for approximately 5 s, yellow for 2 s, and red for 5 s before the sequence repeats. The trace order is green, red, yellow; the operating sequence is green, yellow, red.
The green pulse is followed by the shorter yellow pulse, after which red remains high until the next green interval. The approximately 12 s repetition is consistent with the sum 5 + 2 + 5 = 12 s. The 60 s display therefore spans approximately five cycles. The traces show one active light at a time during the operating sequence.
This behavior follows directly from the three Normal-mode transitions in Table 2. Each transition both changes the active state and sets all output commands, turning the next light on while turning the other two off. The brief startup interval visible near the origin belongs to initialization and should not be used to claim an exact first-edge timestamp.

 6 Emergency mode results
For the Emergency-mode test, mode was set to 1 before starting a separate run. Guard selects Red as the initial operating state. Figure 5 shows that green remains off while red and yellow alternate throughout the displayed 60 s interval.
 
Figure 5. Emergency mode over 60 s. Green remains at 0. Red and yellow alternate with approximately 1 s on intervals, giving each light a nominal 2 s repetition period. Axes show time in seconds and binary output level.
The red and yellow traces are complementary during the repeating sequence: a falling edge of one corresponds to a rising edge of the other. Each light is on for approximately 1 s and off for approximately 1 s. Thus, the two transitions form a 2 s cycle, consistent with the assignment.
Emergency mode shares the Red and Yellow states with Normal mode but uses different guarded transitions. With mode = 1, the 1 s Red-to-Yellow and Yellow-to-Red routes are enabled. The Normal-mode route from Red to Green requires mode = 0, so green remains off in this test.

 7 Maintenance mode results
For the Maintenance-mode test, mode was set to 2 before starting a separate run. Guard selects maintenanceOn and sets all three outputs to 1. Figure 6 shows the three lights switching together throughout the displayed interval.
 
Figure 6. Maintenance mode over 60 s. Green, red, and yellow rise and fall together, with approximately 1 s all on followed by 1 s all off. Axes show time in seconds and binary output level.
The aligned rising and falling edges show that the commands are synchronized. After approximately 1 s in maintenanceOn, the controller enters maintenanceOff and assigns zero to every output. After another 1 s, it returns to maintenanceOn and assigns one to every output. This produces the required 2 s repeating cycle.
The simultaneous activation of all three lights is intentional in Maintenance mode. It differs from the one-light-at-a-time behavior in Normal and Emergency operation and provides a direct visual check that all three LED channels can respond together.

 8 Discussion and conclusion
The three tests show the expected lighting patterns for their selected modes. Normal mode follows the 5 s green, 2 s yellow, and 5 s red sequence. Emergency mode alternates red and yellow every second while keeping green off. Maintenance mode turns all three lights on and off together at 1 s intervals. The observed onboard behavior matched these simulation sequences.
The design makes its decisions explicit through transition conditions and output assignments. The separate Guard state selects the startup route, while the timed guards determine when each subsequent change can occur. Reusing Red and Yellow reduces the state count, and the mode input selects the appropriate transition between them. All six states remain visible at the same chart level.
The evidence supports fixed-mode operation. Mode changes during an ongoing run, unsupported mode values, and precise timing accuracy were not established by these tests. In particular, the shown chart has no general return route to Guard. Any later requirement for unrestricted mode changes would need explicit transition handling and additional tests. These limits do not alter the three sequences demonstrated in Figures 4 to 6.
References
[1] E. A. Lee and S. A. Seshia, Introduction to Embedded Systems — A Cyber-Physical Systems Approach, second edition, version 2.3. MIT Press, 2017.

Demonstration Materials

https://github.com/user-attachments/assets/8fe8a689-8fd1-445c-afb9-9ad295989dc7






