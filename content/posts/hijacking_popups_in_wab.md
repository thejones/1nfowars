---
title: "Hijacking popups in WAB (esri)"
date: 2018-06-11T14:27:41-08:00
draft: false
---

Esri's popups IMO have always been difficult to work with. The latest event was that we needed to control a link so that it had `target="_blank" AND set some window properties. So, we tried using the default map creation but it strips out anything you might want to do. So, the solution that I used was to use a "headless" widget that gets loaded as part of the theme and use that widget to set up a listener on `this.map.infoWindow.domNode`. So, anytime a mouse-over happens I am notified. This seems extreme but also works well. With the event firing I am able to filter out that information I am interested in and reformat the Popup accordingly. 

```
postCreate: function postCreate() {
  var self = this; // <- Life saver right here.
  this.own(
    on(
      this.map.infoWindow.domNode,
      "mouseover",
      lang.hitch(this, function(content) {
        //var jack =lang.hitch(this);
        var link = dojo.query("a", content.toElement)[0];
        if (link && link.href) {
          if (dojo.hasAttr(link, "data-mylink")) {
            return;
          } else {
            if (link.href.toLowerCase().includes("datadisplay") && !link.href.toLowerCase().includes("somethingImportant")) {
              dojo.attr(link, "data-mylink", "true");
              var linkText = link.href;
              dojo.removeAttr(link, "href");
              dojo.style(link, "cursor", "pointer");
              dojo.connect(link, "onclick", function(evt) {
                window.open(linkText, "_blank", "scrollbars=yes, resizable=yes, height=500, width=700");
              });
            }
          }
        }
      })
    )
  );
}

```

I tired to make this piece stand on its own so you will see some AMD imports and some Global dojo references. I hate AMD with a passion so I use it **WAY** more than I should. Either way. I check if I have `data-mylink` and if so I end early if not I reformat and change the link text. 
