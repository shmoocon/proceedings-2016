# Reverse-Engineering Wireless SCADA Systems

# Abstract

In this work, we reverse-engineer the GE MDS 9710, a legacy RF modem widely deployed in SCADA installations, and show that these systems may be vulnerable to wireless attacks.

# Introduction

With the proliferation of inexpensive software-defined radio hardware, the era of RF “security” through obscurity is over. For example, tools exist to decode pager[^1] and GSM traffic[^2], ACARS[^3] and ADS-B[^4], DMR and P25 digital radios[^5], and even satellite data services[^6]. In this work, we examine the GE MDS 9710, a legacy RF modem still widely deployed in SCADA installations. This modem is particularly interesting because it uses a modulation scheme not typically seen, and as an old design, doesn’t provide any encryption or authentication. Since common SCADA protocols (such as ModBus[^7] and DNP3[^8]) assume a secure physical layer, this means that many SCADA installations are vulnerable to wireless attacks.

# Getting Started

We began our reverse-engineering process by gathering as much open information as we could. Construction standards documents provided by the utility company (as required by law) identified the radios they use as “MDS Radios.” Comparing GE MDS’s product offerings[^9] with frequencies and bandwidths found in the FCC ULS database for the utility company led us to focus on the MDS 9710 as the probable modem.

MDS produced a number of radios in this family, operating on different frequencies, in different modes, or with different hardware. While the FCC OET database had relatively little information on most of these variants, we found one model (FCC ID E5M9710-1[^10]) where MDS forgot to request confidentiality for the Theory of Operation and Block Diagram documents that are required for FCC certification.

In the Theory of Operation document, we found that the modem used continuous-phase frequency shift keying with “root-duobinary signal coding.” This coding makes the modulation scheme fairly unique, and as such, there is no support for it in common tools like GNU Radio. To explain “root-duobinary signal coding,” we must first briefly cover some introductory DSP topics and how modems normally operate.

# Digital Modulation

Modems convey information by *modulating* a carrier wave in some aspect. Modems may modulate the amplitude, frequency, or phase of the carrier wave. A simple example of this is known as “on-off keying” (OOK), sometimes known as “CW” or “continuous wave” transmission, where the amplitude of the carrier is either 0% or 100%. This seems like an easy way to transmit digital bits, but there are problems with this simple approach. One problem is that this modulation scheme uses an infinite amount of bandwidth. 

## Bandwidth-Limited Modulation

One approach to limiting the bandwidth of a modulated signal is to simply use a bandpass filter. However, this causes the modulating signal to be smeared across time, leaving you with a messy analog signal instead of a nice square wave, where the effect of each bit overlaps the effects of other bits. This is known as *inter-symbol interference* (ISI). However, specially-crafted filters can avoid ISI. 

### Impulse Responses

A filter can be described by its *impulse response*. This is the signal you get from feeding an instantaneous pulse (also known as a Dirac delta function) into a filter[^11]. The key to zero-ISI filters is that their impulse response at multiples of the symbol period is zero[^12]. This means that at every symbol period, the other bits do not have any effect on the signal, letting the receiver accurately determine which bit was sent. In most applications, a *raised cosine* filter is typically used as the zero-ISI filter. For optimal performance, half of this filtering is done by the sender, and half is done by the receiver, resulting in each side implementing a *root raised cosine* filter.

### Duobinary Coding

Duobinary coding is a slight variation on zero-ISI filters. Instead of requiring symbols have zero interference on others, duobinary coding uses *controlled* ISI[^12]. In this case, each symbol interferes with exactly one other symbol. This produces a tri-level signal: a positive sample indicates that the bit is a 1, a negative sample indicates that the bit is a zero, and a sample around zero indicates that the bit is the *opposite* of the previously-received bit. Again, half of the filtering is done by the sender, and half by the receiver. Applying this filter to our received signal dramatically improves our demodulator’s performance.

# Decoding the signal

The modem frequency-modulates the carrier with the root-duobinary signal. We can recover this signal in GNU Radio by using an FM demodulator followed by a root-duobinary FIR filter. The recovered binary signal clearly has a preamble but otherwise appears random. By using an MDS radio we purchased from eBay, we determined that the radio did not send enough bits to include a packet length or CRC. We also determined that repeated characters did not produce identical bits. This is the result of *scrambling*, which is used to “whiten” the spectrum of the transmitted signal. By searching the DSP firmware for instances of the XOR instruction, we discovered the structure of the linear feedback shift register (LFSR) used for scrambling. 

After descrambling the signal we noticed that each character used nine bits. If the most-significant bit is zero, the following eight bits are treated as a normal byte to pass through. If the most-significant bit is a one, then it is a special out-of-band character. One use of these OOB characters is to indicate the end of a packet.

We finally note that we expected the bit rate to be 9600 bits/second. However, by analyzing the DSP firmware, we found that the actual bit rate is about 9615 bits/second. Once we accounted for this difference, our GNU Radio block easily synced with received packets. By splitting the received byte stream along ModBus or DNP3 packet boundaries, we can simply use Wireshark to fully decode the received packets.

# Conclusions

We have made our GNU Radio demodulator available on GitHub[^13]. By observing plaintext ModBus and DNP3 packets, we can monitor aspects of SCADA networks. Furthermore, by relying on the FM capture effect, we may be able to inject our own commands. Obviously we cannot experimentally verify this, but it does not seem difficult.

Utilities have been aware that these links are insecure but may have argued that the risk of attack is low due to their relative obscurity. However, thanks to low-cost software-defined radio systems, over the past few years we’ve seen RF “security” through obscurity vanish for many systems. This work is yet another example of that trend, and one that utilities should take seriously.

# References

[^1] http://www.rtl-sdr.com/rtl-sdr-tutorial-pocsag-pager-decoding/
[^2] http://www.rtl-sdr.com/rtl-sdr-tutorial-analyzing-gsm-with-airprobe-and-wireshark/
[^3] http://www.rtl-sdr.com/rtl-sdr-radio-scanner-tutorial-receiving-airplane-data-with-acars/
[^4] http://www.rtl-sdr.com/adsb-aircraft-radar-with-rtl-sdr/
[^5] http://www.rtl-sdr.com/rtl-sdr-radio-scanner-tutorial-decoding-digital-voice-p25-with-dsd/
[^6] https://media.ccc.de/v/32c3-7154-iridium_update
[^7] http://modbus.org/docs/PI_MBUS_300.pdf
[^8] http://www.dnp.org/AboutUs/DNP3%20Primer%20Rev%20A.pdf
[^9] http://www.gegridsolutions.com/communications/DataAcquisition.htm
[^10] https://fcc.io/E5M9710-1
[^11] http://www.dspguide.com/ch6.htm
[^12] http://complextoreal.com/wp-content/uploads/2013/01/qpr.pdf
[^13] https://github.com/supersat/gr-scada/

#### Metadata

Tags: RF, DSP, SCADA, wireless, reverse engineering

**Primary Author Name**: Karl Koscher
**Primary Author Affiliation**: UC San Diego
**Primary Author Email**: supersat@cs.ucsd.edu
