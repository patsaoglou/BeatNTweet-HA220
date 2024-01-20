# Introduction
This repository is dedicated for the BNT HA-220 Headphone amplifier project. BNT stands for BeatNTweet, a neat company like name I was looking for to name my audio projects while HA-220 stands for Headphone Amplifier and 220 total of 2 watts of power per channel at 20V DC with a single supply. In this repository you are going to find the schematic of the project, a pcb report document and a folder with the manufacturing files of the pcb. We’re also going to talk about the initial requirements I had with the design, the reasons I choose to use the final circuit topology, the components I use and finaly some problems and mistakes I did during the design of the project.


![20230709_035918](https://github.com/patsaoglou/BNT-HA220/assets/93339707/521430a7-595b-4db0-a55e-d057e357d60c)


# Requirements
Starting of with the requirements, I wanted to create a headphone amplifier that it could be powered up from a single 20V power supply. The reasons I came up with the 1st requirement were that I wanted to create a low cost device that can be powered up from a 20v SMPS power supply, a very common device used to power up electronics that anybody could have laying around. So this will replace the dual supply rails created by a transformer, a few diodes and filter caps which add a significant cost. As cheap SMPS are known to create noisy power, it was important to add some linear regulation and some additional filter caps to step the voltage from 20V down to 17-17.5V so we can have a somewhat better quality voltage supply. The 2nd requirement I had was that wanted to implement a hybrid design that uses an op amp to do the voltage amplification and a discrete output stage to drive the headphones. Continuing, I wanted the amp to able to easily drive low impedance headphones across the audio range so I had to figure out the the ideal topology of the Class AB output stage. In the 3rd requirement, I wanted the amp to have high gain at 20db(10x) because although there are a few devices that output line level audio signal, there are still a lot of devices that output 0.3Vrms and without gain they can’t fully utilize the amplifier's power. Finaly, I wanted implement passive volume control that uses a pot to attenuate the input signal so after the pot, the signal needs to see high impedance for the pot resistance to dominate and to have linear attenuation.

To sum up:
1)	20V SMPS
2)	2W Peak Power
3)	Op amp/Class AB Output Stage
4)	20db Gain @20-20khz
5)	Ideal for low impedance headphones
6)	Passive volume control

# Circuit Topology
The amp being single supply, we first need to bias up the input of the op amp to half of the supply rails so I used 2 100k resistor voltage divider that give a dc offset to signal of around 8.5-9V, also form the high impedance that we wanted and the signal sees impedance of 50k Omhs. Wanting to block the dc offset getting into our source, the 330n capacitor at the input is used for that and also forms the main high pass filter with cut off 10hz@-3db. For the volume control I chose a 20k linear pot which at low volumes its resistance will dominate and in max volume the input signal sees approx. 15k Omhs which is more than enough. I initially wanted to use one Opamp IC for both of the channels but in order to implement the low pass filter at 20khz we need to buffer the input because if we place it at the input, the variable series resistance of the pot together with the high pass filter resistance and the capacitor will change the response at different volumes. So the output resistance of the omp amp(NE5532 has appox. 7.5) together with the 20 omhs resistor and a 330n Capacitor create a cut off at 24khz.

After, the signal is outputted from the second opamp into the output stage. The initial plan was to use 5k resistors, two diodes with a resistor at the middle that bias up the two transistors of the class AB output stage. Testing the circuit in my bench, I noticed that this topology was fine for high resistance loads but it struggled at low impedances and the signal swing was reduced but I also had thermal issues with the output transistors because of the active biasing. To solve the thermal issue I replace the diodes-resistor bias circuit with a Vbe multiplier that will be placed close to the output resistors and in case the temperature of the output increases, it will turn the bias down and we’re gonna have that thermal feedback. I configured the Vbe multiplier at 1.3V just in the point the output stage starts to conduct less than 10mA. The signal is put from the opamp at the collector because at that point we have symmetrical clipping and therefore more swing than putting it at the emitter. As for the load problem, the current gain of the output stage was not enough to drive low impedance load so i had to implement a driver stage. I first went ahead with a Darlington configuration but although I solve the load problem the addition another transistor to the signal path reduced the swing. Having 17V available, the almost 4V loses of the opamp, the 3V loses from the Darlington pairs and few millivolts drop from the output resistors leave around 9Vpk-pk for the signal to swing which is less that 1W per channel even on low impedance loads. That’s why at the end I opted to use the Sziklai - complementary feedback pair to get that 1.5V pk-pk swing, reduced by the Darlington configuration. Because the output signal has that DC offset, a coupling cap is needed to get the final signal which also acts as a headphone protection in case one of the output transistors shorts and we have dc at the output. Note that this capacitor has to be big enough because together with load forms a high pass filter, and if is not, it will alter the bass response of the amp at the wide variety of the different headphone impedances. Finaly, I applied global feedback to get the required gain which also reduces distortions created by the output stage.


![SCHEM](https://github.com/patsaoglou/BNT-HA220/assets/93339707/89a71c8f-4994-462e-b7c6-5915c49a8e2b)


# Simulation
In the simulations I used a resistor as a load to do the measurement the lowest being at 16Omhs, but as you know speakers are dynamic devices and have different impedances at different frequencies.
Results(0.6V 1K SINE, 16Omhs):
Max Voltage: 11Vpk-pk


![TRAN](https://github.com/patsaoglou/BNT-HA220/assets/93339707/83ba9558-997d-49e8-908a-d9e40351cd40)


Peak Power: 2W per Channel
Gain: 20.8
Frequency response: 24-24khz (@-3db of max Gain)


![Response](https://github.com/patsaoglou/BNT-HA220/assets/93339707/2eca1079-0c3d-4226-ada3-0240c676b8ad)


# Board Implementation

At the start, I did all of the development at the breadboard but I didn’t save any pics. For the pcb I designed, I wanted it to be 5x10 so I could panelize the board and take advantage of all the 10x10 pcb board size that JLCPCB offers. At first I had no idea of how to fit all the components doing nice layouting of the board but at the end I did a decent job. So I had 5 boards that contained 2 amp boards so total of 10 boards for 0.4 Euros a piece if we include the cost of shipping so not bad at all.


![PANEL](https://github.com/patsaoglou/BNT-HA220/assets/93339707/0ccf8cea-9485-4896-8e12-d6d884053422)


One BIG mistake I did in Altium was that I putted the upper pair the opposite way and as I assembled the board and tested it wasn’t getting the figures I was getting from the breadboard and I was wondering why. After 3 hours in 2 days desoldering and soldering components cause I thought that they were faulty I randomly checked the schematic at Altium and realized the stupid mistake I did. So I cut 2 traces and used small wire to do the connections and everything worked as intended.

![20230709_040048](https://github.com/patsaoglou/BNT-HA220/assets/93339707/c6a2b58d-1c3b-45b0-af04-5ac7c58550f9)

For components, I used a barrel fuse and a diode for reverse polarity and input protection, a LM317 regulator, mkt caps for filtering, 2 ne5532 op amp ICs, 2 bc337 and 1  bc327 for the driver stage, 1 TIP31/32 pair for the output transistors. For the transistors I took advice from existing threads on the diyAudio forum and although the bc337/ bc327 are widely used for audio, the TIP31/32 are not considered for audio use because of their limited Bandwidth at 3mhz which does not mean that they are going to be bad for our application. If you want, you can use Higher Bandwidth BJTs like the MJE15030/ MJE15031 which are widely used for audio at driver stages and have Bandwidth of 30Mhz but are more expensive. An other maybe better and cheaper option would be the BD140/BD139 but these are not compatible with our design.

![20230709_035918](https://github.com/patsaoglou/BNT-HA220/assets/93339707/6bffaa9e-0634-4b6e-9e80-904adffb40df)

# Measurements
Doing the measurements at the final board, they were almost the same with the simulation, 11Vpk-pk with a 1k sinewave and loaded with the audio technica ath-m50x headphones which are 32Omhs.

![20230709_041011](https://github.com/patsaoglou/BNT-HA220/assets/93339707/6242018f-cd3a-4147-ac48-f30af63168c2)


With normal use the amp is stable and with full power at 16Omhs it is suggested to put a heatsink because the transistors are dissipating more than 1W which is the dissipition in normal use with my headphones!
The sound was nice, flat, I could hear some hiss when the pot was turned all the way up, but that propably has to do with the breadboard headers and the breadboard cables I used to input the signal. It is deafening to hear at max volume. As a design is quite overkill for headphones but why not…😊


![20230709_041819](https://github.com/patsaoglou/BNT-HA220/assets/93339707/624548d9-a6b0-4ef2-a3b6-abab96f2546c)


# Changes and improvements
I would like to add some additional filtering ideally on the feedback path to make the response slope more instant at the not desired frequencies and maybe reduce the high audio frequencies because it seems a little much for my ears but it can also be the headphones used. Note that these are my own preferences and each of us has different taste of sound. 
Anyway, that was the project which I hope you found it interesting and I am definitely going to do more in the future. If I made any mistakes excuse me, let me know and if you have any suggestions for improvements do not hesitate to contact me!

Thank you

And big thanks to https://www.diyaudio.com/ which has a variety of audio related material!

