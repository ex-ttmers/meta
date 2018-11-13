# Untangling Apangea Teacher / Admin Styles

## Where we are

The css in apangea is not structured very well -- it's a hodgepodge of several completely different attempts to organize it. The most recent (and hopeful) are the remnants Pattern Lab in our `ux-patterned` folder...

There are a proliferation of styles, some of which are relevant, and some of which are accidental...

## Where we want to be

Someplace where we can confidently change the styles of a page, knowing that we will not accidentally break other areas of the application. Someplace where it is easy to find the style rules that govern the elements on the page that we're working on. Someplace where we can fix a major ux problem with a component once and know that we've fixed the interaction throughout the app. Someplace where we don't have meaningless nonsense to wade through to find code that materially affects *any* page. Someplace where we are able to maintain and present a clear, consistent user experience; one that we can study and iteratively improve.

## How do we get there?

Well, we're probably *not* going to get there through a big concerted effort to migrate the entire application at once (mostly because of business needs and the time required). We're going to have to do this piece by piece, one page at a time. It'll be a bit like untangling a ball of yarn, pulling out a piece here and loosening a piece there, until we get close to the end. 

The big things to remember are to __keep things simple__ and __keep separate things separate__. Here are some ideas and guidelines on how to get there:

### Simplify!

Fewer special snowflakes mean a more consistent, easier-to-understand experience for our users, as well as code that is easier to maintain.

If we already have a component that will do the job, use it. If we have one that is close... see if it makes sense to update it across the board.

If you make a new component, you'll have to use your judgement as to whether it makes sense to add it to `ux-patterned` so that it can be shared or to keep it namespaced so as not to get it tangled up with other pieces in other areas of the application.

We should strive to get rid of any 'extra' markup -- if you can get rid of nesting or divs without content of their own for example, do it! Nothing is sacred! :red_circle: :hocho: :heart: :red_circle:

#### CSS structure

I don't think the admin / teacher portal css is ever going to be as clean as avatar builder or lesson player, but I think we can get it a lot closer. Here are some guidelines to follow that may help us get a handle on the css in Apangea:

- Make a 'module' for a reasonable area of the application (possibly for all of the pages that a controller is in charge of). This 'module' will have its own scss file. 
  - The controller should import that module, which will contain all module-specific styles
  - That module should import ux-patterned, so that we have access to our reusable styles
- In the page markup, add classes that are 'namespaced' to each region of the page -- preferably something referring to the module and then a descriptor of what it is... `rollupReport-summaryTitle`, for example.
- In the module, add styles for those page markup classes. If an element on the page is something that is in our catalog of reusable components, you can `@extend` the element's class with the component. If it is *almost* like something in our catalog, you can `@extend` the element with the component and then add more styles that are specifically for that element. This should help us maintain reusability without entanglement.
- Avoid styling html elements directly -- where possible, add a class and style rules for that class instead.

### Progressive page conversion

When you come across an 'Old Apangea' page, try to get it up to date. So far, we've been operating under the rule that if you touch a page, make sure that it is in `.haml` (and not `.slim`). Here are some other things that we should do when we work in the admin / portal bit of apangea:

- Convert any `.slim` pages to `.haml`
- Update the controller to use the 'ux_patterned' layout
- The CSS structure guidelines above give us a recipe for 'upgrading' the page styles:
  - Make a new scss file for the 'module' (if there isn't one already) and use it in the controller
  - Go through the markup and simplify where possible, removing generic divs that don't really do anything / are part of the old admin/teacher css
  - Use namespaced classes for the elements on the page
  - Fill out the new module scss (some of this may be pulled out of the old css and tweaked, but hopefully you get a lot of what you need for almost nothing from our ux-patterned components)
- Delete any old styles that can be deleted

An example of a page that was updated this way is activity_previews. Some example PRs:

- [thinkthroughmath/apangea#2851](https://github.com/thinkthroughmath/apangea/pull/2851)
- [thinkthroughmath/apangea#2732](https://github.com/thinkthroughmath/apangea/pull/2732)
- [thinkthroughmath/apangea#2729](https://github.com/thinkthroughmath/apangea/pull/2729)


This can take a little bit of time, and we're not always going to have that time. But if it is at all possible to do this cleanup, future developers will be thankful... and when it's all done we should be able to kill lots of old cruftiness! :D

### Guidelines for new pages

Use the 'ux_patterned' layout, and follow the CSS structure above.

### Visual catalog of components

This was one thing that Pattern Lab did right. A visual catalog of components is necessary for developers to know what components are available so that the application stays consistent and we don't reinvent the wheel. 

Once upon a time, we had Pattern Lab, but it was a beast that was never properly trained and it had to be put down. `stylesheets/ux-patterned` was born out of the remains of Pattern Lab, and is the beginning of a catalog of reusable components for the teacher / admin portal.

Our front end components in Apangea are really not all that complex; a more manual approach to cataloging shared components may work better. There are a number of style guide generators that will build a catalog based on comments or other documentation that would live alongside the css; Manda is currently exploring options there. Suggestions welcome!

In addition to a visual catalog of individual components, I think it would be super useful to have a catalog of page types and interaction patterns. But, one thing a time. :)

### Automated testing

Another important tool for helping us detangle the css in apangea is automated testing. [This epic](https://github.com/thinkthroughmath/storyboard/issues/1791) is starting us off in the right direction... More to come.
