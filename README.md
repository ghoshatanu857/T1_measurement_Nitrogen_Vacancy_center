# Qauntum Control Experiments

We do various qunatum measurements with instrument automation.

## An Example

Here, we will demonstrate the general lifetime measuring procedure for systems with lifetimes ranging from approximately 100 ns to longer timescales.

### Prerequisites

For experimental application with the attached code, we will need :

* Avalanche photodiode (APD) - for counting incoming photons.
* [Swabian Pulse Streamer 8/2](https://www.swabianinstruments.com/pulse-streamer-8-2/) - for pulsing various devices.
* [NIDAQmx, USB-6361](https://www.ni.com/docs/en-US/bundle/pcie-pxie-usb-6361-specs/page/specs.html) - for counting edges of the incoming pulses from APD.

### Running the tests

The steps are as follows,  

1. Initially, we link the devices to the PC.
2. Three pulses are sent by the pulse streamer: two of the pulses are used to drive the *NIDAQmx* and the third one pulses a *laser*.

  ![Sequence for Lifetime measurement.png](https://github.com/ghoshatanu857/Experimental_Measurements/blob/155f85567a59afe85b9e9bb2bcd17ef469b44b2f/Sequence%20for%20Lifetime%20measurement.png)

4. Pulse to run the laser consists of repeated operation of two consecutive blocks - laser on and laser off. We excite the system with an appropriate wavelength of laser and in the free evolution time, the population relaxes to ground state with an exponential characteristics ($\sim e^{-\frac{t}{tau}}$).
5. After APD counts the photons, it transmits the signal to NIDAQmx (terminal **PFI3**), which counts the rising edges within the specified time range.
6. To select the collection window, two pulses are employed to trigger(terminal **PFI4**) and to do timing(terminal **PFI5**) of NIDAQmx.
7. Both trigger and timing pulses consist of two blocks - *reference* pulse block and *signal* pulse block.

```
# Laser_Initialization Sequence
def seqInit(*args):
  laser_block = [(delay2,0),(laser_on,1),(delay2,0)]
  seq_init = pulser.createSequence()
  seq_init.setDigital(laser_port,laser_block)
  condition_check(seq_init)
  return seq_init

# Free Evolution Sequence
def seqLifetime(*args):
  timing_read_on = read_on-triggerTimingDelay
  for t in range(steps-1): # neglecting the last step
    trigger_block = [(read_on,1),(delay1+timeRange[t],0),(read_on,1),(timeRange[-1]-timeRange[t]-delay1-2*read_on,0)]
    timing_block = [(timing_read_on,1),(delay1+timeRange[t]+triggerTimingDelay,0),
                     (timing_read_on,1),((timeRange[-1]-timeRange[t]-delay1-triggerTimingDelay-2*timing_read_on),0)]

    seq_evolution = pulser.createSequence()
    seq_evolution.setDigital(trigger_port, trigger_block)
    seq_evolution.setDigital(timing_port, timing_block)
    condition_check(seq_evolution)
    seq_lifetime = seqInit(*args) + seq_evolution
    yield seq_lifetime
```

8. Time-difference between falling edge of the laser pulse and rising edge of the signal pulse is varied in given number of **steps**. For each step, the sequence is streamed **samples** number of times.
9. After collecting counts for sample and step number of times, we do **averages** for better constrast.

Here is an example_plot of a real experiment measuring the lifetime of *Up Conversion Particles* (UCPs),  

![Lifetime Measurement of UCP.png](https://github.com/ghoshatanu857/Experimental_Measurements/blob/27416f52e7923b4738c5aa4e8e9bc91908f0a7cb/Lifetime%20Measurement%20of%20UCP.png)

## Authors

* **Prof. Siddharth Dhomkar** - [Department of Physics, IIT Madras](https://physics.iitm.ac.in/faculty-inner.php?fuid=120).
* **Mr. Atanu Ghosh** - Ph.D. student, IIT Madras.
* **Mr. Saksham Mahajan** - [Ph.D. student, University college of London](https://uk.linkedin.com/in/saksham-mahajan-9a1296144).

## Acknowledgments

* [Department of Physics, Indian Institute of Techonology, Madras](https://physics.iitm.ac.in/).
* [CQuICC - Center for Quantum Information, Communication and Computing](https://quantum.iitm.ac.in/).

