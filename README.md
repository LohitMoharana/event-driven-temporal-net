# event-driven-temporal-net
> **Building a Brain: A Discrete Hardware Take on Spiking Neural Networks (SNN)**

## 🧠 Project Overview
Most AI today is "always on," burning power even when nothing is happening. `event-driven-temporal-net` is my attempt to flip that script. Instead of using code, I’ve built a hardware prototype of a **Spiking Neural Network (SNN)** using classic TTL logic. 

The goal was to create a system that only "thinks" when it receives a spike (a pulse). It’s an event-driven approach that prioritizes efficiency and mimics how biological neurons actually integrate information over time.

---

## 🏗️ How the System is Built
I structured the network as a **3-2-1 Hierarchical Funnel**. This layout is designed to handle both **Spatial** (where the signal comes from) and **Temporal** (when the signal arrives) summation.

### **The Layers**
1. **Input Layer (3 Neurons):** These act as the "eyes" of the system, receiving raw pulse data and starting the integration process.
2. **Hidden Layer (2 Neurons):** These units look for patterns. By using OR-gate logic, they sum up spikes from the input layer to see if a specific "feature" is present.
3. **Output Layer (1 Neuron):** This is the decision-maker. It only fires a final inference pulse when the combined signals from the previous layers hit a specific threshold.

---

## 📊 Neural Response & Logic Analysis
To verify the temporal summation of the network, I performed high-speed timing analysis using the XLA1 Logic Analyzer.

### **Temporal Integration Verification**
The waveform below demonstrates the "Integrate-and-Fire" mechanism. You can see the input spike train accumulating in the 74LS161 counter until the 74LS85 comparator triggers a global spike and an immediate asynchronous reset.

<img width="793" height="598" alt="Screenshot 2026-02-14 112625" src="https://github.com/user-attachments/assets/61bfed5e-3f58-4279-a437-dc69e8508d4a" />

*Caption: Spike propagation through the hierarchy. Note the output firing only after the cumulative threshold is reached.*

---

## 🖼️ Circuit Gallery

### **Asynchronous Reset Timing**
A close-up of the RC feedback loop shows a stable reset pulse of ~$1\mu s$. This ensures the "membrane potential" (counter value) returns to zero before the next input event, mimicking a biological refractory period.

### **The Neuron Core (Hierarchical Block)**
This is the fundamental building block of the network. It encapsulates the **74LS161** counter and **74LS85** comparator logic. By modularizing the neuron into a hierarchical block, I was able to scale the 3-2-1 architecture without cluttering the main workspace.

<img width="958" height="716" alt="image" src="https://github.com/user-attachments/assets/909f2c06-320d-40df-9b04-b3495b9e690a" />

*Internal logic of the LIF neuron showing the integration and thresholding stages.*

### **The Full 3-2-1 Hierarchy**
The complete network layout. You can see the spatial summation through the OR-gate layers leading to the final decision neuron.

<img width="1133" height="685" alt="image" src="https://github.com/user-attachments/assets/e8cd4fa6-2a8a-4667-aac2-86707d38360e" />

*The complete event-driven-temporal-net funnel in Multisim.*

---

## ⚙️ The "Neuron" Logic (Technical Specs)
I didn't use a processor here. Every "neuron" is a hardwired circuit realizing the **Leaky Integrate-and-Fire (LIF)** model:

* **Counting the Potential:** I used **74LS161** 4-bit counters to act as the neuron's membrane potential—basically keeping track of the "charge."
* **The Firing Point:** **74LS85** magnitude comparators monitor that count. Once it hits the threshold I've set, the neuron "fires."
* **The Reset (Refractory Period):** To mimic a real neuron's "cool down," I engineered a self-timed reset using an **RC feedback loop** and **74LS04** inverters. This creates a ~1us window where the neuron resets itself to zero after spiking.
* **Combining Signals:** **74LS32** OR gates handle the spatial summation between layers.

---

## 🚀 Why This Matters
* **Truly Event-Driven:** If there are no input spikes, the counters stay still. No wasted computation, no wasted power.
* **Parallel by Default:** Every single neuron in this hierarchy works at the same time. There’s no CPU bottleneck.
* **Stable at Speed:** I've validated this logic for pulse trains up to **10 kHz**, ensuring the temporal summation stays accurate even at high frequencies.
* **Scale-Ready:** By using Hierarchical Blocks in Multisim, the design is modular and ready to be ported to an FPGA.

---

## 🛠️ Dev Log: Solving the 10kHz Bottleneck
During the initial simulation phase, I hit a major wall. The system worked perfectly at low frequencies, but as soon as I pushed the input to **10kHz**, Multisim started throwing convergence errors, or the neurons would fail to reset properly.

**The Problem:** At high speeds, the default **RC time constant** in my reset loop was too slow. The neuron was still trying to "clear" itself while the next input spike was already arriving. This caused the counter to hang or miss pulses entirely.

**The Fix:** I had to fine-tune the feedback loop. By dropping the capacitor value and adjusting the simulation's **Time Step** settings, I narrowed the reset pulse to ~1us. This was just enough time to clear the **74LS161** without interfering with the next 10kHz clock edge. It was a great lesson in the reality of propagation delays, even in a simulated environment.

---

## 📈 Verification
Everything was designed and stress-tested in **NI Multisim 14.3**. 

I used the **XLA1 Logic Analyzer** to verify that the reset pulses were fast enough to clear the neurons without losing data. The final proof was a steady-state inference on the output layer, confirming the 3-2-1 funnel works as intended.

---

## 🗺️ What's Next?
* ✅ **Phase 1:** Multisim Logic Validation (Done)
* 🛠️ **Phase 2:** Moving to the breadboard with a 4-LDR sensor array (Currently working on).
* ⚡ **Phase 3:** Scaling the architecture onto a **Xilinx PYNQ-Z2 FPGA**.

---

## 🎓 About Me
I'm an Electronics and Communication Engineering student at **OUTR Bhubaneswar** (class of 2028), currently exploring the intersection of hardware architecture and neuromorphic computing.
