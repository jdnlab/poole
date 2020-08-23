---
layout: post
title: "Drovorub's Implication for Defenders"
---

Recently (2020-08-13), the U.S. National Security Agency (NSA) and U.S. Federal Bureau of Investigation (FBI) issued a joint [technical security bulletin](https://media.defense.gov/2020/Aug/13/2002476465/-1/-1/0/CSA_DROVORUB_RUSSIAN_GRU_MALWARE_AUG_2020.PDF) detailing the disclosure of a malware family, "Drovorub". At 45 pages, it's quite lengthy for most people; however, in my experience it was well worth the read. In this post I will attempt to interpret the report and various other OSINT resources to address the following points:

- Overview
- Takeaways for Network Defenders

## Overview

![Data Exfiltration Ladder Diagram](./figures/drovorub-1.png)

### Threat Actor
Russian General Staff Main Intelligence Directorate (GRU) 85th Main Special Service Center (GTsSS), military unit 26165, has been attributed per the NSA/FBI report. What do we know about this actor? The NSA/FBI public report is understandably lacking details on this, but we can turn to OSINT resources:

- This actor maintains the following aliases: APT28, Strontium, Fancy Bear [1]
- This actor is a military intelligence service, thus more likely to participate in computer network exploitation (CNE) and attack (CNA) operations. [lab52] The latter of which can significantly degrade or destroy computer operations.
- Cited, relevant TTPs of the actor as it pertains to Drovorub includes (rather unsurprisingly)
    - Endpoint file acquisition [lab52]
    - Data compression during exfiltration
    - 




## Takeaways for Network Defenders
- [Russian GRU 85th GTsSS Deploys Previously Undisclosed Drovorub Malware, U.S. NSA/FBI](https://media.defense.gov/2020/Aug/13/2002476465/-1/-1/0/CSA_DROVORUB_RUSSIAN_GRU_MALWARE_AUG_2020.PDF)
- [Lab52]()
[] [](https://openaccess.leidenuniv.nl/bitstream/handle/1887/64569/Pols_P_2018_CS.pdf?sequence=2)
[2] [RedHat Security Advisory](https://access.redhat.com/articles/5320961)