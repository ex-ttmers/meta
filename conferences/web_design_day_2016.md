# Web Design Day

> Pittsburgh, PA

> Friday, June 24, 2016

## Revolutionize Your Page: Real Art Direction on the Web
> Jen Simmons

Jen Simmons from Mozilla talked to us about some of the upcoming css technologies that will help make it easier to avoid boring layouts and grid systems (where appropriate) and use 'art direction' or 'editorial design' to *communicate* a story with presentation. She reminded us to keep tooling needs and design process separate; this allows us to think outside of the norm, and really design for our content.

Use of negative space and visual hierarchy to guide the user are some of the techniques been a bit neglected lately due to the tools that we've been using, but some of the upcoming CSS magic should help us address that; at least for newer browsers (we'll have to rely on progressive enhancement for most of these, so remember that some users will have different experiences). Here are some of the super cool things she showed us:
  - Drop Caps using initial letter
  - Browser support with CSS (@supports)
  - Viewport units
  - Object fit
  - Clip paths
  - CSS Shapes and the amazing CSS Shapes editor plugin for Chrome
  - Flexbox and multicolumn (which was buggy, but is getting better)
  - CSS Grid

Jen encouraged us to learn these new techniques by experimenting and making demos, and pointed us to her website where she has a number of demos and links: [labs.jensimmons.com](http://labs.jensimmons.com). Also, [www.layout.land](http://www.layout.land) is coming soon with lots of examples and demos of this upcoming tech!

## How to Talk to Humans
> Sharon Steed

Despite having a stutter that has impacted her life for as long as she can remember, Sharon Steed bravely gave a short talk about empathy in communication. She asked what the world would be like if, instead of thinking about how _I_ can explain something, how do _I_ feel about something, or how can _I_ add value, people approached conversations and idea sharing from the perspective of asking how the _other person_ can help grow an idea. Empathy fuels connection, and empathetic communication drives collaboration, so speak up and encourage others to do so, think about the listener, and keep empathy in mind when you are communicating with others.

## Hold Fast: Managing Design Teams When Projects Go Sideways
> Aaron Irizarry

Aaron says that design is a journey, and like any journey, there will be storms along the way. It's important to "hold fast" and weather the storm until things are ok again, but that can be tough to do! Companies are moving at a faster and faster pace, and software isn't getting any less complex. Neither are people; there are so many things that can affect a project, including departmental make-up, internal politics, team distribution, understanding of design's role, and a limited budget and lofty goals.

We need to remember that shipping is just the beginning. Managing stakeholders, teams and processes throughout the lifetime of a project requires trust, which you can build with transparency and by taking responsibility when things under your control go awry. Avoid unnecessary conflict; progress towards positive resolutions by focusing on the product and customer. Don't be afraid to admit when you don't know something. Management and team support is just another thing that we can apply observation and design to.

Your process should provide a good troubleshooting starting point, but be prepared to ditch it if you see that it isn't working. You have remain flexible to weather storms, and remember that the needed solution might not be the ideal solution.

## Taming Complexity, One Story at a Time
> Kate Daly

Kate Daly gave a short talk about how story mapping is a great tool to handle complexity.

Tools required:
  - Post Its
  - Sharpies
  - Walls
  - 0 Naysayers

Cover the walls with ideas; see how they converge!

## Design for Real Life
> Eric Meyer

Imagine checking FaceBook on the first Christmas after a loved one has passed away. How does their playful 'Year In Review' video, that places a photograph of the child who can no longer be there to celebrate in the middle of a jubilant holiday party, make you feel? Eric Meyer started off his talk with his somber story, having experienced this not long after his young daughter lost her battle with cancer. While this feature appeals to a certain demographic, it can be a horrifying experience for 'edge case' users.

Eric urges us to remember that we are designing for the __World Wide__ Web, and we don't get to pick who visits our site, or what their context might be like. He recommends calling 'edge cases' 'stress cases' instead - and remembering that these are real people in real situations. Certain design choices can actually *hurt* them, especially in this emotional uncanny valley we find ourselves in these days, so plan for the worst cases by making sure you represent your personas in different contexts - including in crisis.

Some strategies for designing for real life people and scenarios include making sure that you communicate intent to mitigate risk of hurting users in stress cases, being careful with your voice and tone (MailChimp has a good guide for this), and not frustrating users by making them feel like they have to lie (allow non-answers where possible).

## JavaSocks
> Caitlin Steinert

Caitlin gave a short talk about how one passion has led to another throughout her life, and how knitting patterns became her gateway into javascript.

## Drawing Inspiration for UX and Teamwork from Classical Music
> Smitha Prasadh

This talk was basically a conceit, comparing product design and development to making music. She drew parallels between team size and ensemble size, emphasizing the collaboration that makes a team greater than the sum of their parts. Rehearsing and performing are likened to development and releasing.

On a more personal level, learning to perform classical music can provide strategies for learning to deal with design and development problems. Learning a piece of music can be hard - it's ok for problems to be hard. We have to break problems apart into manageable chunks, but don't loose sight of the whole. Patterns (and broken patterns) can be clues to untangling a problem. Remember that you are not always the melody (or the most visible part of a team) and underlying details are important, too. Also, what you _can_ do may be enough, don't always be comparing yourself to others.

As far as the audience, both classical music and products need to keep in mind lowering barriers to entry to attract and retain people.

## I Was Fired From My First Post-Grad Job
> Bridget Reed

This was a short talk, which was good because I didn't understand the point of it _at all_. Bridget was very enthusiastic and was hired on the morning of the conference though, so good for her.

## Better Web Accessibility You Can Do Right Now
> Robert Jolly

A short talk about a few things you can do to help make sure your site is accessible:
  - Make your defaults accessible
  - Treat elements semantically
  - Watch color contrast
  - Test with keyboard only

## Adaptive Content, Context, and Controversy
> Karen McGrane

Karen also emphasized that the web is for everyone, and we can't pick who accesses our site -- or what device they are using. She cited studies where it turns out that people want to do basically the same things regardless of what device that they are using; so while it may make sense to highlight certain things depending on whether you are on mobile or a desktop, make sure that both versions get the same information and services.

She talked about hearing that people want an 'adaptive' website, or something 'beyond responsive'... what exactly does that mean? Responsive isn't necessarily good enough for a site to work for anyone in any context. She compared responsive design with adaptive design and the old m dot site to convey a better idea of what 'adaptive' design is:

|   | Responsive | Adaptive | M. Site |
|---|------------|----------|---------|
| Same content across devices | yes | can serve something different | nope |
| Same url | yes | yes | no |
| Same design | yes | not necessarily | no |
| | client side | server side | server side |

So... adaptive design is _serving something different_ based on the device accessing the site. This means the client is going to have to detect the device and send that information back to the server so that the content can be determined -- this isn't necessarily a solution that minimizes complexity!

Generally, it is best to build a responsive design where you can, and keep adaptive content to a minimum (Karen noted a site that uses responsive design except for the audio player, which varies per device). NPR tried using COPE (Create Once Publish Everywhere) strategies and adaptive design to serve the same content in different ways to different devices, but it turned out that the maintenance cost was too high, and they moved back to a mostly responsive site.

Adaptive design does give you the ability to target based on device, user, or context (defined as any info you can glean from device sensors -- be careful because it may not mean what you think it means, nor does it tell you what the user actually _wants_), which can be useful if your content is separate from your design and granular enough. When should you use it? Content prioritization based on device is one situation where it might be appropriate. Another could when your devices are very different and in different contexts (dashboard on tv vs website).

In conclusion, adaptive design can be a great tool in your toolbox, but it can be costly to implement and may not be worth it in most situations. You also need to be very careful about the assumptions you are making about context based on what you can glean from a device.

## Designing a Design System
> Jina Bolton

Jina is the lead designer on the design system that Salesforce uses, and talked to us about what design systems are and how to build them.  She says that a style guide is not a design system; a style guide is an artifact, and a design system is a living, funded project, that encompasses an entire ecosystem (which is really big for Salesforce, including plugin developers, lots of teams, etc). There's a lot of trickiness in  dealing with the complexity that comes from legacy code, lots of people, and different products - a design system is something that can be useful in dealing with that trickiness.

A design system should be a _clear vision_ to align efforts across these divides. It is important that the design system is a product itself... don't put off tough decisions. Drive your decisions with design principles; Salesforce has these (in priority order): clarity, efficiency, consistency, and beauty. Aligning to principles is more important than aligning to particular standards. You need to evangelize your standards to make sure that everyone is on the same page. When designing, you should apply these standards, do research ( don't just copy), and embrace your constraints.

Salesforce is a large organization, so they keep everyone on the same page by having a centralized design system team, with people who cycle in and out, as well as federated contributors from other teams. They also embed members of the core team in other teams to help encourage communication and collaboration between the design team and the rest of the organization.

Building a design system usually starts with cataloging existing components (think Brad Frost), standardizing and taking copious notes on components (and when they should be used) -- "If you don't document something, it doesn't exist." You also need to create design guidelines, listing the usual things like color, typography and icons, but don't forget to explain why and where to use these things! Once these details are hashed out, designers are free to look at the bigger picture, and spend time doing things like building user flows, and focusing on the right questions. A design system helps teams share responsibility for a quality product, and empowers designers and developers to get things done. It allows for greater fidelity from concept to user by reducing inefficiencies in the system.

Salesforce's design system does not include any javascript; since they have outside developers they needed to keep their design system system agnostic. Instead, each component's interaction patterns are well documented. They use design tokens, which are named entities that store design values. Design specs will use these tokens, which a utility (Theo) will turn into whatever format they need.

Some other tips - don't make things until you need them, if you *must* use a framework, it's probably best to delete everything you aren't using. Handle deprecation with sass deprecate. To keep your system from becoming a zombie style guide, you have to support your design system adoption with education and consultation. Evangelize and show the impact of the system. Remember that patterns are not dogma - they should change and adapt. Allow evolution.

Jina works on [SLDS](https://www.lightningdesignsystem.com/), which is [open source](https://github.com/salesforce-ux/design-system), and also organizes [a design system conference](http://clarityconf.com/).

## Other thoughts...

Underlying themes that kept surfacing at this conference were empathy and dealing with complexity in people and real life contexts. There weren't too many talks that were very techical, more stuff at a higher level; which is good to think about now and then.
