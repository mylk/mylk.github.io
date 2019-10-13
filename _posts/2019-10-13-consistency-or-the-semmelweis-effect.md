---
layout: post
title: Consistency or the Semmelweis effect?
---

You may have found yourself proposing to your team some "radical" changes. It could be any of these:

- follow official language / community coding standards
- update an obsolete application or obsolete dependencies
- improve the architecture of an application
- improve the architecture of distributed applications

Even if you well-justified your proposal, you may have received the negative answer that "it was made
this way and we have to keep the consistency". Well, have just witnessed the Semmelweis effect!

> The Semmelweis effect is a type of confirmation bias and belief perseverance. Its a
reflex-like tendency to reject new knowledge because it contradicts established norms.

> The term derives from the name of a Hungarian physician, who in 1847 discovered that childbed fever
mortality rates fell ten-fold when doctors washed their hands between patients or, most particularly,
after autopsies. His fellow doctors rejected his hand-washing suggestions, refusing to believe that
hands could transmit diseases.

Well, in our case, even if you talk about established practices of the industry you may face that
resistance, where the current solution is taken as inherited wisdom written on stone. Its not
personal, its just that tendency.

[#](#support-your-opinion-with-data){:.header-anchor}

## Support your opinion with data

If you still believe that your proposal is important, you must provide evidence. "Extraordinary
claims require extraordinary evidence", says the Carl Sagan standard!

So, prepare a brief presentation to prove why you really need that change. The presentation could
contain:

- logs (application or server)
- issues and bugs due to the lack of that change
- excerpts from the official documentation of your programming language or framework
- their changelog or release timetables
- news posts
- security incidents of your programming language or framework
- other development teams' blog posts
- Google trends
- even frequency on job posting sites

It won't be easy. You need data to make your proposal find its way to the backlog!

During this "information gathering" process, if your change is indeed not really needed or false,
you will have the chance to discover it yourself. Just keep your openness.

[#](#is-it-too-radical){:.header-anchor}

## Is it too radical?

Do not be surprised. You have to bring a detailed migration / upgrade plan to the team.
If your change is too radical, you may have to break it into distinct steps to ease the transition
to the new status.
