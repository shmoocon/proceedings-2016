# Making Milware: An Interdisciplinary Tryst

## Abstract

How can political and computer science get together to make something beautiful? The pervasive development and deployment of malicious software by states presents a new challenge for the information security and policy communities. The difference between state and non-state authored code is typically described in vague terms of sophistication, contributing to the inaccurate confirmation bias of many that states simply 'do it better.' This project tries to describe the architectural and behavioral features of state authored code which differ from that written by non-state actors and to highlight the implications of that difference.

## Introduction

In the last half-decade, governments have become some of the most sophisticated proliferators of malicious software. What has changed isn't so much the behavior of states as the scope of our understanding of their activities. Milware is state authored malicious software. Its aims are strategic, not financial; its technical complexity substantial and tactics, occasionally, novel. This term 'milware' is a way to capture the most advanced capabilities of states like Israel, Russia, the U.S., and China. It doesn't describe code used on the battlefield or make a claim that all state code is created equal. Malware is the code of non-state actors and individuals, which runs the gamut from terribly capable to just plain terrible. While malware is largely designed to get scattershot access to systems and execute anywhere it finds a foothold, milware has a narrow target set and appears to prioritize access over effects. Even where data exfiltration and not destructive effects are the objective, milware has specific non-financially motivated targets and tends to be harmed by widespread propagation.

## Why Are We Doing This?

For policymakers, identifying state authored code can help to develop a more robust suite of attribution techniques and may highlight a growing divergence in the evolution of malicious software being authored by these two groups. For the information security community, understanding characteristics far more common in state authored code, like systematic compromise of trust infrastructure \(code-signing processes from vendors or Certificate Authorities\) may contribute to greater awareness and more rapid recognition of new compromise techniques and infection vectors. Firms may use the information to make different insurance claims or solicit help from law enforcement bodies where it might not otherwise happen. The academic community can benefit from this work as it served to provide a model for empirically sound research across multiple disciplines.

## How to Differentiate State and Non-State Code?

Differentiating between sophisticated and unsophisticated samples is not a new area of work in the information security community; static and dynamic analysis techniques have been in use and evolving for several decades. [^4, ^5] Existing techniques tend to emphasize forensic attributes like attempts to 'fingerprint' developers based on their authorship, matching language setting and compiled time stamps to potential threat actors' locations, and selection of targets. [^6, ^7, ^8] Some of these characteristics are useful but all can be spoofed to one degree or another, potentially complicating attribution. Identifying the source of samples has come to the foreground more in the past decade as efforts have shifted to understand the nature of threats and their activities beyond the network perimeter. As part of this change, the motivations of attackers and their political status has become a factor of interest. [^1, ^9]

Analytically, the next step is understanding the relationship between the complexity and capability of a piece of malicious code and the nature of its authors. We do this by focusing on the discrete components outlined above, the propagation method, exploits, and payload. [^3, ^10] We examined a collection of malware samples which, through existing analytic techniques, have been attributed to a mix of state and non-state actors. Reviewing technical information available in the public domain for each sample and reverse-engineering a sub-set, we determined that there is a set of criteria by which state authored code can be differentiated from the conventional malware of non-state groups.

![](/bring/imgs/08_differenting_features.png)

The MAlicious Software Sophistication \(MASS\) Index, found in Figure 1, is a set of functional criteria which could be employed in discussing the high level characteristics of a piece of malicious code. The contents of the index are descriptive and qualitative in nature - observed features of code that has been previously attributed to state or non-state actors. [^2] This index is intended to serve as a collection of characteristics which might be intelligible to business leaders and the policy community in discussing the differentiable nature of state-authored code.

## Implications of Milware

A market like mechanism exists for groups to buy, sell, and trade components of malicious software. While estimates of the prices and popularity of different tools is a subject of debate, there can be little doubt that the vastly deeper pockets of state actors will impact these markets. The rise of milware may be pricing software vendors and other defensive organizations, operating through bug bounty programs, out of the market. More insidiously, the presence of states with financial resources to burn and an appetite for the latest and greatest vulnerabilities in widely used commercial software may well encourage substantial growth in the number and talent of individuals who participate in this market as sellers. [^11] As the prospect for financial gain increases, more and more people join in to sell their malicious wares to the highest bidder. Milware then offers the prospect of becoming a driving force in the sophistication and variety of malicious software components, especially vulnerabilities, available on the web.

Milware also reverses the traditional hierarchy of information security, where defenders have the onus of legitimacy and hackers are operating at the edge or outside of the law Existing legal tools for the location and prosecution of malware authors and distributors, already weak, presume targets who can be subject to a state's jurisdiction. Milware, developed and deployed by states, renders these tools little more than rhetoric. Limiting the use or development of milware is an inter-state monitoring and enforcement task more akin to conventional heavy arms sales or export control restrictions. Non-state actors can be pursued and prosecuted but states and their milware will largely be subject only to the state's own willingness to self-restrain or the ability for other states to compel the same. This constitutes a parallel enforcement and mitigation problem for all parties - malware tends to be large scale and much is indiscriminate.

## Conclusion

Measuring the sophistication of malicious software is challenging. However, the potential value of such an index across multiple disciplines and for the policy community is substantial. This is a first cut at developing such an index and demonstrating that political and computer science can play nice together. With increasing frequency of use and capability, state-authored code presents a challenge to the existing information security research and policy paradigm. Better understanding the threat and the larger implications of this trend can help drive a more complete and informed response.


## References

\[1\] Seth Hardy, Masashi Crete-Nishihata, Katharine Kleemola, and Adam Senft. Targeted threat index: Characterizing and quantifying politically-motivated targeted malware. In This paper is included in the Proceedings of the 23rd USENIX Security Symposium., pages 527-541, August 2014.

\[2\] Trey Herr, and Eric Armbrust, "Milware: Identification and Implications of State Authored Malicious Software." In NSPW '15 Proceedings of the 2015 New Security Paradigms Workshop, 29-43. Twente, Netherlands: ACM, September, 2015. doi:10.1145/2841113.2841116.

\[3\] Trey Herr, "PrEP: A Framework for Malware & Cyber Weapons." The Journal of Information Warfare 13 \(1\): 87-106, January, 2014. [http://dx.doi.org/10.2139/ssrn.2343798](http://dx.doi.org/10.2139/ssrn.2343798)

\[4\] Ekta Gandotra, Divya Bansal, and Sanjeev Sofat. Malware analysis and classification: A survey. [http://www.scirp.org/journal/PaperDownload.aspx?](http://www.scirp.org/journal/PaperDownload.aspx?) paperID=44440, May 2014.

\[5\] U Bayer, A Moser, C Kruegel, and E Kirda. Dynamic analysis of malicious code, August 2006. [http://dx.doi.org/10.1007/s11416-](http://dx.doi.org/10.1007/s11416-) 006- 0012- 2

\[6\] I You and K Yim. Malware obfuscation techniques: A brief survey, November 2010.  
[http://dx.doi.org/10.1109/BWCCA.2010.85](http://dx.doi.org/10.1109/BWCCA.2010.85)

\[7\] A Moser, C Kruegel, and E Kirda. Limits of static analysis for malware detection, 2007.

\[8\] M Schultz, E Eskin, F Zadok, and S Stolfo. Data mining methods for detection of new malicious executables, May 2001.

\[9\] D Ddl F Li, A Lai. Evidence of advanced persistent  
threat: A case study of malware for political espionage, October 2011. [http://ieeexplore.ieee.org/xpl/](http://ieeexplore.ieee.org/xpl/) articleDetails.jsp?tp=&arnumber=6112333&url= http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs all.jsp%3Farnumber%3D6112333

\[10\] Nikolaj Goranin and Cenys Antanas. Analysis of malware propagation modeling methods, April 2008.

\[11\] Trey Herr, "Malware Counter-Proliferation and the Wassenaar Arrangement, Working Paper, February 2016.  [http://dx.doi.org/10.2139/ssrn.2711070](http://dx.doi.org/10.2139/ssrn.2711070)

