Typically, when you add google analytics to a site, any traffic that is from a browser with an ad blocker will not be tracked. This is because the ad blocker will block all web requests to google analytics. 

## High level explanation
A proxy server allows your site to send web requests to google analytics indirectly, thus bypassign the ad blocker. However, this was not something that I wanted to set up even though there are several resources available outlining this process. I'll link two such resources below. For myself though, I found it is instead possible to simply using an IFTTT flow as a quasi-proxy server. The amount of information that is recorded is a much much smaller subset of all the information recorded with a proxy server simply forwarding all requests, but it is more customizable. The flow that these instructions will set up are only for recording page views on google analytics. The steps to set this up are to:
1. create an IFTTT flow forwarding requests to the google analytics measurement protocol collect api endpoint 
2. make your site submit requests to ifttt whenever the url changes

## Details
### 1. Creating an IFTTT flow
You'll have to make an IFTTT account. IFTTT allows for anyone to have unlimited free flows, and only charges companies to make their services available to IFTTT. To create a flow, you simply have to choose a trigger and an action. 

For the "this" trigger, select the "Receive a web request" trigger available from the "Webhooks" service. Make the event name whatever you'd like, you will need it later.

For the "that" action, again from the "Webhooks" service, select the "Make a web request" action. Enter `https://www.google-analytics.com/collect?v=1&tid=<<UA Tracking ID>>&cid=ifttt_user&t=pageview&dp={{Value1}}` into the URL field. Replace the `<<UA Tracking ID>>`` with your specific google analytics tracking id. A string will be passed from the trigger for `Value1`. Select `POST` as the method, and leave the other two fields. 

Now, go to your services and find the documentation for the "Webhooks" service. This page will have your key, which you will need later.

### 2. Making your site submit requests whenever the URL changes
We will be sending requests to IFTTT from the frontend of your site whenever the url is changed. 

The function below will send the request in the appropriate format to the IFTTT endpoint. You'll have to replace `<<event_name>>` with the event name you set before, and `<<key>>` with your specific key.
```
function logToGA(){
    const pathname = window.location.pathname
    fetch('https://maker.ifttt.com/trigger/<<event_name>>/with/key/<<key>>?value1='+pathname, {
        mode: "no-cors",
        method: 'POST',
    })
}
```

To call this function whenever the url changes, I would suggest adding it to the functions for the following events (the examples below are if this is the only thing you have to do for those events, but if you have existing code for those events, then simply add the call to `logToGA` somewhere in the function):
* `window.onload`
```
window.onload = (event) => {
    logToGA()
};
```
* `history.pushState` and `history.replaceState`
** note that for these functions, you have to basically intercept anytime they are called and that this can be done by taking advantage of JS prototypes and setting up the pushState/replaceState function to have an line to execute in some kind of initialization function
```
const pushStateFn = history.pushState;
history.pushState = function () {
    pushStateFn.apply(history, arguments);
    logToGA()
};
```

I added all of this (with some modifications specific to my Notion site hosted using code from [fruition](https://fruitionsite.com)) to the head element in all of my pages like so:
```
<head>
<script>
    function logToGA(){
        const pathname = window.location.pathname
        if (!pathname.startsWith('/app') && !pathname.endsWith('js') && !pathname.endsWith('css') && !pathname.startsWith('/api')) {
            var post_pathname = pathname.slice(1) == "" ? "home": pathname.slice(1)
            if(pages.indexOf(pathname.slice(1)) > -1) {
                post_pathname = PAGE_TO_SLUG[pathname.slice(1)]=="" ? "home" : PAGE_TO_SLUG[pathname.slice(1)];
            }
            fetch('https://maker.ifttt.com/trigger/page_visit/with/key/p6v_orT4YDYs8cTPH8saj?value1='+post_pathname, {
                mode: "no-cors",
                method: 'POST',
                headers: {
                    'Content-type': 'application/json; charset=UTF-8'
                }
            })
        }
    }
    window.onload = (event) => {
        logToGA()
    };
    const pushStateFn = history.pushState;
    history.pushState = function () {
        pushStateFn.apply(history, arguments);
        logToGA()
    };
</script>
```

## Proxy server resources
* [github repo for proxy server](https://github.com/NYPL/google-analytics-proxy)
* [freecodecamp post for proxy server](https://www.freecodecamp.org/news/save-your-analytics-from-content-blockers-7ee08c6ec7ee/)

---
If you found this helpful and want to support me, consider buying me a coffee!

[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/U7U81Q15P)