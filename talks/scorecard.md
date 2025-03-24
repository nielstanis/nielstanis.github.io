---
layout: talk
title: Reviewing NuGet Packages security easily using OpenSSF Scorecard
---

Several studies shown that round 80% of our applications consist of other people's code because why would you re-create something that's already made by someone else? But with using a NuGet Package that is developed by others, we also put a lot of trust in it, which might result in bigger security problems later. 

Of course, it's always a good idea to get updates of libraries in case of a bug fix related to a functional and/or security issue found. But will that be enough? What about packages that have malicious code inside? Even related to your own supply-chain security, any problem in the package its supply-chain implicitly means your supply-chain is compromised as well!

Would it not be nice if there is a better way to review NuGet packages for security? An easier way to perform an assessment based on certain aspects of the package that will tell you more about the package its software security. With the introduction of Scorecards the Open Source Security Foundation (OpenSSF) exactly tries to achieve that. You could consider a Scorecard being the equivalent of a nutrition labels put on food you buy in a supermarket. It will allow you to see what's inside and determine if you want to eat it or not.

In this session we start out with different area's covered by of OpenSSF Scorecards, like how well it's maintained, does the build have dangerous workflows, and does the project use other security tools to check for problems? We're also going to identify additional area's for NuGet packages in which we could add additional information related to reproducibility, insights on what .NET APIs are used, and security review of the codebase. All combined will give us the ability to assess a NuGet package its security posture more easily and improve our own application security.

## Videos

[![NDC Security 2024 - Reviewing NuGet Packages security easily using OpenSSF Scorecard](https://img.youtube.com/vi/4B8LcAlMbsE/0.jpg)](https://www.youtube.com/watch?v=4B8LcAlMbsE "NDC Security 2024 - Reviewing NuGet Packages security easily using OpenSSF Scorecard - Niels Tanis")
[![Assessing 3rd Party Libraries More Easily With Security Scorecards - Niels Tanis - OWASP Global AppSec EU 2024](https://img.youtube.com/vi/BZy5UaiAMDY/0.jpg)](https://www.youtube.com/watch?v=BZy5UaiAMDY "Assessing 3rd Party Libraries More Easily With Security Scorecards - Niels Tanis - OWASP Global AppSec EU 2024")

## Events

- UpdateConference 2024 - [Slides](https://github.com/nielstanis/UpdateConference2024/blob/main/Slides/UpdateConf2024-Slides.pdf) - [GitHub](https://github.com/nielstanis/UpdateConference2024/)
- DevDay 2024  - [Slides](https://github.com/nielstanis/devday2024/blob/main/Slides/DevDay2024NuGet.pdf) - [GitHub](https://github.com/nielstanis/devday2024/)
- Copenhagen Developer Fesitval 2024 - [Slides](https://github.com/nielstanis/cphdevfest2024/blob/main/Slides/CPHDevFest2024-NuGet.pdf) - [GitHub](https://github.com/nielstanis/cphdevfest2024/)
- WeAreDevelopers World Congress 2024 - [Slides](https://github.com/nielstanis/wearedevs2024/blob/main/Slides/Tanis%20Niels%20ScoreCards.pdf) - [GitHub](https://github.com/nielstanis/wearedevs2024/)
- NDC Oslo 2024 - [Slides](https://github.com/nielstanis/ndcoslo2024/blob/main/Slides/NDCOslo2024.pdf) - [GitHub](https://github.com/nielstanis/ndcoslo2024/)
- OWASP 2024 Global AppSec Lisbon - [Slides](https://github.com/nielstanis/appseceu2024/blob/main/Slides/AppSecEU2024-slides.pdf) - [GitHub](https://github.com/nielstanis/appseceu2024/)
- Boston .NET Architecture Group - 15th of May 2024 - [Slides](https://github.com/nielstanis/bostonarch2024/blob/main/Slides/BostonNETArch-NuGetSCC.pdf) - [GitHub](https://github.com/nielstanis/bostonarch2024)
- FutureTech 2024 - [Slides](https://github.com/nielstanis/futuretech2024/blob/main/Slides/FutureTech-NuGetSCC.pdf) - [GitHub](https://github.com/nielstanis/futuretech2024/blob/main/Slides/FutureTech-NuGetSCC.pdf)
- BitBash 2024 - [Slides](https://github.com/nielstanis/BitBash2024/blob/main/Slides/BitBash2024-NuGetSCC.pdf) - [GitHub](https://github.com/nielstanis/BitBash2024/))
- NDC Security 2024 - [Slides](https://github.com/nielstanis/ndcsecurity2024/blob/main/Slides/NuGetSSC.pdf) - [GitHub](https://github.com/nielstanis/NDCSecurity2024/)