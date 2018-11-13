# HTML Emails

## Mail Clients

Updated Information about email clients and usage overall can be found at [Email Client Marketshare](http://emailclientmarketshare.com/), by Litmus.

1. Apple iPhone 27% -1.08
1. Gmail 17% +0.41
1. Apple iPad 12% -0.19
1. Outlook 9% +0.74
1. Apple Mail 8% -0.08
1. Google Android 7% +0.22
1. Outlook.com 5% -0.09
1. Yahoo! Mail 4% -0.1
1. Windows Live Mail 2% -0.05
1. AOL Mail 1% -0.01

## What do Clients Support?

While Email clients will generally support many CSS properties in terms of color, or borders, email clients like Gmail strip all content out of the head tag of the html. All of your CSS styles must be inline in order to work across many clients.

To stay up-to-date on all "what is supported" questions, refer to: [The Ultimate Guide to CSS by Campaign Monitor](https://www.campaignmonitor.com/css/).

### Layout Concerns

The following CSS properties do not work in Outlook, or versions of Outlook:

1. Float
1. Padding
1. Margin
1. Height
1. Min/Max-width

The solution is to code in nested tables. You should have a wrapper table that is set to a width of 600px. If your email is not single blocks on top of each other, you should use tables within tables. This minimizes the chances that your layout will be screwed up because of column-spanning.

Example: ![Text with Tables](https://cloud.githubusercontent.com/assets/4049290/6359881/14c5b94e-bc44-11e4-86c2-dc18b2f59cb7.png)

Each of the above borders represents a table in the email.

### Image Support

Do not try to design all image emails. Many email clients block images by default so users will see nothing. If you have to use an image, specify it's height and width or it could bust your layout. Also, include alt text so that if the image is blocked, the user will know what the image is.

## Mobile Email

### Android/Windows Text issues

Android will automatically scale up the size of a text in an email by 110%, which can totally throw off a table layout. This can be fixed by writing * {-webkit-text-size-adjust:none;}

### Responsive Email

Unlike Responsive Web Design which traditionally starts small and then adjusts as you go bigger, Responsive Email Design has to start BIG and go down. Because email clients like Gmail and others specifically block all content in the head tag, the default styles need to be in the inline styles so they will be picked up.

#### Media Queries

Media Queries do not work for inline styles and so they should be in an embedded stylesheet in the head section. Adding support for mobile devices should be done after the inline styles are created. Every style in your media query needs to be declared to be important to overwrite the inline styles in the HTML.

#### Layout Adjustment

Just because the styles will be picked up and applied, doesn't mean they will work. Float still doesn't work on mobile clients, so you will have to use Inline-Block elements. Most likely, you will declare a table-cell to be "display:inline-block;" and give a width to take up a certain amount of space. Changing the text-align property of the element will help it change position from right to left.

## Tools

### Testing

Litmus allows you to send emails to a wide range of devices and then see screenshots of these emails. It also has a built-in "Interactive Testing Tool" that let's you make minor change to the html and see the updates. This is very helpful for troubleshooting small problems. However, it's difficult if it's an email-wide problem affecting multiple elements because all your CSS is inline.

[Putsmail by Litmus](http://putsmail.com) lets you send an HTML email to anyone, for free. The recipient just has to "Ok" getting mails by putsmail, so let your testers know.

### CSS Inliner

* [Mailchimp](http://templates.mailchimp.com/resources/inline-css/)

## Resources

* [MailChimp Resources](http://templates.mailchimp.com/resources/)
* [Campaign Monitor's CSS Guide](https://www.campaignmonitor.com/css/)
* [Email Marketshare by Litmus](http://emailclientmarketshare.com/)
* [Litmus Resources](https://litmus.com/resources)
* [Will it Work? by Campaign Moniter](https://www.campaignmonitor.com/resources/will-it-work/)

## Workflow

I've found that the easiest way to create an html email is to open a new folder in my documents with the following files:
* index.html
* mobile.css
* [image].jpg

### In Development

The content for the email goes into the body section of the index.html. The styles that need to be inline can be written in an embedded stylesheet in the html and then the mobile styles are written in a media query inside the mobile.css file and linked within the html file.

I work out the layout in Google Chrome and Mozilla Firefox (Firefox has inline-block spacing issues, so it's best to get it out of the way first).

### Testing

#### Device Testing

Then I move onto Testing it with actual devices in the office. I try to send to at least 2 people with iPhones and 2 people with Android devices who don't mind me spamming them. It also helps to try and view the email through multiple apps. I send it to my gmail as well as my work account so I could see it on the Inbox App for iOS, as well as the Mail app.

I use [Putsmail by Litmus](http://putsmail.com) to send them the email, but first I run the index.html file content through Mailchimp's CSS Inliner. I copy the result into Putsmail, and then I remove the link to the media.css and copy the media.css content into the style tags in the head of the index.html document.

This might seem crazy, because Putsmail has an inliner, but I didn't want to use it since you can't see the result before you send it. Also, I wasn't sure how it was going to handle the media query content, so I prefer to use Mailchimp's inliner and then copy the results into Putsmail.

As a note: Mailchimp's CSS Inliner will leave the original embedded styles it inlines in the style section. I delete those because it's unnecessary to have it once it's inline. Once I delete those, I copy the media styles into the style section.

#### Testing on Litmus

I created a 7 day trial account for myself with Litmus. Then I used the same method as above to send emails through Litmus and use their screenshot system. They also have an interactive debugging tool. It's not as seamless as using Inspect Element, but it's pretty damn good. If I decide to change something, I resend the email to make sure it's not something that will screw something up to fix something.

It might seem weird to test on actual devices before testing on Litmus, but Litmus can sometimes take a while for the screenshots to populate. With Putsmail, I can view the email in Outlook, Outlook.com, Gmail (in all browsers), Inbox and the iOS mail app almost instantly. When I'm bugging Seejee, I also have instant access to Android. This let's me get a good percentage of devices quickly without any lag time.

You would want to test on Litmus, though, because versions of Outlook are totally inconsistent, even if the were fine the year before and year after. When I was doing District Admin Emails, it would look fine in Outlook 2011, 2015 and 2007, but terrible in 2013 and 2010.
