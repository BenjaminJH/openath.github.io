---
layout: post
title: England Athletics Mobile Licence Checker
author: Andy Robinson
---

We're working to make it a bit easier to check competition licences with our new 
<a href="/licencecheck/">mobile licence checker</a>, and an easier <a href="/licencecheck/#apidocs">API for web developers</a>.

It's well known within the sport that the rules of competition have just changed.  For some time road races have offered discounts to registered athletes. But <em>from April 2016 Track and Field athletes must be registered</em>, so strict checks will be required before you can enter.  

England Athletics have several tools available on their site for bulk-checking but none that works well on a mobile device.  So we have built a mobile-friendly licence checker, which officials can bookmark and use on their phones this summer.

<div class="row">
    <div class="col-md-6">
        <a href="/licencecheck/">
        <img class="img-rounded" 
            style="border: 1px solid; width:100%" 
            src="https://raceresults.reportlab.com/media/pix/RLIMG_e441ec9b5bf6e4a78a37b56f2bdc784c.png" 
            alt="Input screen"/>
        </a>
    </div>
    <div class="col-md-6">
        <a href="/licencecheck/">
        <img class="img-rounded" 
            style="border: 1px solid; width:100%" 
            src="https://raceresults.reportlab.com/media/pix/RLIMG_7ed3c02fce1c6836222b94d2639a999b.png" 
            alt="Successful check"/>
        </a>
    </div>
</div>

You can look up by first name, last name and date of birth, or by registration number.  It also knows about common nicknames, so you don't have to know whether the athlete is registered as Andy or Andrew.

## Building into your own web sites ##

We built this kind of code into our own entries system, which is taking entries for four English counties at the time of writing.  (We can do the same for you - contact opentrack@reportlab.com if interested).


England Athletics has an interface ("API") which lets developers look up athletes, as we are doing here.  But it uses a complex protocol, SOAP.  We made a simpler wrapper around it which works with very simple URL calls you can make in your browser. We've 
<a href="/licencecheck/#apidocs">explained this here</a>.   

We hope that web developers can now do checks at the time people enter races, using simple Javascript in a web form; or write their own code to do checks within spreadsheet macros.

If you'd like to use this, you're most welcome.  Please 
<a href="mailto:opentrack@reportlab.com">email us</a> to let us know as a courtesy, or join in with our 
<a href="http://forum.opentrack.run">athletics developers' forum</a>.   

It would also be really helpful if you can "like" or "share" the tool itself.

Finally, we would like to say a public thank-you to Richard While at England Athletics for his timely support and help with the API.

- Andy Robinson
