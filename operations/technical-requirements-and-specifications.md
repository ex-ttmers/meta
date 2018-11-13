> This wiki pertains to the customer-facing technical requirements and specifications that feed "TTM-TechCheck.pdf." The goal of this page is to be a living, breathing document that can, at any time, be polled and compared against the customer-facing document for potential edits.

## System Overview and Disaster Recovery

Think Through Math leverages a cloud infrastructure. This approach allows us to increase platform resources to support additional user load in a matter of minutes so that the system can scale quickly and efficiently. Think Through Math is load tested annually at twice its expected peak concurrent user load for the projected school year.

Think Through Math uses a third party to continuously monitor several performance metrics, including uptime, average page load time, and database transaction time. IT staff are alerted if any of these metrics fall outside of our internal standards.

Think Through Math maintains relationships with multiple datacenter providers in geographically distributed locations. In the event of a disaster that completely disables the facility that houses the primary application, Think Through Math’s Disaster Recovery Plan is designed to restore full application functionality within 24 hours. Think Through Math’s application source code and other critical business documents are backed up nightly and stored in multiple, geographically dispersed locations.

As a hosted web-based application, all customer support, teaching and administrative functions for the platform can be provided from any location with Internet access and would be supported by Think Through Math personnel from alternate locations should there be any business interruption at Think Through Math’s office facilities. This environment establishes continuity of service to our customers.

Additionally, communications (email) and customer resource management (CRM) applications are web-based applications. In the event of an office facility business interruption, these internal systems would be accessed from alternate locations without interruption to our business. In addition to email, all Think Through Math personnel have access to a web-based text message service and a web-stored company phone directory to provide multiple modes of communication within our organization in the event of any disruption to office facilities.

## IP Configuration
Think Through Math uses cloud-based infrastructure to provide scalability to ensure student wait times are minimized. Because of this, it is not possible to predict the IP addresses that will be used to serve content in the application. White listing based on IP address is not a supported configuration, as source IP addresses can change over the course of student usage.

## Technical Requirements
Think Through Math supports the following as _minimum_ technical requirements.

### I. Operating Systems and Browsers
#### Using Think Through Math on a Desktop Computer, Laptop Computer or Google Chromebook

Think Through Math supports the following operating system and browser configurations. Think Through Math recommends Safari, Firefox and Chrome run in 'evergreen' (auto-update) configuration. This ensures that browsers are always kept up-to-date with the latest security and performance patches.

| Operating System | Browser |
|:--------------|:----------------|
| Microsoft Windows | Internet Explorer 9+<br/>Firefox 27+&nbsp;<sup><em>F</em></sup><br/>Chrome 31+|
| Apple OSX | Safari 6+<br/>Firefox 27+&nbsp;<sup><em>F</em></sup><br/>Chrome 31+|
| Chrome OS | Chrome 27+|

**Internet Explorer 8** is supported as a deprecated experience only. Users who rely on **Internet Explorer 8** will be unable to experience many new features of Think Through Math.

<sup><em>F</em></sup>&nbsp;**Adobe Flash** is not required except when using the Firefox browser. Installing **Adobe Flash** may enhance your Think Through Math experience by providing audio and interactive whiteboard capabilities when engaging with live certified math teachers, as well as providing customization of student avatars. To enable **Adobe Flash** to work properly with Think Through Math, ensure that content is permitted from the domain fms.thinkthroughmath.com on the following protocols (ports): TCP (1935) and TCP (80).

#### Using Think Through Math on a Tablet or Mobile Device
Think Through Math supports 7" devices in landscape configuration and 10" or larger devices in both landscape and portrait configurations. The application is delivered through the browser without requiring a separate app download. Think Through Math is supported in the following device, operating system and browser configurations.

| Device | Operating System | Browser |
|:--------------|:----------------|:----------------|
| iPad 2 | iOS | Safari 3.2+ |
| iPad Mini Second Generation | iOS | Safari 3.2+ |
| Galaxy Tab | Android | Android Browser 3.0+<br/>Chrome 31+ |
| Kindle Fire | Android | Android Browser 3.0+<br/>Chrome 31+ |
| Surface | Windows | Internet Explorer |

Experience may vary across other Android or Windows devices.

### II. Display Resolution
Think Through Math supports the following display resolutions.

| Device | Resolution (Pixels) |
|:--------------|:----------------|
| Desktop Computer<br />Laptop Computer<br />Google Chromebook | 1024 x 768 |
| Tablet<br />Mobile Device| 7" devices (landscape only)<br />10" or larger devices (landscape and portrait) |

### III. Network and Performance
&gt;1Mbps broadband connection speed between the school or student's device and the internet. Think Through Math uses no synchronous media or video. Actual application performance depends on concurrent student use at any location.

### IV. Cache Server/Content Filtering
Content displayed on Think Through Math may be loaded from any of the following domains.

| Domain | Protocol (Port) | Usage |
|:--------------|:----------------|:----------------|
| &#42;.thinkthroughmath.com | HTTP (80)<br/>HTTPS (443) | General application content |
| thinkthroughlearning.zendesk.com | HTTPS (443) | Customer support trouble ticketing and chat |

If website restrictions are enforced on iOS devices, https://lms.thinkthroughmath.com should be added to the list of allowed websites.
